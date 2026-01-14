# Reference 
docs/ja/setup/faq.md


# Handy scripts

## If long time no start
```bash
# Generate new github token then add to .env READONLY_GITHUB_TOKEN
# Setup sidekiq_pro
bundle config --local gems.contribsys.com <sidekiq_pro_access_key>  
BUNDLE_GEMS__CONTRIBSYS__COM=<sidekiq_pro_access_key> bundle install

# Bundle install and migrate
docker compose run --rm --build web bin/bundle install
docker compose run --rm --build web bin/rails db:migrate

docker compose run --rm --build web bin/bundle update parser
```

## General script
```bash
# sidekiq_pro
bundle config --local gems.contribsys.com <sidekiq_pro_access_key>  

BUNDLE_GEMS__CONTRIBSYS__COM=<sidekiq_pro_access_key> bundle install

# github token expire, 
#   --build for apply new github token (in Dockerfile)
#   bin/bundle install install deps
docker compose run --rm --build web bin/bundle install


# setup 
docker compose run --rm web bin/setup
# migrate
docker compose run --rm web bin/rails db:migrate
# start
docker compose up web worker -d
```




# Account
- デフォルト設定以外は個別にadmin画面から設定する必要がある  
- seedで作成される初期ユーザのパスワードは一律 `password123`  
- 全プランON  
  - 会社名  
      - テスト会社  
  - 会社管理者  
      - `test1@andpad.co.jp`
  - 準会社管理者  
      - `test2@andpad.co.jp`
  - 一般  
      - `test3@andpad.co.jp`
- admin画面確認用  
  - 会社名  
      - adminテスト会社  
  - 会社管理者  
      - `system_developer@andpad.co.jp`
  - 準会社管理者  
      - `system_administrator@andpad.co.jp`
  - 一般  
      - `system_basic@andpad.co.jp`
  - 権限なし  
      - `system_none@andpad.co.jp`


# Login
```zsh
curl --location '[https://mobile.apli.andpaddev.xyz/base_app/api/v1/login](https://mobile.apli.andpaddev.xyz/base_app/api/v1/login)' \ --header 'Content-Type: application/json' \ --header 'Cookie: _andpad_jp_development_session=Z0ZEcEREVmdGNTlXZ2hwWEhkNlEzQUZTTlIwTXViK2pNcjhnS1JHQmt1OUZkREt1MkozblNrWDZ4SjQzNTYvWEdPK2dPS1VuNWpBb0ZPdXljT250aVdpUk0vOUw4ZGlaMEl2bnEyMjlxbHFEV3RhMUR4cmpqbTdobDNCN3p1Y3JQQkhEZVgxL0tBc0tqbis0TDh3c0RrbXlpQjhDelJ3OHk0MEhDb1FJd0ZNdzk5SU8vZ0xNVE42bHZXYm92RnZEV0VrNHNVRm1SNklQRmQ0aTlCZVE5T0EyaDQ2eXExdllXZDNncW5WWm5wVllaQktGQVN2d0Z3ejQ3SnVuOTJIQmxPY0FlaDB5cWpnK2dZN3hiejR5UEJ6ZUlGVmM3cW0wNDdaeEhMeVFjalE9LS1PKy9VUkt3SEY5V2kvQmhQZUN0V2dRPT0%3D--f1d29f2fd0e5b5995349f2b5bede4ea5ebcc11f4' \ --data-raw '{ "email": "phuong.le+bcomippan10@andpad.co.jp", "password": "password123" }
```

# Endpoints

http://local.andpaddev.xyz:3000/

http://local.andpaddev.xyz:3000/account/our/users