---
title: 자동화 도구 Gulp 살펴보기
date: 2019-03-31 23:12:21
tags:
---

# Gulp

## 1. 소개

Gulp는 Node.js 기반의 프로세스 자동화 도구이며 MIT 라이센스의 오픈소스 프로젝트입니다. 회사 일을 하면서 스크립트를 난독화하거나 파일을 복사하는 등의 작업이 반복될 때가 많았는데, Gulp는 이런 반복되는 작업들을 자동화하기 위해 개발된 도구입니다. 또한 Gulp의 플러그인을 사용하면 기능을 다양하게 확장할 수도 있습니다. 

> Gulp 문서 : https://gulpjs.com

## 2. Gulp를 사용한 이유

최근에는 Gulp보다는 Webpack을 사용하고 있는 것 같습니다. Webpack의 주 목적은 원래 여러 개의 파일들을 하나의 파일로 묶는 **번들러**입니다. 그리고 플러그인들을 사용하면 번들링하는 과정에서 Gulp처럼 자동화 작업도 함께 처리할 수 있습니다.

그런데 Webpack으로 파일을 묶기 위해서는 각각의 파일에는 `import`, `export`같은 ECMAScript 6의 모듈 시스템 구문을 추가해서 Webpack이 파일을 어떻게 번들링할지 알려줘야 합니다. 그리고 Webpack으로 묶여진 소스코드에는 Webpack이 부가적으로 삽입하는 코드도 포함되게 됩니다.

담당하던 프로젝트에서는 아직 ECMAScript 5를 사용하고 있었고, 자동화가 필요한 부분이 주로 파일 복사와 소스 난독화 작업이었기 때문에, 소스 번들링이 주 목적인 Webpack과는 맞지 않다고 생각해서 Gulp를 적용했습니다.

## 3. 환경 설정

Gulp 설치 방법은 공식 문서에도 잘 나와있는데, 설치 과정을 정리하면 아래와 같습니다.

### 3.1. Node, NPM, NPX 설치 확인

먼저 개발 환경에 설치된 NodeJS, NPM, NPX의 버전을 확인해야 합니다. 콘솔에서 아래의 명령어를 실행해서 버전을 확인하고 버전을 업데이트하거나 설치해야 합니다. 각 버전들은 Gulp 공식 홈페이지를 참고해서 지정했습니다.

```bash
> node --version
v8.11.1
> npm --version
5.6.0
> npx --version
9.7.1
```

### 3.2. gulp-cli 설치

```bash
> npm install --global gulp-cli
```

`gulp-cli`는 Gulp를 콘솔에서 편리하게 실행할 수 있게 해주는 유틸리티 도구입니다. 콘솔에서 위의 명령어를 입력해서 `gulp-cli`를 글로벌 영역에 설치합니다.

### 3.3 gulp 설치

Gulp는 `gulp-cli`와 다르게 전역으로 설치하지 않고, 개발 프로젝트에만 설치하면 됩니다. 먼저 `gulp`를 테스트해볼 샘플 개발 프로젝트인 `gulp-example`를 생성합니다.

```bash
> mkdir gulp-example
> cd ./gulp-example
> npm init
```

`npm init` 명령어는 'gulp-exmaple' 프로젝트에 NPM 패키지들을 설치하기 위한 초기화 작업을 합니다. `npm init` 명령어를 입력하면 콘솔에서 프로젝트 이름이나 버전 등의 정보들을 물어보는데, 모두 'enter' 키를 입력하면 디폴트값으로 세팅됩니다. 

입력이 모두 끝나면 'gulp-example' 프로젝트에는 `package.json`라는 파일이 생성되고, 이 파일을 살펴보면 `npm init`으로 입력한 정보가 들어간 것을 확인할 수 있습니다.

```bash
> npm install --save-dev gulp
```

다음으로 위의 명령어를 실행해서 Gulp를 설치합니다. `--save-dev` 옵션을 추가하는 이유는 Gulp가 개발 단계에서만 필요한 패키지이기 때문입니다. Gulp가 설치되고 나면 `package.json`의 `devDependencies`에 `gulp`가 추가되는 것을 확인할 수 있습니다.

이제 간단한 예제를 실행해보기 위해서 현재 경로에서 **gulpfile.js**를 만들고, 다음의 내용을 작성합니다.

```javascript
function defaultTask(cb) {
    cb();
}

exports.default = defaultTask;
```

그리고 콘솔에서 Gulp를 실행하고 결과를 확인합니다.

```bash
> gulp
[17:51:24] Using gulpfile ~/Workspace/gulp-example/gulpfile.js
[17:51:24] Starting 'default'...
[17:51:24] Finished 'default' after 3.62 ms
```

## 4. 태스크

Gulp에서 수행하는 작업의 단위를 **Task(태스크)**라고 부르고, 태스크는 **gulpfile.js**에 작성합니다.

3.X 버전까지는 `task()` 메서드로 태스크를 등록할 수 있었지만, 4.0 버전부터는 함수로도 태스크를 선언할 수 있게 되었습니다. Gulp가 제공하는 파일 시스템이나 Command Line을 쉽게 다루기 위한 API를 제외하고는,  그 외의 나머지 것들은 순수 자바스크립트나 Node 모듈을 이용해서 태스크를 작성할 수 있습니다.

### 4.1 실행

태스크는 외부에서 사용할 수 있는 **Public** 태스크와 내부에서 사용하는 **Private** 태스크로 나눌 수 있습니다. 태스크를 외부에서 사용하게 하려면 `exports` 객체에 태스크(함수)를 추가해야 합니다. 이렇게 내보내진 Public 태스크는 Command Line에서 실행할 수 있게 됩니다.

Private 태스크는 Command Line에서 실행할 수 없지만, gulpfile에서는 사용할 수 있습니다. 예를 들어서, 순차적으로 태스크를 실행하는 `series()` 메서드 혹은 최대한 동시에 태스크를 실행하는 `parallel()` 메서드에서도 사용할 수 있습니다.

```javascript
const { series } = require('gulp');

// Private 태스크
function clean(cb) {
  // clean 태스크 내용..
  cb();
}

// Public 태스크
function build(cb) {
  // build 태스크 내용..
  cb();
}

exports.build = build;
exports.default = series(clean, build);
```

`exports.default`로 내보내진 태스크를 실행하기 위해서는 다음의 명령어를 입력합니다.

```bash
> gulp
```

`exports.build`로 내보내진 태스크를 실행하려면 다음의 명령어를 입력합니다.

```bash
> gulp build
```

### 4.2 비동기 완료

gulpfile에서 작성하는 태스크들의 연산들은 파일의 읽기/쓰기가 대부분인데, 대부분 비동기적으로 처리됩니다. 그래서 Gulp에서는 자신의 시스템에 등록된 비동기 태스크들이 완료 시점과 성공 여부를 알 수 있어야 합니다. 그래야 태스크의 성공/실패에 따른 처리를 할 수 있기 때문입니다.

태스크가 Gulp에게 이러한 정보를 알려주는 방법은 크게 두 가지로 나눌 수 있습니다.

1. 'error-first callback'(에러-우선 콜백)을 받아서 그 콜백을 호출
2. stream, promise, event emitter, child process, observable 중 하나를 반환

'에러-우선 콜백'은 Node.js 공식 문서에서도 설명하고 있는 패턴인데, 비동기 콜백 함수에서 비동기 연산의 완료와 성공여부를 알 수 있는 방법입니다.

```javascript
// nodejs의 '에러-우선 콜백' 예제
const fs = require('fs');

function errorFirstCallback(err, data) {
  if (err) {
    console.error('There was an error', err);
    return;
  }
  console.log(data);
}

fs.readFile('/some/file/that/does-not-exist', errorFirstCallback);
fs.readFile('/some/file/that/does-exist', errorFirstCallback);
```

'에러-우선 콜백'은 첫 번째 인자로 에러 객체(위에서는 `err`)를 받는데, 이게 `null`이나 `undefined`이면 성공했다는 의미이고, 아니면 에러가 발생했다는 의미입니다.

다시 Gulp로 돌아와서, 태스크의 첫 번째 인자로는 콜백 함수가 넘어오는데, 이 콜백 함수가 **'에러-우선 콜백'** 입니다. 

```javascript
function task1(cb) {
    if (isSuccess) {
        cb(); // 에러-우선 콜백에 undefined 전달
    } else {
        cb(new Error(’some error raised’); // 에러가 발생하면, 에러 객체를 전달
    }
}

exports.task1 = task1;
```

'에러-우선 콜백'을 호출하는 경우가 아니면, 태스크는 stream, promise, event emitter, child process, observable 중 하나를 반환해야 합니다. 이 중에서 stream을 반환하는 예제는 다음과 같습니다. 

```javascript
const { src } = require('gulp');

function task1() {
	return gulp
        .src('./a.js')
    	.pipe(dest('./output/'));
}

exports.task1 = task1;
```

`src()` 와 `dest()` 메서드는 Gulp에서 제공하는 API 입니다. `src()` 메서드는 주어진 경로에 있는 파일을 읽어서 읽기 가능한 Node.js [Stream](https://github.com/FEDevelopers/tech.description/wiki/Node.js-Stream-%EB%8B%B9%EC%8B%A0%EC%9D%B4-%EC%95%8C%EC%95%84%EC%95%BC%ED%95%A0-%EB%AA%A8%EB%93%A0-%EA%B2%83)을 생성합니다.  `dest()` 메서드는 인자로 주어진 파일 경로에 파일을 쓸 수 있는 Stream을 생성합니다. 

`pipe()` 는 Node.js Stream API 중 하나입니다. `pipe()`는 쓰기 가능한 스트림을 읽기 가능한 스트림으로 연결할 수 있는 메서드이고, 역시 Stream을 반환합니다. 위의 gulpfile에서는 `task1()` 함수를 실행하면 `.pipe`에 의해 Stream을 반환하기 때문에, Gulp에서 태스크 완료나 성공 여부를 알 수 있습니다.

### 4.3 Stream에 파일 추가하기

`src()  ` 메서드는 파이프라인의 중간에도 위치해서 기존의 Stream에 파일을 추가할 수도 있습니다. 예를 들어서, 모든 소스코드를 난독화하기 전에, 특정 파일을 babel로 트랜스파일하는 것과 같은 작업을 할 수 있습니다.

```javascript
const { src, dest } = require('gulp');
const uglify = require('gulp-uglify');
const babel = require('gulp-babel');

function minifyCode() {
  return src('./src/*')
    .pipe(babel({
      presets:['@babel/preset-env']
    }))
    .pipe(src('./vendor/*'))
    .pipe(uglify())
    .pipe(dest('./output'));
}

exports.default = minifyCode;
```

### 4.4 현재 Stream 출력하기

`dest()` 메서드도 파이프라인의 중간에 위치할 수 있습니다. `dest()` 는 현재의 Stream을 주어진 경로에 파일을 출력하는 역할을 합니다. 소스를 난독화 할 때, 난독화 하지 않은 원본 소스도 함께 출력하고 싶을 때 유용합니다.

```javascript
const { src, dest } = require('gulp');
const uglify = require('gulp-uglify');
const babel = require('gulp-babel');
const rename = require('gulp-rename');

function minifyCode() {
  return src('src/*.js')
    .pipe(babel({
      presets: ['@babel/preset-env']
    }))
    .pipe(src('vendor/*.js'))
    .pipe(dest('output/'))
    .pipe(uglify())
    .pipe(rename({ extname: '.min.js' }))
    .pipe(dest('output/'));
}

exports.default = minifyCode;
```

위에서 `minifyCode`라는 태스크가 처리하는 작업들을 순서대로 작성해보면 다음과 같습니다.

1. 'src/' 디렉토리에 있는 모든 'js' 파일을 찾아서 Stream에 추가한다.
2. 현재 Stream의 파일들을 babel로 트랜스파일한다.
3. 'vendor/' 디렉토리에 있는 모든 'js' 파일을 찾아서 Stream에 추가한다.
4. **현재 Stream에 있는 파일들을 'output/' 디렉토리에 출력한다.** 
5. 현재 Stream에 있는 파일들에 대해서 ugilify로 난독화한다.
6. 현재 Stream에 있는 파일들에 대해서 확장자의 이름을 '.min.js'로 수정한다.
7. **현재 Stream에 있는 파일들을 'output/' 디렉토리에 출력한다.**

파일 구조가 어떻게 변했는지 살펴보기 위해, `minifyCode` 태스크가 실행되기 전에는 파일 구조가 다음과 같았다면,

```text
gulp-example/
|- src/
   |- app.js
|- vendor/
   |- jquery.js
|- output/
```

Gulp가 `minifyCode` 태스크를 실행하고 난 다음에는 파일 구조가 다음과 같을 것입니다.

```text
gulp-example/
|- src/
   |- app.js
|- vendor/
   |- jquery.js
|- output/
   |- app.js
   |- app.min.js
   |- jquery.js
   |- jquery.min.js
```

## 5. 참고하면 좋은 자료

- Gulp 4.0 달라진 것들 : https://programmingsummaries.tistory.com/393
- Gulp 공식 사이트 : https://gulpjs.com
- Gulp 주요 개념들 : https://gulpjs.com/docs/en/api/concepts
- 프론트엔드 개발을 위한 Gulp https://github.com/FEDevelopers/tech.description/wiki/%ED%94%84%EB%A1%A0%ED%8A%B8%EC%97%94%EB%93%9C-%EA%B0%9C%EB%B0%9C%EC%9D%84-%EC%9C%84%ED%95%9C-Gulp