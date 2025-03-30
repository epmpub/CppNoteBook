# Aliyun && AWS migration guideline

# Part 1:

## Install aws cli

console ：

​	OS : ubuntu 20.04 ,使用Linux系统

​	install aws command line console tools：

```shell
sudo apt  install awscli
```

## config aws 

​	config command:

```shell
aws configure
```

需要准备的信息：

access key / secret key:

![image-20231115163738720](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231115163738720.png)





![image-20231115163703689](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231115163703689.png)





Asia Pacific (Tokyo)    ap-northeast-1

![image-20231115095957182](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231115095957182.png)

-----------------------------------------------------------------------------------

create security group:

```shell
aws ec2 create-security-group --group-name demo-sg --description "AWS ec2 CLI Demo SG" --tag-specifications 'ResourceType=security-group,Tags=[{Key=Name,Value=demo-sg}]' --vpc-id vpc-010b0dfa3d852a08f
```

change security group:

```shell
aws ec2 authorize-security-group-ingress \
    --group-id "sg-07570e17ab8331f13" \
    --protocol tcp \
    --port 22 \
    --cidr "0.0.0.0/0"
    
```

**Create SSH Key pair:**

首先创建SSH KEY：

```shell
aws ec2 create-key-pair --key-name  demo-key --query 'KeyMaterial' --output text > ~/.ssh/demo-key
```



[Create SSH KEY 参考](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/create-key-pairs.html#having-ec2-create-your-key-pair)

例如：

```shell
aws ec2 create-key-pair \
    --key-name my-key-pair \
    --key-type rsa \
    --key-format pem \
    --query "KeyMaterial" \
    --output text > my-key-pair.pem
```

set the permissions of your private key file :

```shell
chmod 400 key-pair-name.pem
```



-------------template-----------------------------------

1. **VPC ID**: `vpc-0d42bf2f27be967ff`
2. **Subnet ID**: `subnet-00b5ede5e160caa59`
3. **AMI ID**: `ami-0d70546e43a941d70`
4. **Security Group ID:** sg-063c02687e1103c7b
5. **Key name:** demo-key

-------------my----------------

VPC ID: vpc-010b0dfa3d852a08f

Subnet ID:subnet-0571dbe172ba5af78

AMI:ami-0f7b55661ecbbe44c (64-bit (x86))

Security Group ID:sg-0d4a83cb52de86321

**Key name:** demo-key

如何获取上述信息，请参考：

https://devopscube.com/use-aws-cli-create-ec2-instance/



```bash
#script.sh:
ubuntu@vm-ubuntu:~/UserData$ cat script.sh

#!/bin/bash
apt-get update -y
sudo apt install net-tools
sudo apt install nginx -y
sudo systemctl start nginx
```

DEMO CMD:



```shell
aws ec2 run-instances --image-id ami-0f7b55661ecbbe44c --count 1 --instance-type t2.micro --key-name demo-key --security-group-ids sg-0d4a83cb52de86321 --subnet-id subnet-0571dbe172ba5af78 --block-device-mappings "[{\"DeviceName\":\"/dev/sdf\",\"Ebs\":{\"VolumeSize\":30,\"DeleteOnTermination\":false}}]" --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=demo-server}]' 'ResourceType=volume,Tags=[{Key=Name,Value=demo-server-disk}]' --user-data file://~/UserData/script.sh
```

## create ec2 instance

![image-20231115101716842](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231115101716842.png)

MY CMD OUTPUT:

```shell
	 aws ec2 run-instances \
>     --image-id ami-0f7b55661ecbbe44c \
>     --count 1 \
>     --instance-type t2.micro \
>     --key-name demo-key2 \
>     --security-group-ids sg-0d4a83cb52de86321 \
>     --subnet-id subnet-0571dbe172ba5af78 \
>     --block-device-mappings "[{\"DeviceName\":\"/dev/sdf\",\"Ebs\":{\"VolumeSize\":30,\"DeleteOnTermination\":false}}]" \
>     --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=demo-server}]' 'ResourceType=volume,Tags=[{Key=Name,Value=demo-server-disk}]'
{
    "Groups": [],
    "Instances": [
        {
            "AmiLaunchIndex": 0,
            "ImageId": "ami-0f7b55661ecbbe44c",
            "InstanceId": "i-0f440a8141cfedca4",
            "InstanceType": "t2.micro",
            "KeyName": "demo-key",
            "LaunchTime": "2023-11-14T13:07:46.000Z",
            "Monitoring": {
                "State": "disabled"
            },
            "Placement": {
                "AvailabilityZone": "ap-northeast-1d",
                "GroupName": "",
                "Tenancy": "default"
            },
            "PrivateDnsName": "ip-172-31-18-31.ap-northeast-1.compute.internal",
            "PrivateIpAddress": "172.31.18.31",
            "ProductCodes": [],
            "PublicDnsName": "",
            "State": {
                "Code": 0,
                "Name": "pending"
            },
            "StateTransitionReason": "",
            "SubnetId": "subnet-0571dbe172ba5af78",
            "VpcId": "vpc-010b0dfa3d852a08f",
            "Architecture": "x86_64",
            "BlockDeviceMappings": [],
            "ClientToken": "1e564bac-02f4-41c0-920f-95b8b53cca60",
            "EbsOptimized": false,
            "EnaSupport": true,
            "Hypervisor": "xen",
            "NetworkInterfaces": [
                {
                    "Attachment": {
                        "AttachTime": "2023-11-14T13:07:46.000Z",
                        "AttachmentId": "eni-attach-02f736c79ff7c467e",
                        "DeleteOnTermination": true,
                        "DeviceIndex": 0,
                        "Status": "attaching"
                    },
                    "Description": "",
                    "Groups": [
                        {
                            "GroupName": "default",
                            "GroupId": "sg-0d4a83cb52de86321"
                        }
                    ],
                    "Ipv6Addresses": [],
                    "MacAddress": "0e:8c:79:cf:3c:b5",
                    "NetworkInterfaceId": "eni-05b509309ab871da8",
                    "OwnerId": "221093472068",
                    "PrivateDnsName": "ip-172-31-18-31.ap-northeast-1.compute.internal",
                    "PrivateIpAddress": "172.31.18.31",
                    "PrivateIpAddresses": [
                        {
                            "Primary": true,
                            "PrivateDnsName": "ip-172-31-18-31.ap-northeast-1.compute.internal",
                            "PrivateIpAddress": "172.31.18.31"
                        }
                    ],
                    "SourceDestCheck": true,
                    "Status": "in-use",
                    "SubnetId": "subnet-0571dbe172ba5af78",
                    "VpcId": "vpc-010b0dfa3d852a08f",
                    "InterfaceType": "interface"
                }
            ],
            "RootDeviceName": "/dev/sda1",
            "RootDeviceType": "ebs",
            "SecurityGroups": [
                {
                    "GroupName": "default",
                    "GroupId": "sg-0d4a83cb52de86321"
                }
            ],
            "SourceDestCheck": true,
            "StateReason": {
                "Code": "pending",
                "Message": "pending"
            },
            "Tags": [
                {
                    "Key": "Name",
                    "Value": "demo-server"
                }
            ],
            "VirtualizationType": "hvm",
            "CpuOptions": {
                "CoreCount": 1,
                "ThreadsPerCore": 1
            },
            "CapacityReservationSpecification": {
                "CapacityReservationPreference": "open"
            },
            "MetadataOptions": {
                "State": "pending",
                "HttpTokens": "optional",
                "HttpPutResponseHopLimit": 1,
                "HttpEndpoint": "enabled"
            }
        }
    ],
    "OwnerId": "221093472068",
    "ReservationId": "r-06c00811a46e0ab31"
}
```

![image-20231114212309382](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231114212309382.png)



## get-description instance

获取EC2 instance 命令：

```shell
 aws ec2 describe-instances --filters "Name=instance-state-name,Values=running"
```

输出截图：

![image-20231115101810350](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231115101810350.png)



List EC2 machines:



```
aws ec2 describe-instances --filters "Name=instance-state-name,Values=running" | grep "InstanceId"
```

```shell
"InstanceId": "i-01429b19ead5ba7bf",
                    "InstanceId": "i-0a58c21de0eeb6e1c",
                    "InstanceId": "i-016bde7982e69420e",
                    "InstanceId": "i-0d884a5b567102ee9",
                    "InstanceId": "i-0d528acf3ecc23965",
```

批量停止:

```shell
aws ec2 terminate-instances --instance-ids {i-016bde7982e69420e,i-0d884a5b567102ee9,i-0d528acf3ecc23965}
```



## **Terminate an Amazon EC2 instance**

get instance:

```shell
ubuntu@vm-ubuntu:~$  aws ec2 describe-instances --filters "Name=instance-state-name,Values=running"
{
    "Reservations": [
        {
            "Groups": [],
            "Instances": [
                {
                    "AmiLaunchIndex": 0,
                    "ImageId": "ami-0f7b55661ecbbe44c",
                    "InstanceId": "i-01297029a747da3f1",
                    "InstanceType": "t2.micro",
                    "KeyName": "demo-key2",
                    "LaunchTime": "2023-11-15T02:16:51.000Z",
                    "Monitoring": {
                        "State": "disabled"
                    },
```



This example terminates the specified instance.

Command:

```shell
aws ec2 terminate-instances --instance-ids i-1234567890abcdef0
```

输出：

```shell
ubuntu@vm-ubuntu:~$ aws ec2 terminate-instances --instance-ids i-01297029a747da3f1
{
    "TerminatingInstances": [
        {
            "CurrentState": {
                "Code": 32,
                "Name": "shutting-down"
            },
            "InstanceId": "i-01297029a747da3f1",
            "PreviousState": {
                "Code": 16,
                "Name": "running"
            }
        }
    ]
}
```

## Login EC2

```shell
ssh -i ~/.ssh/demo-key ubuntu@35.77.41.47 -v
```

```shell
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.15.0-1048-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Wed Nov 15 06:50:20 UTC 2023

  System load:  0.0               Processes:             97
  Usage of /:   21.4% of 7.57GB   Users logged in:       0
  Memory usage: 22%               IPv4 address for eth0: 172.31.26.14
  Swap usage:   0%


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
New release '22.04.3 LTS' available.
Run 'do-release-upgrade' to upgrade to it.


Last login: Wed Nov 15 06:45:42 2023 from 120.239.190.178
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

ubuntu@ip-172-31-26-14:~$
```



## Elastic IP Operation

## EIP Allocate:

```
aws ec2 allocate-address
```

```shell
ubuntu@vm-ubuntu:~$ aws ec2 allocate-address
{
    "PublicIp": "35.79.179.200",
    "AllocationId": "eipalloc-09bb2bd05bf81004f",
    "PublicIpv4Pool": "amazon",
    "NetworkBorderGroup": "ap-northeast-1",
    "Domain": "vpc"
}
```

![image-20231115154000625](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231115154000625.png)

## EIP Release:

```shell
aws ec2 release-address --allocation-id eipalloc-0714d911c50151979
```

## Associate EIP

```shell
aws ec2 associate-address --instance-id i-067392c87e231cd91 --public-ip 35.79.179.200
```

output:

{
    "AssociationId": "eipassoc-079dea3151990308d"
}

![image-20231115162659633](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231115162659633.png)



# Part 2:

## Python SSH Library Sample(TODO)



# Reference：

## Boto3 

https://boto3.amazonaws.com/v1/documentation/api/latest/guide/quickstart.html

[Delete-key-pair](https://docs.aws.amazon.com/cli/latest/reference/ec2/delete-key-pair.html)

[Create Key Pair](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/create-key-pairs.html#having-ec2-create-your-key-pair)

[How to Use AWS CLI to Create an EC2 instance](https://devopscube.com/use-aws-cli-create-ec2-instance/)
