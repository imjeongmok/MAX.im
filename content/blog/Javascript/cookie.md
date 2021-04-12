---
title: Cookie
date: 2021-04-12 22:04:76
category: javascript
thumbnail: { thumbnailSrc }
draft: false
---

> 쿠키는 기본적으로 서버가 생성하고, 클라이언트에게 전달한다. 일반적으로 클라이언트는 쿠키를 전달받아 브라우저에 저장하고, 동일한 서버에 요청을 보낼 때 HTTP 헤더에 조건에 맞는 쿠키를 포함시켜 전송한다. 

## 특징
쿠키는 브라우저에 쌓이는 작은 데이터 조각이다. 기본적으로 브라우저가 서버에게 무언가 요청할 때, 현재 위치가 저장된 쿠키의 `path`, `domain` 조건을 충족한다면 `Request Header`에 포함시킨다. 아래 코드를 보면 다양한 특징을 확인할수있다.

```javascript

http.createServer(function(request, response) {
  response.writeHead(200, {
    'Set-Cookie': [
      'yumm_cookie=choco',  // 만료 기간이 없다면 세션용도로 사용된다.
      `Permanent=cookies; Max-Age=${60 * 60 * 24 * 30}`, // 반영구적인 쿠키
      'Secure=Secure; Secure', // https 프로토콜일때만 request 헤더에 담긴다.
      'HttpOnly=HttpOnly; HttpOnly', // JS로 해당코드를 가져올수없다.
      'Path=Path; Path=/cookie',// 특정 path 및 path 하위에서만 확인이 가능하도록 옵션을 줄수있다.
      'Domain=Domain; Domain=jr.naver.com' // jr.naver.com 도메인 및 서브도메인 모두 가능하다.
    ]
  });

  response.end('cookie!');
}).listen(3999);
```

## Key-Point

- 쿠키를 삭제할 땐 `Max-Age=0` 을 사용한다.

- `Secure` 쿠키는 `HTTPS` 프로토콜을 사용하는 요청일때만 전송된다. 그렇다고 무조건 안전한건 아니기 때문에 민감한 정보는 담으면 안된다.

- `HttpOnly` 쿠키는 브라우저에서 `document.cookie` API를 이용해 접근이 불가능하다. 실제 프로젝트에서도 경험해본 적이 있었는데, 처음엔 버그인줄 알았는데 공부해보니 이 용도는 서버쪽에서 사용할 용도의 세션이라고 한다.

- `Path`는 쿠키가 저장되는 디렉토리를 명시한것이다. 또한, 서브 디렉토리에서는 접근 가능한 특징이 있다.

- `Domain`을 설정할 경우 도메인별 저장소를 분리시킬 수 있고, 서브 도메인에서 접근이 가능한 특징이 있다.


## Reference
- https://developer.mozilla.org/ko/docs/Web/HTTP/Cookies
