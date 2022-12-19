<!-- note marker start -->
**NOTE**: This repo contains only the documentation for the private BoltsOps Pro repo code.
Original file: https://github.com/boltopspro/cloudfront/blob/master/docs/one-account.md
The docs are publish so they are available for interested customers.
For access to the source code, you must be a paying BoltOps Pro subscriber.
If are interested, you can contact us at contact@boltops.com or https://www.boltops.com

<!-- note marker end -->

## One AWS Account Commands

Here are the summarized commands if you're using the One AWS account strategy.

## Deploy

Let's deploy now with [lono deploy](https://lono.cloud/reference/lono-cfn-deploy/).

    AWS_REGION=us-east-1 LONO_ENV=development lono cfn deploy cloudfront-development --blueprint cloudfront --sure --no-wait
    AWS_REGION=us-east-1 LONO_ENV=production  lono cfn deploy cloudfront-production  --blueprint cloudfront --sure --no-wait
