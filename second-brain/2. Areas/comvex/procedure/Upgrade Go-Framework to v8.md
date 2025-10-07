Replace the repository
```
t\.DatabaseManager\.GetClient\(database\.\w+\)\.GetRepository\(repositories\.\w+\)\.\(\*(\w+)\.(\w+)\)
```
to 
```
new($1.$2)
```