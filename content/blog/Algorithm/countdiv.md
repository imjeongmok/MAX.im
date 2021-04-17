---
title: CountDiv
date: 2021-04-17 11:04:17
category: Algorithm
thumbnail: { thumbnailSrc }
draft: false
---

## 문제
https://app.codility.com/programmers/lessons/5-prefix_sums/count_div/

A ~ B 사이의 숫자를, K로 나눈 나머지가 0인 값의 카운트를 구하라

## 최초 풀이
처음엔 그냥 A ~ B 사이의 모든 `mod`값을 구했다.  
이렇게되면 `O(B-A)` 가 되는바람에 효율성이 0점이었다.


```javascript
function solution(A, B, K) {
    let cnt = 0;

    for(let i = A; i <= B; i++) {
        if( i % K === 0 ) cnt++; 
    }

    return cnt;

}
```

## 개선된 풀이
나누기와 모듈러 연산은 이어진다고 생각해보자.  
8 / 2 = 4 다.

8로 나누어지는 2, 4, 6, 8 4가지가 존재하는것이며,  
동시에 8 % 2 = 0 인 케이스가 4개가 존재한다고 볼 수 있다.  
즉, 1 ~ 8 까지의 모듈러 연산 갯수는 총 4개다.  

결과적으로,  
B를 나누기 한 몫 = B를 모듈러한 갯수    
A를 나누기 한 몫 = A를 모듈러한 갯수  
**B를 나누기 한 몫 - A를 나누기 한 몫 = A ~ B 사이 숫자를 모듈러한 갯수**  
라고 볼 수 있다.  

마지막으로 A ~ B 사이의 값을 모듈러한 값이기 때문에, A 를 한번 더 체크해주면 `O(1)`의 퍼포먼스를 얻을 수 있다.

```javascript
function solution(A, B, K) {
  let count = Math.floor(B/K) - Math.floor(A/K);
  return A % K === 0 ? count + 1 : count;
}
```

