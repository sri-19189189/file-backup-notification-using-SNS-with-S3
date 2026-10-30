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


