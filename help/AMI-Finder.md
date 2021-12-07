# AMI Finder Stack

## References

* [Template](./../templates/bbb-on-aws-amifinder.template.yaml)


## Summary

| Resource | Usage | Values
| ------------- | ------------- | ------------- |
| applicationimageamiid | SSM Parameter for the variable part of the Application Server Ubuntu AMI  | ami-0e472ba40eb589f49 |
| Lambda function | get the latest AMI ID  | Refer lambda below |
| Lambda Role | Required to describe ec2 images and get AMI  | Refer lambda role below |
| A log group | ??  | ?? |
| Turn AMI Image ID | ??  | ?? |
| TURN AMI Parameter ID | SSM Parameter for the variable part of the Turn Ubuntu AMI  | ami-04505e74c0741db8d |


#### Lambda function to fetch the latest AMI-ID

```python

import boto3
import cfnresponse
import json

def handler(event, context):
  try:
    response = boto3.client('ec2').describe_images(
        Owners=[event['ResourceProperties']['Owner']],
        Filters=[
          {'Name': 'name', 'Values': [event['ResourceProperties']['Name']]},
          {'Name': 'architecture', 'Values': [event['ResourceProperties']['Architecture']]},
          {'Name': 'root-device-type', 'Values': ['ebs']},
        ],
    )

    amis = sorted(response['Images'],
                  key=lambda x: x['CreationDate'],
                  reverse=True)
    id = amis[0]['ImageId']

    cfnresponse.send(event, context, cfnresponse.SUCCESS, {}, id)
  except:
    cfnresponse.send(event, context, cfnresponse.FAILED, {}, "ok")

```

##### Lambda Role

##### AWS Lambda basic execution role

* AWSLambdaBasicExecutionRole
* Inline policy describing

```python
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": "ec2:DescribeImages",
            "Resource": "*",
            "Effect": "Allow"
        }
    ]
}
```