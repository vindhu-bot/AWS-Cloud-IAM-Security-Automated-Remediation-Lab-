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


