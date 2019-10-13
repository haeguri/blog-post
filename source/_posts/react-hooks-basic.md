---
title: React Hooks 살펴보기
date: 2019-10-13 20:08:28
tags:
  - React
  - Hooks
  - React Hooks
---

> Note: Hooks를 공부하기 위해서 공식 문서를 읽으면서 개인적으로 정리한 내용입니다. 정리하는 과정에서 내용이 생략되거나 추가된 내용이 있을 수 있습니다.

리액트 Hooks는 16.8.0에서 새로 도입된 기능입니다. 기존에는 함수형 컴포넌트에서 상태를 관리하기 위해서는 클래스 컴포넌트 다시 작성해야 했던 반면에, Hooks가 나오면서 함수형 컴포넌트에서도 상태를 가질 수 있게 되었습니다. Hooks는 이것 외에도 다양한 기능들을 제공합니다.

Hooks를 프로젝트에 도입하기 위해서, Hooks로 작성된 컴포넌트가 기존의 컴포넌트와 호환되는지 여부에 대해 검토해야 할 필요도 있습니다. 이 부분에 대해 공식 문서를 보면 Hooks를 도입했을 때의 기존 코드와 호환이 깨지는 이른바 Breaking Change는 없다고 합니다. 일부 컴포넌트만 Hooks로 전환하는 것도 가능하며 모든 컴포넌트를 Hooks로 변경할 필요는 없습니다. 또 클래스 컴포넌트가 리액트에서 제거될 계획도 없습니다.

그리고 Hooks를 도입한다고 해서 이전에 배웠던 리액트에 대한 개념들이 필요 없어지거나 하지는 않습니다. Hooks는 우리가 이미 알고 있던 props, state, context, refs, lifecycle 등에 대해서 더 직접적인 API를 제공해줄 뿐입니다.

## 왜 필요한가?

Hooks로 개발된 컴포넌트를 보면서 Hooks를 사용하면 좋은 이유에 대해서 궁금했었습니다. 단순히 *"함수형 컴포넌트에서 상태를 관리할 수 있다는 점 때문에 리액트 팀에서 Hooks를 개발했을까?"*라는 의문이 들었습니다. 공식 문서에서는 Hooks를 개발하게 된 동기에 대해서 다음과 같이 소개합니다.

### 1. 상태에 의존하는 로직을 재사용하기 어렵다.

그동안 리액트에서는 API 레벨에서 컴포넌트에 재사용 가능한 로직을 추가하는 방법을 제공하지 않고 있었습니다. 그래서 개발자들은 render props나 higher-order components와 같은 기법으로 컴포넌트에 재사용 가능한 로직을 추가했습니다. 하지만 이 패턴들을 사용했을 때 컴포넌트 계층을 다시 구성해야 했고, 어떤 사람들에게는 코드를 이해하기 힘들게 만들기도 했습니다. 그리고 이 패턴들을 사용하면 결국 `Wrapper`가 추가되었기 때문에 리액트 개발자 도구로 컴포넌트 구조를 확인해 보면 "Wrapper Hell"이 펼쳐지곤 했습니다.

Hooks를 사용하면 상태를 가진 로직을 추출해서 독립적으로 테스트할 수 있고, 재사용할 수도 있습니다. 그리고 상태를 가진 로직을 재사용하기 위해서 컴포넌트 계층을 굳이 변경하지 않아도 됩니다.

### 2. 컴포넌트가 복잡해진다.

클래스 컴포넌트에 lifecycle 메서드를 사용하면 컴포넌트가 복잡해질 수 있습니다.

예를 들어서 Friend라는 컴포넌트가 있고, 이 컴포넌트에서는 친구가 서비스에 접속했는지 여부를 실시간으로 구독(**subscribe**)하고, 친구의 데이터를 가져온다고(**fetch**) 가정해보겠습니다. 그리고 Friend 컴포넌트가 제거될 때는 친구의 접속 여부를 구독 해제(**unsubscribe**)해야 합니다. 기존의 클래스 컴포넌트에서는 다음과 같이 작성해야 했습니다.

```jsx
class Friend extends React.Component {
  // ...
  componentDidMount() {
    ChatAPI.subscribeToFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
    ChatApi.fetchFriend();
  }

  componentWillUnmount() {
    ChatAPI.unsubscribeFromFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }
  // ...
}
```

위의 `componentDidMount`에는 친구의 상태를 구독하는 로직과 친구의 데이터를 가져오는 로직이 함께 있습니다. 어떻게 보면 관련이 없는 두 개의 로직이 하나의 메서드에 있습니다. 나중에 사이드 이펙트를 일으키는 다른 로직이 추가된다면 `componentDidMount`는 더 복잡해질 수 있습니다.

그리고 친구의 상태를 subscribe하면 언젠가는 반드시 unsubscribe하게 되므로 subscribe와 unsubscribe는 연관된 로직입니다. 그런데 클래스 컴포넌트에서는 두 개의 연관된 로직을 다른 메서드에 쪼개서 작성할 수밖에 없었습니다.

Hooks는 이것을 해결하기 위해서 하나의 컴포넌트를 여러 개의 작은 함수로 나누는데, 이때 lifecycle 메서드 단위로 나누는 것이 아니라 연관된 것들에 기반하여 나누도록 합니다.

### 3. 클래스는 사람, 컴퓨터 모두 이해하기 어렵다.

리액트 팀은 자바스크립트의 클래스가 리액트를 배우는데 장애물이 될 수 있겠다는 사실을 깨달았습니다. 자바스크립트에서 **this**가 동작하는 방법은 대부분의 다른 언어에서 this가 동작하는 방법과는 다릅니다. 그래서 매번 메서드를 `bind`하거나 화살표 함수를 클래스의 프로퍼티로 추가해야만 했습니다.

리액트 팀은 **Prepack**을 사용한 **컴포넌트 폴딩**을 실험해왔고, 의미있는 결과를 확인했습니다. 그러나 클래스 컴포넌트가 의도치 않은 패턴을 조장할 수 있고, 그것이 이러한 최적화를 방해할 수 있다는 사실을 알았습니다. 클래스는 최근의 도구들에서 이슈가 있는데, 예로 클래스는 코드 압축(minify)이 잘 안됩니다. 그리고 클래스 컴포넌트들은 hot reloading을 신뢰할 수 없게 만듭니다. 그래서 우리는 최적화 가능한 코드를 유지하기 위한 API를 도입하고 싶었습니다.

이러한 문제들을 해결하기 위해서 Hooks는 클래스 없이 리액트의 기능들을 사용할 수 있게 해줍니다. 개념적으로는 리액트 컴포넌트는 거의 함수에 가까웠습니다. Hooks는 함수를 받아들이지만 리액트의 핵심적인 정신을 희생시키지는 않습니다. Hooks는 복잡한 함수형, 반응형 프로그래밍 기술을 배우는 것을 필요로하지 않고, 긴요한 해결책을 위한 접근법을 제공합니다.

## Hooks API

공식 문서에는 10개의 Hooks API를 소개하고 있지만 이 글에서는 7개의 API `useState`, `useEffect`, `useContext`, `useReducer`, `useCallback`, `useMemo`, `useRef`에 대해서만 정리하였습니다.

### 1. useState

`useState`는 함수형 컴포넌트에서 state를 가질 수 있게 하는 Hook입니다.

```jsx
const [state, setState] = useState(initialState);
```

useState는 두 개의 요소가 담긴 배열을 반환합니다. 첫 번째 요소는 컴포넌트의 **현재 상태**이고 두 번째 요소는 **상태를 설정할 수 있는 함수**입니다. 위의 `const [ state, setState ]`에서 `[ … ]`으로 변수들을 감싸는 문법은 Array Destructuring이라는 ES6 문법입니다. useState가 배열을 반환하기 때문에 Array Destructuring으로 배열의 요소들을 지역 변수로 바로 할당할 수 있습니다.

> 리액트는 컴포넌트가 다시 렌더링되도 `setState`가 동일한 참조값을 가지는 것을 보장합니다. 따라서 뒤에서 설명할 useEffect나 useCallback으로 넘겨주는 디펜던시 목록에 `setState`를 넘겨줄 필요는 없습니다.

그리고 useState는 하나의 인자를 받을 수 있습니다. 이 인자는 **상태의 초기값**이며, 초기값이기 때문에 첫 번째 렌더링 이후에서는 인자가 무시됩니다.

만약 새로운 상태가 이전의 상태에 기반하여 계산되어야 하면, 클래스 컴포넌트에서의 `this.setState`처럼 `setState`에 함수를 넘겨줄 수 있습니다.

<iframe src="https://codesandbox.io/embed/usestate-ex-1-3zxv5?fontsize=13&view=split" title="useState-ex-1" allow="geolocation; microphone; camera; midi; vr; accelerometer; gyroscope; payment; ambient-light-sensor; encrypted-media; usb" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>

useState의 인자인 **초기값**은 컴포넌트가 처음 렌더링될 때만 사용됩니다. 그런데 복잡한 계산 로직의 결과값이 초기값으로 사용된다면 매번 렌더링될 때마다 계산 로직이 실행되어 연산이 낭비되는 결과를 가져올 수 있습니다. 이때 useState에 함수를 넘기면 그 함수에서 반환하는 값이 초기값으로 사용되고, 함수는 첫 번째 렌더링에서만 실행됩니다.

```jsx
// 컴포넌트가 렌더링될 때마다 `someExpensiveComputation`이 실행된다.
const [state, setState] = useState(someExpensiveComputation(props));

// 컴포넌트가 처음 렌더링 될 때만 실행된다.
const [state, setState] = useState(() => {
  const initialState = someExpensiveComputation(props);
  return initialState;
});
```

### 2. useEffect

`useEffect`는 **사이드 이펙트**를 일으킬 수 있는 함수를 전달 받습니다.

```jsx
useEffect(() => {
 // side effect logic.
);
```

외부의 데이터를 변경한다거나, 특정 데이터를 구독한다거나, 타이머 설정, 로깅 등의 동작을 가리켜서 사이드 이펙트라고 하는데, 이 사이드 이펙트는 함수형 컴포넌트의 body에 작성해서는 안 됩니다. 함수에 본문에 작성된다면 컴포넌트가 렌더링될 때마다 실행되기 때문에 UI에 버그를 일으킬 수 있습니다. 그래서 함수형 컴포넌트에서는 useEffect로 사이드 이펙트를 일으키는 타이밍을 제어합니다.

useEffect는 첫 번째 인자로 사이드 이펙트를 일으킬 함수를 받고, 두 번째 인자로 디펜던시 목록을 받습니다. 만약 디펜던시 목록을 넘겨주지 않으면, 컴포넌트가 화면에 렌더링될 때마다 이펙트가 실행되어 버립니다. 그래서 `useEffect`에 두 번째 인자로 디펜던시 목록을 넘겨주는 것이 좋고, 이렇게 하면 특정 값이 변할 때만 이펙트가 실행되게 할 수 있습니다.

#### 이펙트의 뒤처리(cleaning up)

어떤 사이드 이펙트들은 **뒤처리(cleaning up)**가 필요한 경우가 있습니다. 예를 들어 데이터를 구독(subscribe)했다면 구독을 해제(unsubsribe)해야 하고, 타이머를 실행했다면 `clearTimeout`로 타이머를 제거해줘야 합니다. 이펙트에 필요한 뒤처리 작업은 useEffect로 전달하는 이펙트에서 **또 다른 함수를 반환**해서 지정할 수 있습니다. 뒤처리 작업은 컴포넌트에서 메모리 누수를 방지하고 다음 이펙트가 실행되기 전 이전의 이펙트를 정리해주는 역할을 합니다.

<iframe src="https://codesandbox.io/embed/useeffect-ex-1-oljbc?fontsize=13&view=split" title="useEffect-ex-1" allow="geolocation; microphone; camera; midi; vr; accelerometer; gyroscope; payment; ambient-light-sensor; encrypted-media; usb" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>

그런데 앞서 말한 것처럼, 위의 코드처럼 디펜던시 목록을 넘겨주지 않으면, 컴포넌트가 렌더링될 때마다 이펙트와 이펙트에 대한 뒤처리가 실행됩니다. 위에서 `title` 필드를 변경하면, `App` 컴포넌트가 다시 렌더링되고, 타이머와 관련된 이펙트와 이펙트의 뒤처리가 계속해서 실행됩니다. 이때 이펙트와 관련된 디펜던시가 변경되었을 때만 넘겨줘서 useEffect가 이펙트가 되게 할 수 있습니다.

#### 특정 조건에서 이펙트 실행

 <iframe src="https://codesandbox.io/embed/useeffect-ex-2-1678x?fontsize=13&view=split" title="useEffect-ex-2" allow="geolocation; microphone; camera; midi; vr; accelerometer; gyroscope; payment; ambient-light-sensor; encrypted-media; usb" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>

위의 예제에서는 디펜던시 목록으로 `[count]` 를 넘겨줘서 `count` 가 변경되었을 때만 이펙트가 다시 실행되도록 하였습니다. 앞의 예제와 다르게 `Title` 을 변경한다고 해서 이펙트가 다시 실행되지는 않습니다.

만약 클래스 컴포넌트의 `componentDidMount` 혹은 `componentWillUnmount` 메서드처럼, 컴포넌트가 처음 렌더링될 때와 마지막으로 컴포넌트가 제거될 때 실행하고 싶은 이펙트는 어떻게 정의할까요? 두 번째 인자로 빈 배열을 넘겨주면 됩니다.

 <iframe src="https://codesandbox.io/embed/useeffect-ex-3-26dgt?fontsize=13&view=split" title="useEffect-ex-3" allow="geolocation; microphone; camera; midi; vr; accelerometer; gyroscope; payment; ambient-light-sensor; encrypted-media; usb" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>

예제를 보면 버튼을 클릭하여 `Pannel` 컴포넌트가 보일 때는 콘솔에 `component did mount` 라는 메시지가 출력되고, 컴포넌트가 사라질 때는 `component will unmount`라는 메시지가 출력되는 것을 확인할 수 있습니다. 이것이 가능한 이유는 빈 배열을 넘겼을 때 특별한 동작을 하도록 설계된 것은 아니고, 단순히 useEffect가 디펜던시 목록에 기반하여 이펙트를 실행하는 원리에 기반한 것입니다.

### 3. useContext

```jsx
const value = useContext(MyContext);
```

useContext는 `React.createContext`로 생성된 컨텍스트 객체를 인자로 받고 그 컨텍스트의 현재 값을 반환합니다. 현재 컨텍스트 값은, useContext를 호출하는 컴포넌트의 상위 컴포넌트 중 가장 가까운 `<MyContext.Provider>`에서 제공하는 `value` prop에 의해 결정됩니다.

가장 가까운 `<MyContext.Provider>`가 업데이트되면 이 Hook은 `MyContext` 프로바이더에 넘겨진 가장 최신의 컨텍스트 value와 함께 리-렌더링을 일으킬 것입니다.

useContext를 호출하는 컴포넌트는 컨텍스트 값이 변경될 때마다 리-렌더링될 것입니다. 컴포넌트 리-렌더링 비용이 크다고 생각된다면 뒤에서 설명할 메모이제이션으로 최적화할 수도 있습니다.

> `useContext(MyContext)`는 `MyContext`의 현재 값을 읽고, 변경 사항을 구독할 뿐입니다. Hooks가 나오기 전처럼 `<MyContext.Provider>`를 상위 컴포넌트에 위치시켜야 해줘야 하는 것은 동일합니다.

### 4. useReducer

```jsx
const [state, dispatch] = useReducer(reducer, initialArg, init);
```

useReducer는 useState의 **대안**입니다. useReducer는 `(state, action) => newState` 형태의 리듀서를 받고, `dispatch` 메서드와 함께 현재 상태를 반환합니다. 공식 문서에서는 Redux에 친숙하다면 이것이 어떻게 동작하는지 잘 안다고 나와있는데, 저는 Redux를 잘 모르기때문에 조금 더 정리해보겠습니다.

useReducer가 useState보다 더 적합한 상황은 여러개의 부수적인 값들(예를 들면, 객체의 프로퍼티들)을 포함하거나, 이전의 상태에 다음 상태가 의존하는 경우 상태 관련 로직이 복잡해질 수 있는데, 이때는 useState보다 useReducer가 적합합니다.

#### Dispatch 전달하기

useReducer에서 반환하는 `dispatch`를 하위 컴포넌트에 내려주는 방법 중에는 먼저 prop으로 하위 컴포넌트에 콜백을 전달하는 방법이 있습니다. 하지만 `dispatch`를 필요로 하지 않은 중간 계층의 컴포넌트에서도 `dispatch`를 하위로 내려주는 코드를 작성해줘야만 합니다. 이때 `React.createContext`를 사용하여 `dispatch`를 위한 컨텍스트 객체를 만든 후, `dispatch`를 호출할 하위 컴포넌트에서 `useContext`로 전달받는 방법이 있습니다.

코드로 설명하면 `dispatch`를 전달하기 위한 컨텍스트 객체를 만들고 프로바이더의 `value` prop으로 `dispatch` 함수를 넘겨줍니다.

```jsx
const TodosDispatch = React.createContext(null);

function TodosApp() {
  const [todos, dispatch] = useReducer(todosReducer);
  return (
    <TodosDispatch.Provider value={dispatch}>
      <DeepTree todos={todos} />
    </TodosDispatch.Provider>
  );
}
```

`dispatch`를 호출할 컴포넌트에서는 useContext를 사용하여 `dispatch`를 가져오면 됩니다. 이때 `dispatch` 함수에 대한 참조는 절대 변하지 않기 때문에, `dispatch`로 인해 `TodosDispatch`를 참조하는 하위 컴포넌트들이 다시 렌더링되는 일은 없습니다.

```jsx
function DeepChildComponent(props) {
  const dispatch = useContext(TodosDispatch);
  function handleClick() {
    dispatch({ type: "add", text: "hello" });
  }
  return <button onClick={handleClick}>Add todo</button>;
}
```

#### 초기 상태 설정

useReducer가 어떻게 사용되는지에 따라서 다른 방법으로 초기 상태를 설정할 수 있습니다. 가장 간단한 방법은useReducer의 두 번째 인자로 초기 상태을 넘겨주는 방법입니다.

```jsx
const [state, dispatch] = useReducer(reducer, { count: initialCount });
```

useState처럼 useReducer도 초기 상태를 계산하기 위한 함수를 사용할 수 있습니다. useReducer의 세 번째 인자인 `init`으로 초기 상태를 반환하는 함수를 넘겨주면 됩니다. 단, 이 경우 두 번째 인자인 `initialState`를 기반으로 만들어지는데, `init(initialState)`의 결과값이 초기 상태로 설정됩니다.

<iframe src="https://codesandbox.io/embed/usereducer-ex-1-i2g91?fontsize=13&view=split" title="useReducer-ex-1" allow="geolocation; microphone; camera; midi; vr; accelerometer; gyroscope; payment; ambient-light-sensor; encrypted-media; usb" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>

마지막으로 useState처럼 useReducer도 현재 상태와 이전의 상태가 같다면 리액트는 자식 컴포넌트를 리-렌더링하지 않고, 이펙트도 발생시키지 않습니다.

### 5. useCallback

useCallback은 메모이즈된 콜백을 반환합니다.

```jsx
const memoizedCallback = useCallback(() => {
  doSomething(a, b);
}, [a, b]);
```

인자로는 콜백 함수와 디펜던시 배열을 넘깁니다. useCallback은 콜백의 메모이즈된 버전을 반환합니다. 메모이즈된 콜백은 디펜던시 중 하나가 변경되었을 때만 변경됩니다. 이것은 불필요한 렌더를 방지하기 위하여 레퍼런스 비교에 의존하도록 최적화된 자식 컴포넌트에 콜백을 넘길 때 유용합니다.

> useCallback은 뒤에서 설명하게 될 useMemo를 편하게 사용하기 위한 일종의 shortcut으로 볼 수도 있습니다. `useCallback(fn, deps)` 는 `useMemo(() => fn, deps)`와 동일한 표현입니다.

아래의 예제는 디펜던시 목록에 정의된 것 중 하나가 변경되었을 때만 새로운 콜백 함수를 생성하는지 테스트하기 위한 예제입니다. 예제에서는 `firstname`, `lastname` 두 개의 인풋이 있는데, `firstname`이 변경될 때만 useCallback으로 새로운 콜백 함수를 생성하는 것을 확인할 수 있습니다.

<iframe src="https://codesandbox.io/embed/usecallback-2-8zpvw?fontsize=13&view=split" title="useCallback-2" allow="geolocation; microphone; camera; midi; vr; accelerometer; gyroscope; payment; ambient-light-sensor; encrypted-media; usb" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>

### 6. useMemo

`useMemo`는 메모이즈된 값을 반환합니다.

```jsx
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

인자로는 “create” 함수와 디펜던시 배열을 받습니다. useMemo는 디펜던시 중 하나가 변경되었을 때만 메모이즈된 값을 다시 계산할 것입니다. useMemo의 기능을 활용하면 렌더링이 일어날 때마다 시간이 오래 걸리는 연산을 실행하는 것을 피할 수 있습니다. 만약 배열이 제공되지 않으면 컴포넌트가 렌더링될 때마다 새로운 값이 계산될 것이기 때문에 useMemo를 사용하는 의미가 거의 없어집니다.

useMemo는 렌더링 과정에서 넘겨받은 함수를 실행한다는 점을 기억해야 합니다. 렌더링 과정에서 해서는 안 되는 것들을 useMemo에 넘기는 콜백 함수에 작성해서는 안 됩니다. 예를 들어서 사이드 이펙트가 포함되어 있다면 useMemo가 아니라 useEffect를 사용해야 합니다.

또 useMemo는 최적화의 수단으로 사용되어야 합니다. 미래의 리액트에서는 useMemo로 메모이즈된 값을 "잊어버리는" 방법을 선택할 수도 있습니다. 예를 들어서 스크린에서 벗어난 컴포넌트 메모리의 해제가 필요한 상황이 있습니다. 공식 문서에서는 우선 useMemo없이 코드가 동작하도록 만들고, 최적화하는 단계에서 useMemo를 사용하는 것을 권장합니다.

### 7. useRef

useRef는 mutable한 ref 객체를 반환합니다.

```jsx
const refContainer = useRef(initialValue);
```

useRef에 `initialValue` 인자를 전달하면 그 값은 ref 객체의 `current` 프로퍼티의 초기값이 됩니다. 반환된 객체는 컴포넌트가 살아있는 시간동안에 계속해서 유지됩니다.

`useRef`는 Hooks가 나오기 전의 refs처럼 리액트에서 DOM을 참조하기 위한 방법으로 사용될 수도 있습니다.

```jsx
function TextInputWithFocusButton() {
  const inputEl = useRef(null);
  const onButtonClick = () => {
    // `current` points to the mounted text input element
    inputEl.current.focus();
  };
  return (
    <>
      <input ref={inputEl} type="text" />
      <button onClick={onButtonClick}>Focus the input</button>
    </>
  );
}
```

위의 예제에서 리액트는 노드가 변경될 때마다 그 당시의 DOM 노드를 ref 객체의 `current` 프로퍼티에 설정할 것입니다.

`useRef`는 DOM을 참조하는 것 외에 다른 용도로도 사용할 수 있는데, 바로 클래스에서 인스턴스 필드를 사용하는 것처럼 mutable한 값을 간편하게 저장하는 것입니다. `useRef`는 순수 자바스크립트 객체를 생성하지만, `{current: …}`와의 차이점은 `useRef`는 렌더링이 일어나도 동일한 참조 객체를 유지한다는 점입니다. `useRef`의 `current`는 그 값이 변경되어도 렌더링을 일으키거나 하지 않습니다.

이제 `source`가 변경될 때마다 subscription이 다시 생성되게 됩니다.
