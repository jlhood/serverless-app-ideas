# Serverless Application Ideas

This repo holds serverless application ideas that I'd love to see built and added to the [AWS Serverless Application Repository](https://aws.amazon.com/serverless/serverlessrepo/). This file contains a listing of apps with a link to a larger document detailing what the app should do and how it should behave. Feel free to add your own ideas to this list! As apps get built and published, update their status and link to the app in the Serverless Application Repository.

# Apps

| Name | Description | Status | Links |
|------|-------------|--------|-------|
| Twitter Event Source | Convert Twitter's search API into an event source for AWS Lambda | Published | [github](https://github.com/awslabs/aws-serverless-twitter-event-source), [repo](https://serverlessrepo.aws.amazon.com/applications/arn:aws:serverlessrepo:us-east-1:077246666028:applications~aws-serverless-twitter-event-source) |
| SQS Event Source | App that turns SQS into an event source for AWS Lambda | Published | [github](https://github.com/awslabs/aws-serverless-sqs-event-source), [repo](https://serverlessrepo.aws.amazon.com/applications/arn:aws:serverlessrepo:us-east-1:077246666028:applications~aws-serverless-sqs-event-source) |
| DynamoDB Copier | Function that listens on DDB stream of one table and propagates all writes to another table. | Published | [github](https://github.com/jlhood/ddb-copier), [repo](https://serverlessrepo.aws.amazon.com/applications/arn:aws:serverlessrepo:us-east-1:277187709615:applications~ddb-copier) |
| Feature Toggles | Maintains [feature toggles/flags](https://martinfowler.com/articles/feature-toggles.html) (runtime switches that enable/disable features in a distributed system). | Needs Design | [design](https://github.com/jlhood/serverless-app-ideas/blob/master/design/feature-toggles.md) |
| DynamoDB Event Sourcing | Uses Amazon DynamoDB to represent application state as an immutable sequence of events ([event sourcing](https://martinfowler.com/eaaDev/EventSourcing.html) pattern) | Needs Design | [design](https://github.com/jlhood/serverless-app-ideas/blob/master/design/dynamodb-event-sourcing.md) |
| SNS Forwarder | Single-function app that takes an SNS topic and provides a lambda function that can be invoked with a JSON array of strings and will forward them to the given SNS topic | Needs Design | [design](https://github.com/jlhood/serverless-app-ideas/blob/master/design/sns-forwarder.md) |
| Poison Pill Defender | App that acts as a proxy between a DynamoDB or Kinesis stream and a lambda function consumer and protects the lambda function from poison pills (messages that repeatedly cause the lambda function to error) | Needs Design | [design](https://github.com/jlhood/serverless-app-ideas/blob/master/design/poison-pill-defender.md) |
