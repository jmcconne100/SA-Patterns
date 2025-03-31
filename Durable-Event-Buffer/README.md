# Durable Event Buffer

# Pattern Summary:
The Durable Event Buffer pattern captures events sent to an EventBridge bus, persistently stores them in S3, and simultaneously fans them out to an SQS queue for downstream processing. It provides a resilient and decoupled architecture to handle high-throughput event ingestion, protect against processing failures, and ensure that raw event data is always retained.

# Use Cases:
Capturing all incoming event data for compliance or debugging
Decoupling event producers and consumers
Supporting replay, auditing, or delayed reprocessing

# Key Features:
Persistent event capture using Amazon S3
EventBridge-based fan-out to SQS for concurrent consumers
Simple Lambda-based ingestion and buffering logic
All infrastructure is provisioned automatically via CloudFormation
CloudFormation Architecture Notes

# Core Components:
- AWS::Events::EventBus
A custom EventBridge bus (durable-event-bus) receives all incoming events.
- AWS::Lambda::Function
The S3BufferLambda writes the full event payload to S3 using an ISO timestamp and UUID-based key for uniqueness.
- AWS::S3::Bucket
The bucket durable-event-buffer-* stores all raw events under the raw-events/ prefix.
- AWS::SQS::Queue
The durable-fanout-queue is subscribed to the same event pattern and receives copies of each incoming event for parallel processing.
- AWS::Events::Rule (x2)
One rule routes matching events to the Lambda function for storage in S3, and another rule routes the same events to the SQS queue.
- AWS::IAM::Role
Provides the Lambda permission to write to S3 and log to CloudWatch.
- AWS::Lambda::Permission and AWS::SQS::QueuePolicy
Ensure that EventBridge has permission to invoke Lambda and send messages to the queue.

# Event Pattern Used:

EventPattern:
  source: ["buffer.source"]
You can publish events to this pattern by setting the source field of your event to "buffer.source".

Example CLI Test:
```
aws events put-events \
  --entries '[
    {
      "Source": "buffer.source",
      "DetailType": "BufferedEvent",
      "Detail": "{\"user_id\": 123, \"action\": \"signup\"}",
      "EventBusName": "durable-event-bus"
    }
  ]'
```

# Delivery Notes:
Events are immediately delivered to both S3 (via Lambda) and the SQS queue.
Events stored in S3 are complete payloads in JSON format.
The fan-out queue allows batch workers, consumers, or processing pipelines to subscribe independently.
