# Document

[https://confluence.zalopay.vn/x/eAIFAQ](https://confluence.zalopay.vn/x/eAIFAQ)

[https://confluence.zalopay.vn/x/0_8JAw](https://confluence.zalopay.vn/x/0_8JAw)

  

# Direct supplier plan

[[Direct supplier]]

# V2

Requirement

#### Requirement

|Name|Created|Tags|
|---|---|---|
|[[config theo app, supplier]]|July 8, 2021 9:50 AM||
|[[Page 2]]|July 8, 2021 9:50 AM||
|[[Page 3]]|July 8, 2021 9:50 AM||

  
  

  

|Name|Assign|Status|
|---|---|---|
|[[Handle log]]|||
|[[Card 2]]|||
|[[VNG Works/Work Note/Bill tracking/Untitled Database/Card 3\|Card 3]]|||

  
  

# Elastic search query

```SQL
GET /billtrack.customer_auto_bill/_count
{
  "query": {
    "match_all": {}
  }
}

GET /billtrack.customer_auto_bill/_count
{
  "query": {
    "match": {
      "app_id": "21"
    }
  }
}

GET /billtrack.customer_auto_bill/_count
{
  "query": {
    "match": {
      "app_id": "20"
    }
  }
}

GET /billtrack.customer_auto_bill/_count
{
  "query": {
    "match": {
      "app_id": "17"
    }
  }
}

GET /billtrack.customer_auto_bill/_count
{
  "query": {
    "bool": {
      "must": [
          {"match":{"app_id": "17"}},
          {"match":{"enable_query": 0}}
      ]
    }
  }
}

GET /billtrack.customer_auto_bill/_count
{
  "query": {
    "bool": {
      "must": [
        {"match":{"app_id": "17"}},
        {"match":{"provider": "PAYOO"}}
      ]
    }
  }
}


GET /billtrack.customer_auto_bill/_count
{
  "query": {
    "bool": {
      "should": [
        {"match": {"app_id": "17"}},
        {"range": {
            "last_query_time": {
              "gte": "2021-06-01T00:00:00+07:00",
              "lte": "2021-07-01T00:00:00+07:00"
            }
          }
        }
      ]
    }
  }
}


GET /billtrack.customer_auto_bill/_count
{
  "query": {
    "bool": {
      "must": [
        {
          "bool": {
            "should": [
              {
                "range": {
                  "last_query_time": {
                    "gte": "2021-06-01T00:00:00+07:00",
                    "lte": "2021-07-01T00:00:00+07:00"
                  }
                }
              }
            ]
          }
          
        }
      ]
    }
  }
}
```

[[Notify]]