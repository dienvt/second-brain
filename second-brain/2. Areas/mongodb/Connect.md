mongosh mongodb://localhost:27017

```
mongosh --username <username> --password <password> --host <hostname> --port <port> --authenticationDatabase <auth_db>    
```

`docker exec -it mongodb7 mongosh --username admin --password random --host localhost --port 27017`
or 
`docker exec -it mongodb7 mongosh mongodb://admin:random@localhost:27017`

Once connected, you can execute MongoDB shell commands, such as:

- `show dbs`: List available databases.
- `use <database_name>`: Switch to a specific database.
- `show collections`: List collections within the current database.
- `db.<collection_name>.find()`: Query documents in a collection.