<!-- note marker start -->
NOTE: This repo contains only the documentation for the private BoltsOps repo code.
Original file: https://github.com/boltops-pro/cloudfront/blob/master/docs/one-account.md
The docs are publish so they are available for interested subscribers.
For access to the source code, you can become a BoltOps subscriber.
See: https://learn.boltops.com

<!-- note marker end -->

## One AWS Account Commands

Here are the summarized commands if you're using the One AWS account strategy.

## Deploy

Let's deploy now with [lono deploy](https://lono.cloud/reference/lono-cfn-deploy/).

    AWS_REGION=us-east-1 LONO_ENV=development lono cfn deploy cloudfront-development --blueprint cloudfront --sure --no-wait
    AWS_REGION=us-east-1 LONO_ENV=production  lono cfn deploy cloudfront-production  --blueprint cloudfront --sure --no-wait
