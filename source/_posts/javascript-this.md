---
title: 자바스크립트의 this 살펴보기
date: 2019-02-02 18:29:42
tags:
- javascript
- this
---

## 1. 소개

자바스크립트의 `this`는 일반적으로 함수 안에서만 사용된다. 함수 안의 `this`는 **정적(static)**으로 정해질 수도 있고, 실행되면서 **동적(dynamic)**으로 정해질 수도 있다.

## 2. 정적으로 정해지는 경우

`this`가 정적으로 정해진다는 것은 소스코드를 작성하는 단계에서 정해지는 것을 의미한다. 이 때 `this`는 **‘함수’**안에 있을 때와 **‘메서드’**에 있을 때가 다르다.

### 2.1. 함수안의 this

그리고 함수안의 `this`는 `'use strict'`를 사용해서 **'엄격 모드'**로 실행되는 경우와, **'비엄격 모드'**에서 실행되는 경우에서 정해지는 값이 달라진다.

#### 비엄격모드

'비엄격 모드'에서 함수안의 `this`는 `window` 객체를 가리킨다.

```javascript
function func() {
    return this;
}

window === func(); // true
```

#### 엄격모드

자바스크립트가 ‘엄격 모드’에서 실행될 때, 함수 안의 `this`는 `undefined`이 할당된다.

```javascript
'use strict'

function func() {
    return this;
}

window === func(); // false
undefined === func(); // true
```

### 2.2. 메서드안의 this

함수가 객체의 프로퍼티의 값으로 할당되면 그 함수를 **메서드**라고 부른다. 메서드 안에서 `this`가 사용되면, 메서드의 `this`는 메서드가 속한 객체를 가리킨다. 메서드 안의 `this`가 정해지는 매커니즘은 ‘엄격 모드’를 적용했을 때에도 동일하게 동작한다.

```javascript
// 엄격모드 사용
'use strict'

function func() {
    return this;
}
// func를 obj의 methodA로 정의한다.
var obj = {
    methodA: func;
}

// func를 obj의 methodB로 정의한다.
obj.methodB = func;
obj.methodA() === obj; // true
obj.methodB() === obj; // true
```

### 2.3. 화살표 함수

ECMAScript 6에서 추가된 **화살표 함수**는 화살표 함수 안의 `this`를 따로 할당하지 않으며, 화살표 함수를 감싸고 있는 컨텍스트에서의 `this`가 할당된다.

```javascript
'use strict';

// 그냥 함수안의 this
function test1() {
    return this;
}
this === window; // true
this === test1(); // false

// 화살표 함수안의 this
const test2 = () => {
    return this;
}
this === window; // true
this === test2(); // true
```

## 3. 동적으로 결정

`this`는 어떻게 호출되는지에 따라서 동적으로 변경될 수도 있다. 즉, 프로그램이 실행되는 과정에서 `this`를 정할 수 있다.

`this`를 동적으로 변경하는 것은 Function 프로토타입 객체의 메서드인 `call`, `bind`, `apply`로 변경할 수 있다. 자바스크립트의 메서드를 포함한 모든 함수는 내부적으로 Function의 인스턴스가 되기 때문에, 모든 함수들은 프로토타입 체인을 통해서 프로토타입 객체에 정의된 `call`, `bind`, `apply` 메서드를 호출할 수 있다. 

### 3.1. Function.prototype.call

`call` 메서드는 호출하는 함수의 `this`를 다시 정의하면서 함수를 호출한다. `call` 메서드에 넘겨줘야 하는 인자는 다음과 같다.

* 첫 번째 인자는 `this`로 참조할 객체이다.
* 그 외의 인자들은 함수에 넘겨줄 인자들이다.

```javascript
function add(a, b) {
    return this.baseNumber + a + b;
}

add.call({baseNumber: 0}, 1, 2); // 3
add.call({baseNumber: 1}, 1, 2); // 4
```

### 3.2. Function.prototype.apply

`apply` 메서드도 `this`를 재정의 하면서 함수를 호출한다. 하지만 `this` 다음으로 넘기는 인자가 `call` 메서드와는 다르다.

* 첫 번째 인자는 `this`로 참조할 객체이다.
* 두 번째 인자는 함수에 전달할 인자가 담긴 **배열**이다.

```javascript
function add(a, b) {
    return this.baseNumber + a + b;
}
var params = [1, 2];
add.apply({baseNumber: 0}, params); // 3
add.apply({baseNumber: 1}, params); // 4
```

### 3.3. Function.prototype.bind

`bind` 메서드는 인자의 구성은 `call` 메서드와 동일하다. 하지만 `bind`는 `call`처럼 함수를 호출하는 것이 아니라 새로운 함수를 생성한다.

```javascript
function add(a, b) {
    return this.baseNumber + a + b;
}

var newAdd1 = add.bind({baseNumber: 0}, 1, 2); 
var newAdd2 = add.bind({baseNumber: 1}, 1, 2);

newAdd1(); // 3
newAdd2(); // 4
```

그리고 `bind`는 새로운 함수를 생성하기만하고 호출은 하지 않는데, 이것을 이용해서 함수가 나중에 호출될 때 고정된 인자를 갖도록 할 수 있다.

```javascript
// `bind`를 호출할 때는 첫 번째 인자만 넘겨준다.
var newAdd1 = add.bind({baseNumber: 0}, 1); 
var newAdd2 = add.bind({baseNumber: 1}, 1);

// 나머지 두 번째 인자를 넘겨주면서 호출한다.
newAdd1(2); // 3
newAdd2(3); // 4
```

만약, `bind`와 같은 기능을 하는 함수를 구현한다면 다음과 같을 것이다.

```javascript
Function.prototype.bind = function() {
  var bindArgs = Array.prototype.slice.apply(arguments);
  
  var self = this;
  var newThis = bindArgs.shift();
  
  return function() {
    var extraArgs = Array.prototype.slice.apply(arguments) ;
    
    for(var i = 0; i < extraArgs.length; i++) {
      bindArgs.push(extraArgs[i]);
    }
    
    return self.apply(newThis, bindArgs);
  }
}    
```

## 4. 정리

정리하자면 `this`는 정적으로 정해질 수도 있고, 동적으로 정해질 수도 있다. 정적으로 결정될 때는 `this`가 소스코드에서 **'`function` 함수'**, **'메서드'**, **'화살표 함수'** 중 어디에 속해있는지에 따라서 달라진다. 그리고 `this`를 동적으로 결정하고 싶다면 `call`, `apply`, `bind` 메서드를 활용해서 함수의 `this`를 직접 설정해줄 수 있다.

## 5. 참고자료

* [MDN this](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/this)
* [MDN bind](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)
* [MDN 화살표  함수](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Functions/%EC%95%A0%EB%A1%9C%EC%9A%B0_%ED%8E%91%EC%85%98)