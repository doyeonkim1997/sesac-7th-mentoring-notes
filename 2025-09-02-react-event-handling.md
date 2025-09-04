# 09-02 기술 면접 주제: React 이벤트

---

### **Q.React에서 이벤트 핸들링은 어떻게 정의하나요? (일반 JS와 차이점)**

React에서 이벤트 핸들링은 일반 JS와 몇 가지 차이가 있다.

먼저 HTML에서는 속성 이름을 소문자 `onclick`으로 쓰고, 문자열로 함수 호출을 전달한다.

반면 React에서는 속성 이름을 camelCase인 `onClick`으로 쓰고, 중괄호 안에 함수 자체를 전달한다.

또한 HTML에서는 `return false`로 기본 동작을 막을 수 있지만, React에서는 반드시 `event.preventDefault()`를 호출해야 한다.

React 이벤트는 브라우저 이벤트 그대로가 아니라 `SyntheticEvent`라는 객체로 관리되며, 이를 통해 크로스 브라우저 이슈를 해결하고 성능을 최적화한다.

### 1. 속성 이름 (Attribute Naming)

- HTML: 소문자(onclick) 사용

```jsx
<button onclick="handleClick()"></button>
```

- React: camelCase(onClick) 사용

```jsx
<button onclick="handleClick()"></button>
```

React는 JSX를 사용하기 때문에 JavaScript의 관례를 따른다.

### 2. 함수 전달 방식

- HTML: 문자열로 함수 호출을 전달 → 브라우저가 실행
- React: 중괄호 `{}` 안에 함수 그 자체를 전달

```jsx
function handleClick() {
  console.log("Clicked!");
}
<button onClick={handleClick}>Click</button>;
```

React는 DOM에 직접 이벤트를 붙이지 않고, 합성 이벤트(Synthetic Event) 시스템을 통해 효율적으로 관리한다.

### 3. 기본 동작 막기

- HTML: `return false`로 가능

```jsx
function handleClick() {
  console.log("Clicked!");
}
<button onClick={handleClick}>Click</button>;
```

- React: 반드시 `event.preventDefault()` 호출

```jsx
function handleSubmit(e) {
  e.preventDefault(); // 기본 동작 취소
  console.log("Form submitted!");
}
<form onSubmit={handleSubmit}></form>;
```

---

### **Q. onClick={함수}와 onClick={함수()}의 차이를 설명해주세요**

`onClick={함수}`는 클릭 시점에 실행되지만, `onClick={함수()}`는 렌더링 시점에 실행된다.

그래서 매개변수를 전달할 때는 보통 화살표 함수로 감싸서 처리한다.

### 1. `onClick={함수}`

- 함수 그 자체를 전달하는 방식이다.
- 버튼이 클릭되면 React가 그 함수를 실행한다.

```jsx
function handleClick() {
  console.log("Clicked!");
}

<button onClick={handleClick}>Click</button>;
```

### 2. `onClick={함수()}`

- 컴포넌트가 렌더링되는 순간 함수를 실행한 뒤, 반환값을 이벤트 핸들러로 전달한다.
- 대부분의 경우 원하지 않는 동작이다.

```jsx
<button onClick={handleClick()}>Click</button>
```

이 경우 버튼이 보이자마자 함수가 실행되고, 버튼 클릭 시에는 아무 일도 안 일어난다.

- 왜 본인은 JavaScript와 React의 함수 개념 차이를 인지하지 못했을까?
  학습 과정에서 그렇게 배웠기 때문에. React에서는 `onClick={handleClick}`처럼 쓰는 게 기본 문법이라고만 배워서, 자연스럽게 “그런가 보다” 하고 받아들임.
  그래서 `{handleClick}`과 `{handleClick()}` 차이가 JavaScript 함수 실행 규칙에서 비롯된 것임을 깊게 생각하지 못했다.

---

### **Q. React에서 이벤트 전파를 막는 방법은 무엇인가요?**

React에서 이벤트 전파를 막으려면 `event.stopPropagation()`을 사용한다. 이 메서드는 이벤트가 부모 요소로 전파되는 것을 차단해 현재 요소까지만 동작하게 만든다. 예를 들어 부모와 자식 요소 모두에 클릭 이벤트가 있을 때, 자식 이벤트 핸들러에서 `stopPropagation()`을 호출하면 부모 이벤트는 실행되지 않는다.

`event.preventDefault()`는 전파를 막는 것이 아니라 요소의 기본 동작을 차단한다. 링크 이동이나 폼 제출 같은 브라우저 기본 동작을 취소할 때 사용하며, 전파 차단과는 별개의 개념이다. 따라서 `stopPropagation()`은 전파 차단, `preventDefault()`는 기본 동작 차단으로 구분해 기억해야 한다.

### 1. `event.stopPropagation()`

- 이벤트가 부모로 전파되는 걸 막는다.
- 즉, 현재 요소에서 이벤트 처리를 멈추고 더 이상 위로 올라가지 않음.

```jsx
function Parent() {
  function handleParentClick() {
    console.log("부모 클릭!");
  }

  function handleChildClick(e) {
    e.stopPropagation(); // 부모에게 이벤트 전달 X
    console.log("자식 클릭!");
  }

  return (
    <div onClick={handleParentClick}>
      <button onClick={handleChildClick}>Click me</button>
    </div>
  );
}
```

버튼을 눌러도 부모의 `onClick`은 실행되지 않는다.

### 2. `event.preventDefault()`

- 기본 동작(예: `<a>`의 링크 이동, `<form>`의 제출)을 막는 것.

```jsx
function handleSubmit(e) {
  e.preventDefault(); // form 제출 막기
  console.log("폼 제출 막음");
}
```

---

### **Q. onSubmit 이벤트를 사용할 때 주의할 점은 무엇인가요?**

`onSubmit` 이벤트를 사용할 때는 반드시 `event.preventDefault()`를 호출해 폼의 기본 동작(페이지 리로드)을 막아야 한다. 그렇지 않으면 React 앱이 전체 새로고침되면서 상태가 초기화되고 SPA 특성이 깨지게 된다.

또한 `onSubmit`은 단순히 `type="submit"` 버튼을 클릭할 때만 호출되는 것이 아니라, 폼 내부에서 Enter 키를 눌러도 실행된다. 따라서 의도치 않은 동작을 막으려면 이벤트 처리 흐름을 명확히 제어해야 한다.

```jsx
function handleSubmit(event) {
  event.preventDefault(); // 기본 제출 동작 방지
  // 제출 이벤트 처리 로직
}

<form onSubmit={handleSubmit}>
  {/* 폼 내용 */}
  <button type="submit">Submit</button>
</form>;
```

---

### **Q. 이벤트 객체에서 자주 사용하는 속성(e.target, e.currentTarget, e.type)의 차이를 설명하세요.**

`e.currentTarget`은 이벤트가 연결된 요소를 가리킨다. 즉, 내가 `onClick` 같은 이벤트 핸들러를 직접 달아둔 요소다. 반면 `e.target`은 실제로 이벤트가 발생한 요소를 의미한다. 예를 들어 부모 `<div>`에 클릭 이벤트를 걸어두고, 그 안의 버튼을 클릭하면 `e.target`은 버튼을 가리키고, `e.currentTarget`은 여전히 부모 `<div>`를 가리킨다.

`e.type`은 발생한 이벤트의 종류를 나타내는 문자열로, 클릭이면 `"click"`, 입력이면 `"change"` 같은 값이 들어온다.

```jsx
function handleClick(e) {
  console.log("target:", e.target.tagName); // 실제 클릭된 요소의 태그
  console.log("currentTarget:", e.currentTarget.tagName); // onClick을 단 요소의 태그
  console.log("type:", e.type); // 이벤트 종류
}

<div onClick={handleClick}>
  <button>버튼</button>
</div>;
```

버튼을 클릭하면 이렇게 출력된다.

- `"target: BUTTON"` (내 손가락이 닿은 태그)
- `"currentTarget: DIV"` (이벤트 핸들러를 단 태그)
- `"type: click"` (어떤 행동이 일어났는지(클릭, 입력 등)

- 이벤트 객체(e) 기본 개념 정리
  ### e.target
  - “진짜 클릭된 애” = 내가 실제로 마우스로 누른 요소
  - 함수명이나 onClick 이름이 아니라, 그 DOM 태그 자체를 가리킴.
  - 예시: `<div>` 안에 `<button>`이 있을 때 버튼을 클릭하면 `e.target` → `<button>`
  ***
  ### e.currentTarget
  - “이벤트를 연결한 애” = 내가 onClick 같은 핸들러를 달아둔 요소
  - 처음 `onClick={handleClick}`을 붙인 그 요소를 의미함.
  - 예시: `<div onClick={handleClick}> ... </div>`
    - 자식 `<button>`을 클릭해도 `e.currentTarget` → `<div>` (onClick을 달아둔 주체)
  ***
  ### e.type
  - “이벤트의 종류”를 문자열로 알려줌.
  - `<button>`을 클릭하면 `"click"`, `<input>`에 입력하면 `"change"`, 키보드를 누르면 `"keydown"`.
  - 즉, “무슨 행동 때문에 실행됐는가”를 알려줌.
  ***
  ### 정리 예시
  ```jsx
  <div onClick={handleClick}>
    <button>버튼</button>
  </div>
  ```
  - 버튼 클릭 시
    - `e.target` → `<button>` (진짜 누른 애)
    - `e.currentTarget` → `<div>` (이벤트를 연결한 애)
    - `e.type` → `"click"` (발생한 이벤트 종류)

---

### **Q. 키보드 이벤트(onKeyDown, onKeyPress, onKeyUp)의 차이를 설명하세요.**

`onKeyDown`은 키보드를 누르는 순간 실행되고, `onKeyUp`은 키에서 손을 뗄 때 실행된다. 이 두 이벤트는 문자·숫자뿐 아니라 `Ctrl`, `Shift`, `Alt`, 방향키, `F1~F12` 같은 특수키도 모두 인식한다.

반면 `onKeyPress`는 실제로 문자가 입력되는 상황에서만 실행된다. 그래서 알파벳, 숫자, 엔터 같은 일부 키만 감지하고, `Ctrl`, `Shift`, `Alt`, `F1~F12`, `Print Screen` 같은 제어 키는 인식하지 못한다.

또한 React 공식 문서에서는 `onKeyPress`가 더 이상 권장되지 않으며, 문자 입력 여부를 확인하고 싶다면 `onKeyDown`을 사용하는 것을 추천한다.

### 실행 결과 예시

```jsx
function KeyExample() {
  function handleKey(e) {
    console.log(e.type, e.key); // 이벤트 종류, 눌린 키
  }

  return (
    <input
      onKeyDown={handleKey}
      onKeyPress={handleKey}
      onKeyUp={handleKey}
      placeholder="키 입력해보세요"
    />
  );
}
```

- **`a` 키 누름**
  - `keydown a`
  - `keypress a`
  - `keyup a`
- **`Shift` 키 누름**
  - `keydown Shift`
  - `keyup Shift`
  - (`keypress` 없음 → 제어 키는 감지 안 됨)

---

### **Q. 마우스 이벤트(onClick, onMouseEnter, onMouseLeave)의 차이를 설명하세요.**

`onClick`은 요소를 클릭(마우스를 누르고 뗐을 때) 했을 때 실행된다. 버튼, 링크 같은 요소에서 가장 흔히 사용된다.

`onMouseEnter`는 마우스 포인터가 요소의 **경계 안으로 들어올 때** 실행된다. 이때 자식 요소로 마우스가 이동하더라도 추가 실행되지 않으며, 진입 순간에만 한 번 발생한다. 비슷한 이벤트인 `onMouseOver`는 자식 요소 위로 이동할 때마다 다시 발생하는데, React에서는 의도치 않은 중복 실행을 막기 위해 보통 `onMouseEnter`를 사용한다.

`onMouseLeave`는 마우스 포인터가 요소의 **경계 밖으로 나갈 때** 실행된다. 마찬가지로 자식 요소 사이 이동은 감지하지 않고, 해당 요소 영역을 완전히 벗어날 때 한 번만 실행된다. `onMouseOut`은 자식 요소 위로 이동해도 실행되므로 주의가 필요하다.

```jsx
function MouseExample() {
  function handleEvent(e) {
    console.log(e.type); // 발생한 이벤트 종류 출력
  }

  return (
    <divonClick={handleEvent}
      onMouseEnter={handleEvent}
      onMouseLeave={handleEvent}
      style={{ border: "2px solid black", padding: "20px", display: "inline-block" }}
    >
      영역 안에 마우스를 올리거나 클릭해보세요
    </div>
  );
}
```

---

실행 시나리오

- 마우스를 `div` 영역 안으로 가져가면 `"mouseenter"`
- 영역 안에서 클릭하면 `"click"`
- 마우스를 `div` 바깥으로 빼면 `"mouseleave"`

---

- `onClick`: 클릭 시 실행
- `onMouseEnter`: 영역 안으로 들어올 때 실행 (자식 이동 시 추가 실행 X, `onMouseOver`와 구분)
- `onMouseLeave`: 영역 밖으로 나갈 때 실행 (자식 이동 시 실행 X, `onMouseOut`과 구분)

---

### 추가적으로 공부한 부분

- `{}`/`()`는 왜 쓰나?
  ### `{}`는 왜 쓰나?
  - JSX 안에서는 문자 그대로의 문자열이 아니라, 자바스크립트 표현식을 넣고 싶을 때 중괄호 `{}`를 사용한다.
  - `{}`는 **"**여기 안은 JS 코드야**"** 라는 표시다.
  ```jsx
  const name = "Doyeon";
  <p>{name}</p>   // "Doyeon" 출력됨
  <p>name</p>     // 그냥 "name"이라는 글자 출력됨
  ```
  `onClick={handleClick}`에서 `{}`는 "이건 문자열이 아니라 자바스크립트 함수야"라고 React에 알려주는 것.
  ### `()`는 왜 쓰나?
  - `()`는 자바스크립트 함수 실행 연산자다.
  - `handleClick`은 "함수 그 자체"이고, `handleClick()`은 "그 함수를 지금 실행한 결과값"이다.
  ```jsx
  function sayHi() {
    return "Hi!";
  }

  console.log(sayHi); // 함수 정의 (f sayHi) 출력
  console.log(sayHi()); // "Hi!" 출력 (실행 결과)
  ```
  그래서 `onClick={handleClick}`은 **함수 자체를 전달**하는 것이고,
  `onClick={handleClick()}`은 **즉시 실행해서 결과를 전달**하는 것.
  ### 비유하면
  - `handleClick` = "리모컨"
  - `handleClick()` = "리모컨을 눌러버림 → 결과(채널 변경)가 나옴"
  - `onClick={handleClick}` = 버튼을 눌렀을 때 리모컨을 사용해라
  - `onClick={handleClick()}` = 버튼 보이자마자 리모컨 눌러버리고, 버튼 누를 때는 쓸 게 없음
  ***
  그러니까 핵심은
  - **`{}`는 JSX 안에서 JS 코드를 쓰기 위해 필요**
  - **`()`는 함수를 '실행'할 때 필요**
- `onClick={함수}` 식 자체 이해 (함수 호출은 그 함수식이 따로 정의되어있는것인가?)
  나의 질문:
  함수를 실행한다는 것? 예를 들어 onClick={gamestartbutton} 게임 스타트 버튼이란 게 식에 적혀있다면 이 함수를 실행해달라는 건가?
  그러면 실제 gamestartbutton 이라는 function gamestartbutton 블라블라 return 이렇게 코드가 따로 있었던거겠지?
  ### `onClick={gameStartButton}` 라고 쓴 경우
  - React는 버튼이 클릭됐을 때 `gameStartButton`이라는 \*\*\*\*함수를 실행해라 하고 기억해둔다.
  - 그래서 `gameStartButton`이라는 함수는 어딘가에 정의되어 있어야 한다.
  ```jsx
  function gameStartButton() {
    console.log("게임 시작!");
    // 여기서 게임 시작 관련 로직을 적음
  }

  <button onClick={gameStartButton}>시작</button>;
  ```
  이 경우, 버튼을 클릭했을 때 `console.log("게임 시작!")` 이 실행된다.
  ### 그럼 `gameStartButton()` 은?
  - 이건 “그 함수를 지금 당장 실행해서 나온 결과를 넣어라”는 의미.
  - 버튼 보이자마자 실행돼버린다.
  ```jsx
  function gameStartButton() {
    console.log("게임 시작!");
  }

  <button onClick={gameStartButton()}>시작</button>;
  ```
  이렇게 쓰면 버튼이 화면에 나타나는 순간 이미 `console.log("게임 시작!")`이 찍히고, 클릭해도 아무 반응이 없다.
  ***
  ### ✅ 헷갈릴 때 이렇게 생각하기
  - `gameStartButton` → 함수 "리모컨" 자체를 넘겨준다. (클릭하면 실행)
  - `gameStartButton()` → 리모컨을 지금 눌러버린다. (렌더링 시 실행 끝)
  ***
  말씀하신 것처럼, 실제로는 **`function gameStartButton() { … return … }`** 형태로 어딘가에 함수가 정의되어 있어야 하고, 그걸 onClick 속성에 연결해주는 것이다.

---

### **학습에 참고한 자료**

- [github.io](http://github.io) - https://2ssue.github.io/common_questions_for_Web_Developer/docs/Framework/react_event_handling.html#%E1%84%8B%E1%85%A1%E1%86%AF%E1%84%8B%E1%85%A1%E1%84%83%E1%85%AE%E1%84%86%E1%85%A7%E1%86%AB-%E1%84%8C%E1%85%A9%E1%87%82%E1%84%8B%E1%85%B3%E1%86%AB-%E1%84%80%E1%85%A5%E1%86%BA
- 티스토리 - https://wouldyou.tistory.com/9
- 티스토리 - https://codeyun2.tistory.com/entry/React-stopPropagation-preventDefault?utm_source=chatgpt.com
- 티스토리 - https://shawnkim.tistory.com/18
- 티스토리 - https://5ffthewall.tistory.com/59
- 티스토리 - https://fromnowwon.tistory.com/entry/currentTarget
- 티스토리 - https://itprogramming119.tistory.com/entry/React-onKeyDown-onKeyUp-onKeyPress-%EC%B0%A8%EC%9D%B4
- 네이버 블로그 - https://blog.naver.com/tombyun/223065996562

---
