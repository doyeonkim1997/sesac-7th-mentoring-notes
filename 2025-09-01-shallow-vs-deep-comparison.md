### Q. 얕은 비교와 깊은 비교의 차이를 설명해주세요. (예시 코드 첨부 필요)

JavaScript에서 객체나 배열을 비교할 때 두 가지 방식이 있다.

얕은 비교는 겉(참조값, 즉 주소)만 보고, 깊은 비교는 안(내부 값)까지 확인한다.

얕은 비교는 빠르고 간단하지만, 안의 값이 같아도 다른 메모리에 있으면 "다르다"고 본다.

깊은 비교는 값 하나하나를 다 비교해서 정확하지만, 속도가 느릴 수 있다.

상황에 따라 속도가 중요하면 얕은 비교, 정확함이 중요하면 깊은 비교를 사용한다.

### 1. 얕은 비교 (Shallow Comparison)

```jsx
const obj1 = { name: "Alice" };
const obj2 = { name: "Alice" };
const obj3 = obj1;

// 1. 객체끼리 비교
console.log(obj1 === obj2); // false (참조값이 다르기 때문)

// 2. 내부 프로퍼티(문자열) 비교
console.log(obj1.name === obj2.name); // true (내부 값은 같음)

// 3. 같은 객체를 참조할 때
console.log(obj1 === obj3); // true (obj3는 obj1을 그대로 참조(같은 주소를 가리킴)
```

- `===` 연산자는 객체의 참조(주소) 를 비교한다.
- 따라서 `obj1`과 `obj2`는 같은 구조와 값을 가져도, 다른 메모리에 저장되어 있으므로 `false`가 나온다.

### 2. 깊은 비교 (Deep Comparison)

```jsx
function deepEqual(a, b) {
  if (a === b) return true;

  if (
    typeof a !== "object" ||
    typeof b !== "object" ||
    a == null ||
    b == null
  ) {
    return false;
  }

  const keysA = Object.keys(a);
  const keysB = Object.keys(b);

  if (keysA.length !== keysB.length) return false;

  for (let key of keysA) {
    if (!deepEqual(a[key], b[key])) return false;
  }

  return true;
}

const obj3 = { name: "Alice", info: { age: 25 } };
const obj4 = { name: "Alice", info: { age: 25 } };

console.log(deepEqual(obj3, obj4)); // true (내부 값까지 같음)
```

- `deepEqual`은 안쪽 값까지 전부 확인한다.
- 값이 전부 동일하다면 `true`, 하나라도 다르면 `false`.

### 3. deepEqual(obj1, obj2) 호출 흐름

```jsx
deepEqual(obj1, obj2)
 └─ check keys: ['name', 'info']
     ├─ compare key: name
     │    └─ "Alice" === "Alice" → true
     │
     └─ compare key: info
          └─ deepEqual(obj1.info, obj2.info)
               └─ check keys: ['age']
                   └─ compare key: age
                        └─ 25 === 25 → true
```

---

### Q. React는 props/state 비교 시 어떤 방식을 사용하나요?

React는 얕은 비교를 사용한다.

props나 state가 바뀌었는지 확인할 때 내부 값까지 보지 않고 주소(참조)만 확인한다.

새로운 객체나 배열을 만들면 내부 값이 같아도 "바뀌었다"고 판단해 리렌더링이 발생한다.

반대로, 내부 값이 달라도 같은 객체를 쓰면 "안 바뀌었다"고 판단해서 리렌더링이 안 된다.

```jsx
import { useState } from "react";

function App() {
  const [list, setList] = useState([1, 2, 3]);

  // ❌ 잘못된 방식: push → 같은 배열 참조 유지
  const wrongUpdate = () => {
    list.push(4); // 원본 배열 수정 (참조 동일)
    setList(list);
    // React는 같은 참조라고 보고, 변경이 없다고 판단 → 리렌더링 안 됨
  };

  // ✅ 올바른 방식: spread → 새로운 배열 생성
  const correctUpdate = () => {
    setList([...list, 4]);
    // 새로운 배열 참조가 생기므로 변경으로 판단 → 리렌더링 발생
  };

  return (
    <div>
      <h3>{list.join(", ")}</h3>
      <button onClick={wrongUpdate}>잘못된 업데이트</button>
      <button onClick={correctUpdate}>올바른 업데이트</button>
    </div>
  );
}

export default App;
```

---

### Q. 원시값과 객체/배열/함수 비교 시 얕은 비교가 어떻게 동작하는지 설명해주세요. (예시 코드 첨부 필요)

얕은 비교(`===`)는 자료형에 따라 다르게 동작한다.

원시값은 값 자체를 비교하고, 참조형은 주소를 비교한다.

따라서 원시값은 값이 같으면 같다고 보고, 참조형은 내부 값이 같아도 주소가 다르면 다르다고 본다.

### **1. 원시값(Primitive)** → 값 자체를 비교

```jsx
console.log(10 === 10); // true
console.log("hi" === "hi"); // true
console.log("hi" === "hello"); // false
```

### **2. 객체(Object)** → 참조(주소) 비교

```jsx
const obj1 = { name: "Alice" };
const obj2 = { name: "Alice" };
const obj3 = obj1;

console.log(obj1 === obj2); // false (내용 같아도 다른 주소)
console.log(obj1 === obj3); // true (같은 참조)
```

### **3. 배열(Array)** → 참조(주소) 비교

```jsx
const arr1 = [1, 2];
const arr2 = [1, 2];
const arr3 = arr1;

console.log(arr1 === arr2); // false (다른 배열)
console.log(arr1 === arr3); // true (같은 참조)
```

### **4. 함수(Function)** → 참조(주소) 비교

```jsx
function f1() {}
function f2() {}
const f3 = f1;

console.log(f1 === f2); // false (다른 함수)
console.log(f1 === f3); // true (같은 참조)
```

---

### Q. 얕은 비교 때문에 발생할 수 있는 문제 사례를 말해보세요. (예: 매번 새 객체/함수 생성, 불변성 위반)

### **1. 매번 새 객체/함수 생성**

- 렌더링마다 새 함수가 생성되면 참조가 달라져 불필요한 리렌더링이 발생한다.
- 해결: `useCallback`, `useMemo`로 메모이제이션.

```jsx
const Button = React.memo(({ onClick }) => {
  console.log("리렌더링됨");
  return <button onClick={onClick}>클릭</button>;
});

function App() {
  const [count, setCount] = useState(0);

  return (
    <Button onClick={() => setCount(count + 1)} /> // 매번 새 함수 생성 → 리렌더링 발생
  );
}
```

### **2. 불변성 위반**

- 원본을 직접 수정하면 참조가 그대로라서 React가 변경을 감지하지 못한다.
- 해결: 원본은 두고 새로운 배열이나 객체를 만들어 전달한다.

```jsx
// ❌ 잘못된 방식
todos.push(newTodo);
setTodos(todos); // 같은 참조 → 감지 못함

// ✅ 올바른 방식
setTodos([...todos, newTodo]); // 새로운 배열 → 감지 O
```

---

### Q. 객체나 배열을 state로 관리할 때 불변성을 지켜야 하는 이유는 무엇인가요?

React에서 상태(state)는 컴포넌트의 UI 업데이트를 위한 핵심 데이터이다.

특히 객체나 배열과 같은 참조형 데이터를 직접 수정하면, 변경된 값이 새로운 객체로 인식되지 않아 setState 호출 시 React가 변경을 감지하지 못하고 UI가 업데이트 되지 않는다.

따라서, 불변성을 유지하기 위해 원본 배열이나 객체를 직접 변경하지 않고, 복사본을 만들어 변경한 후 새로운 값으로 상태를 업데이트 해야한다.

- **원본 배열을 직접 변경하는 메서드들**
  - push() : 배열의 마지막에 요소 추가
  ```jsx
  const arr1 = [1, 2, 3];
  arr1.push(4);
  console.log(arr1); // [1, 2, 3, 4]
  ```
  - pop(): 배열의 마지막 요소 제거
  ```jsx
  const arr2 = [1, 2, 3];
  arr2.pop();
  console.log(arr2); // [1, 2]
  ```
  - shift(): 배열의 첫 요소 제거
  ```jsx
  const arr3 = [1, 2, 3];
  arr3.shift();
  console.log(arr3); // [2, 3]
  ```
  - unshift(): 배열의 첫 부분에 요소 추가
  ```jsx
  const arr4 = [1, 2, 3];
  arr4.unshift(0);
  console.log(arr4); // [0, 1, 2, 3]
  ```
  - splice(): 배열의 특정 인덱스에서 요소 제거 또는 추가
  ```jsx
  const arr5 = [1, 2, 3, 4];
  arr5.splice(1, 2, 99, 100);
  // 인덱스 1부터 2개 제거 → [2,3], 그 자리에 99, 100 추가
  console.log(arr5); // [1, 99, 100, 4]
  ```
  - sopt(): 배열의 정렬을 수행 (원본 정렬)
  ```jsx
  const arr6 = [3, 1, 4, 2];
  arr6.sort();
  console.log(arr6); // [1, 2, 3, 4]
  ```
  - 이 메서드들은 모두 원본 배열을 직접 변경하므로 React 상태 관리에 부적합하다.
- **불변성을 지키는 메서드**
  - concat(): 배열 합치기 (새 배열 반환)
  ```jsx
  const arr1 = [1, 2];
  const newArr1 = arr1.concat([3, 4]);
  console.log(newArr1); // [1, 2, 3, 4]
  console.log(arr1); // [1, 2] (원본 유지)
  ```
  - slice(): 배열 일부 잘라내기 (새 배열 반환)
  ```jsx
  const arr2 = [1, 2, 3, 4];
  const newArr2 = arr2.slice(1, 3);
  console.log(newArr2); // [2, 3]
  console.log(arr2); // [1, 2, 3, 4] (원본 유지)
  ```
  - map() : 요소 변환 (새 배열 반환)
  ```jsx
  const arr3 = [1, 2, 3];
  const newArr3 = arr3.map((x) => x * 2);
  console.log(newArr3); // [2, 4, 6]
  console.log(arr3); // [1, 2, 3] (원본 유지)
  ```
  - filter() : 조건에 맞는 요소만 남김 (새 배열 반환)
  ```jsx
  const arr4 = [1, 2, 3, 4];
  const newArr4 = arr4.filter((x) => x % 2 === 0);
  console.log(newArr4); // [2, 4]
  console.log(arr4); // [1, 2, 3, 4] (원본 유지)
  ```
  - reduce() : 누적 계산, 새 값 반환 (원본 유지)
  ```jsx
  const arr5 = [1, 2, 3];
  const sum = arr5.reduce((acc, cur) => acc + cur, 0);
  console.log(sum); // 6
  console.log(arr5); // [1, 2, 3] (원본 유지)
  ```

---

### Q. setState에 같은 참조를 넣었는데 리렌더링이 되지 않은 경험이 있나요? 어떻게 해결했나요?

배열을 직접 수정해 `setState`에 같은 참조를 넣었더니 화면이 바뀌지 않는 경험이 있었다.

React는 얕은 비교만 하기 때문에 바뀌지 않은 것으로 판단했다.

이후 불변성을 지켜 새로운 참조를 만들었고, `setTodos([...todos, newTodo])`처럼 업데이트했다.

이 과정을 통해 React는 참조가 달라져야 상태 변경을 감지한다는 점을 배웠다.

---

### Q. React가 props/state 얕은 비교를 쓴다는 걸 알아야 하는 이유는 무엇일까요?

React는 얕은 비교로 상태 변경 여부를 판단한다.

### **1. 불필요한 리렌더링 이해**

- React는 `props`와 `state`를 얕은 비교(shallow comparison) 해서 변경 여부를 판단한다.
- 이 사실을 모르고 매번 새로운 객체/함수를 만들어 넘기면, 실제 값이 같아도 참조가 달라져서 불필요하게 리렌더링이 발생한다.

### **2. 렌더링 안 되는 버그 방지**

- 반대로 배열/객체를 직접 수정(불변성 위반)하면 참조가 그대로라서 React는 변경을 감지하지 못한다.
- “왜 UI가 안 바뀌지?” 같은 흔한 버그가 여기서 발생한다.

### **3. 최적화 기법과 연결**

- `React.memo`, `PureComponent`, `useCallback`, `useMemo` 같은 최적화 훅/컴포넌트는 모두 얕은 비교를 기반으로 동작한다.
- 따라서 얕은 비교의 한계를 알아야 언제 메모이제이션이 효과적이고, 언제 쓸모없는지 판단할 수 있다.

---

### 추가적으로 공부한 부분

- 원시값과 객체 비교 방식 차이

  1. **객체 자체 비교 (`obj1 === obj2`)**

     - 객체는 **주소(참조값 a.k.a 집주소)** 를 비교한다.
     - 같은 객체를 가리키면 `true`, 다르면 `false`.

     ```jsx
     const obj1 = { age: 20 };
     const obj2 = { age: 20 };
     console.log(obj1 === obj2); // false

     const obj3 = obj1;
     console.log(obj1 === obj3); // true
     ```

  ***

  1. **객체 내부 값 비교 (`obj1.key === obj2.key`)**

     - 프로퍼티 값이 원시값이면, **값 자체**를 비교한다.
     - 값이 같으면 `true`, 다르면 `false`.

     ```jsx
     const obj1 = { name: "Alice" };
     const obj2 = { name: "Alice" };
     console.log(obj1.name === obj2.name); // true

     const obj3 = { name: "Alice" };
     const obj4 = { name: "Alices" };
     console.log(obj3.name === obj4.name); // false
     ```

  ***

  ✅ **정리 한 줄**

  - `===` 로 **객체 자체** 비교 → 주소 비교
  - `===` 로 **객체 안의 원시값** 비교 → 값 비교

  ### 📌 내가 헷갈렸던 것

  - `obj1 === obj2` 는 **객체 자체**를 비교하는 거라 → “집 주소(참조값)가 같냐”를 묻는 것
  - 그런데 `obj1.key === obj2.key` 는 **객체 안의 값(원시값)** 을 비교하는 거라 → 실제 “값(스펠링/숫자)”을 묻는 것
  - 그래서 내가 헷갈린 건 👉 **왜 객체 전체를 비교하면 false인데, 내부 프로퍼티를 꺼내 비교하면 true가 나오는지**

- 스프레드 연산자의 사용 위치에 따른 최종 배열 구조

  ```jsx
  const list = [1, 2, 3];

  // 뒤에 쓰면 → 뒤에 추가
  const addBack = [...list, 4];
  console.log(addBack); // [1, 2, 3, 4]

  // 앞에 쓰면 → 앞에 추가
  const addFront = [0, ...list];
  console.log(addFront); // [0, 1, 2, 3]

  // 중간에 끼워 넣는 것도 가능
  const addMiddle = [1, ...list, 99];
  console.log(addMiddle); // [1, 1, 2, 3, 99]
  ```

  ***

  🔹 React에서 스프레드 사용

  ```jsx
  // ✅ 새로운 참조 생성 → 리렌더링 O
  setList([...list, 4]); // 맨 뒤에 추가
  setList([0, ...list]); // 맨 앞에 추가
  ```

  👉 어디에 두느냐에 따라 **앞/뒤/중간** 위치가 바뀌는 것일 뿐,
  핵심은 “**새로운 배열을 만들었다**” → React가 변경 감지 가능.

  ***

  ✅ 정리하면:

  - 스프레드를 **앞, 뒤, 중간 어디에 써도 작동은 다 된다.**
  - 단, 원본 배열을 복사하면서 **새 참조를 만들어야만** 리렌더링이 일어나는 것.

---

### **학습에 참고한 자료**

- F-Lab - https://f-lab.kr/insight/understanding-shallow-and-deep-equality-20250107
- 티스토리 - https://disco-biscuit.tistory.com/169#:~:text=%EC%95%84%EB%9E%98%EB%8A%94%20React%EC%97%90%EC%84%9C%20%EC%83%81%ED%83%9C(%ED%8A%B9%ED%9E%88%20%EC%B0%B8%EC%A1%B0%ED%98%95%20%EB%8D%B0%EC%9D%B4%ED%84%B0)%EB%A5%BC%20%EB%8B%A4%EB%A3%B0%20%EB%95%8C,%ED%95%84%EC%9A%94%EC%84%B1React%EC%97%90%EC%84%9C%20%EC%83%81%ED%83%9C(state)%EB%8A%94%20%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8%EC%9D%98%20UI%20%EC%97%85%EB%8D%B0%EC%9D%B4%ED%8A%B8%EB%A5%BC%20%EC%9C%84%ED%95%9C%20%ED%95%B5%EC%8B%AC
- 티스토리 - https://bitcoins.tistory.com/40
- 벨로그 - https://velog.io/@kms0206/%EB%A6%AC%EC%95%A1%ED%8A%B8%EC%9D%98-%EB%B6%88%EB%B3%80%EC%84%B1%EC%9D%B4%EB%9E%80-%EC%99%9C-%ED%95%84%EC%9A%94%ED%95%9C%EB%8D%B0

---
