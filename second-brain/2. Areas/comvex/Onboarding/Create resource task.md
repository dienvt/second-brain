---
tags:
  - comvex
  - comvex-onboarding
---
  
# Database  migration
Dir: infas -> database -> migrations
Comvex database type: 
+ Multitatent : emails, contacts, ...
+ `core` a.k.a none-multi-tenant: Not depend to client, only using for comvex
folder will have core dir is none multitatnet 

* with mongoDB migrate is create index

learn about index in mongodb how to effective.

# Domain model
* Dir: domain > models
- Just create model
  

# Seeder-> test 
Dir: 

- Using for mock data in database

- store in database add in the framework

- Have Run function that create 

- Run will be call in Command seek conmad. Just need to register to the commanServiceProvider.

- In go-frame work Loop all factory and call seek command. 

- So that how

  

# Factory -> development purpose, in domain layer
Dir: Domain > factories 
-> Move to near model

- Go facker using create dummy data
- For enum, using boundary_start and boundary_end
```go
Feature   feature_usage.Feature `faker:"boundary_start=1, boundary_end=15"`
```
- reflect the model
- With Make function return model instance

# Domain
Domain > repositories
* Define repository interface only.
* 
  

  

# Repository:
Dir: infra > database > repository


- stay in infra, database, repository, 

- define connection account

- same ad

- None-Multi-tennant we have to define in core

- Pass the domain model to 

- Data in repository is call entity, using bson for monogo db

- base is common helper, abstract the conn transfer from entity to our model.

- 

  

# test 

- Unit test 

- only test in repository 

- other nor the repository declare the struct not the logic ??? is this true because I got some error when parsing data

- Since we use mongo sb, we using mock to test

  

# feature test

- e2e test, don't using mock.