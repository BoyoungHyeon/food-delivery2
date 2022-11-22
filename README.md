# 
## 예제 - 음식배달
본 예제는 MSA/DDD/Event Storming/EDA 를 포괄하는 분석/설계/구현/운영 전단계를 커버하도록 구성한 예제입니다. 이는 클라우드 네이티브 애플리케이션의 개발에 요구되는 체크포인트들을 통과하기 위한 예시 답안을 포함합니다.

## 서비스 시나리오
###기능적 요구사항
- 고객이 메뉴를 선택하여 주문한다.
- 고객이 선택한 메뉴에 대해 결제한다.
주문이 되면 주문 내역이 입점상점주인에게 주문정보가 전달된다.
상점주는 주문을 수락하거나 거절할 수 있다.
상점주는 요리시작때와 완료 시점에 시스템에 상태를 입력한다.
고객은 아직 요리가 시작되지 않은 주문은 취소할 수 있다.
요리가 완료되면 고객의 지역 인근의 라이더들에 의해 배송건 조회가 가능하다.
라이더가 해당 요리를 pick 한후, pick했다고 앱을 통해 통보한다.
고객이 주문상태를 중간중간 조회한다.
주문상태가 바뀔 때 마다 카톡으로 알림을 보낸다.
고객이 요리를 배달 받으면 배송확인 버튼을 탭하여, 모든 거래가 완료된다.

## Model
www.msaez.io/#/storming/Q8eftQC5maXPCTCtN9vbXkjAgIl2/d3441e0dfde982ce5847c722ab3a2f82

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

