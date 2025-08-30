### Q. React에서 State와 Props의 차이점을 설명해주세요.

State와 Props는 모두 JavaScript의 객체이며, 렌더링 결과물에 영향을 주는 데이터를 담는다.  
하지만 관리 방식과 용도가 다르다.

Props는 함수 매개변수처럼 컴포넌트에 전달되는 값이고,  
State는 함수 내부 변수처럼 컴포넌트 안에서 선언·관리되는 값이다.

|             | **props**               | **state**               |
| ----------- | ----------------------- | ----------------------- |
| 변경 가능성 | 읽기 전용¹              | 변경 가능               |
| 관리 위치   | 부모 컴포넌트           | 해당 컴포넌트 내부      |
| 용도        | 컴포넌트 간 데이터 전달 | UI 상태를 동적으로 변경 |

---

### Q. 어떤 데이터를 State로 관리해야 하고, 어떤 데이터는 Props로 전달해야 하나요?

- **State** → 시간에 따라 변화하는 값, 즉시 UI에 반영되어야 하는 값  
  예: 버튼을 누를 때마다 숫자가 올라가는 카운터 앱의 `count` 값

- **Props** → 부모가 소유한 데이터를 자식에게 전달할 때  
  예: 부모가 API에서 받아온 사용자 이름을 `<Profile name="도연" />`으로 전달

---

### Q. setState 함수 대신 state를 직접 수정하면 안 되는 이유는 무엇인가요?

State를 직접 수정하면 React가 변화를 감지하지 못해 **리렌더링이 일어나지 않는다.**  
즉, 데이터는 바뀌었는데 화면(UI)은 그대로라서 UI와 실제 데이터가 불일치하는 문제가 발생한다.

따라서 반드시 `setState`를 사용해야 한다.

> 💡 **객체 형태로 관리되는 state**  
> 객체 내부 값을 직접 수정하면 기존 객체의 참조(주소)가 유지된다.  
> React는 참조가 동일하면 “아무 일도 일어나지 않았다”고 판단하기 때문에 리렌더링하지 않는다.  
> → 따라서 state는 반드시 **불변성(immutability)²**을 지켜야 한다.

---

### Q. props를 변경하려고 하면 에러가 나는 이유는 무엇인가요?

Props는 **읽기 전용¹**이기 때문에 직접 수정할 수 없다.

이는 데이터 일관성을 보장하고, 부모 컴포넌트가 데이터의 소유자임을 명확히 하기 위해 설계된 원칙이다.  
만약 자식에서 직접 바꿀 수 있다면 데이터 출처가 불분명해지고 흐름이 복잡해진다.

자식이 값을 변경해야 한다면, 그 값은 부모의 state로 관리되어야 한다.  
자식은 부모가 내려준 함수를 호출해 간접적으로 상태를 바꾸고,  
부모가 다시 변경된 값을 props로 내려준다.

```jsx
// 부모 컴포넌트
function App() {
  const [count, setCount] = useState(0);

  return <Counter count={count} onIncrement={() => setCount(count + 1)} />;
}

// 자식 컴포넌트
function Counter({ count, onIncrement }) {
  return <button onClick={onIncrement}>{count}</button>;
}
```

---

### Q. 부모 state를 자식이 업데이트해야 할 경우 어떻게 하나요?

이 경우 **상태 끌어올리기(Lifting State Up)** 패턴을 사용한다.

- **상태 정의** → 부모가 상태를 "소유"
- **props 전달** → 자식에게 상태 값과 변경 함수 전달
- **자식에서 호출** → 자식은 전달받은 함수를 실행해 간접적으로 변경
- **리렌더링** → 상태 변경 시 부모와 자식이 모두 새로운 상태 반영

---

### 각주

1. **읽기 전용**: 수정 불가능한 상태. C#의 `readonly`, `<input readonly>`, 파일 시스템 속성 등
2. **불변성(immutability)**: 기존 값을 수정하지 않고, 새로운 값을 만들어 교체하는 관리 방식

---

### 학습에 참고한 자료

- React 한글 문서 - [컴포넌트 state](https://ko.legacy.reactjs.org/docs/faq-state.html?utm_source=chatgpt.com)
- React 공식 문서 - [State: 컴포넌트의 기억 저장소](https://ko.react.dev/learn/state-a-components-memory?utm_source=chatgpt.com)
- 티스토리 - [리액트는 어떤 흐름으로 실행될까? state 이해하기](https://dev-dev-dev-dev.tistory.com/5?utm_source=chatgpt.com)
- 티스토리 - [React의 핵심 개념: Props와 State의 차이점 이해하기](https://susuwiki.tistory.com/entry/React%EC%9D%98-%ED%95%B5%EC%8B%AC-%EA%B0%9C%EB%85%90-Props%EC%99%80-State%EC%9D%98-%EC%B0%A8%EC%9D%B4%EC%A0%90-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0?utm_source=chatgpt.com)
- 브런치스토리 - [React.setState를 사용하지 않는 3가지 이유](https://brunch.co.kr/@hee072794/108?utm_source=chatgpt.com)
- 위키독스넷 - [프롭(props)](https://wikidocs.net/257806?utm_source=chatgpt.com)
- 벨로그 - [React] 상태 끌어올리기 (Lifting State Up)](https://velog.io/@yeonjin1357/React-%EC%83%81%ED%83%9C-%EB%81%8C%EC%96%B4%EC%98%AC%EB%A6%AC%EA%B8%B0-Lifting-State-Up?utm_source=chatgpt.com)
