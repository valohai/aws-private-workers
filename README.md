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

## Private Workers Setup

1. Valohai: Create the AssumeRole (`valohai-customer-*`) in our AWS
2. Customer: Create the Key pair for the Worker Queue and Workers
3. Customer: Deploy the packaged CloudFormation template (requires AssumeRole ARN and Key pair name)
4. Valohai: Create the `*.vqueue.net` address for the Worker Queue instance (requires CloudFormation output PublicIp)
5. Customer: Setup the [Worker Queue](https://github.com/valohai/worker-queue) on the instance
6. Valohai: Run [prep](https://github.com/valohai/prep/) to setup environment types (requires CloudFormation outputs)

## Packaging and Deploying

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
aws cloudformation deploy --template-file valohai.yml --parameter-overrides AssumeRoleARN=ASSUMEROLE_ARN KeyPair=KEYPAIR_NAME --capabilities CAPABILITY_NAMED_IAM --stack-name Valohai
```

## Limitations

1) A Key pair must be manually created in the AWS region of choice before running the template.

2) The CloudFormation template does not create a rule for SSH in the `valohai-sq-queue` security group. This needs to be added to be able to SSH in and set up the Worker Queue.

Depending on who does the Worker Queue installation, this rule can allow a customer IP or the Valohai `BastionHost` IP.

3) The `aws package...` command uploads the nested templates into the defined S3 bucket. If this bucket is private, those files are not available from all AWS accounts.

Therefore we need to consider what's the best way to publish the packaged main and nested templates to the customers. Possibly they could all be public in GitHub?
