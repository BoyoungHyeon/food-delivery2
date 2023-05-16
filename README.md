# Cloud Devolper Lv3 Education
![image](https://user-images.githubusercontent.com/49936027/203254202-97ae6b49-9b0b-4204-b2b2-b15b07335774.png)

## 예제 - 음식배달
본 예제는 MSA/DDD/Event Storming/EDA 를 포괄하는 분석/설계/구현/운영 전단계를 커버하도록 구성한 예제입니다. 이는 클라우드 네이티브 애플리케이션의 개발에 요구되는 체크포인트들을 통과하기 위한 예시 답안을 포함합니다.

## 서비스 시나리오
### 기능적 요구사항
- 고객이 메뉴를 선택하여 주문한다.
- 고객이 선택한 메뉴에 대해 결제한다.
- 주문이 되면 주문 내역이 입점상점주인에게 주문정보가 전달된다.
- 상점주는 주문을 수락하거나 거절할 수 있다.
- 상점주는 요리시작때와 완료 시점에 시스템에 상태를 입력한다.
- 고객은 아직 요리가 시작되지 않은 주문은 취소할 수 있다.
- 요리가 완료되면 고객의 지역 인근의 라이더들에 의해 배송건 조회가 가능하다.
- 라이더가 해당 요리를 pick 한후, pick했다고 앱을 통해 통보한다.
- 고객이 해당 요리를 pick 한후, pick했다고 앱을 통해 통보한다.
- 고객이 주문상태를 중간중간 조회한다.
- 주문상태가 바뀔 때 마다 카톡으로 알림을 보낸다.
- 고객이 요리를 배달 받으면 배송확인 버튼을 탭하여, 모든 거래가 완료된다.

## Model
www.msaez.io/#/storming/Q8eftQC5maXPCTCtN9vbXkjAgIl2/d3441e0dfde982ce5847c722ab3a2f82
![image](https://user-images.githubusercontent.com/51141885/203255488-4d1b32c5-b23d-4fad-ab68-f8532c7bfd7d.png)

## 체크포인트
1. Saga (Pub / Sub)
2. CQRS
3. Compensation / Correlation
4. Request / Response
5. Circuit Breaker
6. Gateway / Ingress

## Saga (Pub / Sub)
![image](https://user-images.githubusercontent.com/51141885/203246459-4a0b0b3f-c7f2-4f3c-9c3f-4f684f974a93.png)
![image](https://user-images.githubusercontent.com/51141885/203246709-db78cce0-e1e0-4403-b104-72fc1bd5af4f.png)
![image](https://user-images.githubusercontent.com/51141885/203246860-8654c8a3-7826-453c-9046-8aaf2754e657.png)

## Request  / Response
![image](https://user-images.githubusercontent.com/51141885/203247652-e01cdc55-ca84-4148-b59f-785268d5970e.png)
![image](https://user-images.githubusercontent.com/51141885/203247842-f172bf71-a467-4ccc-be77-7108d1ccba8f.png)

## Circuit Breaker
![image](https://user-images.githubusercontent.com/51141885/203248162-fb1512a4-2c14-49c4-b17f-459a17ecf9eb.png)

## Gateway / Ingress
```
spring:
  profiles: docker
  cloud:
    gateway:
      routes:
        - id: front
          uri: http://front:8080
          predicates:
            - Path=/주문/**, /orders/**, /payments/**, /메뉴판/**, /통합주문상태/**
        - id: store
          uri: http://store:8080
          predicates:
            - Path=/주문관리/**, /orderManages/**, /주문상세보기/**
        - id: customer
          uri: http://customer:8080
          predicates:
            - Path=/logs/**, /orderStatuses/**
        - id: delivery
          uri: http://delivery:8080
          predicates:
            - Path=/deliveries/**, 
        - id: frontend
          uri: http://frontend:8080
          predicates:
            - Path=/**
      globalcors:
        corsConfigurations:
          '[/**]':
            allowedOrigins:
              - "*"
            allowedMethods:
              - "*"
            allowedHeaders:
              - "*"
            allowCredentials: true

server:
  port: 8080

```

## 추가사항 
![image](https://user-images.githubusercontent.com/51141885/203256031-a31e5fbc-2b92-4ab5-b9ce-4a9faa270671.png)
- 배송 완료/픽업 상태 확인 가능
![image](https://user-images.githubusercontent.com/51141885/203256586-2c1fa4ca-2378-41f7-9eed-2c1e0cbd251b.png)
![image](https://user-images.githubusercontent.com/51141885/203256655-c8269504-6a37-4e37-9677-46b87f2d8353.png)
- 주문 수락/거절, 요리 시작/완료 상태 확인 가능
- 사용자는 요리 시작 전 주문 취소 가능

## Before Running Services
### Make sure there is a Kafka server running
```
cd kafka
docker-compose up
```
- Check the Kafka messages:
```
cd kafka
docker-compose exec -it kafka /bin/bash
cd /bin
./kafka-console-consumer --bootstrap-server localhost:9092 --topic 
```

## Run the backend micro-services
See the README.md files inside the each microservices directory:

- front
- store
- customer
- delivery


## Run API Gateway (Spring Gateway)
```
cd gateway
mvn spring-boot:run
```

## Test by API
- front
```
 http :8088/주문 id="id" 품목="품목" 수량="수량" 
 http :8088/orders id="id" foodId="foodId" amount="amount" customerId="customerId" options="options" address="address" status="status" 
 http :8088/payments id="id" orderId="orderId" amount="amount" 
```
- store
```
 http :8088/주문관리 id="id" 
 http :8088/orderManages id="id" foodId="foodId" orderId="orderId" status="status" test="test" couponNumber="couponNumber" 
```
- customer
```
 http :8088/logs id="id" customerId="customerId" message="message" 
```
- delivery
```
 http :8088/deliveries id="id" address="address" orderId="orderId" riderId="riderId" status="status" 
```


## Run the frontend
```
cd frontend
npm i
npm run serve
```

## Test by UI
Open a browser to localhost:8088

## Required Utilities

- httpie (alternative for curl / POSTMAN) and network utils
```
sudo apt-get update
sudo apt-get install net-tools
sudo apt install iputils-ping
pip install httpie
```

- kubernetes utilities (kubectl)
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

- aws cli (aws)
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

- eksctl 
```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
```
