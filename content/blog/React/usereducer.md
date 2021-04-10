---
title: useReducer
date: 2021-04-10 16:04:77
category: react
thumbnail: { thumbnailSrc }
draft: false
---

> 공식 홈페이지에는 `useState`의 대체함수라고 표기하고있다. Redux, Context api의 대체제라고 볼수도 있지만 애초에 그 둘도 엄밀히 말하면 상태값을 변경하기 위한 목적이니 대체함수는 맞다. 하지만 내장 Hook 이라는점이 리덕스에 비해 훨씬 매력적이고, Context API의 ShouldComponentUpdate가 미지원되는 점을 극복하여 완벽한 대체제가 될지 궁금하기도 하다.

## 구성

`useState`의 대체함수라고 표기 할 정도로, 기본 구조와 거의 비슷하다.

```javascript
function reducer(state, action) {
  // reducer..
}

const [state, dispatch] = useReducer(reducer, initialState)

const handleAction = () => {
  dispatch({ type: 'ACTION_TYPE' })
}
```

## 비교

[contextAPI](https://max-im.netlify.app/React/context-api/) 예제와 조합하여 전역에서 상태를 제어할 수 있다.

```javascript
const ThemeContext = createContext({})

const TOGGLE_ACTION = 'TOGGLE_ACTION'

// 상태변경을 위한 리듀서. 리덕스와 동일한 매커니즘이다
function reducer(state, action) {
  console.log('dispatched action', action)
  switch (action.type) {
    case TOGGLE_ACTION:
      return { theme: state.theme === 'light' ? 'dark' : 'light' }
    default:
      return { ...state }
  }
}

const App = () => {
  const [state, dispatch] = useReducer(reducer, { theme: 'light' })
  const handleToggleTheme = () => {
    // 액션을 dispatch 하는 형태로 변경되었다.
    return dispatch({ action: TOGGLE_ACTION })
  }

  console.log('[Render App]', state, handleToggleTheme)

  return (
    <ThemeContext.Provider value={{ theme: state.theme, handleToggleTheme }}>
      <Toolbar />
    </ThemeContext.Provider>
  )
}

// 리렌더링이 발생하지 않았으면 하는 컴포넌트
// 메모이제이션 하는것이 최선일수 있다.
const Toolbar = React.memo(() => {
  console.log('[Render Toolbar]')
  return (
    <div>
      <ThemedButton />
    </div>
  )
})

const ThemedButton = () => {
  const { theme, handleToggleTheme } = React.useContext(ThemeContext)
  console.log('[Render ThemedButton]', theme, handleToggleTheme)

  return (
    <>
      <div>{theme}</div>
      <button onClick={handleToggleTheme}>변경</button>
      핸들러
    </>
  )
}
```

## useReducer + middleware Hook

Redux를 쓸 때 사실 비동기 미들웨어를 사용할 수 없을것이라는 단점을 생각했는데, 훅으로 구현하면서 이부분을 더욱 가독성 좋게 유지할 수 있을것같다.
또한 스토어에 액션이 전달될 때, 모든 액션을 미들웨어에서 한번 후킹 할 필요없이, 필요한 리듀서들에게만 적용한다면 더욱 좋지않을까 생각이 든다.

```javascript
const loggerBefore = (action) => {
  console.log("loggerBefore:", action);
};

const loggerAfter = (action) => {
  console.log("loggerAfter:", action);
};


const useReducerWithMiddleware = (
  reducer,
  initialState,
  beforeMiddlewares,
  afterMiddlewares
) => {
  const [state, dispatch] = useReducer(reducer, initialState);

  const dispatchWithMiddleware = (action) => {
    beforeMiddlewares.forEach((beforeMiddleware) => beforeMiddleware(action));
    // 이 사이에 필요에 따라 비동기 함수를 넣을수도 있겠다 (delay)
    dispatch(action);
    // 이 사이에 필요에 따라 비동기 함수를 넣을수도 있겠다
    afterMiddlewares.forEach((afterMiddleware) => afterMiddleware(action));
  };

  return [state, dispatchWithMiddleware];
};

const App = () => {
  const [state, dispatch] = useReducerWithMiddleware( // 위의 useReducer와 차이점
    reducer,
    { theme: "light" },
    [loggerBefore], // dispatch 이전의 미들웨어
    [loggerAfter] // dispatch 이후의 미들웨어
  );
  ...
  return (...)
}
```

## 정리

결국 useReducer는, useState의 대체함수일 뿐인거같다. 하지만 리액트에서 왜 이러한 내장 훅을 도입했는지 고민해봐야할것같다. Redux 라이브러리에 대한 고민일텐데, 전역상태를 구독하는 중첩된 컴포넌트들이 많아짐에 따라 전역 상태관리 라이브러리의 필요성은 어느정도 강제화 되었다. 커링함수나, 기타 리덕스를 위한 초기셋팅이 `useReducer` 함수 내부에 셋팅되어있다는 점에서 많은 편리함이 있는 것 같다. 또한 확연하게 줄어든 코드와 미들웨어를 추가로 후킹할 수 있다는 점에서 더이상 Redux만을 고집할 필요가 없을 것 같다.

## Reference

- https://ko.reactjs.org/docs/hooks-reference.html#usereducer
- https://www.robinwieruch.de/react-usereducer-middleware
