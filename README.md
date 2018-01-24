# DevOps Playground #17: All Hands on AWS Fargate

<img src='assets/aws_fargate.png' width="100%">

## Introduction

During this Playground we are going to use the AWS CLI to create an ECS cluster and launch a service using the Fargate Launch Type. We will understand it's functionality and it's place amongst the AWS Cloud Services.

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
2. Connect to your cloud instance:
   - `ssh ec2-user@Provided-Ip-Address`
3. When prompted, use the password displayed on the big screen
4. You are ready to go!

## Deployment
1. To simplify the deployment, we will start by defining some variables that we are going to use later
 ```
 cluster_name=<name>-<surname>-cluster
 subnet_id=<provided-subnet-id>
 ghost_sg_id=<provided-sg-id>
 ```

2. The first step in deploying a new service to Elastic Container Service is to define a Cluster.
   - `aws ecs create-cluster --cluster-name ${cluster_name}`

3. (Optional) Once you have your cluster, you need to upload a task definition. This is the description of your image. A sample definition of a service can be found [here](tasks/blog-task-definition.json). The command to upload the definition is: 
   - `aws ecs register-task-definition --cli-input-json file://$PWD/tasks/blog-task-definition.json` 
   
   For your convenience we have already uploded one, so this step is not necessary.

4. Once the definition has been uploaded, is time to create the Service, using the following command.
 ```
    aws ecs create-service --cluster ${cluster_name} \
     --service-name farghost-1 --task-definition my-blog:1 \
     --desired-count 1 --launch-type "FARGATE" \
     --network-configuration "awsvpcConfiguration={subnets=[${subnet_id}],securityGroups=[${ghost_sg_id}],assignPublicIp=ENABLED}"
     
 ```  
 This action creates both a Service and a Task.  
   - Task is the activity that downloads the image, creates or destroys a container 
   - Service is the entity that keeps the image running in the desired state or scales it and spawn Tasks.
 For external access, we can attach Service entity to a Service Load Balancer, so that the load will be evenly distributed amongst our services.

5. We can retrieve up-to-date information about our service by using the `describe-services` command in the CLI
   - `aws ecs describe-services --cluster ${cluster_name} --service farghost-1 `

## Retrieve Public IP
At this point, the only thing we need is the IP address of the AWS Fargate instance.  
The AWS CLI doesn't provide this information directly through the `describe-services` or `describe-tasks` command.
To find it, we need to do some digging through the AWS Networking System.

Do you remember before, in the definition, when we defined `awsvpc`?  
`awsvpc` is a new network mode launched from AWS last year, and it permits any new container to be attached to an ENI - Elastic Network Interface. By discovering and inspecting the ENI we can retrieve the Public IP address of the container.  

In order to do this, we first...  

1. Retrieve the Task ID from our cluster definition:
   - `aws ecs describe-services --cluster ${cluster_name} --service farghost-1  | grep task`

2. Inspect your task using the `describe-tasks` command in the CLI
   - `aws ecs describe-tasks --cluster ${cluster_name} --tasks YOUR_TASK_ID `

3. Retrieve the ENI ID from the task definition:
   - `aws ecs describe-tasks --cluster ${cluster_name} --tasks YOUR_TASK_ID  | grep eni`

4. Retrieve the PublicIp by using the `describe-network-interfaces` command from the `aws ec2` cli
   - `aws ec2 describe-network-interfaces --network-interface-ids YOUR_ENI  | grep PublicIp`

5. See your blog: open http://YOUR_IP_ADDRESS:2368 and enjoy!

## Cleanup

1. Set up the desired count for your blog service to 0
   - `aws ecs update-service --cluster ${cluster_name} --service farghost-1 --desired-count 0 `
   
2. Delete the farghost-1 service
   - `aws ecs delete-service --cluster ${cluster_name} --service farghost-1 `
   
3. Delete the ecs cluster
   - `aws ecs delete-cluster --cluster ${cluster_name} `
