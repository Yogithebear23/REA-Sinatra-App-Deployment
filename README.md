# REA-Sinatra-App-Deployment
## REA Systems Engineer Practical Task

### Deployment: 

##### Prerequisites: 
1. Must have an AWS account in which deployment will be performed.
2. A user must be present in the AWS account with the required permissions/roles to be able to perform the deployment.
3. Must have AWS CLI tools installed
4. CLI session must be authenticated as the user in Step 2 before proceeding with the next steps.

Note: The user in step 2 is a user that must have the required IAM permissions to be able to create AWS resources defined in the CloudFormation Stack 

##### Assumptions:

- Assuming the AWS Account limits for resources (such as EIP, EC2 etc.) are not exceeded for resources that are getting created as part of the stack
- The Stack is getting deployed in the ap-southeast-2 region.

##### Execution Instructions:

```
# CLone git repository locally
git clone https://github.com/Yogithebear23/REA-Sinatra-App-Deployment.git

# Create CF Stack
aws --region ap-southeast-2 cloudformation create-stack --stack-name sinatra-stack --template-body file://REA-Sinatra-App-Deployment/sinatrainfra.yaml --disable-rollback

Note: Deployment is currently only supported in the ap-southeast-2 region. The application deployment will take around ~15 mins.

# Check stack status, only proceed when outcome is CREATE_COMPLETE 
aws --region ap-southeast-2 cloudformation describe-stacks --stack-name sinatra-stack --query Stacks[].StackStatus --output text

# Find out HTTP Endpoint of the ALB
aws --region ap-southeast-2 cloudformation describe-stacks --stack-name sinatra-stack --query Stacks[].Outputs[?OutputKey==`URL`].OutputValue --output text

# Manually querying the ALB public DNS name
curl http://[alb-url-retrieved-from last-step]
Sample Output (When deployment is successful): Hello World!

#Delete the stack
aws --region ap-southeast-2 cloudformation delete-stack --stack-name sinatra-stack
```
##### Solution Description:

This CF template deploys the REA Sinatra App in the ap-southeast-2 (Sydney) region. 
The template creates a multi-AZ, load balanced and auto-scalable website running on apache web servers using Ubuntu 14.04 LTS as the base OS image. The ALB is deployed in public subnets while the web servers reside in private subnets. The solution is zone independent as it takes advantage of AWS's scalable, redundant and highly available IGW and NATGWs.The web servers are configured to span all availability zones in the Sydney region and are auto-scaled based on the number of the healthy web servers.The instances are load balanced with a simple health check against the default web page. Thus, providing a highly fault tolerant architecture.The output of this template is the ALB's public DNS name which can be used to access the website over HTTP.

##### Architecture and its Benefits:
- The architecture the AWS native orchestration tool CloudFormation for consistent, repeatable and predictable deployment of the app.
- The infrastructure is scalable as it makes use of EC2 Autoscaling.
- A full system upgrade is performed during bootstrap to minimize risk of any vulnerable packages being present on the OS.
- The success of the App is proactively checked during deployment before sending signal to CF indicating the creation of ASG is successful
- The system is resilient/fault tolerant as the solution makes use of multiple AZs. There are no single point of failures within the solution (Except for if the entire region was to go down)  
- NACLs and Security Groups are restricted to only allow expected traffic both inbound & outbound.

##### Security Considerations:
- Instances are not exposed directly to public internet and are present in private subnets.
- NACLs and Security Groups are restricted to only allow expected traffic both inbound & outbound.
- No SSH access is permitted and no SSH keys are configured on the EC2 instances.

##### Potential Shortcomings:
- Hardcoded AMI, hence the solution is currently region specific (ap-southeast-2 only). The Mappings section could be used within CF template to make it deployable across multiple/different regions.
- The deployment script (user data) can be improved such that required dependencies & programs are installed at the same time. Script can be potentially improved such that it is more readable.
- AWS CloudFormation Service Role could have been used to explicitly specify the actions that AWS CloudFormation can perform following the principle of least privilege.
- Cloudwatch alarms could have used for making application servers respond to change in conditions automatically.
- SNS topics could have been used for notifying the deployer of various stages of stack creation and for proactive monitoring of the stack.
