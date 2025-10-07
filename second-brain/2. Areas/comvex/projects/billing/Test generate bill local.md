Create Segment. I user db:seed then just keep 1 record
```
db.getSiblingDB("dgm_core_billing").getCollection("account_pricing_plans").find({},{_id:1}).limit(10);


-- example
db.getSiblingDB("dgm_core_billing").getCollection("account_pricing_plans").find({},{id:1}).limit(10);
[ { _id: ObjectId('68de394669077ed3e07ea7c7') } ]
```

Then I will truncate `segments` and `invoices` collection
```
db.getSiblingDB("dgm_core_billing").getCollection("segments").deleteMany({});


test> db.getSiblingDB("dgm_core_billing").getCollection("segments").find({},{id:1}).limit(10);

test> db.getSiblingDB("dgm_core_billing").getCollection("invoices").find({},{id:1}).limit(10);

```

Then clear all redis key
`redis-cli --scan --pattern "counter*" | xargs redis-cli DEL`

Then execute create segment all
`docker exec billing ./tmp/main segments:create all all`

There are 24 segments creation jobs will be pass to rabbitmq
