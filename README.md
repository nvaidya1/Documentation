# CloudWatch agent best practices

CloudWatch is an essential monitoring service offered on AWS Console to track the events and health of various resources. In order to monitor the health of compute resources like EC2 OR OnPrem instances, the CloudWatch agent would need to be installed and configured. The latest [CloudWatch agent](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Install-CloudWatch-Agent.html) offers unified functionality of exporting both the logs and metrics from the compute resources to the CloudWatch service.

This document will guide the user through the series of steps to install CloudWatch agent on EC2 and OnPrem compute instances running Windows OR Linux Operating System and review the exported logs and metrics on CloudWatch.
Though there are multiple methods to install CloudWatch agent (using SSM / manually via command line / using AWS CloudFormation), we will follow the installation using SSM.

## Pre-Requisites

A running EC2 instance:
It is recommended to install AWS Systems Manager agent on the desired compute instances and associate an IAM role with **AmazonEC2RoleforSSM** policy.Once configured, please ensure the instances are visible under the Systems Manager Registered Instances. For more information on procedure to install AWS Systems Manager agent, please follow this link: [About AWS Systems Manager Agent](https://docs.aws.amazon.com/systems-manager/latest/userguide/prereqs-ssm-agent.html)

Ensure the SSM Agent is running and with version 2.2.93.0 or later. Example of EC2 running Amazon Linux2 AMI:

Linux Command(s):
```console
sudo systemctl status amazon-ssm-agent
rpm -qa amazon-ssm-agent
```

List of instances registered under System Manager, Managed Instances:
<kbd>![Managed Instances listing under System Manager](images/ManagedInstancesList.png)</kbd>

Registered OnPrem Instance:
<kbd>![Onprem Instance under Managed Instances](images/OnpremInstance.png)</kbd>

## Quick Overview of Installation Steps:

1. Create IAM Role for the EC2 to access CloudWatch Logs and AWS Systems Manager (SSM) System Parameter Store
2. Attach the IAM Role to the EC2 instance(s)
3. Download and install CloudWatch Agent using AWS Systems Manager
4. Create configuration file for the CloudWatch agent and store it in SSM Parameter Store
5. Start CloudWatch agent using AWS Systems Manager
6. Access CloudWatch console and monitor the Logs and Metrics

## Step By Step Installation of CloudWatch Agent on EC2 and OnPrem Instances:

### Step 1. Create IAM Role for EC2 to access CloudWatch and AWS SSM Parameter Store:

Create IAM role with name CloudWatchAgentServerRole (or another name that you prefer) for EC2 and select following policies:
  * **CloudWatchAgentServerPolicy** for access to CloudWatch service
  * **AmazonSSMManagedInstanceCore** for access to AWS Systems Manager
  * **CloudWatchAgentAdminPolicy** for access to Parameter Store

Select the IAM service, Click on Roles on the Left side, Click Create Role, Make policies selection as shown below:
<kbd>![IAM Policy Selection](images/IAMRolePolicies.png)</kbd>

Give the Role name and create role:
<kbd>![IAM Role Name](images/IAMRoleName.png)</kbd>





### Step 2. Attach the IAM Role to the EC2 instance(s):

Attach the IAM role created above to the desired EC2 instance:
  * Open the Amazon EC2 console and select the instance, choose Actions, Instance Settings, Attach/Replace IAM role
  * Select the IAM role (CloudWatchAgentServerRole) to attach to your instance, and choose Apply

Selecting instance and Attach IAM Role:
<kbd>![Attach IAM Role](images/AttachIAMRole.png)</kbd>

Selecting the Role Name created above:
<kbd>![Select IAM Role](images/SelectingRole.png)</kbd>





### Step 3. Download and install CloudWatch Agent using AWS Systems Manager

Open AWS Systems Manager Console and follow the below steps:
  * Select Managed Instances, then on right side, Select Action dropdown list & select **Run Command**

<kbd>![Run Command Selection](images/RunCommandSelect.png)</kbd>

  * In the Command document list, choose **AWS-ConfigureAWSPackage** (applicable for Windows / Linux)

<kbd>![Run Command Document Package Selection](images/RunCommandDocumentSelectPackage.png)</kbd>

  * Scroll Down to select **Install** as Action, provide a name **AmazonCloudWatchAgent**
  * Select **latest** for the Version and desired instances under Targets by selecting **Choose instances manually**
  * Choose an existing S3 bucket to store the output and Click **Run**
  * Check the command status by clicking button next to Instance name and select **View output**. Systems Manager should show that the agent was successfully installed.

Run Command issued and in progress:
<kbd>![Run Command Output2](images/RunCmd-InProgress.png)</kbd>

Run Command Status showing success and summarized output:
<kbd>![Run Command Output3](images/Windows-RunCmd-OP.png)</kbd>

Run Command Output to check the complete output:
<kbd>![Run Command Output](images/RunCommandOutput.png)</kbd>





### Step 4. Create configuration file for the CloudWatch agent and store it in SSM Parameter Store

Login into the desired EC2 Linux instance to create a configuration file via Command Line by running config-wizard :

Linux Command(s):
```console
sudo systemctl status amazon-ssm-agent
cd /opt/aws/amazon-cloudwatch-agent/bin
ls amazon-cloudwatch-agent-config-wizard
sudo ./amazon-cloudwatch-agent-config-wizard
```

Running Configuration Wizard on Linux Instance:
<kbd>![CloudWatchAgent Configuration Wizard](images/CWAgentConfig.png)</kbd>

Running Configuration Wizard on Windows Instance in Powershell:
<kbd>![CloudWatchAgent Configuration Wizard2](images/Windows-CWAgent-Config1.png)</kbd>

The json file created from the config-wizard is stored at: /opt/aws/amazon-cloudwatch-agent/bin/config.json
On running the config-wizard, answer the questions asked to configure the exporting of desired metrics and logs. At the end, save this configuration on the Systems Parameter Store named as AmazonCloudWatch-linux.

Completion of Configuration Wizard output:
<kbd>![CloudWatchAgent Configuration Wizard3](images/Windows-CWAgent-Config2.png)</kbd>

Later login to the SSM Parameter Store and check the new entry created and the desired configuration file in JSON format.

<kbd>![System Parameter Store Entry](images/SystemParameterStore.png)</kbd>





### Step 5. Start CloudWatch agent using AWS Systems Manager

As per the steps listed in Step 3 above, start the CloudWatch agent using SSM Run Command by selecting **AmazonCloudWatch-ManageAgent** configuration document and configuration location to be the Parameter Store entry name along with a Bucket Name to store the command output.

<kbd>![Run Command Document Selection](images/RunCommandDocumentSelect.png)</kbd>

Lastly, check the status of the CloudWatch agent service using command:

Linux Command(s):
```console
sudo systemctl status amamzon-cloudwatch-agent
```
Windows Command(s):
<kbd>![CloudWatchAgent Status on Windows](images/Windows-ServiceStatus.png)</kbd>





### Step 6. Access CloudWatch console and monitor the Logs and Metrics

Access the CloudWatch console and check the Metrics:

Select the "CWAgent" from the Metrics Window as shown below:
<kbd>![CloudWatch Metrics](images/Metrics-CWAgent.png)</kbd>

Browse various metric and select them to be displayed:
<kbd>![CloudWatch Metrics2](images/Metrics-DisplayWindowsVM.png)</kbd>

Access the CloudWatch console and check the Logs:

<kbd>![CloudWatch Logs](images/Logs-LinuxVM.png)</kbd>





## Cleanup

On the completion of the lab, please feel free to stop the CloudWatch agent on the EC2 instances to avoid any additional charges. Additionally, may terminate the EC2 instances that are launched specifically for this lab.
