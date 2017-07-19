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
4. Assign predefined tags to the EC2 instance: Name, internal-hostname and public-hostname
5. Install python, pip, awscli, ec-metadata and cli53 in the EC2 instance
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
6. Create the script to update the records
```
$ sudo touch /usr/sbin/update-route53-dns
$ sudo chmod +x /usr/sbin/update-route53-dns
$ sudo nano /usr/sbin/update-route53-dns
```




