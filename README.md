# Springboot + AWS Study Project :computer:
> 참고 : 스프링부트와 AWS로 혼자 구현하는 웹 서비스 (프리렉 / 이동욱 지음)
   
### INDEX
- [x] 스프링 부트 시작하기
- [x] 테스트 코드를 작성
- [x] JPA로 데이터베이스 다루기
- [x] Mustache 화면구성
- [x] 스프링 시큐리티와 OAuth2.0 로그인 기능
- [x] AWS 서버환경 - AWS EC2
- [x] AWS 데이터베이스환경 - AWS RDS
- [x] EC2 서버에 프로젝트 배포
- [ ] Travis CI 배포 자동화
- [ ] 무중단서비스

### AWS EC2

- EC2 : Elastic Compute Cloud (AWS에서 제공하는 성능, 용량 등을 유동적으로 사용할 수 있는 서버)
- 1대의 t2.micro만 사용하면 24시간 사용 가능
> 1. 서울 리전 선택
> 2. EC2 인스턴스 시작 
> 3. AMI (Amazon Machine Image) 선택 > Amazon Linux AMI   
EC2 인스턴스를 시작하는 데 필요한 정보를 이미지로 만들어 둔 것  
> 4. 스토리지 : 30GB 까지 프리티어로 가능
> 5. 태그 : 웹 콘솔에서 표기될 태그인 Name 태그를 등록 (본인 서비스의 인스턴스를 나타낼 수 있는 값)
> 6. 방화벽 : SSH (AWS EC2 에 터미널로 접속 ) > 본인 IP에서만 접근 가능하도록 구성하는 것이 안전함
- EIP (Elasic IP, AWS의 고정IP) 할당후 인스턴스와 연결

### EC2 서버 접속
``` shell
ssh -i {pem 키 위치} {EC2의 탄력적 IP주소}
```
- 손쉽게 접속할 수 있도록 설정하기
> ~/.ssh 로 pem 파일 이동 > ssh 실행시 자동으로 읽어 접속
``` shell
cp pem {pem 키 위치} ~/.ssh/
ll ~/.ssh/
```
> pem 키 권한 변경
``` shell
chmod 600 ~/.ssh/pem키이름
``` 
> pem 키가 있는 ~/.ssh 디렉토리에 config 파일 생성
``` shell
vim ~/.ssh/config
```

``` shell
# griffin-springboot2-webservice
Host 원하는서비스명
    HostName ec2의 탄력적IP주소
    User ec2-user
    IdentityFile ~/.ssh/pem키 이름
```
> config 파일 실행권한 설정
``` shell
chmod 700 ~/.ssh/config
```
> ec2 접속하기
``` shell
ssh {config에 등록한 서비스명}
```

#### 1. Java 8 설치
``` shell
sudo yum install -y java-1.8.0-openjdk-devel.x86_64
```
> 인스턴스의 Java 버전을 8로 변경
``` shell
sudo /usr/sbin/alternatives --config java
```
> 불필요한 java 가 있는 경우 삭제
``` shell
sudo yum remove java-1.7.0-openjdk
```
> java 버전 확인
``` shell
java -version
```
#### 2. 타임존변경
``` shell
sudo rm /etc/localtime
sudo ln -s /usr/share/zoneinfo/Asia/Seoul /etc/localtime

date
```
#### 3. 호스트네임 변경
> 각 서버가 어느 서비스인지 표현하기 위해 HOSTNAME 을 변경
``` shell
sudo vim /etc/sysconfig/network
```
> HOSTNAME 으로 되어있는 부분을 원하는 서비스명으로 변경
``` shell
HOSTNAME=griffin-springboot2-webservice
```
> /etc/hosts 에 변경한 hostname 등록
``` shell
sudo vim /etc/hosts

127.0.0.1  등록한HOSTNAME

curl 등록한 호스트 이름
```

### AWS RDS
#### 1. RDS 인스턴스 생성하기
> 1. DBMS - MariaDB 선택
> 2. 템플릿 - 프리티어
> 3. 네트워크 - 퍼블릭 엑세스 기능 on (지정된 IP만 접근 가능하도록 제어)

#### 2. RDS 운영환경에 맞는 파라미터 설정
> 1. 타임존 - Asia/Seoul
> 2. Character Set (utf8mb4 / utf8mb4_general_ci로 설정)
> - character_set_client
> - character_set_connection
> - character_set_database 
> - character_set_filesystem
> - character_set_results
> - character_set_server
> - collation_connection 
> - collation_server   
> 3. Max Connection (150으로 설정)
>
> 파라미터 변경 후, 해당 데이터베이스 옵션 수정 (DB 파라미터 그룹) / 재부팅

#### 3. 내 PC 에서 RDS 접속
> 1. 데이터베이스 보안그룹 선택   
> 2. EC2에 사용된 보안그룹의 그룹ID 를 RDS 보안그룹의 인바운드로 추가   
> MYSQL/Aurora | TCP | 3306 | 사용자지정 | {EC2 보안그룹ID}   
> MYSQL/Aurora | TCP | 3306 | 내 IP | {내 IP}
> 3. Database 플러그인 설치 (RDS정보 페이지에서 엔드포인트 확인)
``` shell
use griffin_springboot_webserivce;

show variables like 'c%';

alter database griffin_springboot_webserivce
CHARACTER SET = 'utf8mb4'
COLLATE = 'utf8mb4_general_ci'
;

select @@time_zone, now();

CREATE TABLE TEST (
    id bigint(20) not null auto_increment,
    content varchar(255) default null,
    primary key (id)
) engine = InnoDB;

insert into TEST(content) value ('테스트');

select * from TEST;
```

#### 4. EC2 에서 RDS 접근 확인
> EC2 에 MySQL CLI 설치
``` shell
ssh griffin-springboot2-webservice

sudo yum install mysql

mysql -u 계정 -p -h Host주소

mysql> show databases;
+-------------------------------+
| Database                      |
+-------------------------------+
| griffin_springboot_webserivce |
| information_schema            |
| innodb                        |
| mysql                         |
| performance_schema            |
| test                          |
+-------------------------------+
6 rows in set (0.00 sec)
```

### EC2 서버에 프로젝트 배포
#### 1. EC2 에 프로젝트 clone
``` shell
# git 설치
sudo yum install git
git --version

# 디렉토리 생성
mkdir ~/app && mkdir ~/app/step1
cd ~/app/step1

# git clone
git clone 프로젝트주소
```

- 테스트수행
``` shell
./gradlew test
```
> 성공
``` shell
BUILD SUCCESSFUL in 2m 26s
5 actionable tasks: 5 executed
```

> 실패 : -bash: ./gradlew: Permission denied
``` shell
# 실행 권한을 추가한 뒤 다시 테스트 진행
chmod +x ./gradlew
```

#### 2. 배포 스크립트 만들기
> ~/app/step1/deploy.sh
``` shell
REPOSITORY=/home/ec2-user/app/step1
PROJECT_NAME=griffin-springbbot2-webservice

cd /$REPOSITORY/$PROJECT_NAME/

echo "> Git Pull"
git pull

echo "> 프로젝트 Build 시작"
./gradlew build

echo "> step1 디렉토리 이동"
cd $REPOSITORY

echo "> Build 파일 복사"
cp $REPOSITORY/$PROJECT_NAME/build/libs/*.jar $REPOSITORY/

echo "> 현재 구동중인 애플리케이션 pid 확인"
CURRENT_PID=$(pgrep -f ${PROJECT_NAME}.*.jar)

echo "> 현재 구동중인 애플리케이션 pid : $CURRENT_PID"
if [ -z "$CURRENT_PID" ]; then
	echo "> 현재 구동중인 애플리케이션이 없으므로 종료하지 않습니다."
else
	echo "> kill -15 $CURRENT_PID"
	kill -15 $CURRENT_PID
	sleep 5
fi

echo "> 새 애플리케이션 배포"
JAR_NAME=$(ls -tr $REPOSITORY/ | grep jar | tail -n 1)

echo "> JAR Name: $JAR_NAME"
nohup java -jar $REPOSITORY/$JAR_NAME 2>&1 &

# nohup : 애플리케이션 실행자가 터미널을 종료해도 애플리케이션은 계속 구동
```
> deploy.sh 실행권한 추가 및 실행
``` shell
chmod +x ./deploy.sh

./deploy.sh

> Git Pull
이미 업데이트 상태입니다.
> 프로젝트 Build 시작

Deprecated Gradle features were used in this build, making it incompatible with Gradle 5.0.
Use '--warning-mode all' to show the individual deprecation warnings.
See https://docs.gradle.org/4.10.3/userguide/command_line_interface.html#sec:command_line_warnings

BUILD SUCCESSFUL in 4s
6 actionable tasks: 1 executed, 5 up-to-date
> step1 디렉토리 이동
> Build 파일 복사
> 현재 구동중인 애플리케이션 pid 확인
> 현재 구동중인 애플리케이션 pid : 
> 현재 구동중인 애플리케이션이 없으므로 종료하지 않습니다.
> 새 애플리케이션 배포
> JAR Name: griffin-springboot2-webservice-1.0-SNAPSHOT.jar
```

> 외부 Security 파일 등록하기
> - ClientId 와 ClientSecret 설정을 서버에 직접 가지고 있도록 설정
``` shell
vim /home/ec2-user/app/application-oauth.properties

# deploy 수정
nohup java -jar \-Dspring.config.location=classpath:/application.properties,/home/ec2-user/app/application-oauth.properties \/$REPOSITORY/$JAR_NAME 2>&1 &

```

#### 3. 스프링부트 프로젝트로 RDS 접근하기
1. 테이블 생성
``` sql
# 애플리케이션 테이블
Hibernate: create table posts (id bigint not null auto_increment, created_date datetime, modified_date datetime, author varchar(255), content TEXT not null, title varchar(500) not null, primary key (id)) engine=InnoDB
Hibernate: create table user (id bigint not null auto_increment, created_date datetime, modified_date datetime, email varchar(255) not null, name varchar(255) not null, picture varchar(255), role varchar(255) not null, primary key (id)) engine=InnoDB

# 스프링 세션 테이블 (schema-mysql.sql)
CREATE TABLE SPRING_SESSION (
                                PRIMARY_ID CHAR(36) NOT NULL,
                                SESSION_ID CHAR(36) NOT NULL,
                                CREATION_TIME BIGINT NOT NULL,
                                LAST_ACCESS_TIME BIGINT NOT NULL,
                                MAX_INACTIVE_INTERVAL INT NOT NULL,
                                EXPIRY_TIME BIGINT NOT NULL,
                                PRINCIPAL_NAME VARCHAR(100),
                                CONSTRAINT SPRING_SESSION_PK PRIMARY KEY (PRIMARY_ID)
) ENGINE=InnoDB ROW_FORMAT=DYNAMIC;

CREATE UNIQUE INDEX SPRING_SESSION_IX1 ON SPRING_SESSION (SESSION_ID);
CREATE INDEX SPRING_SESSION_IX2 ON SPRING_SESSION (EXPIRY_TIME);
CREATE INDEX SPRING_SESSION_IX3 ON SPRING_SESSION (PRINCIPAL_NAME);

CREATE TABLE SPRING_SESSION_ATTRIBUTES (
                                           SESSION_PRIMARY_ID CHAR(36) NOT NULL,
                                           ATTRIBUTE_NAME VARCHAR(200) NOT NULL,
                                           ATTRIBUTE_BYTES BLOB NOT NULL,
                                           CONSTRAINT SPRING_SESSION_ATTRIBUTES_PK PRIMARY KEY (SESSION_PRIMARY_ID, ATTRIBUTE_NAME),
                                           CONSTRAINT SPRING_SESSION_ATTRIBUTES_FK FOREIGN KEY (SESSION_PRIMARY_ID) REFERENCES SPRING_SESSION(PRIMARY_ID) ON DELETE CASCADE
) ENGINE=InnoDB ROW_FORMAT=DYNAMIC;

```
2. 프로젝트 설정
>  build.gradle 에 MariaDB 드라이버 등록
``` properties
compile('org.mariadb.jdbc:mariadb-java-client')
```
> application-real.properties 추가
``` properties
spring.profiles.include=oauth.real-db
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
spring.session.store-type=jdbc
```
3. EC2 설정
> app 디렉토리에 application-real-db.properties 생성
``` shell
vim ~/app/application-real-db.properties

# JPA 로 테이블이 자동생성되는 옵션을 none 설정 (RDS는 실제 운영 테이블이기 때문에 스프링부트에서 새로 만들면 안됨)
spring.jpa.hibernate.ddl-auto=none
spring.datasource.url=jdbc:mariadb://{rds주소}:{port}/{database명}
spring.datasource.username=db계정
spring.datasource.password=db계정 비밀번호
spring.datasource.driver-class-name=org.mariadb.jdbc.Driver
```
> deploy.sh 수정 (real profile 사용)
``` properties
nohup java -jar \-Dspring.config.location=classpath:/application.properties,/home/ec2-user/app/application-oauth.properties,/home/ec2-user/app/application-real-db.properties,classpath:/application-real.properties \-Dspring.profiles.active=real \/$REPOSITORY/$JAR_NAME 2>&1 &
```

#### 4. EC2 소셜로그인
0. AWS 보안그룹 변경
> EC2 보안그룹에 8080 포트가 열려있는지 확인
1. Google 에 EC2 주소 등록
> 구글웹콘솔 (https://console.cloud.google.com/home/dashboard)   
> API 및 서비스 > 사용자인증정보
- OAuth 동의 화면 > 승인된 도메인
> http:// 없이 EC2 퍼블릭 DNS 등록
- 사용자인증정보 > 서비스 선택
> 퍼블릭 DNS 주소에 {퍼블릭}:8080/login/oauth2/code/google 승인된 리디렉션 URL 등록    
2. Naver 에 EC2 주소 등록 
> 네이버 개발자 센터 (https://developers.naver.com/apps/#/myapps)
- 서비스 URL 
> 로그인을 시도하는 서비스가 네이버에 등록된 서비스인지 판단하기 위한 항목
> 8080 포트는 제외하고 실제 도메인 주소만 입력
> 1개의 주소만 허용 (수정 후 localhost 사용불가)
> 서비스 추가 후 새로운 키 발급받으면 localhost 추가 사용가능  
- Callback URL
> 전체주소 등록 
> {EC2퍼블릭 DNS}:8080/login/oauth2/code/naver