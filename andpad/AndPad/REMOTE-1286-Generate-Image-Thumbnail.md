https://aws.amazon.com/blogs/networking-and-content-delivery/image-optimization-using-amazon-cloudfront-and-aws-lambda/

Using
https://aws.amazon.com/lambda/
https://aws.amazon.com/s3/
https://aws.amazon.com/cloudfront/


 `AWS Cloudfront Signed URL`
  - Canned policy
  - Custom policy
  
  `AWS S3 pre-signed URL`



FE -> presign URL từ S3, 

proto: Sửa proto, Tạo PR bên andpad API rồi update sau.


presigned URL lấy được meta data nên có thể inject vào response luôn.
https://docs.aws.amazon.com/AmazonS3/latest/userguide/ShareObjectPreSignedURL.html
presigned URL phía FE có lấy được metadata hay không?



Data sẽ ntn? thông tin user gồm có gì, user name + id?


proto file in (andpad must contain) api APIs


### Create photos
temporaryQueriesGateway
FileStorageQueriesGateway -> s3 reader
Move file from temprary to pernament
remove file in s3 temporary and update db
return pernament URL

## Question

What do `temporaries` table store?
Are those image files, using has shoot ?

When was the photo uploaded? Cause in Endpoint `createSessionPhotos` and `uploadOrderSessionPhotos` using string?



## Solution
Read user & upload date from db. Shooted date from image meta data

chỉ có thông tin user thôi nên là sẽ chỉ truyền 

### Change bên APIs
SessionPhoto??
Photo??

### Change bên BFF trả về thông tin user full

Mỗi lần send thì nó có tốn resource hay không ?
update flow lưu thông tin và xuất thông tin đồng thời update cả docs
có 

### Change bên remote
* [x] store shot_at
* [ ] return shot_at
* [ ] 

