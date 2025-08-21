# Notes

https://andpadvietnam.slack.com/archives/C01SUTLJYSG/p1730288503710889

terraform
```diff
resource "aws_dynamodb_table" "remote_bff_dynamodb_short_urls" {
  name             = var.remote_bff_short_url_table
  billing_mode     = "PAY_PER_REQUEST"
  hash_key         = "id"
  stream_enabled   = true
  stream_view_type = "NEW_AND_OLD_IMAGES"

  point_in_time_recovery {
    enabled = var.env == "production" ? true : false
  }

  server_side_encryption {
    enabled = true
  }

  dynamic "replica" {
    for_each = var.replica
    content {
      region_name            = replica.value.region_name
      propagate_tags         = replica.value.propagate_tags
      point_in_time_recovery = replica.value.point_in_time_recovery
    }
  }

  attribute {
    name = "id"
    type = "N"
  }

 attribute {
    name = "group_id"
    type = "S"
  }

  global_secondary_index {
    name               = "partition_by_remote_group_id"
    hash_key           = "group_id"
    range_key          = "id"
    write_capacity     = 10
    read_capacity      = 10
    projection_type    = "ALL"
  }

+  deletion_protection_enabled = true
}
```