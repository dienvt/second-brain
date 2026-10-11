From garafana 12, we have to add datasource in the aws managed console
```
Starting in Amazon Managed Grafana version 12, SigV4 authentication support in the Core Prometheus plugin has been removed. All Amazon Managed Service for Prometheus data sources that were previously using the Core Prometheus plugin are automatically migrated to the Amazon Managed Service for Prometheus plugin. Any dashboards using these data sources are automatically updated to reflect this change. See [Connect to an Amazon Managed Service for Prometheus data source](https://docs.aws.amazon.com/grafana/latest/userguide/amazon-prometheus-data-source.html).
```
https://docs.aws.amazon.com/grafana/latest/userguide/amazon-AMP-adding-AWS-config.html

