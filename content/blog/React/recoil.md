---
title: (State Managerment - 3) Recoil
date: 2021-04-13 22:04:23
category: React
thumbnail: { thumbnailSrc }
draft: false
---

> 상태관리 마지막 방법. 리덕스를 쓰지않고 리코일까지 왔다. Recoil은 페이스북에서 직접 만든 상태관리 라이브러리다. 사실 리액트를 만든곳에서 제작할 정도의 상태관리 라이브러리라면, 리덕스를 대체하기 위한 목적이라고 봐도 매우 무방하다. <del>사실상 그게 아니면 뭐하러 만들었나?</del> 아직은 완벽한 릴리즈가 아니긴 하지만, 페이스북에서 만들었다면 미리 알아보는것도 나쁘지 않다.

## Redux, Mobx, Context API의 단점
`Redux`, `Mobx`는 다시 말하지만 리액트 상태관리 라이브러리가 아니다. 그냥 웹 애플리케이션을 제작할때 사용할 수 있는 상태관리 라이브러리 일 뿐이고, 리액트 환경에서 지원되는 부분이 많을 뿐이다. 이번에 등장하는 `Recoil`과 가장 큰 차이점이라 함은, 내부적으로 React의 상태를 사용한다는 점이다. 또한 상태관리를 위해 너무 작성할 코드가 많다. 배 보다 배꼽이 더 커진 경우라고 볼 수 있다.(물론 리덕스 사용패턴에 따라 다르긴 하다.)
또한, [`ContextAPI`의 경우 페이스북에서도 이미 애플리케이션의 상태관리를 목적으로 적극 권장하는 분위기는 아니며 낮은 빈도의 업데이트에서 사용하라고 추천하고있다.](https://github.com/facebook/react/issues/14110#issuecomment-448074060)


## Recoil 구조

- `Atom`: 하나의 전역 객체 단위.
- `useRecoilState`: useState 와 동일한 방식이다. atom 값을 구독하고 업데이트한다.
- `useRecoilValue`: atom의 값을 반환.
- `useSetRecoilValue`: atom의 값을 setter function 반환.

4가지 키워드를 알아봤는데, 벌써 사용하기 쉬워보인다. 코드를 보면 바로 쓰자고 얘기가 나올정도다.

```javascript
import Recoil from 'Recoil';

const nameState = Recoil.atom({
  key: 'nameState', // atom 객체의 키
  default: 'Jane Doe' // atom 객체의 value
});

const NameInput = () => {
  const [name, setName] = Recoil.useRecoilState(nameState);
  const onChange = (event) => {
   setName(event.target.value);
  };
 
  return <>
   <input type="text" value={name} onChange={onChange} />
   <div>Name: {name}</div>
  </>;
}

// useRecoilValue
const SomeOtherComponentWithName = () => {
  const name = Recoil.useRecoilValue(nameState); // name atom 참조
  return <div>{name}</div>;
}

// useSetRecoilState  
const SomeOtherComponentThatSetsName = () => {
  const setName = Recoil.useSetRecoilState(nameState); // name atom update
  return <button onClick={() => setName('Jon Doe')}>Set Name</button>;
}

```

- `selector`: Redux의 `reslect` 개념을 떠올리면 될거같다. 복잡한 계산을 캐싱해준다.

```javascript
import Recoil from 'Recoil';

// 동물 목록 상태
const animalsState = Recoil.atom({
  key: 'animalsState',
  default: [{
    name: 'Rexy',
    type: 'Dog'
  }, {
    name: 'Oscar',
    type: 'Cat'
  }],
});

// 필터링 동물 상태
const animalFilterState = Recoil.atom({
 key: 'animalFilterState',
 default: 'dog',
});

// 파생된 동물 필터링 목록
const filteredAnimalsState = Recoil.selector({
 key: 'animalListState',
 get: ({get}) => {
   const filter = get(animalFilterState); // 'dog'
   const animals = get(animalsState); // animalsState.default 배열
   
   return animals.filter(animal => animal.type === filter);
 }
});

// 필터링된 동물 목록을 사용하는 컴포넌트
const Animals = () => {
  const animals = Recoil.useRecoilValue(filteredAnimalsState);
  return animals.map(animal => (<div>{ animal.name }, { animal.type}</div>));
}

```

## 정리
Redux를 이제는 걷어내고, Recoil이 완전 도입되어 코드가 더욱 간결해지고 직관적이게 발전했으면 좋겠다.


## Reference
- https://medium.com/swlh/recoil-another-react-state-management-library-97fc979a8d2b