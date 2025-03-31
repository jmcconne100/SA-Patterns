# EventBridge Replay + Archive Pattern

# Pattern Summary
This pattern captures and archives specific event traffic on a custom EventBridge bus and provides the capability to replay those events into a Lambda target. It is useful for forensic debugging, restoring state, backfilling data pipelines, or rerunning downstream logic for a specific event range or window.

# Architecture Diagram:
```
                    +------------------------+
                    |    CDC Event Source    |
                    | (e.g., app.dynamodb)   |
                    +-----------+------------+
                                |
                                v
                    +------------------------+
                    |  EventBridge (Custom)  |
                    |   outbox-event-bus     |
                    +-----------+------------+
                                |
                                v
                +-------------------------------+
                | Archive: OutboxEventArchive   |
                | - Stores all matching events  |
                | - Source: "app.dynamodb"      |
                +-------------------------------+

    (manual replay via Console or CLI)

                                |
                                v
                    +------------------------+
                    | Rule: OutboxReplayRule |
                    | - Matches "app.dynamodb"|
                    +-----------+------------+
                                |
                                v
                  +-----------------------------+
                  | Lambda: ReplayTargetLambda  |
                  | - Receives replayed events  |
                  +-----------------------------+
```

# Use Cases:
- Reprocessing CDC (Change Data Capture) events after failure
- Debugging and auditing historical event payloads
- Backfilling data into downstream systems
- Simulating production workloads for staging environments

# Key Components: Resource/Purpose
- AWS::Events::EventBus	Custom bus (outbox-event-bus) to isolate CDC event traffic
- AWS::Events::Archive	Captures and stores matching events for later replay
- AWS::Events::Rule	Replays archived events into a Lambda based on matching rule
- AWS::Lambda::Function	Consumes the replayed event data
- AWS::IAM::Role	Grants Lambda access to log events to CloudWatch
- AWS::Lambda::Permission	Allows EventBridge to invoke the Lambda

# Event Filtering:
The archive and replay rule both filter on the event source:
```
{
  "source": ["app.dynamodb"]
}
```
Only events with this source field are captured and eligible for replay.

# How to Replay Events:
- Navigate to Amazon EventBridge > Archives
- Select the OutboxEventArchive
- Click "Start replay"

# Choose:
- Time range of events to replay
- Destination bus (should be outbox-event-bus)
- The replayed events will be routed through the existing OutboxReplayRule and sent to the ReplayTargetLambda.

# CloudFormation Notes
- Archive is configured with a matching event pattern and source ARN from the custom bus
- Replayed events must still match the event pattern defined in the replay rule
- All permissions required for invocation and logging are preconfigured
- You can extend this architecture by swapping the Lambda with an SQS queue, Step Functions, or Kinesis
