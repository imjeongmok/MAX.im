---
title: JavaScript Pass By Value Function Parameters
date: 2021-07-14 17:07:14
category: Article
thumbnail: { thumbnailSrc }
draft: false
---
> [Kent C. Dodds 블로그 글](https://kentcdodds.com/blog/javascript-pass-by-value-function-parameters)을 번역했습니다.
단순 직역이 아닌 프론트엔드 개발자 입장에서 의역했습니다.

### TL;DR
- 자바스크립트의 함수의 매개변수는 값의 전달이다.
- 매개변수 값의 전달로써 불변값을 넘길수도 있고, 참조값을 넘겨 동기화 시킬 수 있다.

---


아래 코드를 예상해보고, 어떻게 동작하고 다른 코드에 어떤 영향을 주는지 알아봅시다

```javascript
function getLogger(arg) {
  function logger() {
    console.log(arg)
  }
  return logger
}
let fruit = 'raspberry'
const logFruit = getLogger(fruit)
logFruit() // "raspberry"
fruit = 'peach'
logFruit() // "raspberry" 잠깐, 왜 "peach"가 아니지?
```

위에 코드에서 무슨일이 벌어지는 지 얘기하기 위해서, `fruit` 이라는 변수를 생성하고, `raspberry` 문자열을 할당하고, `fruit` 변수를 `logger`를 반환하는 함수의 인자로 넘겼다. (`logger` 함수는 호출할 때 마다 `fruit` 변수를 반환한다.) 해당 함수를 호출하면 예상한 것 처럼 `console.log` 출력값으로 `raspberry`가 출력됩니다.


하지만, `fruit` 변수에 `peach`를 재할당 하고 `logger` 함수를 호출하면, `console.log`의 출력값은 새로 할당된 값(`peach`)이 아닌 예전 값(`raspberry`)이 출력됩니다.

이를 해결하기 위해, `getLogger` 함수를 다시 호출 해 보았습니다.

```javascript
const logFruit2 = getLogger(fruit)
logFruit2() // "peach" 정말 다행이다..
```

하지만, 변수 값을 변경한 뒤 `logger` 함수가 최신 값을 반환할 수 없는건 왜그럴까요?  
정답은, 자바스크립트에서 함수의 `arguments`는 참조값인 아닌 **값 그대로 전달**된다는 사실입니다.


```javascript
function getLogger(arg) {
  function logger() {
    console.log(arg)
  }
  return logger
}
// 참고. 이렇게도 작성할 수 있습니다.
// 그리고 어떠한 차이도 없습니다.
// const getLogger = arg => () => console.log(arg)
// 좀더 장황하게 설명해보겠습니다.

```

`getLogger`함수가 호출될 때, `logger` 함수가 생성됩니다. 해당 함수는 완전히 새로운 함수입니다. 새로운 함수가 생성되고, 주변에 접근 가능한 변수들을 찾고, **클로저**를 형성합니다. 즉, `logger`기능이 존재하는 한, 해당 함수가 생성되는 시점에 접근 가능한 변수를 기억하게 됩니다.

그렇다면, `logger`가 생성될 때 접근 가능한 변수(들)은 무엇일까요? 예시 코드를 다시 보면, `getLogger`, `arg`, `logger`에 접근할 수 있습니다. 이는 코드가 작동하는 방식에서 중요한 단서이므로, 다시한 번 확인해보세요. 뭔가 느낌이 오나요? `fruit`과 `arg`는 정확히 같은 값임에도 불구하고, 중복해서 설명하고있습니다.

`fruit`, `arg` 두 변수가 값으로서는 동일하게 할당되어 있지만, 완벽하게 동일한 변수는 아닙니다.(동일한 주소값을 가리키지는 않습니다.) 다음은 비슷한 개념의 간단한 예시입니다.

```javascript
let a = 1
let b = a
console.log(a, b) // 1, 1
a = 2
console.log(a, b) // 2, 1 ‼️
```

변수 `b`에 변수 `a`를 할당한 뒤, 변수 `a`의 값을 변경하였다 하더라도 변수 `b`에 담겨있는 값이 변경되지는 않습니다. 왜냐면 `let b = a` 구문을 실행할 때, 단순 `a`에 담긴 값의 주소를 할당했을 뿐입니다.

Kent C 저자는 **변수를 컴퓨터의 기억속에 있는 위치를 가리키는 작은 화살표(포인터 개념)로 생각한다고 표현**했습니다. `let a = 1` 구문을 실행하면, "자바스크립트 엔진아, 메모리상에 숫자 `1`을 가리키는 화살표가 변수 `a`를 가리키도록 해줘" 라고 말이죠.

그런 다음 이렇게 말합니다. "자바스크립트 엔진아, `let b = a` 구문을 실행하면 숫자 `1`을 가리키던 화살표를 변수 `a`가 아니라, `b`를 가리키도록 바꿔줘" 라고 말이죠.

마찬가지로, 함수를 호출할 때, 자바스크립트 엔진은 함수의 `arguments`에 대한 새로운 변수를 생성합니다. 해당 예시에서는 `getLogger(fruit)`라고 사용되고있으며, 자바스크립트 엔진은 기본적으로 아래 코드와 같이 내부적으로 동작했습니다.

```javascript
let arg = fruit
```

그렇기 때문에 우리가 `fruit = 'peach'`를 넣게되면, 전혀 다른 변수에 문자열을 대입한것이기 때문에 `arguments`에 전혀 영향을 미치지 않습니다.

이것을 자바스크립트의 한계 또는 기능으로 생각하던 간에, 이것 자체가 작동방식인건 팩트이다. 만약 두 변수값의 씽크를 맞추길 원한다면, 다음과 같은 방법이 있습니다. 변수가 가리키는 위치를 변경하는 대신에, 변수에 넣는 값이 담겨있는 주소값의 위치를 잘 고려해보면 됩니다. (pass-by-reference 개념)

```javascript
let a = {current: 1}
let b = a
console.log(a.current, b.current) // 1, 1
a.current = 2
console.log(a.current, b.current) // 2, 2 🎉
```

위의 코드의 경우에, 변수 `b`에 변수 `a`가 가리키는 객체의 주소값을 재할당했고, 해당 객체가 가리키는 주소의 프로퍼티(`current`)를 수정했습니다. 프로퍼티가 가리키는 값의 주소에 `2`를 재할당했기 때문에 원하는 결과값을 얻을 수 있었습니다.

그렇다면, 이 솔루션을 `logger` 문제에 적용해보겠습니다.

```javascript
function getLatestLogger(argRef) {
  function logger() {
    console.log(argRef.current)
  }
  return logger
}
const fruitRef = {current: 'raspberry'}
const latestLogger = getLatestLogger(fruitRef)
latestLogger() // "raspberry"
fruitRef.current = 'peach'
latestLogger() // "peach" 🎉
```

Ref는 `reference`의 줄임말입니다. 즉, 변수가 가리키는 단순히 다른 값이 담겨있는 주소값을 가리키고 있을 뿐입니다.

---

### 결론

물론 이것은 트레이드오프가 있지만, 자바스크립트의 `arguments`가 참조가 아닌 값 자체가 전달될것을 요구하는것이 매우 기쁘다. 또한, 필요한 경우 객체에 `ref`를 담아 사용하는 방식이 크게 문제가 되지 않습니다. 많은분들에게 도움이 되길 바라고, 행운을 빕니다! 

---


### 번역 후 필자의 사견

어느정도 흥미로운 주제이나 영어로 의미를 풀어쓰다보니, 의역에 조금 어려움이 있었다. 코어 자바스크립트라는 도서에서 값의 할당에 대해 도식화하여 자세하게 설명해준 챕터가 있는데, 예전에 책을 10번정도 읽어서인지 원문을 해석하는 데 많은 도움이 되었다.





