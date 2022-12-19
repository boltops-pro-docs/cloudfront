<!-- note marker start -->
**NOTE**: This repo contains only the documentation for the private BoltsOps Pro repo code.
Original file: https://github.com/boltopspro/cloudfront/blob/master/README.md
The docs are publish so they are available for interested customers.
For access to the source code, you must be a paying BoltOps Pro subscriber.
If are interested, you can contact us at contact@boltops.com or https://www.boltops.com

<!-- note marker end -->

# CloudFront Blueprint

![CodeBuild](https://codebuild.us-west-2.amazonaws.com/badges?uuid=eyJlbmNyeXB0ZWREYXRhIjoiSzBnL0VYbjc2Vm54a1lqbWREV3VsczBvWWw3WHNsbTNnMkowMjNaOVl4VmtxMGpJQlFWWjJROUxJdnlYcFh3dkRacnRmNHdpTjd2aVE1TlpzdFlTQVdRPSIsIml2UGFyYW1ldGVyU3BlYyI6IldpMGNuZHVERFUrOEE1R1UiLCJtYXRlcmlhbFNldFNlcmlhbCI6MX0%3D&branch=master)

[![BoltOps Badge](https://img.boltops.com/boltops/badges/boltops-badge.png)](https://www.boltops.com)

This blueprint provisions a CloudFront Distribution.

* It can provision a Distribution with an S3 bucket origin or Custom origin.
* The S3 bucket can be an existing bucket, or the blueprint can create and manage the S3 bucket for you.
* In addition to being able to set up CloudFront Distribution Aliases, the blueprint can also automatically add the matching Route53 records and configure an existing ACM cert.
* Basic Auth support allows you to protect the CloudFront distribution with standard htpasswd files.

Most resource properties are configurable with [Parameters](https://lono.cloud/docs/configs/params/). Additionally, all properties are generally configurable with [Variables](https://lono.cloud/docs/configs/params/). The blueprint is extremely flexible and customizable for your needs.  More info in the "Configure: More Details" section.

## Usage

1. Add blueprint to Gemfile
2. Configure: configs/cloudfront values
3. Deploy blueprint

## Add

Add the blueprint to your lono project's `Gemfile`.

```ruby
gem "cloudfront", git: "git@github.com:boltopspro/cloudfront.git"
```

## Configure

Use the [lono seed](https://lono.cloud/reference/lono-seed/) command to generate a starter config params files.

    LONO_ENV=development lono seed cloudfront
    LONO_ENV=production  lono seed cloudfront

The files in `config/cloudfront` folder will look something like this:

    configs/cloudfront/
    ├── params
    │   ├── development.txt
    │   └── production.txt
    └── variables
        ├── development.rb
        └── production.rb

Configure the `configs/cloudfront/params` and `configs/cloudfront/variables` files.  Here's an example:

    Comment=my cloudfront distribution
    DomainName=test-website-3.s3.amazonaws.com # existing bucket
    AcmCertificateArn=arn:aws:acm:us-east-1:112233445566:certificate/8d8919ce-a710-4050-976b-b33d8EXAMPLE

## Deploy

Use the [lono cfn deploy](http://lono.cloud/reference/lono-cfn-deploy/) command to deploy.  It is recommended that you deploy to the us-east-1 region. If you are using basic auth, then it is required.

    AWS_REGION=us-east-1 LONO_ENV=development lono cfn deploy cloudfront --sure --no-wait
    AWS_REGION=us-east-1 LONO_ENV=production  lono cfn deploy cloudfront --sure --no-wait

It takes about 15-30m to deploy the CloudFront distribution. Times may vary.

If you are using One AWS Account, use these commands instead: [One Account](docs/one-account.md).

## Configure: More Details

## Default Root Object

To configure the default root object, use the `DefaultRootObject` parameter. Example:

    DefaultRootObject=index.html

Do *not* include / at the beginning.

### Redirect HTTP to HTTPS

If you wish to configure the CloudFront Behavior to redirect all HTTP requests to HTTPS, you can use the  `ViewerProtocolPolicy` parameter.  Example:

    ViewerProtocolPolicy=redirect-to-https

The available values are: `allow-all`, `redirect-to-https`, and `https-only`.

[ViewerProtocolPolicy Docs](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-cloudfront-distribution-defaultcachebehavior.html#cfn-cloudfront-distribution-defaultcachebehavior-viewerprotocolpolicy).

### Default TTL

It is generally recommended that your origin set the `Cache-Control max-age`, `Cache-Control s-maxage`, or `Expires` headers to finely control how long CloudFront should cache each piece of content. If the origin does not set these headers, then you may want to configure the `DefaultTTL` parameter.  Example:

    DefaultTTL=3600 # 1h

CloudFront uses `DefaultTTL=86400` by default, which is 24 hours.

### Origin Path

You can tell CloudFront to request your content from a directory in your Amazon S3 bucket or your custom origin. When you include the OriginPath element, specify the directory name, beginning with a /. CloudFront appends the directory name to the value of DomainName, for example, example.com/production. Do not include a / at the end of the directory name. Use the `OriginPath` parameter to achieve this. Example:

    OriginPath=/production

### Basic Auth Support

This blueprint supports adding basic auth in front of the CloudFront distribution. This is accomplished with a [Lambda@Edge](https://aws.amazon.com/lambda/edge/) function.  To enable basic auth, set the `@htpasswd_s3_bucket` and `@htpasswd_s3_key` variables. Example:

    @htpasswd_s3_bucket = "my-bucket"
    @htpasswd_s3_key = "path/to/htpasswd.txt"

The `htpasswd.txt` file is a standard [Apache htpasswd file](https://httpd.apache.org/docs/2.4/programs/htpasswd.html) and looks like this:

    tung:$apr1$leaLBRid$IQLvTWZcRfvOqZgIHPvbR0
    bob:$apr1$uFZrcXVQ$JvPi4UQ13M3v5gIxpLn1F1

You do not have to redeploy the CloudFront distribution to add or remove basic auth users. Just update the file on s3. By default, changes take 60 seconds to reflect. This is configurable with the `@htpasswd_ttl` variable. Example:

    @htpasswd_ttl = 600 # 600s or 10m

The Lambda@Edge function caches the s3 getObject call within the [Lambda Execution Context](https://docs.aws.amazon.com/lambda/latest/dg/running-lambda-code.html). Lambda functions get recycled every few hours. So the max ttl will be limited by that.

**IMPORTANT**: If you are using basic auth, the stack *must* be deployed to the us-east-1 region. The Lambda@Edge function must be in us-east-1. CloudFront then replicates the function to all edge locations.  A simple way to deploy to the us-east-1 region is to use the `AWS_REGION` environment variable.

It is recommended to not use the same s3 bucket to store the htpasswd file as your content because you don't want the htpasswd file to be reachable.

### Existing vs Managed S3 Bucket

You can use an existing S3 bucket or have the blueprint create a managed an S3 bucket for you.  This is determined by the `DomainName` parameter. If you want to use an existing bucket, set the parameter with the S3 Domain url. For example:

    DomainName=my-bucket.s3.amazonaws.com

If you want to have the blueprint create and manage the s3 bucket, leave the parameter blank.  The managed s3 bucket has SSE-S3 encryption enabled.

### Custom Origin

If you want a custom origin like an HTTP server, this can be achieved with the `@custom_origin_config` variable. Example:

```ruby
@custom_origin_config = {
  HTTPPort: 80, # Integer
  HTTPSPort: 443 # Integer
  OriginKeepaliveTimeout: 5, # Integer
  OriginProtocolPolicy: "match-viewer", # String http-only | https-only | match-viewer - Required
  OriginReadTimeout: 30, # Integer
  OriginSSLProtocols: %w[SSLv3 TLSv1 TLSv1.1 TLSv1.2], # [ String, ... ] # SSLv3 | TLSv1 | TLSv1.1 | TLSv1.2
}
```

Here are the docs for the [CustomOriginConfig](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-cloudfront-distribution-customoriginconfig.html) structure.

Using a custom origin will disable S3 bucket origins. Unless you have set the `@origins` variable to customize completely, the blueprint is designed for either an S3 Bucket origin or a custom web server origin only.

### CloudFront Origin Access Identity

CloudFront supports [AWS::CloudFront::CloudFrontOriginAccessIdentity](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-cloudfront-cloudfrontoriginaccessidentity.html) to allow secure access to the S3 bucket objects. The OriginAccessIdentity is essentially a virtual user that will be given `s3:GetObject` via a S3 bucket policy.

If you have configured the blueprint to use a managed S3 bucket, an OriginAccessIdentity is automatically created, and the S3 Bucket policy is automatically configured.

If you're using an existing S3 Bucket, then you'll need to add the Bucket Policy manually yourself.  The bucket IAM policy structure looks like this:


    {
        "Version": "2008-10-17",
        "Id": "PolicyForCloudFrontPrivateContent",
        "Statement": [
            {
                "Sid": "1",
                "Effect": "Allow",
                "Principal": {
                    "AWS": "arn:aws:iam::cloudfront:user/CloudFront Origin Access Identity REPLACE_WITH_ORIGIN_ACCESS_IDENTITY"
                },
                "Action": "s3:GetObject",
                "Resource": "arn:aws:s3:::REPLACE_WITH_S3_BUCKET_NAME/*"
            }
        ]
    }

You can find the `OriginAccessIdentity` in the CloudFormation Console Outputs.

If you're using a Custom Origin, the OriginAccessIdentity does not apply.

### CloudFront Aliases

Aliases can be configured with the `Aliases` parameter or the `@aliases` variable.  The `@aliases` has higher precedence.  Example:

configs/cloudfront/parameters/development.txt:

```ruby
Aliases=www1.domain.com,www2.domain.com # comma separated
```

Or set aliases with a variable:

configs/cloudfront/variables/development.rb:

```ruby
@aliases = %w[www1.domain.com www2.domain.com] # Ruby Array
```

If you are configuring an alias, you must configure `AcmCertificateArn` and `SslSupportMethod` also.

configs/cloudfront/params/development.txt:

    AcmCertificateArn=arn:aws:acm:us-east-1:112233445566:certificate/d68472ba-04f8-45ba-b9db-14f83EXAMPLE
    SslSupportMethod=sni-only

IMPORTANT: CloudFront *only* supports ACM certs in the `us-east-1` region.

### Route53 Records

You can also have blueprint create a route53 record with the`@route53_records` variable and the `HostedZoneId` parameter. Example:


configs/cloudfront/variables/development.rb:

```ruby
@route53_records = %w[www1.domain.com www2.domain.com]
```

configs/cloudfront/params/development.txt:

```ruby
HostedZoneId=/hostedzone/Z01963071SQH02EXAMPLE
```

### Match aliases and route53 records

You can match the `@aliases` and `@route53_records` by assigning them at the same time.

```ruby
@aliases = @route53_records = %w[www1.domain.com www2.domain.com]
```

### Advanced Properties Customizations

Most of the [AWS::CloudFront::Distribution DistributionConfig](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-cloudfront-distribution-distributionconfig.html) properties are configurable with [Parameters](https://lono.cloud/docs/configs/params/). Some properties are with [Variables](https://lono.cloud/docs/configs/shared-variables/) though when needed.  For example, if the property is a complex type.  Here's example of configuring DistributionConfig with variables for [AWS::CloudFront::Distribution CustomErrorResponse](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-cloudfront-distribution-customerrorresponse.html).

```ruby
custom_error_responses = {
  ErrorCachingMinTTL: 60, # Double
  ErrorCode: 500, # Integer
  ResponseCode: 200, # Integer
  ResponsePagePath: "/4xx-errors/403-forbidden.html", # String
}
```

You can view all configurable variables in [helpers/variables_helper.rb](app/helpers/variables_helper.rb).

The blueprint is written so that Variables take higher precedence than Parameters.

### Lambda@Edge Function Replicas Slow to Delete

If you have added basic auth and the Lr later removed it, you may see a [CloudFormation stack error](https://gist.github.com/tongueroo/b8492f369cce144670678350d33f99cd) that says the Lambda function fails to delete. This is due to how Lambda@Edge works.  CloudFront creates replicas of the Lambda function and copies them to the edge locations. These replica functions take a while to delete, sometimes a couple of hours.

Instead of CloudFormation rolling back the stack, it reports that it failed to delete the Lambda function and moves on. It is safe to ignore. But in these cases, you have to manually delete and cleanup the orphaned Lambda function when the replicas are gone.
