---
title: a first look at aws fargate
description: Fargate is an AWS service that allows you to run containers on ECS without managing servers or clusters of EC2 instances.
date: 2021-09-06
tags:
  - aws
  - fargate
  - ecs
  - docker
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/wlf9txdpajbfdtbcae5h.jpeg
layout: layouts/post.njk
---

[Fargate](https://aws.amazon.com/fargate/) is an AWS service that allows you to run containers on ECS without managing servers or clusters of EC2 instances. It manages provisioning, configuring, and scaling clusters of virtual machines to run containers.

This includes selecting the server type, deciding when to scale the clusters, and optimizing cluster packing. Running your tasks and services with the Fargate launch type includes:
* Packaging your application in containers
* Specifying the CPU and memory requirements
* Defining network and IAM policies
* Launching the application

Each Fargate task has its own isolation boundary and does not share the underlying kernel, CPU resources, memory resources, or elastic network interface with another task.

## Outline

This tutorial shows how to install and verify the [ECS CLI](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI.html), configure an [IAM role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html), set up a [cluster](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/clusters.html) with a [security group](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/get-set-up-for-amazon-ecs.html#create-a-base-security-group), and deploy a [service](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs_services.html) with [tasks](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definitions.html) using the [Fargate launch type](https://docs.aws.amazon.com/AmazonECS/latest/userguide/launch_types.html).

* [Example Container Task Definition](#example-container-task-definition)
* [Setup and Verify ECS CLI with PGP Signatures](#setup-and-verify-ecs-cli-with-pgp-signatures)
  * [Install GnuPG and Create File for ECS PGP Public Key](#install-gnupg-and-create-file-for-ecs-pgp-public-key)
  * [Import ECS PGP Public Key](#import-ecs-pgp-public-key)
  * [Download and Verify ECS CLI Signature](#download-and-verify-ecs-cli-signature)
  * [Apply Execute Permissions to CLI Binary](#apply-execute-permissions-to-cli-binary)
  * [Check CLI Version Number](#check-cli-version-number)
* [Configure AWS Credentials and IAM Role](#configure-aws-credentials-and-iam-role)
  * [Create Directory and Project Files](#create-directory-and-project-files)
  * [Create Task Execution Role](#create-task-execution-role)
  * [Attach Task Execution Role Policy](#attach-task-execution-role-policy)
  * [Configure ECS CLI](#configure-ecs-cli)
  * [Create a CLI Profile](#create-a-cli-profile)
* [Create ECS Cluster and Security Group](#create-ecs-cluster-and-security-group)
  * [Create Cluster](#create-cluster)
  * [Retrieve VPC Default Security Group ID](#retrieve-vpc-default-security-group-id)
  * [Add Security Group Rule](#add-security-group-rule)
* [Deploy Docker Container to the Cluster](#deploy-docker-container-to-the-cluster)
  * [Define Docker Compose Web Service](#define-docker-compose-web-service)
  * [ECS Parameters](#ecs-parameters)
  * [Deploy Compose File](#deploy-compose-file)
  * [View Running Containers](#view-running-containers)
  * [View Web Application](#view-web-application)
* [Clean Up and Summary](#clean-up-and-summary)
  * [Delete Your Service](#delete-your-service)
  * [Take Down Your Cluster](#take-down-your-cluster)

## Example Container Task Definition

A task definition is required to run Docker containers in Amazon ECS. Parameters you can specify include:

|Parameters|Definition|
|----------|----------|
|`image`|Docker images for each container in a task|
|`cpu`, `memory`|CPU and memory for each task or each container within a task|
|`requiresCompatibilities`|Launch type to determine the infrastructure on which tasks are hosted|
|`networkMode`|Docker networking mode for containers in a task|
|`logConfiguration`|Logging configuration for tasks|
|`command`|Command to run when the container is started|
|`volumes`|Data volumes for containers in a task|
|`executionRoleArn`|IAM role tasks should use|

This task definition sets up a web server using the Fargate launch type and an Apache `httpd:2.4` image. The container is named `sample-fargate-app` and includes log configuration and port mappings. It sets the entry point to `sh -c` and runs a shell `command` that prints an HTML document to a file called `index.html`. This is placed inside the `usr/local/apache2/htdocs/` directory.

```json
{
  "containerDefinitions": [{
    "command": [
      "/bin/sh -c \"echo '<html><head><title>ECS Sample App</title></head><body><div>
      <h1>ECS Sample App</h1><p>App running on a container in Amazon ECS.</p>
      </div></body></html>' >  /usr/local/apache2/htdocs/index.html && httpd-foreground\""
    ],
    "entryPoint": [ "sh", "-c" ],
    "essential": true,
    "image": "httpd:2.4",
    "logConfiguration": { 
      "logDriver": "awslogs",
      "options": { 
        "awslogs-group" : "/ecs/fargate-task-definition",
        "awslogs-region": "us-east-1",
        "awslogs-stream-prefix": "ecs"
      }
    },
    "name": "sample-fargate-app",
    "portMappings": [{ 
      "containerPort": 80,
      "hostPort": 80,
      "protocol": "tcp"
    }]
  }],
  "cpu": "256",
  "executionRoleArn": "arn:aws:iam::012345678910:role/ecsTaskExecutionRole",
  "family": "fargate-task-definition",
  "memory": "512",
  "networkMode": "awsvpc",
  "requiresCompatibilities": [ "FARGATE" ]
}
```

Lastly, the Apache HyperText Transfer Protocol (HTTP) server program, [`httpd`](https://httpd.apache.org/docs/2.4/programs/httpd.html), is started with `containerPort` and `hostPort` set to `80`. The [network mode](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-networking.html) is set to [`awsvpc`](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-networking-awsvpc.html) which means the task is allocated the same networking properties as Amazon EC2 instances. This includes having its own [elastic network interface (ENI)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html) and a primary private IPv4 address.

## Setup and Verify ECS CLI with PGP Signatures

Instructions for downloading the Amazon ECS CLI binary will be different for each operating system. The instructions in this article are for macOS (M1 specifically). See the AWS docs to find installation instructions for [Windows or Linux](https://docs.aws.amazon.com/AmazonECS/latest/userguide/ECS_CLI_installation.html). Run the following `curl` command for the ECS CLI install script:

```bash
sudo curl -Lo /usr/local/bin/ecs-cli \
  https://amazon-ecs-cli.s3.amazonaws.com/ecs-cli-darwin-amd64-latest
```

`sudo` is required because files located within the systems directory require root permissions.

### Install GnuPG and Create File for ECS PGP Public Key

The Amazon ECS CLI executables are cryptographically signed using PGP signatures. The PGP signatures can be used to verify the validity of the Amazon ECS CLI executable. To verify the signatures download and install [GnuPG](https://www.gnupg.org/) with [Homebrew](https://brew.sh/).

```bash
brew install gnupg
```

See [Step 2: Verify the Amazon ECS CLI using PGP signatures](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_installation.html) for the public key block.

```bash
touch aws-ecs-pgp
```

The details of the Amazon ECS PGP public key for reference:

```
Key ID: BCE9D9A42D51784F
Type: RSA
Size: 4096/4096
Expires: Never
User ID: Amazon ECS
Key fingerprint: F34C 3DDA E729 26B0 79BE AEC6 BCE9 D9A4 2D51 784F
```

### Import ECS PGP Public Key

```bash
gpg --import aws-ecs-pgp
```

### Download and Verify ECS CLI Signature

The signatures are ASCII detached PGP signatures stored in files with the extension `.asc`.

```bash
curl -Lo ecs-cli.asc \
  https://amazon-ecs-cli.s3.amazonaws.com/ecs-cli-darwin-amd64-latest.asc
```

The signatures file has the same name as its corresponding executable but with `.asc` appended.

```bash
gpg --verify ecs-cli.asc \
  /usr/local/bin/ecs-cli
```

The output will include a warning that there is no [chain of trust](https://en.wikipedia.org/wiki/Chain_of_trust) between your personal PGP key (if you have one) and the Amazon ECS PGP key.

```
Signature made Tue Apr 3 13:29:30 2018 PDT using RSA key DE3CBD61ADAF8B8E
Good signature from "Amazon ECS <ecs-security@amazon.com>" [unknown]

WARNING: This key is not certified with a trusted signature!
There is no indication that the signature belongs to the owner.

Primary key fingerprint: F34C 3DDA E729 26B0 79BE AEC6 BCE9 D9A4 2D51 784F
Subkey fingerprint: EB3D F841 E2C9 212A 2BD4 2232 DE3C BD61 ADAF 8B8E
```

### Apply Execute Permissions to CLI Binary

Running `chmod +x` followed by a file name will make that file executable.

```bash
sudo chmod +x /usr/local/bin/ecs-cli
```

### Check CLI Version Number

See the [`amazon-ecs-cli` changelog](https://github.com/aws/amazon-ecs-cli/blob/mainline/CHANGELOG.md) for the current version.

```bash
ecs-cli --version
```

Output:

```
ecs-cli version 1.21.0 (bb0b8f0)
```

## Configure AWS Credentials and IAM Role

You must [configure the ECS CLI](https://docs.aws.amazon.com/AmazonECS/latest/userguide/ECS_CLI_Configuration.html) with your AWS credentials, an AWS Region, and an Amazon ECS cluster name before you can use it.

Make sure you have the [AWS CLI](https://aws.amazon.com/cli/) installed and an [AWS account](https://aws.amazon.com/). For general use, [`aws configure`](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html) is recommended as the fastest way to set up your AWS CLI installation.

```bash
aws configure
```

Go to [My Security Credentials](https://console.aws.amazon.com/iam/home?#/security_credentials) to find your Access Key ID, Secret Access Key, and default region. You can leave the output format blank.

```
AWS Access Key ID: <YOUR_ACCESS_KEY_ID>
AWS Secret Access Key: <YOUR_SECRET_ACCESS_KEY>
Default region name: <YOUR_REGION_NAME>
Default output format [None]: 
```

The Amazon ECS container agent makes calls to AWS APIs on your behalf, so it requires an IAM policy and role for the service to know that the agent belongs to you. An IAM role is an IAM identity that you can create in your account that has specific permissions.

An IAM role is similar to an IAM user; it is an AWS identity with permission policies determining what the identity can and cannot do. But unlike an IAM user, a role is intended to be used by anyone who needs it instead of being uniquely associated with one person.

### Create Directory and Project Files

After configuring the CLI create a blank directory with three files:
* `task-execution-assume-role.json` for configuring the application's task execution IAM role.
* `docker-compose.yml` for specifying the image, running our container, and configuring logs.
* `ecs-params.yml` for ECS specific parameters.

```bash
mkdir ajcwebdev-fargate
cd ajcwebdev-fargate
touch task-execution-assume-role.json docker-compose.yml ecs-params.yml
```

### Create Task Execution Role

[`AssumeRole`](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRole.html) returns a set of temporary security credentials that you can use to access AWS resources that you might not normally have access to. Add the following contents to the `task-execution-assume-role.json` file.

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "",
    "Effect": "Allow",
    "Principal": {
      "Service": "ecs-tasks.amazonaws.com"
    },
    "Action": "sts:AssumeRole"
  }]
}
```

This will generate temporary credentials consisting of an access key ID, a secret access key, and a security token. Set the new task execution role with [`iam create-role`](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/iam/create-role.html) and the two required options:
* `--assume-role-policy-document` grants an entity permission to assume the role by setting the trust relationship policy document which is specified in the JSON string inside `task-execution-assume-role.json`.
* `--role-name` sets the name of the newly created role to `ecsTaskExecutionRole`.

```bash
aws iam create-role \
  --role-name ecsTaskExecutionRole \
  --assume-role-policy-document file://task-execution-assume-role.json
```

This role is called a [task execution IAM role](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_execution_IAM_role.html).

```json
{
  "Role": {
    "Path": "/",
    "RoleName": "ecsTaskExecutionRole",
    "RoleId": "AROARZ5VR5ZCPAE2OSEPD",
    "Arn": "arn:aws:iam::124397940292:role/ecsTaskExecutionRole",
    "CreateDate": "2021-12-29T07:40:34+00:00",
    "AssumeRolePolicyDocument": {
      "Version": "2012-10-17",
      "Statement": [{
        "Sid": "",
        "Effect": "Allow",
        "Principal": {
          "Service": "ecs-tasks.amazonaws.com"
        },
        "Action": "sts:AssumeRole"
      }]
    }
  }
}
```

The unique ID for an IAM resource is not available in the IAM console and must be obtained through the AWS CLI via the [`iam` command](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/iam/index.html#cli-aws-iam) or the [IAM API](https://docs.aws.amazon.com/IAM/latest/APIReference/welcome.html). When you assume a role it provides you with temporary security credentials for your role session. You can see this role in the [IAM console](https://console.aws.amazon.com/iamv2/home#/home).

![01-iam-role-in-console](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/lkraq7zcc6nwsc0xitp9.png)

### Attach Task Execution Role Policy

A role does not have standard long-term credentials associated with it such as a password or access keys. Instead, we will attach the role policy created in the previous section with [`iam attach-role-policy`](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/iam/attach-role-policy.html).

```bash
aws iam attach-role-policy \
  --role-name ecsTaskExecutionRole \
  --policy-arn arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy
```

`--policy-arn` uses an [AWS managed policy](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-vs-inline.html#aws-managed-policies) with its own Amazon Resource Name (ARN) that includes the policy name. It is a standalone policy that is created and administered by AWS. Back in your console you will see a policy name under the Permissions tab.

![02-role-permissions-policies](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/134m6h8w5uv6h2gy34d8.png)

Click the name of the policy to see more details.

![03-role-policy-summary](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/p7u8rtqjyq9pl00bpeyn.png)

### Configure ECS CLI

The Amazon ECS CLI requires credentials in order to make API requests on your behalf. [`ecs-cli configure`](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-configure.html) can pull credentials from environment variables, an AWS profile, or an Amazon ECS profile.

```bash
ecs-cli configure \
  --region us-west-1 \
  --cluster tutorial \
  --config-name tutorial-config \
  --default-launch-type FARGATE
```

Output:

```
Saved ECS CLI cluster configuration tutorial-config.
```

This creates a cluster configuration defining the AWS region to use, resource creation prefixes, and the cluster name to use with the ECS CLI.
* `--region` specifies `us-west-1` as the AWS Region.
* `--cluster` specifies `tutorial` as the Amazon ECS cluster name.
* `--config-name` specifies `tutorial-config` as the name of the cluster configuration which can be referenced in commands using the `--cluster-config` flag.
* `--default-launch-type` specifies `FARGATE` as the default launch type.

### Create a CLI Profile

The Amazon ECS CLI supports the configuring of multiple sets of AWS credentials as named profiles using the [`ecs-cli configure profile`](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-configure-profile.html) command. Use your AWS access key and secret key found on [My Security Credentials](https://console.aws.amazon.com/iam/home?#/security_credentials) and set `--profile-name` to `tutorial-profile`.

```bash
ecs-cli configure profile \
  --access-key AWS_ACCESS_KEY_ID \
  --secret-key AWS_SECRET_ACCESS_KEY \
  --profile-name tutorial-profile
```

Output:

```
Saved ECS CLI profile configuration tutorial-profile.
```

## Create ECS Cluster and Security Group

An Amazon ECS cluster is a logical grouping of tasks or services. Your tasks and services are run on infrastructure that is registered to a cluster.

### Create Cluster

Since the default launch type is set to Fargate in the cluster configuration, [`ecs-cli up`](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-up.html) creates an empty cluster and a VPC configured with two public subnets.

```bash
ecs-cli up \
  --cluster-config tutorial-config \
  --ecs-profile tutorial-profile
```

Output:

```
Created cluster

cluster=tutorial
region=us-west-1

VPC created: vpc-02f2b177d9e8c0f61
Subnet created: subnet-03072fb22009b2ba0
Subnet created: subnet-047889e3250dfb5b0
```

This may take a few minutes to complete as your resources are created. Once the cluster is created you can find it on the [ECS console](https://console.aws.amazon.com/ecs/).

![04-ecs-clusters](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/v4b38hisfu879f45atec.png)

Each cluster is region specific, so if you do not see your cluster in the console double check the region at the top right of the AWS console. In this example the cluster is in `us-west-1`. Click the name of the cluster to see more details.

![05-tutorial-cluster-overview](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/vhc82r4t1trk8px638sg.png)

You can find more VPC details on the [VPC console](https://console.aws.amazon.com/vpc/) under the **Your VPCs** and **Subnets** tabs.

![06-vpc-details](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/xjpvpzizxvsfbiiraf5u.png)

![07-vpc-subnets](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/azwxwiphjt7k7nwhsw4e.png)

### Retrieve VPC Default Security Group ID

[`ec2 describe-security-groups`](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ec2/describe-security-groups.html) describes the specified security groups. Filters can be used to return a more specific list of results from a describe operation. A filter name and value pair can be used to match a set of resources by specific criteria, such as tags, attributes, or IDs. Replace `VPC_ID` with the VPC ID from the previous output. This will retrieve the VPC default security group ID.

```bash
aws ec2 describe-security-groups \
  --filters Name=vpc-id,Values=VPC_ID
```

This will print out a JSON object with your `GroupId`. Save this value for future reference.

```json
{
  "SecurityGroups": [{
    "Description": "default VPC security group",
    "GroupName": "default",
    "IpPermissions": [{
      "IpProtocol": "-1",
      "UserIdGroupPairs": [{
        "GroupId": "sg-0acd34feae1cd3bb6",
        "UserId": "124397940292"
      }]
    }],
    "OwnerId": "124397940292",
    "GroupId": "sg-0acd34feae1cd3bb6",
    "IpPermissionsEgress": [{
      "IpProtocol": "-1",
      "IpRanges": [{
        "CidrIp": "0.0.0.0/0"
      }],
    }],
    "VpcId": "vpc-02f2b177d9e8c0f61"
  }]
}
```

You can also find this information under the **Security Groups** tab.

![08-vpc-security-groups](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/l5dufolnzmsjqv6hegah.png)

### Add Security Group Rule

[`ec2 authorize-security-group-ingress`](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ec2/authorize-security-group-ingress.html) adds the specified inbound (ingress) rules to a security group. Use the security group ID from the previous output to allow inbound access on port `80`.

```bash
aws ec2 authorize-security-group-ingress \
  --group-id SECURITY_GROUP_ID \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0
```

Output:

```json
{
  "Return": true,
  "SecurityGroupRules": [{
    "SecurityGroupRuleId": "sgr-07e0fc15e502634a1",
    "GroupId": "sg-0acd34feae1cd3bb6",
    "GroupOwnerId": "124397940292",
    "IsEgress": false,
    "IpProtocol": "tcp",
    "FromPort": 80,
    "ToPort": 80,
    "CidrIpv4": "0.0.0.0/0"
  }]
}
```

## Deploy Docker Container to the Cluster

[Compose](https://docs.docker.com/compose/) is a tool for defining and running multi-container Docker applications.

### Define Docker Compose Web Service

After configuring your application’s services with a YAML file, you can create and start all your services with a single command. Enter the following content into `docker-compose.yml` to create a PHP web application.

```yaml
# docker-compose.yml

version: '3'
services:
  web:
    image: amazon/amazon-ecs-sample
    ports:
      - "80:80"
    logging:
      driver: awslogs
      options: 
        awslogs-group: tutorial
        awslogs-region: us-west-1
        awslogs-stream-prefix: web
```

The `web` container exposes port `80` for inbound traffic to the web server. It also configures container logs to go to the CloudWatch log group created earlier.

### ECS Parameters

In addition to the Compose information, there are some parameters specific to ECS that you must specify for the service. Enter the following content into `ecs-params.yml` with the subnet and security group IDs from the previous steps.

```yaml
# ecs-params.yml

version: 1
task_definition:
  task_execution_role: ecsTaskExecutionRole
  ecs_network_mode: awsvpc
  task_size:
    mem_limit: 0.5GB
    cpu_limit: 256
run_params:
  network_configuration:
    awsvpc_configuration:
      subnets:
        - "subnet ID 1"
        - "subnet ID 2"
      security_groups:
        - "security group ID"
      assign_public_ip: ENABLED
```

### Deploy Compose File

After you create the compose file, you can deploy it to your cluster with [`ecs-cli compose service up`](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service-up.html). The command looks for files called `docker-compose.yml` and `ecs-params.yml` in the current directory.

```bash
ecs-cli compose --project-name tutorial service up \
  --create-log-groups \
  --cluster-config tutorial-config \
  --ecs-profile tutorial-profile
```

Output:

```
Using ECS task definition
  TaskDefinition="tutorial:1"

Created Log Group tutorial in us-west-1 
Auto-enabling ECS Managed Tags 

(service tutorial) has started 1 tasks:
(task c8d20b796bb7450799dfb8887331a930). 
  timestamp="2021-12-29 08:19:37 +0000 UTC"

Service status
  desiredCount=1
  runningCount=1
  serviceName=tutorial

ECS Service has reached a stable state
  desiredCount=1
  runningCount=1
  serviceName=tutorial

Created an ECS service
  service=tutorial
  taskDefinition="tutorial:1"
```

By default, the resources created by this command have the current directory in their titles, but you can override that with the `--project-name` option. The `--create-log-groups` option creates the CloudWatch log groups for the container logs.

### View Running Containers 

After you deploy the compose file, you can view the containers that are running in the service by returning to the ECS console.

![09-task-definition-status](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/f93y7gabp6x307j0yrfv.png)

Click the network tab to see your public IP address.

![10-task-definition-network](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hsr6yul3z91offqyqj0s.png)

Alternatively, you can run [`ecs-cli compose service ps`](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service-ps.html) to view the running containers.

```bash
ecs-cli compose --project-name tutorial service ps \
  --cluster-config tutorial-config \
  --ecs-profile tutorial-profile
```

Output:

```
Name: tutorial/c8d20b796bb7450799dfb8887331a930/web
State: RUNNING
Ports: 54.241.222.168:80->80/tcp
TaskDefinition: tutorial:1
Health: UNKNOWN
```

### View Web Application

Open the IP address in your browser to see a PHP web application.

![11-deployed-php-application](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/75pv4dkvf4v2und8aj80.png)

## Clean Up and Summary

You may have heard or read that Fargate is a service that provides *[Serverless compute for containers.](https://aws.amazon.com/fargate/)* You may also have heard that *[Serverless is the way of the future thanks to scaling to zero.](https://aws.amazon.com/blogs/publicsector/scaling-zero-serverless-way-future-university-of-york/)* You would then be surprised to learn that [Fargate does not scale to zero](https://github.com/aws/containers-roadmap/issues/1017) except with [significant additional configuration with other services](https://serverfault.com/questions/951429/aws-fargate-service-scale-to-zero).

### Delete Your Service

Since Fargate does not scale to zero, you should clean up your resources when you are done with this tutorial. This will ensure the resources will not incur any more charges. Stop the running container with [`ecs-cli compose service down`](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service-rm.html).

```bash
ecs-cli compose --project-name tutorial service down \
  --cluster-config tutorial-config \
  --ecs-profile tutorial-profile
```

### Take Down Your Cluster

Use [`ecs-cli down`](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-down.html) to clean up the resources created earlier with `ecs-cli up`.

```bash
ecs-cli down \
  --cluster-config tutorial-config \
  --force \
  --ecs-profile tutorial-profile
```

Wow that took a long time. Should have just used [Fly](https://fly.io/) or [Qovery](https://www.qovery.com/).