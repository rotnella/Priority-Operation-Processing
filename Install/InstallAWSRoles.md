AWS Roles and Permissions
==

Template Location
==
```
./deploy/endpoint/aws/AWS-Roles-CloudFormation.json
```

Roles
-----
The CloudFormation template used to deploy the endpoints requires the creation of a number of roles so the component access is limited. The above template is specific to the creation of those roles.

Installation
==
1. Deploy the `AWS-Roles-CloudFormation` stack via [AWS Cloudformation](https://console.aws.amazon.com/cloudformation/)
1. Access the roles via [AWS IAM](https://console.aws.amazon.com/iam)
1. Copy the ARNs out for the Roles created by the stack. The ARNs are used to deploy the rest of the components.

Role Specifications
==
Roles
-----
| Role| Policies|
| ----| -------|
| POP.UnifiedEndpoint.app| AWSLambdaVPCAccessExecutionRole<br/>CustomerManaged-POP.SQSEditor<br>CustomerManaged-POP.DynamoDBEditor |
| POP.Scheduler.app| AWSLambdaVPCAccessExecutionRole<br/>CustomerManaged-POP.SQSEditor<br/>CustomerManaged-POP.DynamoDBEditor<br/>CustomerManaged-POP.LambdaLauncher|
| POP.Callback.trigger| AWSLambdaVPCAccessExecutionRole<br/>CustomerManaged-POP.DynamoDBEditor|
| POP.Reclaimer.app| AWSLambdaVPCAccessExecutionRole<br/>CustomerManaged-POP.DynamoDBEditor|
| POP.Reaper.app| AWSLambdaVPCAccessExecutionRole<br/>CustomerManaged-POP.DynamoDBEditor|
| POP.authorizer| AWSLambdaVPCAccessExecutionRole|

### Trust Relationship

In order for a role to be able to be used by a Lambda you must add the following to the **Trust Relationship** on the role.
```
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
```

Policies
--------

:warning: The roles below are prefixed with "CustomerManaged-" before the name. This may not apply to you and you can name the role accordingly.

### CustomerManaged-POP.SQSEditor
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "sqs:ChangeMessage*",
                "sqs:GetQueue*",
                "sqs:*Message",
                "sqs:*MessageBatch"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```
### CustomerManaged-POP.DynamoDBEditor
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "dynamodb:*Table",
                "dynamodb:*Item",
                "dynamodb:Get*",
                "dynamodb:Query",
                "dynamodb:Scan",
                "dynamodb:DescribeStream",
                "dynamodb:Put*",
                "dynamodb:List*"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```
        
### CustomerManaged-POP.LambdaLauncher
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "lambda:InvokeFunction",
            "Resource": "*"
        }
    ]
}
```
