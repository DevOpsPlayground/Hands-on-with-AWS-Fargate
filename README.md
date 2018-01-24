# DevOps Playground #17: All Hands on AWS Fargate

<img src='assets/aws_fargate.png' width="100%">

## Introduction

During this Playground we are going to use the AWS CLI to create an ECS cluster, launching a service using the Fargate Launch Type. We will understand it's functionality and it's place amongst the AWS Cloud Services.

**Name:** Enzo Rivello  
**Role:** DevOps And Continuous Delivery Consultant  
**Email:** enzo@ecs-digital.co.uk  
**Linkedin:** https://www.linkedin.com/in/enzoriv/  
**Github:** https://github.com/enzor
**Twitter:** https://twitter.com/enzorivelllo

## Requirements

1. SSH Client (Terminal or Putty)
2. Paper with neccesary variables provided by ECS Digital:
 - IP Address
 - Security Group ID
 - Public Subnet ID


## Setup
1. Open your SSH Client
2. Connect to your provisional instance:
 - `ssh ec2-user@Provided-Ip-Address`
3. You are ready to go!

## Deployment
1. To ease the deployment 
 - 
 ```
 cluster_name=<name>-<surname>-cluster
 subnet_id=provided-subnet-id
 ghost_sg_id=provided-sg-id
 ```

2. Create your cluster: 
 - `aws ecs create-cluster --cluster-name ${cluster_name}`
3. Create your blog service
 - 
 ```
    aws ecs create-service --cluster ${cluster_name} \
     --service-name farghost-1 --task-definition my-blog:1 \
     --desired-count 1 --launch-type "FARGATE" \
     --network-configuration "awsvpcConfiguration={subnets=[${subnet_id}],securityGroups=[${ghost_sg_id}],assignPublicIp=ENABLED}"
     
 ```
   
4. Inspect your service
 - `aws ecs describe-services --cluster ${cluster_name} --service farghost-1 `
5. Retrieve the task-id of the service
 - `aws ecs describe-services --cluster ${cluster_name} --service farghost-1  | grep task`
6. Inspect your service task
 - `aws ecs describe-tasks --cluster ${cluster_name} --tasks YOUR_TASK_ID `
7. Retrieve Service IP Address
 - `aws ecs describe-tasks --cluster ${cluster_name} --tasks YOUR_TASK_ID  | grep eni`
 - `aws ec2 describe-network-interfaces --network-interface-ids YOUR_ENI  | grep PublicIp`
8. See your blog: open http://YOUR_IP_ADDRESS:2368 and enjoy!

## Cleanup
1. Set up the desired count for your blog service to 0
 - `aws ecs update-service --cluster ${cluster_name} --service farghost-1 --desired-count 0 `
2. Delete the farghost-1 service
 - `aws ecs delete-service --cluster ${cluster_name} --service farghost-1 `
3. Delete the ecs cluster
 - `aws ecs delete-cluster --cluster ${cluster_name} `
