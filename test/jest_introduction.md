# Jest 입문하기

스프링을 공부할 때 처음으로 JUnit을 사용한 테스트를 경험한 뒤 한동안 프론트 엔드에 관심이 생겨 관련된 공부를 하느라 테스트 코드 작성에 대해 무관심해졌습니다. 테스트 코드 작성은 자신의 코드에 대해 한번 더 생각해볼 수 있을 뿐더러 기능을 증명하는 코드를 추가적으로 작성하여 로직을 보장할 수 있는 과정으로 생각합니다. 프론트엔드 개발을 할 때, 매번 변경되는 화면을 확인하면서 개발할 수 있겠지만 TDD를 적용하여 테스트 코드를 작성하면서 개발하면 브라우져 없이 테스트 코드가 잘 작동하는 지만 확인하면서 개발할 수 있습니다. (물론 스타일링은 별개의 이야기인 것으로..)

## TDD로 Todo List 개발하기

새로운 라이브러리를 적용할 때 hello world와도 같은 Todo List를 만들어 보면서 튜토리얼을 진행해 보겠습니다. CRA를 통해 React 프로젝트를 생성하면 내부적으로 jest가 설치, 설정되어있습니다.

> 내부 설정이 궁금하면 eject로 한번 확인해보면 좋을 듯 합니다. CRA에서 src폴더 내의 test파일을 찾고, test파일 내에서 Import 구문 없이 Jest의 메서드를 사용할 수 있는 이유도 알아볼 수 있습니다.

CRA를 통해 프로젝트를 생성한 뒤, 작성할 Todo 리스트 앱의 요구사항을 간단하게 나열해 보겠습니다.

- 복수의 todo를 갖는 todo list가 있다.
- 각 todo는 id, todo, isComplete를 props로 갖는다.
- isComplete가 true인 경우 del Tag로 표현된다.
- todo를 클릭하면 isComplete가 toggle 된다.
- '추가' 버튼을 통해 todo를 추가할 수 있다.
- '제거' 버튼을 통해 todo를 제거할 수 있다.

자. 익숙하진 않지만 test파일 먼저 생성해보겠습니다. root 경로에서 `mkdir -p /src/components/__tests__`를 통해 테스트 코드를 작성할 폴더를 생성하고, Todo 컴포넌트의 test파일을 Todo.js로 생성합니다.

> `__tests__` 를 생성한 이유는 eject 이후의 package.json(아래의 코드 참고)을 확인해보면 컴포넌트와 test 코드를 분리된 폴더에서 작성할 수 있도록 설정이 작성되어있고, 파일 이름에서 .test. 를 생략할 수 있습니다. 테스트 파일이 여러 개가 된다면 이 방법이 더 깔끔하다고 생각합니다. 컴포넌트 별로 폴더가 분리되어있다면 .test를 사용하는게 더 관리하기 좋을 수 있습니다.

```json

// package.json after npm run eject
...
"testMatch": [
      "<rootDir>/src/**/__tests__/**/*.{js,jsx,ts,tsx}",
      "<rootDir>/src/**/*.{spec,test}.{js,jsx,ts,tsx}"
    ]
...
```

테스트 코드를 작성하기 전에 주요 메서드를 확인해 봅시다.

- [describe(name, fn)](https://jestjs.io/docs/api#describename-fn)
  - 여러개의 test를 그룹화하는 용도.
  - describe.(파생 메서드) 를 통해 다양한 구조의 그룹화가 가능하다.
- [test(name, fn , timeout)](https://jestjs.io/docs/api#testname-fn-timeout)
  - 하나의 테스트를 작성할 수 있는 블록 생성.
  - describe와 마찬가지로 파생 메서드를 통해 다양한 기능을 제공한다.
- [expect(value)](https://jestjs.io/docs/expect#expectvalue)

  - 값을 테스트할 때 사용.
  - `어떤 값에 대해 예측할거야`를 선언적으로 표현할 수 있다.

  ```javascript
  test('the best flavor is grapefruit', () => {
    // 가장 맛좋은 사탕을 예측하겠다. 그것은 틀림 없이 포도 맛이야!
    expect(bestCandyFlavor()).toBe('grapefruit');
  });
  ```

  - 위에서 사용한 `.toBe()`와 같이 `expect()`뒤에는 예측을 위한 다양한 [method](https://jestjs.io/docs/expect#methods)들이 제공된다.

## 기능을 생각해보기

작성할 Todo List 컴포넌트가 어떻게 존재하는지 생각해보자 1) 초기화를 위해 기존의 todo들을 불러와 Todo 컴포넌트들을 생성. 2) 사용자의 추가/제거를 통해 리스트가 수정.

### 1. TodoList 초기화

```javascript
// src/components/__tests__/TodoList.js
import { render } from '@testing-library/react';
import TodoList from '../TodoList';

describe('<TodoList />', () => {
  test('Todo 목록을 초기화한다.', () => {
    // given
    const todos = [
      { id: 1, todo: 'test todo 1', isComplete: false },
      { id: 2, todo: 'test todo 2', isComplete: true },
      { id: 3, todo: 'test todo 3', isComplete: true },
    ];
    // when
    const { getAllByTestId } = render(<TodoList todos={todos} />);
    const TodoElements = getAllByTestId(/test/i);

    // then
    expect(TodoElements.length).toBe(3);
    TodoElements.forEach(todo => {
      expect(todo).toBeInTheDocument();
    });
  });
});
```

위의 테스트를 요약하면 다음과 같습니다.

- render함수를 통해 TodoList 컴포넌트가 렌더링 되었을 때를 재현합니다.
- getAllByTestId를 통해 자식 컴포넌트를 가져옵니다.
- 총 갯수와 모든 todo가 document에 렌더링되었는지 확인합니다.

render함수는 RenderResult 타입의 객체를 반환하고 자식 컴포넌트를 특정하기 위한 메소드들을 가지고 있습니다. 이 중에 `getAllByTestId()`를 사용해서 todo 컴포넌트들을 가져오겠습니다.

이제 테스트 코드를 작성했으니 컴포넌트를 작성해보겠습니다.

```javascript
// src/components/TodoList.tsx
import Todo from './todo';

const TodoList = ({ todos }) => {
  const mapTodos = todo => <Todo {...todo} key={todo.id} />;
  return <div id="todo_list">{todos.map(mapTodos)}</div>;
};

export default TodoList;

// src/components/Todo.tsx
import { useState } from 'react';

function Todo({ id, todo }) {
  return (
    <>
      <div data-testid={`test-${id}`}>{todo}</div>
    </>
  );
}

export default Todo;
```

jest는 data-testid 프로퍼티를 통해 특정 컴포넌트를 가져올 수 있는 선택자를 제공합니다. Enzyme와 같은 서드파티 라이브러리를 연동하면 querySelector로 특정할 수 있습니다.

다음은 완료된 Todo는 del 태그로 미완료된 Todo는 span Tag로 Todo 컴포넌트의 텍스트가 표현는지 테스트하는 코드를 작성해보겠습니다. 그리고 클릭 이벤트가 발생하면 isComplete가 토글되어 del 태그로 리렌더되는 기능도 테스트 해보겠습니다.

```javascript
// src/components/__tests__/Todo.js
import { render, screen } from '@testing-library/react';
import Todo from '../Todo';

describe('<Todo />', () => {
  test('완료된 Todo는 del 태그, 미완료된 Todo는 span 태그로 랜더', () => {
    // given
    const compeleteTodo = { id: 1, todo: 'test todo', isComplete: true };
    const notCompeleteTodo = { id: 2, todo: 'test todo', isComplete: false };

    // when
    render(<Todo {...compeleteTodo} />);
    render(<Todo {...notCompeleteTodo} />);
    const compeleteElement = screen.getByTestId('test-1');
    const notCompeleteTodoElement = screen.getByTestId('test-2');

    // then
    expect(compeleteElement.firstChild.tagName).toBe('DEL');
    expect(notCompeleteTodoElement.firstChild.tagName).toBe('SPAN');
  });
  test('Todo를 클릭하면 isComponent 속성이 토글된다.', () => {
    // given
    const todo = { id: 1, todo: 'test todo', isComplete: false };

    // when
    render(<Todo {...todo} />);
    const TodoElement = screen.getByTestId('test-1');

    // then
    expect(TodoElement.firstChild.tagName).toBe('SPAN');
    fireEvent.click(TodoElement);
    expect(TodoElement.firstChild.tagName).toBe('DEL');
  });
});
```

이제 test script를 실행하면 모든 테스트가 인식되면서 fail을 보여줍니다. 사실 테스트를 작성하면서 기능이 등장하면 바로바로 구현해주는게 TDD의 멋이라고 생각합니다.. 테스트 코드를 통해 구현해야할 기능들이 더욱 명확해 졌기 때문에 지체없이 구현합니다.

```javascript
// src/components/Todo.tsx
import { useState } from 'react';

function Todo(todo) {
  const [isComplete, setIsComplete] = useState(todo.isComplete);
  const handleClick = e => {
    e.preventDefault();
    setIsComplete(!isComplete);
  };
  const text = isComplete ? <del>{todo.todo}</del> : <span>{todo.todo}</span>;
  return (
    <>
      <div data-testid={`test-${todo.id}`} onClick={handleClick}>
        {text}
      </div>
    </>
  );
}
export default Todo;

// src/components/TodoList.tsx
import Todo from './Todo';

const TodoList = ({ todos }) => {
  const mapTodos = todo => <Todo {...todo} key={todo.id} />;
  return <div id="todo_list">{todos.map(mapTodos)}</div>;
};

export default TodoList;
```

예상한 결과 값과 모두 일치하면 다음과 같이 pass 표시를 볼 수 있습니다.
![jest-pass](https://user-images.githubusercontent.com/61958795/116811185-dd30e780-ab82-11eb-9c49-c046c10e89aa.png)
