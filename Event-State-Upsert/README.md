Event-Carried State with Idempotent Upserts
Pattern Summary
This pattern ensures that event-driven state updates are reliably applied to a target database (DynamoDB) by carrying the entire state in the event itself and making the operation idempotent. This avoids double processing and ensures eventual consistency even under retry conditions or duplicates.

# Architecture Diagram
```
                   +------------------------+
                   |   Event Producer(s)    |
                   +-----------+------------+
                               |
                               v
                   +------------------------+
                   |     SQS Queue          |
                   |  event-state-queue     |
                   +-----------+------------+
                               |
                               v
                   +------------------------+
                   | Lambda: EventUpsertLambda |
                   | - Reads from SQS           |
                   | - Writes to DynamoDB       |
                   +-----------+----------------+
                               |
                               v
                 +------------------------------------+
                 | DynamoDB: event-state-target-table |
                 | - Key: event_id                    |
                 | - Entire event = stored item       |
                 +------------------------------------+
```
# Use Cases
- Recording full state snapshots in an append-only or upsert table
- Debouncing repeated events that carry identical or near-identical state
- Avoiding partial reads/writes and cross-service coordination
- Safely retrying message delivery and processing without data corruption

# Key Components: Resource/Purpose
- AWS::SQS::Queue	Buffer and decouple events for delivery
- AWS::Lambda::Function	Parses JSON event payloads and writes to DynamoDB
- AWS::DynamoDB::Table	Stores the event-carried state using event_id as key
- AWS::IAM::Role	Allows Lambda to write logs, consume SQS, and write to DB
- AWS::Lambda::EventSourceMapping	Binds the SQS queue to Lambda for automatic invocation

# Event Format
Each event should be a JSON object containing at least an event_id key.
Example:
```
{
  "event_id": "evt-1001",
  "user_id": 42,
  "status": "active",
  "timestamp": "2025-03-30T01:00:00Z"
}
```
This entire payload becomes the stored record in DynamoDB.

# Testing the Pattern
You can send a test event using the AWS CLI:
```
aws sqs send-message \
  --queue-url https://sqs.<region>.amazonaws.com/<account-id>/event-state-queue \
  --message-body '{"event_id":"evt-1001","user_id":42,"status":"active","timestamp":"2025-03-30T01:00:00Z"}'
```
# CloudFormation Notes:
- DynamoDB uses PAY_PER_REQUEST billing mode for automatic scaling
- Lambda environment variable TABLE_NAME is injected using !Ref
- IAM policy allows full SQS receive/delete and DynamoDB PutItem
- Lambda is invoked via an event source mapping from SQS (batch size 10)
