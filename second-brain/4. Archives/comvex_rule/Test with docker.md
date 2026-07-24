```
To reset test db, run: docker exec -it mongodb7 mongosh mongodb://admin:random@localhost:27017/dgm_auth_test_core\?authSource=admin --eval "db.dropDatabase();"
```

Enum 
```
type TypeName uint

const (
	TypeUnspecified TypeName = iota
	TypeOne
	TypeTwo
)

var allTypes = map[TypeName]string{
	TypeOne: "one",
	TypeTwo: "two",
}

// String returns the string representation of the TypeName.
func (t TypeName) String() string {
	s, ok := allTypes[t]
	if !ok {
		return "unspecified"
	}
	return s
}

// TypeNameFromString converts a string to a TypeName
func TypeNameFromString(s string) TypeName {
	for t, str := range allTypes {
		if str == s {
			return t
		}
	}
	return TypeUnspecified
}
```