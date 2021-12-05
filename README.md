# Create-a-Virtual-Machine-in-AWS
### VPC
aws ec2 create-default-vpc


### adding the keypair: 
aws ec2 import-key-pair --key-name "my-key" --public-key-material fileb://~/.ssh/id_rsa.pub


### create security group:
aws ec2 create-security-group \
	--description "Allow ssh and icmp" \
	--group-name MySecurityGroup

# allow ssh
aws ec2 authorize-security-group-ingress \
    --group-name MySecurityGroup \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0
	
# allow icmp
aws ec2 authorize-security-group-ingress \
    --group-name MySecurityGroup \   
    --ip-permissions IpProtocol=icmp,FromPort=-1,ToPort=-1,IpRanges='[{CidrIp=0.0.0.0/0}]'


### Instances:
aws ec2 run-instances \
	--image-id ami-09042b2f6d07d164a \
	--count 1 \
	--instance-type t2.micro \
	--key-name my-key \
	--security-groups MySecurityGroup
	
	

### get instance adress
EC2_DNS_ADRESS=$(aws ec2 describe-instances | grep --m 1 -oP '(?<="PublicDnsName": ")[^"]*')

### upload bench.sh
scp -i ~/.ssh/id_rsa bench.sh ubuntu@$EC2_DNS_ADRESS:/home/ubuntu # upload to machine

### login to ec2 instance
ssh -i "~/.ssh/id_rsa" ubuntu@$EC2_DNS_ADRESS

### install newest version of sysbench
sudo curl -s https://packagecloud.io/install/repositories/akopytov/sysbench/script.deb.sh | sudo bash
sudo apt -y install sysbench

### Add cronjob: use "which sh"
echo "*/30 * * * * /bin/sh /home/$USER/bench.sh" | crontab
