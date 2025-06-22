RDS Snapshot Replication Automation (Cross-Region)
This project automates the creation and cross-region copying of Amazon RDS manual snapshots using an AWS Lambda function written in Python (Boto3).

Project Overview
This project performs the following:

Checks the status of an existing RDS instance (eshop).

If status = "available", the Lambda:

Creates a manual DB Snapshot.

Waits for the snapshot to become "available".

Copies the snapshot to another AWS region (us-east-1).

Safe handling: If DB status is not "available", the Lambda aborts snapshot creation.


Architecture
AWS Service	---Usage
RDS	--->Source MySQL database (eshop)
Lambda	---> Automation function for snapshot creation & copy
IAM Role ---> 	Permissions to allow Lambda to manage RDS snapshots
CloudWatch Logs ---> 	To log Lambda activity

RDS Snapshot Automation Architecture (Simple Diagram)
+---------------------+             +---------------------+             +--------------------+
|                     |  Snapshot   |                     |  Copy       |                    |
|  RDS Instance       +-----------> | RDS Manual Snapshot +-----------> | RDS Snapshot (us-east-1) |
|  (eshop, eu-west-1) |             |  (eshop-snapshot)   |             | (Copied Snapshot)   |
+---------------------+             +---------------------+             +--------------------+
           |                                                                           |
           |                      +-----------------------+                            |
           |                      | Lambda Function (Python) |<-------------------------+
           |                      |  - Check RDS Status      |
           |                      |  - Create Snapshot      |
           |                      |  - Wait & Copy Snapshot |
           |                      +-----------------------+     
           |
+---------------------+
|                     |
| CloudWatch Logs     |
| (Logs Lambda Events)|
+---------------------+




Setup Instructions
 Prerequisites
AWS Account with:

RDS Instance (e.g., MySQL eshop).

Permissions for RDS, IAM, Lambda, KMS.

AWS CLI (Optional for manual verification).

Python 3.12+ (for local testing, optional).


Manual RDS Snapshot (for Verification)
Go to AWS Console → RDS → Databases.

Select your RDS instance (eshop).

Click Actions → Take Snapshot.

Name it (e.g., eshop-snapshot) and confirm.

Verify snapshot in Snapshots tab (status: Available).



Lambda Function Creation
Go to AWS Lambda → Create Function.

Select Author from Scratch.

Runtime: Python 3.12.

Attach an IAM Role with the following permissions:
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "rds:DescribeDBInstances",
        "rds:DescribeDBSnapshots",
        "rds:CreateDBSnapshot",
        "rds:CopyDBSnapshot"
      ],
      "Resource": "*"
    }
  ]
}


Test the Lambda Function
Create a test event in Lambda (empty JSON {}).

Click Test.

If RDS instance = available, Lambda will:

Create snapshot.

Copy snapshot to us-east-1.

Check CloudWatch Logs for detailed output.

6️ Verify Snapshot in Destination Region
Switch AWS Console to us-east-1 region.

Go to RDS → Snapshots.

Confirm new snapshot exists and status is Available.


Conclusion
- Manual Snapshot creation verified.
- Lambda function created & tested for automation.
- Snapshot successfully copied cross-region.






