---
title: REST-API
date: 2021-03-26 17:03:18
category: web
thumbnail: { thumbnailSrc }
draft: false
---

> **RE**presentational **S**tate **T**ransfer API
서버가 클라이언트에게 리소스를 어떻게 제공할지에 대한 API 설계방식이다. 
**HTTP URI** + **HTTP METHOD**를 이용해 설계하고, 자원 **포맷(JSON)** 을 결정해 클라이언트에게 제공한다. 프론트엔드 개발을 하면서 API 설계시 궁금했던 부분들만 정리하고, 너무 기본적인 내용은 제외하였다.

## REST 구성요소

![](./images/rest_1.png)

### 자원
- HTTP URI를 통해 자원 정보를 명시한다.
- 모든 자원은 서버에 존재하며 고유한 ID가 존재한다.
- `/resource/:id` 형태의 HTTP URI 가 곧 고유한 ID다

### 행위
- 표현 : HTTP METHOD를 이용해 CRUD 동작을 나타낸다.
- `GET` (READ)
- `POST` (CREATE)
- `PUT` (UPDATE)
- `DELETE` (Delete)

### 표현
- 클라이언트에 대한 서버의 응답 포맷이다.
- JSON/XML 형태로 주로 전달한다.

## 규칙
RESTful API의 단점을 굳이 꼽자면 표준이 없다는것이다. 이름에서부터 API 설계 철학이 담겨있듯이, `~ful`이다. REST 스럽게 설계하라는 것이지 정형화 된 것이 아니다. 어찌보면 전 세계적인 API 컨벤션이 있다고 보면 된다.

1. URI는 정보의 자원을 표현해야한다.
- 자원정보는 **명사**를 사용한다.
  - GET /get_movies (X)
  - GET /movies (O)
- 대문자보다는 **소문자**를 사용한다.
  - GET /MOVIES (X)
  - GET /movies (O)
- 컬렉션 이름은 **복수명사**를 사용한다.
  - GET /movie (X)
  - GET /movies (O)
- **HTTP METHOD**가 들어가면 안된다.
  - GET /movies/get/1 (X)
  - GET /movies/1 (O)
  - DELETE /movies/1 (O)


## POST vs PUT
서버쪽에서 `CREATE` API를 제공하는데, `PUT` Method를 이용해서 설계되어 있길래 찾아봤다.  
보통 `POST`를 하지 않나? 생각했는데, 다음과 같은 차이점이 있다.

```node
POST /v1/contents HTTP/1.1

{ "name": "new item" }

HTTP/1.1 201 Created
```

위의 POST 요청을 10번 하면 어떻게될까?
`/contents/1` ~ `/contents/10` 까지 생성되게된다. `/v1/contents` URI를 들여다보면, '리소스의 위치'가 정확하지 않다고 해석할 수 있다. REST API를 작성할 때 POST Method를 이용한 URI 설계시, 리소스 위치를 정확하게 명시하진 않는다. 표준은 아니지만 컨벤션이기 떄문에. 즉, POST Method를 이용한 새로운 자원생성은 끊임없이 새로운 자원을 추가하게 되고, 이를 '멱등하지 않다' 라고 말한다. (하나의 요청이, 하나의 결과를 낳는것이 아니다)

이제 `PUT` Method를 보자.
```node
POST /v1/contents/${contentsNo} HTTP/1.1

{ "name": "new item" }

HTTP/1.1 201 Created
```
`${contentsNo}` 이라는 고유한 자원의 ID를 명시함으로써, 10번 요청해도 결국 멱등한 자원결과를 기대할수 있다. 

근데, 또 아래와같은 경우가 실무에서 있었다.

```node
PUT /v1/sketchbook

{
  "sketchbookNo": 0,
  "contentsNo": 0,
  "type": "jpg",
  "viewImageData": "string",
  "orgImageData": "string"
}
```
위와같이 새로운 `blob`형태의 캔버스데이터를 서버에 저장할 때 사용하던 API가 있다. `id`값을 넘기는게 아닌데 굳이 POST를 쓰지 않고 PUT을 쓰는 이유는 뭘까?

```
resource 존재 ? UPDATE : CREATE
```

위의 이유로 무슨짓을 해도 PUT은 '멱등성을 가진' Method인 것이다. 스케치북 저장값에 대한 리소스를 하나만 가지고 있겠다는 의도일수도 있겠다. 메모리 낭비를 우려한것일수도 있고, 어쨋거나 멱등함을 의도한건 사실일테니..

## OPTIONS, HEAD, PATCH

### OPTIONS
제공가능한 API method 정보를 제공한다

```node
OPTIONS /users/

HTTP/1.1 200 OK
Allow: GET,PUT,DELETE,OPTIONS,HEAD
```

### HEAD
요청에 대한 Header 정보만 응답하고, body가 없다.

```node
HEAD /users/

HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 120
```

### PATCH
PUT 으로 자원의 일부를 수정해야 하는 용도일때, PATCH를 쓰자.
PUT은 거의 CREATE가 되어버린것같다.

```node
PUT /users/1
{
  "age": 30
}

HTTP/1.1 200 OK
{
  "name": null, // 단, 일부를 수정할때도 모든 속성값을 적어주자.
  "age": 30
}
```

## 정리

RESTful 하다는것은 곧 URL + HTTP Method를 이용한 REST API 설계를 지향하자는 개념이다.
프론트엔드 개발자여도, express 서버에서 간단한 구현은 필요하다고 생각한다.


## 기본 응답코드
`1xx` : 전송 프로토콜 수준의 정보 교환
`2xx` : 클라어인트 요청이 성공적으로 수행됨
`3xx` : 클라이언트는 요청을 완료하기 위해 추가적인 행동을 취해야 함 (캐시이슈)
`4xx` : 클라이언트의 잘못된 요청
`5xx` : 서버쪽 오류로 인한 상태코드

## Reference

- https://gmlwjd9405.github.io/2018/09/21/rest-and-restful.html
- https://medium.com/@js230023/url-%EA%B3%BC-uri%EC%9D%98-%EC%B0%A8%EC%9D%B4-154d70814d2a
- https://1ambda.github.io/javascripts/javascript-inheritance/