```markdown
# AWS Session Manager for Private EC2 Access :closed_lock_with_key:

This guide explains how to securely access **private EC2 instances** (no public IP) using **AWS Systems Manager (SSM) Session Manager**, eliminating the need for bastion hosts or SSH keys.

## :wrench: Prerequisites
- **EC2 Instance** in a private subnet
- **SSM Agent** installed (pre-installed on Amazon Linux 2/2023, Ubuntu 16.04+, Windows Server 2016+)
- **IAM permissions** to create roles and VPC endpoints

## :arrow_forward: Quick Start

### 1. IAM Role Setup
```bash
aws iam create-role --role-name SSM-Session-Manager-Role --assume-role-policy-document '{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {"Service": "ec2.amazonaws.com"},
    "Action": "sts:AssumeRole"
  }]
}'

aws iam attach-role-policy --role-name SSM-Session-Manager-Role --policy-arn arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore
```

### 2. Attach Role to EC2 Instance
```bash
aws ec2 associate-iam-instance-profile \
  --instance-id <YOUR_INSTANCE_ID> \
  --iam-instance-profile Name=SSM-Session-Manager-Role
```

### 3. VPC Endpoints (For No-Internet Networks)
Create these **Interface Endpoints** in your VPC:
- `com.amazonaws.<region>.ssm`
- `com.amazonaws.<region>.ec2messages`
- `com.amazonaws.<region>.ssmmessages`

## :computer: Connection Methods

### Web Console
1. Navigate to **AWS Systems Manager → Session Manager**
2. Click **Start Session** and select your instance

### AWS CLI
```bash
aws ssm start-session --target <instance-id>
```

### SSH Over Session Manager (Advanced)
Add to `~/.ssh/config`:
```config
Host i-*
  ProxyCommand sh -c "aws ssm start-session --target %h --document-name AWS-StartSSHSession --parameters 'portNumber=%p'"
```
Then connect normally:
```bash
ssh ec2-user@i-0123456789abcdef0
```

## :triangular_flag_on_post: Troubleshooting
| Issue | Solution |
|-------|----------|
| SSM Agent not running | `sudo systemctl restart amazon-ssm-agent` |
| Permission errors | Verify IAM role has `AmazonSSMManagedInstanceCore` |
| Connection timeout | Check VPC endpoints and security groups |

## :lock: Security Best Practices
- Enable **CloudTrail logging** for session auditing
- Restrict access using **IAM policies**
- Rotate **IAM credentials** regularly

## :books: Documentation
- [AWS Session Manager Docs](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html)
- [VPC Endpoint Pricing](https://aws.amazon.com/privatelink/pricing/)

---

> :bulb: **Pro Tip**: Combine with **SCP policies** for organization-wide security controls!

```

This `README.md` provides:
1. Clear prerequisites and quick setup commands
2. Multiple connection methods (Web, CLI, SSH)
3. Troubleshooting table for common issues
4. Security recommendations
5. Official documentation links

The formatting uses GitHub-Flavored Markdown with emojis for better readability. You can copy this directly into a `README.md` file in your project.
