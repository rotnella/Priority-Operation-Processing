Install AWS Components
=============================

Roles and Permissions
---------------------
You will need your AWS account configured to allow access for installing the Priority Operation Processing (POP) components.

Follow [Install AWS Roles](InstallAWSRoles) to configure the necessary roles and permissions.

Build the Source Code
-------------------------
There is a parent pom in the root of the Git project.
`mvn clean install`

Setup an S3 Bucket to Store Compiled Lambdas
---------------------------------------------
Our AWS CloudFormation looks in a configured S3 bucket for the lambda binaries. You can remove this dependency by installing the lambdas manually, but it would take more work. The s3 bucket does **NOT** require public access. It should exist within the same account as where you plan to deploy the CloudFormation stack.

In your AWS cosole go to the S3 configuration page.
1. Create a new bucket called `pop` with directory `deploy` (or edit the cloudformation for your bucket name)
2. Upload the lambda binaries to your S3 Bucket. When you build the project the zip files should be auto-copied to the deploy/endpoint/aws/binaries directory. Here are the files:
- pop-authorizer-sample-aws-<version>.zip
- pop-endpoint-impl-aws-<version>-lambda_deployment_package_assembly.zip
- pop-callback-progress-retry-aws-<version>-SNAPSHOT-lambda_deployment_package_assembly.zip
- pop-scheduling-master-aws-<version>-lambda_deployment_package_assembly.zip
- pop-scheduling-queue-aws-<version>-lambda_deployment_package_assembly.zip
- pop-scheduling-monitor-aws-<version>-lambda_deployment_package_assembly.zip
- pop-data-object-reaper-aws-<version>-lambda_deployment_package_assembly.zip
- pop-agenda-reclaimer-aws-<version>-lambda_deployment_package_assembly.zip

Deploying Cloudformation
--
Priority Operation Processing software uses [AWS Cloudformation](https://console.aws.amazon.com/cloudformation/) to deploy the POP's AWS software components.

### Template Location
```
./deploy/endpoint/aws/Main-AWS-CloudFormation.json
```

### Steps
1. Change all instances of _[add your accountID]_ in the Main-AWS-CloudFormation.json to your AWS accountID
1. Access the AWS Console CloudFormation page.
1. Select Stacks
1. Click `Create stack` (with new Resources)
1. This will open the `Create stack` screen
1. Select `Template is ready`
1. Select `Upload a template file`
    1. Select `Main-AWS-CloudFormation.json` when prompted
1. Click Next to open the `Specify stack details` screen
1. Set the `Stack name` and adjust any settings as necessary.
1. Click Next to open the `Configure stack options` screen
    1. There are no specific option adjustments needed.
1. Click Next to open the `Review` screen
1. Click `Create stack` to initiate the process.

What the template installs
--
* Endpoint Lambdas
* Scheduler Lambdas
* Retry Lambda
* API Gateway
* Authorizer

### Test your AWS installation
See [How to build the code](BuildCode) and run the [Endpoint Functional Tests](EndpointFunctionalTests.md).
