# Location
go/services/remote/database/schemas


```
t.datetime "shot_at", limit: 3,null: true
```

mean create column name shot_at, datatype is `datetime` and `limit: 3` mean it will get to millisec, if not database will only get to second

Add `t.string "metadata", null: true` into `go/services/remote/database/schemas/session_users.schema`

## Using AWS CodeBuild to migrate data

Need create a PR for apply change about DB migration, waiting for approval and merge to main

Access AWS and change rejoin to `ap-northeast-1(Tokyo)`

run `andpad-remote-develop-dryrun-db-migrate`
Check the result
