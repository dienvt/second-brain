## Check sync
```
mutagen sync list
```

## Test
`php artisan test`

## Test specifice package
```
php artisan test tests/Integration/Listeners/Lead/Inbox/Message/ProduceLeadEmailMessageConvertedEventSubscriberTest.php
```

## Install package
### Install package
```
composer require comvex-jp/digima-backend-app-proto:1.4.0
composer update comvex-jp/digima-backend-app-proto:1.4.0
composer install
```
### Install dev package
* change the `composoer.json` with `dev-` prefix to "comvex-jp/digima-backend-billing-proto": "dev-feat/account-pricing-plan-workflow-name"
```
!composer.json
"comvex-jp/digima-backend-billing-proto": "dev-feat/account-pricing-plan-workflow-name",


composor update
composor install
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

## List router
```
php artisan route:list
```

## Debug log
ravel, you can write debug logs using:

**1. Log facade:**

\Log::debug('Debug message', ['context' => $variable]);

**2. logger() helper:**

logger()->debug('Debug message', ['context' => $variable]);
- 
- 
- 
- 

**3. Other log levels:**
```
<?php
\Log::info('Info message');
\Log::warning('Warning message');
\Log::error('Error message', ['exception' => $e]);
```
- 
- 
- 
- 

**Example in your current file:**
```
<?php
public static function findByFeatureAt(string $feature, Carbon $date): ?self
{
    [$from, $to] = get_month_bounds($date, 'UTC');
    
    \Log::debug('Finding usage by feature', [
        'feature' => $feature,
        'from' => $from,
        'to' => $to,
    ]);
    
    return self::query()
               ->where('feature', $feature)
               ->where('from', $from)
               ->where('to', $to)
               ->first();
}
```
- 
- 
- 
- 

Logs are written to `storage/logs/laravel.log` by default. The log channel configuration is in