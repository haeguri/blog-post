---
title: Express res.json과 res.send 비교
---

[개인 프로젝트](https://github.com/haeguri/reviewers)를 개발하면서 서버가 필요하게 되었는데 자바스크립트를 더 공부해보고 싶어서 서버를 Node.js, Express를 사용하여 개발하고 있다.

지금 작성된 모든 서버 API는 Express를 통해서 JSON 응답을 하도록 구현되어 있다. 그런데 내가 작성한 코드를 살펴보다가 JSON 응답을 보낼 때 Express의 `res.json` 함수와 `res.send` 함수를 혼용하고 있었다는 사실을 알았다.  

클라이언트에서는 API를 호출했을 때 JSON 응답을 받은 것을 확인했으니, 두 개의 함수 모두 JSON 데이터를 보낼 수 있다는 말인데, 이렇게 사용해도 상관없는 것인지 궁금했다. 그래서 `res.send` 함수와 `res.json` 함수의 내부 코드를 살펴보기로 했다. 

### res.json [[source](https://github.com/expressjs/express/blob/master/lib/response.js#L239)]

먼저 `res.json` 소스코드의 일부는 다음과 같다. 

```javascript
res.json = function json(obj) {
  var val = obj;

  // 생략...

  var app = this.app;
  var escape = app.get('json escape')
  var replacer = app.get('json replacer');
  var spaces = app.get('json spaces');
  var body = stringify(val, replacer, spaces, escape)

  if (!this.get('Content-Type')) {
    this.set('Content-Type', 'application/json');
  }

  return this.send(body);
};
```

`res.json` 함수에 명시된 인자로는 `obj`가 있다. `obj`는 JSON 문자열로 변환되서 `body`라는 변수에 저장된다. 여기서 **Content-Type** 헤더가 세팅되지 않았을 경우 **this**(res 객체)에 **Content-Type** 으로 **application/json**을 세팅한다. 그리고 마지막으로 `res.send(body)`를 실행하면서 그 결과를 반환한다. 결국은 `res.json`은 내부적으로 `res.send`를 호출하고 있었다. 

### res.send [[source](https://github.com/expressjs/express/blob/master/lib/response.js#L107)]

`res.send`의 소스코드의 일부는 다음과 같으며, `chunk` 타입에 따른 실행 흐름을 분기하는 코드에 초점을 맞춰서 살펴봤다. 

```javascript
res.send = function send(body) {
    var chunk = body;
    
    // 생략....

    switch (typeof chunk) {
        // string defaulting to html
        case 'string':
            if (!this.get('Content-Type')) {
                this.type('html');
            }
            break;
        case 'boolean':
        case 'number':
        case 'object':
            if (chunk === null) {
                chunk = '';
            } else if (Buffer.isBuffer(chunk)) {
                if (!this.get('Content-Type')) {
                    this.type('bin');
                }
            } else {
                return this.json(chunk);
            }
            break;
    }
    
    // 생략..

    return this;
}
```

`res.send` 함수의 인자로는 `body`가 있다. `body`는 바로 `chunk`로 할당되고, `chunk`에 대한 타입 검사가 진행된다. 여기에서 `chunk`가 **object** 타입이면 `res.json`을 호출한다. 여기서 의문이 들었는데, 이렇게 되면 두 개의 함수가 서로 호출하기 때문에 함수 호출 스택이 넘쳐버리지 않을까 생각했다. 

그래서 `res.send(object)`로 코드를 실행했을 때 함수의 실행 순서가 어떻게 되는지 살펴봤다.

1. **res.send(object)**
2. **res.json(object)**
3. **res.send(string)**

`res.json`에서 `res.send`를 호출할 때는 `body`로 문자열을 넘겨주기 때문에, **두 번째 실행**되는 `res.send`의 `chunk` 타입은 **string**이다. `chunk`가 **string**일 경우에는 **object**일 때와 다른 분기를 타게 되서 `res.json`을 호출하지 않기 때문에, 계속해서 서로를 호출하는 일은 없게 된다.

### 실행 흐름을 비교

그러면 두 개의 함수를 각각 사용했을 때 차이점은 무엇일까 생각해봤다. 그리고 각각을 실행했을 때 실행 흐름이 어떻게 흘러가는지 비교해봤다.

`res.send(object)`를 실행하면 함수의 호출 순서는 다음과 같다. 

1. **res.send(object)**
2. **res.json(object)**
3. **res.send(string)**

그리고 `res.json(object)`를 실행했을 때 함수의 호출 순서는 다음과 같다.

1. **res.json(object)**
2. **res.send(string)**

`object`를 인자로 `res.send`를 호출하면 `res.json`을 호출했을 때 보다 **불필요한 호출**이 한 번 더 발생한다.

### 결론

불필요한 함수 호출이 한번 더 발생하는 것은 어쨌든 추가 비용은 발생하는 것이기 때문에, JSON 응답을 한다면 `res.send`보다 `res.json`이 적절한 것 같다.

또한 소스코드를 읽을 때에도 `res.json`이 JSON 데이터를 보낸다는 의도가 더 명확하게 드러난다. 예를 들어서 `res.send({data:1})`라면 객체를 즉시 생성해서 전달하기 때문에 JSON을 응답하는 것을 유추할 수 있지만, `res.send(data)`처럼 객체의 참조값을 변수에 담아서 인자로 넘긴다면 JSON 응답을 하는지 쉽게 구분이 가지 않을 것이다.