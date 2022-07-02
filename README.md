## CloudFront Dynamic Content Websites
[Amazon CloudFront](https://aws.amazon.com/cloudfront/) can also be used for [dynamic content delivery](https://aws.amazon.com/cloudfront/dynamic-content/). These are sites with personalized and dynamic web content that are not cacheable.  Examples include blog, e-commerce, news, travel and retail web sites.

While dynamic content are not cacheable on CloudFront edge locations, static content embeded in the web pages (such as javascript, stylesheets and images) can be cached by CloudFront for performance. However, configuring caching requires custom configuration of cache policies for different file types. 

This repository provides a starter [AWS CloudFormation](https://aws.amazon.com/cloudformation/) template that can be used to provision a secure and higher performance CloudFront distribution for dynamic content websites.

## Cache Busting and CloudFront cache key
To ensure that CloudFront caches are optimized and do not contain outdated content, an understanding of common web cache busting techniques and CloudFront [cache key](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/controlling-the-cache-key.html) is required.

Web developers typically use unique URL paths or query strings to perform cache busting.
Below are some examples
1. `/path/example_v1.css` or `/path/v1/example.css`
1. `/path/example.js?version=1`
1. `/path/example.jpg?1`

To cache bust, new versions can be as follows
1. `/path/example_v2.css` or `/path/v2/example.css`
1. `/path/example.js?version=2`
1. `/path/example.jpg?2`

While first entry can use CloudFront's [CachingOptimized](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/using-managed-cache-policies.html) managed cache policy, the rest requires custom cache policy with [query string parameters](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/QueryStringParameters.html) to ensure that CloudFront do not cache out-dated content. The cache policies to be used are therefore

1. Default: [CachingDisabled] managed policy
1. *.css: [CachingOptimized] managed policy
1. *.js: Custom cache policy with query string `version` in Cache key
1. *.jpg Custom cache policy with all query strings in Cache key 



## CloudFront configuration overview
The CloudFormation template allows users to select type of cache policy for common static content file types and configure CloudFront distribution settings as illustrated below
![CloudFormation config](../images/Architecture.png "Architecture")

## Deployment via CloudFormation console
Download [template](template.yaml) file and login to AWS [CloudFormation console](https://console.aws.amazon.com/cloudformation/home#/stacks/create/template). Choose **Create Stack**, **Upload a template file**, **Choose File**, select your .YAML file and choose **Next**.


Specify a **Stack name** and specify parameters values. All fields are required. You will need to enter origin domain name and configure cache policy accordingly for each file type. 
![CloudFormation parameters](../images/CloudFormation-1.png "Parameters")
![CloudFormation parameters](../images/CloudFormation-2.png "Parameters")




It usually take up to 5 minutes to provision. After your stack has been successfully created, its status changes to **CREATE_COMPLETE**.

Go to **Outputs** tab where you can navigate to created CloudFront distribution and CloudFront console
![Outputs](../images/Outputs.png "Outputs")

Navigate to **CloudFront console**, **Behaviors** to verify settings for differnt file types
![Behaviors](../images/Behaviors.png "Behaviors")

## Security features
The following security features are enabled
1. Enable [Security headers response policy](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/using-managed-response-headers-policies.html#managed-response-headers-policies-security)
1. [Require HTTPS](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/using-https-viewers-to-cloudfront.html) for all communications between viewers and CloudFront
1. (Optional) Create [AWS WAF](https://aws.amazon.com/waf/) Web ACL with common [AWS Managed Rules](https://docs.aws.amazon.com/waf/latest/developerguide/aws-managed-rule-groups-list.html) enabled and associate with CloudFront to provide layer 7 protection

In addition, the use of CloudFront provides layer 3 and layer 4 DDoS protection at the edge network and ensures that only well-form HTTP/HTTPS requests are sent to origin.



