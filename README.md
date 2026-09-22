# codyssey3-1mission
https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/create-vpc.html#create-vpc-cli



- 최소권한법칙

<img width="1470" height="956" alt="스크린샷 2026-09-19 오전 5 14 43" src="https://github.com/user-attachments/assets/bef6d276-d9f3-4870-9597-f6724bb09e7c" />

<img width="742" height="78" alt="스크린샷 2026-09-19 오전 5 22 47" src="https://github.com/user-attachments/assets/3342e111-bcf3-4822-b179-173370ea5eb3" />



```


# 1. VPC 생성 (10.0.0.0/16 대역)
aws ec2 create-vpc --cidr-block 10.0.0.0/16 --query Vpc.VpcId --output text
vpc-0c20d57a2ed9c9ea1


# 2. Public Subnet 생성 (VPC 범위 내인 10.0.1.0/24 대역)
aws ec2 create-subnet --vpc-id vpc-0c20d57a2ed9c9ea1 --cidr-block 10.0.1.0/24 --query Subnet.SubnetId --output text
subnet-06320370147d23e9f


# 3. 서브넷 퍼블릭 IP 자동 할당 설정
aws ec2 modify-subnet-attribute --subnet-id subnet-06320370147d23e9f --map-public-ip-on-launch


# 4. 인터넷 게이트웨이 생성
aws ec2 create-internet-gateway --query InternetGateway.InternetGatewayId --output text
igw-0e7457dcd6af97068


# 5. 인터넷 게이트웨이를 VPC에 연결
aws ec2 attach-internet-gateway --vpc-id vpc-0c20d57a2ed9c9ea1 --internet-gateway-id igw-0e7457dcd6af97068


# 6. 퍼블릭 라우팅 테이블 생성
aws ec2 create-route-table --vpc-id vpc-0c20d57a2ed9c9ea1 --query RouteTable.RouteTableId --output text
rtb-049931ff7dec68685


# 7. 인터넷으로 나가는 경로(0.0.0.0/0) 추가
aws ec2 create-route --route-table-id rtb-049931ff7dec68685 --destination-cidr-block 0.0.0.0/0 --gateway-id igw-0e7457dcd6af97068
{
    "Return": true
}


# 8. 라우팅 테이블을 퍼블릭 서브넷과 연결
aws ec2 associate-route-table --route-table-id rtb-049931ff7dec68685 --subnet-id subnet-06320370147d23e9f
{
    "AssociationId": "rtbassoc-02674df1415d1b634",
    "AssociationState": {
        "State": "associated"
    }
}


```



```


MY_IP=$(curl -s https://checkip.amazonaws.com)
echo "내 공인 IP: $MY_IP"
내 공인 IP: 121.135.181.35


VCP_ID="vpc-0c20d57a2ed9c9ea1"


# 보안 그룹 생성 및 ID 저장
SG_ID=$(aws ec2 create-security-group --group-name new-sg --description "Security group for web server" --vpc-id $VPC_ID --query GroupId --output text)
echo "생성된 보안 그룹 ID: $SG_ID"
생성된 보안 그룹 ID: sg-0717498e423f9232f


# 1) HTTP (80) - 0.0.0.0/0 전체 허용
aws ec2 authorize-security-group-ingress --group-id $SG_ID --protocol tcp --port 80 --cidr 0.0.0.0/0
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-058b7ca765ed357b4",
            "GroupId": "sg-0717498e423f9232f",
            "GroupOwnerId": "018272776176",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 80,
            "ToPort": 80,
            "CidrIpv4": "0.0.0.0/0",
            "SecurityGroupRuleArn": "arn:aws:ec2:ap-northeast-2:018272776176:security-group-rule/sgr-058b7ca765ed357b4"
        }
    ]
}


# 2) SSH (22) - 본인 IP만 허용
aws ec2 authorize-security-group-ingress --group-id $SG_ID --protocol tcp --port 22 --cidr ${MY_IP}/32
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-00e04f628da57bfaa",
            "GroupId": "sg-0717498e423f9232f",
            "GroupOwnerId": "018272776176",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 22,
            "ToPort": 22,
            "CidrIpv4": "121.135.181.35/32",
            "SecurityGroupRuleArn": "arn:aws:ec2:ap-northeast-2:018272776176:security-group-rule/sgr-00e04f628da57bfaa"
        }
    ]
}



```  
  



  
```

# ED25519 키 페어 생성
aws ec2 create-key-pair --key-name ec2-key --key-type ed25519 --query 'KeyMaterial' --output text > ec2-key.pem


# Mac 키 파일 접근 권한 변경 (필수)
chmod 400 ec2-key.pem


# Subnet ID 입력 (예: subnet-0123456789abcdef0)
SUBNET_ID="subnet-06320370147d23e9f"


# Ubuntu 22.04 LTS 최신 AMI ID 자동 가져오기 (트러블 슈팅)
AMI_ID=$(aws ssm get-parameters --names /aws/service/canonical/ubuntu/server/22.04/stable/current/amd64/hvm/ebs-gp2/ami-id --query "Parameters[0].Value" --output text)

# E2C 전용 명령어로 AMI ID 가져오기 (대체)
AMI_ID=$(aws ec2 describe-images \
  --owners 099720109477 \
  --filters "Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*" "Name=state,Values=available" \
  --query "reverse(sort_by(Images, &CreationDate))[0].ImageId" \
  --output text)

echo "조회된 Ubuntu AMI ID: $AMI_ID"
조회된 Ubuntu AMI ID: ami-0621cf8f7a0902253


# EC2 인스턴스 시작
INSTANCE_ID=$(aws ec2 run-instances \
  --image-id $AMI_ID \
  --instance-type t3.micro \
  --key-name ec2-key \
  --security-group-ids $SG_ID \
  --subnet-id $SUBNET_ID \
  --associate-public-ip-address \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=my-web-server}]' \
  --query 'Instances[0].InstanceId' --output text)

echo "생성된 EC2 인스턴스 ID: $INSTANCE_ID"
생성된 EC2 인스턴스 ID: i-0711e55eda37b7b97  


# EC2 인스턴스가 running 상태가 될 때까지 10~20초 대기 후 실행
PUBLIC_IP=$(aws ec2 describe-instances --instance-ids $INSTANCE_ID --query 'Reservations[0].Instances[0].PublicIpAddress' --output text)

echo "EC2 퍼블릭 IP: $PUBLIC_IP"
EC2 퍼블릭 IP: 43.203.120.39  


# SSH 접속 (터미널에서 입력)
ssh -i ec2-key.pem ubuntu@$PUBLIC_IP


```


<img width="632" height="546" alt="스크린샷 2026-09-22 오후 10 47 15" src="https://github.com/user-attachments/assets/3cf6a608-3501-47c5-a810-dcf1c1e9cf42" />



```


# 패키지 업데이트 및 Nginx 설치
sudo apt update -y
sudo apt install nginx -y

# Nginx 실행 및 자동 시작 등록
sudo systemctl start nginx
sudo systemctl enable nginx
  
# [과제 요구사항 검증 1] EC2 내부 로컬 접속 확인 (200 응답 확인)
curl -I http://localhost

# [과제 요구사항 검증 2] 외부 아웃바운드 인터넷 통신 확인
curl -I https://naver.com


```
  
  
  
[과제 요구사항 검증 1] EC2 내부 로컬 접속 확인
<img width="742" height="144" alt="스크린샷 2026-09-22 오후 10 50 29" src="https://github.com/user-attachments/assets/e57a0c9d-e469-4a66-8142-cfdfef81a9e2" />
  
  
  
[과제 요구사항 검증 2] 외부 아웃바운드 인터넷 통신 확인
<img width="745" height="157" alt="스크린샷 2026-09-22 오후 10 50 04" src="https://github.com/user-attachments/assets/0969faeb-2f78-43c1-a7ed-3ad97e2c2288" />
  
  
  
____ 

https://app.diagrams.net/  
  
  
<img width="1146" height="634" alt="스크린샷 2026-09-18 오후 9 50 49" src="https://github.com/user-attachments/assets/b9a024c4-4de3-4010-baab-aecbf3c1de9d" />  
  
  
<img width="1470" height="956" alt="스크린샷 2026-09-18 오후 10 18 45" src="https://github.com/user-attachments/assets/06b92d0b-a66d-47bd-90a2-5f162370b7c6" />  


```
ubuntu@ip-10-0-1-11:~$ curl http://localhost
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```



<img width="734" height="433" alt="스크린샷 2026-09-19 오전 12 55 05" src="https://github.com/user-attachments/assets/09b36d23-07b8-43ae-b566-26a38bf7fd03" />   
  
  
  
  
  
  
(A) 브라우저로 http://<퍼블릭IP> 접속    
http://3.34.191.0   
http://43.203.120.39  
  
  
<img width="844" height="623" alt="스크린샷 2026-09-19 오전 12 58 02" src="https://github.com/user-attachments/assets/deabb1d6-4d58-4c59-b8be-68380c6d338c" />  
  
<img width="811" height="447" alt="스크린샷 2026-09-22 오후 10 55 33" src="https://github.com/user-attachments/assets/a1680578-bb4f-4df0-a33e-316ac765453a" />
  







