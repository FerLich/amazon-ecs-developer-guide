# Updating the Amazon ECS container agent on an Amazon ECS\-optimized AMI<a name="agent-update-ecs-ami"></a>

If you are using an Amazon ECS\-optimized AMI, you have several options to get the latest version of the Amazon ECS container agent \(shown in order of recommendation\):
+ Terminate the container instance and launch the latest version of the Amazon ECS\-optimized Amazon Linux 2 AMI \(either manually or by updating your Auto Scaling launch configuration with the latest AMI\)\. This provides a fresh container instance with the most current tested and validated versions of Amazon Linux, Docker, `ecs-init`, and the Amazon ECS container agent\. For more information, see [Amazon ECS\-optimized AMI](ecs-optimized_AMI.md)\.
+ Connect to the instance with SSH and update the `ecs-init` package \(and its dependencies\) to the latest version\. This operation provides the most current tested and validated versions of Docker and `ecs-init` that are available in the Amazon Linux repositories and the latest version of the Amazon ECS container agent\. For more information, see [To update the `ecs-init` package on an Amazon ECS\-optimized AMI](#procedure_update_ecs-init)\.
+ Update the container agent with the `UpdateContainerAgent` API operation, either through the console or with the AWS CLI or AWS SDKs\. For more information, see [Updating the Amazon ECS container agent with the `UpdateContainerAgent` API operation](#agent-update-api)\.

**Note**  
Agent updates do not apply to Windows container instances\. We recommend that you launch new container instances to update the agent version in your Windows clusters\.<a name="procedure_update_ecs-init"></a>

**To update the `ecs-init` package on an Amazon ECS\-optimized AMI**

1. Log in to your container instance via SSH\.

1. Update the `ecs-init` package with the following command\.

   ```
   sudo yum update -y ecs-init
   ```
**Note**  
The `ecs-init` package and the Amazon ECS container agent are updated immediately\. However, newer versions of Docker are not loaded until the Docker daemon is restarted\. Restart either by rebooting the instance, or by running the following commands on your instance:  
Amazon ECS\-optimized Amazon Linux 2 AMI:  

     ```
     sudo systemctl restart docker
     ```
Amazon ECS\-optimized Amazon Linux AMI:  

     ```
     sudo service docker restart && sudo start ecs
     ```

## Updating the Amazon ECS container agent with the `UpdateContainerAgent` API operation<a name="agent-update-api"></a>

**Important**  
The `UpdateContainerAgent` API is only supported on Linux variants of the Amazon ECS\-optimized AMI, with the exception of the Amazon ECS\-optimized Amazon Linux 2 \(arm64\) AMI\. For container instances using the Amazon ECS\-optimized Amazon Linux 2 \(arm64\) AMI, update the `ecs-init` package to update the agent\. For container instances that are running other operating systems, see [Manually updating the Amazon ECS container agent \(for non\-Amazon ECS\-Optimized AMIs\)](manually_update_agent.md)\. If you are using Windows container instances, we recommend that you launch new container instances to update the agent version in your Windows clusters\.

The `UpdateContainerAgent` API process begins when you request an agent update, either through the console or with the AWS CLI or AWS SDKs\. Amazon ECS checks your current agent version against the latest available agent version, and if an update is possible, the update process progresses as shown in the flow chart below\. If an update is not available, for example, if the agent is already running the most recent version, then a `NoUpdateAvailableException` is returned\.

![\[Agent update flow\]](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/images/update-flow.png)

The stages in the update process shown above are as follows:

`PENDING`  
An agent update is available, and the update process has started\.

`STAGING`  
The agent has begun downloading the agent update\. If the agent cannot download the update, or if the contents of the update are incorrect or corrupted, then the agent sends a notification of the failure and the update transitions to the `FAILED` state\.

`STAGED`  
The agent download has completed and the agent contents have been verified\.

`UPDATING`  
The `ecs-init` service is restarted and it picks up the new agent version\. If the agent is for some reason unable to restart, the update transitions to the `FAILED` state; otherwise, the agent signals Amazon ECS that the update is complete\.

**Note**  
Agent updates do not apply to Windows container instances\. We recommend that you launch new container instances to update the agent version in your Windows clusters\.

**To update the Amazon ECS container agent on an Amazon ECS\-optimized AMI in the console**

1. Open the console at [https://console\.aws\.amazon\.com/ecs/v2](https://console.aws.amazon.com/ecs/v2)\.

1. From the navigation bar, choose the Region where your external instance is registered\.

1. In the navigation pane, choose **Clusters** and select the cluster that hosts the external instance\.

1. On the **Cluster : *name*** page, choose the **Infrastructure** tab\.

1. Under **Container instances**, select the instances to update, and then choose **Update agent**\.