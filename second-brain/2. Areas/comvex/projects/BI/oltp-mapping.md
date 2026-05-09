# Redshift ↔ OLTP mapping (full reference)

Every Redshift reporting table corresponds to exactly one JSON definition in
`digima-mgmt/jenkins/scripts/BI-New/definitions/{daily,monthly,on_demand}/`. That JSON is
the authoritative mapping — this file just collates it for quick lookup. Back to [[README]].

## Full mapping — 60 tables

Bold OLTP names = the source name differs from the Redshift name (a rename at load time).

| Redshift table | OLTP source table | Source DB | Type | Schedule | Update |
|---|---|---|---|---|---|
| access_logs | access_logs | — | ga | daily | insert |
| account_usage_statistics | account_usage_statistics | — | rds | monthly | insert |
| accounts | accounts | dgm_core | rds | daily | overwrite |
| accounts_statistics | accounts_statistics | dgm_core | rds | monthly | insert |
| activities | activities | dgm_account | rds | daily | insert |
| activity_contacts | **activities** | dgm_account | rds | daily | insert |
| billing_segments | billing_segments | dgm_account | rds | monthly | upsert |
| call_outcomes | call_outcomes | dgm_account | rds | monthly | upsert |
| connections | connections | dgm_account_andpad | mongo | daily | upsert |
| contact_customer_type_pivot | contact_customer_type_pivot | dgm_account | rds | daily | overwrite |
| contact_fields | contact_fields | dgm_account | rds | monthly | overwrite |
| contact_files | contact_files | dgm_account | rds | daily | upsert |
| contact_group_pivot | contact_group_pivot | dgm_account | rds | daily | upsert |
| contact_groups | contact_groups | dgm_account | rds | monthly | upsert |
| contact_lead_acquisition_channel_pivot | contact_lead_acquisition_channel_pivot | dgm_account | rds | daily | overwrite |
| contact_list_contact_views | contact_list_contact_views | — | s3 | on_demand | overwrite |
| contact_portal_submission_proxy | contact_portal_submission_proxy | dgm_account | rds | daily | upsert |
| contact_real_estate_construction_inquiries | contact_real_estate_construction_inquiries | dgm_account | rds | daily | upsert |
| contact_statuses | contact_statuses | dgm_account | rds | daily | upsert |
| contacts | contacts | dgm_account | rds | daily | upsert |
| contacts_andpad | **contacts** | dgm_account | rds | daily | upsert |
| customer_types | customer_types | dgm_account | rds | daily | upsert |
| email_envelope_recipients | email_envelope_recipients | dgm_account | rds | daily + monthly | insert |
| email_envelopes | email_envelopes | dgm_account | rds | daily + monthly | insert |
| email_templates | email_templates | dgm_account | rds | monthly | upsert |
| emails | emails | dgm_account | rds | daily | upsert |
| feature_subscriptions | feature_subscriptions | dgm_account | rds | monthly | upsert |
| feature_usages | **feature_usage** | dgm_account | rds | monthly | upsert |
| files | files | dgm_file_account | mongo | daily | upsert |
| ga_events | ga_events | — | ga | daily | insert |
| inbox_connections | **connections** | dgm_account_email | rds | daily | upsert |
| inbox_envelope_users | **envelope_users** | dgm_account_email | rds | monthly | overwrite |
| inbox_envelopes | **envelopes** | dgm_account_email | rds | daily | upsert |
| industries | industries | dgm_core | rds | monthly | upsert |
| lead_acquisition_channels | lead_acquisition_channels | dgm_account | rds | daily | upsert |
| lead_inbox_messages | lead_inbox_messages | dgm_account | rds | daily | upsert |
| line_friend_messages | **contact_messages** | dgm_account_line | rds | monthly | upsert |
| line_messages | **contact_messages** | dgm_account_line | rds | daily | upsert |
| phone_outgoing_calls | phone_outgoing_calls | dgm_account | rds | monthly | upsert |
| portals | portals | dgm_lead_account | mongo | daily | upsert |
| reminders | reminders | dgm_account | rds | monthly | upsert |
| reports | reports | dgm_account_andpad | mongo | daily | upsert |
| sms_clicks | **v2_clicks** | dgm_sms_account | rds | daily | upsert |
| sms_contact_messages | **v2_contact_messages** | dgm_sms_account | rds | daily | upsert |
| sms_contact_message_replies | **v2_contact_messages** | dgm_sms_account | rds | daily | upsert |
| sms_events | **v2_events** | dgm_sms_account | rds | daily | upsert |
| sms_messages | **v2_messages** | dgm_sms_account | rds | daily | upsert |
| sub_industries | **industry_classifications** | dgm_core | rds | monthly | overwrite |
| sub_industries_pivot | **account_industry_classification_pivot** | dgm_core | rds | monthly | overwrite |
| user_sessions | user_sessions | — | ga | daily | insert |
| users | users | dgm_account | rds | daily | overwrite |
| web_form_submissions | web_form_submissions | dgm_account | rds | daily | upsert |
| web_forms | web_forms | dgm_account | rds | daily | upsert |
| web_tracking_pages | web_tracking_pages | dgm_account | rds | daily | upsert |
| web_tracking_visits | web_tracking_visits | dgm_account | rds | daily | upsert |
| workflow_action_participant_pivot | workflow_action_participant_pivot | dgm_account | rds | monthly | upsert |
| workflow_actions | workflow_actions | dgm_account | rds | daily | upsert |
| workflow_participants | workflow_participants | dgm_account | rds | monthly | upsert |
| workflows | workflows | dgm_account | rds | daily | upsert |

## Non-trivial extracts (JoinClause / ExtraWhereClause)

Most tables are straight SELECTs. These are the ones that apply a JOIN or pre-filter, so
the Redshift row count will not equal the OLTP row count even at steady state:

- **`inbox_envelopes`** — `envelopes INNER JOIN envelope_addresses ON envelopes.id = envelope_addresses.envelope_id` **WHERE `envelope_addresses.resource_type ="ENVELOPE_ADDRESS_RESOURCE_TYPE_CONTACT"`**. Rows tied to internal users (`…_USER`) are dropped. Relevant to [[case-email-envelopes]].
- **`contacts_andpad`** — same OLTP table as `contacts`, but the JSON transforms most columns into booleans (`is this field populated?`) instead of exporting the raw PII values. This is a privacy-aware view intended for the Andpad export.
- **`line_friend_messages` / `line_messages`** — both source from `dgm_account_line.contact_messages`; they differ in filter and schedule (monthly/daily).
- **`sms_contact_messages` / `sms_contact_message_replies`** — both source from `dgm_sms_account.v2_contact_messages`.

## OLTP DB → owning microservice (cheat-sheet)

| Source DB | Owner | Notes |
|---|---|---|
| `dgm_account` | Legacy monolith `digima-backend-app` (PHP/Laravel) | Most tables live here |
| `dgm_core` | Core/shared data | Accounts, industries, etc. |
| `dgm_account_email` | `digima-backend-email-api` (Go) | `envelopes`, `envelope_addresses`, `connections`, `envelope_users` |
| `dgm_account_line` | `digima-backend-line-api` | LINE messaging |
| `dgm_account_andpad` | `digima-backend-andpad-api` | MongoDB |
| `dgm_sms_account` | `digima-backend-sms-api` | Note the `v2_` prefix on every source table |
| `dgm_lead_account` | `digima-backend-lead-api` | MongoDB (portals) |
| `dgm_file_account` | `digima-backend-file-api` | MongoDB (files) |

## How to look up one table

```bash
cd digima-mgmt/jenkins/scripts/BI-New/definitions

# Redshift table → full JSON definition
find . -name "<redshift_table>.json" -exec jq . {} \;

# Or search by OLTP source table name
grep -l '"SourceTableName": "<oltp_table>"' -r .
```

To regenerate this mapping table from the JSON files (e.g. after new definitions land):

```bash
cd digima-mgmt/jenkins/scripts/BI-New/definitions
for f in $(find . -type f -name "*.json" | sort); do
  jq -r '[(.TableName // "?"),
          (.SourceTableName // .TableName // "?"),
          (.SourceDB // "-"),
          (.DataSourceType // "-"),
          (.Schedule // "-"),
          (.UpdateType // "-")] | @csv' "$f"
done | sort -u | column -t -s, | sed 's/"//g'
```

## Where these mappings come from in the loader

The JSON definitions are read by `digima-mgmt/jenkins/scripts/BI-New/common.py`, which
resolves `SourceDB` to a connection (Aurora vs Mongo), builds the SELECT or find() query
using `JoinClause` + `ExtraWhereClause` + `TimeBoundColumnNames` + the `Columns` array,
applies per-column `Transform` code, and writes to Redshift via the update strategy (`insert`,
`upsert`, `overwrite`).
