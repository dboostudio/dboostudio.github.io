---
layout: post
author: Dboo
categories: IT_엔지니어를_위한_네트워크_입문
title: CH3. 네트워크 통신하기 (1)
tags: Network MAC_ADDR IP
---

## 1. 캐스트

출발지에서 목적지로 데이터를 전송할 때 사용하는 통신 방식은 다음과 같다.

- 유니캐스트 (1:1)
  출발지와 목적지가 각각 1개로 명확히 정해져 있는 1:1 통식방식이다. 대부분의 통신은 유니캐스트 통신이다.

- 브로드캐스트 (1:All)
  목적지 주소가 `모든`으로 표기되어 있는 통신 방식이다. 동일 네트워크에 존재하는 모든 호스트가 목적지이다.

- 멀티캐스트 (1:Group)
  멀티 캐스트는 그룹 주소를 이용해 해당 그룹에 속한 다수의 호스트에게 전달하기 위한 통신 방식이다.
  IPTV와 같은 실시간 방송을 볼 때 이 멀티캐스트 통신 방식을 사용한다. 사내 방송이나 증권 시세 전송과 같이
  단방향으로 다수에게 같은 내용을 전달할 때 사용한다.

- 애니캐스트 (1:1)
  애니캐스트는 애니캐스트 주소가 같은 호스트들 중에서 가장 가깝거나 가장 빠르게 서비스할 수 있는 호스트와
  통신하는 방식이다. 이런 애니캐스트 게이트웨이 성질을 이용해 가장 가까운 DNS 서버를 찾거나 가장 가까운
  게이트웨이를 찾는 기능에 사용하기도 한다.  
  현재 주로 사용되는 주소 체계는 IPv4기반이고, 일부 모바일 네트워크와 대규모 데이터 센터 위주로 IPv6
  기반 주소 체계가 사용되고 있다.

### 통신 방식 정리

|:-:|:-:|:-:|:-:|:-:|:-:|
|타입|통신 대상|범위|IPv4|IPv6|예제|
|-|-|-|-|-|-|
|유니캐스트|1:1|전체 네트워크|O|O|HTTP|
|브로드캐스트|1:All|서브넷(로컬 네트워크)|O|X|ARP|
|멀티캐스트|1:Group|정의된 구간|O|O|방송|
|애니캐스트|1:1|전체 네트워크|△|O|6 TO 4, DNS|

## 2. MAC 주소

MAC 주소는 Media Access Control의 줄임말로 2계층에서 통신을 위한 고유 식별자이다.

### 2.1 MAC 주소 체계

MAC 주소는 변경할 수 없도록 하드웨어에 고정되어 출하된다. MAC주소는 IEEE가 제조사에 할당하는 제조사
코드와 제조업체가 자체적으로 할당하는 주소로 이루어져있다.

MAC 주소는 48비트의 16진수 12자리로 표현된다.

![](/assets/img/Network-Starter/mac-address-format.jpeg)

앞의 24비트는 IEEE가 부여한 제조사 코드이고 뒤의 24비트값인 UAA는 각 제조사에서 할당한다. MAC 주소는
생산할 때 부여된다는 의미로 BIA(Burned-In Address)라고도 불린다.

### 2.2 MAC 주소 동작

NIC(Network Interface Card)는 전기신호가 들어오면 데이터 형태인 패킷으로 변환하여 도착지 MAC 주소를
확인한다. 만약 자신의 MAC 주소와 동일하지 않으면 페기한다. 패킷의 목적지가 자기 자신이거나 혹은
브로드캐스트, 멀티캐스트와 같은 그룹 주소이면 처리해야 할 주소로 인지해 패킷 정보를 상위 계층으로 넘긴다.
따라서 브로드캐스트나 멀티캐스트의 경우 OS나 어플리케이션에서 처리하게 되고 시스템에 부하로 작용한다.

## 3. IP 주소

OSI 7계층에서 주소를 갖는 계층은 2계층과 3계층이다. 2계층은 물리주소인 MAC 주소를 사용하고, 3계층에서는
논리 주소인 IP주소를 사용한다. IP주소를 포함한 3계층 주소는 다음과 같은 성질을 갖는다.

1. 사용자가 변경 가능한 논리 주소이다.
2. 주소에 레벨이 있고, 그룹을 의미하는 네트워크 주소와 호스트 주소로 나뉜다.

### 3.1. IP 주소 체계

우리가 흔히 알고 있는 IP주소는 32비트인 IPv4 주소이다. IP는 v4, v6 두 체계가 사용된다. IPv6 주소는
128비트이다. IPv4 주소는 32비트이고 총 4개의 옥텟이라고 부르는 8비트 단위로 나눠서 표기한다. MAC 주소가
16진수로 표기된 것과 달리 IP주소는 10진수로 각 옥텟은 0~255의 값을 쓸 수 있다.

![](/assets/img/Network-Starter/IPv4.jpeg)

IP 주소는 네트워크 주소와 호스트 주소로 나뉜다. MAC 주소의 경우처럼 절반씩 해당 주소를 의미하는 것은
아니고, IP 주소는 필요한 호스트 IP 갯수에 따라 네트워크의 크기를 다르게 할당할 수 있는 클래스 개념을
도입했다. 클래스에 따라서, 네트워크 주소를 나타내는 옥텟이 1개일수도 있고 3개일 수도 있게 된다.

![](/assets/img/Network-Starter/ip-class.jpeg)

- A클래스 : 2^8개 네트워크 주소, 2^24개 호스트 주소 (한개의 네트워크 주소 당)
- B클래스 : 2^16개 네트워크 주소, 2^16개 호스트 주소 (한개의 네트워크 주소 당)
- C클래스 : 2^24개 네트워크 주소, 2^8개 호스트 주소 (한개의 네트워크 주소 당)

![](/assets/img/Network-Starter/classes.jpeg)

각 클래스는 첫번째 옥텟의 값으로 구분한다.

- A클래스 : 0~127 (0 ~ 2^7 -1)
- B클래스 : 128~191 (128 ~ 2^7 + 2^6 -1)
- C클래스 : 192~223 (192 ~ 2^7 + 2^6 + 2^5 -1)
- D클래스 : 224~239 (224 ~ 2^7 + 2^6 + 2^5 + 2^4 -1)
- E클래스 : 240~255 (240 ~ 2^8-1)

값을 유심히 보면 알겠지만, 2^7 즉, 2진수 7번째 자리부터 아래 한자리씩 더 1로 늘려갈때마다 클래스가 변경
된다. 또는, 2^8개인 전체 주소의 갯수를 반씩 나누어 A클래스부터 차근차근 할당한다고 설명해도 이해하기 쉽다.

### 3.2 클래스풀과 클래스리스

IP 주소의 클래스 기반의 주소체계를 일컬어 클래스풀이라고 부른다. IP 주소 체계를 처음 확립했을 때 클래스
를 도입한 것은 좋은 선택이었지만 인터넷이 상용화됨에 따라 연결되는 호스트 숫자가 폭발적으로 증가했고, 기존
클래스풀 주소 체계는 IP주소 요구를 감당하기에 부족했다.

이를 위해 3가지 대책을 마련하였는데 다음과 같다.

- 단기 대책 : 클래스리스 기반의 주소 체계
- 중기 대책 : NAT, 사설 IP
- 장기 대책 : IPv6

#### 클래스리스 기반의 IP 주소체계

![](/assets/img/Network-Starter/ip-class.jpeg)

위에서 봤던 이 사진에서 기본 서브넷이 현재 클래스리스 기반의 IP주소 체계에서 A,B,C 클래스에 해당하는
기본 서브넷 마스크이다. 호스트 IP의 요구가 많지 않은 네트워크에서는 해당 기본 서브넷 마스크를 사용하고
있다. 즉, 클래스 주소처럼 사용한다는 의미이다.

서브넷 마스크는 네트워크 주소를 1로, 호스트 주소를 0으로 표시한다. 서브넷 마스크는 1의 연속과 0의
연속되도록 구성한다. 예를 들어, 111101000000 과 같이 0이 나온 이후 1이 들어가면 안된다. 그래서
서브넷 마스크를 표기할 때 /8, /24 와 같이 1이 연속되는 갯수를 표기하기도 한다.

![](/assets/img/Network-Starter/subnet-mask.jpeg)

IP주소와 서브넷 마스크를 & 연산자로 계산하게되면 네트워크 영역이 가질수 있는 2진수의 자리의 영역을 알 수
있다. 즉 서브넷 마스크의 표기가 /24 이라면, IP주소를 2진수로 나타냈을때 세번째 옥텟까지가 네트워크 주소
이다.

### 3.3 서브네팅

클래스리스 기반의 IP주소를 사용하게 되면서, 서브넷 마스크를 활용해 기존 클래스풀 단위의 네트워크보다 더
잘게 쪼개서 네트워크-호스트 구분을 하는것을 서브네팅이라고 한다.

![](/assets/img/Network-Starter/subnetting.jpeg)

#### 1) 네트워크 사용자의 서브네팅

네트워크 사용자는 이미 설계되어있는 네트워크에서 사용할 수 있는 IP의 범위를 파악해야 한다. 주어진 네트워크
범위 밖의 IP를 할당하거나 서브넷 마스크를 잘못 입력하면 통신에 문제가 생기게 될 수 있다.

![](/assets/img/Network-Starter/subnetting-user.jpeg)

IP주소와 서브넷을 통해 네트워크 주소를 연산할 수 있다. 그러면 네트워크 주소 부분을 제외한 나머지가 자연스럽게
사용자가 사용할 수 있는 IP주소가 되는데, 그중 가장 큰 IP는 브로드캐스트 주소로 사용되므로 제일 큰 IP주소를
제외한 나머지가 사용가능한 IP범위가 된다.

#### 2) 네트워크 설계자 입장

설계자의 입장에서는 네트워크의 크기를 고려해 서브넷 마스크를 결정하고 설계에 반영하면 된다.

예제) 현재 가진 네트워크는 103.9.32.0/24 이다. 12개의 IP가 필요한 10개 개소의 네트워크망이 필요한
회사가 있다.  
  1) 각 개소당 12개의 IP를 할당해야 한다. --> 12개의 IP를 할당할 수 있는 가장 작은 네트워크는 16개단위
  이므로 16개를 할당할 수 있도록 서브넷 마스크를 6자리부터 막게 하면 된다.

  > TIP : 16개짜리 네트워크를 할당했을때 사용가능한 IP주소의 갯수는 실제로 네트워크 주소와 브로드캐스트 주소
  를 뺀 14개 이다.

  2) 16개짜리 네트워크 10개를 확보한다. 16의 배수를 0부터 나열해 네트워크 주소를 확인한다.  
    0~16, 17~32, .... 와 같이 확보한다.

### 3.4 공인 IP 와 사설 IP

인터넷에 접속하기 위해서는 IP주소가 있어야 하고, 이 IP는 전세계에서 유일해야 하는 식별자이고, 이런 IP를
공인 IP라고 한다. 하지만 인터넷에 연결하지 않고 개별적으로 네트워크를 구성한다면 공인 IP를 할당받지 않고
네트워크를 구축할 수 있다. 이 IP를 사설IP라고 한다.

가정에서 많이 사용하는 공유기는 NAT 장비중 하나인데, NAT장비는 사설IP를 사용하지만 공인IP로 변환하여
인터넷 접속이 가능하도록 해주는 장비이다. 클래스별로 사용할 수 있는 사설IP 또한 인터넷 표준 문서인 RFC에
명시되어 있어 해당 사설IP를 사용해야 한다.

- A클래스 1개   : 10.0.0.0 ~ 10.255.255.255
- B클래스 16개  : 172.16.0.0 ~ 172.31.255.255
- C클래스 256개 : 192.168.0.0 ~ 192.168.255.255

그러면 만약에 내가 사용하고 있는 사설IP와 같은 공인IP가 있고, 해당 IP로 접속을 시도하면 어떻게 될까?
이 경우, 현재 내 네트워크 내에 있는 IP로 인식하고 브로드캐스트로 통신하려 하기 때문에 실제로 연결은 되지
않는다.  
위와 같은 이유로 사설 IP에 해당하는 주소들은 공인 IP로 할당하지 않고 있다. IP 주소를 할당하는 최상위 기구인
IANA가 여러가지 목적으로 공인 IP를 할당하지 않는 주소를 Bogon IP라고 하는데 사설 IP도 이에 해당한다.
