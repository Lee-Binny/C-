# AWS에 배포하기

### EC2 인스턴스 생성
- 리전 선택(한국 추천) > EC2 > 인스턴스 시작 > 1. Amazon Machine Image(AMI) 선택 > 2. 인스턴스 유형 선택 -  > 6. 보안그룹 구성 - HTTP, HTTPS 추가 > 
기존 키 페어 선택 또는 새 키 페어 생성(pem) - 새 키 페어 생성, 키 페어 이름 입력 > pem 키를 프로젝트에 넣고 .gitignore에 `키페어이름.pem` 코드를 추가하여 공개되지 않도록 해야한다. 

  
### github 레퍼지토리에 코드 올리기
- github에서 레퍼지토리 생성 > repository 링크 복사 > 프로젝트 터미널에서 `git remote add orgin [레퍼지토리 링크]` 명령어 실행 > `git add.` 명령어 실행 > 
`git commit -m "prepare for aws"` 명령어 실행 > `git push origin master` 명령어 실행
> **git commit -am**  
  `git add.` 과 `git commit -m`을 합친 명령어
  
    
### git과 EC2 연결하기 (프론트 배포)
- EC2 인스턴스 > 연결 > 인스턴스 연결창의 ssh 복사 > 프로젝트 터미널에 붙여넣기 > ubuntu 서버에서 `git clone [레퍼지토리 링크]` 명령어 실행 > ubuntu에 node 설치하기 > `npm run build` 명령어 실행

  
### ubuntu에 노드 설치하기
```
$ sudo apt-get update
$ sudo apt-get install -y build-essential
$ sudo apt-get install curl
$ curl -sL https://dep.nodesource.com/setup_10.x | sudo -E bash --
$ sudo apt-get install -y nodejs
```

### ubuntu에 mysql 설치하기
```
$ wget -c https://repo.mysql.com//mysql-apt-config_0.8.13-1_all.deb
$ sudo dpkg -i mysql-apt-config_0.8.13-1_all.deb
$ sudo apt-get update
$ sudo apt-get install -y mysql-server

```

**mysql 접속하기**
- `sudo su` 명령어로 root 계정으로 전환 > mysql_secure_installation > 설정, 패스워드 입력 > `mysql -uroot -p` 명령어 실행 후 패스워드 입력

### git과 EC2 연결하기 (서버 배포)
- EC2 인스턴스 > 연결 > 인스턴스 연결창의 ssh 복사 > 프로젝트 터미널에 붙여넣기 > ubuntu 서버에서 `git clone [레퍼지토리 링크]` 명령어 실행 > ubuntu에 node, mysql 설치하기 > `vim .env` 명령어로 .env 파일 생성 > `npx sequelize db:create` 명령어 실행 > `vim app.js` 명령어 실행 후 port 병경 > `npm run start` 명령어 실행

### PM2
- `node app.js` 명령어로 실행하면 프로젝트는 foreground에서 시작되는데, 터미널을 종료하면 프로젝트도 종료되는 문제 발생
- 이러한 문제를 막기 위해 프로젝트를 background에서 실행해주는 pm2 사용
- `npm i pm2` 명령어로 설치
- package.js 파일의 start script 명령어를 `pm2 start app.js`로 수정
- `npx pm2 logs` 명령어 실행 시 로그 확인 가능 (`npx pm2 logs --error`는 에러 로그 확인)
- `npx pm2 list` 명령어 실행 시 실행중인 프로세스 확인 가능
- `npx pm2 kill` 명령어 실행 시 실행중인 프로세스 종료

### 상용 환경으로 변경하기
- start 스크립트를 `cross-env NODE_ENV=production pm2 start app.js`로 변경

### 프론트 배포하기
- 프론트가 서버 사이드 렌더링 코드라서 서버가 실행되지 않으면 프론트도 실행되지 않음
- 코드에 localhost로 적은 부분 EC2 인스턴스 ip로 변경 필요

**/front/config/config.js**
```javascript
// 인스턴스 ip 입력
export const backUrk = 'http://34.232.77.127';
```
- start 스크립트를 `cross-env NODE_ENV=production next start -p 80`으로 변경
- 서버 코드 cors에 프론트 인스턴스 주소 추가
- `npm run build` 명령어 실행 > `npx pm2 start npm -- start`명령어로 실행  

### 도메인 연결하기
- 인스턴스의 ip 주소가 계속 변경되고 cors 문제로 인해 도메인 연결 추천
- 도메인 구입하기 > AWS Route53에서 호스팅 영역 생성 후 도메인 입력 > 생성된 호스팅의 네임서버를 도메인 구입 사이트에 입력 > AWS EC2에서 탄력적 IP 주소 할당받기 > 인스턴스 연결하기 > 
Route53 호스팅에서 프론트 - 도메인.com 레코드 생성, 유형 A 선택, 값에 프론트 ip 입력 / 백엔드 - api.도메인.com 레코드 생성, 유형 A 선택, 값에 백엔드 ip 입력, www.도메인.com 레코드 생성, CNAME 유형 선택, 값에 도메인.com 입력

### S3 연결하기
- 버킷 만들기 > 생성된 버킷 정책에 아래 코드 입력 > 내 보안 자격 증명 클릭 > 액세스 키 만들기 > 키 파일 다운로드 > 프로젝트에서 `npm i multer-s3 aws-sdk` 명령어 실행 > .env 파일에 S3_ACCESS_KEY_ID, S3_SECRET_ACCESS_KEY 추가 > app.js 파일 변경
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AddPerm",
      "Effect": "Allow",
      "Principal": "*",
      "Action": [
        "s3:GetObject",
        "s3:PubObject"
      ],
      "Resource": "arn:aws:s3:::[생성한 버킷명]/*"
    }
  ]
}
```

**/back/app.js**
```javascript
...
const multerS3 = require('multer-s3');
const AWS = require('aws-sdk');
...

AWS.config.update({
  accessKeyId: process.env.S3_ACCESS_KEY_ID,
  secretAccessKey: process.env.S3_SECRET_ACCESS_KEY,
  region: 'ap-northeast-2',
});

const upload = multer({
  storage: multerS3({
    s3: new AWS.S3(),
    bucket: 'react-nodebird-s3',
    key(req, file, db) {
      cb(null, `original/${Date.now()}_${path.basename(file.originalname)}`)
    }
  }),
  limits: { fileSize: 20 * 1024 * 1024 },
});
...

router.post('/images', isLoggedIn, upload.array('image'), (req, res, next) => { // POST /post/images
  console.log(req.files);
  // v.filename을 v.location으로 변경
  res.json(req.files.map((v) => v.filename));
});
...

```

### Lambda 이미지 리사이징
- 여러 가지 API 처리를 하는 서버 대신에 이미지 리사이징을 해주는 작은 함수
- 이미지를 압축헤서 S3에 저장
- lambda 디렉토리 생성 > `npm init` 명렁어 실행 > `npm i aws-sdk sharp` 명령어 실행 > index.js 파일 생성 후 코드 입력 > `sudo apt install zip` 명령어 실행 > 
`sudo zip -r aws-upload.zip ./*` 명령어 실행 > `sudo curl "https://awscli.amazoneaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"` 명령어 실행 > 
`sudo unzip awscliv2.zip` 명령어 실행 > `sudo ./aws/install` 명렁어 실행 > `aws configure` 명령어 실행 후 S3 엑세스 키 입력 > `aws s3 cp "aws-upload.zip" s3://[생성한 s3 버킷명]` 명령어 실행 > AWS lambdad에서 image-resize 함수 생성 > 함수 코드 작업에서 AWS S3에서 파일 업로드 클릭 > s3에 올라간 aws-upload.zip 파일 선택 > 트리거 추가엣 s3 추가 > 접두사에 `original/` 입력하기 

**/lambda/index.js**
```javascript
const AWS = require('aws-sdk');
const sharp = require('sharp');

const s3 = new AWS.S3();

exports.handler = async (evnet, context, callback) => {
  const Bucket = event.Records[0].s3.bucket.name; // 생성한 s3 버킷 이름
  const Key = decodeURIComponent(event.Records[0].s3.object.key); 파일 이름
  const filename = Key.split('/')[Key.split('/').length - 1];
  const ext = Key.split('.')[Key.split('.').length - 1].toLowerCase();
  // jpg 확장자느 jpeg로 변경
  const requiredFormat = ext === 'jpg' ? 'jpeg' : ext;
  
  try {
    const s3Object = await s3.getObject({ Bucket, Key }).promise();
    const resizedImage = await sharp(s3Object.Body)
      .resize(400, 400, { fit: 'inside' })
      .toFormat(requireFormat)
      .toBuffer();
    await s3.putObject({
      Bucket,
      Key: `thumb/${filename}`,
      Body: redizedImage,
    }).promise();
  } catch (error) {
    console.log(error);
    return callback(error);
  }
```
