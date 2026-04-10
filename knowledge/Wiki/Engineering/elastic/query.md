---
title: "Limit"
date: 2026-01-14
tags:
  - engineering
  - elasticsearch
---

# Limit 
```
GET /_search
{
  "from": 5,
  "size": 20,
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  }
}
```

[[upsert]]

[[Kafka]]