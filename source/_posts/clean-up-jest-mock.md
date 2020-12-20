---
title: Jest에서 Mock을 정리하는 방법
date: 2020-12-21 01:05:23
tags:
  - frontend
  - test
  - jest
  - mock
---

## 소개

테스트에서는 Mock을 테스트 대역(Test Double)으로 부른다. **테스트 대역**은 구현 코드를 테스트하는데 필요한 것(객체, 함수, 데이터 등)들을 **테스트를 실행하는 동안** 대신하는 요소들을 말한다. 테스트 대역이 구현 코드를 잠깐 대신한다는 점에서 영화 촬영할 때 액션 연기를 대신해주는 스턴트 배우와 비슷하다.

### 왜 정리해야 하는가?

다음 테스트 케이스를 실행하기 전에는 현재 테스트 케이스에서 사용했던 Mock을 정리해주는 것이 좋다. 다음 테스트 케이스에 영향을 줄 수도 있기 때문이다. 예를 들어 `console.log` 의 테스트 대역을 만들기 위해`jest.spyOn`을 사용한 이후에는 `console.log` 는 다른 함수가 될 수도 있다.

아래 테스트가 통과하는 것으로 위의 내용을 검증해볼 수 있다.

```javascript
const consoleLog = console.log;
test("spyOn으로 console.log를 mocking하면, console.log는 다른 함수가 된다.", () => {
  jest.spyOn(console, "log");
  const consoleLogAfterMocking = console.log;

  // 결과는 Success.
  // console.log는 spyOn으로 mocking한 이후 다른 함수가 된다.
  expect(consoleLog).not.toBe(consoleLogAfterMocking);
});
```

물론 `spyOn`은 테스트 대역을 만들 때 유용하게 사용될 수 있지만, 다른 테스트 케이스 입장에서는 사이드 이펙트를 일으킨 주범이 될 수 있다. 따라서 모든 테스트 케이스가 독립적으로 실행되게 하기 위해선 mock을 정리하는 것이 좋다.

### 어떻게 정리하는가?

Jest에서 mock을 정리할 수 있는 방법은 크게 두 가지가 있다.

- 테스트 코드에서 수동으로 mock을 정리하기
- Jest의 어떤 설정을 활성화해서 mock이 자동으로 정리되게 하기

## 수동으로 정리하기

Jest에서 테스트 코드에서 수동으로 mock을 정리하는 방법에도 여러 가지가 있다.

- `mockFn.mockClear`
- `mockFn.mockReset`
- `mockFn.mockRestore`

`mockFn` 은 Jest에서 생성한 mock 함수를 말한다. 참고로 mock 함수를 만드는 방법에도 여러 가지(...)가 있는데, 대표적으로 `jest.fn`, `jest.spyOn`이 있다.

혹은 `jest` 객체에 있는 메서드로 정리할 수도 있다.

- `jest.clearMocks`
- `jest.resetMocks`
- `jest.restoreMocks`

### mockFn.mockClear

`mockFn.mock.calls`와 `mockFn.mock.instances` 배열을 초기화한다. 다음 테스트 케이스를 실행하기 전에 mock 함수를 호출했던 정보를 비우고 싶을 때 유용하다.

> `mockFn.mock.calls` 에는 mock 함수가 함수로 호출됐을 때의 매개변수의 목록이 있고, `mockFn.mock.instances` 는 mock 함수가 생성자로 호출됐을 때 생성했던 인스턴스의 목록이 있다.

예를 들어서 `jest.fn` 으로 mock 함수를 생성하고 `mockFn.mockClear` 로 `mockFn.mock.calls` 와 `mockFn.mock.instances` 를 초기화한다는 내용은 다음과 같은 테스트 코드로 확인해볼 수 있다.

```javascript
test("mock 함수를 호출한 후 mockClear를 호출하면, mock.calls는 초기화된다.", () => {
  const mockFn = jest.fn();
  mockFn("1");
  mockFn("1", "2");
  expect(mockFn.mock.calls[0]).toEqual(["1"]);
  expect(mockFn.mock.calls[1]).toEqual(["1", "2"]);
  expect(mockFn.mock.calls).toHaveLength(2);

  mockFn.mockClear();

  expect(mockFn.mock.calls).toHaveLength(0);
});

test("mock 생성자를 호출한 후 mockClear를 호출하면, mock.instances는 초기화된다.", () => {
  const MockConstructor = jest.fn();
  const a = new MockConstructor();
  const b = new MockConstructor();
  expect(MockConstructor.mock.instances).toHaveLength(2);
  expect(MockConstructor.mock.instances[0]).toBe(a);
  expect(MockConstructor.mock.instances[1]).toBe(b);

  MockConstructor.mockClear();

  expect(MockConstructor.mock.instances).toHaveLength(0);
});
```

### mockFn.mockReset

이 함수는 `mockFn.mockClear()` 함수가 하는 일을 모두 할 수 있다. 이것에 더해 `mockFn.mockReset`은 mock 함수의 구현(ex. `jest.fn()` 에 넘기는 함수)을 `undefined` 을 반환하는 빈 함수로 초기화한다.

```javascript
test("mock 함수를 호출한 후 mockReset을 호출하면, mock 함수는 undefined을 반환하는 함수가 된다.", () => {
  const mockAdd = jest.fn((a, b) => a + b);
  expect(mockAdd(1, 2)).toBe(3);
  expect(mockAdd.getMockImplementation()).toBe(add);

  mockAdd.mockReset();

  expect(mockAdd(1, 2)).toBe(undefined);
  expect(mockAdd.getMockImplementation()).toBe(undefined);
});
```

### mockFn.mockRestore

이 함수도 `mockReset`이 그랬던 것처럼 `mockReset` 함수가 하는 일을 모두 할 수 있다. 이것에 더해 `mockFn.mockRestore`는 mocking 하면서 오염된(?) 함수를 다시 원래대로 되돌릴 수 있다. 말로 설명하면 복잡하니, `mockRestore`가 필요한 이유를 코드로 살펴보자.

```javascript
const someModule = { api: () => "origin" };
test("spyOn으로 테스트 더블을 만든 뒤에, someModule.api는 다른 함수가 된다.", () => {
  const originApi = someModule.api;
  const mockApi = jest.spyOn(someModule, "api");

  const changedApi = someModule.api;
  expect(originApi).not.toBe(changedApi);
  expect(changedApi()).toBe("mock");
});
```

`jest.spyOn` 은 객체의 메서드를 테스트 대역으로 사용하고 싶을 때 유용하다. 하지만 이 테스트 케이스를 실행하고 나면 `console.log` 는 다른 함수가 된다. 정리해주지 않으면 다음 테스트 케이스에서 의도하지 않은 결과가 나타날 수도 있다. 만약 객체의 메서드를 mocking했다면 `mockRestore`를 호출하여 원래대로 되돌려줘야 한다.

```javascript
const someModule = { api: () => "origin" };
test("spyOn으로 테스트 더블을 만든 뒤에 mockRestore를 호출하면, someModule.api는 원래대로 돌아온다.", () => {
  const originApi = someModule.api;
  const mockApi = jest.spyOn(someModule, "api");

  mockApi.mockRestore();
  const changedApi = someModule.api;

  expect(originApi).toBe(changedApi);
  expect(changedApi()).toBe("origin");
});
```

주의해야 할 것은 `jest.fn` 으로 만들어진 mock 함수에서는 `mockFn.mockRestore` 가 동작하지 않는 점이다. 따라서 어떤 객체의 메서드를 테스트 대역으로 사용하고 싶을 때는 `jest.spyOn`을 사용하는 것이 더 편리하고 안전하다.

### clear vs reset vs restore

mock 함수를 정리하는 방법은 `mockClear`, `mockReset`, `mockRestore` 순으로 강력(?)하다. 이는 Jest 소스코드를 살펴봐도 알 수 있다. `mockRestore`는 `mockReset`을 호출하고, `mockReset`은 `mockClear`를 호출한다.

```javascript
// jest-mock/src/index.ts
class ModuleMocker {
  // ...
  _makeComponent(metadata, restore) {
    // ...
    const f = this._createMockFunction(metaData, mockConstructor);

    f.mockClear = () => {
      this._mockState.delete(f);
      return f;
    };

    f.mockReset = () => {
      f.mockClear();
      this._mockConfigRegistry.delete(f);
      return f;
    };

    f.mockRestore = () => {
      f.mockReset();
      return restore ? restore() : undefined;
    };
    // ...
  }
}
```

지금까지는 `mockFn` 으로 직접 mock 함수를 정리하는 방법을 살펴봤다. 다음으로는 `jest` 객체에서 mock을 정리하는 방법을 살펴보자.

### `jest`로 mock 정리하기

Jest에서 제공하는 객체인 `jest`를 활용하면 `mockFn` 에서 일일이 mock 정리하는 번거로움을 조금 줄일 수 있다.

- `jest.clearAllMocks` : 모든 mock 함수에서 `mockFn.clearAllMocks` 을 호출하는 것과 동일하다.
- `jest.resetAllMocks` : 모든 mock 함수에서 `mockFn.resetAllMocks` 을 호출하는 것과 동일하다.
- `jest.restoreAllMocks` : 모든 mock 함수에서 `mockFn.restoreAllMocks` 을 호출하는 것과 동일하다.

## 자동으로 정리하기

확실히 `jest.clearAllMocks`, `jest.resetAllMocks`, `jest.restoreAllMocks` 를 호출하는 것이 `mockFn` 의 메서드를 직접 호출하는 것보다는 쉬운 방법이다. 하지만 이 방법도 `afterEach`, `beforeEach` 와 같은 방법으로 테스트 케이스가 실행되기 전이나 후에 호출해줘야 하는 번거로움이 있다.

```javascript
beforeEach(() => {
  jest.restoreAllMocks();
});

test("테스트 케이스 1", () => {
  // ...
});

test("테스트 케이스 2", () => {
  // ...
});
```

이것보다 더 쉬운 방법은 `jest.config.js` 설정값을 변경하는 것이다. 설정을 수정하면 테스트 케이스를 실행하기 전에 알아서 mock이 정리되게 할 수 있다.

- `clearMocks` (default `false`): `jest.clearAllMocks` 를 각 테스트 케이스를 실행하기 전에 호출하는 것과 동일하다.
- `resetMocks` (default `false`): `jest.restAllMocks` 를 각 테스트 케이스를 실행하기 전에 호출하는 것과 동일하다.
- `restoreMocks` (default `false`): `jest.restoreAllMocks` 를 각 테스트 케이스를 실행하기 전에 호출하는 것과 동일하다.

앞에서 설명한 것처럼 `restore` 작업에는 `reset`, `clear` 가 포함되어 있다. 따라서 편하게 mock을 정리하고 싶다면 `jest.config.js`에서는 `restoreMocks`만 활성화해도 된다.

```javascript
module.exports = {
  restoreMocks: true,
};
```

## 참고

- Jest : [https://jestjs.io/docs/en/jest-object,](https://jestjs.io/docs/en/jest-object) [https://jestjs.io/docs/en/mock-function-api](https://jestjs.io/docs/en/mock-function-api)
- Github Issue : [mockClear vs mockReset vs mockRestore]()
