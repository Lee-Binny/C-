# AWS에 배포하기

**EC2 인스턴스 생성**  
- 리전 선택(한국 추천) > EC2 > 인스턴스 시작 > 1. Amazon Machine Image(AMI) 선택 > 2. 인스턴스 유형 선택 -  > 6. 보안그룹 구성 - HTTP, HTTPS 추가 > 
기존 키 페어 선택 또는 새 키 페어 생성(pem) - 새 키 페어 생성, 키 페어 이름 입력 
> pem 키를 프로젝트에 넣고 .gitignore에 `키페어이름.pem` 코드를 추가하여 공개되지 않도록 해야한다. 

  
**github 레퍼지토리에 코드 올리기**  
- github에서 레퍼지토리 생성 > repository 링크 복사 > 프로젝트 터미널에서 `git remote add orgin [레퍼지토리 링크]` 명령어 실행 > `git add.` 명령어 실행 > 
`git commit -m "prepare for aws"` 명령어 실행 > `git push origin master` 명령어 실행
> **git commit -am** 
  `git add.` 과 `git commit -m`을 합친 명령어
  
    
**git과 EC2 연결하기 (프론트 배포)**
- EC2 인스턴스 > 연결 > 인스턴스 연결창의 ssh 복사 > 프로젝트 터미널에 붙여넣기 > ubuntu 서버에서 `git clone [레퍼지토리 링크]` 명령어 실행 > ubuntu에 node 설치하기 >
`npm run build` 명령어 실행

  
**ubuntu에 노드 설치하기**
```
$ sudo apt-get update
$ sudo apt-get install -y build-essential
$ sudo apt-get install curl
$ curl -sL https://dep.nodesource.com/setup_10.x | sudo -E bash --
$ sudo apt-get install -y nodejs
```

**ubuntu에 mysql 설치하기**
```
$ wget -c https://repo.mysql.com//mysql-apt-config_0.8.13-1_all.deb
$ sudo dpkg -i mysql-apt-config_0.8.13-1_all.deb
$ sudo apt-get update
$ sudo apt-get install -y mysql-server

```

**mysql 접속하기**
- `sudo su` 명령어로 root 계정으로 전환 > mysql_secure_installation > 설정, 패스워드 입력 > `mysql -uroot -p` 명령어 실행 후 패스워드 입력

**git과 EC2 연결하기 (서버 배포)**
- EC2 인스턴스 > 연결 > 인스턴스 연결창의 ssh 복사 > 프로젝트 터미널에 붙여넣기 > ubuntu 서버에서 `git clone [레퍼지토리 링크]` 명령어 실행 > ubuntu에 node, mysql 설치하기 
> `vim .env` 명령어로 .env 파일 생성 > `npx sequelize db:create` 명령어 실행 > `vim app.js` 명령어 실행 후 port 병경 > `npm run start` 명령어 실행

**PM2**
- `node app.js` 명령어로 실행하면 프로젝트는 foreground에서 시작되는데, 터미널을 종료하면 프로젝트도 종료되는 문제 발생
- 이러한 문제를 막기 위해 프로젝트를 background에서 실행해주는 pm2 사용
- `npm i pm2` 명령어로 설치
- package.js 파일의 start script 명령어를 `pm2 start app.js`로 수정
- `npx pm2 logs` 명령어 실행 시 로그 확인 가능 (`npx pm2 logs --error`는 에러 로그 확인)
- `npx pm2 list` 명령어 실행 시 실행중인 프로세스 확인 가능
- `npx pm2 kill` 명령어 실행 시 실행중인 프로세스 종료
