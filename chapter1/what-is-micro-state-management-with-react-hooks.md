# 리액트 훅을 사용한 마이크로 상태 관리란? (p.3 - 4)

- 기존(리액트 훅이 등장하기 전)에는 단일하고(monolithic) 보편적인(general) 방식만 있어서 목적에 맞게 사용하기 어려웠다.
- 리액트 훅이 등장하고 스테이트 관리가 '경량화'되고 '마이크로'해졌다.
- 스테이트 관리에서 우리의 중점은 글로벌 스테이트다.
  - 글로벌 스테이트란, 다수의 컴포넌트가 하나의 스테이트를 공유하는 것을 말한다.
  - 글로벌 스테이트는 리액트에서 직접적으로 제공할 수 없기 때문에 어려운 주제다.
  - 글로벌 스테이트는 리액트가 아닌, 개발자 커뮤니티나 생태계에서 대신 감당하고 있다.
- 마이크로 스테이트 관리를 위한 라이브러리들 중 이 책에서는 Zustand, Jotai, Valtio와 React Tracked에 대해 이야기해볼 것이다.

---

## 마이크로 상태 관리 이해하기 (p.5 - 6)

- 스테이트(상태)는 유저 인터페이스를 반영하는 데이터를 말한다.
- 개발자 경험을 위해 단일한 스테이트가 여러 목적을 위해 사용되었으나, 단일 스테이트 라이브러리들은 '과잉 대응'인 경우가 있었다.
- 훅을 사용하면서 새로운 방법으로 스테이트를 생성할 수 있게 되었고, 이 방식은 개발자가 필요로 하는 특수한 목적에 따른 해결책들을 만들어낼 수 있게 했다.
- 단일 스테이트가 적절하지 않은 상황(리액트 훅이 해결하려는 상황) 예시:

  - 폼 스테이트: 폼을 작성하거나 제출할 때마다 변경될 수 있는 폼 컴포넌트의 내부 상태인 폼 스테이트와 단일 전역 스테이트는 서로 다른 목적을 가지고 있기 때문에 구분하여 다루는 것이 중요하다.
  - 서버 캐시 스테이트: 서버에서 가져온 데이터를 캐싱하여 클라이언트에서 재사용하는 스테이트다. 서버와의 통신을 통해 동적으로 변경될 수 있으며, 이를 다시 가져오는 리페칭 과정 같은 독특한 특징을 가지기 때문에 단일 스테이트와 구분하여 다루어야 한다.
  - 내비게이션 스테이트: 내비게이션 스테이트는 보통 브라우저에서 처리되고, 브라우저 url과 함께 관리되고, 브라우저 히스토리에 저장된다. Node.js와 같이 브라우저 이외의 서버사이드 환경에서도 상태를 관리해야 할 때가 있기 때문에 전역으로 관리하지 않는 것이 좋다.

- 단일 상태 관리를 어느 정도 비중으로 사용할 것인지는 앱의 성격에 따라 다르다. 따라서 단일 상태 관리에 대한 해결책은 경량화되고, 요구사항에 맞게 고를 수 있어야 한다. 이 경량화되고, 목적 지향적인 해결책을 '마이크로 상태 관리'라 부른다.
- 마이크로 상태 관리는 기본적으로 스테이트를 읽고, 업데이트하고, 스테이트로 렌더할 수 있어야 한다.
- 마이크로 상태 관리는 추가적으로 리렌더 최적화, 다른 시스템과의 상호작용, 비동기 지원, 파생 스테이트, 간단한 문법 등이 요구된다.
- 그러나 위의 요구사항들이 전부 필요하지도 않을 뿐더러, 상충하는 특징들도 있기 때문에 마이크로 상태 관리 솔루션은 한 가지가 아니다.
- 마이크로 상태 관리의 낮은 학습난도 또한 개발자 경험과 생산성에 중요한 영향을 미친다.

🤔 `단일 상태 관리 라이브러리`는 전역 상태를 집중화된 저장소에서 관리하는 방식을 사용한다. 애플리케이션의 전역 상태를 하나의 객체로 관리하는 것을 지향하며, 이를 단일 상태(Single-state)라고 한다. 이를 통해 전역 상태를 일관성 있게 처리하고, 관리하기 용이하게 만들 수 있다.

🤔 `개발자 경험(DX)`은 소프트웨어 개발자들이 소프트웨어 애플리케이션을 구축하기 위해 도구, 기술 및 플랫폼을 사용하는 전반적인 경험을 나타내는 용어다. DX는 개발 환경 설정의 쉬움, 문서의 가용성 및 개발자 지원의 질, 사용자 인터페이스의 직관성 및 개발 프레임워크의 유연성 등 모든 것을 포함한다. 이는 코드 작성 및 배포 과정을 가능한 한 원활하고 효율적으로 만드는 것을 목표로 하며, 이는 개발자 생산성, 만족도 및 유지 보수성을 향상시킬 수 있다.

## 훅 사용하기 (p.7 - 10)

- 리액트 훅은 마이크로 상태 관리에 있어 필수적이다.
- 상태 관리 솔루션을 구현하기 위한 리액트 원시 훅은 다음과 같다:

  - useState: 로컬 스테이트를 만드는 기본적인 훅으로, useState를 기반으로 다양한 커스텀 훅을 만들 수 있다.
  - useReducer: 또한 로컬 스테이트를 만드는 훅으로, useState 대체재로 사용되기도 한다.
  - useEffect: 리액트 렌더 프로세스의 밖에서 로직이 작동될 수 있게 한다. 전역 스테이트를 위한 상태 관리 라이브러리를 개발하는 데 특히 중요한데, 그 이유는 리액트 컴포넌트의 생애와 관련된 기능을 구현할 수 있게 해주기 때문이다.

- 리액트 훅은 UI 컴포넌트에서 로직을 분리할 수 있게 해준다. 다음 예시에서,
  - Component는 useCount의 구현에 의존하지 않는다.
    - 컴포넌트를 훼손하지 않으면서, useCount 내에 로직을 추가할 수 있다.
  - 또한 단순히 useState를 사용하는 것보다 명시적인 이름을 갖는다. 가독성을 위해 매우 중요한 포인트다.

```jsx
const useCount = () => {
  const [count, setCount] = useState(0);
  return [count, setCount];
};

const Component = () => {
  const [count, setCount] = useCount();
  return (
    <div>
      {count}
      <button onClick={() => setCount((c) => c + 1)}>+1</button>
    </div>
  );
};
```

- 커스텀 훅으로 로직을 분리함으로써 얻는 또 하나의 장점은, 컴포넌트를 변경하지 않고도 새로운 기능(예를 들면 디버깅 메시지를 보여주는 것)을 추가할 수 있다는 것이다.

```jsx
const useCount = () => {
  const [count, setCount] = useState(0);
  useEffect(() => {
    console.log("count is changed to", count);
  }, [count]);
  return [count, setCount];
};
```

### 서스펜스와 동시성 렌더링

- 리액트 훅은 서스펜스와 동시성 렌더링과 같은 메커니즘과 함께 작동하도록 디자인 되어 있다.

  - 데이터 페칭을 위한 서스펜스: 비동기에 대한 걱정 없이 컴포넌트를 개발할 수 있도록 한다.
  - 동시성 렌더링: CPU가 오랫동안 블로킹되는 걸 막기 위해 렌더 프로세스를 덩어리 단위로 쪼개는 매커니즘을 말한다.

- 그러나 이러한 리액트 훅을 오용해서는 안 된다.
  - 리렌더를 일으키지 않거나, 너무 많은 리렌더를 일으키거나, 부분적인 리렌더만 일으키는 등의 예상치 못한 행동으로 이어질 수 있기 때문에, 이미 존재하는 스테이트 객체나 ref 객체를 직접 조작해서는 안 된다.
  - 훅 함수나 컴포넌트 함수는 여러 번 호출될 수 있기 때문에, 여러 번 호출되더라도 일관성 있게 작동할 수 있도록 순수성을 유지해야 한다.

🤔 `함수의 순수성`이란, 함수가 동일한 입력에 대해 항상 동일한 출력을 반환하며, 함수 외부의 상태를 변경하지 않는 성질을 말한다. 함수가 순수 함수가 아닌 경우는, 예를 들면 함수가 외부의 상태를 변경하거나, 난수 또는 현재 시간 같이 입력값 외에 다른 상태를 참조해서 동일한 입력에 대해 항상 동일한 출력을 보장할 수 없는 것을 말한다. 함수가 순수한 경우, 해당 함수를 테스트하기 쉽고, 예측하기 쉽기 때문에, 컴포넌트를 개발하고 유지보수하기가 쉬워진다.

---

## 전역 스테이트 탐구하기 (p.10 - 12)

- 전역 스테이트는 다수의 컴포넌트에서 사용되는 스테이트를 말한다.
- 전역 스테이트는 싱글톤(애플리케이션에서 전역적으로 사용되는 유일한 인스턴스)일 필요는 없다.
- 리액트에서 전역 스테이트를 구현하는 것이 사소한 문제가 아닌 이유는, 리액트는 컴포넌트 모델을 기반으로 하기 때문이다.
  - 컴포넌트 모델에서는 지역성이 매우 중요하다.
  - 컴포넌트는 '자가 포함적(self-contained)'이다.
  - 즉, 컴포넌트는 독립적인 기능을 가지며, 자체적으로 필요한 데이터와 스테이트를 가지고 있어 다른 컴포넌트와 분리하여 (재)사용될 수 있다.
  - 컴포넌트는 입력값(props)과 스테이트에 의해서만 출력을 결정하므로, 동일한 입력값과 상태값이 주어지면 항상 동일한 출력을 보장한다.
  - 따라서 엄밀히 따지면, 컴포넌트는 전역 스테이트에 의존해서는 안 된다.

---

## useState 사용하기 (p.12 - 16)

### 스테이트 값을 값으로 업데이트하기

- `Bailout`(퍼내다, 구제하다, 탈출하다)이란, 리렌더를 유발하지 않도록 하는 것을 의미한다.
- 다음의 예시에서, 버튼을 아무리 클릭해도 count 스테이트는 계속 1을 유지하기 때문에 리렌더가 발생하지 않는다.

```jsx
const Component = () => {
  const [count, setCount] = useState(0);
  return (
    <div>
      {count}
      <button onClick={() => setCount(1)}>Set Count to 1</button>
    </div>
  );
};
```

- 그런데 다음과 같이 스테이트가 원시값이 아닌 객체라면, state.count가 매번 1이라고 해도, 버튼 클릭 시마다 새로운 객체를 생성하기 때문에 매번 리렌더가 발생한다.

```jsx
const Component = () => {
  const [state, setState] = useState({ count: 0 });
  return (
    <div>
      {state.count}
      <button onClick={() => setState({ count: 1 })}>Set Count to 1</button>
    </div>
  );
};
```

- 다음과 같이 스테이트 객체가 참조적으로 변경되지 않은 경우라면, bailout이 일어나고 리렌더를 발생시키지 않는다.

```jsx
const Component = () => {
  const [state, setState] = useState({ count: 0 });
  return (
    <div>
      {state.count}
      <button
        onClick={() => {
          state.count = 1;
          setState(state);
        }}
      >
        Set Count to 1
      </button>
    </div>
  );
};
```

- 다음과 같은 경우, 버튼을 빠르게 두 번 클릭하는 경우 2가 증가하는 게 아니라 1만 증가한다.

```jsx
const Component = () => {
  const [count, setCount] = useState(0);
  return (
    <div>
      {count}
      <button onClick={() => setCount(count + 1)}>
        Set Count to {count + 1}
      </button>
    </div>
  );
};
```

### 스테이트 값을 함수로 업데이트하기

- 위의 예시와 비슷하게 보이지만, 다음과 같이 값이 아니라 함수로 스테이트 값을 업데이트하는 경우에는 빠르게 두 번 클릭하면 2가 증가하게 된다.

```jsx
const Component = () => {
  const [count, setCount] = useState(0);
  return (
    <div>
      {count}
      <button onClick={() => setCount((c) => c + 1)}>Increment Count</button>
    </div>
  );
};
```

- 두 예시가 다르게 작동하는 이유는 업데이트가 어떤 값을 기초로 하는지가 다르기 때문이다.
- `(c) => c + 1`은 순차적으로 이전 값을 기초로 업데이트되지만,
- `count + 1`은 화면에 나타나는 값을 기초로 업데이트 된다.

- 다음과 같이 업데이트 함수가 이전 스테이트와 같은 스테이트를 반환한다면 bailout하게 되고, 이 컴포넌트는 리렌더가 발생하지 않을 것이다.

```jsx
const Component = () => {
  const [count, setCount] = useState(0);
  useEffect(() => {
    const id = setInterval(() => setCount((c) => c + 1), 1000);
    return () => clearInterval(id);
  }, []);
  return (
    <div>
      {count}
      <button onClick={() => setCount((c) => (c % 2 === 0 ? c : c + 1))}>
        Increment Count if it makes the result even
      </button>
    </div>
  );
};
```

### 지연 초기화(lazy initialization)

- useState는 다음과 같이 오직 첫 렌더에만 계산되는 초기화 함수를 가질 수 있다.

```jsx
const init = () => 0;

const Component = () => {
  const [count, setCount] = useState(init);
  return (
    <div>
      {count}
      <button onClick={() => setCount((c) => c + 1)}>Increment Count</button>
    </div>
  );
};
```

- init 함수는 매우 무거운 함수를 포함할 수 있고, 이 함수는 초기 스테이트를 호출하기 위해서만 사용된다는 점이 중요하다.
- init 함수는 useState를 호출하기 전이 아니라 '게으르게, 지연되어' 계산된다. 다시 말해, 마운트 될 때만 딱 한 번 호출된다.

---

## useReducer 사용하기 (p.16 - 20)

### 일반적인 사용 예시

- 리듀서는 복잡한 스테이트를 다룰 때 좋다.
- 미리 리듀서 함수를 정의하고 초기 스테이트를 매개변수로 받을 수 있다.
- 훅의 외부에 리듀서 함수를 정의하는 것의 이점은, 코드의 분리 및 테스트 가능성 때문이다.
- 리듀서 함수는 순수함수라서 그것의 행동을 테스트해보기가 쉽다.

### Bailout

- useState와 마찬가지로, 이전 스테이트와 달라진 게 없다면 useReducer도 조기종료(bailout)한다.

```jsx
const reducer = (state, action) => {
  switch (action.type) {
    case "INCREMENT":
      return { ...state, count: state.count + 1 };
    case "SET_TEXT":
      if (!action.text) {
        // bail out
        return state;
      }
      return { ...state, text: action.text };
    default:
      throw new Error("unknown action type");
  }
};
```

- 그런데 만약 `action.text`가 없을 때 `state` 대신 `{ ...state, text: action.text || state.text }`를 반환한다면, 이것은 새로운 객체를 생성하는 것이기 때문에 조기종료하지 않는다.

### 원시값

- action은 객체일 필요가 없다.
- 다음 예시에서 스테이트 값은 원시값인 숫자이지만, 조건문이 추가되어 로직이 약간 더 복잡하다.

```jsx
const reducer = (count, delta) => {
  if (delta < 0) {
    throw new Error("delta cannot be negative");
  }
  if (delta > 10) {
    // too big, just ignore
    return count;
  }
  if (count < 100) {
    // add bonus
    return count + delta + 10;
  }
  return count + delta;
};
```

### 지연 초기화(lazy initialization)

- useReducer는 두 개의 매개변수를 필요로 한다.

  - 1. 리듀서 함수
  - 2. 초기 스테이트
  - 3. init (선택적)
    - init 함수는 마운트 될 때만 한 번 호출되기 때문에, 무거운 계산을 포함할 수 있다.
    - useState와 달리 useReducer의 두 번째 인자인 초기인자(initailArg)를 갖는다. (아래의 예시에서는 0)

  ```jsx
  const Component = () => {
    const [state, dispatch] = useReducer(reducer, 0, init);
    return (
      <div>
        {state.count}
        <button onClick={() => dispatch({ type: "INCREMENT" })}>
          Increment count
        </button>
        <input
          value={state.text}
          onChange={(e) => dispatch({ type: "SET_TEXT", text: e.target.value })}
        />
      </div>
    );
  };
  ```

---

## useState와 useReducer의 유사점과 차이점 (p.20 - 24)

### useReducer로 useState 구현하기

- useReducer로 useState를 구현하는 것은 언제나, 100% 가능하다.
- 사실, 리액트 내부에서 useState는 useReducer로 구현되어 있다.

```jsx
const useState = (initialState) => {
  const [state, dispatch] = useReducer(
    (prev, action) => (typeof action === "function" ? action(prev) : action),
    initialState
  );
  return [state, dispatch];
};
```

```jsx
const reducer = (prev, action) =>
  typeof action === "function" ? action(prev) : action;

const useState = (initialState) => useReducer(reducer, initialState);
```

### useState로 useReducer 구현하기

- useReducer의 예시들을 useState로 대체할 수 있다는 것은 '거의' 사실이다.

```jsx
const useReducer = (reducer, initialState) => {
  const [state, setState] = useState(initialState);
  const dispatch = (action) => setState((prev) => reducer(prev, action));
  return [state, dispatch];
};
```

- 만약 init 함수와 지연 초기화를 구현하고 싶다면 useCallback을 사용해서 안정적인 dispatch 함수를 갖게 될 수 있다.

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

- 위의 구현은 거의 완벽하게 useState로 useReducer를 대체할 수 있다.
- 하지만, 다음의 'init 함수'와 '인라인 리듀서' 사용과 관련해서 미묘한 차이가 있다.

### init 함수 사용하기

- useReducer를 사용하면 reducer와 init 함수를 훅이나 컴포넌트 밖에 정의할 수 있지만, useState를 사용하면 불가능하다.
- 즉, useState를 사용하면 인라인 함수(inline function, 함수를 컴포넌트의 JSX 코드 내부에서 정의하는 것)를 필요로 한다.
- 사소한 것이긴 하지만, 몇몇 인터프리터와 컴파일러는 인라인 함수가 없을 때 최적화를 더 잘한다.

### 인라인 리듀서

- 인라인 리듀서 함수는 외부 함수에 의존할 수 있다.
- 이 점은 useState는 갖지 못하는 useReducer만의 특징이다. (가능하다는 것이지, 더 좋고, 필수적임을 의미하는 건 아니다.)

```jsx
const useScore = (bonus) =>
  useReducer((prev, delta) => prev + delta + bonus, 0);
```

- 위의 코드는 심지어 bonus와 delta가 둘 다 업데이트될 때에도 정확히 작동한다.

🤔 `리액트의 인라인 리듀서(inline reducer)`는 컴포넌트 내부에서 사용되는 리듀서(reducer) 함수를 의미합니다. 인라인 리듀서는 컴포넌트 내부에서 스토어(store)의 데이터를 수정하고 업데이트할 때 사용된다. 일반적으로, 리듀서 함수는 별도의 파일로 분리하여 작성하고, 액션(action)과 함께 리듀서 함수를 호출하여 스토어 데이터를 업데이트한다. 그러나 인라인 리듀서는 컴포넌트 내부에서 직접 작성되는 리듀서 함수로, 컴포넌트의 상태(state)를 업데이트하고, 이벤트 처리 등과 같은 로직을 수행하는데 사용된다. 인라인 리듀서는 간단한 애플리케이션에서 사용할 수 있지만, 큰 규모의 애플리케이션에서는 별도의 파일로 분리된 리듀서 함수를 사용하는 것이 일반적이다.
