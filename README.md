리눅스와 관련한 정리입니다.

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
