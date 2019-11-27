---
title: Websocket과 STOMP 배경&개념 스크랩
description: Websocket과 STOMP 배경&개념 스크랩
categories:
 - Spring
tags:
---  


## WebSocket

레퍼런스에 따르면, websocket은 웹 브라우저(클라이언트)와 서버간의 full-duplex(양방향), bi-directional(전이중적), persistent connection(지속적인 연결)의 특징을 갖는 프로토콜이라고 규정한다.

여기서 핵심은 클라이언트와 서버간의 지속적인 연결을 한다는 점이다.  Websocket이 탄생하기 이전에는 HTTP을 통해 양방향 통신을 하기 위해  polling, long polling, http streaming과 같은 방식을 사용했다.

​


​

**Http Polling** : 클라이언트가 지속적으로 서버로  request를 하여 이벤트를 수신하는 방식이다. 가장 간단한 방법이지만, 지속적으로 서버에 요청을 던지기 때문에 서버의 오버헤드를 고려할 수 밖에 없는 상황이다.

**Http Long Polling** : polling에 비해 클라이언트는 이벤트를 받기 전까지 다음 요청을 날리지 않는다. 하지만, 원하는 이벤트를 얻기 위해 지속적으로 요청해야한다는 점에서 서버의 부담은 여전히 증가된다.

**Http Streaming** : 서버는 클라이언트로부터 request를 받으면, response을 주고 연결을 끊지 않고. 이벤트가 발생함에 따라 클라이언트로 전송하는 방식인데 역시나 근본적인 원인인 해결하지 못한다.

​

물론 위의 방식으로 원하는 데이터를  클라이언트와 서버간의 주고 받는데 문제는 없다. 하지만 *HTTP 프로토콜 특성의  request-response의 지속적인 수행 그리고 그에 따른 중복적인 패킷전달(http-header) 문제로 인해 속도 저하 및 오버헤드는 근본적으로 문제*가 있게 된다.

​

쉽게 말하면, 내가 원하는 데이터에 비해  동반되는 데이터들이 너무 많고, 지속적으로 이 데이터를 포함해야된다는 점이다.

​

이를 해결하기 위해, 2011년 Websocket 프로토콜이 탄생하게 되었다.

그렇다면, application layer에 존재하는 Websocket과 TCP는 어떤 차이가 있는 걸까?

​

1. HTTP 프로토콜을 통해 request하고 Handshaking이 이루어진다.

2. TCP는 Binary 데이터만 주고 받을 수 있지만, Websocket은 Text 데이터를 주고 받을 수 있다

​

탄생 배경과 정의 그리고 특성으로 미루어 보아, WebSocket은 HTTP와 TCP의 특성을 섞어 놓은 프로토콜이며 웹 브라우저에서 통신하기 위한 기술이다.

​

​

##### Spring에서 2가지 방식으로 WebSocket 구현 가능

1) WebSocket 데이터를 직접 처리

2) Stomp 프로토콜을 사용하여  메세징 처리

​

1) WebSocketHandler를 상속받은 클래스에서 직접 세션레벨에서 데이터를 Handle 해야한다.

​

2) Stomp 프로토콜을 사용하여  메세징 처리

​

우선 STOMP 프로토콜은 (simple text oriented messaging protocol)의 약자이며, 텍스트 기반의 프로토콜이다.

​


​

​

스프링 문서에 Websocket과 함께 STOMP 프로토콜을 사용하는 방법은 위와 같은데, Spring 내부의 In Memory Broker를 통해 메세지를 처리한다.

도식화 된 그림의 키포인트는 3가지이다.

​

1)  Receive Client

2) Send Client

3) Brkoer

​

Receive Client

- 메세지를 받기 위해 특정 토픽이 사전에 서버에 subscribe 되어야 한다.

​

Send Client

- 서버와 연결된 클라이언트는 특정 path로 ex) /app/message  전달한다.

​

Broker

- 메세지 브로커는 Kafka, RabbitMQ, ActiveMQ 등의 오픈소스들 처럼  MQ 이며, pub/sub 모델을 따른다. 토픽에 따라 메세지를 전달해야 하는 사용자를 구분한다.

- 연결된 클라이언트의 세션을 관리한다.

- 특정 토픽과 메세지를 Mapping 하여, 토픽을 구독하는 세션에 존재하는 클라이언트에게 메세지를 전달한다.

​
#### STOMP 구독과 발행 개념
기본적인 콘셉트를 예로 들면

우체통(topic)이 있으면

집배원(publisher)이 신문을 우체통에 배달하는 액션이 있고,

우체통에 신문이 배달되는 것을 기다렸다가 빼서 보는 구독자(subscriber)의 액션이 있습니다. 여기서 구독자는 여러명이 될 수 있습니다.

pub/sub 콘셉트를 채팅룸에 대입하면 다음과 같습니다.

​

채팅방을 생성한다 – pub/sub 구현을 위한 Topic이 하나 생성된다.

채팅방에 입장한다 – Topic을 구독한다.

채팅방에서 메시지를 보내고 받는다 – 해당 Topic으로 메시지를 발송하거나(pub) 메시지를 받는다(sub)

​

pub/sub방식을 이용하면 구독자 관리가 알아서 되므로 웹소켓 세션 관리가 필요 없어집니다. 또한 발송의 구현도 알아서 해결되므로 일일이 클라이언트에게 메시지를 발송하는 구현이 필요 없어집니다.

출처 : 어떤 블로그.. 정확히 기억이안남

​

##### MQ :  메시지 기반의 미들웨어.

메시지를 이용하여 여러 어플리케이션, 시스템, 서비스들을 연결해주는 솔루션이다.

서버 간에 데이터를 주고 받거나 어떤 작업을 요청을 할 때는 항상 시스템 장애를 염두에 두어야 한다. 서버가 갑자기 죽거나 서버 점검 등으로 다운타임이 발생하는 동안에는 요청을 보낼 수가 없다. 요청하는 서버에서 failover 처리를 해놓고 연계 시스템이 다시 살아났을 때 요청을 보내는 방법도 있지만 MQ를 이용하면 더욱 간편하게 처리할 수 있다.

P -> MQ -> C

​

##### sockjs

websocket를 지원하지 않는 낮은 버전의 브라우저에서도 websocket를 사용할수 있도록 해주는 라이브러리

​

출처 https://swiftymind.tistory.com/tag/Websocket%20%2B%20STOMP
