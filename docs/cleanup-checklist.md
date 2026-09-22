
1. EC2 인스턴스
2. 보안 그룹
3. 인터넷 게이트웨이
4. 서브넷
5. VPC
6. 키 페어 등 잔여 리소스 정리

```

1.  
aws ec2 terminate-instances --instance-ids $INSTANCE_ID
{
    "TerminatingInstances": [
        {
            "InstanceId": "i-0711e55eda37b7b97",
            "CurrentState": {
                "Code": 32,
                "Name": "shutting-down"
            },
            "PreviousState": {
                "Code": 16,
                "Name": "running"
            }
        }
    ]
}
  
  
2.
aws ec2 delete-security-group --group-id $SG_ID
{
    "Return": true,
    "GroupId": "sg-0717498e423f9232f"
}
  
  
3.
aws ec2 detach-internet-gateway --vpc-id $VPC_ID --internet-gateway-id $IGW_ID
aws ec2 delete-internet-gateway --internet-gateway-id $IGW_ID
  
  
4.
aws ec2 delete-subnet --subnet-id $SUBNET_ID
  

5. 
aws ec2 delete-vpc --vpc-id $VPC_ID
  

6. AWS CLI, 키 파일 삭제
aws ec2 delete-key-pair --key-name ec2-key
rm ec2-key.pem
  
```