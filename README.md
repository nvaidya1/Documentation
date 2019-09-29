# CloudWatch agent best practices

CloudWatch is an essential monitoring service offered on AWS Console to track the events and health of various resources. In order to monitor the health of compute resources like EC2 OR OnPrem instances, the CloudWatch agent would need to be installed and configured. The latest [CloudWatch agent](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Install-CloudWatch-Agent.html) offers unified functionality of exporting both the logs and metrics from the compute resources to the CloudWatch service. 

This document will guide the user through the series of steps to install CloudWatch agent on EC2 and OnPrem compute instances running Windows OR Linux Operating System and review the exported logs and metrics on CloudWatch.

## Pre-Requisites

It is recommended to install AWS Systems Manager agent on the compute instances and ensure the instances are visible under the Systems Manager Registered Instances. For more information on procedure to install AWS Systems Manager agent, please follow this link: [About AWS Systems Manager Agent](https://docs.aws.amazon.com/systems-manager/latest/userguide/prereqs-ssm-agent.html)

## Quick Overview of Installation Steps:

1. Create IAM Role for the EC2 to access CloudWatch Logs and AWS Systems Manager (SSM) System Parameter Store
2. Attach the IAM Role to the EC2 instance(s)
3. Donwload and install CloudWatch Agent using AWS Systems Manager OR manually via EC2 command line
4. Create configuration file for the CloudWatch agent and store it in SSM Parameter Store
5. Start CloudWatch agent using AWS Systems Manager OR manually via EC2 command line

## Step By Step Installation of CloudWatch Agent on EC2 and OnPrem Instances:



