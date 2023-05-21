# 지역(local) 및 전역(global) 상태(state) 사용하기

## 목차

- 지역 상태는 언제 사용해야 하는가
- 지역 상태를 효과적으로 사용하는 법
- 전역 상태를 사용하는 법

## 지역 상태는 언제 사용해야 하는가

```js
let base = 1;
const addBase = (n) => n + base;
```

전역 변수를 싱글톤(in memory)으로 관리하는 것은 추천되지 않는다. 다른 어느 곳에서 어떻게 `base` 값을 변경할지 알수 없기 때문이고 재사용성이 떨어지기 때문이다.

```js
const createContainer = () => {
  let base = 1;
  const addBase = (n) => n + base;
  const changeBase = (b) => (base = b);
  return { addBase, changeBase };
};
const { addBase, changeBase } = createContainer();
```

위의 함수는 base가 changeBase로 변경될 가능성이 있으므로 순수함수는 아니지만 base가 변하지 않는다는 가정에서는 순수하며(이 특성을 idempotent; 무능력이라 부른다), 전역 상태가 아니므로 코드의 재사용이 가능하다.

```jsx
const AddBase = ({ number }) => {
  const [base, changeBase] = useState(1);
  return <div>{number + base}</div>;
};
```

위의 createContainer를 jsx로 옮겼다고 생각해보자. useState의 base값에 의존하고 있으므로 `AddBase` 컴포넌트는 순수함수가 아니지만, changeBase 함수가 외부에 노출되지 않으므로 이 함수는 순수하다(idempotent 때문에).

### 지역 상태의 한계

우리가 지역화를 원하지 않을때 적절하지 않다. 코드의 완전한 다른 부분에서 해당 상태를 사용하고자 한다면 전역 상태가 적절하다.

전역 상태를 사용하면 컴포넌트 상태의 예측 가능성이 감소하지만 외부에서 컴포넌트를 제어할수 있다는 장점이 있다. 이는 완전한 트레이드 오프에 해당한다.

지역 상태를 기본 수단으로 사용하고 전역 상태를 고려하는것이 좋다. 그렇기 때문에 지역 상태로 얼마나 많은 케이스들을 다룰수 있는지 잘 알아두는 것이 좋다.

## 지역 상태를 효과적으로 사용하는 법

**Lift state up** 및 **Lift content up**을 사용한다.

```jsx
const AdditionalInfo = () => {
  return <p>Some information</p>;
};

const Component1 = ({ count, setCount, children }) => {
  return (
    <div>
      {count} {/** Lift state up **/}
      <button onClick={() => setCount((c) => c + 1)}>Increment Count</button>
      {children} {/** Lift content up **/}
      {/**
       * 컴포넌트가 이렇게 있을 경우 늘 리렌더링이 됨
       * <AdditionalInfo />
       **/}
    </div>
  );
};

const Parent = ({ children }) => {
  // Lift state up
  const [count, setCount] = useState(0);

  return (
    <>
      <Component1
        // Lift state up
        count={count}
        setCount={setCount}
      >
        {children} {/** Lift content up **/}
      </Component1>
      <Component2
        // Lift state up
        count={count}
        setCount={setCount}
      />
    </>
  );
};

const GrandParent = () => {
  return (
    <Parent>
      {/** Lift content up **/}
      <AdditionalInfo />
    </Parent>
  );
};
```

컴포넌트마다 상태를 가지고 있으면 동기화가 되지 않기 때문에 부모 컴포넌트에서 상태를 선언하고 자식 컴포넌트로 내려(lift state up)준다.

만약 Component1에 AdditionalInfo 컴포넌트가 존재한다면, **Component1의 값(count)이 변경될 때마다 AdditionalInfo가 리렌더링이 된다**(지금은 memo를 고려하지 않는다). 이를 Lift content up 방식을 통해 컴포넌트를 내려 피할수 있다.

## 전역 상태를 사용하는 법

### 전역 상태는 무엇인가

쉽게 정의하자면, 지역 상태가 아닌 것들이다. 지역 상태는 단일 컴포넌트에 속해있다. 따라서 **전역 상태는 다중 컴포넌트에서 사용되는 상태**를 뜻한다.

그러나 모든 컴포넌트에서 사용되는 지역 상태일 경우(lift state up) 이는 전역 상태인가? 따라서 지역/전역 상태는 구분하기가 모호하다.

지역/전역 상태의 두 가지 측면을 뽑아 보면서 이 두 상태의 구분하는 법을 배워나갈수 있다.

1. 싱글톤
   - 일부 컨텍스트에서 상태가 하나의 값을 갖는다는 것을 의미한다.
2. 상태를 공유한다
   - 다른 컴포넌트 사이에서 값을 공유함을 의미한다.

상태가 싱글톤이 아닌 경우 다양한 컴포넌트에서 상태를 사용할 때 서로 다른 값을 렌더링 할수 있는 문제가 있다.

### 전역 상태는 언제 사용해야 하는가

전역 상태를 선택하기 이전에 아래의 두 가지 가이드를 확인해보자.

1. prop으로 값을 전달하는 것은 추천하지 않는다
2. React 외부에 이미 상태가 있을 때에는 적절하다

#### Prop으로 값을 전달하는 것은 추천하지 않는다

먼 거리의 컴포넌트간 상태가 필요할 때 prop으로 전달하는 것은 적절하지 않다. 이 경우 각 컴포넌트 사이의 모든 경로의 컴포넌트에서 prop 드릴링이 발생하기 때문에 개발자 경험에 매우 좋지 않다. 게다가 **전달하는 상태가 변경될 경우 중간의 컴포넌트들이 모두 리렌더링이 일어나게 되므**로 퍼포먼스에도 좋지 않다.

#### React 외부에 이미 상태가 있을 때에는 적절하다

```jsx
const globalState = {
  authInfo: { name: "React" },
};

const Component1 = () => {
  const { authInfo } = globalState();
  return <div>{authInfo.name}</div>;
};
```

위의 코드에서 우리는 전역 상태가 지역 상태가 될수 없는 상태라는 것을 알 수 있다.
