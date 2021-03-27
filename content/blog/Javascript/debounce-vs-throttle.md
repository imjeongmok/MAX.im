---
title: Debounce vs Throttle
date: 2021-03-27 14:03:51
category: javascript
thumbnail: { thumbnailSrc }
draft: false
---

> 두 함수 모두 이벤트를 제어하는 함수이다. 차이점을 명확히 알고 용도를 구분해서 사용해야한다.


## debounce(fn, delay)

- 동일한 이벤트가 실행될 때, 제일 마지막 이벤트가 터진 후 `delay` 이후에 콜백함수가 실행된다. 
- **autocomplete**, **검색** 기능을 구현할 때, API 요청을 최소화 하기 위해 사용한다.

```javascript
function debounce(cb, delay) {
  let timerID = null;

  return function () {
    /*
    * delay 이내에 함수가 실행되는것을 막기위함.
    * 결과적으로 제일 마지막 setTimeout을 제외하고 모두 호출되지않는다.
    */
    clearTimeout(timerID);

    timerID = setTimeout(() => {
      cb.call(this, arguments);
    }, delay);
  };
}

```



## Throttle(fn, delay)
- 동일한 이벤트가 실행될 때, 일정 주기마다 콜백함수 `fn`을 실행한다.
- **resize**, **scroll** 기능을 구현할 때, 콜백함수의 지나친 실행으로 브라우저 렌더링 성능을 위해 사용한다.



```javascript
function throttle(cb, delay) {
  let isFired = false;

  return function () {
    if (!isFired) {
      cb.apply(this, arguments);
      isFired = true;
    }

    setTimeout(() => {
      if (isFired) isFired = false;
    }, delay);
  };
}


## 소스코드 예제

https://codesandbox.io/s/headless-lake-6dzcr?file=/src/index.js

## Reference

- [lodash](https://lodash.com/docs/4.17.15)

```