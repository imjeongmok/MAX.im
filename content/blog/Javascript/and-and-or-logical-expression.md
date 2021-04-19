---
title: AND and OR logical expression
date: 2021-04-20 00:04:42
category: javascript
thumbnail: { thumbnailSrc }
draft: false
---

> 논리적인 비교를 위해 사용하던 표현식을 좀더 깊게 파봐야만 한다. 대충 쓰지 말자
보통 AND의 경우 모두 참인지를 판단하기 위해 사용하고,
OR의 경우 둘 중 하나만 참인지 판단하기 위해 사용한다.
하지만 정확한 표현식의 스펙은 아래와 같다.

## AND

`A && B`
#### if (A === true) return B 
- A가 `true`로써 변환된다면, B를 리턴한다.
- 앞이 참이라면 무조건 뒤의 값을 리턴한다.

#### if (A !== true) return A 
- A가 `false`로써 변환된다면, A를 리턴한다.
- 앞이 거짓이라면 뒤는 비교도 안하고 앞의 값을 리턴한다.

```javascript
true && true // true
true && false // false
false && false // false

true && 3 //3
true && 'hello' //hello
'hello' && 'world' //world
3 && 100 // 100
3 && console.log("3 출력") // 3 출력
true && return <div></div> // div 템플릿 출력

undefined && true // undefined
null && true // null
0 && true // 0 
console.log('3 출력') && 3 // 3 출력 (console.log는 undefined를 반환한다.)
```

## OR

`A || B`
#### if (A === true) return A
- A가 `true`로써 변환된다면, A를 리턴한다.
- 앞이 참이라면 뒤는 비교도 안하고 앞의 값을 리턴한다.

####  if (A !== true) return B
- A가 `false`로써 변환된다면, B를 리턴한다.
- 앞이 거짓이라면 무조건 뒤의 값을 리턴한다.

```javascript
true || false // true
false || true // true
false || false // false

'hello' || 'world' //hello
'hello' || null // hello
3 || false // 3

false || 'hello' //hello
null || false // false
null || undefined // undefined
false || 0 // 0 
```


## Reference
- https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_OR
- https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_AND
