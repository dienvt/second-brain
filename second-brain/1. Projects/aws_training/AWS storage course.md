---
tags:
  - aws_training
---
**on-premises**: meaning in our company, private data

## S3 store policy
| type                         | defination                 | use-case      |
| ---------------------------- | -------------------------- | ------------- |
| S3 standard                  |                            | Fast retrieve |
| S3 standard IA               | Lower prices               | Less access   |
| S3 one zone  IA              |                            |               |
| S3 glacier flexible retrival | take 1-5 minutes to access |               |
| S3 glacier deep archive      | take 20 hours to access    |               |
| S3 Itelligen tiering         |                            |               |

Transfer policy
New add -> transfer to standard IA object -> archive -> remove (after 1 year)
