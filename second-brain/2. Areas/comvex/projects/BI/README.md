# BI / Redshift pipeline at Comvex

Reporting, analytics, and the nightly Andpad data export all run through one pipeline: a
Jenkins-orchestrated Python ETL that pulls from the OLTP databases of the various
microservices, UPSERTs into a Redshift reporting cluster, and (for Andpad) UNLOADs back to a
cross-account S3 bucket.

There is **no** AWS-native ingestion (no DMS, no Kinesis Firehose, no Glue). Everything is
code in `digima-mgmt/jenkins/scripts/BI-New/` running on dedicated Jenkins EC2 nodes over
VPC peering.

## Architecture at a glance

```
 digima-mgmt (account 195806501753)        digima-production (OLTP + Redshift)
 ────────────────────────────────────      ────────────────────────────────────────────
 Jenkins ASG ec2-fleet-bi-andpad            RDS Aurora   MongoDB   (various per service)
   │                                             │          │
   │    Jenkinsfile-Digest ──Stage 1 read────────┴──────────┘
   │                         (SQL / Mongo query over VPC peering)
   │                      ──Stage 1 upsert──▶ Redshift digima_reporting
   │                                             (rs-cluster-reporting.*.redshift.amazonaws.com)
   │
   │    Jenkinsfile-Andpad ──Stage 2 UNLOAD──▶ s3://digima-2-production-reporting/…
   │                                                │
   │                         ──sts:AssumeRole──▶ arn:aws:iam::625407242656:role/comvex_integration
   │                         ──S3 cross-copy──▶ s3://comvex-production/production/digima/…
```

## Repos involved

| Repo | Role |
|---|---|
| `digima-mgmt` | Jenkins pipelines + Python loaders (`jenkins/scripts/BI-New`, `jenkins/scripts/Andpad`) |
| `digima-infra` | Terraform for the Redshift cluster, IAM `reporting` role, S3 bucket, VPC peering |
| `digima-backend-app` | Legacy monolith (PHP/Laravel), owns `dgm_account` — most ingested tables |
| `digima-backend-email-api` | Go microservice, owns `dgm_account_email` — envelope/inbox tables |
| `digima-backend-line-api` | Owns `dgm_account_line` |
| `digima-backend-sms-api` | Owns `dgm_sms_account` |
| `digima-backend-andpad-api` | Owns `dgm_account_andpad` (Mongo) |
| `digima-backend-file-api` | Owns `dgm_file_account` (Mongo) |
| `digima-backend-lead-api` | Owns `dgm_lead_account` (Mongo) |

## Contents of this folder

- [[redshift-pipeline]] — How data actually moves: Stage 1 ingest, Stage 2 Andpad export,
  infra wiring, notable observations.
- [[oltp-mapping]] — Reference table for all 60 Redshift tables → their OLTP source. The
  renames and joins that aren't obvious from the name.
- [[case-email-envelopes]] — Worked case: why `email_envelope_recipients` rows often have
  no matching `inbox_envelopes` row, and how to tell a real bug from expected divergence.

## When to open which file

- "Where does this Redshift column come from?" → [[oltp-mapping]].
- "Why is there / isn't there a row here?" → [[case-email-envelopes]] for email; the same
  reasoning applies to other dual-write legacy/new-service pairs.
- "How do I rerun a failed BI load?" / "Who owns this infra?" → [[redshift-pipeline]].
