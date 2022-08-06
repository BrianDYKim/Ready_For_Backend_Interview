## [TCP] 3-way handshaking

### 1. TCP에 대해서 설명해주실래요?

<details>
<summary>접기/숨기기</summary>
<div markdown="1">

TCP는 데이터 전송/수신의 신뢰성을 보장하기 위한 전송계층의 연결지향형 프로토콜입니다.

TCP는 데이터의 신뢰성을 보장하기 위해서 대표적으로 아래의 4가지를 수행합니다.

* Segment마다 sequence number를 부여하여 데이터의 순서성을 보장합니다.
* Congestion Control을 이용해서 Network의 혼잡도를 고려하여 Sender의 Congestion window size를 조절함으로써 송신 속도를 제어합니다.
* Flow Control을 이용해서 Receiver측의 receive buffer 사이즈를 고려하여 Sender측의 송신 속도를 제어합니다.
* Segment의 Checksum field를 검증하여 해당 Segment에 비트에러가 없는지 검증합니다.

</div>
</details>

### 2. TCP의 3-way handshaking 과정에 대해서 설명해주실래요?

<details>
<summary>접기/숨기기</summary>
<div markdown="1">

![](./img/3-way-handshaking.png)

1. Client 측에서는 Server 측에 **SYN field**가 1로 채워진 segment를 전송합니다.
2. Server는 client 측으로부터 SYN Segment를 수신받으면 SYN+ACK segment를 client측에 송신합니다. 
그리고 state를 SYN_RECEIVED로 전환하고 Server 내부의 SYN Backlog Queue에 client측이 송신한 syn 정보를 저장해둡니다. 그리고 일정 시간동안 해당 syn이 해소되지 않으면 SYN+ACK을 재전송합니다.
특정 횟수동안 queue 내부의 SYN이 해소되지 않으면 해당 SYN을 expire시킵니다.
3. Client가 SYN+ACK을 수신하면 Server에 ACK을 전송합니다. 이 때 Server는 ACK에 대응하는 SYN을 SYN Backlog Queue를 탐색하여 expire시킵니다.

* 3-way handshaking 과정에서 Client, Server는 각각의 Sequence Number를 동시에 주고받습니다. 데이터의 시작점을 정확하게 알려야하니까요.
* Client는 마지막 ACK Segment에 Packet을 채워서 보낼 수 있습니다. 이를 piggy-bagging이라고 부릅니다.

</div>
</details>

### 2-1. 위에서 설명한 3-way handshaking에는 취약점이 존재하는데 눈치채셨나요? (심화적인 내용입니다. 건너뛰셔도 무방합니다.) 

<details>
<summary>접기/숨기기</summary>
<div markdown="1">

위의 전통적인 TCP 3-way handshaking 과정에서는 2가지의 문제점을 발견할 수 있습니다.

* DOS, DDOS **(Distributed DOS Attack)** 공격에 취약점을 가잡니다. (특히, SYN Flood Attack에 취약점을 가집니다.)
* 대용량 아키텍처를 운영하는 경우, 하나의 Cluster가 fallover 상태에 빠졌다가 recovery 될 때 fallback 과정에서 의도치않은 SYN Flood 현상을 받을 수 있습니다.

특히나 두번째 취약점 사례의 경우 대용량  아키텍처를 운영하는 가운데서 Kafka Cluster의 리커버리 과정에서 겪을수도 있는 문제입니다. **(실제로도 NHN측에서 이 현상으로 인해 고생을 했던적이 있다고 하기도하구요.)**

첫번째 사례의 경우 악의적인 사용자가 자신의 IP를 스푸핑해서 Server측에 SYN Segment만 전송하게 될 시, 서버측에서는 SYN Backlog Queue에 해당 SYN을 계속해서 적재하게됩니다. 
그 과정에서 Server측에서는 **SYN Backlog Queue 오버플로우** 현상을 발생시키게 되고, 이는 서비스 장애로 직결합니다.

해결 방식은 두가지입니다.

1. 운영중인 서버 가상머신에서 SYN Cookie 설정을 1로 설정합니다. 그렇게되면 SYN Backlog Queue가 비활성화되고, 대신에 SYN+ACK에서 SYN Cookie를 실어서 전송하게됩니다. 그리고 이를 수신한 Client는 ACK에 SYN Cookie를 같이 실어서 전송해야합니다.
2. 1번 방식에 더하여, Client와 Server의 가운데에 DDOS 방어장비를 설치합니다. 해당 장비는 신뢰가능한 SYN을 필터하여 SYN Proxy를 Server측에 전송합니다.

그러나 2번의 방식도 완벽하지는 않습니다. 아직도 DDOS 공격의 50%는 SYN Flood Attack이 주류를 이루니까요. 결국 Server는 SYN Proxy를 받더라도 SYN+ACK을 전송해야한다는 사실은 변함이 없고, DDOS 물량이 DDOS 방어장비의 수준을 넘어버리면 결국 서비스가 뻗을수밖에 없습니다.

그러나 1번의 방식만으로는 대용량 아키텍처에서 SYN Backlog Queue 오버플로우 현상에서는 자유로워질수는 있을걸로 기대합니다. SYN+ACK, ACK에 SYN Cookie를 실어야한다는 점에서 오버헤드가 발생할수는 있으나, 연결 단계에서 지연을 겪는게 낫지, Cluster의 장애가 다시 발생하는것보다는 낫다고 판단됩니다.

</div>
</details>