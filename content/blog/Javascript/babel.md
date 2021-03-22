---
title: Babel
date: 2021-03-22 22:03:11
category: javascript
thumbnail: { thumbnailSrc }
draft: false
---

> 바벨은 ECMA2015+(ES6+) 코드를 원하는 버전의 자바스크립트로 변환하는 데 사용되는 **자바스크립트 트랜스컴파일러**다. 일반적인 컴파일러와 다르게 **입력과 출력이 모두 자바스크립트**로 이루어져며, 공식 홈페이지에서도 'Javascript Compiler' 라고 쓰여잇다.

## 바벨의 컴파일 단계

- `파싱` : 입력된 자바스크립트 코드를 기반으로 AST를 생성
- `변환` : AST를 원하는 형태로 변환
- `생성` : 변환된 AST를 자바스크립트 코드로 출력  
  
<br />
❗️ AST는 코드의 구문(syntax)이 분석된 결과를 담고 있는 구조체다. 코드가 같다면 AST도 같기 때문에 같은 코드에 대해서 하나의 AST를 만들어 놓고 재사용할 수 있다.


<br />
<br />

❓ Polyfill
- 폴리필은 런타임 환경에 필요한 기능을 주입하는것이다.
- 바벨은 `컴파일 시점`에 하위브라우저에서 사용가능한 코드를 변환해준다,
- 폴리필은 `런타임 시점`에 현재 브라우저에 필요한 코드를 추가해준다.

## 바벨을 위한 Dependency
바벨은 컴파일러일 뿐, 해당 컴파일러를 어떻게 사용하는지는 여러가지 방법이 있다. `Gulp`, `webpack`이 대표적으로 있고, `webpack`을 이용한 모듈 번들링에선 거의 필수적으로 사용된다.
`webpack`을 이용한 모듈 번들링에서 빠질 수 없는것이 바벨이다. `package.json`에서 일반적으로 사용 될 만한 의존모듈들을 이해하고 넘어가자. 

### @babel/core
설정파일을 바탕으로 **코드를 변환하는 API** 를 제공한다. 실제 컴파일을 담당하는 모듈이다.

```javascript
import * as babel from '@babel/core';

babel.transform(code, options, (err, result) => {
  console.log(result); // => { code, map, ast }
});
```

---- 
### @babel/cli
바벨 코어의 **API를 직접 접근하지 않고, babel 명령어를 사용**할 수 있도록 도와주는 모듈이다.

```javascript
// test.js
let a = [1,2,3];
a = [...a, 4, 5, 6];

```

```node
yarn babel test.js
```

```javascript
// bundle.js
function _toConsumableArray(arr) { ... }

var a = [1, 2, 3];
a = [].concat(_toConsumableArray(a), [4, 5, 6]);

```

---- 
### @babel/preset-env
**ECMAScript 버전별 문법들이 모아져있는 프리셋 집합**이다. 불과 몇년전만 해도 babel-preset-es2015, babel-preset-es2016 ... 처럼 ECMAScript 버전별 프리셋이 존재하여 모듈 번들설정이 매우 복잡했다. 바벨 7 버전 이후부터 @babel 스코프를 사용하기 시작하여 지금의 종합 프리셋이 탄생하게되었다. 즉, 컴파일을 명령하는것이 코어라면, 실제로 '어떻게' 변환할지는 프리셋의 역할이다.

```node
yarn babel sample.js --presets=@babel/env
```

---- 
### .babelrc.json ( === .babelrc )
바벨과정의 수고를 덜어주는 config 파일이다. 위에 cli 명령시 옵션값으로 전달한 presets 정보를 적어줄 수 있다.

```javascript
{
  "presets": ["@babel/env"]
}
```

---- 
### @babel/polyfill
바벨 7.3 버전까지 공식으로 지원하다가 depreacted 되었다. 7.4 버전부터 `core-js` 를 사용하라고 권하고있다. 브라우저 런타임환경에서 지원하지 않는 자바스크립트를 주입한다고 이해하면된다. 보통 웹팩 모듈의 `entry`에 추가해서 간편하게 사용하지만, 모든 폴리필이 추가되기때문에 사용하는 폴리필만 `core-js`를 이용해 나누어서 사용하면 성능적인 이점을 볼 수 있다. 

```node
module.exports = {
    entry: ['@babel/polyfill', './src/index.js'], // 모든 폴리필을 로드함
    ...
}
```  

```javascript
const exist = arr.includes(20);

> 런타임환경에서 폴리필을 이용해 미지원 코드가 주입된다.
import _includesInstanceProperty from "@babel/runtime-corejs3/core-js-stable/instance/const exist = includes"; _includesInstanceProperty(arr).call(arr, 20);
```

---- 
### 잠시 컴파일, 트랜스파일, 폴리필링 용어에 대하여
- 컴파일 : **A** 라는 언어가 **B** 라는 언어로 변환되는 것 (`javascript` -> `bytecode`)
- 트랜스파일 : **A** 라는 언어가 **A'** 언어로 변환되는 것 (`es6` -> `es5`, `sass` -> `css`)
- 폴리필링 : 현재 브라우저에서 지원하지 않는 함수를 주입하는 것  

<br />

❗️**transpiler** = source to source **compiler**  
즉, 컴파일러가 트랜스파일을 포함하고 있는 개념이다.

## 정리
바벨은 결국 컴파일 도구다. 개발환경과 용도를 정확히 알아야 산출물을 효율적으로 관리할 수 있다는것을 명심하자.

## Reference

- https://babeljs.io/docs/en/