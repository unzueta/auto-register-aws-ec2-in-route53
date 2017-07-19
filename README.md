# auto-register-aws-ec2-in-route53

This repository explains the steps to auto-register an ec2 instance in route53 that belong to different accounts. This is for a linux aws image.
1. Get the Id of the zone
2. Create a new custom IAM Policy for a restricted Route 53 Access
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "route53:ChangeResourceRecordSets",
                "route53:GetHostedZone",
                "route53:ListResourceRecordSets",
                "route53:ListHostedZones",
                "route53:ListHostedZonesByName"
            ],
            "Resource": [
                "arn:aws:route53:::hostedzone/YourZoneId"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "route53domains:Get*",
                "route53domains:List*"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```
2. Create a new IAM Group. Assign the policy created. 
3. Create a new IAM User with programming access
4. Create an EC2 user and group with read only access
5. Assign predefined tags to the EC2 instance: Name, internal-hostname and public-hostname
6. Install python, pip, awscli, ec-metadata and cli53 in the EC2 instance
```
$ sudo su -
$ yum install python34
$ curl -O https://bootstrap.pypa.io/get-pip.py
$ python3 get-pip.py 
$ pip install awscli --upgrade 
$ cd
$ wget http://s3.amazonaws.com/ec2metadata/ec2-metadata
$ chmod u+x ec2-metadata
$ wget https://github.com/barnybug/cli53/releases/download/0.8.7/cli53-linux-amd64
$ sudo mv cli53-linux-amd64 /usr/local/bin/cli53
$ sudo chmod +x /usr/local/bin/cli53
```
7. Create the script to update the records
```
$ sudo touch /usr/sbin/update-route53-dns
$ sudo chmod +x /usr/sbin/update-route53-dns
$ sudo nano /usr/sbin/update-route53-dns
```
Script:
```
#!/bin/sh

# Load configuration and export access key ID and secret for cli53 and aws cli
ZONE="YourZoneId"

# The TimeToLive in seconds we use for the DNS records
TTL="300"

# Get the private and public hostname from EC2 resource tags
REGION=$(curl -s http://169.254.169.254/latest/dynamic/instance-identity/docume$
INSTANCE_ID=$(ec2-metadata | grep 'instance-id:' | cut -d ' ' -f2)
INTERNAL_HOSTNAME=$(aws ec2 describe-tags --filters "Name=resource-id,Values=$I$
PUBLIC_HOSTNAME=$(aws ec2 describe-tags --filters "Name=resource-id,Values=$INS$

# Get the local and public IP Address that is assigned to the instance
LOCAL_IPV4=$(ec2-metadata | grep 'local-ipv4:' | cut -d ' ' -f2)
PUBLIC_IPV4=$(ec2-metadata | grep 'public-ipv4:' | cut -d ' ' -f2)

# Create a new or update the A-Records on Route53 with public and private IP ad$
cli53 rrcreate --profile route53user --replace "$ZONE" "$INTERNAL_HOSTNAME $TTL$
cli53 rrcreate --profile route53user --replace "$ZONE" "$PUBLIC_HOSTNAME $TTL A$
```
8. Modify the file .aws/credentials to create a default and a route53 profile
```
$ cd
$ nano .aws/credentials
```
```
[default]
aws_access_key_id = EC2 user Key Id
aws_secret_access_key = EC2 user Secret Key

[route53user]
aws_access_key_id = Route53user Key Id
aws_secret_access_key = Route53user secret key
```
8. Modify the file .aws/config to assign a region to the default and route53 profiles
```
$ cd
$ nano ./aws/config
```
```
[default]
region=your region
[route53user]
region= your region
```





