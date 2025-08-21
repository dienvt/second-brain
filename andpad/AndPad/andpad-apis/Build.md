```bash
aws ecr get-login-password --region ap-northeast-1 | docker login --username AWS --password-stdin 969998347131.dkr.ecr.ap-northeast-1.amazonaws.com
docker compose run --rm builder
```
