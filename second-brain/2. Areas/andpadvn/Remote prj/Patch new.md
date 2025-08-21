
## Create path branch
Create branch format: `patch/{{env}}-{{service}}-xxx-YYYYMMDD`
env: `production` or `develop`
service: remote
What change: 

```diff
+++ b/go/services/remote/cmd/patch/patch_migrate_client_role_permission_session_update/main.go
-               // For testing before execute the patch, we return error to rollback tx
-               // For running patch, create a new branch and remove the line below
-               return errors.New("rollback for testing purpose")
+               return nil

```

#### Step one: create a release tag
Access: [git release](https://github.com/88labs/andpad-vanguard-backend/releases)
Hit "Draft a new release"
Create new tag with formats:
* Development: `patch-develop-{{service}}-xxx-YYYYMMDD`
* Staging: `patch-staging-{{service}}-xxx-YYYYMMDD`
* Production: `patch-production-{{service}}-xxx-YYYYMMDD`

Create as latest release

## Monitoring
Search: `env:production service:andpad-remote-patch`
or via [link](https://app.datadoghq.com/logs?query=env%3Aproduction%20service%3Aandpad-remote-patch&agg_m=count&agg_m_source=base&agg_t=count&cols=host%2Cservice&fromUser=true&messageDisplay=inline&refresh_mode=sliding&storage=hot&stream_sort=desc&viz=stream&from_ts=1745291039296&to_ts=1745291939296&live=true)