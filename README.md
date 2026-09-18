# codyssey3-1mission
https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/create-vpc.html#create-vpc-cli


```

# 1. VPC 생성 (10.0.0.0/16 대역)
aws ec2 create-vpc --cidr-block 10.0.0.0/16 --query Vpc.VpcId --output text

# 2. Public Subnet 생성 (VPC 범위 내인 10.0.1.0/24 대역)
aws ec2 create-subnet --vpc-id <VPC_ID> --cidr-block 10.0.1.0/24 --query Subnet.SubnetId --output text

# 3. 서브넷 퍼블릭 IP 자동 할당 설정
aws ec2 modify-subnet-attribute --subnet-id <SUBNET_ID> --map-public-ip-on-launch

# 4. 인터넷 게이트웨이 생성
aws ec2 create-internet-gateway --query InternetGateway.InternetGatewayId --output text

# 5. 인터넷 게이트웨이를 VPC에 연결
aws ec2 attach-internet-gateway --vpc-id <VPC_ID> --internet-gateway-id <IGW_ID>

# 6. 퍼블릭 라우팅 테이블 생성
aws ec2 create-route-table --vpc-id <VPC_ID> --query RouteTable.RouteTableId --output text

# 7. 인터넷으로 나가는 경로(0.0.0.0/0) 추가
aws ec2 create-route --route-table-id <ROUTE_TABLE_ID> --destination-cidr-block 0.0.0.0/0 --gateway-id <IGW_ID>

# 8. 라우팅 테이블을 퍼블릭 서브넷과 연결
aws ec2 associate-route-table --route-table-id <ROUTE_TABLE_ID> --subnet-id <SUBNET_ID>

```


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



<img width="1105" height="118" alt="스크린샷 2026-09-19 오전 12 56 30" src="https://github.com/user-attachments/assets/6286c9de-fa6e-4c7a-9a55-a8d50d76e8cc" />  



<img width="844" height="623" alt="스크린샷 2026-09-19 오전 12 58 02" src="https://github.com/user-attachments/assets/deabb1d6-4d58-4c59-b8be-68380c6d338c" />  

(A) 브라우저로 http://<퍼블릭IP> 접속   
http://3.34.191.0  


<img width="793" height="466" alt="스크린샷 2026-09-19 오전 1 33 39" src="https://github.com/user-attachments/assets/86bf38e8-405b-4b95-84b3-de6a379692aa" />







