# Redshift pipeline — ingest and export

Two Jenkins jobs, one Redshift cluster, and an S3 staging bucket. That's the whole BI
pipeline. Back to [[README]].

## Stage 1 — OLTP → Redshift

**Entry point:** `digima-mgmt/jenkins/Jenkinsfile-Digest` (runs on ASG label
`ec2-fleet-bi-andpad`). Same Jenkins node that Stage 2 uses; Stage 1 fires first each night.

**Loaders:** `digima-mgmt/jenkins/scripts/BI-New/`. The framework is declarative:

- One JSON file per Redshift target table, under `definitions/daily/`, `definitions/monthly/`,
  or `definitions/on_demand/`.
- Generic Python in `common.py` + modules (`default_rds`, `default_mongo`, `default_helper`)
  reads the JSON, pulls rows from the named `SourceDB` (Aurora/MySQL or MongoDB), applies
  per-column `Transform` functions, and upserts into Redshift using psycopg2.
- Incremental by `TimeBoundColumnNames` (usually `updated_at`). UPSERT keyed by whatever
  columns are flagged `PrimaryKey: true` in the JSON.

**Redshift target:**
- Host: `digima-production-rs-cluster-reporting.c6ohn1aaazor.ap-northeast-1.redshift.amazonaws.com`
- DB: `digima_reporting`, user: `dgm_root`.
- Password comes from the `${env}_secrets` Jenkins credential (base64 terraform tfvars
  blob, `grep rs_reporting_master_password`).

**JSON definition shape** (pulled verbatim from `inbox_envelopes.json`):

```json
{
  "TableName":       "inbox_envelopes",       // Redshift target
  "Schedule":        "daily",
  "UpdateType":      "upsert",                 // upsert | insert | overwrite
  "DataSourceType":  "rds",                    // rds | mongo | ga | s3
  "PerAccount":      true,
  "SourceDB":        "dgm_account_email",      // OLTP database
  "SourceTableName": "envelopes",              // optional, defaults to TableName
  "TimeBoundColumnNames": "updated_at",
  "ExcludeDeleted":  true,
  "JoinClause":      "envelopes inner join envelope_addresses on envelopes.id = envelope_addresses.envelope_id",
  "ExtraWhereClause":"envelope_addresses.resource_type =\"ENVELOPE_ADDRESS_RESOURCE_TYPE_CONTACT\"",
  "Columns": [ { "Name": "...", "Type": "...", "PrimaryKey": true, "SourceName": "...", "Transform": "..." }, ... ]
}
```

Per-account is the default; the loader iterates over tenants. For mongo sources it shells
out to the Mongo driver through `default_mongo`.

## Stage 2 — Redshift → S3 → Andpad

**Entry point:** `digima-mgmt/jenkins/Jenkinsfile-Andpad` (same label as Stage 1).

**Script:** `digima-mgmt/jenkins/scripts/Andpad/export_to_s3.py` (~668 lines, single file).
The exporter runs the following per table:

1. `UNLOAD` the table from Redshift with a dedup window:
   ```sql
   UNLOAD ('SELECT {cols} FROM (
             SELECT *, ROW_NUMBER() OVER (PARTITION BY {unique_cols}) AS rn
             FROM {table} {where}
             ORDER BY {order_by}
           ) WHERE rn = 1;')
   TO 's3://digima-2-production-reporting/andpad-export-tmp/{table}/{table}.'
   IAM_ROLE 'arn:aws:iam::195806501753:role/digima-production-role-reporting'
   PARALLEL OFF MAXFILESIZE 50MB JSON GZIP;
   ```
2. For `FullExport=true` tables the WHERE clause is empty; for the rest it's
   `WHERE {date_column} BETWEEN {start_of_yesterday_jst} AND {end_of_yesterday_jst}`.
   `date_column` is `updated_at` except for `access_logs` which uses `access_date`.
3. `sts:AssumeRole` → `arn:aws:iam::625407242656:role/comvex_integration` (ExternalId is
   **hardcoded** at `export_to_s3.py:462` — worth moving to the secrets bundle).
4. S3-to-S3 copy each gzipped file into
   `s3://comvex-production/production/digima/<table>/year=YYYY/month=MM/day=DD/{file}`
   (or `development/sample/digima/…` when `qa_upload=true`).
5. Append per-table log lines to `s3://digima-2-production-reporting/andpad-export-logs/<YYYY-MM-DD>.txt`.

Twenty-two tables are included (accounts, contacts_andpad, emails, sms_*, web_forms,
workflows, etc.). The list lives inline in `BiExport.Tables` — add a new entry there if a
new table needs to be exported.

## Infra wiring (`digima-infra`)

| What | Where |
|---|---|
| Redshift cluster (single-node `dc2.large`, restored from snapshot) | `modules/infra-core/redshift/redshift.tf:1-23` |
| IAM role `reporting` trusted by `redshift.amazonaws.com` | `modules/infra-core/redshift/iam.tf:6` |
| Redshift's COPY/UNLOAD policy (`s3:*` on reporting bucket) | `modules/infra-core/redshift/policies/role-policy.json` |
| S3 staging bucket `digima-2-production-reporting` | `modules/infra-core/redshift/s3.tf:1-15` |
| Bucket policy (grants cross-account Jenkins role) | `modules/infra-core/redshift/s3.tf:17-51` |
| `sg-reporting` opens 5439 to 0.0.0.0/0 | `modules/infra-core/network/main.tf` |
| VPC peering digima-production ↔ digima-mgmt | `modules/infra-core/network/main.tf` (search `digima-mgmt-peer`) |
| Jenkins BI compute on prod side (ASG) | `deployments/digima-production/main.tf:49-61` |

Corresponding on the mgmt side (`digima-mgmt/terraform/`):

| What                                        | Where                                                           |
| ------------------------------------------- | --------------------------------------------------------------- |
| Jenkins BI Andpad ASG                       | `terraform/subdeployments/asg-jenkins-node/main.tf:144`         |
| IAM module `iam_jenkins_bi_andpad` + policy | `terraform/iam.tf:167` + `policies/role-jenkins-bi-andpad.json` |
| S3 grant (mirror of prod side)              | `terraform/s3.tf:54`                                            |

## Rerunning a failed load

Jenkins is the only orchestration — if a load fails, restart the job. Both Jenkinsfiles
accept optional `from_date` and `to_date` env vars used in `yesterday_gen()`
(`export_to_s3.py:24-38`) and the loader's WHERE construction. Format:
`YYYY-MM-DD HH:MM:SS`. Use this to backfill gaps or reprocess.

Logs for Stage 2 are persisted to
`s3://digima-2-production-reporting/andpad-export-logs/<YYYY-MM-DD>.txt`.

## Notable observations

- Cluster is **publicly accessible** (`redshift.tf:10`) **and encryption at rest is disabled**
  (`redshift.tf:11`). Security team may want to revisit.
- **ExternalId hardcoded** in `export_to_s3.py:462`. Should live in the `${env}_secrets` tfvars.
- **No AWS-native scheduler.** If Jenkins is down, nothing runs. No alerting surfaced in
  the Terraform — monitoring would need to be separate.
- **Partial-export tables** filter on `updated_at`. Hard-deletes with no tombstone never
  propagate — pipeline relies on upstream soft-delete semantics (Laravel's `deleted_at`).
- **No shared primary key across systems.** When Redshift contains tables from the legacy
  monolith AND the new microservices describing the same concept (emails, contacts, inboxes),
  cross-system joins are fuzzy. See [[case-email-envelopes]] for the canonical example.

## Data sources summary

| DB | Owner | Source type |
|---|---|---|
| `dgm_account` | Legacy monolith `digima-backend-app` (PHP/Laravel) | RDS (Aurora MySQL) |
| `dgm_core` | Core / shared (accounts, industries) | RDS |
| `dgm_account_email` | `digima-backend-email-api` (Go) | RDS (MySQL) |
| `dgm_account_line` | `digima-backend-line-api` | RDS |
| `dgm_sms_account` | `digima-backend-sms-api` | RDS |
| `dgm_account_andpad` | `digima-backend-andpad-api` | Mongo |
| `dgm_lead_account` | `digima-backend-lead-api` | Mongo |
| `dgm_file_account` | `digima-backend-file-api` | Mongo |
| — | Google Analytics export | `DataSourceType: ga` (access_logs, ga_events, user_sessions) |
| — | Ad-hoc S3 loads | `DataSourceType: s3` (contact_list_contact_views only) |

See [[oltp-mapping]] for the full per-table breakdown.
