# React Hooks를 사용한 Micro State Management란?

## Intro

React hooks가 나타나면서 훅을 사용해 상태 관리를 더욱 가볍게, 마이크로하게 할수 있게 되었다.

기존의 모놀리식은 범용적이었다면, Micro State Management는 더욱 목적 지향적이고 특정 코딩 패턴과 함께 사용될수 있다.

React hooks는 전역 상태를 직접 제공하는 기능이 없으므로 우리는 이에 대해 논하게 된다.

이번 챕터에서는 아래의 6가지를 목표로 한다.

- [Micro state management 이해](#micro-state-management-이해)
- [Hooks로 작업하기](#hooks로-작업하기)
- [글로벌 상태 알아보기](#글로벌-상태-알아보기)
- [useState 이해하기](#usestate-이해하기)
- [useReducer 이해하기](#usereducer-이해하기)
- [useState vs useReducer](#usestate-vs-usereducer)

## Micro state management 이해

리액트에서 **상태(state)**는 사용자 인터페이스를 나타내는 **모든 데이터**이다. 리액트는 상태와 함께 렌더링할 컴포넌트를 처리한다.

Hooks 이전에는 모놀리식 상태 라이브러리들이 유행이었다. 이는 DX(Developer Experience) 항샹에 큰 도움을 주었지만 사용되지 않는 기능들도 함께 포함해야 한다는 문제점이 있다.

이는 아래의 경우에서 문제점들이 있다.

1. Form 상태는 글로벌 상태에서 분리되어야 하지만, 싱글 스토어에서는 불가능하다.
2. 서버 캐시 상태는 refetching과 같은 고유한 특징이 있으나 싱글 스토어에는 함께 존재해야 한다.
3. 네비게이션 상태는 브라우저 측에서만 쓰이는 데이터이므로 단일 상태 솔루션으로 적합하지 않다.

Hooks로 인해 위의 문제점을 처리할수 있다.

앱에 따라서 프론트에 상태가 많을수 있고 서버 데이터만 조금 필요할수도 있다. 따라서 보편적인 상태 관리를 위해선 경량화 되어야한다.

Micro state management는 아래의 요구조건을 충족하여야 한다.

1. Read state
2. Update State
3. Render with state
4. Optimize re-renders
5. Interact with other systems
6. Async support
7. Derived state
8. Simple syntax

## Hooks로 작업하기

커스텀 훅을 사용하여 기본 훅을 래핑해 동일한 동작을 하는 다양한 커스텀 이름을 지정할 수 있으며 다양한 컴포넌트에서 재사용 가능하다. 또한 커스텀 훅이 변경되어도 기본 컴포넌트 로직에는 영향을 주지 않으므로 라이브러리 관점에서 특히 중요하다.

이는 우리가 목표로하는 Micro state management의 첫 시작이다.

훅을 이해하기 전에, 아래 두 가지를 유의해야한다.

- Suspense for Data Fetching
  - async와 같은 비동기에 걱정 없이 컴포넌트를 코딩 해야하는 메커니즘
- Concurrent Rendering
  - 렌더링 프로세스를 여러 청크로 분할해 장기간 blocking 되는 것을 방지하는 메커니즘

리액트 훅은 위의 두 가지를 기본 전제로 만들어졌다. 훅 및 함수 컴포넌트는 여러번 호출될수 있기 때문에, 우리는 기존에 존재하는 state 혹은 ref 값을 변경해선 안된다. 어길 경우 re-render 혹은 trigger 되지 않는(not render) 문제를 직면할수 있다. 그러므로 여러번 호출되어도 일관된 동작을 할 수 있도록 훅은 순수함수여야 한다.

## 글로벌 상태 알아보기

글로벌 상태란 다양한 컴포넌트에서 공유되는 상태이다. 공유되는 상태이기 때문에 엄연히 따져 싱글톤이 아니며, 싱글톤일 필요도 없다.

리액트는 기본적으로 컴포넌트 모델을 따르고 있다. 컴포넌트 모델에서는 지역화가 중요한데, 이 뜻은 컴포넌트는 격리되어야 하며 재사용 가능해야 한다.

이 때문에 리액트는 글로벌 스테이트를 제공해주지 않는다. 우리는 이를 해결하기 위한 다양한 장단점 솔루션들을 이해해야 한다.

## useState 이해하기

```jsx
// CASE 1.
const [count, setCount] = useState(0);
setCount(1);
setCount(1); // Not render. Bailout. 동일한 값이므로 렌더링 하지 않는다.

// CASE 2.
const [state, setState] = useState({ count: 0 });
setState({ count: 1 });
setState({ count: 1 }); // Re-render! 주소 참조는 항상 다른 값이 된다.

// CASE 3.
state.count = 1;
setState(state); // Not render. Bailout. state 주소 값은 변하지 않았기 때문에 렌더링하지 않는다.

// CASE 4.
setCount(count + 1);
setCount(count + 1); // 빠르게 클릭하는 등으로 동일하게 두번 호출되면 +2가 아닌 +1만 될수 있다. 이를 해결하기 위해선 함수 업데이트가 필요하다.

// CASE 5.
setCount((prev) => prev + 1); // 함수로 작성하게 될 경우 아무리 빠르게 눌러도 횟수만큼의 업데이트가 될것을 보장한다. 이는 내부적으로 함수를 연속적으로 호출하기 때문이다.

// CASE 6.
setCount((prev) => prev); // 이는 리렌더링이 일어나지 않는다.
```

지연 초기화는 아래와 같이 할수 있다.

```jsx
const init = () => 0;

/**
 * init 함수는 useState를 호출하기 전에 실행되지 않는다.
 * 이는 컴포넌트가 마운트 될 때 한 번만 호출함을 뜻한다.
 **/
const [count, setCount] = useState(init);
setCount((prev) => prev + 1);
```

## useReducer 이해하기

```jsx
// CASE 1.
// reducer 함수를 통해 상태 변화를 더욱 세밀하게 제어할수 있다.
const reducer = (state, action) => {
  switch (action.type) {
    case "INCREMENT":
      return { ...state, count: state.count + 1 };
    case "SET_TEXT":
      if(!action.text) {
        // bailout. 리렌더링이 일어나지 않음
        return state;
      }
      return { ...state, text: action.text };
    default:
      throw new Error("unknown action type");
  }
};

// reducer는 순수함수여야한다.
const [state, setState] = useReducer(reducer, { count: 0, text: "hi" });

<div>{state.count}</div>
<button onClick={() => dispatch({ type: 'INCREMENT'})}>Increment</button>
dispatch({type: 'SET_TEXT', text: 'hello'});

// CASE 2.
// init 함수는 mount 될때 한 번만 실행된다.
const init = () => ({count, text: 'hi'});
// 3번째 파라미터 값은 지연 초기화를 위해 존재한다.
const [state, setState] = useReducer(reducer, 0, init);
```

## useState vs useReducer

useState를 useReducer로 구현하는것은 100% 가능하다. 실제로 알려진 바에 의하면 리액트에서 useState는 useReducer로 구현되어 있다.

```jsx
const reducer = (prev, action) => {
  return typeof action === "funcion" ? action(prev) : action;
};
const useState = (initialState) => {
  const [state, dispatch] = useReducer(reducer, initialState);
  return [state, dispatch];
};
```

### useReducer를 useState로 구현하기

모든 `useReducer`를 `useState`로 변경하는 것은 가능할까? 이는 거의 가능하다. "거의"라고 표현한 것은 미묘한 차이가 있기 때문이다.

대부분의 상황에서 우리는 useReducer가 useState보다 flexible 할 것이라고 기대한다. 아래에서 useState가 얼마나 유연할 수 있는지 확인해보자.

```jsx
const useReducer = (reducer, initialArg, init) => {
  const [state, setState] = useState(
    init ? () => init(initialArg) : initialArg
  );
  const dispatch = useCallback(
    (action) => setState((prev) => reducer(prev, action)),
    [reducer]
  );
  return [state, dispatch];
};
```

useReducer의 기본 기능 및 lazyInit 까지 useState를 통해 구현할 수 있다. 하지만 앞에서 "거의"라고 말한 미묘한 부분은 명확히 다른 부분이기 때문에, 아래에서 자세하게 다뤄보자.

#### init 함수 사용하기

하나의 차이점은 reducer 및 init을 훅이나 컴포넌트 밖에서 정의할수 있다는 것이다. 이는 useReducer에서만 가능하며 useState에서는 불가능하다.

```jsx
const init = (count) => ({ count });
const reducer = (prev, delta) => prev + delta;

const ComponentWithUseReducer = ({ initialCount }) => {
  const [state, dispatch] = useReducer(reducer, initialCount, init);
  return (
    <div>
      {state}
      <button onClick={() => dispatch(1)}>+1</button>
    </div>
  );
};

const ComponentWithUseState = ({ initialCount }) => {
  const [state, setState] = useState(() => init(initialCount));
  const dispatch = (delta) => setState((prev) => reducer(prev, delta));
  return [state, dispatch];
};
```

`ComponentWithUseState` 컴포넌트는 인라인 함수가 있는 반면 `ComponentWithUseReducer`는 존재하지 않는 것을 확인할 수 있다.

이는 사소한 차이일수 있으나, 컴파일러나 인터프리터 관점에서 본다면 인라인 함수가 없는 것이 최적화에 더 큰 장점이 있다.

#### 인라인 reducer 사용하기

인라인 reducer 함수는 외부의 변수에 종속을 가질수 있다. 이는 useState에서는 불가능하며 오직 useReducer에서만 가능하다. **이것이 useReducer의 특별한 기능**이다.

그러므로 아래와 같은 코드는 기술적으로 가능하다.

```jsx
// Works
const useScore = (bonus) =>
  useReducer((prev, delta) => prev + delta + bonus, 0);

// Not work
const useScore = (bonus) => {
  const [state, setState] = useState(0);
  const dispatch = (delta) => setState((prev) => prev + delta + bonus);
  return [state, dispatch];
};

// Example
const MyComponent = ({ bonus, setBonus }) => {
  const [state, dispatch] = useScore(bonus);
  const handleClick = () => {
    dispatch(1);
    setBonus((prev) => prev + 1); // useReducer는 props의 bonus를 계속 갱신하지만 useState는 최초 마운트시 한번 만 적용한다.
  };
};
```

delta 및 bonus의 값이 모두 변해도 위의 useReducer는 정상적으로 동작하지만 useState로 해당 구현을 하게 될 경우 정상적으로 동작하지 않는다.

**useState는 이전 렌더 값(bonus)을 바라보고 있으므로 정상적인 출력이 되지 않는다. 반면 useReducer는 렌더 단계에서 reducer를 호출하므로 현재 렌더 값으로 정상적인 계산을 할 수 있다.**

> useState는 init값을 갱신하지 않는다. 최초 마운트 시에만 적용한다
