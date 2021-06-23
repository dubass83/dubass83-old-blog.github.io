# AWS config
---

[Product page](https://aws.amazon.com/config/)  
[Documentation for AWS config](https://docs.aws.amazon.com/config/latest/developerguide/WhatIsConfig.html)  
[AWS Config pricing](https://aws.amazon.com/config/pricing/)

AWS Config is a service that enables you to assess, audit, and evaluate the configurations of your AWS resources. Config continuously monitors and records your AWS resource configurations and allows you to automate the evaluation of recorded configurations against desired configurations. With Config, you can review changes in configurations and relationships between AWS resources, dive into detailed resource configuration histories, and determine your overall compliance against the configurations specified in your internal guidelines. This enables you to simplify compliance auditing, security analysis, change management, and operational troubleshooting.

## Example config

+ Created AWS config
![](https://s3.eu-central-1.amazonaws.com/cdn.dubass83.xyz/imgs/create-config-1.png)

+ Added rules
![](https://s3.eu-central-1.amazonaws.com/cdn.dubass83.xyz/imgs/added-rule-1.png)
![](https://s3.eu-central-1.amazonaws.com/cdn.dubass83.xyz/imgs/added-rule-2.png)
![](https://s3.eu-central-1.amazonaws.com/cdn.dubass83.xyz/imgs/added-rule-3.png)

+ Review config before apply
![](https://s3.eu-central-1.amazonaws.com/cdn.dubass83.xyz/imgs/config-review.png)

+ After first start
![](https://s3.eu-central-1.amazonaws.com/cdn.dubass83.xyz/imgs/first-start.png)

+ AWS config Dashboard
![](https://s3.eu-central-1.amazonaws.com/cdn.dubass83.xyz/imgs/dashboard.png)
![](https://s3.eu-central-1.amazonaws.com/cdn.dubass83.xyz/imgs/resources-in-scope.png)
![](https://s3.eu-central-1.amazonaws.com/cdn.dubass83.xyz/imgs/advanced-queries.png)
![](https://s3.eu-central-1.amazonaws.com/cdn.dubass83.xyz/imgs/conformance-packs.png)

+ Pricing example
![](https://s3.eu-central-1.amazonaws.com/cdn.dubass83.xyz/imgs/Pricing-example.png)
![](https://s3.eu-central-1.amazonaws.com/cdn.dubass83.xyz/imgs/some-important-info-from-docs.png)

## Example of usage lambda function for non compliance remediation in AWS config

```python3
"""
Lambda function to poll Config for noncompliant resources, and automatically
apply remediation by removing S3 bucket ACLs which allow public read.
Notifications are sent to an SNS topic.
"""

import boto3

# AWS Config settings
ACCOUNT_ID = boto3.client('sts').get_caller_identity()['Account']
CONFIG_CLIENT = boto3.client('config')
MY_RULE = "s3-bucket-public-read-prohibited"

# AWS SNS Settings
SNS_CLIENT = boto3.client('sns')
SNS_TOPIC = 'arn:aws:sns:us-east-1:' + ACCOUNT_ID + ':' + 'mytopic'
SNS_SUBJECT = 'Compliance Update'


def lambda_handler(event, context):
    """Entry point"""

    # Get compliance details
    non_compliant_detail = CONFIG_CLIENT.get_compliance_details_by_config_rule(
        ConfigRuleName=MY_RULE, ComplianceTypes=['NON_COMPLIANT'])

    if len(non_compliant_detail['EvaluationResults']) > 0:

        print(
            'The following resource(s) are not compliant with AWS Config rule: ' + MY_RULE)

        non_complaint_resources = ''

        for result in non_compliant_detail['EvaluationResults']:
            print(result['EvaluationResultIdentifier']
                  ['EvaluationResultQualifier']['ResourceId'])
            non_complaint_resources = non_complaint_resources + \
                result['EvaluationResultIdentifier']['EvaluationResultQualifier']['ResourceId'] + '\n'

        sns_message = 'AWS Config Compliance Update\n\n Rule: ' \
            + MY_RULE + '\n\n' \
            + 'The following resource(s) are not compliant:\n' \
            + non_complaint_resources

        SNS_CLIENT.publish(TopicArn=SNS_TOPIC,
                           Message=sns_message, Subject=SNS_SUBJECT)

        resource_type = result['EvaluationResultIdentifier']['EvaluationResultQualifier']['ResourceType']

        if resource_type == 'AWS::S3::Bucket':
            bucket_name = result['EvaluationResultIdentifier']['EvaluationResultQualifier']['ResourceId']
            print('Bucket: ' + bucket_name)
            s3 = boto3.resource('s3')
            bucket = s3.Bucket(bucket_name)
            acl = bucket.Acl()
            acl.put(ACL='private')

    else:
        print('No noncompliant resources detected.')
```