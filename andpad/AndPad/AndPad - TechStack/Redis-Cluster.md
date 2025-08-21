# References
https://88-oct.atlassian.net/wiki/spaces/OCT/pages/2819885627/Users+Guide+of+Shared+Redis+Cluster+by+microservices+Incubator
https://docs.google.com/spreadsheets/d/1m6lYkEt4o7NrOOEq2jInzeGTVz42RmqsDqUS5XJu-yc/edit?gid=0#gid=0&range=B10

# Note

Select the Redis logical database having the specified zero-based numeric index. New connections always use the database 0.

# Config

| Env   | Adress                                                                                                                                                                         | db  | prefix   | user/pass                                   |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --- | -------- | ------------------------------------------- |
| `dev` | [master.andpad-incubator-development-cluster.jiodym.apne1.cache.amazonaws.com:6379](http://master.andpad-incubator-development-cluster.jiodym.apne1.cache.amazonaws.com:6379/) | `0` | `remote` | `andpad-remote-dev-user`/`s6m9k0aYG0PgzTTa` |
| `stg` |                                                                                                                                                                                | `0` | `remote` | `andpad-remote-stg-user`/`1q379POuudqf53qG` |
|       |                                                                                                                                                                                |     |          |                                             |
