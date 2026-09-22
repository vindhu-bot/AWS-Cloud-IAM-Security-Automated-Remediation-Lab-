# AWS Cloud IAM Security & Automated Remediation Lab 

## Project Overview 
This project demonstrates how AWS controls can be used to discover excessive permissions, enforce least privilege, discover insecure cloud configurations, and automatically remediate incorrect security configurations.   

This lab specifically focuses on two major components of cloud security: 

- **Identity and Access Security:** Identify an IAM user who has been given overprivileged access, test the users permissions, and finally replace the excessive access with a least-privilege IAM policy.
- **Configuration Security:** Monitor an Amazon S3 bucket using AWS Config, purposefully introduce public access configuration, detect that insecure config, and then automatically restore the security controls using AWS Systems Manager Automation.

### Workflow of Lab 
