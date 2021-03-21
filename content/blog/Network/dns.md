---
title: DNS
date: 2021-03-21 16:03:53
category: network
thumbnail: { thumbnailSrc }
draft: false
---



## 도메인과 호스트
![](./images/dns0.png)

## WWW.EXAMPLE.COM 을 입력하면 무슨일이 발생할까
![](./images/dns1.png)
1. 입력한 도메인에 대한, 로컬환경의 IP 정보를 검색한다
    - `cached IP` 검색
    - `hosts file` 검색
2. cached IP 혹은 host 파일의 도메인 매핑이 없다면, `Local DNS` 서버로 요청
3. Local DNS 서버에 해당 도메인에 대한 IP정보가 없다면
    - `ROOT DNS` 서버로 요청
    - `TLD(top-lvel) DNS` 서버로 요청
    - `SLD(second-level) DNS` 서버로 요청
    - 하위 DNS 서버로 재귀 요청하는 `DNS Recursive Query` 수행
4. Local DNS 서버는 IP 주소를 자체 캐시에 저장해두고, 클라이언트에게 제공
5. 클라이언트는 IP주소를 캐시에 저장하고 프로토콜을 해석하여 요청을 수행
6. 웹 서버에 정적 리소스를 요청하고, 화면을 렌더링한다


## Reference
- http://www.codns.com/b/B05-195
- https://aws.amazon.com/ko/route53/what-is-dns/