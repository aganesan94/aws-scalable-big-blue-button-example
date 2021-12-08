# AMI Finder Stack

## References

* [Template](./../templates/bbb-on-aws-ses.template.yaml)


## Summary

| Resource | Usage | Values
| ------------- | ------------- | ------------- |
| Lambda | SES Provider | Refer CF script |
| IAM Role | SES Provider IAM Role  | ------------- |
| A log group | ??  | ?? |
| Lambda | SES Provider  | ?? |
| IAM Role | Role for the secret provider  |  |



##### Lambda Role

##### AWS Lambda basic execution role

* AWSLambdaBasicExecutionRole
* Inline policy describing

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "ses:VerifyDomainDkim",
                "ses:DeleteIdentity",
                "ses:ListIdentities",
                "ses:VerifyDomainIdentity",
                "ses:DescribeActiveReceiptRuleSet",
                "ses:SetActiveReceiptRuleSet",
                "ses:GetIdentityVerificationAttributes",
                "ses:GetIdentityNotificationAttributes",
                "ses:SetIdentityNotificationTopic",
                "ses:SetIdentityHeadersInNotificationsEnabled",
                "ses:SetIdentityFeedbackForwardingEnabled"
            ],
            "Resource": "*",
            "Effect": "Allow"
        },
        {
            "Action": [
                "route53:GetHostedZone",
                "route53:ChangeResourceRecordSets",
                "route53:ListResourceRecordSets"
            ],
            "Resource": "arn:aws:route53:::hostedzone/Z06535511CU5UKTIIP8CL",
            "Effect": "Allow"
        },
        {
            "Action": [
                "lambda:InvokeFunction"
            ],
            "Resource": [
                "arn:aws:lambda:us-east-1:716927497993:function:bbbexample-BBBSESProviderStack-1FF1NCRGY0SXA-ses-provider"
            ],
            "Effect": "Allow"
        }
    ]
}
```