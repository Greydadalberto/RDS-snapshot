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
|  RDS Instance       +-----------> | RDS Manual Snapshot +-----------> | RDS Snapshot (eu-central-1) |
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

Copy snapshot to eu-central-1.

Check CloudWatch Logs for detailed output.

6️ Verify Snapshot in Destination Region
Switch AWS Console to eu-central-1 region.

Go to RDS → Snapshots.

Confirm new snapshot exists and status is Available.


Conclusion
- Manual Snapshot creation verified.
- Lambda function created & tested for automation.
- Snapshot successfully copied cross-region.



Connect to the Replicated RDS Instance using MySQL

After the snapshot was restored into a new RDS instance in the Frankfurt region (eu-central-1), follow these steps to connect to it using the MySQL client.

Step 1: Ensure MySQL Client is Installed
Check if MySQL is installed:
mysql --version

If it’s not installed, install it using your OS's package manager:

Ubuntu/Debian:
sudo apt update
sudo apt install mysql-client

Step 2: Confirm Your Database Endpoint
From the AWS Console → RDS → Databases → your restored DB (eshop-replica), copy the endpoint and port. Example:
pgsql
Endpoint: eshop-replica.cn60o6mcyatg.eu-central-1.rds.amazonaws.com  
Port: 3306

Step 3: Open Database Port in the Security Group
Ensure the inbound rules of the RDS security group allow traffic on port 3306 from your IP or 0.0.0.0/0 if for testing:
Type:       MYSQL/Aurora  
Protocol:   TCP  
Port:       3306  
Source:     Your IP / 0.0.0.0/0 (not recommended for production)

Step 4: Connect to the Database
Use the admin username (or any other DB user you configured):
mysql -h eshop-replica.cn60o6mcyatg.eu-central-1.rds.amazonaws.com -P 3306 -u admin -p
Enter the password when prompted. If forgotten, reset it in the RDS Console under Modify → Master password.


Step 5: Run Test Queries
Once connected, verify the database contents:
sql
SHOW DATABASES;
USE your_database_name;
SHOW TABLES;
SELECT * FROM some_table LIMIT 5;

confirm Verification of the replicated database
