# Index
## create new index
```
PUT /stg.bill.storage.relations
{
  "settings": {
    "index": {
      "number_of_shards": 3,  
      "number_of_replicas": 2 
    }
  }
}
```

## Create index with mapping
```
PUT my-index-000001
{
  "mappings": {
    "properties": {
      "number_of_bytes": {
        "type": "integer"
      },
      "time_in_seconds": {
        "type": "float"
      },
      "price": {
        "type": "scaled_float",
        "scaling_factor": 100
      }
    }
  }
}
```

# Mapping
## List all mapping
```
GET sb.billtrack.customer_auto_bill/_mapping
{}
```

## Add mapping
```
PUT /stg.bill.storage.relations/_mapping
{
  "properties": {
    "email": {
      "type": "keyword"
    }
  }
}

PUT billtrack.customer_auto_bill/_mapping
{
  "properties": {
    "suggest_day": {
      "type": "long"
    },
    "score": {
      "type": "float"
    }
  }
}
```


()