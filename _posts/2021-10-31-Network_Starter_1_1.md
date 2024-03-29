---
layout: post
author: Dboo
categories: IT_엔지니어를_위한_네트워크_입문
title: CH1. 네트워크 시작하기 (1)
tags: Network TCP/IP Protocol OSI-7-Layer
---

이번 모 회사의 면접을 진행할 때, 네트워크 계층에서 기본적인 것들의 관한 질문이 꽤 있었다.
하지만, 평소 내가 개발을 하면서 접하고 있음에도 불구하고 질문에 대해 제대로 된 답변을 하지 못하였다. 이에,
미뤄왔던 네트워크 공부를 해야겠다고 생각이 들었고 기본 입문서로 많이 추천하는 `IT 엔지니어를 위한 네트워크
입문`이라는 책을 구매하여 정리를 시작해보려 한다.

1장에서는 모든 네트워크의 이론적 기반이 되는 OSI 7계층, 그리고 패킷이 실제로 전송되는 인캡슐레이션에 대해
알아본다.

## 1. 네트워크 구성도 살펴보기

### 1-1. 홈 네트워크

보통 홈 네트워크는 `인터넷 - 모뎀 - 공유기 - 단말`의 순서로 구성되어 있다.

### 1-2. 데이터 센터 네트워크

데이터 센터 네트워크의 경우 안정적이고 빠른 대용량 서비스 제공을 목표로 하는 경우가 많다.  
따라서 10G, 25G, 40G, 100G, 400G ... 등의 고속 이더넷 기술이 사용된다.  
기존에는 3계층 구성이 일반적이었으나 가상화 기술과 (도커와 같은 가상 컨테이너를 의미하는 것 같다.) 높은
대역폭을 요구하는 스케일아웃 기반의 서비스 등장으로 2계층 구성인 스파인-리프(Sping-Leaf)구조로 변화하였다.

![https://www.arubanetworks.com/ko/spine-leaf-architecture/](/assets/img/Network-Starter/3-tier-and-2-tier-arch.jpeg)

해당 아키텍처에 관한 자세한 내용은 2장과 13장에 나누어 다룬다.

## 2. 프로토콜

프로토콜이란, 컴퓨터 내부에서 혹은 컴퓨터 간의 데이터 교환방식을 정의하는 규칙체계이다.  
일반적으로 우리가 TCP/IP라고 부르는 통신방식은 TCP프로토콜과 IP프로토콜을 같이 사용하는 것이어서, TCP/IP
프로토콜 스택이라고 부르는것이 올바른 표현이다. 실제로 TCP/IP프로토콜 스택은 총 4개의 부분으로 나뉜다.  
해당 내용은 3-2 TCP/IP 프로토콜 스택에서 자세히 다룬다.

|:-:|:-:|
|계층|프로토콜|
|-|-|
|Application|FTP, SSH, TELNET, DNS, SNMP|
|Transport|TCP, UDP|
|Network|ICMP, IP, ARP|
|Physical|Ethernet|

## 3. OSI 7계층과 TCP/IP

### 3-1. OSI 7계층

|:-:|:-:|:-:|:-:|
|Layer#|Layer|PDU||
|-|-|-|
|7|Application Layer|Data|Application Layer|
|6|Presentation Layer|Data|Upper Layer|
|5|Session Layer|Data|^^|
|4|Transport Layer|Segments|Data Flow Layer|
|3|Network Layer|Packets|Lower Layer|
|2|Data Link Layer|Frames|^^|
|1|Physical Layer|Bits|^^|

*PDU : Protocol Data Unit

어렴풋이 듣던 OSI 7계층을 이제서야 정리하게되어 매우 기쁘다.  
복잡한 데이터 과정을 7개의 계층으로 분리하는 것을 OSI 7계층 이라고 한다.  
7계층을 다시 두가지 계층으로 나눌 수 있는데 다음과 같다.

- 1~4계층 : Data Flow Layer / Lower Layer
- 5~7계층 : Application Layer / Upper Layer

### 3-2. TCP/IP 프로토콜 스택

현대 네트워크는 대부분 TCP/IP 와 이더넷으로 구성이 되어 있다.  

|:-:|:-:|:-:|
|Layer#|OSI Model|TCP/IP Model|
|-|-|-|
|7|Application|Application|
|6|Presentation|^|
|5|Session|Data|
|4|Transport|Transport|
|3|Network|Internet|
|2|Data Link|Network Access|
|1|Physical|^|

TCP/IP의 경우 OSI 7계층과는 달리, 4계층으로 구분한다. 이는 TCP/IP 프로토콜의 성향이 이론보다 실용성에
중심을 두고 현실에 잘 반영하려는 성향에 기반한다.

## 4. OSI 7계층별 이해하기

### 4-1. 1계층 (Physical)

물리 계층으로, 전기신호를 보내는데 초점이 맞추어져 있다.  
1계층 주요장비로는 Hub, Repeater, Cable, Connector, Tranceiver, TAP 등이 있다.

Hub, Repeater : 네트워크 통신을 중재하는 장비  
Tranceiver : 랜 카드와 케이블을 연결하는 장치  
TAP : 네트워크 모니터링과 패킷분석을 위해 전기신호를 다른 장비로 복제

- 1계층 장비는 전기신호의 전달이 목적이므로, 받은 전기신호를 재생성하여 내보낸다.
- 1계층 장비는 주소의 개념이 없으므로 전기신호가 들어온 포트를 제외한 나머지 포트에 동일한 전기 신호를 전송한다.

### 4-2. 2계층 (Data Link)

데이터 링크 계층으로, 전기신호를 모아 알아볼 수 있는 데이터 형태로 처리한다.  

- 주소정보를 정의하고 정확한 주소로 통신이 되도록 하는것이 주 목적이다.  
출발지와 도착지 주소를 확인하고 내게 보낸것이 맞는지, 내가 처리해야 하는지에 관해 검사 후 데이터 처리를 수행한다.
- 주소체계가 생기면서 여러 통신이 한꺼번에 이루어지는것을 구분하기 위한 기능이 주로 정의된다.  
데이터에 대한 에러를 탐지하거나 고치는 역할도 수행할 수 있다.
- 2계층은 MAC주소라는 주소체계를 가지고 있다.

2계층에서 동작하는 네트워크 구성요소는 네트워크 인터페이스 카드와 스위치이다.  
이 두개의 구성요소 모두 MAC주소를 이해할 수 있다.

1. 네트워크 인터페이스 카드
- 네트워크 인터페이스 카드(NIC)는 고유한 MAC주소를 지닌다.
- 자신에게 온 전기신호를 데이터형태로 만든다.
- 목적지 MAC주소가 자기자신의 주소와 동일하면 메모리에 적재하여 상위계층에 공급하고, 주소가 다를시에는 데이
터를 폐기한다.

2. 스위치
- 스위치는 단말의 MAC주소와 연결된 포트를 습득하여 알 수 있다.
- 스위치는 전기신호의 필터링과 포워딩을 하는 역할을 한다.
- 허브와의 차이점 : 허브(1계층)는 모든 포트에 전기신호를 전달하지만, 스위치는 필요로 하는 포트에만 전달한다.

### 4-3. 3계층 (Network)

3계층에서는, IP주소와 같이 논리적인 주소가 정의된다. 데이터 통신을 할 때에는 두가지 주소가 사용되는데,
한가지는 앞서 보았던 2계층의 MAC주소이고 또 하나는 3계층의 논리적인 IP주소이다.

IP주소는 고유하게 부여되는 MAC주소와는 달리 사용자가 환경에 맞게 변경하여 사용할 수 있다는 점이다.

- IP주소는 네트워크 주소와 호스트 주소로 세부적으로 구분된다.  
ex) 172.31.0.1 -> 네트워크 주소 : 172.31, 호스트 주소 : 0.1

3계층을 이해할 수 있는 장비나 단말은 네트워크 주소 정보를 이용해 자신이 속한 네트워크와 원격지 네트워크를
구분할 수 있고, 원격지 네트워크로 가려면 어디로 가야하는지 지정하는 능력이 있다.

- 라우터
3계층에서 동작하는 장비는 라우터이다. IP주소를 사용해 최적의 경로를 찾고 해당경로에 패킷을 전송한다.

### 4-4. 4계층 (Transport)

4계층은 앞서 보았던 1~3계층과는 역할의 종류가 조금 달라진다.

- 4계층은 실제로 데이터들이 정상적으로 잘 보내지는지 확인하는 역할을 한다.  
패킷 네트워크는 데이터를 분할하여 패킷에 실어보낸다. 그 중에 패킷이 유실되거나 순서가 바뀌는 경우가 있는데
이 문제를 해결하는 역할을 4계층에서 맡는다.
- 패킷에는 보내는 순서를 명시한 시퀀스 번호와 받는 순서를 명시한 ACK번호가 있다.

4계층에서 동작하는 장비는 로드 밸런서와 방화벽이 있고, 포트번호와 시퀀스,ACK 번호를 통해 부하를 분산하거나
보안정책을 수립해 패킷을 통과,차단하는 기능을 수행한다.

### 4-5. 5계층 (Session)

세션 계층은 양 끝단의 응용 프로세스가 연결을 성립하도록 도와주고 연결의 유지, 관리, 중단하는 역할을 한다.
흔히 우리가 부르는 '세션'을 관리하는 것이 주요역할이며 TCP/IP 세션을 만들고 없애는 책임을 진다.

### 4-6. 6계층 (Presentation)

프레젠테이션 계층은 표현방식이 다른 어플리케이션이나 시스템간의 통신을 돕기위해 하나의 통일된 구문형식으로
변환시키는 기능을 수행한다. 인코딩, 암호화, 압축, 코드변환과 같은 동작이 이 계층에서 이뤄진다.

### 4-7. 7계층 (Application)

OSI 7계층의 최상위 계층이다. 어플리케이션 계층은 프로세스를 정의하고 서비스를 수행한다. 네트워크 소프트웨어의
UI나 사용자 입,출력 부분을 정의하는 것이 어플리케이션 계층의 역할이다.

- 대표적인 프로토콜 : FTP, SMTP, HTTP, TELNET 등

### 계층별 주요 프로토콜 및 장비

|:-:|:-:|:-:|
|계층|주요 프로토콜|장비|
|:-:|:-|:-|
|Application|HTTP, SMP, SMTP, STUN, TFTP, TELNET|ADC, NGFW, WAF|
|Presentation|TLS, AFP, SSH||
|Session|L2TP, PPTP, NFS, RPC, RTCP, SIP, SSH||
|Transport|TCP, UDP, SCTP, DCCP, AH, AEP|로드 밸런서, 방화벽|
|Network|ARP, IPv4, IPv6, NAT, IPSec, VRRP, 라우팅 프로토콜|라우터, L3 스위치|
|Data Link|IEEE 802.2, FDDI|스위치, 브릿지, 네트워크 카드|
|Physical|RS-232, RS-449, V.35, S 등의 케이블|케이블, 허브, 탭 등|
