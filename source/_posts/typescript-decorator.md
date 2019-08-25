---
title: 타입스크립트 Decorator
date: 2019-08-25 20:07:00
tags:
- typescript
- decorator
---

타입스크립트의 데코레이터를 사용하면 클래스의 메서드, 프로퍼티, 접근자 혹은 함수의 파라미터에 이전에 제공하지 않던 방법으로 새로운 기능을 추가할 수 있습니다. 사실 데코레이터라는 문법은 이미 자바스크립트 표준으로써 논의되고 있는 단계이며 현재는 초안 단계에 있습니다. 자바스크립트를 확장한 언어라고 할 수 있는 타입스크립트에서는 실험적인 기능으로써 데코레이터를 제공하고 있습니다. 

 ## 시작하기

 데코레이터는 타입스크립트에서도 실험적인 기능으로써 사용할 수 있기 때문에 커맨드 라인이나 `tsconfig.json` 에서 `experimentalDecorators` 옵션을 추가해줘야 합니다. 여기에서 `target` 설정을 `ES5`로 하는 이유는 뒤에서 다시 살펴보겠습니다. 

### 커맨드 라인

```shell
tsc --target ES5 --experimentalDecorators
```

### tsconfig.json

```json
{
    "compilerOptions": {
        "target": "ES5",
        "experimentalDecorators": true
    }
}
```

## 살펴보기

 데코레이터는 앞에서 말한 것처럼 "클래스", "메서드", "접근자", "프로퍼티", "파라미터"에 적용할 수 있습니다. 데코레이터는 `@expression` 형식으로 적용하는데, 여기에서 `expression` 은 반드시 **함수**여야 합니다. 

 간단한 예로 `@firstDecorator` 라는 데코레이터를 클래스의 프로퍼티에 적용하고 싶다면 `findDecorator` 함수를 다음과 같이 작성하면 됩니다.

```typescript
function fistDecorator(target, name) {
    console.log('fistDecorator');
}

class Person {
    @fistDecorator
    job = 'programmer';
}

const p = new Person();
console.log(p.job);
```

실행결과

```shell
> fistDecorator
> programmer
```

### Decorator Factory

 위의 예제에서 `@firstDecorator` 데코레이터를 일반화해서 조금 더 다양한 상황에서 사용할 수 있도록 **파라미터**를 전달해야 할 수도 있습니다. 그러면 데코레이터 함수를 생성하는 함수인 **데코레이터 팩토리**를 작성하면 됩니다.

```typescript
function firstDecorator(param) {
    console.log('factory’);
    return function(target, name) {
        console.log('decorator');
    }
}

class SomeClass {
    @firstDecorator(123)
    prop = ‘a';
}

console.log('인스턴스가 만들어지기 전');
console.log(new SomeClass());
```

실행결과

```shell
> factory
> decorator
> 인스턴스가 만들어지기 전
```

 실행 결과를 살펴보면 데코레이터는 클래스를 인스턴스화하기 위해 클래스를 호출하기 전에 실행됩니다. 

### Decorator 조합하기

데코레이터의 또 다른 장점으로는 하나의 선언에 동시에 여러 개의 데코레이터를 적용할 수 있는 점이 있습니다.

```typescript
function decoA(param) {
    console.log('decoA factory');
    return function(target, name) {
        console.log('decyA decorator')
    }
}

function decoB(target, name) {
	console.log('decoB decorator');
}

function decoC(param) {
    console.log('decoC factory');
    return function(target, name) {
        console.log('decoC decorator');
    }
}

class SomeClass {
    @decoA(1)
    @decoB
    @decoC(2)
    prop = 1;
}
```

여러 개의 데코레이터가 적용될 때 실행되는 순서는 다음과 같습니다. 

실행결과

```shell
> decoA factory
> decoC factory
> decoC decorator
> decoB decorator
> decoA decorator
```

  `@expression` 에서 `expression` 표현식을 함수로 평가하는 순서는 **"위에서 아래"**입니다. 실행결과에서 `"decoA factory"` , `"decoC factory"` 만 출력되고 `"decoB decorator"` 가 출력되지 않은 이유는 `@decoB` 는 팩토리 함수가 없기 때문입니다.

 그리고 `expression` 이 함수로 평가된 후에 데코레이터 함수가 실행되는 순서는 **"아래에서 위"**입니다. 여기에서 말하는 데코레이터 함수란 `@decoA`, `@decoC`의 경우에는 팩토리 함수에서 반환하는 익명 함수이고, `@decoB`의 경우에는 `decoB` 함수입니다.

## Decorator 종류

 데코레이터는 클래스, 메서드, 프로퍼티, 접근자, 파라미터의 선언에 적용될 수 있습니다. 앞에서는 프로퍼티 데코레이터만을 예로 들었지만, 데코레이터가 어디에 적용되는지에 따라서, 데코레이터 함수로 넘어오는 인자의 길이나 구성이 달라집니다. 그래도 공통되는 부분도 있는데 공식 문서에 나와있는 것들을 추려보면 다음과 같습니다. 

- 데코레이터는 declaration 파일(`*.d.ts`), 혹은 `declare class` 안에서는 사용될 수 없습니다.
- 데코레이터 표현식은 런타임에 함수로서 호출됩니다.

### Class Decorator

 클래스 선언에 사용되는 클래스 데코레이터는 기존의 클래스 정의를 확장하는 용도로 사용할 수 있습니다. 

 클래스 데코레이터 함수의 인자로는 **클래스(생성자 함수)**가 전달됩니다. 클래스 데코레이터 함수에서는 **새로운 클래스(생성자 함수)**만을 반환할 수 있고, 함수 외의 값들은 무시됩니다. 

 다음은 타입스크립트 공식 문서에서 볼 수 있는 예제입니다.

```typescript
function classDecorator<T extends {new(...args:any[]):{}}>(constructor:T) {
  return class extends constructor {
      newProperty = "new property";
      hello = "override";
  }
}
```

 `@classDecorator` 데코레이터는 자신이 적용되는 클래스를 `extends`해서 새로운 프로퍼티를 추가하거나 기존의 프로퍼티를 오버라이드하는 역할을 합니다. 

```typescript
@classDecorator
class Greeter {
  property = "property";
  hello: string;
  constructor(m: string) {
      this.hello = m;
  }
}

console.log(new Greeter("world")); 
// {property: "property", hello: "override", newProperty: "new property"}
```

 실행결과를 보시면 `Greeter` 클래스에서 직접 선언하지 않았던 `newProperty` 프로퍼티가 추가되었고, `hello` 프로퍼티가 오버라이드된 것을 확인할 수 있습니다.

 위의 예제를 조금 응용해서 `@inject` 라는 데코레이터를 작성했습니다. 이 데코레이터는 "*클래스에 필요한 의존성을 클래스의 `constructor` 를 통해 주입(inject)*"하는 기능을 수행하게 했습니다.

```typescript
const dependencyPool = {
  dep1: {name: 'dep1'},
  dep2: {name: 'dep2'},
  dep3: {name: 'dep3'},
  dep4: {name: 'dep4'},
};

function inject(...depNames) {
  return function<T extends {new(...args: any[]): {}}> (constructor: T) {
    return class extends constructor {
      constructor(...args: any[]) {
        const deps = depNames.reduce((deps, name) => ({
          ...deps,
          [name]: dependencyPool[name],
        }), {});

        super(deps);
      }
    }
  }
}
```

 `@inject` 데코레이터는  `dependencyPool` 이라는 객체로부터, `depNames` 으로 넘어오는 프로퍼티 이름을 가진 디펜던시만골라서 `@inject` 데코레이터가 적용된 클래스의 생성자 함수로 넘겨주는 역할을 합니다. 

```typescript
@inject('dep1', 'dep2')
class Product {
  constructor(deps) {
    console.log('product dependency is', deps);
  }
}

function createProduct(...args) {
  return new Product(args);
}

const p = createProduct(); // product dependency is {dep1: {…}, dep2: {…}}
```

 `@inject` 를 사용하여 `dependencyPool` 에서 `Product` 클래스에 필요한 의존성인 `'dep1'`, `'dep2'` 를 추가합니다. 그러면 `Product`가 인스턴스화되는 시점에서 필요한 의존성을 주입받을 수 있습니다. 

### Method Decorator

 메서드에 적용되는 데코레이터는 클래스와 마찬가지로 메서드의 기능을 확장할 수 있습니다. 클래스 데코레이터는 클래스(생성자 함수)를 `extends` 하는 방법으로 기능을 확장할 수 있었지만, 메서드 데코레이터는 메서드의 **Property Descriptor**를 수정하여 메서드를 확장합니다.

 **Property Descriptor**는 객체의 프로퍼티들을 기존보다 정교하게 정의할 수 있는 ES5에서 처음 소개된 스펙입니다. 이 Property Descriptor는 `Object.getOwnPropertyDescriptor` 를 사용해서 가져올 수 있습니다.

```typescript
class Product {
  setPrice() {
    console.log('setPrice');
  }
}

const descriptor = Object.getOwnPropertyDescriptor(Product.prototype, 'setPrice');
console.log(descriptor); // {value: ƒ, writable: true, enumerable: false, configurable: true}
console.log(descriptor.value === Product.prototype.setPrice); // true
```

 `descriptor` 에는 `value`, `enumerable` 등의 키가 들어있는 것을 확인할 수 있습니다. 여기에서 `value` 프로퍼티는 프로퍼티의 값이나 참조를 가지고 있는데, 마지막 출력 결과를 보면 `value`는  `setPrice` 함수에 대한 참조를 가리키고 있습니다. 

> Property Descriptor는 ES5에서 처음 소개된 스펙이기 때문에 `tsconfig.json`의 `target`으로 가리키는 자바스크립트 버전이 `es5` 보다 낮게 되어있을 경우 Property Descriptor가 `undefined`으로 할당되어 사용할 수 없습니다.

 다시 메서드 데코레이터로 돌아와서, 메서드 데코레이터의 함수는 3개의 인자를 받을 수 있습니다.

1. `static` 메서드라면 클래스의 생성자 함수, 인스턴스의 메서드라면 클래스의 `prototype` 객체
2. 메서드 이름
3. 메서드의 Property Descriptor

그리고 Property Descriptor 형식의 객체를 반환할 수 있습니다. Property Descriptor 형식의 객체를 반환하지 않더라도 3번 째 인자로 넘어오는 객체를 수정하면 Property Descriptor를 반환하는 것과 같은 동작을 하게 됩니다.

예시로 작성한 `@logging` 데코레이터는 적용된 메서드에 넘어온 인자들과 메서드가 반환하는 값을 콘솔로 출력하게 됩니다. 

```typescript
function logging(target, name, descriptor) {
  const originMethod = descriptor.value;
  descriptor.value = function(...args) {
    const res = originMethod.apply(this, args);
    console.log(`${name} method arguments: `, args);
    console.log(`${name} method return: `, res);
    return res;
  }
}
```

 `@logging` 데코레이터에서는 Property Descriptor를 직접 반환하지 않고, 인자로 넘어온 `descriptor` 객체를 직접 수정하는 방식으로 메서드 동작 방법을 변경하였습니다.

```typescript
class Product {
  price: number = 1000;

  @logging
  setPrice(p: number) {
    this.price = p;

    return this.price;
  }
}

const p = new Product();
p.setPrice(1000);
// setPrice method arguments:  [1000]
// setPrice method return:  1000
```

`@logging` 데코레이터를 적용 후 `setPrice` 메서드를 호출하면 콘솔에 인자와 반환값이 기록되는 것을 확인할 수 있습니다.

### Accessor Decorator

 Accessor Decorator(접근자 데코레이터)는 `getter`, `setter` 에 적용되는 데코레이터를 말합니다. 데코레이터 함수에서는 메서드 데코레이터와 동일한 인자를 받습니다.

1. `static` 메서드라면 클래스의 생성자 함수, 인스턴스의 메서드라면 클래스의  `prototype` 객체
2. 프로퍼티 이름
3. Property Descriptor

접근자 데코레이터도 메서드 데코레이터처럼 인자로 넘어온 Property Descriptor를 변경하거나, 새로운 Property Descriptor를 반환해서 원래 접근자의 기능을 확장할 수 있습니다.

접근자 데코레이터에는 제약이 있는데, 하나의 프로퍼티에 대한 `get`, `set` 메서드에 동일한 데코레이터가 적용될 수 없습니다. 원래 자바스크립트에서는 하나의 프로퍼티가 `get`, `set`  메서드를 둘 다 가질 수 있습니다.

```typescript
class Product {
  _price: number = 1000;

  get price() {
    return this._price;
  }

  set price(p) {
    this._price = p;
  }
}

const p = new Product();
p.price = 123;
```

 그런데 타입스크립트에서는 `price` 프로퍼티에 동일한 데코레이터를 적용하려고 하면 컴파일 에러가 발생하고, 실행 결과를 보더라도 `get` 메서드에만 데코레이터가 적용된 것을 확인할 수 있습니다.

```typescript
function accessorDeco(accessorType) {
  console.log('decorator for', accessorType);
  return function(target, name, descriptor) {

  }
}

class Product {
  _price: number = 1000;

  @accessorDeco('getter')
  get price() {
    return this._price;
  }

  // Compile Error
  // Decorators cannot be applied to multiple get/set accessors of the same name.
  @accessorDeco('setter')
  set price(p) {
    this._price = p;
  }
}

const p = new Product(); // decorator for getter
```

타입스크립트 문서에서는 하나의 프로퍼티에 `get`, `set` 접근자가 모두 있고, 두 접근자에 동일한 데코레이터가 적용되었을 경우 소스코드 상에서 순서가 앞선 접근자에만 데코레이터가 적용된다고 나와있습니다. 이유는 접근자 데코레이터 또한 Property Descriptor에 적용되는데, 접근자의 Property Descriptor는 `get`, `set` 접근자를 모두 포함할 뿐 각 접근자에 대한 Property Descriptor가 없기 때문입니다.

 다음은 프로퍼티의 Property Descriptor를 변경 여부를 설정할 수 있게 해주는 `@configurable`입니다.

```typescript
function configurable(value: boolean) {
  return function (target, name, descriptor) {
      descriptor.configurable = value;
  };
}
```

 단순히 Property Descriptor의 `configurable` 속성을 변경해주는 `@configurable` 데코레이터는, 앞에서 살펴 본 메서드 데코레이터로서도 적용될 수 있습니다.

```typescript
class Product {
  _price: number = 1000;
  _count: number = 0;

  @configurable(false)
  get price() { return this._price; }
}

const p = new Product();

Object.defineProperty(Product.prototype, 'price', {
  get() {
    console.log('new price getter!');
    return this._price;
  }
});
// Uncaught TypeError: Cannot redefine property: price
```

 `Object.defineProperty` 로 `price` 의 속성을 변경하려고 하면 런타임 에러가 발생됩니다.

### Property Decorator

 클래스의 프로퍼티 선언에 사용되는 프로퍼티 데코레이터는 두 개의 인자를 받습니다.

1. `static` 으로 선언된 프로퍼티라면 클래스의 생성자 함수, 그게 아니라 인스턴스 프로퍼티람련 클래스의 `prototype` 객체
2. 프로퍼티 이름

프로퍼티 데코레이터는 메서드 데코레이터와 다르게 데코레이터 함수에 Property Descriptor 가 인자로서 제공되지 않습니다. 

프로퍼티 데코레이터는 Property Descriptor 형식의 객체를 반환해서 프로퍼티의 설정을 바꿀 수 있습니다. 두 개의 인자는 다음과 같습니다.

> 공식 문서의 프로퍼티 데코레이터를 설명하는 부분에서는 `"the return value is ignored too"`라고 되어있습니다. 이것을 보고 프로퍼티 데코레이터는 반환 값이 무시되는 줄 알았는데, 여러 예제를 살펴본 결과 프로퍼티 데코레이터에서  Property Descriptor 형식으로 객체를 반환할 때는, 프로퍼티에 정상적으로 적용되고 있었습니다. 처음에는 문서가 잘못된 것이 아닌가 생각했는데, 관련 이슈(https://github.com/microsoft/TypeScript/issues/32395)를 읽어보니 의도된 문장이라고 합니다.

 아래는 `Product`의 `name`, `price` 프로퍼티에  `@readOnly` 데코레이터를 적용한 예제입니다. `@readOnly` 데코레이터는 조건이 주어지면 해당 조건이 `true` 일 때만 프로퍼티를 read-only로 하고, 조건이 주어지지 않으면 무조건 read-only로 설정합니다.

```typescript
function readOnly(condition?: () => boolean) {
  return function decorator(target, name): any {  
    return {
      writable: condition ? condition() : true,
    }
  }
}

class Product {
  @readOnly(() => {
    return new Date > new Date(2020, 0, 1)
  })
  name: string;

  @readOnly()
  price: number;

  constructor(name: string, price: number) {
    this.name = name;
    this.price = price;
  }
}

const p1 = new Product('foo', 2000);

p1.name = 'foo';
p1.price = 3000; // Cannot assign to read only property 'price' of object '#<Product>'
```

`name` 프로퍼티는 2020년 1월 1일이 넘어가면 read-only가 되도록 처리되었고, `price` 프로퍼티는 무조건 read-only가 되어 값을 할당할 경우 에러가 발생하게 됩니다. 

### Parameter Decorator

함수의 파라미터에 사용되는 파라미터 데코레이터는 세 개의 인자를 받고, 반환값은 무시됩니다. 세 개의 인자는 다음과 같습니다.

1. static 메서드에서 사용되었다면 클래스의 생성자 함수, 인스턴스의 메서드에서 사용되었다면 `prototype` 객체
2. 파라미터 데코레이터가 적용된 메서드의 이름
3. 메서드 파라미터 목록에서의 index

아래는 `Product` 클래스의 `setPrice` 메서드에 메서드 데코레이터인 `@validate` 를 적용하고, `setPrice` 메서드의 파라미터에 파라미터 데코레이터인 `@minNumber` 를 적용한 예제입니다. 

> 공식 문서와 다른 자료를 찾아본 결과 파라미터 데코레이터는 다른 데코레이터들처럼 단독으로 사용되는 경우보다 메서드 데코레이터와 함께 사용되는 경우가 많았습니다. 예제 코드를 작성하면서도 파라미터 데코레이터는 메서드 데코레이터와 함께 사용하면 더 좋겠다는 생각이 들었습니다. 

```typescript
function minNumber(min: number) {
  return function decorator(target, name, index) {
    target.rules = {
      minNumber: function(cb) {
        const value = cb(index);
        if (value < min) {
          throw 'not valid!';
        }
      }
    }
  }
}

function validate(target, name, descriptor) {
  const originMethod = target[name];
  descriptor.value = function(...args) {
    Object.keys(target.rules).forEach(ruleName => {
      target.rules[ruleName]((index) => args[index])
    })
    originMethod.apply(this, args);
  }
}

class Product {
  name: string;
  price: number;
  purchased: number = 0;

  constructor(name: string, price: number) {
    this.name = name;
    this.price = price;
  }

  @validate
  public setPrice(@minNumber(2000) price: number) {
    this.price = price;
  }
}

const p1 = new Product('foo', 2000);
p1.setPrice(2000);
p1.setPrice(2001);
p1.setPrice(1000); // Error. not valid!
```

 `setPrice` 의 `price` 인자에는 `@minNumber` 데코레이터가 적용되었습니다. 이 데코레이터는 인자로서 전달되는 값이 최소 몇 이상이 맞는지 확인하는 용도입니다. 실행 결과를 보면 `setPrice` 에 2000, 2001을 넘겨줄 때는 잘 실행되지만, 2000보다 작은 1000을 넘겨줄 경우 에러가 발생하는 것을 확인할 수 있습니다. 