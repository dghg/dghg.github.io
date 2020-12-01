---
layout: post
title: "Node에서의 비동기 에러 핸들 : try/catch"
date: 2020-11-28 00:00:00
excerpt: ""
tags:
- study
categories:
-
---

일반적으로 자바스크립트에서 발생하는 에러는 `try/catch`로 잡아서 핸들링 할 수 있다.

### 일반적인 에러 핸들링
```
const errorThrow = () => {throw new Error('errorThrow()')};
try {
  errorThrow();
} catch(err) {
  // Error Handling
} finally {
  console.log('FINALLY');
}
```
이렇게 `try/catch` 로 묶으면 원하는 대로 `catch` 에서 오류를 잡아서 처리하고 `finally` 문을 실행할 것이다. 즉 `catch -> finally` 순으로 동작한다.

하지만 `try/catch`는 동기적으로 작동하고, 만약 비동기적인 코드가 `try/catch` 안에 포함되어 있고, 여기서 에러가 발생하는 경우에는 `try/catch`가 잡을 수 없다.

### 비동기가 포함된 try/catch
여기서 `errorThrow` 함수를 비동기 함수로 만든다면 결과는 달라질 것이다.
```
const errorThrow = () => {setTimeout(()=>{throw new Error("errorThrow()")}, 1000)}
```
이렇게 `errorThrow` 를 수정하고, 똑같이 이전의 코드를 실행한다면, 에러를 핸들링 하지 못하고 `finally` 먼저 실행될 것이다.
```
try {
  errorThrow();
} catch(err) {
  // Error Handling
} finally {
  console.log('FINALLY');
}
```

이유는 `try/catch` 문이 비동기 코드를 기다리지 않고 동기적으로 실행되기 때문이다. 따라서 앞의 코드를 실행한다면 `catch` 문이 에러를 잡지 못하여 `finally` 구문이 먼저 실행되고, 이후
`setTimeout` 에 등록된 콜백이 실행되며 에러가 발생함에 따라 노드 프로세스에 `uncaughtException`이 발생하게 된다.

이걸 해결하려면 `errorThrow` 내부에 `try/catch` 문을 사용하여 에러를 잡거나,
`process.on('uncaughtException')` 함수를 이용하여 `uncaughtException`을 핸들링 하거나,
또는 `uncaughtException` 직전에 발생하는 `uncaughtExceptionMonitor` 이벤트를 이용해 노드의 기본 Exception Handler를 건드리지 않고 처리할 수 있다.

하지만 [노드 문서](#https://nodejs.org/dist/latest-v14.x/docs/api/process.html#process_event_uncaughtexception) 에서는 `uncaughtException` 이 발생한 뒤에는 노드를 재실행 하는것이 좋다고 한다.
> The event should not be used as an equivalent to On Error Resume Next.
Unhandled exceptions inherently mean that an application is in an undefined state. Attempting to resume application code without properly recovering from the exception can cause additional unforeseen and unpredictable issues.,
To restart a crashed application in a more reliable way

### 요약
1. 비동기 에러를 잡으려면 비동기 로직마다 `try/catch`를 사용함으로써 잡을 수 있다.
2. `uncaughtException` 나 `uncaughtExceptionMonitor` 핸들러를 통해 에러를 처리할 수 있지만, 이는 많은 문제를 야기할 수 있다.
3. 외부 모듈에서 에러가 발생해 `uncaughtException` 이 발생한다면 ? 노드를 재실행하는게 낫다.
