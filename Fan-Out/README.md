# Fan-Out Pattern

# Pattern Summary
The Fan-Out pattern distributes a single published message to multiple downstream consumers simultaneously. This implementation uses an SNS topic to fan out messages to two SQS queues, each connected to its own Lambda function. Each Lambda writes the received message to a separate S3 bucket, supporting independent processing logic or pipelines.

# Architecture Diagram
```
                         +----------------------+
                         |     SNS Topic        |
                         |     fanout-topic     |
                         +-----------+----------+
                                     |
                 +-------------------+-------------------+
                 |                                       |
         +-------v--------+                     +--------v--------+
         | SQS Queue A    |                     | SQS Queue B     |
         | MetadataQueue  |                     | NotifierQueue   |
         +-------+--------+                     +--------+--------+
                 |                                       |
         +-------v--------+                     +--------v--------+
         | Lambda:        |                     | Lambda:         |
         | MetadataLambda |                     | NotifierLambda  |
         +-------+--------+                     +--------+--------+
                 |                                       |
         +-------v--------+                     +--------v--------+
         | S3 Bucket:     |                     | S3 Bucket:      |
         | metadata-bucket|                     | notifier-bucket |
         +----------------+                     +-----------------+
```

# Use Cases
- Sending a single event to multiple services for parallel handling (e.g., notifications, logging, analytics)
- Implementing loosely coupled pipelines that act on the same event data
- Maintaining audit logs and real-time response logic in separate systems

# Key Components: Resource/Purpose
- AWS::SNS::Topic	Central message broadcaster
- AWS::SQS::Queue (x2)	Receives messages for individual processing branches
- AWS::SNS::Subscription (x2)	Connects SNS to SQS queues
- AWS::Lambda::Function (x2)	Consumes messages from queues and writes them to S3
- AWS::S3::Bucket (x2)	Persists messages for audit, further processing, or archival
- AWS::IAM::Role	Grants Lambda access to SQS, S3, and CloudWatch Logs
- AWS::Lambda::EventSourceMapping (x2)	Connects SQS queues to Lambda functions
- AWS::SQS::QueuePolicy	Allows SNS to publish messages to the queues

# Behavior Summary
- A single message published to the SNS topic is delivered to both SQS queues.
- Each Lambda processes its corresponding queue independently.
- Message bodies are written to different S3 buckets:
  - metadata-message.json to metadata-bucket-jons-account
  - notifier-message.json to notifier-bucket-jons-account

# Testing the Pattern
You can test the fan-out by publishing a message to the SNS topic:

```
aws sns publish \
  --topic-arn arn:aws:sns:<region>:<account-id>:fanout-topic \
  --message '{"id": "1234", "type": "signup", "user": "alice"}'
```
Check both S3 buckets afterward for the stored message objects.

# CloudFormation Notes
- SQS policies allow only SNS to publish to the queues
- Each Lambda function is wired to a different queue via EventSourceMapping
- IAM permissions for Lambda are broad by default but can be narrowed
- SNS subscriptions use RawMessageDelivery: true for direct JSON delivery
- S3 bucket names are static in this template but can be parameterized for reusability
