## Remove auth users
```
php artisan digima-tmp:auth-api-users-migrate Qwerty1!


Then have to start worker
php artisan queue:work queue-1 --tries=1 --timeout=300 -v
```

# Remove account
```
php artisan digima-tmp:auth-api-user-account-code-update
```

# Reinit
```
git config --global --add safe.directory /var/www/digima/vendor/comvex-jp/digima-backend-billing-proto
```


```
## create admin group
docker exec auth-api ./tmp/main group:create 'Admin group' \
--policies='[{"description":"Allow to authorize Admin Web App","statements":[{"effect":"allow","action":["authorize:create"],"resource":"drn:client:"}]}]'

## update callback
docker exec auth-api ./tmp/main client:update 69b10ae97630299db3ed9e2f --redirect-uris=https://app.adminweb.digima.local:8083/auth/callback


## Update admin user

```