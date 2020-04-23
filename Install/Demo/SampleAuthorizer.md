Sample Authorizer
==
The included authorizer is an AWS lambda based authorizer intended for use with an API Gateway.

:warning: The authorizer is incomplete and is only intended to be used for testing purposes.

Source Location: `/authorization/authorizer-sample-aws/pom.xml`

What is included
==
* The authorizer lambda shell
* The setup of the authorizer lambda and association to the API Gateway in the CloudFormation template

What is missing
==
* Token evaluation/verification (see AWSLambdaStreamEntry.java)
    * When a call is made the token needs to be validated
* Token association with customer ids
    * When a call is made the visibility for the caller is determined by what the token will allow
* Full policy construction (to support token caching)
    * Multiple requests to different endpoints using the same token should have a policy that can be cached. This policy may allow visibility into one endpoint, while not another. A well formed policy should be constructed to indicate all access based on the initial token validation.

Visibility Doc? Do we explain this anywhere?
