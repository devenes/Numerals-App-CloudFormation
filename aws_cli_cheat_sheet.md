# AWS CLI Cheat Sheet

```bash
sudo hostnamectl set-hostname aws-cli
```

```bash
bash
```

```bash
aws --version
```

## Install AWS CLI Version 2

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
/usr/local/bin/aws --version
which aws
```

```bash
aws configure
```

```bash
Access key ID: XXXXXXXXXXXXXXXXXXX
Secret access key: XXXXXXXXXXXXXXXXXXXXXXXXXXXXX
Default region name [None]: us-east-1
Default output format [None]: json
```

### Check AWS CLI Configuration

```bash
aws sts get-caller-identity --query Account --output text
```

### Create Security Groups

```bash
aws ec2 create-security-group \
 --group-name roman_numbers_sec_grp \
 --description "This Sec Group is to allow ssh and http from anywhere"
```

Check Security Groups with the following command:

```bash
aws ec2 describe-security-groups --group-names roman_numbers_sec_grp
```

You can check IP Addresses of the Security Group with the following command:

```bash
curl https://checkip.amazonaws.com
```

### Create Rules for the Security Groups

```bash
aws ec2 authorize-security-group-ingress \
 --group-name roman_numbers_sec_grp \
 --protocol tcp \
 --port 22 \
 --cidr 0.0.0.0/0
```

```bash
aws ec2 authorize-security-group-ingress \
--group-name roman_numbers_sec_grp \
--protocol tcp \
--port 80 \
--cidr 0.0.0.0/0
```

### Create EC2 Instance

After creating Security Groups, we will create our EC2 instances.

- Latest AMI id should be used.

### Get the latest AMI id with the following command:

```bash
aws ssm get-parameters --names /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 --region us-east-1
```

### Get the latest AMI id with using query:

```bash
aws ssm get-parameters --names /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 --query 'Parameters[0].[Value]' --output text
```

### Assign the value to a variable:

```bash
LATEST_AMI=$(aws ssm get-parameters --names /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 --query 'Parameters[0].[Value]' --output text)
```

```bash
echo $LATEST_AMI
```

### Create `userdata.sh` file:

```bash
sudo vim userdata.sh
```

### Run EC2 Instance with `userdata.sh` file:

```bash
aws ec2 run-instances --image-id $LATEST_AMI --count 1 --instance-type t2.micro --key-name xxxxxxx --security-groups roman_numbers_sec_grp --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=roman_numbers}]' --user-data file:///Users/ODG/Desktop/git_dir/devenes-cw/porfolio_lesson_plan/week_6/CLI_solution/userdata.sh
```

### Or use the following command:

```bash
aws ec2 run-instances \
 --image-id $LATEST_AMI \
 --count 1 \
 --instance-type t2.micro \
 --key-name devenes \
 --security-groups roman_numbers_sec_grp \
 --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=roman_numbers}]' \
 --user-data file:///home/ec2-user/userdata.sh
```

### To see the each instances Ip we'll use `describe instance` CLI command:

```bash
aws ec2 describe-instances --filters "Name=tag:Name,Values=roman_numbers"
```

### You can run the query to find `Public IP` and `Instance ID` of instances:

```bash
aws ec2 describe-instances --filters "Name=tag:Name,Values=roman_numbers" --query 'Reservations[].Instances[].PublicIpAddress[]'
```

```bash
aws ec2 describe-instances --filters "Name=tag:Name,Values=roman_numbers" --query 'Reservations[].Instances[].InstanceId[]'
```

### Delete Instances:

```bash
aws ec2 terminate-instances --instance-ids xxxxxxxxxxxx
```

### Delete Security Groups:

```bash
aws ec2 delete-security-group --group-name roman_numbers_sec_grp
```

### Resources

https://docs.aws.amazon.com/cli/latest/userguide
