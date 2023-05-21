# 컨텍스트를 사용하여 컴포넌트간 상태 공유하기

리액트는 16.3 버전부터 context를 제공하고 있다. 컨텍스트는 상태와 아무런 관련이 없지만 상태를 prop 전달이 아닌 방식으로 사용하는 메커니즘으로써 알아두어야 한다.

컨텍스트와 컴포넌트 상태를 조합하여 우리는 전역 상태를 제공할수 있게 된다.

리액트 16.8 버전에 제공되는 `useContext` 훅 및 `useState` 훅을 통해 우리는 전역 상태를 만들수 있다.

컨텍스트는 전역 상태를 완전히 지원하기위해 디자인되지 않았다. 따라서 컨텍스트를 사용하는 하위 컴포넌트는 컨텍스트 값이 변경될 경우 모두 리렌더링이 된다는 한계가 있다. 이 때문에 우리는 컨텍스트(전역 상태)를 잘게 나누어 적절하게 사용해야 한다.

## 목차

- useContext 및 useState 탐색
- 컨텍스트 이해하기
- 전역 상태를 위한 컨텍스트 생성하기
- 컨텍스트를 사용 모범 사례

## useContext 및 useState 탐색

useContext는 컴포넌트에서 prop 드릴링을 제거할수 있게 해준다.

```jsx
const ColorContext = createContext('black');

const Component = () => {
  const color = useContext(ColorContext);
  return <div style={{ color }}>Hello {color}</div>;
};

const App = () => (
  <>
    <Component /> {/** black **/}
    <ColorContext.Provider value="red">
      <Component /> {/** red **/}
    </ColorContext.Provider>
    <ColorContext.Provider value="green">
      <Component /> {/** green **/}
    </ColorContext.Provider>
    <ColorContext.Provider value="blue">
      <Component /> {/** blue **/}
      <ColorContext.Provider value="skyblue">
        <Component /> {/** skyblue **/}
      </ColorContext.Provider>
    </ColorContext.Provider>
  </>
);
```

컨텍스트는 가장 가까운 Provider의 값을 취하며, 재사용성이 가능하다.

### useContext와 함께 useState 사용하기

```jsx
const CountStateContext = createContext({ count: 0, setCount: () => {} });

const App = () => {
  const [count, setCount] = useState(0);

  return (
    <CountStateContext.Provider value={{ count, setCount }}>
      <Parent />
    </CountStateContext.Provider>
  );
};

const Parent = () => {
  const { count, setCount } = useContext(CountStateContext);
  return <div>...</div>;
};
```

## 컨텍스트 이해하기

컨텍스트 프로바이더가 새로운 컨텍스트 값을 가지게 될 경우, **모든** 컨텍스트 컨슈머(useContext)들은 해당 값을 전파 받으며 리렌더링 된다.

### 컨텍스트 전파의 작동 방식

컨텍스트를 사용할 때 자식 컴포넌트는 부모 혹은 컨텍스트의 두 가지 이유로 렌더링 되는 경우가 있다.

컨텍스트 값이 변경되는 경우를 제외하고 리렌더링을 막기 위해선 `lift content up` 혹은 `memo`를 사용할수 있다. memo 함수는 래핑된 컴포넌트의 prop이 변경되지 않으면 리렌더링을 하지 않도록 해준다.

```jsx
const ColorContext = createContext('black');

// 컨텍스트를 사용한 컴포넌트
const ColorComponent = () => {
  const color = useContext(ColorContext);
  const renderCount = useRef(1);
  useEffect(() => {
    renderCount.current += 1;
  });

  return (
    <div style={{ color }}>
      Hello {color} (renders: {renderCount.current})
    </div>
  );
};
const MemoedColorComponent = memo(ColorComponent);

// 단순 컴포넌트
const DummyComponent = () => {
  const renderCount = useRef(1);
  useEffect(() => {
    renderCount.current += 1;
  });

  return <div>Dummy (renders: {renderCount.current})</div>;
};
const MemoedDummyComponent = memo(DummyComponent);

const Parent = () => (
  <ul>
    <li>
      <DummyComponent />
    </li>
    <li>
      <MemoedDummyComponent />
    </li>
    <li>
      <ColorComponent />
    </li>
    <li>
      <MemoedColorComponent />
    </li>
  </ul>
);

const App = () => {
  const [color, setColor] = useState('red');
  return (
    <ColorContext.Provider value={color}>
      <input value={color} onChange={(e) => setColor(e.target.value)} />
      <Parent />
    </ColorContext.Provider>
  );
};
```

위의 코드에서 렌더링 과정을 살펴보자.

1. 최초에는 모든 컴포넌트가 렌더링된다.
2. text input을 변경할 경우 `App` 컴포넌트가 리렌더링 된다(useState 때문).
3. 그 후 `ColorContext.Provider`가 새로운 값(color)을 가져오고 동시에 `Parent` 컴포넌트가 리렌더링 된다.
4. `DummyComponent`는 리렌더링 되지만(부모 컴포넌트가 리렌더링 되므로) `MemoedDummyComponent`는 리렌더링 되지 않는다(메모되었으므로).
5. `ColorComponent`는 부모 컴포넌트가 리렌더링 될 뿐만 아니라 컨텍스트 값의 변화가 있었기 때문에 두 가지 이유로 리렌더링이 된다.
6. `MemoedColorComponent`는 메모되었지만 컨텍스트 변경으로 인해 리렌더링이 일어난다.

**우리는 이를 통해 `memo`는 컨텍스트를 사용하는 내부 컴포넌트의 리렌더링을 막지 못한다는 것을 알수 있다**.

### 객체에 컨텍스트를 사용할 때의 제한 사항

컨텍스트 값으로 원시 값을 사용할 때에는 직관적이므로 문제가 되지 않지만, 객체로 사용할 경우 혼란스러울수 있다. 객체에 값을 컨텍스트 컴포넌트에서 모두 사용하지 않을수 있다.

```jsx
const CountContext = createContext({ count1: 0, count2: 0 });

const Counter1 = () => {
  const { count1 } = useContext(CountContext);
  return <div>...</div>;
};
const memoedCounter1 = memo(Counter1);

const Counter2 = () => {
  const { count2 } = useContext(CountContext);
  return <div>...</div>;
};
const memoedCounter2 = memo(Counter2);

const App = () => {
  const [count1, setCount1] = useState(0);
  const [count2, setCount2] = useState(0);
  return (
    <CountContext.Provider value={{ count1, count2 }}>
      <button onClick={() => setCount1((c) => c + 1)}>{count1}</button>
      <button onClick={() => setCount2((c) => c + 1)}>{count2}</button>
      <Counter1 />
      <Counter2 />
    </CountContext.Provider>
  );
};
```

위와 같은 예제에서 `count1` 값이 변경되어도 `Counter2` 컴포넌트가 리렌더링이 되며 역도 성립힌다. 이는 컨텍스트의 값을 쓰지 않았음에도 리렌더링이 일어나지 않아 혼란을 야기한다고 느껴진다. 하지만 엄연히 따지면 주소값이 변경되므로 "다른"값으로 인식되기 때문에, 원시 타입이 아닌 객체 참조에서는 memo가 무력화 되는 이유가 된다.

## 전역 상태를 위한 컨텍스트 생성하기

React Context의 두 가지 동작을 기반으로 전역 상태와 Context를 사용하는 방법을 설명해 보려고 한다.

1. 작은 상태 조각 만들기
2. useReducer로 하나의 상태 만들기 및 여러 컨텍스트로 전파하기

### 작은 상태 조각 만들기

첫 번째 방법은 작은 상태 조각 만들기다. 큰 결합된 객체를 사용하지 않고 각 조각에 대한 전역 상태 및 컨텍스트를 만든다.

```jsx
const App = () => (
  {/* context1 */}
  <Count1Provider>
      {/* context2 */}
    <Count2Provider>
      <Parent />
    </Count2Provider>
  </Count1Provider>
);

const Parent = () => (
  <div>
    <Counter1 />
    <Counter1 />
    <Counter2 />
  </div>
);
```

위와 같이 상태를 나누면 이전과 같은 리렌더링 고통에서 어느정도 벗어날수 있게 된다. context1에서 사용되는 값은 context2의 리렌더링에 영향을 주지 않는다. 또한 context1에서는 원시 값만 사용함으로 더더욱 문제가 되지 않는다

**객체를 한 번에 사용**(객체 내부의 프로퍼티만 변경한다거나 하지 않고;리렌더를 트리거할수 없음)하고 컨텍스트 동작에 제한을 주지 않는다면 객체를 컨텍스트 값으로 사용해도 된다.

### useReducer로 하나의 상태 만들기 및 여러 컨텍스트로 전파하기

두 번째 방법은 단일 상태를 만들고 멀티 컨텍스트에 상태 조각을 배포해서 사용하는 것이다.

```tsx
type Action = { type: 'INC1' } | { type: 'INC2' };

const Count1Context = createContext<number>(0); // 상태 조각 컨텍스트
const Count2Context = createContext<number>(0); // 상태 조각 컨텍스트
const DispatchContext = createContext<Dispatch<Action>>(() => {}); // dispatch 함수를 위한 컨텍스트
```
