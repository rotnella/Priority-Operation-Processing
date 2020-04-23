Web UI
==
There is a web based UI for Priority Operation Processing (POP) to perform basic operations.

Source location: `./web-ui`

Setup
==
* Adjust the `./web-ui/src/main/resources/js/server-info.js`
    * Set `endpointServerURL` to the URL of your API Gateway
* See the README for how to launch the web ui locally. It can also be deployed to any web host (s3+cloudfront etc.).

What's missing?
==
* The web ui is configured to not work using a valid authorization token (lining up with the [Sample Authorizer](SampleAuthorizer)). Tweak the authorization.js as necessary.
