# Turn Stack


templates\bbb-on-aws-bbbturn.template.yaml

## Summary

| Resource | Usage | Values
| ------------- | ------------- | ------------- |
| Autoscaling group |  | Refer CF script |
| Autoscaling group Launch Configuration | | Refer CF script |
| EC2 instance profile |  | Refer CF script
| Secrets Manager | Store secrets | Refer CF script and the values stored |
| Turn hostname | Hostname of turn server | Refer CF script AMI Instance id |
| Turn imageid | Hostname of imageid | Refer CF script AMI Instance |

## Secrets Manager

Stores the following values

* turnkeyname
* turnkeyvalue


## Launch Configuration

The scripts perform the following functions

| Resource | Usage | 
| ------------- | ------------- |
| DEBIAN_FRONTEND='noninteractive' apt -y -o Dpkg::Options::='--force-confdef' -o Dpkg::Options::='--force-confold' full-upgrade| Image result for DEBIAN_FRONTEND='noninteractive' noninteractive – You use this mode when you need zero interaction while installing or upgrading the system via apt. It accepts the default answer for all questions. Make apt-get (or aptitude) run with -y but not prompt for replacement of configuration files| 
| apt install -y git binutils python3-pip build-essential python-dev python-setuptools jq | Install the following utilities | 
| instance_ipv4 | Holds the public ip of the coturn instance | 
| instance_random | Generates the random alphanumeric to be used with route 53 as an FQDN | 
| instance_publichostname | Public IPV4 hostname | 
| Route 53 CLI management | https://github.com/barnybug/cli53/ | 
| scripts/route53-handler.sh | Managed creation and deletion of Route 53 instances via a service | 
| scripts/route53-handler.sh | Managed creation and deletion of Route 53 instances via a service | 
| Turn Secret obtained from the secrets manager | Refer CF | 

### Features of the script

* Waits until ec2 instances is up before installing coturn.
* The cfn-signal helper script signals CloudFormation to indicate whether Amazon EC2 instances have been successfully created or updated. If you install and configure software applications on instances, you can signal CloudFormation when those software applications are ready. 


```shell
#!/bin/bash -xe
apt update -y 

while fuser /var/{lib/{dpkg,apt/lists},cache/apt/archives}/lock >/dev/null 2>&1; do sleep 1; done

DEBIAN_FRONTEND='noninteractive' apt -y -o Dpkg::Options::='--force-confdef' -o Dpkg::Options::='--force-confold' full-upgrade

apt autoremove -y
apt autoclean
 
while fuser /var/{lib/{dpkg,apt/lists},cache/apt/archives}/lock >/dev/null 2>&1; do sleep 1; done

apt install -y git binutils python3-pip build-essential python-dev python-setuptools jq


pip3 install https://s3.amazonaws.com/cloudformation-examples/aws-cfn-bootstrap-py3-latest.tar.gz
pip3 install -U awscli

# associate Elastic IP
instance_ipv4=$(curl -s http://169.254.169.254/latest/meta-data/public-ipv4)
instance_random=$(cat /dev/urandom | tr -dc 'a-z0-9' | fold -w 6 | head -n 1)
instance_publichostname=tu-$instance_random
instance_fqdn=$instance_publichostname.cloudtrainer.online

aws ssm --region us-east-1 put-parameter --name "turnhostname" --value "$instance_publichostname" --type String --overwrite

# register in route53
wget --tries=10 https://github.com/barnybug/cli53/releases/download/0.8.18/cli53-linux-amd64 -O /usr/local/bin/cli53
sudo chmod +x /usr/local/bin/cli53

# create script for route53-handler
aws s3 cp s3://bbbexample-sources-bbbstackbucket-1bvk2zpfuiugd/route53-handler.service /etc/systemd/system/route53-handler.service
aws s3 cp s3://bbbexample-sources-bbbstackbucket-1bvk2zpfuiugd/route53-handler.sh /usr/local/bin/route53-handler.sh
chmod +x /usr/local/bin/route53-handler.sh

sed -i "s/INSTANCE_PLACEHOLDER/$instance_publichostname/g" /etc/systemd/system/route53-handler.service
sed -i "s/ZONE_PLACEHOLDER/Z06535511CU5UKTIIP8CL/g" /etc/systemd/system/route53-handler.service

systemctl daemon-reload
systemctl enable route53-handler
systemctl start route53-handler

turnsecret=$(aws secretsmanager get-secret-value --region us-east-1 --secret-id arn:aws:secretsmanager:us-east-1:716927497993:secret:BBBTurnSecret-MMmb3xfnD7gt-JG4vPw --query SecretString --output text | jq -r .turnkeyvalue)
sleep 1m

x=1
while [ $x -le 5 ]
do
  until host $instance_fqdn  | grep -m 1 "has address $instance_ipv4"; do sleep 5 ; done
  x=$(( $x + 1 ))
done

wget -qO- https://ubuntu.bigbluebutton.org/bbb-install.sh | bash -s -- -c $instance_fqdn:$turnsecret -e arun.ganesan@cloudtelsolutions.com

curl https://s3.amazonaws.com/aws-cloudwatch/downloads/latest/awslogs-agent-setup.py -O
chmod +x ./awslogs-agent-setup.py
aws s3 cp s3://bbbexample-sources-bbbstackbucket-1bvk2zpfuiugd/turn-cwagent.config /tmp/turn-cwagent.config
sed -i "s|SYSTEMLOGS_PLACEHOLDER|/bbbexample/systemlogs|g" /tmp/turn-cwagent.config
sed -i "s|APPLICATIONLOGS_PLACEHOLDER|/bbbexample/systemlogs|g" /tmp/turn-cwagent.config
./awslogs-agent-setup.py -n -r us-east-1 -c /tmp/turn-cwagent.config
systemctl enable awslogs

/usr/local/bin/cfn-signal -e $? --stack bbbexample-BBBTurnStack-I2WT90UE8I8E --resource BBBTurnAutoScaling --region us-east-1 || true


```