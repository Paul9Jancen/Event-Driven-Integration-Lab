# Event-Driven Integration Lab (AWS Console Only)

## Overview
This lab demonstrates a fully serverless, event-driven integration pipeline built entirely through the AWS Management Console using Free Tier services. The architecture connects **S3 → Lambda → DynamoDB → CloudWatch → SNS** to process events in real time and provide monitoring and alerting.

---

## Architecture
S3 Upload → Lambda Trigger → DynamoDB Write → CloudWatch Monitoring → SNS Alert

yaml
Copy code

---

## Components

| Service | Role |
|--------|------|
| **S3** | Event source (object uploads trigger Lambda) |
| **Lambda** | Processes S3 events and writes metadata to DynamoDB |
| **DynamoDB** | Stores event records and automatically expires them via TTL |
| **CloudWatch Logs** | Logs Lambda execution and diagnostics |
| **CloudWatch Alarms** | Monitors DynamoDB activity and throttling |
| **SNS** | Sends email notifications when alarms trigger |

---

## Features
- Console-only deployment (no CLI / no Terraform)
- Free Tier safe
- Real-time event-driven processing
- DynamoDB TTL for automatic cleanup
- CloudWatch monitoring + alerting
- SNS email notifications

---

## Setup Summary (Completed)

### 1. SNS Topic
- Topic: `event-driven-alerts`
- Email subscription added
- Tag: `Environment=Lab`

### 2. DynamoDB Table
- Table: `S3ObjectEvents`
- Partition key: `objectId (String)`
- Streams enabled: **New image**
- TTL enabled: `ttl` (24-hour expiry)

### 3. IAM Role
- Role: `Lambda-S3-DDB-Role`
- Policies attached:
  - `AWSLambdaBasicExecutionRole`
  - `AmazonDynamoDBFullAccess`
  - `AmazonS3ReadOnlyAccess`

### 4. Lambda Function
- Function: `S3ToDynamoProcessor`
- Runtime: **Python 3.12**
- Handler: `lambda_function.lambda_handler`
- Role: `Lambda-S3-DDB-Role`
- Writes records to DynamoDB with TTL

### 5. S3 Bucket
- Bucket: `event-driven-lab-pauljancen`
- Event notification configured: **All object create events**
- Trigger linked to Lambda

### 6. CloudWatch Alarms
| Alarm | Metric | Purpose |
|------|--------|---------|
| `DynamoDBWriteActivityAlert` | `ConsumedWriteCapacityUnits` | Monitors write activity |
| `DynamoDBWriteThrottleAlert` | `WriteThrottleEvents` | Detects throttling |

---

## Lambda Code (Python 3.12)

```python
import json
import boto3
import uuid
import time

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('S3ObjectEvents')

def lambda_handler(event, context):
    for record in event['Records']:
        bucket = record['s3']['bucket']['name']
        key = record['s3']['object']['key']
        table.put_item(
            Item={
                'objectId': str(uuid.uuid4()),
                'bucket': bucket,
                'objectKey': key,
                'timestamp': int(time.time()),
                'ttl': int(time.time()) + 86400
            }
        )
    return {'statusCode': 200, 'body': json.dumps('Item stored')}
Verification Steps
1. Upload Test File
Upload a file to the S3 bucket:

event-driven-lab-pauljancen

2. Confirm DynamoDB Item
Go to DynamoDB:

Table: S3ObjectEvents

Verify a new record exists with fields:

objectId

bucket

objectKey

timestamp

ttl

3. Check CloudWatch Logs
Go to:

Lambda → Monitor → View logs in CloudWatch
Verify:

Invocation succeeded

No errors

Status code 200

4. Confirm Alarm & SNS
Go to CloudWatch Alarms:

Ensure both alarms exist

Check state (OK / Insufficient data)

Go to SNS:

Confirm subscription email is confirmed

Receive email when alarms trigger
