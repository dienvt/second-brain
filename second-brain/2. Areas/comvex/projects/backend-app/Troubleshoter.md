## Config cached
remove folder `/bootstrap/cache/*` except `/bootstrap/cache/.gitignore`
run this `php artisan config:clear`

```
Memo to myself: How I fixed mutagen this time  

- delete mutagen lockfile
- stop and delete the backend app container manually
- digiform apply app
- run `mutagen sync list`
```
