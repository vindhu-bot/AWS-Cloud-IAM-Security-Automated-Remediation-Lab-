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


## Testing the Overprivileged IAM User

Instead if relying on just the policy configuration, I instead tested the permissions to verify what the IAM user could actually do.

I created a disposable object named `delete-test.txt` alongside the original `test.txt`. This let me test destructive permissions without risking the actual test object.

<img width="391" height="393" alt="S8" src="https://github.com/user-attachments/assets/cf810cf4-219a-4422-bfe1-1b4334dab1b2" />

^ Created a disposable delete-test.txt object alongside the original test.txt object to safely test whether the overprivileged IAM identity can perform unauthorized delete operations. 

### Verifying CLI Identity

The AWS CLI was configured to authenticate as `cloud-security-user`. Before testing S3 permissions, I used AWS STS to verify that the CLI session was operating as the intended IAM identity.

<img width="368" height="354" alt="S9" src="https://github.com/user-attachments/assets/6f2759a7-c9cc-47bf-87ba-317196983a90" />

This ensured that the following permission tests represented the permissions of `cloud-security-user`, rather than another AWS identity.

### Testing S3 Access

I first tested whether the user could list objects in `cloud-iam-lab`.

<img width="367" height="164" alt="S10" src="https://github.com/user-attachments/assets/9460bae2-ef6b-45ef-a8d6-2f072280bc5b" />

^ The request succeeded, confirming that the IAM user could access and list objects in the bucket.

Next, I attempted to delete the disposable `delete-test.txt` object.

<img width="409" height="116" alt="S11" src="https://github.com/user-attachments/assets/7ab461f4-4ab5-4b5f-a4de-0afb3b72d35e" />

The deletion worked 

**What was found:** The `cloud-security-user` was had access to destructive S3 permissions that were unnecessary for its intended read-only role.

This demonstrated the risk of assigning broad policies without limiting the users permissions to the actions and resources actually required.

## Implementing Least Privilege

To remediate the excessive permissions, I created a customer-managed IAM policy named: `Cloud-IAM-Lab-Read-ListOnly`

Instead of allowing all S3 actions, the new policy only allowed the operations needed by the test user. These consisted of:

- `s3:ListBucket` — allows the user to list objects in `cloud-iam-lab`
- `s3:GetObject` — allows the user to read objects stored in the bucket

The policy was also scoped specifically to the `cloud-iam-lab` bucket and its objects.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ListLabBucket",
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::cloud-iam-lab"
    },
    {
      "Sid": "ReadLabObjects",
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::cloud-iam-lab/*"
    }
  ]
}
```

<img width="392" height="305" alt="S11" src="https://github.com/user-attachments/assets/a76d7f44-e7c6-4209-b80b-ee301e5ac57e" />

<img width="394" height="299" alt="S12" src="https://github.com/user-attachments/assets/2fb50358-c0d9-4841-9eb2-9871184621ff" />

<img width="389" height="380" alt="S13" src="https://github.com/user-attachments/assets/460f8e61-2b50-42a6-b35b-ccedc25fd343" />

### The new policy was created and then attached to `cloud-security-user`.

<img width="376" height="404" alt="S14" src="https://github.com/user-attachments/assets/73444516-2e51-4d24-bc4b-2c8eadef5711" />

<img width="377" height="398" alt="image" src="https://github.com/user-attachments/assets/00241049-a825-4bf3-877f-a2c932768ad3" />

<img width="377" height="398" alt="S16" src="https://github.com/user-attachments/assets/128fce0b-2a8d-4fcb-90a1-c30956894fb9" />



The original `AmazonS3FullAccess` policy was removed so that the user would only work with the new restricted permissions.

### Before vs. After

Before:

- List objects: ✅ Allowed
- Read objects: ✅ Allowed
- Delete objects: ✅ Allowed
- Broad S3 access: ✅ Allowed

After replacing the policy with the custom read-only policy:

After:

- List objects: ✅ Allowed
- Read objects: ✅ Allowed
- Delete objects: ❌ Denied
- Broad S3 access: ❌ Removed

The next step was to ensure that the new policy still allowed legitimate operations while preventing unnecessary actions.


























