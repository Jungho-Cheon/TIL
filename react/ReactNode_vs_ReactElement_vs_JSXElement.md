# ReactNode vs ReactElement vs JSX.Element

## ReactElement

Typescript에서 함수형 컴포넌트를 생성할 때, 리턴 타입으로 사용할 수 있는 타입이나 인터페이스가 어떤게있는지 조사하였다. 우선 함수형 컴포넌트의 타입은 `React.FC`로 `React.FunctionComponent` 인터페이스를 줄여 alias로 존재한다. 인터페이스의 구조는 다음과 같다.

```typescript
interface FunctionComponent<P = {}> {
  (props: PropsWithChildren<P>, context?: any): ReactElement<any, any> | null;
  propTypes?: WeakValidationMap<P>;
  contextTypes?: ValidationMap<any>;
  defaultProps?: Partial<P>;
  displayName?: string;
}
```

가장 위에 정의된 함수로 보아 함수형 컴포넌트의 리턴 타입은 `ReactElement<any, any> | null`인 것을 알 수 있다. 비교하기로 했었던 3가지 중 하나가 등장하였다. 일단 정의된 바에 따르면 기본적으로는 `ReactElement<any, any>`를 리턴하도록 되어있지만 다른 사람들이 작성한 코드에서 리턴타입으로 `JSX.Element`로 작성한 것을 다수 발견할 수 있었다. `JSX.Element`는 무엇이며 왜 존재하는지 궁금해졌다.

## JSX.Element

JSX는 React에서 제공하는 `createElement()`를 대체하여 html과 같은 구성으로 코드를 작성하기 위한 Syntactic Sugar이다. 이런 편의를 제공하기 위해 global namespace에 정의되어져 있는 것이 JSX이다. 정의되어있는 부분을 찾아보았다.

```typescript
declare global {
    namespace JSX {
        interface Element extends React.ReactElement<any, any> { }

        ...
    }
}
```

`JSX.Element`로 사용하고 있었던 것은 다름아닌 `React.ReactElement<any, any>`를 extends한 인터페이스인 것을 알 수 있다. 음 그렇다면 `JSX.Element`를 리턴타입으로 지정하였을 때와 차이는 `null`을 리턴할 수 있느냐가 있게된다.

## ReactNode

`ReactNode`는 클래스 컴포넌트의 `render()` 메소드의 리턴타입이다. 위의 둘은 확장의 개념으로 연관되어있어 보이는 것을 확인했지만 `ReactNode`는 왜 따로 존재하는 것인지 정의를 살펴보기로 하였다.

```typescript
//
// React Nodes
// http://facebook.github.io/react/docs/glossary.html
// ----------------------------------------------------------------------

type ReactText = string | number;
type ReactChild = ReactElement | ReactText;

interface ReactNodeArray extends Array<ReactNode> {}
type ReactFragment = {} | ReactNodeArray;
type ReactNode =
  | ReactChild
  | ReactFragment
  | ReactPortal
  | boolean
  | null
  | undefined;
```

여기에도 다시 한번 `ReactElement`가 등장한다. ReactNode는 `render()`메소드가 리턴할 수 있는 타입들을 정의하는 더 넓은 의미로 사용되는 듯 하다. 눈에 띄는 차이는 boolean, null, undefined같은 프리미티브 타입을 반환할 수 있도록 정의 된 것이다.

> _그렇다면 왜 함수형 컴포넌트의 리턴 타입으로 ReactNode를 사용하지 않을까?_
> historical한 문제로 둘을 리턴하는 곳에 따라 분리해서 사용한다고 한다.
> 자세한 내용은 [여기](https://stackoverflow.com/questions/58123398/when-to-use-jsx-element-vs-reactnode-vs-reactelement/59840095#59840095)를 참고해보자..

### 결론

함수형 컴포넌트를 만들 때 가끔 조건부 렌더링이 필요한 경우가 있다. 그동안 `JSX.Element`를 리턴 타입으로 설정해 `if(!state) return <></>;`와 같은 코드를 자주 사용하곤 했다. 함수형 컴포넌트에서도 `null`을 반환해도 되는 것을 확인한 셈이니 더 깔끔한 코드를 위해 사용해보도록 해야겠다.
