Email and SMS mismatch guarentee
### Email


## LINE message
```
  What the daily BI does

  The "daily BI" is a Jenkins-scheduled Python pipeline (jenkins/scripts/BI-New/) that copies operational data from production databases into the Redshift reporting warehouse (digima_reporting), once per day, table-by-table. Each table is driven by a JSON definition file in definitions/daily/.

  For line_messages, the flow (bi.py → modules/line_messages.py → common.py) is:

  1. Wait if the source DB is hot — db_alarm_check polls the digima-production-alarm-cpu-line CloudWatch alarm and sleeps 60s in a loop while it's in ALARM (common.py:297).
  2. Per-account loop — PerAccount: true, so it iterates every account and queries that account's DB dgm_account_line_{account_id}.
  3. Time-bounded SELECT from contact_messages (default_rds.py:42-48). The WHERE clause built from TimeBoundColumnNames: "updated_at,deleted_at" is:
  where ((contact_messages.updated_at >= {start} and <= {end})
      or (contact_messages.deleted_at >= {start} and <= {end}))
  4. where {start}/{end} = yesterday 00:00–23:59 JST (yesterday_gen, common.py:251).
  5. Group-message compression (line_messages.py:20-53) — rows sharing a non-null group_message_recipient_id are collapsed into one row; surviving row keeps the first id and aggregates the others' content into a JSON array.
  6. Write JSON-lines → upload to S3 s3://digima-2-production-reporting/tmp/line_messages/line_messages_rs.json.
  7. Load into Redshift (update_rs, common.py:506): COPY into line_messages_tmp, then because UpdateType: upsert, run INSERT of new PKs + MERGE ... REMOVE DUPLICATES on PK (account_id, id), then drop the temp table.

  (A separate downstream job, Andpad/export_to_s3.py, later UNLOADs Redshift → Andpad's S3. That's a different stage.)

  Why records in contact_messages don't appear in Redshift

  There is no code term "mismatch" in this repo — the mismatch you're seeing is operational: the row exists in source contact_messages but not in Redshift line_messages. The pipeline design has several ways this happens:

  8. It's an incremental upsert with no backfill. A row is only picked up if its updated_at or deleted_at lands inside yesterday's window. If the daily job for the day a record changed failed, was blocked, or errored — and that record is never touched again — it is permanently missing. There is no
  full-refresh/reconciliation pass for line_messages (it's upsert, not overwrite).

  9. The column-order bug you just fixed (commits b48efa47 → c2cfe7ff). This is the most likely culprit for a batch of missing records. The COPY uses format as json 'auto' (maps by name), but the subsequent insert into line_messages select line_messages_tmp.* and the MERGE are positional (common.py:480). The temp
  table is created from the JSON definition's column order; the live root table got group_message_recipient_id appended at the end via ALTER TABLE. While the definition listed that column out of order, the positional SELECT * shifted values into the wrong columns → type error → the whole COPY+merge transaction for
  that account threw and rolled back (common.py:570), so nothing for that account synced that day. c2cfe7ff (move column to end) realigned the definition with the physical Redshift schema. Records missed on the broken days won't come back on their own.

  10. Group-message compression hides individual IDs. If you're checking Redshift by an individual message id, members of a group were merged into a single row keyed by one id; the others legitimately don't exist as separate rows.

  11. Cross-day group divergence. Compression only operates on one day's result set. If two messages of the same group change on different days, day 1's row and day 2's row aren't merged together — producing inconsistent grouping vs. source.

  12. occurred-only edge. occurred_at is not a time-bound column. A message whose updated_at/deleted_at is old won't enter any recent window even if it's recent by occurred_at.

  How to recover the missing records

  The pipeline supports a manual date-range re-run via env vars (yesterday_gen → env_date_gen, common.py:254):

  from_date / to_date  →  re-runs the line_messages BI job over an arbitrary window

  So the fix for already-missing rows is to re-run the line_messages daily job with from_date/to_date covering the period the records were created/updated (especially the days the column-order bug was live). The upsert will backfill them.

  ---
  Want me to (a) confirm the exact env_date_gen window format and draft the re-run parameters, or (b) trace the git history to pin down the exact date range when the column-order mismatch was crashing the sync?

```