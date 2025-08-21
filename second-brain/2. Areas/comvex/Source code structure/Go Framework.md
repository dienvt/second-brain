forms: input

transformers: output

We will have many Provider.

Provider like a container and each container contain many more containers.

The registry will require an name, may name duplicate???? why should we use other key like struct????


controller

interface > package name > resource > v1 > controller
domain > services > resource > services

## Todo
Should investigate test suit

why it don't support parallel

should we replace it? should we wrap it with plant go test

  

# database 

infas -> database -> migration

  

our stature 

Multitatent

none-multi-tenant -> call them core 

  

# migrations 

- folder will have core dir is none multitatnet 

with monogdi migrate is create index

learn about index in mongodb how to effective.

  

# Seeder-> test 

- Using for mock data in database

- store in database add in the framework

- Have Run function that create 

- Run will be call in Command seek conmad. Just need to register to the commanServiceProvider.

- In go-frame work Loop all factory and call seek command. 

- So that how

  

# Factory -> development purpose, in domain layer

- Go facker using create dummy data

- reflect the model

- With Make function return model instance

  

# Domain

  

  

# Repository:

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

  

  

Next -> create grpc endpoint.