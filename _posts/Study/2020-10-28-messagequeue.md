---
layout: post
title: "비동기 시스템과 Message Queue"
date: 2020-10-28 00:00:00
excerpt: ""
tags:
- study
categories:
-
---

# 비동기 시스템과 Message Queue

이 글은 [Aynchronous Systems & Message Queue](#https://medium.com/datadriveninvestor/what-is-message-queue-b5468ff6db50)를 번역한 것입니다.

메세지 큐를 알아보기 전에, 먼저 비동기 시스템이 무엇인지 알아봅시다.

다음과 같은 상황을 고려해 봅시다. : 고객이 음식을 주문하기 위해 식당에 들어왔습니다.

![rest](https://miro.medium.com/max/1188/0*vyoUPhDZ9bvhm-RI)

- Reception
 > 고객의 주문을 받고, 확인합니다. [Event Recived]
 > 계산을 합니다. [Action Perform]
 > **Kitchen**에 음식 요청을 합니다. [Event Triggered]

- Kitchen
 > 주문을 받습니다. [Event Recived]
 > 요리를 하고 [Action Perform]
 > **Reception**에게 보냅니다.[Event Triggered]

 - Reception
  > 처리된 주문을 **Kitchen**으로부터 받습니다.[Event Received]
  > 주문이 끝났다고 Mark 합니다. [Action Perfomed]

음식이 **Kitchen**에서 준비될 동안, **Reception** 은 주문을 처리하기 위해 **Kitchen** 을 기다리지 않고, 다른 고객들의 주문을 받습니다. 만약 **Kitchen**이 바쁘다면, 바로 주문을 처리하고, 처리되었다면 다음 주문을 선택하여 처리할 수 있습니다.

여기서 알아야 할 점들은, **Reception** **Kitchen** 둘다
 - 각각 독립적인 할 것들(Responsibilities)이 있다.
 - 서로가 끝나기를 기다리지 않는다.
 - 시간에 영향을 받지 않는다.

 이것이 **비동기 시스템** 이라고 말하는 것입니다. 그것은 Events의 집합을 가진 컴포넌트로 구성되고, 전체적인 흐름은 컴포넌트가 이벤트 발생에 근거하여 특정 행동을 수행하고, 통신을 위해 다른 이벤트를 발생시키는 EDA(Event-Driven-Architecture)의 지배를 받습니다.


말한대로, **Kitchen**은 주문을 준비하느라 바쁠 것이고, 새로운 주문을 동시에 처리할 수 없습니다. 따라서 새로운 주문 요청은 반드시 어느 시간동안 저장되어 있어야 합니다. 이것이 Message Queue의 방식입니다.

![reception](https://miro.medium.com/max/1400/0*8lGL3X2OHtRReUnO)

Message Queue는 어플리케이션 간 비동기 통신을 위해 사용됩니다.

![mq](https://miro.medium.com/max/1152/1*jyvAtVsULzry5eYG1QL3dw.png)

### "Asynchronous application-to-application commnunication"

비동기 통신이란 어플리케이션 'A'가 다른 어플리케이션 'B'로 메시지 'M'을 보낼 때, 처리를 하기위한 즉각적인 응답은 요구하지 않는 것을 의미합니다.
또한 B는 바쁘거나 또는 연결이 종료되었을 수도 있습니다. 'B'가 사용가능해지면, 'A'에게 응답을 보낼 수 있습니다. 동시에 'A'는 다른 메시지를 보낼 수도 있습니다. 그래서 우리는 'M'을 어디다 저장해둘까요? 명백히 우리는 M이 중간에 사라지길 원하지 않습니다. 여기서 Message Queue는 일시적인 저장소를 제공해 줍니다.

이메일이 비동기 메시지의 대표적인 에시입니다. 이메일을 보낼 때, 보내는 사람은 받는사람의 응답에 상관 없이 다른 일을 계속 할 수 있습니다.

### Message Queue: Message + Queue
큐는 어플리케이션 간에 보내진 메세지들의 순서를 포함하고 있습니다. 메시지들은 Consumer가 그것들을 회수하기 전에는 큐에 존재합니다.

여기서 메시지를 만들어 보내는 어플리케이션을 **producer**라고 하고, 메시지를 받아 쓰는 어플리케이션은 **consumer**라고 합니다.

**Message** 는 **producer** 에서 **Consumer** 로 보내지는 데이터 입니다. 그것은 요청일 수도 있고, 응답일 수도 있습니다. MQ는 메시지를 처리하지 않고, 단지 저장만 하빈다. 이렇게 메시지를 다루는 방식은 **Consumer**로부터 **Producer**를 분리합니다.

### 모델의 종류
1. Producer - Consumer : Producer는 Consumer를 알고 있고, 특정한 Consumer만 메시지를 꺼낼 수 있는 1-1 메시지 모델입니다.
2. Publish - Subscriber : 광범위한 메시지 모델입니다. producer(publisher)는 consumer(receiver)에 대해 알지 못하고, 클래스 또는 태그를 명시함으로써 큐에 메시지를 보내기만 합니다. 그 클래스나 태그를 구독하는 Reciver는 메시지를 받을 수 있습니다. 사실은, 모든 Receiver가 동일한 메시지를 받습니다. 또한 모든 메시지들은 만료 시간이 있고 그것은 자동적으로 보유 기간이 지난 뒤 삭제됩니다.

### Message Queue의 장점
메시지 큐는 **decoupling**을 구현하는 것을 도와주기 때문에 중요합니다. 두개 또는 이상의 어플리케이션들이 서로 연결되어 있지 않고 통신을 할 수 있습니다.
또한, 하나의 어플리케이션은 다른 어플리케이션의 구현을 알지 못합니다. 즉, 둘 사이의 의존성이 없습니다.

이렇게 분리된 어플리케이션을 통해,

1. 한 어플리케이션의 변경이 다른 어플리케이션에 영향을 주지 못한다.
2. 한개의 통제하는 어플리케이션을 전반적인 복잡성을 낮추는 여러개의 작은 어플리케이션으로 나눌 수 있다.
3. 유지보수와 디버깅이 쉽다.
4. 크로스플랫폼 어플리케이션을 만들 수 있다. 작은 어플리케이션들은 언어와 스케일에 상관없이 독립적으로 개발될 수 있다.

메시지 큐를 통해, 시스템의 안정성과 성능 향상을 가져올 수 있습니다. Producer는 Consumer가 사용가능할때까지 기다릴 필요없이, 큐에 요청을 추가할 수 있습니다.
Consumer는 가능할 때 메시지를 처리합니다. 따라서 대기 시간의 오버로드가 발생하지 않습니다. Message Queue는 메시지를 항상 보관하고 있기 때문에, 심지어 양쪽 어플리케이션 모두 사용이 불가능하더라도, 데이터 유실 없이 시스템은 고장을 방지할 수 있습니다.
