# Code Splitting

webpack은 모듈 번들러로써 우리가 작성한 javascript 파일들이나 css 파일들을 하나의 파일로 번들링 하는 등 다양한 기능을 제공한다. 더 자세한 학습을 위해 공식문서를 읽는 도중 `chunk`라는 단어와 함께 코드를 다시 분리시키는 것을 소개하고 있었다. 어째서 이런 작업을 하는 것일까..

몇 가지 예시 중 페이스북을 예로 든 글이 있었다.

> 만약 브라우저를 통해 페이스북에 접속 했을 때, 서비스를 위한 모든 파일을 요청해야 한다면 첫 화면을 볼 때 까지 엄청난 시간이 걸릴 것이다.

SPA의 특성상 작업량이 많아질수록 하나로 번들링된 javascript파일은 거대해진다. 때문에 webpack은 모듈에서 사용하는 외부 의존성들을 `vender`라는 이름으로 따로 번들링하기도 하고, 같은 외부 의존성들에 대해 중복이 생기지 않도록 설정파일을 수정할 수 있다. 또한, 사용자가 특정 기능을 사용할 때 해당 부분에 대한 파일 요청할 수 있도록 `Code Splitting`라는 기술을 제공한다. 이렇게 분리된 번들을 `chunk`라고 부른다.

우선 하나의 파일로 번들링했을 때 `webpack-bundle-analyzer` 플러그인을 이용해 번들을 시각화하면 다음과 같다.
![](https://miro.medium.com/max/700/1*Tzo7ki8deVX0ADRFCm1E7Q.png)
_출처 https://angular-evan.medium.com/using-webpack-bundle-analyzer-to-cure-identify-your-bundle-bloat-57477678ac7b_

bundle.js에 모든 파일들이 하나로 번들링 된 것을 알 수 있다. 웹팩 공식문서에 따르면 동적인 `Code Splitting`을 사용하기 위해 Dynamic Imports를 지원하며, 이를 위한 문법으로 ECMAScript을 준수하는 `import()`와 기존 레거시에 대한 접근으로는 `require.ensure()`를 사용할 수 있다고 한다. 해당 내용은 [React.js 도큐먼트](https://reactjs.org/docs/code-splitting.html#import)에서도 소개하고 있다. CRA와 같은 프로젝트 생성툴을 통해 프로젝트를 생성하게 되면 내부적으로 설정된 webpack을 통해 react.js와 next.js에서도 사용할 수 있다.

> Babel을 사용하는 경우 Dynamic Imports에 대한 설정을 확인해야 한다.
> 프로젝트에`@babel/plugin-syntax-dynamic-import` 패키지가 없다면 Babel은 해당 문법을 파싱만 할 뿐 해석하지 않는다고 한다.

webpack에서 제공하는 `import()`를 사용한 Code Splitting 예제를 살펴보자.

```javascript
function getComponent() {
  const element = document.createElement('div');
  return import('lodash')
    .then(({ default: _ }) => {
      const element = document.createElement('div');

      element.innerHTML = _.join(['Hello', 'webpack'], ' ');

      return element;
    })
    .catch(error => 'An error occurred while loading the component');
}

getComponent().then(component => {
  document.body.appendChild(component);
});
```

- Dynamic Import에 성공한 경우 default 키로 import한 패키지를 사용할 수 있다. 결과적으로 위의 예제는 외부라이브러리인 lodash를 Dynamic Import하여 컴포넌트의 innerHTML을 초기화하여 반환해준다.

- `getComponent`를 통해 알 수 있듯 `import()`는 Promise를 생성한다. 때문에 async/await로 작성하여 가독성을 늘리는 방법도 있다.

위와 같이 작성한 뒤 webpack을 통해 빌드하고 결과를 확인해보면

```bash
[webpack-cli] Compilation finished
asset vendors-node_modules_lodash_lodash_js.bundle.js 549 KiB [compared for emit] (id hint: vendors)
asset index.bundle.js 13.5 KiB [compared for emit] (name: index)
runtime modules 7.37 KiB 11 modules
cacheable modules 530 KiB
  ./src/index.js 434 bytes [built] [code generated]
  ./node_modules/lodash/lodash.js 530 KiB [built] [code generated]
webpack 5.4.0 compiled successfully in 268 ms
```

둘째 줄을 통해 lodash 라이브러리가 vendor 번들로 분리되어진 것을 확인할 수 있다.

이렇게 분리된 번들은 사용자의 앱에서 필요할 때 `지연 로딩`되어 성능 향상과 함께
초기화에 대한 비용을 획기적으로 줄여 준다. Code Splitting을 이용하여 페이지를 구성하는 컴포넌트와 외부 패키지를 Dynamic Import하게 되면 최종적으로 만들어지는 번들의 상태는 다음과 같아질 것이다.
![](https://cloud.githubusercontent.com/assets/302213/20628702/93f72404-b338-11e6-92d4-9a365550a701.gif)
_출처 https://www.npmjs.com/package/webpack-bundle-analyzer_

> 참고 사진은 각각 다른 프로젝트로 예시를 위해 스크랩함.
