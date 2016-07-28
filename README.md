# my-maven-repository

## 나만의 메이븐 저장소 만들기
프로젝트를 진행하다가 컴포넌트가 늘어난다면 여러 컴포넌트간 공통 로직이 발생한다.
공통로직을 하나의 컴포넌트를 만들고 다른 컴포넌트들에서 그걸 땡겨쓰도록 하면 되겠다.
이런걸 좀 쉽게 하기 위해 나만의 메이븐 저장소를 하나 구축하기로 했다.
그 방법은 쉽다. 아래를 따라서...

## 설치
이미 솔루션급으로 만들어 놓은게 있다. 그걸 그냥 쓰기만 하면 된다.
http://www.sonatype.org/nexus/downloads/ 에서 다운로드 하자.
나는 centOS에 설치했음.
자바는 설치되어 있어야 구동할 수 있음.
OSS라고 써있는 것을 다운받으면 됨.

```
wget http://download.sonatype.com/nexus/3/latest-unix.tar.gz
tar -xvzf latest-unix.tar.gz
```
내려받고 압축만 풀면 설치가 끝난 것이다.

## 설정
나는 다른 부분을 건들지는 않고 포트번호만 바꾸었다.
포트는 설치경로/etc/org.sonatype.nexus.cfg 파일을 vi로 열어서 수정하였다.

## 실행
```
설치경로/bin/nexus start
```

## 종료
```
설치경로/bin/nexus stop
```

## 툴접근
http://서버IP:아까지정한포트번호/ 로 접근하면 툴에 접근된다.
혹시 접근되지 않는다면 루트계정으로 들어간 다음 방화벽에서 해당 포트를 추가해주자.
```
iptables -R INPUT 3 -p tcp -m multiport --dports 22,25,80,443,3306,8080 -j ACCEPT
service iptables save
service iptables restart
service postfix restart
```
3번라인을 교체한다는 의미로 자기가 열고싶은 포트를 쭉 써주자.

## 툴어드민
툴로 들어간 다음 sign in을 누르면 어드민으로 로그인가능하다.
초기 계정, 비밀번호는 아래와 같다.
```
계정 : admin
비밀번호 : admin123
```
어드민에서 적절히 세팅해주면 된다. 나는 admin 암호만 바꾸고 별도로 안함.

## 라이브러리 프로젝트 만들기
저장소가 잘 동작하나 테스트 해보기 위해 아주 간단한 프로그램을 하나 만들었다.
메이븐 프로젝트로 만들고 소스는 아래와 같다.
```
package me.ujung.adder;

public class Adder {

    public int add(int a, int b){
        return a + b;
    }
}
```

pom.xml는 아래와 같이 했다.
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>me.ujung</groupId>
    <artifactId>adder</artifactId>
    <version>1.0.0</version>

    <distributionManagement>
        <repository>
            <id>ujung</id>
            <url>http://서버IP:아까지정한포트번호/repository/maven-releases/</url>
        </repository>
    </distributionManagement>

</project>
```
version에 스냅샷이라는 이름이 들어가면 안된다. 릴리즈에 올리는 것이기 떄문임.

## 라이브러리 프로젝트를 빌드하는 컴퓨터의 m2 레포지토리 설정
라이브러리 프로젝트를 빌드하는 컴퓨터의 m2 레포지토리 설정을 추가주어야 한다.
m2 레포지토리 바로 하위에 아래 settings.xml 파일을 만들고 내용은 아래와 같이 넣어주자.
```
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                      http://maven.apache.org/xsd/settings-1.0.0.xsd">
	<servers>
		<server>
			<id>ujung</id>
			<username>admin</username>
			<password>admin123</password>
		</server>	
	</servers>

</settings>
```
아까 툴어드민 암호를 바꾸었다면 바꾼 암호를 넣어주자.
id는 라이브러리 프로젝트 pom.xml에 설정한 distributionManagement -> repository -> id 의 값과 일치해야 한다.
여기까지 한거면 올릴준비는 끝난거다.

## 메이븐을 통하여 저장소에 배포
라이브러리 프로젝트에서 deploy goal을 실행하면 자동으로 배포된다.
```
mvn deploy
```

## 다른 프로젝트에서 내 라이브러리 사용하기
메이븐 중앙저장소에서 쓰던 것과 별만 다르지 않다.
저장소하나만 설정에 추가해주면 됨.
아래는 다른 프로젝트의 pom.xml 내용중 일부이다.
```
...
<repositories>
		<repository>
			<id>ujung</id>
			<url>http://서버IP:아까지정한포트번호/repository/maven-releases/</url>
		</repository>
	</repositories>
	...
	<dependency>
			<groupId>me.ujung</groupId>
			<artifactId>adder</artifactId>
			<version>1.0.0</version>
		</dependency>
	....
	
```
이제 소스에 가서 라이브러리를 맘껏 사용하면 된다.

## 참고
* http://javacan.tistory.com/entry/%EA%B8%B0%EC%96%B5%EC%9A%A9-Maven-%EC%A4%91%EC%95%99-%EB%A6%AC%ED%8F%AC%EC%A7%80%ED%86%A0%EB%A6%AC-%EB%A7%8C%EB%93%A4%EA%B3%A0-Maven-%EC%84%A4%EC%A0%95%ED%95%98%EA%B8%B0
* http://blog.naver.com/PostView.nhn?blogId=pky0706&logNo=140146609820
