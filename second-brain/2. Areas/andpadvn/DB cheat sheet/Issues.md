# Missing platform
Update docker-compose
```
database:  
  image: mysql:8.0.28  
  platform: linux/arm64/v8
```

```
SET global general_log = 'ON';
SHOW VARIABLES LIKE 'general_log_file';

```