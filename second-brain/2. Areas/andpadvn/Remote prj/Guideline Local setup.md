# Forward remote and workman
```
kubectl port-forward service/remote 50051:50051 -n andpad-remote


kubectl port-forward service/andpad-workman-mobile 5001:5001 -n andpad-workman
```


# Create dynamodb
```shell
# create table
aws dynamodb create-table \
    --table-name "short-urls" \
    --billing-mode PAY_PER_REQUEST \
    --attribute-definitions AttributeName=id,AttributeType=N AttributeName=group_id,AttributeType=S AttributeName=deleted_at,AttributeType=S \
    --key-schema AttributeName=id,KeyType=HASH \
    --stream-specification StreamEnabled=true,StreamViewType=NEW_AND_OLD_IMAGES \
    --sse-specification Enabled=true \
    --global-secondary-indexes \
        "[{\"IndexName\": \"partition_by_remote_group_id\",
           \"KeySchema\": [{\"AttributeName\":\"group_id\",\"KeyType\":\"HASH\"},
                          {\"AttributeName\":\"id\",\"KeyType\":\"RANGE\"}],
           \"Projection\": {\"ProjectionType\":\"ALL\"},
           \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 10}}]" \
--endpoint-url http://localhost:8000

# list tables
aws dynamodb list-tables --endpoint-url http://localhost:8000
aws dynamodb scan --table-name='short-urls' --endpoint-url http://localhost:8000
```


## Load data from testfixture
```shell
# install 
brew install go-testfixtures/tap/testfixtures


# load data
testfixtures -d mysql -c "root:password@tcp(127.0.0.1:3306)/remote_test" -D testdata/remote_fixtures
```

## localstack
```shell
# create sns
awslocal sqs create-queue --queue-name notify
awslocal sqs create-queue --queue-name notify-user
awslocal sqs create-queue --queue-name notify-push

awslocal sns create-topic --name notify
awslocal sns create-topic --name notify-user
awslocal sns create-topic --name notify-push
awslocal sns create-topic --name session-events


```


[[AWS]]