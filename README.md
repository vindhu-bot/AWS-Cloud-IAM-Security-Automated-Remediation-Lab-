# AWS Cloud IAM Security & Automated Remediation Lab 

## Project Overview 
This project demonstrates how AWS controls can be used to discover excessive permissions, enforce least privilege, discover insecure cloud configurations, and automatically remediate incorrect security configurations.   

This lab specifically focuses on two major components of cloud security: 

- **Identity and Access Security:** Identify an IAM user who has been given overprivileged access, test the users permissions, and finally replace the excessive access with a least-privilege IAM policy.
- **Configuration Security:** Monitor an Amazon S3 bucket using AWS Config, purposefully introduce public access configuration, detect that insecure config, and then automatically restore the security controls using AWS Systems Manager Automation.

### Workflow of Lab 
<img width="482" height="389" alt="visual" src="https://github.com/user-attachments/assets/c5c7824c-0ac7-47f3-a9cf-91e866f548f0" />

## Technologies Used 
- **AWS Identity and Access Management(IAM)** --> Managed identities and permissions
- **Amazon S3** --> Cloud storage resource used for security testing purposes
- **AWS Config** --> Monitored S3 configuration and compliance
- **AWS System Manager Automation** --> Performed the automatic remediation
- **PowerShell** --> Command line used for AWS CLI testing, and test permissions. 

## Develop a Secure S3 Baseline 
To start the lab I created an Amazon S3 bucket named 'cloud-iam-lab'. This bucket essentially was the protected cloud resource which was used for IAM permission testing and configuration monitoring.

Upon creation I made sure the bucket was configured with **ACLs disabled** and **Block Public Access enabled** --> This basically developed a secure starting config before actually introducing any controlled security issues. 

### Secure S3 Configuration 
<img width="389" height="361" alt="S1" src="https://github.com/user-attachments/assets/149075a4-ad86-4379-af85-159cea6e3d38" /> <img width="380" height="281" alt="S2" src="https://github.com/user-attachments/assets/4cca6e54-f28b-43be-a6d3-a5ee232fcdcd" />

The initial security configuration consisted of: 
- **ACL's disabled**
- **Public Access Blocked**
- **No public bucket policy**
- **No sensitive data**

This was a good starting point/baseline which could later be compared to insecure configurations(intentional for lab purpose). 

### Create a test object 

I uploaded a dummy text file "text.txt" to the bucket. The file contained non-sensitive writing so it could be used to test the S3 permissions. 


<img width="377" height="279" alt="S3" src="https://github.com/user-attachments/assets/dffff05b-73f8-45d3-809f-1fb8d776fe48" /> <img width="377" height="320" alt="S4" src="https://github.com/user-attachments/assets/f85702e4-a1e8-4821-ab99-4383048d8039" />

The test environment was now: 
```text
cloud-iam-lab
│
└── test.txt
```
This object will let me safely test whether a specific IAM identity would be able to list, read, or delete objects from the bucket.

### Baseline Configuration 

Before beginning IAM testing, I verified that **Block Public Access remained enabled** and that the bucket had **no public bucket policy**.

<img width="397" height="373" alt="S5" src="https://github.com/user-attachments/assets/2e1b8750-4189-4474-9246-c8981901975d" />

**Result:** The S3 bucket was established in a secure state and was ready for controlled IAM security testing.

## Simulating an Overprivileged IAM Identity

With the S3 environment established, I created a test IAM user named `cloud-security-user`.

To simulate a common cloud security problem, I intentionally assigned the AWS-managed `AmazonS3FullAccess` policy to the user.

<img width="358" height="341" alt="S6" src="https://github.com/user-attachments/assets/64692689-fb94-4294-aba7-e48aed78c255" />

^ Overprivileged IAM Configuration: Created a test IAM identity with the AWS-managed AmazonS3FullAccess policy to simulate excessive cloud permissions before applying least privilege. 

### Why is this a security problem?

`AmazonS3FullAccess` provides broad permissions to Amazon S3. This means the test identity receives significantly more access than would be necessary for a user whose job only requires viewing data.

For example, an overprivileged identity may be able to:

- List objects
- Read objects
- Upload/modify objects
- Delete objects
- Perform other S3 administrative operations

^ **This essentially violated the **principle of least privilege** which states that an identity should only receive the permissions needed to perform required tasks.  

So instead of just assuming the permissions were excessive, the next step was to **test the IAM identity through the AWS CLI and verify what actions it could actually perform** 

## Identifying Excessive IAM Permissions

After creating the `cloud-security-user`, I reviewed the AWS-managed `AmazonS3FullAccess` policy that had been intentionally assigned to the user.

The policy contained the following permissions:

```json
{
  "Effect": "Allow",
  "Action": [
    "s3:*",
    "s3-object-lambda:*"
  ],
  "Resource": "*"
}
```
<img width="384" height="425" alt="S7" src="https://github.com/user-attachments/assets/a7c7b9d0-22da-4b1e-bd11-b4d7b1d23ef5" />

1. **`s3:*`*** --> wildcard allows all Amazon S3 actions
2. **`Resource: "*"`** --> applies those actions broadly to S3 resources supported by the actions

It's important to note, for this lab, the user only needed to **list and read objects from one specific S3 bucket**. Therefore, `AmazonS3FullAccess` provided significantly more access than necessary.
   
   ^ This created the overprivileged condition that I wanted to test and remediate.


