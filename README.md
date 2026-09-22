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

