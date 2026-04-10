#### KudoFoto

|Name|Assign|Created By|Date Created|Due Date|Priority|Status|Tags|
|---|---|---|---|---|---|---|---|
|[[S3 Preasigned link]]||DDien Vo|May 10, 2021 9:08 AM|October 28, 2019|High 🔥|Next Up|User Need|
|[[Turning resize speed]]||DDien Vo|July 12, 2021 9:11 AM|||||
|[[ENV]]||DDien Vo|July 7, 2021 8:50 AM|||||
|[[Resize benchmark]]||DDien Vo|May 16, 2021 8:34 PM|||Completed||
|[[Upload images by campaign]]||DDien Vo|May 16, 2021 5:03 PM|||Next Up|Product Request|

  
  

  

#### ENV

|Name|Tags|
|---|---|
|[[STG]]||
|[[REAL]]||
|[[SB]]||
|[[Jenkins]]||
|[[Account]]||

  
  

  

# Drive GG API

project: [https://console.cloud.google.com/home/dashboard?project=kudo-foto&authuser=1](https://console.cloud.google.com/home/dashboard?project=kudo-foto&authuser=1)

1. Create credencial: [https://console.cloud.google.com/apis/credentials?authuser=1&project=kudo-foto](https://console.cloud.google.com/apis/credentials?authuser=1&project=kudo-foto)
2. Enable gg drive API: [https://console.cloud.google.com/marketplace/product/google/drive.googleapis.com?q=search&referrer=search&authuser=1&project=kudo-foto](https://console.cloud.google.com/marketplace/product/google/drive.googleapis.com?q=search&referrer=search&authuser=1&project=kudo-foto)

# Grafana

```SQL
{
  "annotations": {
    "list": [
      {
        "builtIn": 1,
        "datasource": "-- Grafana --",
        "enable": true,
        "hide": true,
        "iconColor": "rgba(0, 211, 255, 1)",
        "name": "Annotations & Alerts",
        "type": "dashboard"
      }
    ]
  },
  "editable": true,
  "gnetId": null,
  "graphTooltip": 0,
  "id": 1,
  "links": [],
  "panels": [
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": null,
      "fieldConfig": {
        "defaults": {},
        "overrides": []
      },
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 12,
        "w": 21,
        "x": 0,
        "y": 0
      },
      "hiddenSeries": false,
      "id": 2,
      "legend": {
        "alignAsTable": true,
        "avg": false,
        "current": false,
        "max": true,
        "min": true,
        "show": true,
        "total": false,
        "values": true
      },
      "lines": true,
      "linewidth": 1,
      "nullPointMode": "null",
      "options": {
        "alertThreshold": true
      },
      "percentage": false,
      "pluginVersion": "7.5.5",
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "exemplar": true,
          "expr": "(histogram_quantile(0.99, sum(rate(kudo_foto_consumer_request_duration_seconds_bucket{}[1m])) by (path, servicename, transport, le)))",
          "hide": false,
          "interval": "",
          "legendFormat": "{{path}}",
          "refId": "A"
        },
        {
          "exemplar": true,
          "expr": "kudo_foto_consumer_request_duration_seconds_bucket",
          "hide": true,
          "interval": "",
          "legendFormat": "",
          "refId": "B"
        }
      ],
      "thresholds": [],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "P99 monitoring",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "format": "s",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    }
  ],
  "refresh": "5s",
  "schemaVersion": 27,
  "style": "dark",
  "tags": [],
  "templating": {
    "list": []
  },
  "time": {
    "from": "now-30m",
    "to": "now"
  },
  "timepicker": {},
  "timezone": "",
  "title": "Kudo",
  "uid": "Gv-nKdW7z",
  "version": 5
}
```

  

sum(go_goroutines{job="evn-providers"}) by (instance)

  

aws docker: [https://docs.aws.amazon.com/AmazonECS/latest/developerguide/docker-basics.html](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/docker-basics.html)

  

[[DB design]]