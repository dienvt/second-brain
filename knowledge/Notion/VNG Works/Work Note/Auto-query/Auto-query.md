[https://confluence.zalopay.vn/display/PD/Bill+Auto+query+logic+V2+optimize+-+Update+add+more+AppID](https://confluence.zalopay.vn/display/PD/Bill+Auto+query+logic+V2+optimize+-+Update+add+more+AppID)

# Issues

[[10-09-2021 - 21-09-2021]]

# Git repo

git: [https://gitlab.zalopay.vn/telco/billtracking/auto-query](https://gitlab.zalopay.vn/telco/billtracking/auto-query)

trigger build:

build stg billing

→ last-update

# Production cfg

- Last paid (RPU)
    
    ```JSON
    "cron": "0 18 * * *",
    "app_supplier": {
    "17": [2,4,5],
    "18": [100,101,111],
    "20": [201]
    },
    "month_offset": 3
    ```
    
- Last update
    
    ```JSON
    "cron":"0 16 * * *",
    "app_supplier": {
    	"17": [1,2,3,4,5],
    	"18": [100,101,111],
    	"20": [201]
    },
    "month_offset": 3,
    
    ```
    

---

# SB docker cmd

- Run auto-query
    
    ```Bash
    # have port
    docker run -d --name=auto-query \
    -v /home/hungnv4/auto-query:/app \
    -p 8088:8080 \
    --restart=always \
    acq-go:0.1 ./srv service
    ```
    
- Run last-update
    
    ```Bash
    # not port
    docker run -d --name=auto-query-by-last-updated \
    -v /home/hungnv4/auto-query:/app \
    --restart=always \
    acq-go:0.1 ./srv last_updated -a=17 -s=3
    ```
    

---

# Kibana search

```SQL
GET billtrack.customer_auto_bill/_count
{
  "query": {
    "bool": {
      "must": [
        {
          "bool": {
            "must": [
              {
                "range": {
                  "last_paid_time": {
                    "gte": "2021-06-20T00:00:00.00",
                    "lte": "2021-06-21T00:00:00.00"
                  }
                }
              },
              {
                "range": {
                  "last_query_time": {
                    "gte": "2021-07-01T00:00:00.00",
                    "lte": "2021-07-21T00:00:00.00"
                  }
                }
              }
            ]
          }
        },
        {
          "match": {
            "enable_query": 1
          }
        }
      ]
    }
  }
}
```