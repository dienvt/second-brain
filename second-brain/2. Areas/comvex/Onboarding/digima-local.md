---
tags:
  - comvex
---
* [x] Github dual account ✅ 2025-06-16
* [ ] Presentation about Hoppies, my type 
* [ ] Do not have datadog
* [x] Postman logged in ✅ 2025-06-16
* [x] Vault sharing credential ✅ 2025-06-16
* [x] Tailscale to connect VPN ✅ 2025-06-16
* [ ] Setup calendar


- [x] Python 3 (Last tested with 3.13.3) ✅ 2025-06-16
- [x] Docker (Last tested with 28.1.1 with `General > Use containerd for pulling and storing images` unchecked) ✅ 2025-06-16
- [x] Terraform (Last tested with 1.11.3) ✅ 2025-06-16
- [x] Packer (Last tested with 1.12.0 and Docker plugin 1.1.1 and Ansible plugin 1.1.2) ✅ 2025-06-16
	- `brew install hashicorp/tap/packer`
- [x] Ansible (Last tested with 2.18.5) ✅ 2025-06-16 
	- `brew install ansible`
	- `packer plugins install github.com/hashicorp/ansible`
- [x] Mutagen (Last tested with 0.18.1) ✅ 2025-06-16
	- `brew install mutagen-io/mutagen/mutagen`
- [x] AWS CLI (Last tested with 2.22.28) ✅ 2025-06-16


# notes
There are two scrip `digibuild` and `digiform`
`digibuild` using for build docker file for PHP. In production, we don't use docker and ECR

account : database/seeders/account/dev/AccountUserSeeder.php
https://github.com/comvex-jp/digima-backend-app/blob/develop/database/seeders/account/dev/AccountUserSeeder.php


## Install vault
`brew install hashicorp/tap/vault`

## Stuck when execute
Auth-API
1. Run the Migrate Command `db:migrate all` to create the database schema.
2. Run the Bootstrap Command `db:bootstrap` to create the initial data.

````
$ git config --global url."git@github-comvexcojp:comvex-jp/backend-service-go-framework".insteadOf "https://github.com/comvex-jp/backend-service-go-framework"
````

Cachs run bash

### Start Digima

```bash
Start those services
digiform apply -y app
digiform apply -y auth
digiform apply -y userweb
cd digima-webapp && yarn start
```

* [ ] Migrate digima backend app
```bash
php artisan digima:db-core-migrate [[--force] [--pretend]]

php artisan digima:db-core-seed [[--force] [--dev]]

php artisan digima:db-account-migrate 1 [[--force] [--pretend]]

php artisan digima:db-account-seed 1 [[--force] [--dev]]

php artisan digima:db-account-create 2 [[--force] [--pretend]]

php artisan digima:db-account-migrate 2 [[--force] [--pretend]]
```
* [ ] 

## Common Errors

### Can not start web app
- Error: Port in use even thought there is no running program on that pod
	- Solution: Check the hosts
