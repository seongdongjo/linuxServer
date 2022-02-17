외부 DB Server와 관련한 정리입니다.

- DB연결
예를들어 외부IP가 222.x.x.x 그안에 vmware의 ip가 192.168.1.100이라고 가정하자
그때 나는 내 노트북으로 외부 IP인 222.x.x.x로 접근을 하고 싶다.
그럴때 먼저 (vmware 브릿지 설정 할필요없다)
1.제어판 - 방화벽설정 - 인바운드규칙에 파일 및 프린터 공유 (ipv4 공용) 우클릭 - 규칙사용
2. 내 노트북에서 ping 222.x.x.x
그 다음 linux에 DB가 깔려있다는 가정하에 (port 1521)
3. 222.x.x.x 대역의 PC에서는 포트포워딩을 해줘야한다.(관리자 권한 - cmd에서)
- netsh interface portproxy add v4tov4 listenport=1521 listenaddress=222.x.x.x connectport=1521 connectaddress=192.168.1.100 (추가)
- netsh interface portproxy delete v4tov4 listenport=1521 listenaddress=222.x.x.x (삭제)
- netsh interface portproxy show v4tov4 (확인)
4. 또, 윈도우 방화벽에서 새인바운드 규칙 - 프로토콜 및 포트 TCP(체크) - 특정로컬포트(1521)
4. tcping.exe를 설치해서 확인한다. C:\\WINDOWS\SYSTEMS32에 붙여넣기
5. tcping 222.x.x.x 1521 로 연결됬는지 확인
(단, 리눅스에서 netstat -nltp 로 1521포트가 열려있는지 확인해야한다)
6. 내 노트북에서 Developer 접속

=======================================================================
---배포서버 구축
[리눅스 서비스 설치 및 운용]

	기본 포트		서비스 이름	용도
===============================================================
apache	80/tcp		httpd		정적 웹 파일 서비스         //html파일 보여주는 용도
tomcat	8080/tcp	apache-tomcat	스프링 프로젝트 배포	  
oracle	1521/tcp	oracle-xe-18c	Database
ssh	22/tcp		sshd		원격접속/파일전송
===============================================================

root#) javac -version , java -version으로 자바가 리눅스에 깔려있는지 확인
root#) cd /usr/local 
root#) wget https://dlcdn.apache.org/tomcat/tomcat-8/v8.5.75/bin/apache-tomcat-8.5.75.tar.gz --no-check-certificate (인증을 체크하지않겠다)
root#) tar xf apache-tomcat-8.5.75.tar.gz
root#) ln -s apache-tomcat-8.5.75 tomcat
root#) vi /etc/profile.d/tomcat.sh
#!/bin/bash
esc -> :$r! readlink -f /bin/java  -> 엔터  -> 이거를 주석처리하기 (그냥 복사할려고쓴거임)
그다음줄에
JRE_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.302.b08-0.el7_9.x86_64/jre
JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.302.b08-0.el7_9.x86_64
CLASSPATH=$JAVA_HOME/bin
PATH=$PATH:$JAVA_HOME/bin:$CATALINA_HOME/bin

저장 후 
export JRE_HOME JAVA_HOME CATALINA_HOME CLASSPATH PATH

source /etc/profile.d/tomcat.sh
echo $CATALINA_HOME
cd $CATALINA_HOME/bin
./startup.sh
-> 그후 윈도우 브라우저에 가상머신 IP치면 톰캣뜬다
//포트가 열려있는지 먼저 확인
netstat -lntup | grep 8080

firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --permanent --add-forward-port=port=80:proto=tcp:toport=8080  //80번으로 들어오면 8080으로 연결해주겠다
firewall-cmd --reload

cd /usr/local/tomcat/webapps
nautilus . &  --> 리눅스 탐색기(노틸러스)가 뜬다

이클립스에서 아무프로젝트(우클릭 -> export -> war -> dest는 바탕화면에 저장 , overwrirting체크) 이거는 현재 war로 압축한다는 뜻이다 -> finish -> 파일이 만들어 진다
윈도우에 있는 파일을 끌어서 놓으면된다. (cd /usr/local/tomcat/webapps )여기 탐색기에다가 -> 해당파일을 더블클릭하면 내용이 나와야한다

드래그드롭이안되면 winscp를 설치
호스트이름 : 리눅스ip

192.168.1.100/exam11A 입력하면 나온다.
exam11A를 ROOT로 바꾸면 알아서 된다. -> 192.168.1.100만입력해도 데이터가 뜬다

vi /etc/systemd/system/tomcat.service
[Unit]
Description=tomcat 8.5
After=network.target syslog.target

[Service]
Type=forking
User=root
Group=root
ExecStart=/usr/local/tomcat/bin/startup.sh
ExecStop=/usr/local/tomcat/bin/shutdown.sh

[Install]
WantedBy=multi-user.target

systemctl daemon-reload
systemctl restart tomcat

cd /usr/local/tomcat/
tail -f $CATALINA_HOME/logs/*      */-> 톰캣의 이벤트가 발생할때마다

