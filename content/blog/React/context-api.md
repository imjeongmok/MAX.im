---
title: (State Managerment - 1) Context API
date: 2021-04-10 15:04:65
category: React
thumbnail: { thumbnailSrc }
draft: false
---

> 리액트 컴포넌트 개발을 진행하다보면, 중첩된 컴포넌트 간 상태값을 관리함에 있어서 고민하게된다. 상태값을 어느 레벨의 컴포넌트에서 관리 할 것인지, 어느 수준의 레벨까지 props로 내려줄 지에 대한 결정을 해야한다. 또한 중첩된 컴포넌트에게 상태변화 핸들러를 넘겨줘야하는데, 중첩 단계가 깊어질수록 유지보수는 어려워지고 코드는 복잡해진다. 대표적으로 리액트와 잘 어울리는 상태관리 라이브러리 중 하나인 Redux를 많이들 사용하는데, 사용하다보면 리액트보다 더욱 어려운게 리덕스가 아닌가 싶다. 단지 상태값을 전역에서 관리하기위해 리덕스를 이용한다면, 액션, 리듀서, 디스패치 등 다양한것들을 배워야하고, 러닝커브도 만만치 않다. 이를 고민하던 중 리액트는 context를 추가하였고, 전역에서 상태값을 관리할 수 있게 지원하고있다.

## Context API

`React.createContext(기본값)`
컨텍스트 객체를 생성하여 Provider, Consumer를 생성한다.

- `Provider`: 컨텍스트를 구독하는 컴포넌트들에게 상태변화를 알려주는 컴포넌트
- `Consumer`: 컨텍스트 변화를 구독하여 상태가 변하면 리렌더링하는 컴포넌트

아래 코드는 리액트의 공식문서에서 가져온 코드다.

```javascript
// 전역에서 사용할 컨텍스트 생성
const ThemeContextObject = { theme: 'light', handleToggleTheme: () => {} } // 기본 fallback 값
const ThemeContext = React.createContext(ThemeContextObject)

// 전역 컨텍스틀 제공할 상위 컴포넌트
const App: React.FC = () => {
  const [theme, setTheme] = useState('light')
  const handleToggleTheme = () => {
    setTheme(theme === 'light' ? 'dark' : 'light')
  }

  // 컨텍스트를 구독할 컴포넌트가 있다면, Provider로 감싸줘야한다.
  return (
    <ThemeContext.Provider value={{ theme, handleToggleTheme }}>
      <Toolbar />
    </ThemeContext.Provider>
  )
}

// 중간에 필요없는 중첩된 컴포넌트
const Toolbar: React.FC = () => {
  return (
    <div>
      <ThemedButton />
    </div>
  )
}

/**
 실제 컨텍스트 상태값을 받아야 할 Consumer컴포넌트
 구독중인 컴포넌트를 설계하는 방법은 여러 방법이다.
*/
// 컴포넌트를 이용한 컨텍스트 참조
class ThemedButton extends Component {
  static contextType = ThemeContext

  render() {
    const { theme, handleToggleTheme } = this.context
    return (
      <>
        <div>{theme}</div>
        <button onClick={handleToggleTheme}>변경</button>
      </>
    )
  }
}
// Consumer를 직접 컨트롤하는 방법
const ThemedButton = () => {
  return (
    <ThemeContext.Consumer>
      {({ theme, handleToggleTheme }) => {
        return (
          <>
            <div>{theme}</div>
            <button onClick={handleToggleTheme}>변경</button>
          </>
        )
      }}
    </ThemeContext.Consumer>
  )
}

// useContext hook을 사용하는 방법
const ThemedButton: React.FC = () => {
  const { theme, handleToggleTheme } = React.useContext(ThemeContext)

  return (
    <>
      <div>{theme}</div>
      <button onClick={handleToggleTheme}>변경</button>
      핸들러
    </>
  )
}
```

## Context-API와 성능

공식문서에서 Provider 하위에서 컨텍스트를 구독하고 있는 모든 컴포넌트는 Provider의 전역 상태값이 변경될 때 마다 모두 리렌더링된다고 한다.
그렇기때문에 전역컨텍스트 객체를 기능에따라 분리하거나, `React.memo`를 이용해서 렌더링 성능에 주의하여 사용해야한다. 위 코드를 이용해 성능을 개선해보자.

```javascript
const App: React.FC = () => {
  const [theme, setTheme] = useState('light')
  const handleToggleTheme = () => {
    setTheme(theme === 'light' ? 'dark' : 'light')
  }

  return (
    <ThemeContext.Provider value={{ theme, handleToggleTheme }}>
      <Toolbar />
    </ThemeContext.Provider>
  )
}

// 리렌더링이 발생하지 않았으면 하는 컴포넌트 설계 1
const Toolbar: React.FC = React.memo(() => {
  console.log('[Render Toolbar]')

  return (
    <div>
      <ThemedButton />
    </div>
  )
})

// 리렌더링이 발생하지 않았으면 하는 컴포넌트 설계 2
class Toolbar extends React.Component {
  static contextType = ThemeContext

  shouldComponentUpdate() {
    return false
  }

  render() {
    console.log('[Render Toolbar]')

    return (
      <div>
        <ThemedButton />
      </div>
    )
  }
}

const ThemedButton: React.FC = () => {
  const { theme, handleToggleTheme } = React.useContext(ThemeContext)

  return (
    <>
      <div>{theme}</div>
      <button onClick={handleToggleTheme}>변경</button>
      핸들러
    </>
  )
}
```

## Provider 중첩시 발생하는 강한 Coupling
다수의 Provider를 사용하게되면, 자연스럽게 Provider끼리 감싸고 감싸진 구조가 형성된다.  
최상위 ResizeContext가 변경이된다고 할 때, 그렇다면 자연스럽게 ThemeContext를 구독하는 Consumer 컴포넌트도 리렌더링 된다. 리사이즈 경우 전역에서 상태값을 관리해야 하는 경우가 발생하는데, 최상위 Provider가 되어야 한다. 하지만 그 결과 리사이즈가 될 때 중첩된 Provider의 Consumer 컴포넌트가 모두 리렌더링되는 참사가 벌어진다. 이 경우 React.memo를 이용해 리렌더링을 막을수도 없다. 어찌보면 리덕스로 빼버리는게 답일 수 있다.


[예제코드](https://codesandbox.io/s/vigilant-forest-hbem4?file=/src/App.js)
```javascript

<ResizeContext.Provider>
  <ThemeContext.Provider>
    {/* Consumer List */}
    <ThemeContextConsumer> {/* 리렌더링되지 않길 바라지만 리렌더링된다. */}
      <ResizeContextCounsumer> {/* 실제 리렌더링 목적의 컴포넌트 */}
    </ThemeContextConsumer>>

  </ThemeContext.Provider>
</ResizeContext.Provider>

```

## 결론

결론부터 말하면, `React.memo` 혹은 `useMemo`를 사용하여 컴포넌트를 생성하면, 전역 컨텍스트의 상태값이 변경되어도 리렌더링되지 않는다.
하지만 공식홈페이지에서 언급한것처럼, 클래스형 컴포넌트와 `shouldComponentUpdate(scu)`, 혹은 `PureComponent(PC)`를 사용한 컴포넌트 생성으로 리렌더링을 막을 순 없었다.
모든 중간에 중첩된 컴포넌트를 메모하는건 어떨까? 당연히 메모리낭비를 불러온다. 메모이제이션은 무적이 아니다. 이럴때 차라리 리덕스를 사용하고 기존 컴포넌트는 `scu`, `PC`를 이용해 구현하는게 조금더 올바른 선택지 아닐까 싶다.

## Reference

- https://ko.reactjs.org/docs/context.html
- https://medium.com/react-native-seoul/context-api-%EC%96%B4%EB%96%BB%EA%B2%8C-%EC%82%AC%EC%9A%A9%ED%95%B4%EC%95%BC-%ED%95%98%EB%8A%94%EA%B0%80-9ef90247713
