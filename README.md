# High Availability Deployment with CloudFormation


## Problem description

Your company is creating an Instagram clone called Udagram. Developers pushed the latest version of their code in a zip file located in a public S3 Bucket.

You have been tasked with deploying the application, along with the necessary supporting software into its matching infrastructure.

This needs to be done in an automated fashion so that the infrastructure can be discarded as soon as the testing team finishes their tests and gathers their results.


## Server Specs

You'll need to create a Launch Configuration for your application servers in order to deploy four servers, two located in each of your private subnets. The launch configuration will be used by an auto-scaling group.

You'll need two vCPUs and at least 4GB of RAM. The Operating System to be used is Ubuntu 18. So, choose an Instance size and Machine Image (AMI) that best fits this spec. Be sure to allocate at least 10GB of disk space so that you don't run into issues.


## Security Groups and Roles

Since you will be downloading the application archive from an S3 Bucket, you'll need to create an IAM Role that allows your instances to use the S3 Service.

Application communicates on the default HTTP Port: 80, so your servers will need this inbound port open since you will use it with the Load Balancer and the Load Balancer Health Check. As for outbound, the servers will need unrestricted internet access to be able to download and update its software.

The load balancer should allow all public traffic (0.0.0.0/0) on port 80 inbound, which is the default HTTP port. Outbound, it will only be using port 80 to reach the internal servers.

The application needs to be deployed into private subnets with a Load Balancer located in a public subnet.

One of the output exports of the CloudFormation script should be the public URL of the LoadBalancer.



# Solution

An example solution to this approach is shown in diagram below

![Deployment](/assets/HA-Template.png)


To deploy an example application use CloudFormation templates in `/templates` folder.
Minimal requirements are to create:
- s3-stack
- network-stack
- bastion-stack
- auto-scaling-server-stack
Extend with:
- bastion-stack
- cloud-front-stack

# Bastion Host

To have Bastion Host enabled you need to run `bastion-stack` before creating `server-stack`.
This will populate private bucket with ssh key pair and bastion host would be able to communicate with servers in Private Subnets.

You can put keys in corresponding bucket manually, or after Bastion stack is succeeded.
Just ssh into bastion host machine and issue copy commands.

After completing previous step, all machines from an auto scaling group will be able during runtime to access the private key.


# Programmatic access

Another option is to create necessary stacks from console:
```
./create.sh bucket-stack ./template/s3-stack.yaml ./template/s3-params.json

./create.sh network-stack ./template/HA-network-stack.yaml ./template/network-params.json

./create.sh server-stack ./template/HA-server-stack.yaml ./template/server-params.json

./create.sh bastion-stack ./template/bastion-stack.yaml ./template/bastion-params.json

./create.sh cloud-front-stack ./template/cloud-front-stack.yaml ./template/cf-params.json

```
