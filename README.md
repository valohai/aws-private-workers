# Valohai Hybrid Setup - CloudFormation

This repository contains a CloudFormation template to deploy the resources required by a Valohai Hybrid setup in AWS.

There are six templates in total:

```
iam.yml

main.yml
├── network.yml
|   └── subnet.yml
├── s3bucket.yml
└── worker-queue.yml
```

These templates must be packaged in order to be deployed by customers.

## Deploy Current Version

The current version of these CloudFormation templates can be deployed from:
1. https://valohai-cfn-templates-public.s3.eu-west-1.amazonaws.com/iam.yml
2. https://valohai-cfn-templates-public.s3.eu-west-1.amazonaws.com/aws-hybrid-workers.yml

Before running the template you'll need the following information from Valohai:
* `AssumeRoleARN` is the ARN of the user Valohai will use to assume a role in your AWS subscription to manage EC2 instances.
* `QueueAddress` will be assigned for the queue in your subscription.

You will also need to generate a EC2 Key Pair in your AWS Console before creating a stack. This key will be used as the default SSH key for all Valohai created resources.

## What will get deployed?

This template is designed to provision the required services in a fresh AWS Account. The following services will be deployed:

* **VPC and Subnets** in the selected region. Valohai will also deploy a Internet Gateway and RouteTables.
* **Two security groups** for Valohai resources:
  * `valohai-sg-workers` that all the Valohai autoscaled EC2 instances will use.
    * By default it doesn't have ports open. You'll have to open ports to allow for example connecting over SSH to the instances.
  * `valohai-sg-queue` for the `valohai-queue` EC2 instance.
    * It will allow app.valohai.com to connect to Redis (over TLS) on port 63790.
    * Allow the autoscaled Valohai workers to connect to Redis on port 63790.
    * Open port 80 for the Let's Encrypt challenge and certificate renewal.

* **EC2 instance** (`valohai-queue`) that's responsible for storing the job queue, job states, and short-term logs. Valohai communicates with this machines (Redis over TLS) to schedule new jobs and access the logs of existing jobs.
  * You'll need to provide a key pair that can be uploaded to your AWS account for connecting to this instance.
  * The machine will also have an Elastic IP attached to it.

* **A secret** stored in your AWS Secrets Manager. The secret `ValohaiRedisSecret` contains the password for Redis that's located inside in your `valohai-queue` instance.
* **S3 Bucket** where Valohai will upload logs from your executions and commit snapshots. All the generated artefacts will be uploaded to this bucket by default.
* **IAM Roles:**
  * `ValohaiQueueRole` will be attached to the Valohai Queue instance, and allows it to fetch the generated password from your AWS Secrets Manager. Access is restricted to secrets that are tagged `valohai:1`
  * `ValohaiWorkerRole` is attached to all autoscaled EC2 instances that are launched for machine learning jobs.
  * `ValohaiMaster` is the role that the Valohai service will use to manage autoscaling and EC2 resources. The role is also used to manage the newly provisioned `valohai-data-*` S3 Bucket.
