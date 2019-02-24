---
title: ( 번역 ) Refs and the DOM
---

> 리액트 문서의 "Refs and the DOM"을 번역한 자료 
> 원문 : https://reactjs.org/docs/refs-and-the-dom.html

Ref는 DOM 노드 혹은 `render` 메서드로 생성된 리액트 엘리먼트에 접근할 수 있는 방법을 제공한다.

일반적인 리액트 데이터 플로우에서는 [props](https://reactjs.org/docs/components-and-props.html)만이 부모 컴포넌트와 자식 컴포넌트와 상호작용할 수 있는 유일한 방법이다. 자식을 수정하기 위해서는 새로운 props을 넘겨줘야 자식이 다시 렌더링된다. 하지만 몇 가지 예외 경우에는 일반적인 데이터 플로에서 벗어나 긴급하게 자식을 수정해야 할 경우가 있다. 여기서 말하는 자식은 리액트 컴포넌트의 인스턴스이거나 DOM 요소일 수도 있다. 이러한 경우, 리액트는 비상용 탈출구를 제공한다.

### 언제 Ref를 쓰는가?

Ref를 사용하면 좋은 몇 가지의 경우가 있다.

* 포커스, 텍스트 선택, 미디어 재생을 다룰 때
* 애니메이션을 시작할 때
* 써드-파티 DOM 라이브러리와 상호작용

선언적으로 수행할 수 있는 모든 것에 대해서는 Ref를 사용하지 마라.

예를 들어서, `Dialog` 컴포넌트의 `open()`과 `close()` 메서드를 외부에 노출하는 것 대신에 `Dialog` 컴포넌트로 `isOpen` prop을 넘겨줘라.

### Ref를 남용하지 마라

당신은 처음에 Ref를 당신의 앱에서 “무언가를 일어나도록 하기 위해” 사용할 것이다. 이 경우에는 시간을 갖고 컴포넌트 계층에서 상태를 소유해야 하는 장소에 대해서 비판적으로 생각해봐라. 보통은 상태를 소유하는 적절한 장소가 더 높은 위치에 있는 것을 알게 된다. 이러한 경우에는 [Lifting State Up](https://reactjs.org/docs/lifting-state-up.html) 가이드를 읽어봐라.

> **주의**
> 아래의 예제는 16.3에서 소개된 `React.createRef()` API를 사용하도록 업데이트됐다. 만약 이전의 React 버전을 사용한다면 [callback refs](#callback-refs)를 사용하는 것을 권한다.

### <a id="creating-refs"></a> Ref 만들기

Ref는 `React.createRef()`로 만들어지며 `ref` 속성으로 리액트 엘리먼트와 연결한다. 보통 Ref는 컴포넌트가 생성될 때 인스턴스 프로퍼티로 연결되서 컴포넌트를 통해 참조될 수 있게 된다.

```jsx
class MyComponent extends React.Component {
    constructor(props) {
        super(props);
        this.myRef = React.createRef();
    }
    render() {
        return <div ref={this.myRef} />;
    }
}
```

### Ref에 접근하기

Ref가 `render`에서 엘리먼트로 전달되면, 노드에 대한 참조는 ref의 `current` 속성에서 접근할 수 있게 된다.

``` js
const node = this.myRef.current;
```

Ref의 값은 노드의 유형에 따라 달라진다.

* `ref` 속성이 HTML 엘리먼트에서 사용될 때에는 생성자에서 `Reac.createRef()`로 만들어지는 `ref`는 `current` 프로퍼티를 통해 DOM 엘리먼트를 받게 된다.
* `ref` 속성이 커스텀 클래스 컴포넌트에서 사용될 때, `ref` 객체는 `current`를 통해 마운트된 컴포넌트의 인스턴스를 받는다.
* **함수 컴포넌트는 인스턴스가 없기 때문에 `ref` 속성은 사용할 수 없다.**

아래의 예제들은 차이점을 보여준다.

#### Ref를 DOM 엘리먼트에 추가하기

이 코드는 `ref`를 DOM 노드에 대한 참조를 저장하기 위해 사용한다.

```jsx
class CustomTextInput extends React.Component {
    constructor(props) {
        super(props);
        // textInput DOM 엘리먼트를 저장하기 위한 ref를 만든다.
        this.textInput = React.createRef();
        this.focusTextInput = this.focusTextInput.bind(this);
    }

    focusTextInput() {
        // DOM API를 이용하여 명시적으로 텍스트 입력을 포커스한다.
        // 주의: DOM 노드를 가져오기 위해 `current`에 접근하고 있음.
        this.textInput.current.focus();
    }

    render() {
        // React에게 우리가 생성자에서 만들었던 `textInput` 프로퍼티에 <input> ref를 연결하고 싶다고 말한다.
        return (
            <div>
                <input
                    type="text"
                    ref={this.textInput} />
                <input
                    type="button"
                    value="Focus the text input"
                    onClick={this.focusTextInput}
                />
            </div>
        );
    }
}
```

리액트는 컴포넌트가 마운트 될 때 DOM 요소를 `current` 프로퍼티에 할당할 것이며 컴포넌트가 언마운트되면 다시 `null`을 할당할 것이다. `ref`는 `componentDidMount` 혹은 `componentDidUpdate` 라이프 사이클 훅이 실행되기 전에 갱신된다.

#### <a id="adding-ref-to-class-comp"></a>Ref를 클래스 컴포넌트에 추가하기

만약 위의 `CustomTextInput`을 래핑해서 컴포넌트가 마운트되면 입력이 클릭된 것처럼 하려는 경우, ref를 통해서 커스텀 입력에 접근하고, `focusTextInput` 메서드를 수동으로 호출할 수 있다.

```jsx
class AutoFocusTextInput extends React.Component {
    constructor(props) {
        super(props);

        this.textInput = React.createRef();
    }

    componentDidMount() {
        this.textInput.current.focusTextInput();
    }

    render() {
        return (
            <CustomTextInput ref={this.textInput} />
        );
    }
}
```

`CustomTextInput`가 클래스로서 선언되어야만 동작할 수 있음을 유의해라.

```jsx
class CustomTextInput extends React.Component {
  // ...
}
```

#### Ref와 함수형 컴포넌트

함수형 컴포넌트들은 인스턴스가 없기 때문에 ** `ref` 속성을 가질 수 없을 것이다.**

```jsx
function MyFunctionalComponent() {
    return <input />;
}

class Parent extends React.Component {
    constructor(props) {
        super(props);
        this.textInput = React.createRef();
    }
    render() {
        // 이것은 동작하지 않을 것이다!
        return (
            <MyFunctionalComponent ref={this.textInput} />
        );
    }
}
```

만약 컴포넌트에 대한 ref가 필요하다면 라이프 사이클 메서드나 상태가 필요할 경우와 마찬가지로 함수형 컴포넌트들은 클래스로 변환해야 한다.

그러나 **함수형 컴포넌트 내에서** DOM 엘리먼트 혹은 클래스 컴포넌트를 참조한다면 **ref 속성을 사용할 수 있다**.

```jsx
function CustomTextInput(props) {
    // textInput은 여기서 선언되어야 ref가 DOM 엘리먼트를 참조할 수 있다.
    let textInput = React.createRef();
    function handleClick() {
        textInput.current.focus();
    }

    return (
        <div>
            <input
                type="text"
                ref={textInput} />
            <input
                type="button"
                value="Focus the text input"
                onClick={handleClick}
            />
        </div>
    );
}
```

### 부모 컴포넌트에 DOM Ref를 노출하기

드문 경우에 부모 컴포넌트에서 자식의 DOM 노드에 접근하기를 원할 수 있다. 이것은 컴포넌트 캡슐화를 망가뜨리기 때문에 일반적으로 추천되지 않지만, 포커스를 주거나 자식 DOM 노드의 크기나 포지션을 측정할 때 유용할 수 있다.

위에서 [자식 컴포넌트에 ref를 추가할 수 있었지만](#adding-ref-to-class-comp), DOM 노드보다는 컴포넌트 인스턴스만을 가져오기 때문에 이것은 이상적인 해결책이 아니다. 덧붙이면 이것은 함수형 컴포넌트로도 할 수 없다.

이러한 경우에서 리액트 16.3 이상을 사용한다면 [ref forwarding](https://reactjs.org/docs/forwarding-refs.html)을 추천한다. Ref Forwarding은 컴포넌트가 자식 컴포넌트의 ref를 노출하는 것을 선택할 수 있게 한다. 자식의 DOM 노드를 부모 컴포넌트에 노출하는 방법에 대한 자세한 예제는 [ref forwarding 문서](https://reactjs.org/docs/forwarding-refs.html#forwarding-refs-to-dom-componentsV)에서 찾을 수 있다.

만약 리액트 16.2 이하을 사용하거나 ref forwarding에 의해 제공되는 기능보다 유연해야 한다면, [이 대안](https://gist.github.com/gaearon/1a018a023347fe1c2476073330cc5509)을 사용하거나 다르게 명명된 prop으로 ref를 명시적으로 넘겨줄 수 있다.

가능하다면, DOM 노드를 노출하지 않을 것을 권장하지만, 때때로 유용할 수 있다. 이 접근 법은 자식 컴포넌트에 어떤 코드를 추가해야 할 경우의 접근 법이다. 만약 자식 컴포넌트 구현체를 변경할 수 없다면 마지막 선택지는 [findDOMNode](https://reactjs.org/docs/react-dom.html#finddomnode)를 사용하는 것이지만, 실망스러울 것이다.

### <a id="callback-refs"></a> Callback Refs

리액트는 Ref를 설정하기 위한 또 다른 방법인 'callback refs'를 제공한다. 'callback refs'는 Ref가 설정되고 해제될 때 더욱 세세한 제어를 하게 한다.

`createRef( )`로 만든 `ref`속성을 넘겨주는 대신에, 함수를 넘겨줄 수 있다. 함수는 리액트 컴포넌트 인스턴스 혹은 HTML DOM 엘리먼트를 인자로 받는다. 그리고 인자는 다른 곳에 저장하고, 접근할 수 있다.

아래의 예제는 인스턴스 프로퍼티에서 DOM 노드에 접근할 참조를 저장하기 위한 `ref` 콜백을 사용하는 일반적인 패턴을 구현한다. 

```jsx
class CustomTextInput extends React.Component {
    constructor(props) {
        super(props);

        this.textInput = null;
        this.setTextInputRef = element => {
            this.textInput = element;
        };
        this.focusTextInput = () => {
            // 원래의 DOM API를 이용하여 텍스트 입력을 포커싱한다
            if (this.textInput) this.textInput.focus();
        };
    }

    componentDidMount() {
        // 마운트됐을 때 입력을 포커싱을 한다.
        this.focusTextInput();
    }

    render() {
        // 인스턴스 필드(`this.textInput`)에 텍스트 입력 DOM 엘리먼트에 대한
        // 참조를 저장하기 위해 `ref` 콜백을 사용한다
        return (
            <div>
                <input
                    type="text"
                    ref={this.setTextInputRef}
                />
                <input
                    type="button"
                    value="Focus the text input"
                    onClick={this.focusTextInput}
                />
            </div>
        );
    }
}
```

리액트는 컴포넌트가 마운트되면 DOM 엘리먼트를 파라메터로 `ref` 콜백을 실행할 것이다. 그리고 언마운트되면 `null`을 파라메터로 호출할 것이다. Ref는 `componentDidMount` 혹은 `componentDidUpdate`가 호출되기 전에 최신 상태로 보장된다.

`React.createRef()`로 만든 객체 Ref와 같이 컴포넌트 사이에서 Ref 콜백을 넘겨줄 수 있다. 컴포넌트 사이에서 콜백을 호출할 수 있다

```jsx
function CustomTextInput(props) {
    return (
        <div>
            <input ref={props.inputRef} />
        </div>
    );
}

class Parent extends React.Component {
    render() {
        return (
            <CustomTextInput
                inputRef={el => this.inputElement = el}
            />
        );
    }
}
```

위의 예제에서, `Parent`는 ref 콜백을 `inputRef` prop으로 `CustomTextInput`에 넘겨준다. `CustomTextInput`은 넘겨받은 함수를 특별한 `ref` 속성으로써  `<input>`로 넘긴다.

### 레거시 API: 문자열 Ref

예전에 리액트를 사용했다면 `ref` 속성이 `”textInput"`과 같이 문자열이고, DOM 노드를 `this.refs.textInput`으로 접근하는 오래된 API에 더 친숙할 것이다. 문자열 Ref는 [어떤 이슈](https://github.com/facebook/react/pull/8333#issuecomment-271648615)들을 가져서 레거시로 간주됐고, 앞으로의 릴리즈에서 제거될 것이기 때문에 문자열 Ref를 사용하는 일은 피해라

> **주의**
> Ref에 접근하기 위해 `this.refs.textInput`를 사용하고 있다면, [콜백 패턴](#callback-refs)이나 [createRef API](#creating-refs)를 대신에 사용할 것을 권장한다.

### Callback Refs를 사용할 때 주의사항

`ref` 콜백이 인라인 함수로 정의된다면 컴포넌트가 업데이트되는 동안 두 번 호출될 것이다. 첫 번째는 `null`, 그 다음에는 DOM 엘리먼트가 파라메터로 넘어간다. 이것은 각각의 `render` 메서드로 함수의 새 인스턴스가 만들어지기 때문인데, 그래서 리액트는 예전의 ref를 비워우고 새로운 것으로 설정해줄 필요가 있다. `ref` 콜백을 클래스에 바운드된 메서드로 정의하면 이것을 피할 수 있지만, 대부분의 경우에서는 문제가 되지 않음을 기억해라.

