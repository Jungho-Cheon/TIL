# Functional Programming 1

ES6 이후 자바스크립트에는 Symbol이라는 새로운 원시 타입(Primitive Type)이 등장하였다. Symbol의 특징은 다음과 같다.

- Unique를 보장한다.
- `for in`, `Object.keys()`를 통해 조회되지 않는다.

오늘 알아볼내용은 기본적으로 자바스크립트에서 제공하는 Symbol 중 iterator와 이를 이용하여 객체를 iterable로 만들고 함수로 다양하게 활용하는 방법이다.

자바스크립트의 객체는 정해진 내용을 지키기만 하면 인터페이스를 구현한 것으로 인정하는 덕타이핑을 지원한다. 이를 이용하여 객체를 `Iterable`로 만들 수 있다. 조건은 다음과 같다.

- `Symbole.iterator`를 키로 `Iterator`를 반환하는 메서드를 갖는다.
- `Iterator`는 `next()` 메서드를 갖는다.
- `next()` 메서드는 `value`와 `done`을 가진 객체를 반환한다.

count를 받아 하나씩 줄여나가는 iterable을 구현해보면 다음과 같다.

```javascript
const MyIterable = class {
  constructor(count) {
    this.count = count;
  }
  [Symbol.iterator]() {
    let count = this.count;
    return {
      next() {
        return count >= 0
          ? {
              value: count--,
              done: false,
            }
          : {
              done: true,
            };
      },
    };
  }
};
```

위의 MyIterable은 덕타이핑을 통해 `Iterable` 인터페이스의 조건을 만족하기 때문에 이제 `Iterable`이 들어갈 수 있는 자리에 들어갈 수 있게 된다. 예를 들면 다음과 같다.

```javascript
const myIterable = new MyIterable(10);
for (const i of myIterable) {
  console.log(i);
}
/**
 * 출력
 * 10
 * 9
 * 8
 * 7
 * 6
 * 5
 * 4
 * 3
 * 2
 * 1
 * 0
 */
```

함수형 프로그래밍에서는 `Iterable`을 이용하여 다양한 함수를 만들어 내게 된다. `Array` 타입에서 제공하는 `.map()` 함수는 `Set`이나 `Map`에는 존재하지 않는다. 하지만 제공되지 않는 함수들이 있기 때문에, 직접 구현하여 사용하게 되면 궁극적으로 공통적으로 사용되는 코드들을 함수로 분리하고 추상화하여 선언적인 프로그래밍이 가능해 진다고 생각된다.

## map

`map`은 각 요소들에 함수를 적용하여 새로운 요소로 만든 뒤 반환하는 함수이다.
간단하게 구현해보면 다음과 같다.

```javascript
const map = (f, iter) => {
  const ret = [];
  for (const el of iter) {
    ret.push(f(el));
  }
  return ret;
};
```

적용할 함수와 `iterable`을 받으면 `for in` 문을 통해 쉽게 구현할 수 있다.

## filter

`filter`도 마찬가지로 함수와 `iterable`을 인자로 받는다. 다만 함수에 인자를 넣어 실행했을 때, `falsey`한 값은 필터링 해버린다.

```javascript
const filter = (f, iter) => {
  const ret = [];
  for (const el of iter) {
    f(el) && ret.push(el);
  }
  return ret;
};
```

## reduce

`reduce`는 약간 특이하다. `iterable`의 요소들을 하나의 값으로 추합하는 용도이며, 초기값을 option으로 지정해줄 수 있다.

```javascript
const reduce = (f, iter, acc) => {
  if (!acc) {
    iter = iter[Symbol.iterator]();
    acc = iter.next().value;
  }
  for (const el of iter) {
    acc = f(acc, el);
  }
  return acc;
};
```

만약 초기값이 주어지지 않으면 iter의 `[Symbol.iterator]()`를 실행하여 `iterator`를 반환받아 첫번째 값으로 초기값을 설정한다.

iter는 이미 `iterator`인데 `[Symbol.iterator]()`가 어떻게 가능한지 의문이 생길 수 있다. 이것은 iter가 `well-formed`로 생성된 경우 가능하다.

> ### well-formed-iterable
>
> well-formed-iterable은 반환된 iterator 또한 `[Symbol.iterator]` 갖으며 this를 반환한다.
>
> ```javascript
> ...
> [Symbol.iterator]() {
>   next() {
>     return {
>       value: ...,
>       done: ...
>     }
>   }
>   [Symbol.iterator]() {
>     return this;
>   }
> }
> ...
> ```

위에서 만들어본 map, filter, reduce는 기능은 문제없지만 길어지게되면 보기 좋지 않다.

```javascript
const products = [
  { name: 'p1', price: 1000 },
  { name: 'p2', price: 1500 },
  { name: 'p3', price: 2000 },
  { name: 'p4', price: 2500 },
];

// 가격이 2000원 이하인 제품의 합.
const sum = reduce(
  (acc, cur) => acc + cur,
  filter(
    p => p <= 2000,
    map(p => p.price, products)
  )
);
```

함수가 늘어나더라도 일관되게 표현하기 위해 go, pipe함수를 사용하게 된다.

## go() & pipe()

```javascript
const go = (...args) => reduce((a, f) => f(a), args);
const pipe = (f, ...fs) => (...args) => go(f(...args), ...fs);
```

`go()`는 `reducer()`를 이용하여 만들게 된다. `reducer()`의 초기화의 특징을 이용하여 첫번째 값을 인자로 시작하여 나머지 함수들을 차례대로 실행해준다. 예를들면 다음과 같이 사용할 수 있다.

```javascript
const sum = go(
  products,
  products => map(p => p.price, products),
  prices => filter(p => p <= 2000, prices),
  prices => reduce((acc, cur) => acc + cur, prices)
);
```

위에서 아래로 실행되며 일관적인 표현이 가능해졌다. `go`는 즉시 실행되어 값을 반환하지만 `pipe`는 `go`를 실행해주는 함수를 반환하기 때문에 언제든지 재사용할 수 있게해준다.

```javascript
const getSum = pipe(
  products => map(p => p.price, products),
  prices => filter(p => p <= 2000, prices),
  prices => reduce((acc, cur) => acc + cur, prices)
);
const sum = getSum(products);
```

마지막으로 코드를 좀더 깔끔하게 만들기 위해 `curry`라는 함수를 사용하게 된다.

## curry()

```javascript
const curry = (f = (a, ..._) => (_.length ? f(a, ..._) : (..._) => f(a, ..._)));
```

복잡해 보이는 `curry` 함수는 다음과 같은 일을 한다. 함수`(f)={}`를 인자로 받아 다수의 인자를 받아 함수`(f)=>{}`를 실행해주는 함수`(a..._)=>{}`를 반환한다. 이때, 두번째 인자`(..._)`가 존재하는 경우 즉시 실행하지만 두번째 인자가 존재하지 않는다면 또 다시 함수형태로 반환하여 두번째 인자가 들어올 때 까지 대기했다가 실행하는 기능을 제공한다.

`curry()`는 다음과 같이 응용할 수 있다. 위에서 만들었던 `map`, `filter`, `reduce`를 모두 `curry`로 감싸보자.

```javascript
const map = curry((fn, iter) => {
  const ret = [];
  for (const item of iter) {
    ret.push(fn(item));
  }
  return ret;
});

const filter = curry((fn, iter) => {
  const ret = [];
  for (const item of iter) {
    fn(item) && ret.push(item);
  }
  return ret;
});

const reduce = curry((fn, iter, acc) => {
  if (acc === undefined) {
    iter = iter[Symbol.iterator]();
    acc = iter.next().value;
  }
  for (const item of iter) {
    acc = fn(acc, item);
  }
  return acc;
});
```

현재 세개의 함수는 인자를 받아 실행 될 때 첫번째 인자로 함수를 받게 되면 `(..._) => f(a, _)`의 형태를 가지게 된다. 따라서 다음과 같은 표현이 가능해진다.

```javascript
const sum = go(
  products,
  map(p => p.price),
  filter(p => p <= 2000),
  reduce((acc, cur) => acc + cur)
);

const getSum = pipe(
  map(p => p.price),
  filter(p => p <= 2000),
  reduce((acc, cur) => acc + cur)
);
const sum = getSum(products);
```

기존의 코드보다 훨신 이해하기 쉬워졌다. 가독성이 좋아진 것 뿐만아니라 코드를 마치 블록처럼 사용할 수 있게되었기 때문에 공통된 코드를 관리하기 쉬워졌다.
