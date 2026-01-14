```
// migrate db 
docker exec app-api php artisan digima:db-core-migrate


// execute
# Run the migration for all accounts (with confirmation in production)
php artisan digima:billing-properties-migrate all

# Dry run to preview changes
php artisan digima:billing-properties-migrate all --dry-run

# Migrate specific accounts
php artisan digima:billing-properties-migrate 1,2,3

# Migrate a range of accounts
php artisan digima:billing-properties-migrate 1-100

# Force re-migration of already migrated accounts
php artisan digima:billing-properties-migrate all --force

# Use smaller batch size
php artisan digima:billing-properties-migrate all --batch=50
```