# aws-deploy

## Description

Automated procedure for deploying and updating [instant-search-demo](https://github.com/algolia/instant-search-demo). 

## Architecture

The application has been containerized to simplify its deployment and maintenance.

The deployment is based on AWS services, both on deployment and execution times. CloudFormation templates are being used for automated deployment, and the ECS service will host the application.

### Source

All the code required is stored in this repository.
- **docker** directory for containerization related files
- **cfn** directory for CloudFormation templates

![sources](/instant-search-demo.sources.png)

Docker images are pushed to [DockerHub](https://hub.docker.com/repository/docker/albocanegra/instant-search-demo), where the deployment process will retrieve them.

### Stack architecture

The application is a Web application, which has been deployed as an ECS Service. An Application Load balancer has been configured to receive requests from the clients and send them to the ECS tasks for processing.

![architecture](/instant-search-demo.arch.png)


### Versions

Three different versions have been defined to test the deployment process. Each one corresponds to a different branch in the GitHub Repo and a different tag in DockerHub.

##### 1.0

Branch: RELEASE1.0
DockerHub image tag: 1.0

This version is the working initial state. 

##### 2.0

Branch: RELEASE2.0
DockerHub image tag: 2.0

This version points to a broken container that will not be able to reach the running state.

##### 3.0

Branch: RELEASE3.0
DockerHub image tag: 3.0

This version is another working state. 

## Usage

### Prerequisites

Since everything is automated to deploy on AWS, there are a few requisites that should be met:
- An AWS account.
- VPC and two public subnets in that AWS account.
- Credentials belonging to a user with administrative privileges.
- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) installed and configured with said credentials and default region.

### Deployment

The first step is to configure the parameters in cfn/stack-conf.json:
- **VPC**
- **SubnetId1**
- **SubnetId2**

The CloudFormation stack will be created using the template included in the **cfn** folder using this command:
```
aws cloudformation create-stack --stack-name instant-search-demo --capabilities CAPABILITY_NAMED_IAM --template-body file://stack.yaml --parameters file://stack-conf.json
```

The state of the deployment can be monitored in the AWS console, or using this command:
```
aws cloudformation describe-stacks --stack-name instant-search-demo
```

That command will also provide the stack outputs, which include the Application Load Balancer DNS Name (something similar to <lb-name>.<region>.elb.amazonaws.com). The application will be publicly accessible using HTTP with that hostname, so it can be used in any web browser.

### Update

In order to ensure that any update is secure and to be able to preview any change before it is executed, CloudFormation ChangeSets will be used in various steps.

In order to facilitate the testing of the update process, three different branches have been provided, as stated in the **Versions** section.

##### Create ChangeSet

The changeset will be created using the following command:
```
aws cloudformation create-change-set --stack-name instant-search-demo --change-set-name instant-search-demo-update --capabilities CAPABILITY_NAMED_IAM --template-body file://stack.yaml --parameters file://stack-conf.json
```

Creating a ChangeSet will evaluate and validate any change to the template, but it will not be applied until it is executed in the following steps. 

##### Review ChangeSet

In order to review which changes would be applied if the ChangeSet is executed, they can be obtained either in the AWS console or through this command:
```
aws cloudformation describe-change-set --stack-name instant-search-demo --change-set-name instant-search-demo-update
```

##### Execute ChangeSet

Once validated the changes in the ChangeSet, they can be executed using:
```
aws cloudformation execute-change-set --stack-name instant-search-demo --change-set-name instant-search-demo-update --capabilities CAPABILITY_NAMED_IAM --template-body file://stack.yaml --parameters file://stack-conf.json
```

In the case that a failure would occur during the udpate process, CloudFormation would RollBack automatically to the previous working state.

##### Monitor stack deployment

The state of the stack update can be monitored in the AWS console, or using this command:
```
aws cloudformation delete-stack --stack-name instant-search-demo
```

### Delete

The stack can be deleted using this command:
```
aws cloudformation delete-stack --stack-name instant-search-demo
```

## Next steps

This is a very basic approach to an automated deployment, but there are some improvements that are highly recommended before using them in a real environment:
- Using HTTPS on the load balancer.
- Deploying with High availability and AutoScaling on the ECS cluster.
- Substitute the AWS CLI commands for an Automated deployment triggered by the GitHub repository.
- Optimization of the Docker image to reduce weight and to validate the required fix explined in the Dockerfile.
- Adding extra parameters in the CloudFormation template to increase Reusability.
- Include testing before the deployment with tools like TaskCat.
