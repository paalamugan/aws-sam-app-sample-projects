# aws-sam-app-sample-projects
AWS sam app sample example projects that included step-functions-demo, lambda-function-demo and dynamodb-demo.

## Pre-requisites

1. To Install aws-cli, follow the instructions at [https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html]()
2. To Install aws-sam-cli, follow the instructions at [https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html]()
3. To Install Docker, follow the instructions at [https://docs.docker.com/get-docker/]()
4. To install NoSQL Workbench, follow the instructions at [https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/workbench.settingup.html]()

## Run the Docker Compose Using Below Command

```sh
docker-compose up -d
```

## General SAM CLI Command

sam init - Used to create a new project

sam local invoke - Used to trigger a function locally

sam local start-api - Used to start a api server locally

sam local start-lambda - Used to start a lambda function locally

sam local generate-event - Used to generate a event payload locally (Like, sam local generate-event dynamodb update > event.json)

## To test the dynamodb function locally

Invoke dynamodb lambda add item function
`sam local invoke putItemFunction -e ./events/event-post-item.json -n env.json`

Invoke dynamodb lambda get by id item function
`sam local invoke getByIdFunction -e ./events/event-get-by-id.json -n env.json`

Invoke dynamodb lambda get all items function
`sam local invoke getAllItemsFunction -e ./events/event-get-all-items.json  -n env.json`

## How to import cloud dynamodb table data into local dynamodb table through cli

1. aws dynamodb describe-table --table-name `<table-name>` --region us-east-1 --output json > table_structure.json
2. aws dynamodb scan --table-name `<table-name>` --region us-east-1 --output json > data.json
3. aws dynamodb create-table --cli-input-json file://table_structure.json --endpoint-url http://localhost:8000
4. aws dynamodb batch-write-item --request-items file://data.json --endpoint-url http://localhost:8000
5. aws --endpoint-url=http://localhost:4566 dynamodb list-tables
6. aws --endpoint-url=http://localhost:4566 dynamodb create-table --table-name myTable --attribute-definitions AttributeName=id,AttributeType=N --key-schema AttributeName=id,KeyType=HASH --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5

## Step functions Instructions

## LambdaSQSIntegration Step functions

aws stepfunctions create-state-machine --endpoint http://localhost:8083 --definition file://state-machine.json --name "LambdaSQSIntegrationState" --role-arn "arn:aws:iam::012345678901:role/DummyRole"

aws stepfunctions start-execution --endpoint http://localhost:8083 --name executionWithTwoState --state-machine arn:aws:states:us-east-1:123456789012:stateMachine:LambdaSQSIntegrationState

aws stepfunctions get-execution-history --endpoint http://localhost:8083 --execution-arn arn:aws:states:us-east-1:123456789012:execution:LambdaSQSIntegrationState:executionWithTwoState

aws stepfunctions --endpoint http://localhost:8083 delete-state-machine --state-machine-arn "arn:aws:states:us-east-1:123456789012:stateMachine:LambdaSQSIntegrationState"

## Sample Hello World Lambda Function

aws stepfunctions --endpoint http://localhost:8083 create-state-machine --definition file://hello-world-state-machine.json --name "HelloWorld" --role-arn "arn:aws:iam::012345678901:role/DummyRole"

aws stepfunctions --endpoint http://localhost:8083 start-execution --state-machine arn:aws:states:us-east-1:123456789012:stateMachine:HelloWorld --name test

aws stepfunctions --endpoint http://localhost:8083 describe-execution --execution-arn arn:aws:states:us-east-1:123456789012:execution:HelloWorld:test

## How to run stepfunctions local service using docker

docker run -p 8083:8083 --env-file aws-stepfunctions-local-credentials.txt amazon/aws-stepfunctions-local

docker run -p 8083:8083 --mount type=bind,readonly,source=$(pwd)/MockConfigFile.json,destination=/home/StepFunctionsLocal/MockConfigFile.json -e SFN_MOCK_CONFIG="/home/StepFunctionsLocal/MockConfigFile.json" --env-file aws-stepfunctions-local-credentials.txt amazon/aws-stepfunctions-local

## The steps to add item in the dynamodb directly without apigateway lambda function

aws stepfunctions --endpoint http://localhost:8083 create-state-machine --definition file://dynamodb-remote-state-machine.json --role-arn "arn:aws:iam::012345678901:role/DummyRole" --name "DynamoDBStateMachine"

aws stepfunctions --endpoint http://localhost:8083 start-execution --state-machine-arn "arn:aws:states:us-east-1:123456789012:stateMachine:DynamoDBStateMachine" --name test --input '{"enclosedItemType":"enclosedItemType1", "itemCode":"itemCode1"}'

aws stepfunctions --endpoint http://localhost:8083 describe-execution --execution-arn arn:aws:states:us-east-1:123456789012:execution:DynamoDBStateMachine:test

aws stepfunctions --endpoint http://localhost:8083 delete-state-machine --state-machine-arn "arn:aws:states:us-east-1:123456789012:stateMachine:DynamoDBStateMachine"

## The steps to get all items in the dynamodb through apigateway lambda function

aws stepfunctions --endpoint http://localhost:8083 create-state-machine --definition file://get-all-dynamodb-remote-state-machine.json --role-arn "arn:aws:iam::012345678901:role/DummyRole" --name "DynamoDBStateMachine"

aws stepfunctions --endpoint http://localhost:8083 start-execution --state-machine-arn "arn:aws:states:us-east-1:123456789012:stateMachine:DynamoDBStateMachine" --name test --input file://./dynamodb-demo/events/event-get-all-items.json

aws stepfunctions --endpoint http://localhost:8083 describe-execution --execution-arn arn:aws:states:us-east-1:123456789012:execution:DynamoDBStateMachine:test

aws stepfunctions --endpoint http://localhost:8083 delete-state-machine --state-machine-arn "arn:aws:states:us-east-1:123456789012:stateMachine:DynamoDBStateMachine"

## The steps to add a item in the dynamodb through apigateway lambda function

aws stepfunctions --endpoint http://localhost:8083 create-state-machine --definition file://put-and-get-all-dynamodb-remote-state-machine.json --role-arn "arn:aws:iam::012345678901:role/DummyRole" --name "DynamoDBStateMachine"

aws stepfunctions --endpoint http://localhost:8083 start-execution --state-machine-arn "arn:aws:states:us-east-1:123456789012:stateMachine:DynamoDBStateMachine" --name test --input file://./dynamodb-demo/events/event-post-item.json

aws stepfunctions --endpoint http://localhost:8083 describe-execution --execution-arn arn:aws:states:us-east-1:123456789012:execution:DynamoDBStateMachine:tes

## How to create SQS queye using AWS command line

To create an SQS queue in LocalStack, you can use the AWS CLI with the `--endpoint-url` option set to the LocalStack endpoint. Here's an example:

`aws --endpoint-url=http://localhost:4566 sqs create-queue --queue-name sqsQueue`

This command will create a standard SQS queue. If you want to create a FIFO queue, you need to add the `.fifo` suffix to the queue name and set the `FifoQueue` attribute to `true`. Here's an example:

`aws --endpoint-url=http://localhost:4566 sqs create-queue --queue-name sqsQueue.fifo --attributes FifoQueue=true`
