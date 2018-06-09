# Poison Pill Defender

Poison Pill Defender is a serverless app that protects stream-based Lambdas from "poison pill" messages, i.e., messages that cause the Lambda function to repeatedly fail.

From the AWS documentation on [Lambda error retry behavior](https://docs.aws.amazon.com/lambda/latest/dg/retries-on-errors.html):

> **Stream-based event sources** – For stream-based event sources (Amazon Kinesis Data Streams and DynamoDB streams), AWS Lambda polls your stream and invokes your Lambda function. Therefore, if a Lambda function fails, AWS Lambda attempts to process the erring batch of records until the time the data expires, which can be up to seven days for Amazon Kinesis Data Streams. The exception is treated as blocking, and AWS Lambda will not read any new records from the stream until the failed batch of records either expires or processed successfully. This ensures that AWS Lambda processes the stream events in order.

While this behavior is understandable for preserving order of messages, in a production scenario, having a stream processor Lambda stuck on a message for up to 7 days is frequently not acceptable. With Poison Pill Defender, messages that cause a stream-based Lambda to repeatedly fail are instead sent to a Dead-Letter Queue (DLQ) so the Lambda is able to continue processing messages. It also provides a Lambda function for operational tasks on the DLQ such as redriving messages or draining them from the DLQ.

**WARNING:** Adding the Poison Pill Defender app in front of your stream-based Lambda functions means the message ordering provided by the DynamoDB or Kinesis streams will no longer be guaranteed in an error scenario. Therefore, your stream-based Lambdas must be written to tolerate out of order messages if you choose to use this serverless app. For cases where order absolutely must be preserved, this app should not be used.

## Architecture

![App Architecture](https://github.com/jlhood/serverless-app-ideas/raw/master/design/images/poison-pill-defender-architecture.png)

1. Rather than connecting a stream-based Lambda directly to a DynamoDB or Kinesis stream, the app's Poison Pill Defender Lambda is connected instead.
1. The Poison Pill Defender Lambda receives batches of messages from the stream and synchronously invokes the Protected Stream Processor Lambda to process them.
1. If the Protected Stream Processor Lambda errors on a batch of messages, the Poison Pill Defender breaks the batch up into individual messages and invokes the Protected Stream Processor Lambda with each message until it finds the message that causes repeated errors. It then puts that message on the DLQ, which is an SQS queue.
1. The DLQOps Lambda is a function that allows an operator to take action on the DLQ.

## DLQOps Lambda

The DLQOps Lambda is a function that allows an operator to take action on the DLQ. It provides two operations: redrive and drain.

Redrive reads messages off the DLQ, deletes them, and invokes the Poison Pill Defender Lambda to process the DLQ messages. If processing fails again, they will be added to the DLQ by the Poison Pill Defender Lambda. This operation is useful if the error was caused by a bug in the Protected Stream Processor Lambda that has since been fixed or a transient dependency failure that has been resolved.

Drain reads messages off the DLQ and deletes them. These messages will be lost so this operation should be used with care.

### Request Structure

TODO

### Response Structure

TODO

## Parameters

1. `ProtectedStreamProcessorFunctionArn` - ARN of the Lambda function that should be protected by this Poison Pill Defender app.
1. `SourceKinesisStreamArn` - Kinesis stream ARN. Populate if this Poison Pill Defender app should protect a Kinesis stream.
1. `SourceDynamoDBTableStreamArn` - DynamoDB table stream ARN. Use if this Poison Pill Defender app should protect a DynamoDB stream.
1. `StartingPosition` - Starting position of Poison Pill Defender Lambda. Default: TRIM_HORIZON
1. `DLQMessageRetentionPeriod` - Message retention period of DLQ in seconds. Default: 604800 (7 days)
1. `BatchSize` - Batch size of Poison Pill Defender Lambda function trigger. Default: 5
