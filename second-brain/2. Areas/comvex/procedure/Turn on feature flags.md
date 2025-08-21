# Note
stg account_id: 532
# Check Feature flag existances

## Create 
```
digima:feature-flag-create USERWEB_ACTIVITY  \\\"Userweb Activity\\\"
```

# Turn On
You need to go to "Staging Command Execution": https://jenkins.mgmt.digima.com/job/Digima%20Staging%20Command%20Execution/
Then "Run with parameters" this command:
```
digima:account-feature-flag-add 1 FEATURE_FLAG_CODE
```

where:  
`1` : is the account id
`FEATURE_FLAG_CODE`  is the feature flag you want to attach to an account. You can check the backend-app [Models/Core/FeatureFlag.php](https://github.com/comvex-jp/digima-backend-app/blob/develop/app/Models/Core/FeatureFlag.php) file for the list of feature flags.

# Turn Off

If you want to remove the feature flag, it's almost the same command, just replace "add" by "remove":
```
digima:account-feature-flag-remove 1 FEATURE_FLAG_CODE
```

