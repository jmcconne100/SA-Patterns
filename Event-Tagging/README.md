# Event Tagging + Metadata Enrichment

# Pattern Summary
This pattern enriches incoming events by appending metadata such as tags, ingestion time, and region before persisting them into S3. This enables lightweight event annotation for auditing, downstream analytics, or tracking provenance without requiring a database write.

# Architecture Diagram:
```
                   +------------------------+
                   |    Event Producer(s)   |
                   +-----------+------------+
                               |
                               v
                    +-----------------------+
                    | Lambda: TaggerLambda |
                    | - Adds metadata       |
                    | - Writes to S3        |
                    +-----------+-----------+
                                |
                                v
          +-------------------------------------------+
          | S3 Bucket: tagged-event-output-{acct}-{reg}|
          | - Prefix: /tagged/                         |
          +-------------------------------------------+
```

# Use Cases:
- Tagging events with ingestion metadata for traceability
- Pre-processing before storage or downstream workflows
- Writing enriched data to S3 for later analysis, ML, or ETL
- Replacing lightweight CDC pipelines with event-based enrichment

# Key Components: Resource/Purpose
- AWS::Lambda::Function	Adds metadata fields and writes enriched event to S3
- AWS::S3::Bucket	Stores the enriched JSON event files
- AWS::IAM::Role	Grants Lambda permissions to write to S3 and log
- AWS::Lambda::Permission	Allows the function to be triggered manually or via EventBridge

# Metadata Enrichment Logic:
Each event is tagged with the following metadata:
```
{
  "metadata": {
    "source_region": "<aws-region>",
    "event_tag": "ingested",
    "ingestion_time": "<ISO-8601 timestamp>"
  }
}
```
The enriched event is stored in S3 with a generated UUID filename under the tagged/ prefix.

# Example Event Payload (Before and After):
Input Event:
```
{
  "user_id": 42,
  "action": "login"
}
```
Stored Output:
```
{
  "user_id": 42,
  "action": "login",
  "metadata": {
    "source_region": "us-west-1",
    "event_tag": "ingested",
    "ingestion_time": "2025-03-30T07:41:00.000Z"
  }
}
```

# Testing the Pattern:
You can test the Lambda manually from the AWS Console or use a tool like the AWS CLI:

```
aws lambda invoke \
  --function-name <your-function-name> \
  --payload '{ "user_id": 42, "action": "login" }' \
  output.json
```
Then check your S3 bucket’s tagged/ prefix for the enriched file.

# CloudFormation Notes:
- Lambda is written inline with Python 3.11
- The S3 bucket and key prefix are fully configurable
- IAM role grants minimal required permissions to write and log
- Lambda permission is open to EventBridge but can be scoped further
- This is a stateless, single-step enrichment process ideal for glue code between services
