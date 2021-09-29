# Valohai Private Workers CloudFormation

This repository contains a CloudFormation template to deploy the resources required by a Valohai Private Workers setup in AWS.

There are five templates in total:

```
main.yml
├── network.yml
|   └── subnet.yml
├── worker-queue.yml
└── workers.yml
```

These templates must be packaged in order to be deployed by customers.

## Deploy Current Version

Before running the template you'll need the following information from Valohai:
* `AssumeRoleARN` which is Valohai's user that will assume a role in your AWS subscription to manage EC2 instances
* `QueueAddress` that will be assigned for the queue in your subscription

You will also need to generate a EC2 Key Pair in your AWS Console before creating a stack. This key will be used as the default SSH key for all Valohai created resources.

The current version of this CloudFormation template can be deployed from https://valohai-cfn-templates-public.s3.eu-west-1.amazonaws.com/aws-private-workers.yml.

## Package and Deploy New Version

Prerequirements:
* AWS command-line client
* AWS account, configured in the CLI: `aws configure --profile PROFILE_NAME`
* S3 bucket in the AWS account. We will refer to this as bucket S3_BUCKET
* KeyPair in the AWS account. We will refer to this as KEYPAIR_NAME
* AssumeRole in the Valohai AWS account. We will refer to this as ASSUMEROLE_ARN

```bash
# Use the AWS account
export AWS_PROFILE=PROFILE_NAME

# Package the nested stacks to one YAML file
# if you only want to build a new release version, you can stop after this
aws cloudformation package --template-file main.yml --output-template valohai.yml --s3-bucket S3_BUCKET

# Deploy the CloudFormation template
# or do it via the AWS Management Console
aws cloudformation deploy --template-file valohai.yml --parameter-overrides AssumeRoleARN=ASSUMEROLE_ARN KeyPair=KEYPAIR_NAME QueueAddress=ADDRESS --capabilities CAPABILITY_NAMED_IAM --stack-name Valohai
```
