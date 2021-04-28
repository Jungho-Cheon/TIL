# Event Delegation

만약 10개의 이벤트 리스너를 가진 엘리먼트가 동적으로 생성되고 제거된다고 했을 때, 그 갯수가 100개만 되어도 1000개의 이벤트 리스너가 생성되어 메모리 낭비를 발생시키며, 또한 이러한 구조는 메모리 누수를 만들기 쉽다. 이를 해결하기 위한 방법이 `이벤트 위임`이다.

사용자의 액션에 의해 이벤트가 발생하면 해당 이벤트는 버블링되어 document 까지 올라간다. 이 때문에 자식 엘리먼트에서 발생하는 이벤트를 부모 엘리먼트에서도 감지할 수 있다. 이러한 특징을 이용해 이벤트 위임을 사용할 수 있다. 특정 엘리먼트에 하나하나 이벤트 리스너를 등록하지 않고 부모 엘리먼트에만 등록하여 이벤트를 위임하는 것이다.

예를 들어, 3개의 메뉴 버튼이 있고 클릭 이벤트를 통해 특정한 작업을 수행해야 한다고 생각해보자. 만약 메뉴 버튼이 10개로 늘어난다면 이벤트 리스너 또한 그만큼 만들어야 할 것이다. 하지만 부모 엘리먼트에 이벤트를 등록하면 다음과 같이 해결할 수 있다.

```html
<ul id="menu">
  <li><button id="file">file</button></li>
  <li><button id="edit">edit</button></li>
  <li><button id="view">view</button></li>
</ul>

<script>
  document.getElementById('menu').addEventListener('click', function (e) {
    const target = e.target;
    if (target.id === 'file') {
      console.log('file');
    } else if (target.id === 'edit') {
      console.log('edit');
    } else if (target.id === 'view') {
      console.log('view');
    }
  });
</script>
```

사실 이런식으로 분기를 만드는 것이 단일 책임의 원칙을 지키기에는 좋은 코드는 아니라고 생각한다. 다행히도, javascript 라이브러리나 프레임워크에서는 각자의 방식으로 이벤트 위임을 제공하고 있으며, 우리가 위와 같은 방식으로 처리하지 않아도 이미 이벤트 위임을 사용하여 최적화를 해주고 있는 라이브러리도 있다.

> React는 이미 이벤트 위임을 통한 최적화를 지원하고 있으며, [React 17](https://ko.reactjs.org/blog/2020/10/20/react-v17.html#changes-to-event-delegation) 버전부터 이벤트 위임을 하는 부모 엘리먼트를 document 레벨이 아닌 react의 가상 DOM을 업데이트하는 Root 엘리먼트로 변경하였다고 한다.
