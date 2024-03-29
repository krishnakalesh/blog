+++
title = 'Uploading Docker Image from WSL 2 to AWS ECR 🚢'
date = 2024-03-12T16:23:39+05:30
draft = false
+++
# Uploading Docker Image from WSL 2 to AWS ECR 🚢

In this post, we'll walk through the process of uploading a Docker image from Windows Subsystem for Linux 2 (WSL 2) to Amazon Elastic Container Registry (ECR) on AWS. Docker has become a cornerstone technology for containerizing applications, and AWS ECR provides a managed container registry service for storing, managing, and deploying Docker container images.
<!--more-->
## Prerequisites
Before we begin, ensure that you have the following prerequisites:

1. An AWS account
2. Docker installed on WSL 2 - [Guide](https://krishnakalesh.github.io/pinchofcode/post/wsl2_docker_windows/)
3. An existing Dockerized project to create a docker image to upload

## Install AWS CLI on WSL 2
Refer AWS documentation - https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html
The AWS CLI uses glibc, groff, and less. These are included by default in most major distributions of Linux.

```shell
sudo apt update -y && sudo apt upgrade -y
sudo apt install glibc-source groff less unzip -y
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
aws --version
```
## Connect to AWS ECR from WSL 2
If you are freshly installing AWS CLI, it will ask for configuring AWS
Use below command for configuration
```shell
aws configure
```
Provide AWS Access Key ID, AWS Secret Key and region details when prompted

## Push image to AWS ECR from WSL 2
Create a repository in AWS ECR and when you click 'View Push Commands', you will 
get commands that need to be used to push to ECR.

![Alt text](/pinchofcode/images/ecr1.png)
```shell
aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin AWSACCTNUM.dkr.ecr.REGION.amazonaws.com
docker build -t xyz .
docker tag xyz:latest AWSACCTNUM.dkr.ecr.REGION.amazonaws.com/xyz:latest
docker push AWSACCTNUM.dkr.ecr.REGION.amazonaws.com/xyz:latest
```

## Verify in AWS Console

Once image has been pushed, we can go to AWS ECR console and verify.
![Alt text](/pinchofcode/images/ecr2.png)

This process enables you to seamlessly deploy containerized applications on AWS infrastructure using Docker containers managed by ECR.