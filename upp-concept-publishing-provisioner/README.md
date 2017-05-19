# AWS Concept Publishing provisioner

The AWS Concept Publishing provisioning process will:

* Create an S3 Bucket, SNS Topic and 1/2 SQS Queues using the specified Cloudformation Template which is used as a part of the event-driven concept publishing pipeline.

The decommissioning process will:

* Delete the S3 Bucket and its contents
* Delete the SNS Topic and the SQS queues


## Building the Docker image
The Concept Publishing provisioner can be built locally as a Docker image:

`docker build -t coco/upp-concept-publishing-provisioner:local .`

## Provisioning a cluster
- Grab, customize and export the environment variables from the **AWS Concept Publishing SQS/SNS provisioning** LastPass note.
- The stack name will be `upp-concept-publishing-${ENVIRONMENT_TAG}` - eg, `upp-concept-publishing-pre-prod`
- The environment tag length must be less than or equal to 28
- The cloudformation script requires two parameters: 
  * Environment Tag - Which environment this belongs to. e.g pre-prod
  * IsMultiRegion - Whether the stack needs to be read in multiple regions (will spin up 2 SQS queues rather than 1)
  * AwsSecondaryRegion - The region in which to deploy the second SQS queue
- Run the following Docker commands:
```
docker pull coco/upp-concept-publishing-provisioner:latest
docker run \
    -e "AWS_DEFAULT_REGION=$AWS_DEFAULT_REGION" \
    -e "ENVIRONMENT_TAG=$ENVIRONMENT_TAG" \
    -e "ENVIRONMENT_TYPE=$ENVIRONMENT_TYPE" \
    -e "VAULT_PASS=$VAULT_PASS" \
    -e "IS_MULTI_REGION=$IS_MULTI_REGION" \
    -e "AWS_SECONDARY_REGION=$AWS_SECONDARY_REGION" \
    coco/upp-concept-publishing-provisioner:latest /bin/bash provision.sh
```

- You can check the progress of the CF stack creation in the AWS console [here](https://eu-west-1.console.aws.amazon.com/cloudformation/home?region=eu-west-1#/stacks).

## Manual Step

- Unfortunately there are a few manual steps that cannot be automated via cloud formation.
- For any stack created using this script you must grant access to the SQS queue to upp apps
- In the AWS console navigate to IAM -> Users and locate the content-container-apps user
- Copy the value for the User ARN to your clipboard
- In any region where an SQS queue was provisioned you must navigate to Simple Queue Service and locate your newly create queue
- Select Permissions -> Add a Permission. Paste the User ARN into the Principal field and tick the box for All SQS Actions, then Save.
- If you have provisioned a multiple stacks follow the next set of instructions; if not you can stop here
- Next you must go into the aws console for the secondary region. 
- Navigate to Simple Nofication Service dashboard and select topics
- Find the newly created topic associated with your environment and copy the ARN to your clipboard
- Navigate to the Simple Queue Service dashboard and locate your newly create 'notifications' queue
- Select permissions -> Add a Permission
- Click the tick box for SendMessage from the Actions drop down menu
- Then add a condition where the key aws:SourceArn has a value equal to what you copied from above
- Secondly select `Subscribe queue to SNS topic` from the Queue Actions drop down menu
- Select the default region from your provisioning and find the newly created topic `upp-concept-publishing-{environment_tag}...`
- Subscribe

## Decommissioning a cluster
- Grab, customize and export the environment variables from the **AWS Concept Publishing SQS/SNS provisioning** LastPass note.
- - Run the following Docker commands:
```
docker pull coco/upp-concept-publishing-provisioner:latest
docker run \
    -e "AWS_DEFAULT_REGION=$AWS_DEFAULT_REGION" \
    -e "ENVIRONMENT_TAG=$ENVIRONMENT_TAG" \
    -e "VAULT_PASS=$VAULT_PASS" \
    -e "IS_MULTI_REGION=$IS_MULTI_REGION" \
    -e "AWS_SECONDARY_REGION=$AWS_SECONDARY_REGION" \
    coco/upp-concept-publishing-provisioner:latest /bin/bash decom.sh
```

- You can check the progress of the CF stack creation in the AWS console [here](https://eu-west-1.console.aws.amazon.com/cloudformation/home?region=eu-west-1#/stacks).