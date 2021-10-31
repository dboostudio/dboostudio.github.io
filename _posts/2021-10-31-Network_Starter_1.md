---
layout: post
author: Dboo
categories: IT 엔지니어를 위한 네트워크 입문
title: CH1. 네트워크 시작하기
tags: Network TCP/IP Protocol OSI-7-Layer
---

## CH1. 네트워크 시작하기

이번 모 회사의 면접을 진행하면서 제일 많이 받은 질문이 네트워크 계층에서 기본적인 것들의 관한 질문이 많았다.  
하지만, 평소 내가 개발을 하면서 접하고 있음에도 불구하고 질문에 대해 제대로 된 답변을 하지 못하면서
미뤄왔던 네트워크 공부를 해야겠다고 생각이 들었고 기본 입문서로 많이 추천하는 `IT 엔지니어를 위한 네트워크
입문`이라는 책을 구매하여 정리를 시작해보려 한다.

챕터1에서는 모든 네트워크의 이론적 기반이 되는 OSI 7계층, 그리고 패킷이 실제로 전송되는 인캡슐레이션에 대해
알아본다.

### 1. 네트워크 구성도 살펴보기

1. 홈 네트워크

보통 홈 네트워크는 `인터넷 - 모뎀 - 공유기 - 단말`의 순서로 구성되어 있다.

2. 데이터 센터 네트워크

데이터 센터 네트워크의 경우 안정적이고 빠른 대용량 서비스 제공을 목표로 하는 경우가 많다.  
따라서 10G, 25G, 40G, 100G, 400G ... 등의 고속 이더넷 기술이 사용된다.  
기존에는 3계층 구성이 일반적이었으나 가상화 기술과 (도커와 같은 가상 컨테이너를 의미하는 것 같다.) 높은
대역폭을 요구하는 스케일아웃 기반의 서비스 등장으로 2계층 구성인 스파인-리프(Sping-Leaf)구조로 변화하였다.

![https://www.arubanetworks.com/ko/spine-leaf-architecture/](/assets/img/IT-Starter/3-tier-and-2-tier-arch.jpeg)

해당 아키텍처에 관한 자세한 내용은 2장과 13장에 나누어 다룬다.

### 2. 프로토콜

프로토콜이란, 컴퓨터 내부에서 혹은 컴퓨터 간의 데이터 교환방식을 정의하는 규칙체계이다.  
일반적으로 우리가 TCP/IP라고 부르는 통신방식은 TCP프로토콜과 IP프로토콜을 같이 사용하는 것이어서, TCP/IP
프로토콜 스택이라고 부르는것이 올바른 표현이다. 실제로 TCP/IP프로토콜 스택은 총 4개의 부분으로 나뉜다.  
해당 내용은 1.3.2 TCP/IP 프로토콜 스택에서 자세히 다룬다.

|:-:|:-:|
|계층|프로토콜|
|-|-|
|Application|FTP, SSH, TELNET, DNS, SNMP|
|Transport|TCP, UDP|
|Network|ICMP, IP, ARP|
|Physical|Ethernet|

### 3. OSI 7계층과 TCP/IP

1. OSI 7계층

|:-:|:-:|:-:|:-:|
|Layer#|Layer|PDU||
|-|-|-|
|7|Application Layer|Data|Application Layer|
|6|Presentation Layer|Data|^ Upper Layer|
|5|Session Layer|Data|^|
|4|Transport Layer|Segments|Data Flow Layer|
|3|Network Layer|Packets|^ Lower Layer|
|2|Data Link Layer|Frames|^|
|1|Physical Layer|Bits|^|

*PDU : Protocol Data Unit

어렴풋이 듣던 OSI 7계층을 이제서야 정리하게되어 매우 기쁘다.  
복잡한 데이터 과정을 7개의 계층으로 분리하는 것을 OSI 7계층 이라고 한다.  
7계층을 다시 두가지 계층으로 나눌 수 있는데 다음과 같다.

- 1~4계층 : Data Flow Layer / Lower Layer
- 5~7계층 : Application Layer / Upper Layer

2. TCP/IP 프로토콜 스택

현대 네트워크는 대부분 TCP/IP 와 이더넷으로 구성이 되어 있다.  

|:-:|:-:|:-:|:-:|
|Layer#|OSI Model|TCP/IP Model|
|-|-|-|
|7|Application Layer|Application|
|6|Presentation Layer|^|
|5|Session Layer|Data|^|
|4|Transport Layer|Segments|Data Flow Layer|
|3|Network Layer|Packets|^ Lower Layer|
|2|Data Link Layer|Frames|^|
|1|Physical Layer|Bits|^|
