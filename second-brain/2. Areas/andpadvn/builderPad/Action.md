
## Setup
```shell
docker compose run --rm --no-deps web bundle install


```


// migrate
`docker compose run --rm web bin/rails db:environment:set RAILS_ENV=test

// regenerate routes.rb
`docker compose run --rm web bin/rails db:migrate`

// format 
rubocop -a spec/requests/api/internal/remote/clients/users/remote_enable_spec.rb

// run spec


// debug run
