# Amazon Elastic Container Service on AWS Outposts<a name="ecs-on-outposts"></a>

AWS Outposts enables native AWS services, infrastructure, and operating models in on\-premises facilities\. In AWS Outposts environments, you can use the same AWS APIs, tools, and infrastructure that you use in the AWS Cloud\. Amazon ECS on AWS Outposts is ideal for low\-latency workloads that need to be run in close proximity to on\-premises data and applications\. For more information about AWS Outposts, see the [AWS Outposts User Guide](https://docs.aws.amazon.com/outposts/latest/userguide/)\.

## Prerequisites<a name="ecs-outposts-prereq"></a>

 The following are the prerequisites for using Amazon ECS on AWS Outposts:
+ You must have installed and configured an AWS Outposts in your on\-premises data center\.
+ You must have a reliable network connection between your AWS Outposts and its AWS Region\.
+ You must have sufficient capacity of instance types available in your AWS Outposts\.
+ All Amazon ECS container instances must have Amazon ECS container agent 1\.33\.0 or later\.

## Limitations<a name="ecs-outposts-limit"></a>

The following are the limitations of using Amazon ECS on AWS Outposts:
+ Amazon Elastic Container Registry, AWS Identity and Access Management, Network Load Balancer, and Amazon Route 53 run in the AWS Region, not on AWS Outposts\. This will increase latencies between these services and the containers\.
+ AWS Fargate is not available on AWS Outposts\.

## Network Connectivity Considerations<a name="ecs-outposts-considerations"></a>

The following are network connectivity considerations for AWS Outposts:
+ If network connectivity between your AWS Outposts and its AWS Region is lost, your clusters will continue to run\. However, you cannot create new clusters or take new actions on existing clusters until connectivity is restored\. In case of instance failures, the instance will not be automatically replaced\. The CloudWatch Logs agent will be unable to update logs and event data\.
+ We recommend that you provide reliable, highly available, and low latency connectivity between your AWS Outposts and its AWS Region\.

## Creating an Amazon ECS Cluster on an AWS Outposts<a name="ecs-outposts-create"></a>

Creating an Amazon ECS cluster on an AWS Outposts is similar to creating an Amazon ECS cluster in the AWS Cloud\. When you create an Amazon ECS cluster on an AWS Outposts, you must specify a subnet associated with your AWS Outposts\.

An AWS Outposts is an extension of an AWS Region, and you can extend an Amazon VPC in an account to span multiple Availability Zones and any associated AWS Outposts\. When you configure your AWS Outposts, you associate a subnet with it to extend your Regional VPC environment to your on\-premises facility\. Instances on an AWS Outposts appear as part of your Regional VPC, similar to an Availability Zone with associated subnets\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/images/network-components.png)

 **AWS CLI** 

 To create an Amazon ECS cluster on an AWS Outposts with the AWS CLI, specify a security group and a subnet to associate with your AWS Outposts\.

To create a subnet associated with your AWS Outposts\.

```
aws ec2 create-subnet \
    --cidr-block 10.0.3.0/24 \
    --vpc-id vpc-xxxxxxxx \
    --outpost-arn arn:aws:outposts:us-west-2:123456789012:outpost/op-xxxxxxxxxxxxxxxx \
    --availability-zone-id usw2-az1
```

The following example creates an Amazon ECS cluster on an AWS Outposts\.

1. Create a role and policy with rights on AWS Outposts\.

   The `role-policy.json` file is the policy document that contains the effect and actions for resources\. For information about the file format, see [PutRolePolicy](https://docs.aws.amazon.com/IAM/latest/APIReference/API_PutRolePolicy.html) in the *IAM API Reference*

   ```
   aws iam create-role –-role-name ecsRole \
       --assume-role-policy-document file://ecs-policy.json
   aws iam put-role-policy --role-name ecsRole --policy-name ecsRolePolicy \
       --policy-document file://role-policy.json
   ```

1. Create an IAM instance profile with rights on AWS Outposts\.

   ```
   aws iam create-instance-profile --instance-profile-name outpost
   aws iam add-role-to-instance-profile --instance-profile-name outpost \
       --role-name ecsRole
   ```

1. Create a VPC\.

   ```
   aws ec2 create-vpc --cidr-block 10.0.0.0/16
   ```

1. Create a security group for the container instances, specifying the proper CIDR range for the AWS Outposts\. \(This step is different for AWS Outposts\.\)

   ```
   aws ec2 create-security-group --group-name MyOutpostSG
   aws ec2 authorize-security-group-ingress --group-name MyOutpostSG --protocol tcp \
       --port 22 --cidr 10.0.3.0/24
   aws ec2 authorize-security-group-ingress --group-name MyOutpostSG --protocol tcp \
       --port 80 --cidr 10.0.3.0/24
   ```

1. Create the Cluster\.

1. Define the Amazon ECS container agent environment variables to launch the instance into the cluster created in the previous step and define any tags you want to add to help identify the cluster \(for example, `Outpost` to indicate that the cluster is for an Outpost\)\.

   ```
   #! /bin/bash
   cat << ‘EOF’ >> /etc/ecs/ecs.config
   ECS_CLUSTER=MyCluster
   ECS_IMAGE_PULL_BEHAVIOR=prefer-cached
   ECS_CONTAINER_INSTANCE_TAGS={“environment”: ”Outpost”}
   EOF
   ```
**Note**  
In order to avoid delays caused by pulling container images from Amazon ECR in the Region, use image caches\. To do this, each time a task is run, configure the Amazon ECS agent to default to using the cached image on the instance itself by setting `ECS_IMAGE_PULL_BEHAVIOR` to `prefer-cached`\. 

1. Create the container instance, specifying the VPC and subnet for the AWS Outposts where this instance should run and an instance type that is available on the AWS Outposts\. \(This step is different for AWS Outposts\.\)

   The `userdata.txt` file contains the user data the instance can use to perform common automated configuration tasks and even run scripts after the instance starts\. For information about the file for API calls, see [Run commands on your Linux instance at launch](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html) in the *Amazon EC2 User Guide for Linux Instances*\.

   ```
   aws ec2 run-instances --count 1 --image-id ami-xxxxxxxx --instance-type c5.large \
       --key-name aws-outpost-key –-subnet-id subnet-xxxxxxxxxxxxxxxxx \
       --iam-instance-profile Name outpost --security-group-id sg-xxxxxx \
       --associate-public-ip-address --user-data file://userdata.txt
   ```
**Note**  
This command is also used when adding additional instances to the cluster\. Any containers deployed in the cluster will be placed on that specific AWS Outposts\.

1. Register your task definition\. Use the following command and substitute `ecs-task.json` with the name of your task definition\.

   ```
   aws ecs register-task-definition --cli-input-json file://ecs-task.json
   ```

1. Run the task or create the service\.

------
#### [ Run the task ]

   ```
   aws ecs run-task --cluster mycluster --count 1 --task-definition outpost-app:1
   ```

------
#### [ Create the service ]

   ```
   aws ecs create-service –-cluster mycluster --service-name outpost-service \
       --task-definition outpost-app:1 --desired-count 1
   ```

------