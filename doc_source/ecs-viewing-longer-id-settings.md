# Viewing account settings<a name="ecs-viewing-longer-id-settings"></a>

You can use the AWS Management Console and AWS CLI tools to view your account settings\.

**Important**  
The `dualStackIPv6` account setting can only be viewed or changed using the AWS CLI\.

**To view your account settings \(Console\)**

1. Open the Amazon ECS console at [https://console\.aws\.amazon\.com/ecs/](https://console.aws.amazon.com/ecs/)\.

1. In the navigation bar at the top, select the Region to view your account settings for\. 

1. From the dashboard, choose **Account Settings**\.

1. On the **Amazon ECS ARN and resource ID settings**, **AWSVPC Trunking**, and **CloudWatch Container Insights** sections, you can view your opt\-in status for each account setting for the authenticated IAM user and role\.

**To view your account settings \(AWS CLI\)**
+ [list\-account\-settings](https://docs.aws.amazon.com/cli/latest/reference/ecs/list-account-settings.html) \(AWS CLI\)

  ```
  aws ecs list-account-settings --effective-settings --region us-east-1
  ```
+ [Get\-ECSAccountSetting](https://docs.aws.amazon.com/powershell/latest/reference/items/Get-ECSAccountSetting.html) \(AWS Tools for Windows PowerShell\)

  ```
  Get-ECSAccountSetting -EffectiveSetting true -Region us-east-1
  ```

**To view the account settings for a specific IAM user or IAM role \(AWS CLI\)**
+ [list\-account\-settings](https://docs.aws.amazon.com/cli/latest/reference/ecs/list-account-settings.html) \(AWS CLI\)

  ```
  aws ecs list-account-settings --principal-arn arn:aws:iam::aws_account_id:user/principalName --effective-settings --region us-east-1
  ```
+ [Get\-ECSAccountSetting](https://docs.aws.amazon.com/powershell/latest/reference/items/Get-ECSAccountSetting.html) \(AWS Tools for Windows PowerShell\)

  ```
  Get-ECSAccountSetting -PrincipalArn arn:aws:iam::aws_account_id:user/principalName -EffectiveSetting true -Region us-east-1
  ```