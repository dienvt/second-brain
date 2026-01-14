## Check sync
```
mutagen sync list
<<<<<<< HEAD
=======

>>>>>>> 41ec96b (vault backup: 2026-01-14 09:58:32)
```
## Test
`php artisan test`

## Test specifice package
```
php artisan test tests/Integration/Listeners/Lead/Inbox/Message/ProduceLeadEmailMessageConvertedEventSubscriberTest.php

```

## Install package
```
composer require comvex-jp/digima-backend-app-proto:1.4.0
composer update comvex-jp/digima-backend-app-proto:1.4.0
composer install
```

## Turn on feature flag
```
php artisan digima:feature-flag-create USERWEB_ACTIVITY  \\\"Userweb Activity\\\"

php artisan digima:account-feature-flag-add 1 FEATURE_FLAG_CODE
```
## Seeder
```bash
php artisan db:seed --class=Database\\Seeders\\DevelopmentAccountSeeder


# IDN
php artisan db:create
```

## Error Don't found Class
```
composer dump-autoload
```

```bash
docker exec -it app-api composer update comvex-jp/digima-backend-app-proto

docker exec -it app-api git config --global --add safe.directory /var/www/digima/vendor/comvex-jp/digima-backend-app-proto

docker exec -it app-api composer dump-autoload

```

## Run PHP function on demand
```
> php artisan tinker

-- then run 
> use Digima\Services\UidGenerator;
> UidGenerator::make(12)
```