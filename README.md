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




## 참고
* http://javacan.tistory.com/entry/%EA%B8%B0%EC%96%B5%EC%9A%A9-Maven-%EC%A4%91%EC%95%99-%EB%A6%AC%ED%8F%AC%EC%A7%80%ED%86%A0%EB%A6%AC-%EB%A7%8C%EB%93%A4%EA%B3%A0-Maven-%EC%84%A4%EC%A0%95%ED%95%98%EA%B8%B0
* http://blog.naver.com/PostView.nhn?blogId=pky0706&logNo=140146609820
