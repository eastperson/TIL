# Overview

클러스터링은 하나의 엔드포인트를 동일한 서비스 노드(인스턴스) 그룹이 공유하여 가용성과 확장성을 높게 가져가기 위해서 사용한다. 우리는 그 부하의 균형을 맞춰야 한다. ‘로드밸런서’는 하나의 애플리케이션으로 요청이 오면 어떤 노드에게 전달해야할지 결정해주는 역할을 하게 된다. 

로드 밸런서는 서버로 들어오는 트래픽에 대해 부하를 분산시켜주는 역할을 하는 소프트웨어 혹은 하드웨어이다. 들어오는 트래픽의 라우팅을 결정짓기 위해는 수많은 알고리즘이 있다. 트래픽에 부하가 오지 않게 하고 서버가 꺼졌을 때 요청을 다른 서버에게 보내주는 역할을 하는 것이 목표이다.

로드밸런서는 아래와 같은 역할을 한다.

1. 서버가 부하가 오지 않고 요청의 라우팅을 효율적으로 하기 위해서다.
2. 온라인 서버의 요청을 라우팅하여 서비스의 중단 되는 시간을 줄이고 높은 가용성을 제공한다.
3. 서버가 부담하는 부하에 따라 서버를 추가하거나 제거하여 서버의 확장성을 처리한다.

![image](https://user-images.githubusercontent.com/66561524/194558434-33fe0936-0628-48a0-a0e8-c56adf9e734c.png)

# Load balancing의 종류

로드 밸런싱의 종류는 OSI 7계층에 따라 나뉜다.

- L4 : Transport 계층을 사용, IP 주소와 포트 번호 부하 분산이 가능
- L7 : Application 계층을 사용, URL 또는 HTTP 헤더에서 부하 분산이 가능

![image](https://user-images.githubusercontent.com/66561524/194558474-3ee9fd94-cdc3-41c2-abda-3a6fe727a0dc.png)

## L4 Load Balancing

![image](https://user-images.githubusercontent.com/66561524/194558503-0a3e573e-9561-46b5-93ab-799b536b1e14.png)

- **전송 계층**에서 로드를 분산한다.
- **IP주소나 포트번호, MAC주소** 등에 따라 트래픽을 나누고 분산처리가 가능하다.
- CLB(Connection Load Balancer) 혹은 SLB(Session Load Balancer)라고 부르기도 한다.

## L7 Load Balancing

![image](https://user-images.githubusercontent.com/66561524/194558560-e80d15e7-813a-4ce6-b3e2-bcf6b4969a97.png)

- **애플리케이션 계층**에서 로드를 분산한다.
- OSI 7계층의 프로토콜(HTTP, SMTP, FTP 등)을 바탕으로도 분산 처리가 가능하다.

# AWS의 로드밸런서

로드밸런서는 몇 계층에서 분산작업을 수행하느냐에 따라 **NLB(Network Load Balancer)**와 **ALB(Application Load Balancer)**로 나눌 수 있다. 기존에는 **CLB(Classic Load Balancer)**라고 하는 여러 EC2 인스턴스간에 간단한 트래픽 부하 분산하는 로드 밸런서도 있었지만 최근에는 잘 사용하지 않는다.

## NLB

![image](https://user-images.githubusercontent.com/66561524/194558600-64220462-11c3-494a-bdc0-f9a9e6be9795.png)

- L4단의 로드밸런서를 지원한다.
- TCP/IP 프로토콜의 헤더를 보고 적절한 패킷으로 전송한다.
- IP + 포트번호를 보고 스위칭한다.
- Client IP와 서버사이에 서버로 들어오는 트래픽은 Load Balancer를 통하고 나가는 트래픽은 Client IP와 직접 통신합니다.
- NLB는 Security Group 적용이 되지 않아 서버에 적용된 Security Group에서 보안이 가능하다.
- Client → Server에서 Access 제한 가능
- NLB는 할당한 Elastic IP를 Static IP로 사용이 가능하여 DNS Name과 IP 주소 모두 사용이 가능하다.
- Name Server 또는 Route 53에서 Record 사용이 가능하다.
- SSL 적용이 인프라 단에서 불가능해 애플리케이션에서 따로 적용해야 한다.

## ALB

![image](https://user-images.githubusercontent.com/66561524/194558685-2508c160-d65f-4e45-9c5b-c93866886140.png)

- L7단의 로드 밸런서를 지원한다
- HTTP/HTTPS 프로토콜의 헤더를 보고 적절한 패킷으로 전송한다.
- IP 주소 + 포트번호 + 패킷 내용을 보고 스위칭한다.
- IP 주소가 변동되기 때문에 Client에서 Access 할 ELB의 DNS Name을 이용해야 한다.
- L7단을 지원하기 때문에 SSL 적용이 가능하다.
- Reverse Proxy 대로 Client IP와 서버사이에 들어오고 나가는 트래픽이 모두 Load Balancer 와 통신한다.
- CLB/ALB는 Security Group 을 통한 보안이 가능하다.
- Client → Load Balancer의 Access 제한이 가능하다.
- ALB/CLB는 IP 주소가 변동되기 때문에 Client 에서 Access 할 ELB의 DNS Name을 이용해야 한다.
- Name Server 또는 Route 53에서 CNAME 을 사용해야 Domain Name 연동이 가능하다.

# Load Balancer Algorithm

로드밸런서는 아래의 알고리즘을 사용한다.

## 라운드 로빈 방식(Round Robin)

![image](https://user-images.githubusercontent.com/66561524/194559259-318eab48-66f9-436b-ac50-abb57aae55dc.png)


- 클라이언트로부터 받은 요청을 대상 서버에 순서대로 할당하는 방식.
- 클라이언트의 요청을 순서대로 분배하기 때문에 여러 대의 서버가 동일한 스펙을 갖고 있고, 서버와의 연결(세션)이 오래 지속되지 않는 경우에 활용하기 적합함.

## 가중 라운드 로빈 방식(Weighted Round Robin)

![image](https://user-images.githubusercontent.com/66561524/194559280-f99837a4-32d4-4873-b82a-368c31847385.png)

1. 각각의 서버마다 가중치를 매기고 가중치가 높은 서버에 클라이언트 요청을 우선적으로 배분하는 방식
2. 주로 서버의 트래픽 처리 능력이 상이한 경우 사용되는 부하 분산 방식이다. 예를 들어 `A라는 서버가 5`라는 가중치를 갖고 `B라는 서버가 2`라는 가중치를 갖는다면, 로드밸런서는 라운드로빈 방식으로 `A 서버에 5개` `B 서버에 2개`의 요청을 전달함.

## 최소 연결 방식(Least Connections)

![image](https://user-images.githubusercontent.com/66561524/194559312-08909a44-33ee-4195-8421-106a4037a779.png)

- 요청이 들어온 시점에 가장 적은 연결상태를 보이는 서버에 우선적으로 트래픽을 배분하는 방식
- 자주 세션이 길어지거나, 서버에 분배된 트래픽들이 일정하지 않은 경우에 적합함.

## 최소 응답 타임 방식(Least Response Time)

- 서버의 현재 연결 상태와 응답시간(Response Time, 서버에 요청을 보내고 최초 응답을 받을 때까지 소요되는 시간)을 모두 고려하여 트래픽을 배분하는 방식
- 가장 적은 연결 상태와 가장 짧은 응답시간을 보이는 서버에 우선적으로 로드를 배분함.

## 해시 방식(Hash)

![image](https://user-images.githubusercontent.com/66561524/194559363-0d04743a-b8a5-4229-8c9a-e4f1a0c001fb.png)


- 클라이언트의 요청 URL 같이 정의한 키를 기반으로 요청을 처리하는 방식

## IP 해시 방식(IP Hash)

- 클라이언트의 IP 주소를 특정 서버로 매핑하여 요청을 처리하는 방식
- 사용자의 IP를 해싱(Hashing, 임의의 길이를 지닌 데이터를 고정된 길이의 데이터로 매핑하는 것, 또는 그러한 함수)하여 로드를 분배하기 때문에 사용자가 항상 동일한 서버로 연결되는 것을 보장함.

## Random with Two Choices

- 무작위로 두 개의 서버를 선택하고 최소 연결 알고리즘을 적용하여 선택한 서버에 요청을 보낸다.

# 로드 밸런싱의 장점

## 확장성

- 클러스터에서 서버 또는 서비스 노드를 추가하거나 제거할 시기를 결정할 수 있다.

## 가용성

- 서버의 가동 중지 시간을 줄이고 요청을 온라인 서버로 다시 라우팅하여 이를 달성한다.

## 고효율

- 일련의 작업을 병렬로 실행할 수 있다.


