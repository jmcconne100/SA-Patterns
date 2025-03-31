# Fan-In Pattern

# Pattern Summary
The Fan-In pattern aggregates messages from multiple SQS queues into a single Lambda function, which processes the messages in batches and writes them to S3. This architecture enables scalable, cost-efficient consolidation of event streams from different sources or components into a unified processing pipeline.

# Architecture Diagram
```
     +----------------+     +----------------+     +----------------+
     | SQS Queue A    |     | SQS Queue B    |     | SQS Queue C    |
     | fanin-buffer-a |     | fanin-buffer-b |     | fanin-buffer-c |
     +--------+-------+     +--------+-------+     +--------+-------+
              \                    |                       /
               \                   |                      /
                \                  |                     /
                 \_________________|____________________/
                                   |
                                   v
                    +-----------------------------+
                    | Lambda: FanInBatchLambda    |
                    | - Batches messages          |
                    | - Writes JSON to S3         |
                    +-------------+---------------+
                                  |
                                  v
             +----------------------------------------------+
             | S3 Bucket: fanin-batch-{account-id}-{region} |
             | - Batches saved under /batches/ prefix       |
             +----------------------------------------------+
```

# Use Cases
- Aggregating logs or telemetry from multiple sources
- Merging different ingestion pipelines into a single processing stage
- Preparing data for bulk transformation or ETL processes
- Centralizing event stream capture across services or environments

# Key Components: Resource/Purpose
- AWS::SQS::Queue (x3)	Independent sources of messages to fan into processing
- AWS::Lambda::Function	Processes batches of SQS messages and stores them in S3
- AWS::S3::Bucket	Stores consolidated JSON files of batched records
- AWS::IAM::Role	Grants Lambda permissions for SQS, S3, and logging
- AWS::Lambda::EventSourceMapping (x3)	Connects each SQS queue to the Lambda function

# Batching Logic
- Each Lambda invocation receives a batch of up to 10 messages (configurable)
- Messages are parsed from JSON strings and grouped into an array
- The array is serialized and saved to S3 as a timestamped .json file under the /batches/ folder

# Example S3 key:
- batches/2025-03-30T07:55:00.000Z_d2c7f9a1f3454b4b8aa312d.json

# Testing the Pattern
You can send a message to any of the queues using the AWS CLI:

```
aws sqs send-message \
  --queue-url https://sqs.<region>.amazonaws.com/<account-id>/fanin-buffer-queue-a \
  --message-body '{"source": "service-a", "action": "eventX", "timestamp": "2025-03-30T07:55:00Z"}'
```
After a few messages, the Lambda will be triggered and write a new batch file to S3.

# CloudFormation Notes
- The S3 bucket is created using the account ID and region to avoid name conflicts
- Lambda code is embedded inline and uses Python 3.11
- IAM policy grants wildcard S3 write access for flexibility; it can be narrowed
- Event source mappings are configured with MaximumBatchingWindowInSeconds: 5 to allow short latency before writing
