# Functional Programming 2

## Lazy Evaluation(지연 평가)

지난번 포스트에서 작성한 함수들은 모든 값들이 연산된 후 다음 함수를 수행하는 특징을 가지고 있다. 만약 1000만개의 값들을 연산하고 그 중 3개의 값만 필요한 경우가 생겼다고 생각해보면, 3개의 값만 필요하지만 나머지 값들에 대한 불필요한 연산이 이뤄진다.

이를 해결하기 위해 generator를 사용하여 함수를 구현하게 된다. 우선 기존의 map, filter함수를 살펴보자

```javascript
const map = (f, iter) => {
  const ret = [];
  for (const el of iter) {
    ret.push(f(el));
  }
  return ret;
};

const filter = (f, iter) => {
  const ret = [];
  for (const el of iter) {
    f(el) && ret.push(el);
  }
  return ret;
};
```

함수 내부를 보면 값을 반환하기 위해 `ret` 배열을 `for of`문을 통해 모두 채우게 된다. 이를 지연적으로 필요에 따라 처리하게 만들기 위해 `generator`로 변환해 보자.

```javascript
L.map = function* (fn, iter) {
  for (const item of iter) {
    yield fn(item);
  }
};

L.filter = function* (fn, iter) {
  for (const item of iter) {
    if (fn(item)) yield item;
  }
};
```

함수 내부에 배열을 가지고 있지 않아도 되며, 평가가 진행됨에 따라 generator에서 yield를 통해 값을 하나씩 생성하게 된다. generator는 기본적으로 iterable을 반환하기 때문에 기존에 만들었던 함수와 호환이 가능하다. 따라서 즉시평가, 지연평가의 특징을 가지고 적절하게 사용하면 더 좋은 성능을 만들어낼 수 있을 듯 하다.

## yield *

지금까지 `Iterable`의 모든 값을 `yield`하기 위해 `for of`문을 사용하여 값을 순회해왔다. 이는 `yield *`를 통해 간결하게 표현할 수 있다. 중첩된 배열을 펼쳐주는 `flat`함수를 구현해보면서 `yield *` 을 사용해보자.

```javascript
const isIterable = iter => iter && iter[Symbol.iterator]
const flat = function* (iter) {
  for(const a of iter) {
    if(isIterable(a)) for(const b of a) yield b;
    else yield a;
  }
}

const arr = [[1, 2], 3, 4, [5, 6, 7], 8, 9, 10]
go(arr, flat, map(log))
// 
```

`yield* a`는 `for(const b of a) yield b`와 동일한 결과를 만들어 낸다.
따라서 위의 flat 함수는 다음과 같이 쓸 수 있다.

```javascript
const flat = function* (iter) {
  for (const a of iter) {
    if (isIterable(a)) yield* a;
    else yield a;
  }
};
```

만약 여러번 중첩된 배열을 모두 flat하는 `deepFlat` 함수를 구현해야 한다면 재귀적으로 해결할 수 있다.

```javascript
const deepFlat = function* (iter) {
  for (const a of iter) {
    if (isIterable(a)) yield* deepFlat(a);
    else yield a;
  }
};
```

결과를 확인해보면 다음과 같다.

```javascript
const flat = require('./flat');
const arr = [[1, 2], 3, 4, [5, 6, 7], 8, 9, 10];
go(arr, flat, map(log));
// 1 2 3 4 5 6 7 8 9 10

const deepFlat = require('./deepFlat');
const arr2 = [[[1, 2], [3], 4], 5, 6, [7, [8, [9, 10]]]];
go(arr2, deepFlat, map(log));
// 1 2 3 4 5 6 7 8 9 10
```
