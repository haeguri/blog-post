---
title: Jest Mock 타이머 적용하기
date: 2020-01-12 17:04:39
tags:
  - frontend
  - test
  - jest
---

자바스크립트의 네이티브 타이머 함수들은 실제 시간에 의존하기 때문에 테스트 환경에는 적합하지 않다. 예를 들어 n분 뒤에 실행되는 로직에 대한 테스트 결과를 실제로 n분이 지난 후에 알 수 있다면 테스트 실행 시간이 너무 길어질 것이다. 테스트 환경에서는 네이티브 타이머를 Mock 타이머로 대체해서 n분 뒤에 실행되는 로직도 바로 실행해볼 수 있어야 한다.

Mock 타이머를 제공하는 다른 테스트 라이브러리도 있지만, Jest에서도 기본적으로 Mock 타이머를 제공하고 있다. 이 글에서는 Jest에서 제공하는 Mock 타이머로 타이머에 의존하는 로직을 테스트하는 방법을 다룬다.

## 네이티브 타이머로 테스트

먼저 다음의 코드가 있다고 가정해보자. 테트리스가 시작되면 1초에 1번 블록을 아래로 이동시키도록 구현된 코드다.

```javascript
class Tetris {
  constructor() {
    this.timerId = undefined;
  }

  start() {
    setInterval(this.tick, 1000);
  }

  tick() {
    this.moveDown();
  }

  moveDown() {
    // 테트리스 블록을 아래로 한 칸 이동시킨다.
  }
}
```

그리고 만약 Mock 타이머 없이 테스트 코드를 작성한다면 다음과 같이 작성해야 할 것이다.

```javascript
describe("Tetris", () => {
  const sleep = seconds => new Promise(res => setTimeout(() => res(), seconds * 1000));

  describe("start", () => {
    it("1초 뒤에 블록을 1번 내린다.", async () => {
      const tetris = new Tetris();
      const moveDown = jest.spyOn(tetris, "moveDown");

      tetris.start();

      await sleep(1);

      expect(moveDown).toBeCalledTimes(1);
    });

    it(“3초 뒤에 블락을 3번 내린다.", async () => {
      const tetris = new Tetris();
      const moveDown = jest.spyOn(tetris, "moveDown");

      tetris.start();

      await sleep(3);

      expect(moveDown).toBeCalledTimes(3);
    });
  });
});
```

`start`가 실행하는 네이티브 타이머의 결과를 테스트하기 위해서는 실제로 n초가 지나가는 것을 기다려야 한다. 따라서 `sleep` 함수로 n초 동안 테스트 코드의 실행을 blocking하는 방법을 적용했다.

이 테스트의 **첫 번째** 문제는 테스트가 부정확하다는 것이다. 실행해본 결과, 첫 번째 케이스는 통과하는데, 두 번째 케이스는 실패했다.

```shell
[FAIL] Tetris › start › 3초 뒤에 블락을 3번 내린다.

expect(jest.fn()).toBeCalledTimes(expected)

Expected number of calls: 3
Received number of calls: 2
```

테스트 코드에서는 3초 뒤면 `moveDown`이 3번 호출되길 기대했지만 결과는 2번 호출된다. 반면 첫 번째 케이스는 성공하고 있는데, 이렇게 되면 구현 코드의 어디가 문제인지 알아내기 힘들다.

[타이머 함수들의 스펙](https://html.spec.whatwg.org/multipage/timers-and-user-prompts.html#timers)을 살펴보면, 인자로 전달되는 timeout은 실제로는 그 timeout 만큼 뒤에 interval 함수가 실행되는 것을 보장하지 않는다. `sleep` 함수로 3000ms 동안 코드 실행이 blocking되는 것을 기대하지만, 실제로는 정확히 3000ms만큼 blocking되지 않는다는 의미다.

**두 번째** 문제는 테스트 실행 시간이 느려진다는 것이다. `sleep`으로 테스트를 blocking하는 코드가 더 추가된다면, 개발자가 코드를 수정하고 테스트 실행 결과를 확인하기까지 수십초가 걸릴 수도 있다. 테스트가 프로덕트를 배포하는 과정에서도 실행된다면 배포 시간도 덩달아 느려진다.

## 개선하기

네이티브 타이머 대신 Jest의 Mock 타이머를 사용하도록 테스트 코드를 수정해보자.

```javascript
jest.useFakeTimers(); // 네이티브 타이머를 Jest 타이머로 대체

describe("Tetris", () => {
  afterEach(() => {
    jest.clearAllTimers();
  });

  describe("start", () => {
    it("1초 뒤에 블록을 1번 내린다.", () => {
      const tetris = new Tetris();
      const moveDown = jest.spyOn(tetris, "moveDown");

      tetris.start();

      jest.advanceTimersByTime(1000); // 이후 1000ms 뒤에 실행되어야 하는 로직이 실행된다.

      expect(moveDown).toBeCalledTimes(1);
    });

    it("3초 뒤에 블락을 3번 내린다.", () => {
      const tetris = new Tetris();
      const moveDown = jest.spyOn(tetris, "moveDown");

      tetris.start();
      jest.advanceTimersByTime(3000); // 이후 3000ms 뒤에 실행되어야 하는 로직이 실행된다.

      expect(moveDown).toBeCalledTimes(3);
    });
  });
});
```

`jest.useFakeTimers()`를 실행하면 `setTimeout`, `setInterval`, `clearTimeout`, `clearInterval` 등의 네이티브 함수들이 Jest 타이머로 대체된다. 즉, 테스트 코드로 실행되는 타이머가 Jest 타이머가 된다.

`jest.advanceTimersByTime(ms)`은 모든 타이머가 `ms` 밀리세컨드만큼 시간이 지나가게 한다. 이 코드를 통해 `ms` 밀리세컨드 후에 실행되어야 하는 콜백 함수들이 실행되는 것을 보장할 수 있다.

마지막으로 `jest.clearAllTimers()`로 테스트 케이스가 실행된 후 Mock 타이머를 정리해서 테스트가 독립적으로 실행되는 것을 보장해줄 수 있다.

테스트를 실행해보면 테스트의 실행 시간도 훨씬 짧아지고, 작성한 테스트 케이스가 모두 성공하는 것을 확인할 수 있다.
