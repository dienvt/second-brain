# Idea

- Created anh cron manager for auto query job.
- We can config query to DB and start job execute that query
- Config has to have
    - ID (AI)
    - Expression
    - Formula
    - IntervalRest
    - CreatedAt
    - CreatedBy
    - UpdatedAt
    - UpdateBy
- Separate **Config** and **Job**
    - **Config** don't have status
    - **Job** have status read config for run. Persistence to DB
- CRUD for config
- Handler for handle cron job
    - list
    - refresh
    - stop

# Cron Job state

```JSON
Created => Idle <=> Stopped
						/\
						||
						\/
					Executing
```

# API