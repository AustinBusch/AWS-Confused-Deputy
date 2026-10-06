# AWS Confused Deputy

The AWS Confused Deputy is a security concern that is common in many environments. The Confused Deputy attack is a privilege escalation attack that can allow unauthorized access into your environment. In this article I will go in depth on what the confused deputy attack is, how it can occur, and how to prevent it. 

## What is the Confused Deputy Problem?
The AWS Confused Deputy Problem is a security vulnerability that occurs when an attacker can trick an AWS service into performing an action on their behalf, even though they don't have the necessary permissions. This can happen when the attacker is able to forge a request that appears to come from a trusted source, such as an IAM role or user. Trusted sources can be vendors utilizing AWS, AWS Services or other AWS principals. This attack can lead to unauthorized access to resources, data exfiltration, and other malicious activities.

## How does it work? 
![Confused Deputy with AWS Services - Vulnerable](https://github.com/AustinBusch/AWS-Confused-Deputy/blob/main/assets/confused%20deputy%20vulnerability-vulnerable.gif)

In the diagram above, I utilize the AWS VPC Flow Logs service to demonstrate the confused deputy problem. The attack works as follows:
1. A developer creates a role that trusts the VPC Flow Logs service to write logs to a Cloud Watch Log group in their account. 
2. With some reconnaissance, the attacker discovers this role.
3. The attacker creates their own VPC Flow Logs and configures it to use the role created by the developer.
4. The attacker can now send flow logs from their account to the Cloud Watch Log group in the developer's account.

## How to prevent it?
![Confused Deputy with AWS Services - Secure](https://github.com/AustinBusch/AWS-Confused-Deputy/blob/main/assets/confused%20deputy%20vulnerability-secure.gif)
To prevent the confused deputy problem with AWS services, you should use resource-based policies that include the `aws:SourceAccount` and `aws:SourceArn` conditions. This will ensure that only the trusted services in the specified account and resource can perform the action. In the example above, the trust policy would include these conditions to prevent the attack. 

## How about Vendor Services? 
![Confused Deputy with Vendor - Attack Path](https://github.com/AustinBusch/AWS-Confused-Deputy/blob/main/assets/confused%20deputy%20vulnerability-Attack%20Path.gif)

The confused deputy problem can also occur with vendor services. In this case, the vendor service is the confused deputy. The attack works as follows: 
1. The attacker signs up as a customer of the vendor. They get their own legitimate tenant in the vendor's platform.
2. The attacker registers the developer's role ARN as their own integration. The ARN isn't secret, and the vendor's service accepts it without verifying who owns it.
3. The vendor's service assumes the developer's role. The vendor's backend calls sts:AssumeRole on that ARN. Because the developer's trust policy has no ExternalId condition, it sees only the trusted vendor-service-role calling and allows it.
4. The vendor accesses the developer's S3 buckets and returns the data to the attacker. The vendor's service reads the data using the role's permissions and shows it in the attacker's tenant.

## How to prevent it?
To prevent the confused deputy problem with vendor services, you should ensure that the vendor service requires an ExternalID condition in the trust policy. This will ensure that only the trusted vendor service can assume the role. 

Thank you for reading! 
Please leave any feedback in the comments below. 
## Reference Sources

- [Confused Deputy Problem](https://docs.aws.amazon.com/IAM/latest/UserGuide/confused-deputy.html) - AWS documentation on the AWS Confused Deputy problem. 
- [VPC Flow Logs](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs-iam-role.html) - AWS documentation on creating secure VPC Flow Log roles. 
