# Critical Render Path

우리가 브라우저에 URL이나 이벤트를 통해 페이지를 요청했을 때, 브라우저가 우리에게 시각화된 페이지를 보여줄 때 까지 일어나는 6단계의 과정을 `Critical Render Path`라고 한다.

![Critical Render Path](https://miro.medium.com/max/700/0*A24XmmK2IUz0wvkg.png)

위의 다이어그램은 Critical Render Path가 어떤 순서로 일어나는지 보여주는 요약본이라고 보면 된다. 우선 브라우저는 네트워크를 통해 HTML 파일을 받아온다. (TCP의 특성으로 여러번의 요청이 있을 수 있다고 한다.) 그 다음 렌더러 프로세스의 메인 스레드가 HTML 파일을 파싱하여 각 태그를 토큰화 하고 DOM트리를 구성한다. 하지만 만약 CSS와 JS가 HTML을 통해 명시 되어있다면 어떤 일이 발생할까?

## Parser Blocking & Render Blocking

브라우저는 link 태그를 통해 발견한 외부 CSS 파일을 네트워크를 통해 요청한다. 마찬가지로 CSS 파일이 다운로드 되면, 메인 스레드를 통해 파싱되어 CSSOM 트리를 구성한다.

문제는 javascript이다. javascript는 DOM과 CSSOM 조작이 가능하기 때문에 브라우저는 document.write()와 같은 최악의 상황을 고려하여 동작하게 된다.

> document.write()는 문서를 초기화 한다.

이 때문에, 브라우저는 HTML 문서를 파싱하는 도중 script 태그를 만나게 되면 이를 처리하기 위해 `파싱을 중지`한다.

![sync script load](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/images/analysis-dom-css-js.png?hl=ko)
_동기적으로 script의 리소스를 다운로드하는경우._

script 태그는 인라인으로 작성되거나 외부 js파일을 링크할 수 있다. script 태그에 아무런 옵션이 없는 경우, 브라우저는 js파일을 다운로드하고 CSSOM 트리가 만들어진 이후 js파일을 실행한뒤에야 나머지 HTML 파일을 파싱하여 DOM 트리를 만들게 된다.

script 태그의 `defer`, `async` 옵션은 이 때문에 낭비되는 시간을 줄여줄 수 있다. 우선 두 옵션 모두 HTML parser의 일을 막지 않는다. 해당 옵션을 가진 script 태그는 비동기적으로 js파일을 요청하게 된다. 차이점은 파일을 다운로드한 이후 실행되는 시점에 있다.

### defer

- `</html>`까지 파싱이 끝나고 `DOMContentLoaded` 이벤트가 발생하기 전에 실행됨
- script 태그가 여러개인 경우 순서가 보장된다. 예를들어, 크기가 큰 js파일이 먼저오고 작은 js파일이 뒤에 오는 경우 순서가 보장되기 때문에 좋은 방법이 아니게 된다. (물론 의존성을 따져봐야 하겠지만..)
- `src` 옵션이 없는 경우 `defer` 옵션은 무시된다.

### async

- `async`는 페이지와 완전히 비동기적으로 동작한다.
- 메인 스레드는 마찬가지로 파싱 작업을 멈추지 않지만, `async` 옵션으로 다운로드된 js파일은 다운로드 완료가 된 즉시 실행된다. **js가 실행되는 동안에는 파싱 작업을 멈춘다.**
- 이러한 특성 때문에 `async` 옵션으로 다운로드된 js파일의 실행은 `DOMContentLoaded` 이벤트 발생 전이나 후에 모두 가능해진다.
- 그리고 순서를 보장하지 않는다.

![async script load](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/images/analysis-dom-css-js-async.png?hl=ko)

비동기로 script를 처리하면 DOM 트리의 구성을 막지 않을 뿐더러 동시에 다른 script 태그의 외부 js파일 요청이 가능해지기 때문에 성능을 향상시킬 수 있다.

link 태그 또한 media type과 media query를 media 옵션으로 명시하면 비동기로 다운로드할 수 있다. 브라우저는 현재 media type과 media query 값을 고려하여 렌더링에 필요없는 리소스를 lazy loading하게 된다.

```html
<link href="style.css" rel="stylesheet" media="print" />
```

_해당 파일은 print시에만 필요하다고 브라우저에게 힌트를 준다._

DOM 트리와 CSSOM 트리가 완성되면 브라우저는 이 둘을 결합하여 렌더링 트리를 만든다. 렌더링 트리의 각 노드는 이후에 레이아웃을 계산하는데 사용되고, 픽셀을 화면에 렌더링하는 페인트 프로세스의 입력으로 사용되게 된다.

## Rendering Tree

![rendering tree](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/images/render-tree-construction.png?hl=ko)

렌더링 트리는 DOM 트리의 루트에서 출발하여 렌더링에 필요하지 않은 노드들을 생략하며 적합한 CSSOM 트리의 노드와 결합한다.(script 태그, meta 태그 등). 또한, CSS에 의해 렌더링 되지 않은 노드 또한 생략된다.

> ### `display: none;` vs `visibility: hidden;`
>
> `display: none;`은 렌더링 트리를 형성할때 생략되기 때문에 렌더링에 영향을 주지 않지만 `visibility: hidden;`은 요소를 보이지 않게 하지만, 렌더링 트리에 포함되기 때문에 해당 요소의 공간만큼 차지하게된다.

렌더링 트리가 완성되면 `레이아웃 단계`로 넘어갈 수 있게 된다.

## Layout

렌더링 트리의 결과로 어떤 노드가 어떤 스타일을 갖는지 알 수 있었다. 브라우저는 이제 뷰포트 내에서 요소가 어떤 크기로 어느 위치에 렌더링 되어야 하는지 알아내야 한다. 이를 위해 브라우저는 렌더링 트리의 루트에서 시작해 탐색하기 시작한다.

이 과정에서 우리가 상대적으로 부여한 스타일 값들이 절대적인 값으로 변환되어 브라우저에서 요소들을 출력할 수 있게 된다. 이렇게 계산된 값을 화면의 실제 픽셀로 변환하는 마지막 단계를 `Paint` 또는 `Raster` 단계라고 한다.

## Paint

개별 노드를 실제 픽셀로 나타내게 된다. 브라우저는 초당 60프레임의 부드러운 화면을 사용자에게 보여주기 위해서 요소들을 `Layer`라는 단위로 나누어 미리 픽셀값들을 연산해놓고 뷰포트에 보이는 부분을 `Composite(합성)` 하기도 한다. 이 과정에서 GPU를 사용하게 된다고 한다.


> 참조
> https://developers.google.com/web/fundamentals/performance/critical-rendering-path/analyzing-crp?hl=ko
> https://medium.com/comparethemarket/critical-render-path-optimisation-how-to-increase-your-page-speed-820241a4552f
