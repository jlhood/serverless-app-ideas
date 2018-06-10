# Metrics Publisher

The Metrics Publisher serverless app provides efficient, async batch publishing of CloudWatch Metrics supporting all features supported by CloudWatch Metrics' [PutMetricData API](https://docs.aws.amazon.com/AmazonCloudWatch/latest/APIReference/API_PutMetricData.html).

AWS Lambda offers a set of included CloudWatch metrics, such as invocation and error counts. However, more complex production applications frequently want to record application-specific custom metrics. Today in Lambda, there are two primary ways to accomplish this.

The first option is to have the Lambda call CloudWatch Metrics' PutMetricData API directly on each request. However, that API has a [150 transactions per second (TPS) limit](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/cloudwatch_limits.html). This means at scale, metrics API requests will be throttled at which point either you can choose to fail the Lambda request or drop the metrics.

The second option is to use [CloudWatch Logs Metric Filters](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/MonitoringPolicyExamples.html) feature. This feature allows you to log metrics and define filters that convert those logs into metrics. This works at high scale but has two major drawbacks: (1) dimensions and percentiles are not supported, and (2) it dramatically increases the friction to adding new metrics to your serverless app.

The Metrics Publisher serverless app is designed to provide a third option: invoke a lambda function with all custom metrics collected for a given Lambda request. Then asynchronously, the app batches metrics across Lambda requests in order to minimize calls to PutMetricData.

## Architecture

![App Architecture](https://github.com/jlhood/serverless-app-ideas/raw/master/design/images/metrics-publisher-architecture.png)

1. The app provides a MetricsRecorder Lambda which can be invoked either synchronously or asynchronously to record a batch of metrics. It accepts input very similar to the PutMetricData API.
1. The MetricsRecorder Lambda writes the metrics to a CloudWatch Log Group (created and managed internally by the app) and saves the namespace to a DynamoDB table.
1. A CloudWatch Events Rule triggers a scheduled Dispatcher Lambda, which async invokes a NamespacePublisher Lambda for each metric namespace that requires recording.
1. The NamespacePublisher Lambda reads metrics from the CloudWatch Log Group, and batches them into PutMetricData API calls. It maintains a cursor in the DynamoDB table so it can continue to the next metrics on subsequent invokes.

## MetricsRecorder Lambda

The MetricsRecorder Lambda is meant to be invoked with metrics data in a format similar to the PutMetricData API. It also adds some additional fields for idempotency so it doesn't log the same metrics multiple times.

TODO: Request Format

## Parameters

1. `TODO` - TODO
