# Serverless-Architecture

This repository contains solutions for four AWS Lambda assignments. Each assignment leverages AWS services and Python with Boto3 to automate cloud operations. Below are the details for each assignment, including the purpose, steps, and how to invoke the Lambda functions.

---

## Assignment 1: Automated Instance Management Using AWS Lambda and Boto3

### Objective
Automate the stopping and starting of EC2 instances based on tags.

### Steps
1. **Set Up EC2 Instances**
    - Launch two EC2 instances.
    - Assign the following tags:
        - Instance 1: Key = `Action`, Value = `Auto-Stop`.
        - Instance 2: Key = `Action`, Value = `Auto-Start`.

2. **Create IAM Role for Lambda**
    - Navigate to the IAM dashboard.
    - Create a new role with the `AmazonEC2FullAccess` policy.

3. **Write Lambda Function**
    - Create a Lambda function with Python 3.x runtime.
    - Assign the IAM role to the Lambda function.
    - Use the following code:

```python
import boto3

def lambda_handler(event, context):
    ec2 = boto3.client('ec2')
    instances = ec2.describe_instances(Filters=[{'Name': 'tag:Action', 'Values': ['Auto-Stop', 'Auto-Start']}])
    
    for reservation in instances['Reservations']:
        for instance in reservation['Instances']:
            instance_id = instance['InstanceId']
            action = next(tag['Value'] for tag in instance['Tags'] if tag['Key'] == 'Action')
            
            if action == 'Auto-Stop':
                ec2.stop_instances(InstanceIds=[instance_id])
                print(f"Stopped instance {instance_id}")
            elif action == 'Auto-Start':
                ec2.start_instances(InstanceIds=[instance_id])
                print(f"Started instance {instance_id}")
```

4. **Invoke the Function**
    - Use the **Test** feature in the AWS Lambda Console or manually invoke it using the AWS CLI:

    ```bash
    aws lambda invoke --function-name <Function-Name> response.json
    ```

---

## Assignment 2: Automated S3 Bucket Cleanup Using AWS Lambda and Boto3

### Objective
Automate the deletion of files older than 30 days in an S3 bucket.

### Steps
1. **Set Up S3 Bucket**
    - Create an S3 bucket and upload test files.

2. **Create IAM Role for Lambda**
    - Create a role with the `AmazonS3FullAccess` policy.

3. **Write Lambda Function**
    - Create a Lambda function with Python 3.x runtime.
    - Use the following code:

```python
import boto3
from datetime import datetime, timezone, timedelta

s3 = boto3.client('s3')
BUCKET_NAME = '<Your-Bucket-Name>'


def lambda_handler(event, context):
    objects = s3.list_objects_v2(Bucket=BUCKET_NAME)
    if 'Contents' in objects:
        for obj in objects['Contents']:
            obj_date = obj['LastModified']
            age = datetime.now(timezone.utc) - obj_date
            if age > timedelta(days=30):
                s3.delete_object(Bucket=BUCKET_NAME, Key=obj['Key'])
                print(f"Deleted {obj['Key']}")
```

4. **Invoke the Function**
    - Use the **Test** feature or manually invoke:

    ```bash
    aws lambda invoke --function-name <Function-Name> response.json
    ```

---

## Assignment 3: Monitor Unencrypted S3 Buckets Using AWS Lambda and Boto3

### Objective
Detect S3 buckets without server-side encryption.

### Steps
1. **Set Up S3 Buckets**
    - Create S3 buckets, with some unencrypted.

2. **Create IAM Role for Lambda**
    - Attach the `AmazonS3ReadOnlyAccess` policy.

3. **Write Lambda Function**
    - Use the following code:

```python
import boto3

s3 = boto3.client('s3')

def lambda_handler(event, context):
    buckets = s3.list_buckets()['Buckets']
    for bucket in buckets:
        encryption = None
        try:
            encryption = s3.get_bucket_encryption(Bucket=bucket['Name'])
        except Exception as e:
            print(f"Bucket {bucket['Name']} is not encrypted.")
```

4. **Invoke the Function**
    - Use the **Test** feature or invoke it:

    ```bash
    aws lambda invoke --function-name <Function-Name> response.json
    ```

---

## Assignment 4: Automatic EBS Snapshot and Cleanup Using AWS Lambda and Boto3

### Objective
Automate snapshot creation and cleanup for EBS volumes.

### Steps
1. **Set Up EBS Volume**
    - Identify or create an EBS volume.

2. **Create IAM Role for Lambda**
    - Attach the `AmazonEC2FullAccess` policy.

3. **Write Lambda Function**
    - Use the following code:

```python
import boto3
from datetime import datetime, timezone, timedelta


def lambda_handler(event, context):
    ec2 = boto3.client('ec2')
    volumes_ID = ec2.describe_volumes() # Replace with your EBS volume ID
    retention_days = 30

    for volume in volumes['Volumes']:
        snapshot = ec2.create_snapshot(VolumeId=volume['VolumeId'], Description='Automated Backup')
        print(f"Created snapshot {snapshot['SnapshotId']} for volume {volume['VolumeId']}")

        snapshots = ec2.describe_snapshots(OwnerIds=['self'], Filters=[{'Name': 'volume-id', 'Values': [volume['VolumeId']]}])
        for snap in snapshots['Snapshots']:
            age = datetime.now(timezone.utc) - snap['StartTime']
            if age > timedelta(days=retention_days):
                ec2.delete_snapshot(SnapshotId=snap['SnapshotId'])
                print(f"Deleted snapshot {snap['SnapshotId']} due to retention policy.")
```

4. **Invoke the Function**
    - Use the **Test** feature or invoke it:

    ```bash
    aws lambda invoke --function-name <Function-Name> response.json
    ```

---

## Conclusion
Each assignment demonstrates the power of AWS Lambda and Boto3 for cloud automation. Follow the steps, customize the functions as needed, and ensure IAM roles are appropriately configured for secure operations.

