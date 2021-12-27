---
layout: post
author: Dboo
categories: 분류미정
title: MacOS에 virtualBox + vagrant로 OracleDB 설정하기
tags: short_tip vagrant virtualBox OracleDB
---

일단 virtualBox와 vagrant를 설치해야한다.
사실 도커로 할 수도 있지만, docker 볼륨설정에 대한 공부가 덜 되어있어서 이 방법으로 진행하기로 했다.

~~~bash
brew install --cask vagrant
brew install --cask virtualBox
~~~

그 후에, Oracle에서 제공하는 ![vagrant-project](https://github.com/oracle/vagrant-projects/tree/main/OracleDatabase)깃허브 저장소에서 프로젝트를 내려받는다.

프로젝트를 열어보면 많은 것들이 있는데 OracleDatabse폴더로 진입한다.

~~~bash
.
├── CODEOWNERS
├── CONTRIBUTING.md
├── ContainerRegistry
├── ContainerTools
├── DockerEngine
├── LAMP
├── LICENSE.txt
├── OLCNE
├── OracleAPEX
├── OracleDG
├── OracleDatabase
│   ├── 11.2.0.2
│   ├── 12.1.0.2
│   ├── 12.2.0.1
│   ├── 18.3.0
│   ├── 18.4.0-XE
│   ├── 19.3.0
│   │   ├── LINUX.X64_193000_db_home.zip
│   │   ├── README.md
│   │   ├── Vagrantfile
│   │   ├── ora-response
│   │   ├── scripts
│   │   └── userscripts
│   └── README.md
├── OracleFPP
├── OracleGoldenGate
├── OracleLinux
├── OracleRAC
├── README.md
├── SECURITY.md
└── THIRD_PARTY_LICENSES.txt
~~~

나는 19.3.0 버전이 설치된 가상머신을 띄우기로 결정했다.  
19.3.0 폴더안으로 진입해서 README.md를 먼저 읽어보았다. 그 내용은 다음과 같다.

1. vagrant-project 깃허브 저장소를 클론하라.
2. 19.3.0 버전 폴더안으로 진입한다. //여기까지는 이미 완료한 사항이다.
3. Oracle홈페이지에서 'LINUX.X64_193000_db_home.zip'을 다운받아 현재 폴더(19.3.0)에 옮긴다.  
  ![다운로드링크](http://www.oracle.com/technetwork/database/enterprise-edition/downloads/index.html)
4. `vagrant up` 명령어를 실행한다.
- 가상머신에 설치된 Oracle접속정보는 다음과 같다.
* Hostname: `localhost`
* Port: `1521`
* SID: `ORCLCDB`
* PDB: `ORCLPDB1`
* EM Express port: `5500`
* 데이터베이스 패스워드는 자동생성되며, 설치될때 보여진다.

위 단계를 수행하면 vagrant가 linux에 oracleDB가 올라간 가상머신을 virtualBox를 통해 설치하기
시작한다.

![](/assets/img/ETC/short-tip/vagrant-oracle-install.png)

설치가 완료된 후 로그이다. SYS, SYSTEM, PDBADMIN에 대한 패스워드가 `aqz0HxCPWlE=1`라고 나와있다.

README.md에는 패스워드 리셋하는 방법이라던지 뭔가 어려운것들이 잔뜩 더 기재되어있지만 나는 안쓸거 같으니
DB접속하러 가보자.

터미널에서 `vagrant ssh`를 입력한다. 그러면 가상머신으로 접속이 되어진다.

![](/assets/img/ETC/short-tip/vagrant-oracle-install.png)

이제 REAME.md마지막줄에 나와있는대로 `sudo su - oracle`로 `OracleDB`서버에 접속한 다음 `sqlplus`
명령어로 RDBMS를 실행한다.

![](/assets/img/ETC/short-tip/vagrant-oracle-sqlplus.png)

일단 설치가 다 되었으니, 가상머신의 DB에 내가사용하는 DB툴로 접속을 해보자.

먼저 virtualBox를 실행시키면 이미 vagrant가 만들어놓은 linux+oracle 이미지가 보일 것이다.  
우클릭->설정->네트워크탭 으로 진입하고, 하단 고급탭을 열면 포드포워드라는 버튼을 누른다.

![](/assets/img/ETC/short-tip/vagrant-virtualbox-network.png)

포트포워드 안에 보면 이미 오라클이 호스트포트 1521로 열려있음을 확인할 수 있다.

![](/assets/img/ETC/short-tip/vagrant-virtualbox-portforwarding.png)

나는 datagrip이라는 jetbrain DB툴을 주로 사용한다. 연결정보는 동일하니 다른 DB툴에서도 동일하게 연
결설정을 해주면 된다.

Datagrip에서는 좌상단 + 버튼을 눌러 DB연결을 추가한다.

![](/assets/img/ETC/short-tip/vagrant-oracle-datagrip.png)

DB연결 추가버튼을 누르면 연결정보를 입력하는 화면이 나오는데, 위에 REAME.md에 나와있는 hostname, port, sid와 username에 SYSTEM, password는 콘솔에 떴었던 oracle패스워드를 입력한다. 그후 test
connection을 진행한다.

![](/assets/img/ETC/short-tip/vagrant-oracle-datagrip-connect.png)

테스트 연결이 된것을 확인했다. 이제 확인버튼을 눌러 DB연결을 확정하고, 이제 공부만 열심히 하면된다!
