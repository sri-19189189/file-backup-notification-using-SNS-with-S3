# File-backup-notification-using-SNS-with-S3
Automating AWS backup event alerts with Amazon S3 and Amazon SNS. Receiving email notifications in real-time when a file is uploaded to an S3 bucket. Event-based monitoring. Testing is done with AWS CLI.

# PROBLEM – 
The issue is that organizations wanted to retain operational reliability while guaranteeing commitment to data privacy laws on real-time file transfer and backup processes. Lacking automated alerts, IT teams frequently finds missing files or unsuccessful backups hours or days later, posing a risk of data loss and violating regulations. Maintaining consistent backup verification procedures is challenging since manual S3 bucket monitoring is time-consuming and prone to human mistake.

# SOLUTION – 
A programmed notification system using Amazon S3 event notifications integrated/combined with Amazon SNS (Simple Notification Service) to send instant email alerts whenever files are uploaded to your backup bucket. This cloud-based solution ensures instant awareness of successful file transfers and backup completions by providing real-time monitoring of backup activities without the need for manual inspection or custom monitoring scripts.

# Architecture Diagram
<img src="https://github.com/sri-19189189/file-backup-notification-using-SNS-with-S3/blob/main/Architecture_S3%26SNS.png" alt="architecture">

* Amazon S3: Stores backup files in a bucket.
* S3 Event Notification: Triggers when a new object is created.
* Amazon SNS: Sends notifications through email.
* End Users: Receive email alerts about new backups.
* Purpose: Automates backup notifications and keeps users informed.

Skills demonstrated \n
Amazon S3 · Amazon SNS · AWS CLI · Event-Driven Architecture · Cloud Storage · Serverless Notifications · IAM Resource Policies

#

Environment Preparing
These following codes are run and successfully completed in aws cloud shell and the comments will be followed by # sign also I tried to explain what each command does and what happens with services in background while we play in cloud shell.
aws configure get region
#gives the region where you are working.
aws sts get-caller-identity \--query Account --output text
#The command get-caller-identity gone give basic account details and then query account will be able to give account number as text output.
aws secretsmanager get-random-password \--exclude-punctuation \--password-length 8 --require-each-included-type \--output text --query RandomPassword   - let name it as “ Pass”
#Generates random password and to create names of Resources uniquely. Without query you will get in JSON format.
echo backup-notifications-$(Pass)
#To create unique resource names for S3 buckets need to be globally unique.
echo backup-alerts-$(Pass)
#To create SNS Name.
echo "domain@yourmail.com"
#email for confirmations.

STEPS
1.	Create SNS Topic for Email Notifications:
aws sns create-topic \--name SNS Name\--query 'TopicArn' --output text
 
2.	Subscribe Email Address to SNS Topic:
aws sns subscribe \--topic-arn arn:aws:sns:us-east-1:193461099169:backup-alerts-ddm3gg52 \--protocol email \--notification-endpoint sg885*6n@pace.edu
 
3.	Create S3 Bucket for File Storage:
aws s3 mb s3://backup-notifications-4uefn52b  --region us-east-1

aws s3api put-bucket-versioning \--bucket backup-notifications-4uefn52b \--versioning-configuration Status=Enabled

aws s3api put-bucket-encryption --bucket backup-notifications-4uefn52b --server-side-encryption-configuration '{"Rules":[{"ApplyServerSideEncryptionByDefault":{"SSEAlgorithm":"AES256"}}]}'

 

4.	Configure SNS Topic Policy for S3 Access:
cat > sns-policy.json << EOF
{
"Version": "2012-10-17",
"Statement": [
{
"Sid": "AllowS3ToPublish",
"Effect": "Allow",
"Principal": {
"Service": "s3.amazonaws.com"
},
"Action": "SNS:Publish",
"Resource": "${SNS_TOPIC_ARN}",
"Condition": {
"StringEquals": {
"aws:SourceAccount": "${AWS_ACCOUNT_ID}"
},
"StringLike": {
"aws:SourceArn": "arn:aws:s3:::${BUCKET_NAME}"
}
}
}
]
}

aws sns set-topic-attributes \--topic-arn arn:aws:sns:us-east-1:193461099169:backup-alerts-ddm3gg52 \--attribute-name Policy \--attribute-value file://sns-policy.json
 




5.	Configure S3 Event Notifications:
cat > notification-config.json
{
"TopicConfigurations": [
{
"Id": "BackupNotification",
"TopicArn": "arn:aws:sns:us-east-1:193461099169:backup-alerts-ddm3gg52",
"Events": [
"s3:ObjectCreated:*"
]
}
]
}

aws s3api put-bucket-notification-configuration \--bucket backup-notifications-4uefn52b \--notification-configuration file://notification-config.json
 

 


6.	Test files creation for validation:
echo Backup test file created at %Date% %Time% > test-backup-1.txt
echo Database backup completed at %Date% %Time% > test-backup-2.txt
echo Application logs archived at %Date% %Time%  > test-backup-3.txt












Validation & Testing
1.	Upload Test Files to S3 to Trigger Notifications:
aws s3 cp test-backup-1.txt s3://${BUCKET_NAME}/
aws s3 cp test-backup-2.txt s3://${BUCKET_NAME}/backups/
aws s3 cp test-backup-3.txt s3://${BUCKET_NAME}/logs/

Expected result: Will receive 3 email notifications within minutes for each uploaded file.



2.	Verify S3 Objects and Notifications:
# List objects in bucket
aws s3 ls s3://${BUCKET_NAME}/ --recursive

# Check SNS topic metrics
aws cloudwatch get-metric-statistics \
    --namespace AWS/SNS \
    --metric-name NumberOfMessagesPublished \
    --dimensions Name=TopicName,Value=${SNS_TOPIC_NAME} \
    --start-time $(date -u -d '10 minutes ago' \
        '+%Y-%m-%dT%H:%M:%S' 2>/dev/null || \
        date -u -v-10M '+%Y-%m-%dT%H:%M:%S') \
    --end-time $(date -u '+%Y-%m-%dT%H:%M:%S') \
    --period 300 \
    --statistics Sum
Output: Objects should be listed in the bucket, and CloudWatch metrics should show message publications.

3.	Test Notification Content Structure:
aws s3 cp validation-test.txt s3://${BUCKET_NAME}/validation/
echo "Check your email for notification containing:"
echo "- Bucket name: ${BUCKET_NAME}"
echo "- Object key: validation/validation-test.txt"
echo "- Event type: ObjectCreated:Put"

	

Cleaning the environment & Services
1.	Remove Test Files from S3 Bucket:
aws s3 rm s3://${BUCKET_NAME} --recursive

2.	Delete S3 Bucket and Configuration:
aws s3api put-bucket-notification-configuration \
    --bucket ${BUCKET_NAME} \
    --notification-configuration '{}'

aws s3 rb s3://${BUCKET_NAME}
3.	Remove SNS Topic and Subscription:
aws sns delete-topic --topic-arn ${SNS_TOPIC_ARN}
4.	Clean Up Local Files:
rm -f test-backup-*.txt validation-test.txt
rm -f sns-policy.json notification-config.json
unset BUCKET_NAME SNS_TOPIC_NAME EMAIL_ADDRESS SNS_TOPIC_ARN RANDOM_SUFFIX

