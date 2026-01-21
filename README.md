Event-Driven Integration Lab (AWS Console Only)
Overview

A serverless, event-driven pipeline built entirely through the AWS Console using only Free Tier services.
This architecture connects S3 → Lambda → DynamoDB → CloudWatch → SNS to demonstrate real-time event processing, monitoring, and alerting.

Architecture Flow
S3 Upload → Lambda Trigger → DynamoDB Write → CloudWatch Monitoring → SNS Alert

Components
Service	Purpose
S3	Event source (object upload triggers Lambda)
Lambda	Processes S3 event and writes to DynamoDB
DynamoDB	Stores metadata and TTL cleanup
CloudWatch Logs	Lambda execution logs
CloudWatch Alarms	Monitoring and alerting
SNS	Sends email alerts when alarms trigger
Features

Console-only setup (no CLI)

Free Tier safe

Event-driven architecture

DynamoDB TTL enabled

CloudWatch monitoring & alarms

SNS email notifications

Setup Steps (Completed)

SNS Topic

event-driven-alerts

Email subscription added

DynamoDB

Table: S3ObjectEvents

Partition key: objectId

Streams enabled (New image)

TTL enabled (24 hours)

IAM Role

Lambda-S3-DDB-Role

Policies:

AWSLambdaBasicExecutionRole

AmazonDynamoDBFullAccess

AmazonS3ReadOnlyAccess

Lambda Function

S3ToDynamoProcessor

Runtime: Python 3.12

Handler: lambda_function.lambda_handler

Writes records to DynamoDB with TTL

S3 Bucket

event-driven-lab-pauljancen

Event trigger configured (All object create events)

CloudWatch Alarms

DynamoDBWriteActivityAlert (ConsumedWriteCapacityUnits)

DynamoDBWriteThrottleAlert (WriteThrottleEvents)

Verification

Upload file to S3 bucket.

Verify DynamoDB item created.

Check CloudWatch Logs for successful Lambda execution.

Confirm CloudWatch alarm triggers and SNS email notification.

Lambda Code (Python 3.12)
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
