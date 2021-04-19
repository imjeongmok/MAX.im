---
title: Critical Rendering Path
date: 2021-03-20 20:03:96
category: Web
thumbnail: { thumbnailSrc }
draft: false
---

> **주요 렌더링 경로 최적화** 
> HTML, CSS, Javascript 코드를 브라우저가 렌더링 하는 일련의 파이프라인 과정

- DOM : HTML 문서의 객체 모델
- CSSOM : CSS 문서의 객체 모델

![](./images/crp_1.png)  

## DOM (Document Object Model)
1. **변환** : 텍스트 문서를 읽고, 각각의 문자열로 변환합니다.  
  `3C 62 6F` -> `<html><head> ... </head></html>`  
  
    
2. **토큰화** : 변환된 문자열을 W3C 표준에 맞게 토큰으로 변환합니다.  
  `<html><head> ... </head></html>`   
  ->  
  `[StartTag: html][StartTag: head] ... [EndTag: head][EndTag: html]`

3. **렉싱** : 각 토큰을 하나의 노드 객체로 변환합니다.  
  `{ html } { head } ... `

4. **DOM 생성** : HTML 문서 태그간의 관계를 정의하기 위해 트리형태 구조로 연결합니다.

5. 1 ~ 4 과정을 통해 브라우저에서 접근 가능한 DOM 을 출력하고, 자바스크립트를 이용해 접근 및 제어합니다. 하지만 DOM은 '어떻게 렌더링 될지' 에 대한 정보는 알려주지 않고, CSSOM에 의존하여 렌더링됩니다.


## CSSOM (CSS Object Model)
```css
body { font-size: 16px }
p { font-weight: bold }
span { color: red }
```

1. CSS 문서도 HTML 문서와 동일한 과정으로 CSS Object Model을 생성합니다.
2. CSS Object Model이 **왜 트리구조인가**?  
  2.1. 기본적으로 1:1 스타일 매칭을 한다 (`body {} ` < - > `<body>`)  
  2.2. 흔히 **상속**되는 속성을 각 브라우저에서 구현함으로써, '**하향식**'으로 적용되는 규칙을 통해 최적화  


## Redner Tree

![](./images/crp_2.png)
- DOM + CSSOM 결합
- 렌더트리는 **화면에 실제로 렌더링** 할 노드만 포함하여 트리구조로 형성
  - `<script>`, `<meta>` 태그 등 렌더링 출력에 반영되지 않는 노드는 생략합니다.
  - `span { display : none }` 처럼, CSS를 통해 화면에 렌더링되지 않는 노드는 생략합니다. 


## Layout / Reflow
- 레이아웃 / 리플로우 : 노드의 정확한 **위치** 및 **크기(박스모델)** 계산
- 위치와 크기를 계산하기 위해 루트에서 렌더트리를 순회합니다.
- `%`를 이용한 계산식이, **픽셀단위의 절대값**으로 변환됩니다.
- Firefox : 리플로우
- IE, Chrome, Safari, Opera : 레이아웃
- 레이아웃 과정의 비용이 크기때문에, `Composite-Thread`를 사용하는 속성을 이용해서 레이어를 나눠주면 성능개선에 도움이된다. 

#### [레이어 생성 조건]
무조건 빠른건 아니다. **최대 30개** 이내로 생성되도록 생각해서 사용하고, `will-change`를 사용한 GPU 가속을 만능이라 생각하면 안된다.
- position: relative, absolute
- overflow, alpha
- transform
- canvas
- will-change
- etc..


## Painting / Rasterize
- 각 노드의 박스모델과 위치를 참고하여 실제 화면에 픽셀화 하는 작업
- Layout 과정이 완료되면 브라우저가 `Paint Setup`, `Paint` 이벤트를 발생

## Composite
- 위에서 형성된 N개의 레이어들을 합성하여, 한장의 bitmap을 화면에 찍어내는 작업


#### [VSync]
브라우저의 동작 과정 단위인 프레임을 메모리에 저장된 버퍼로부터, GPU 로 fetch 하는데 걸리는 시간이 16.6ms이다.
이를 해석하면 DOM ~ Composite 과정을 16.6ms 이내에 끝내야 렌더링 성능이 보장된다는 것이다. 하지만 현실과 이상은 다르고, 이를 완벽하게 구현해내기 어렵다. 2012 Google I/O 에서 발표된 `VSync` 내용은, 프레임 생성을 16.6ms 단위에 맞게 보정해주는 역할이다. 페인트 과정을 래스터를 나누거나, 구간을 당기는 식으로 최적화한다.

## Over the Main Thread

모던 웹 환경에서 메인쓰레드만을 이용해 16.6ms의 프레임을 구현해내기는 불가능하다. 자바스크립트 자체는 싱글스레드로 동작하지만, 런타임 환경인 브라우저는 다행히 멀티스레드 환경으로 구성되어있다. 

#### [Compositer Thraed]
Layout 단계에서 분리한 레이어들은 컴포지트 쓰레드에서 동작하고, 마지막 컴포지트 과정을 거치며 성능적 이점이 있다. 또한, 브라우저 스크롤, 애니메이션, 줌 인/아웃 이벤트는 컴포지트 쓰레드를 이용하기 때문에 순간적으로 빈 화면을 노출하는것이 아니라 나뉘어진 레이어를 화면에 렌더링하는 것이다.

#### [Raster Thread]
Paint 과정의 일부를 Raster Thread에서 담당한다. 


## 정리
Critical Rendering Path 최적화란, 결국 위 단계의 수행 시간을 최소화 하는 프로세스입니다. 
우리는 Layout, Paint 과정을 줄이고, VSync가 잘 동작하도록 구간별 최적화를 해야한다.
특히 [CSS 트리거](https://csstriggers.com/)를 이용해 컴포지트 쓰레드를 사용하는 속성을 적용하도록 노력해야 하며, 자바스크립트 애니메이션을 구현할 땐 rAF를 최대한 사용해도록 노력해보자.


## plus 이슈

#### [레이아웃 스레싱?]

아래 두 코드는 무슨 차이일까? 둘다 console 함수를 이용해 박스모델에 접근할 뿐이고, 공통점으로는 동기적인 레이아웃이 예상된다는것이다. 하지만 레이아웃 작업이 끝난 후 콘솔을 찍는것과, 레이아웃 전에 콘솔을 찍는건 분명 결과값이 다르고, 이로인한 레이아웃 병목현상이 존재한다는 것이다. 콘솔뿐만 아닌 프로덕션 코드에서도 이러한 코드는 피하자.

```javascript
function logBoxHeight() {
  console.log(box.offsetHeight);

  box.classList.add('super-big');

  // Gets the height of the box in pixels
  // and logs it out.
}

/* 개선코드 */
function logBoxHeight() {
  box.classList.add('super-big');

  // Gets the height of the box in pixels
  // and logs it out.
  console.log(box.offsetHeight);
}

```

```javascript
function resizeAllParagraphsToMatchBlockWidth() {

  // Puts the browser into a read-write-read-write cycle.
  for (var i = 0; i < paragraphs.length; i++) {
    paragraphs[i].style.width = box.offsetWidth + 'px';
  }
}

/* 개선코드 */
var width = box.offsetWidth; // 레이아웃 스레싱을 피하기 위함.

function resizeAllParagraphsToMatchBlockWidth() {
  for (var i = 0; i < paragraphs.length; i++) {
    // Now write.
    paragraphs[i].style.width = width + 'px';
  }
}

```

## Reference
- [MDN 객체모델 생성](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/constructing-the-object-model?hl=ko)

- [렌더링 트리 생성, 레이아웃 및 페인트](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/render-tree-construction?hl=ko)
- [브라우저 동작원리](https://coffeeandcakeandnewjeong.tistory.com/55)
