```bash
sudo hostnamectl set-hostname aws-cli
```

```bash
aws --version
```

- version2 yi yükleyelim:

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

- konfigürasyonun tamamlandığını kontrol edelim:

  ```bash
  aws sts get-caller-identity --query Account --output text
  ```

1. Security Groupların oluşturulması:

```bash
aws ec2 create-security-group \
 --group-name roman_numbers_sec_grp \
 --description "This Sec Group is to allow ssh and http from anywhere"
```

We can check the security Group with these commands:

```bash
aws ec2 describe-security-groups --group-names roman_numbers_sec_grp
```

You can check IPs with this command into the EC2:

```bash
curl https://checkip.amazonaws.com
```

2. Create Rules

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

3. After creating security Groups, We will create our EC2s. Latest AMI id should be used.

Bu en son AMIyi bulmak için kullanılacak komuttur.

- ilk olarak son ami bilgilerini çekelim

```bash
aws ssm get-parameters --names /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 --region us-east-1
```

- ikinci aşamada query çalıştırarak son ami numarasını elde edelim

```bash
aws ssm get-parameters --names /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 --query 'Parameters[0].[Value]' --output text
```

- sonra bu değeri bir variable a atayalım:

```bash
LATEST_AMI=$(aws ssm get-parameters --names /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 --query 'Parameters[0].[Value]' --output text)
```

```bash
echo $LATEST_AMI
```

- userdata dosyamızı oluşturalım

```bash
sudo vim userdata.sh
```

- şimdi de instance ı çalıştıralım:

```bash
aws ec2 run-instances --image-id $LATEST_AMI --count 1 --instance-type t2.micro --key-name xxxxxxx --security-groups roman_numbers_sec_grp --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=roman_numbers}]' --user-data file:///Users/ODG/Desktop/git_dir/devenes-cw/porfolio_lesson_plan/week_6/CLI_solution/userdata.sh
```

veya

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

- To see the each instances Ip we\'ll use describe instance CLI command:

```bash
aws ec2 describe-instances --filters "Name=tag:Name,Values=roman_numbers"
```

- You can run the query to find Public IP and instance_id of instances:

```bash
aws ec2 describe-instances --filters "Name=tag:Name,Values=roman_numbers" --query 'Reservations[].Instances[].PublicIpAddress[]'
```

```bash
aws ec2 describe-instances --filters "Name=tag:Name,Values=roman_numbers" --query 'Reservations[].Instances[].InstanceId[]'
```

- To delete instances:

  ```bash
  aws ec2 terminate-instances --instance-ids xxxxxxxxxxxx
  ```

- To delete security groups:

  ```bash
  aws ec2 delete-security-group --group-name roman_numbers_sec_grp
  ```

### Resources

https://docs.aws.amazon.com/cli/latest/userguide
