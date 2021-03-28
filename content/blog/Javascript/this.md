---
title: This
date: 2021-03-28 12:03:70
category: javascript
thumbnail: { thumbnailSrc }
draft: false
---
> this는 실행 컨텍스트가 생성될 때 this로 지정된 객체 정보가 저장되며, 함수를 호출하는 '**방법**'에 따라 바인딩 되는 대상이 다르다.

- 일반 함수 호출
- 생성자 함수 호출
- 객체 메서드 호출
- 이벤트 리스너 호출
- call, apply, bind


## 일반 함수 호출
브라우저에서 일반함수 호출시, this는 항상 `Window`를 바인딩한다.(NodeJS 환경이라면 `global`)

```javascript
var obj = {
    a: 10,
    b: function() {
        console.log('func b', this, this.a);
    }
}

var fn = obj.b;
b() // Window
```

## 생성자 함수 호출
생성자 함수 내에서 호출시, this는 항상 `생성자 함수` 스스로를 바인딩한다.

```javascript
function Person() {
  this.name = 'person';
  this.getName = function() { return this.name };

  console.log('this', this); // Person {name: "person", getName: ƒ}
}

var p = new Person();
```

## 객체 메서드 호출
메서드로서 함수호출시, 자신을 호출한 `인스턴스` 를 바인딩한다. 하지만 이때 함수를 선언한 ECMA 문법에 따라 다르게 바인딩 될 수 있는 예외가 있다.

```javascript
var obj = {
  a: 10,
  b: function() {
    console.log('func b', this, this.a);
  }
}

obj.b() // {a: 10, b: ƒ} 10

```

```javascript
var obj = {
  a: 10,
  b: () => { // 화살표함수를 이용할 경우 this는 자신을 감싼 상위 스코프를 가리킨다.
    console.log('func b', this, this.a);
  }
}

obj.b() // Window undefined
```


## 이벤트 리스너 호출
이벤트 리스너로 함수를 바인딩할 경우, this는 바인딩 된 돔 객체를 가리킨다.
하지만 돔 객체를 이용한 스크립트를 작성할 때 `eventTarget` 혹은 `currentTarget`을 이용해서 개발해야 한다.  
아래 화살표함수를 이용한 이벤트함수 등록시 의도치 않은 바인딩이 되는걸 예시로 참고하자.

```javascript
const element = document.querySelector('.element');
element.textContent = 'Div Element'

element.addEventListener('click', function() {
  console.log('this', this); // Div Element
});


// 화살표함수를 이용할 경우, this 바인딩이 되지 않은 채, 자신을 감싼 상위스코프를 가리킨다.
element.addEventListener('click',() => {
  console.log('this arrow', this.textContent); // undefined (Window.textContent)
});

```

## call, apply, bind
`call(arg: arg1: unknown, arg2: unknown, ...)`, `apply(arg: unknown[])` 를 이용해 `this`값을 바인딩하여 즉시 실행하거나, `bind` 함수를 이용해 `this`값을 바인딩 한 함수를 반환받을 수 있다.

```javascript
var obj = {
  a: 10,
  b: function() {
    console.log('func b', this, this.a, arguments);
  }
}

var obj2 = {
  a: 20
}

obj.b.call(obj2, 'hi'); // func b {a: 20} 20 Arguments ["hi", ... ]
obj.b.apply(obj2, [1,2,3]); // func b {a: 20} 20 Arguments(3) [1, 2, 3, ...]
obj.b.bind(obj2, 'binded')(); // func b {a: 20} 20 Arguments ["binded", ...]

```

## Reference
- https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/this


