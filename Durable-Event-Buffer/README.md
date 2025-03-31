# Durable Event Buffer

# Pattern Summary:
The Durable Event Buffer pattern captures and stores incoming events reliably before downstream processing. It ensures that transient issues such as service throttling, batch window timing, or compute failures do not result in data loss. This architecture decouples producers from consumers and enables flexible, fault-tolerant event handling at scale.

# Use Case Examples:

Ingest events from multiple producers and batch-process them later

Handle traffic spikes gracefully

Queue events for machine learning pipelines or ETL workflows

# Core Features:

Durable buffering using Amazon Kinesis Firehose

Writes to Amazon S3 in raw format for long-term storage

Optional downstream processing using AWS Glue, Athena, or Lambda

Fully managed and serverless infrastructure

CloudFormation Architecture Notes

# Managed Components:

The AWS::KinesisFirehose::DeliveryStream resource creates a delivery stream with a direct PUT interface

The AWS::S3::Bucket resource provisions the storage destination for buffered events

The AWS::IAM::Role allows Firehose to write to the bucket and publish delivery logs to CloudWatch

# Delivery and Logging:

S3DestinationConfiguration specifies the target S3 bucket and logging configuration

CloudWatchLoggingOptions enables Firehose to write delivery logs to a dedicated log group and stream

# S3 Folder Structure:

By default, Firehose organizes data into S3 using a date-based prefix structure: /YYYY/MM/DD/HH/

This can be customized by adjusting the Prefix property in the template

Testing: You can send test records using the AWS CLI:

```
aws firehose put-record \
  --delivery-stream-name firehose-to-s3-buffer \
  --record='{"Data":"{\"event\":\"upload\",\"user_id\":123}\n"}'
```

Firehose buffers data until it reaches a size threshold or timeout period, after which it delivers the batch to S3.