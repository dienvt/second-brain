Define first 
# Reference
https://www.howtographql.com/basics/2-core-concepts/
https://gqlgen.com/getting-started/

# Concept
## Schema Definition Language (SDL)

`Schema` is the definition of your data which will be transfer, query, create in GrapQL application layer
`Chema` with the prefix keyword will define your API, like `query` or `mutation`

[for more types](https://graphql.org/learn/schema/#the-query-and-mutation-types)
type define 
```
type Post {
  title: String!
  author: Person!
  publisher: String
}
```

Basic query
```
{
  allPersons (conditions) {
  }
}
```

Write data
```
mutation {
  createPerson(name: "Bob", age: 36) {
	id
    name
    age
  }
}
```

Subscriptions for catch data change
```
subscription {
  newPerson {
    name
    age
  }
}
```


input
```
input NewTodo {  
  text: String!  
  userId: String!  
}
type Mutation {  
  createTodo(input: NewTodo!): Todo!  
}
```



## Operations (Query, Mutation and Subscription)
https://graphql.org/learn/schema/#the-query-and-mutation-types

## Resolver

**Each field in a GraphQL schema is backed by a resolver**
So a `resolver` is a algorithm to get the schema or data

```
Query.author(root, { id: 'abc' }, context) -> author
Author.posts(author, null, context) -> posts
for each post in posts
  Post.title(post, null, context) -> title
  Post.content(post, null, context) -> content
```


## How to query
First need define query in SDL
ex:
```
type Query {  
  todos: [Todo!]!  
  todosByUser (userID: ID): [Todo!]!  
}

```

then Create query like elasticSearch dev tool
```
query findTodosByUser{
  todosByUser(userID:"6"){
    id
    text
    user{
      id
    }
  }
}
```


# Architecture
GraphQL is like an BFF server which is expose an friendly interface to client.


# Question 
1. Tại sao phải refresh (Khi add user, call, ....) trong khi graphQL có subscription?
2.


start BFF

```bash
get-graphql-schema http://localhost:8080/graphql -h X-PLATFORM='WEB' -h X-APP-VERSION="1.0.0" -h Authorization="Bearer KUsqm0h5qOJkciNgKZ1hkYStwQ4q2sN-eo8RMTUsigYy7cTQMngEDl-bRIH2VqDTA60"> graph/exported.graphql
```

