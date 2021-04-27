# CJS, AMD, UMD, ESM

웹 개발을 할 때 javascript로 처리해야하는 로직이 많아지면서 작성한 코드를 모듈화 하려는 움직임이 생겨났다. 원래 javascript에서는 모듈화의 개념이 없었기 때문에 여러가지 방법의 모듈 명세가 등장하게 되었다. 이것이 오늘의 주제인 `CJS(CommmonJS)`, `AMD(Asynchronous Module Definition)`, `UMD(Universal Module Definition)`, `ESM(ES Module)`에 대한 이야기이다.

## CJS (CommonJS)

[CommonJS](http://www.commonjs.org/)는 이름에서도 알 수 있듯, javacript를 브라우져 뿐만 아니라 범용 언어로 사용하기 위한 움직임에서 탄생하였다. 기본적인 문법을 살펴보자.

```javascript
let module require('./other') // 모듈 불러오기

module.something() // 모듈 사용하기

module.exports = { // 모듈 내보내기
  name: "CommonJS"
  something() {
    console.log("hello world")
  }
}
```

### 특징

- Node.js 모듈 시스템에 사용된다.
- 모듈을 동기적으로 불러온다.
- Backend 개발을 목적으로 탄생하였다.

## AMD (Asynchronous Module Definition)

CommonJS와 달리 AMD는 비동기적 상황에서도 모듈화를 사용하기 위해 만들어 지게 되었다. CommonJS가 사용되는 로컬머신에서는 사용하는 모든 모듈이 이미 로컬디스크에 존재하기 때문에 동기적으로 바로 불러오는데 문제가 없지만, 브라우져에서는 필요한 모듈을 다운로드해야 하기 때문에 비동기적으로 모듈을 불러와야 하는 상황이 발생한다. 이렇게 CommonJS와 다른 환경에서 javascript를 모듈화 하기 위해 탄생하였고, 합의점을 찾지 못해 독립된 명세라고 한다.

```javascript
// 사용할 모듈을 첫번째 인자에 넣으면, callback 함수를 통해 사용할 수 있게된다.
define(['other/lib'], function (lib) {
  function something() {
    // 모듈 사용.
    lib.something();
  }
  // 새롭게 생성한 something 함수를 내보내기.
  return {
    something,
  };
});
```

### 특징

- 모듈을 비동기적으로 불러온다.
- 브라우저에서 모듈을 사용하기 위해 등장하였다.

## UMD (Universal Module Definition)

AMD와 CommonJS는 두 그룹으로 나뉘어져 서로 호환되지 않는 문제가 발생한다. 이러한 문제를 해결하기 위해 등장한 명세가 UMD이다. 사실 UMD는 두 모듈화 방식을 모두 사용할 수 있게하는 디자인 패턴이다. 아래 예시는 그 중 하나인[returnExports.js](https://github.com/umdjs/umd/blob/master/templates/returnExports.js) 패턴이다.

```javascript
(function (root, factory) {
  if (typeof define === 'function' && define.amd) {
    // AMD
    define(['exports', 'b'], factory);
  } else if (
    typeof exports === 'object' &&
    typeof exports.nodeName !== 'string'
  ) {
    // CommonJS
    factory(exports, require('b'));
  } else {
    // Browser globals
    factory((root.commonJsStrict = {}), root.b);
  }
})(this, function (exports, b) {
  //use b in some fashion.

  // attach properties to the exports object to define
  // the exported module properties.
  exports.action = function () {};
});
```

> b 는 임의의 모듈을 의미한다.

우선 등장한 즉시실행함수(IIFE)는 코드가 실행되는 컨택스트에서 `exports`와 `module`이 존재하면 CommonJS 방식으로, `define`이 함수이고 `define.amd`가 존재하면 AMD 방식으로, 그것도 아니라면 window 객체에 모듈을 내보낸다.

### 특징

- AMD와 CommonJS를 모두 사용할 수 있다는 특징이 있기 때문에, 클라이언트 사이드와 서버사이드의 모듈을 따로 만들 필요가 없어진다.
- 이러한 이유로 두 모듈 방식을 모두 제공해야 하는 라이브러리에서 사용하게 된다.

## ESM (ECMAScript Module)

ES6부터 공식적으로 지원하는 모듈화 방법으로, CJS와 AMD의 장점을 차용하여
만들어졌다. 예제 코드는 다음과 같다. (가장 익숙함..)

```javascript
import lib from 'others';

function something() {
  lib.something();
}

export { something };
```

### 특징

- CJS의 가독성과 AMD의 비동기적 모듈 임포트를 차용하여 만들어졌다.
- 최신 브라우져에서 대부분 지원한다. (IE 제외)
- script 태그 속성으로 type="module" 값을 지정해주면 브라우져에서도 모듈을 사용할 수 있다.
- Tree-Shaking이 가능하다. (모듈에서 사용하는 부분만 추적하여 번들링하는 최적화 기법)

```html
<!-- HTML -->
<script type="module" src="index.js">
```
