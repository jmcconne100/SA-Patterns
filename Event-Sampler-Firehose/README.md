# Event Sampler + Firehose Pattern

# Pattern Summary:
This pattern implements an event sampling strategy using EventBridge and Lambda, where a subset of incoming events (e.g., 20%) are captured and forwarded to Amazon Kinesis Firehose for durable storage in S3. It’s ideal for observability, analytics, and cost-efficient monitoring when only a portion of events need to be retained.

# Architecture Diagram:
```
                   +----------------------+
                   |  Event Producer(s)   |
                   +----------+-----------+
                              |
                              v
                +-----------------------------+
                |  EventBridge: sampler-bus   |
                +------+----------------------+
                       |
                       v
             +----------------------+
             | Lambda: Sampler     |
             | - 20% event sampling|
             | - Sends to Firehose |
             +----------+----------+
                        |
                        v
          +-------------------------------+
          | Firehose: SampleFirehose      |
          | - DirectPut Delivery Stream   |
          | - Writes to S3 Bucket         |
          +-------------------------------+
                        |
                        v
         +--------------------------------+
         | S3: sampled-events-{acct}-{reg}|
         +--------------------------------+
```

# Use Cases:
- Sampling high-throughput analytics data for cost-effective observability
- Capturing event telemetry or metadata for auditing
- Feeding downstream dashboards or training datasets with partial data

# Key Components: Resource/Purpose
- AWS::Events::EventBus	Custom bus (sampler-bus) receives analytics events
- AWS::Lambda::Function	Samples incoming events with a 20% probability
- AWS::KinesisFirehose::DeliveryStream	Buffers and writes sampled data to S3
- AWS::S3::Bucket	Stores sampled event JSONs
- AWS::Events::Rule	Routes matching events to the sampler Lambda
- AWS::IAM::Role	Grants Lambda and Firehose appropriate access

# Event Pattern:
Only events with this source are sampled:

```
{
  "source": ["analytics.events"]
}
```

You can publish test events with this CLI:

```
aws events put-events --entries '[
  {
    "Source": "analytics.events",
    "DetailType": "UserEvent",
    "Detail": "{\"user_id\": 123, \"event\": \"click\"}",
    "EventBusName": "sampler-bus"
  }
]'
```

# CloudFormation Notes:
- Firehose is created in DirectPut mode and writes directly to S3.
- Lambda uses the firehose:PutRecord API to send sampled data.
- IAM permissions are scoped to Firehose, S3, and basic CloudWatch logging.
- EventBridge Rule connects the custom event bus to the Lambda.
- S3 bucket structure will follow Firehose's default time-based partitioning unless configured otherwise.
