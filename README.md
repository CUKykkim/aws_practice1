# aws_실습 1

## S3

### Web을 통한 S3 파일 관리

- aws Console 로 접속 (https://console.aws.amazon.com/)

- aws s3 서비스로 이동

![s3_access](./images/s3_access.jpg)

- 버킷 만들기

![makebucket](./images/makebucket.jpg)


- 버킷 이름 만들기
  * 버킷 버전 관리 활성화

![bucketname](./images/bucketname.jpg)

![version](./images/version.jpg)

- 버킷에 파일 업로드

![fileupload](./images/fileupload.jpg)

- 객체 확인

![object](./images/object.jpg)

- 같은 객체를 다시 업로드

- 버전 확인하기

![secondversion](./images/secondversion.jpg)


### AWS 의 IAM 생성

- IAM(AWS Identity and Access Management)은 AWS 리소스에 대한 액세스를 안전하게 제어할 수 있는 웹 서비스
- IAM을 사용하여 리소스를 사용하도록 인증 및 권한 부여된 대상을 제어

- IAM 서비스로 접속

- 사용자 메뉴에서 사용자 추가 선택

![adduser](./images/adduser.jpg)

- 신규 사용자의 이름을 원하는 대로 입력

- 액세스 유형은 액세스 키 – 프로그래밍 방식 액세스 체크하고 [다음]을 클릭

![iam_access_key](./images/iam_access_key.jpg)

- 권한 설정에서 그룹을 생성
- 그룹에 권한을 설정하고 유저를 추가하면, 일괄적으로 관리할 수 있어 편리

![create_group](./images/create_group.jpg)

- 그룹 이름을 만들고 ‘AdministratorAccess’를 체크하여 관리자 권한을 해당 그룹에 부여함

![admin](./images/admin.jpg)

- 그후 생성된 관리자 그룹에 사용자를 추가

![add_user](./images/add_user.jpg)


- 생성된 엑세스 키ID 및 비밀 엑세스 키는 반드시 CSV 형태로 다운받아 보관해야함

![accesskey](./images/accesskey.jpg)

### EC2를 통한 S3 파일 접근

- ec2 서비스로 이동

- ec2 인스턴스 생성
  * Amazon Linux 2 AMI (HVM), SSD Volume Type - ami-0e4a9ad2eb120e054 (64비트 x86) / ami-0b71a05eb4f283fab (64비트 Arm) AMI 선택

  * t2.micro 선택
  * 인스턴스 시작

- terminal을 활용하여 인스턴스 연결


- aws configure 명령어를 이용해 만들어 놓은 IAM 권한을 설정

```
aws configure
```

- 이전에 저장해 놓은 IAM의 .csv 파일에 기록된 엑세스 키 및 비밀 엑세스 키를 입력

- 버킷 만들기

```
aws s3 mb s3://cuk-test1
```
- 버킷 및 객체 리스트 보기

```
aws s3 ls
```


```
aws s3 ls s3://cuk-test1
```

- 객체 복사


```
aws s3 cp s3.txt s3://cuk-test1
```

```
aws s3 cp s3://cuk-test1/iris.csv iris.csv
```

- 객체 삭제

```
aws s3 rm s3://cuk-test1/iris.csv --recursive
```

## ECR 실습

- 실습 전 로컬 PC에 AWS CLI 설치
- Ubuntu 상에서 AWS CLI 설치

```
sudo apt update
sudo apt install curl
sudo apt install unzip
sudo curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo unzip awscliv2.zip
sudo ./aws/install
```

- aws cli가 설치 완료되었는지를 다음 명령어로 확인

```
aws --version
```


- 만약 실습 pc가 macOS 인 경우 terminal에서 다음 명령어를 수행

```
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
sudo installer -pkg AWSCLIV2.pkg -target /
```

- docker의 기존 계정 로그인 정보 삭제

```
 vi  ~/.docker/config.json
```

- aws configure 명령어를 이용해 만들어 놓은 IAM 권한을 설정

```
aws configure
```


- AWS Console에서 ECR 서비스로 이동

- 퍼블릭 리포지토리 생성

![public_repo](./images/public_repo.jpg)

- 리포지토리 이름 및 운영체제 선택후 리포지토리 생성

![reponame](./images/reponame.jpg)

- 리포지토리 URI를 기반으로 도커 이미지를 리포지토리에 업로드해 보기


- 우분투 터미널에서 기존 docker의 login 정보를 제거한다. 

```
vi  ~/.docker/config.json
```

![ecr](./images/ecr.png)

```
"credsStore": "desktop.exe"     ## 제거
```


- docker login으로 리포지토리에 대한 push 권한 획득

```
aws ecr-public get-login-password --region us-east-1  | docker login --username AWS --password-stdin public.ecr.aws/m6t0g4k12
```

- nginx 이미지를 다운 받는다.

```
docker run -d -p 8080:80 nginx:latest
```

- nginx container로 진입

```
docker exec -it [container_name or id] /bin/bash
```

- 웹 페이지 수정

```
vi /usr/share/nginx  html/index.html
```

- 컨테이너를 빠져 나온후 컨테이너 commit

```
docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]

```


- windows powershell 을 열어 docker의 기존 이미지를 생성했던 리포지토리 네임으로 태깅한다.

```
docker tag nginx:latest public.ecr.aws/z5y8t7e2/cuk-repo:latest
```

- docker 이미지를 푸시 한다.

```
docker push public.ecr.aws/z5y8t7e2/cuk-repo:latest
```


## EKS 클러스터 생성 실습
- 본 실습에는 비용이 과금될 수 있으므로, 시연을 보는 것으로 함께 하시길 바랍니다.

- 우분투상에서 eksctl 설치

- temp 폴더에 바이너리 파일 다운로드 진행

```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
```

- 다운로드 된 것을 확인후  /usr/local/bin 으로 파일 이동

```
cd /tmp
ls -al

sudo mv -v /tmp/eksctl /usr/local/bin
```

- eksctl 버전확인

```
eksctl version
```



- eksctl을 이용한 kubernetes 클러스터 구축

```
 eksctl create cluster --name node-example --nodes-min 1 --nodes-max 2 --nodes 1 --node-type m5.xlarge --asg-access --node-volume-size 100
```


- kuberenetes 의 context를 로컬 pc로 가져오기

```
aws eks --region ap-northeast-2 update-kubeconfig --name node-example
```

- aws에 구축된 kubernetes 정상 동작 확인

```
kubectl get nodes
```