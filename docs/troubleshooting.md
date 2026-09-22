## troubleshooting

증상 : SSM Parameter Store를 이용한 최신 AMD ID 자동 조회 실패(ssm:GetParameters AccessDeniedException)

가설 : IAM 사용자 권한을 최소화 했기 때문에 SSM 서비스 접근 권한이 없어서 발생 했을 것이다.

검증 : AccessDenied 발생 확인.

조치 : SSM 명령어 대신 부여된 EC2 권한 범위 내의 aws ec2 describe-images 명령어로 변경하여 AMI ID 조회 성공.

결과 : 한 오남용 없이 최소 권한을 유지한 상태에서 원하는 AMI ID 확보 및 인스턴스 생성 진행 완료.