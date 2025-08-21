https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update.html#upserts
```
POST test/_update/1
{
  "script": {
    "source": "ctx._source.counter += params.count",
    "lang": "painless",
    "params": {
      "count": 4
    }
  },
  "upsert": {
    "counter": 1
  }
}
```



```
@startuml
node Disbursement_Core
node node2
Disbursement_Core -- node2
@enduml
```