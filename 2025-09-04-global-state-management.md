# 09-04 기술 면접 주제: 전역 상태 관리

---

### **Q. 상태 업데이트가 비동기적으로 처리되는 이유는 무엇인가요?**

React의 상태 업데이트가 비동기적으로 동작하는 이유는 React가 효율적으로 렌더링 하기 위해 여러 개의 상태값 변경 요청을 일괄 처리하기 때문이다.

하나의 페이지나 컴포넌트 내에도 수많은 상태값이 존재하는데 만약 리렌더링을 발생하는 setState가 일어날 때마다 리렌더링이 일어난다면 성능 이슈가 생길 수 있다.

---

### **Q. props drilling이란 무엇이며, 어떤 문제를 일으키나요?**

Props Drilling은 React 컴포넌트에서 하위 컴포넌트로 계속해서 props를 전달하는 과정에서 발생하는 문제다. 부모 컴포넌트에서 자식 컴포넌트로 props를 하나씩 전달하게 되면, 중간에 있는 컴포넌트들까지 해당 props를 계속 전달해야 하는 번거로운 상황이 발생할 수 있다. 이 방식이 계속 이어지면 컴포넌트 계층이 깊어질수록 코드가 복잡하고 유지보수가 어려워진다.

이미지 출처: [https://disco-biscuit.tistory.com/185](https://disco-biscuit.tistory.com/185)

---

### **Q. Context API는 무엇인가요?**

React의 Context API는 컴포넌트 트리 전체에 데이터를 전역적으로 제공하거나, 여러 컴포넌트 간에 데이터를 쉽게 공유하기 위해 사용되는 기능이다.

일반적으로 부모 → 자식으로만 데이터를 전달할 수 있기 때문에 Props Drilling 문제가 발생한다. (데이터가 필요 없는 중간 컴포넌트에도 props를 계속 전달해야 하는 상황)

이때 Context API를 사용하면 전역적으로 데이터를 관리할 수 있어 불필요한 props 전달을 줄이고, 필요한 컴포넌트에서 바로 값을 받아올 수 있다.

Context API는 다음과 같은 요소로 구성된다.

1. **`React.createContext()`**
   - Context 객체를 생성한다.
   - 기본값(default value)을 설정할 수 있다.
2. **`Context.Provider`**
   - Context 값을 하위 컴포넌트에 제공한다.
   - Provider 하위의 모든 컴포넌트에서 값을 사용할 수 있다.
3. **`Context.Consumer`** 또는 **`useContext`** Hook
   - 하위 컴포넌트에서 Context의 값을 소비한다.
   - 함수형 컴포넌트에서는 `useContext(Context)`를 사용하여 간단히 접근할 수 있다.

```jsx
// 1. Context 생성
const ThemeContext = React.createContext("light");

function App() {
  return (
    // 2. Provider로 전역 값 제공
    <ThemeContext.Provider value="dark">
      <Toolbar />
    </ThemeContext.Provider>
  );
}

function Toolbar() {
  return <ThemedButton />;
}

function ThemedButton() {
  // 3. useContext로 값 소비
  const theme = React.useContext(ThemeContext);
  return <button>{theme} mode</button>;
}
```

실행하면 `ThemedButton`에서 "dark mode"가 출력된다.

---

### **Q. Context API의 동작 원리와 장단점은 무엇인가요?**

Context API는 `Provider`를 통해 전역 값을 내려주고, 하위 컴포넌트가 `useContext`로 해당 값을 직접 꺼내 쓰는 방식으로 동작한다. 이 덕분에 Props Drilling 문제를 해결하고 전역 상태 관리가 간단해지는 장점이 있다. 다만 Context 값이 바뀌면 해당 값을 구독하는 모든 컴포넌트가 리렌더링되기 때문에, 불필요한 렌더링이 발생할 수 있다는 단점도 있다. 따라서 단순한 전역 데이터(테마, 인증, 언어 설정 등)에는 유용하지만, 복잡한 상태 관리에는 Redux나 Zustand 같은 라이브러리가 더 적합하다.

### 장점

- Props Drilling 해결
- 중첩된 컴포넌트 계층을 통해 데이터를 게속 전달하지 않아도 됨
- 전역 상태 관리 간소화
- React 내장 기능
- 추가적인 라이브러리 설치 미필요

### 단점

- 불필요한 리렌더링이 발생
- Context의 값이 변경되면 해당 값을 사용하는 모든 컴포넌트가 다시 렌더링 됨
  - 이를 해결하려면 Context를 분리하거나 메모이제이션을 활용해야 함
- 여러 Context를 사용해야하는 경우 코드가 복잡해짐
- 상태 관리의 한계 (복잡한 상태는 라이브러리가 필수화 될 지도)

### 활용 사례

- 테마 관리: 라이트/다크 모드 전환
- 사용자 인증 정보: 로그인 상태 관리
- 언어 설정: 다국어 지원

---

### **Q. Context API만으로 전역 상태를 관리하기 어려운 이유는 무엇인가요?**

Context API는 전역 상태를 관리하고 여러 컴포넌트 간에 상태를 공유하는 데에 매우 유용하지만, 비동기 작업(API 호출 등)을 처리할 때는 그 자체로 특별한 도구를 제공하지 않는다. 이로 인해 비동기 로직을 직접 구현해야 하며, 상태 관리와 비동기 작업을 함께 처리할 때 코드가 복잡해질 수 있다.

### 문제점

- 비동기 로직의 처리: 비동기 로직(API 호출 등)을 처리하는 코드를 Context API에서 직접 작성하면 상태 변경과 비동기 작업이 뒤섞여 코드가 복잡해지고 관리하기 어려워진다.
- 상태 관리 분리 부족: 상태 관리 로직과 비동기 로직을 잘 분리하는 것이 중요한데, Context API에서는 이를 위한 내장 기능이 부족하다. 따라서 개발자가 비동기 작업에 필요한 상태(예: 로딩 상태, 에러 상태 등)를 수동으로 추가하고 관리해야한다.

```jsx
// 비동기 API 호출을 처리하는 Context
export const fetchData = async (dispatch) => {
  try {
    dispatch({ type: "LOADING" });
    const data = await fetch("https://api.example.com/data");
    const result = await data.json();
    dispatch({ type: "SUCCESS", payload: result });
  } catch (error) {
    dispatch({ type: "ERROR", payload: error });
  }
};
```

---

### **Q. Recoil, Zustand 같은 경량 상태 관리 라이브러리가 Redux와 다른 점은 무엇인가요?**

### 1. **`Redux`**

- **가장 오래되고 검증된 상태 관리 라이브러리, 대규모 프로젝트에 적합**
- 상태가 하나의 Store에 모여 있어 예측 가능하고, 디버깅·테스트에 강함
- Redux Toolkit 덕분에 복잡한 코드 문제(보일러플레이트)가 크게 줄어듦
- 다만 처음 배우기 어렵고, 작은 프로젝트에는 무겁게 느껴질 수 있음

### 2. **`Recoil`**

- `useState`처럼 직관적인 API를 제공해 **배우기 쉬움**
- 비동기 상태 관리와 파생 상태 처리에 강점
- 아직 실험적인 단계라, 커뮤니티와 생태계는 Redux만큼 크지 않음

### 3. **`Zustand`**

- Hooks 기반으로 **매우 가볍고 간단하게 사용 가능**
- Redux DevTools 연동으로 디버깅 편리
- 구조화된 패턴을 강제하지 않아 자유롭지만, 팀 단위 개발에서는 일관성이 떨어질 수 있음

**npm trends를 통해 실제 패키지 사용 추세 확인**
[https://npmtrends.com/react-redux-vs-recoil-vs-zustand](https://npmtrends.com/react-redux-vs-recoil-vs-zustand)

- react-redux는 여전히 가장 많이 쓰이는 전통적인 상태 관리 라이브러리이다.
- 그러나 최근 몇 년 사이에 zustand의 성장세가 빠르게 올라가며, 경량 상태 관리의 선호가 늘고 있음을 확인할 수 있다.
- recoil은 꾸준히 사용되고 있지만 성장세는 상대적으로 완만하다.

- **나의 선택?**
  여러 라이브러리를 리서치한 뒤, 채용 플랫폼에서 자주 보였던 Recoil과 Zustand에 관심이 갔지만, 전통적으로 많이 사용되는 Redux으로 마음이 기울었다. 다만 실제 트렌드를 확인해보니 Zustand의 성장세가 빠르게 Redux를 따라가고 있어 인상 깊었다.
  2차 프로젝트는 Zustand를 사용하기로 결정했다. (아마도…🤔)

---

### **Q. React Query와 Redux 같은 전역 상태 관리 라이브러리의 역할 차이는 무엇인가요?**

| 구분                 | Redux                                           | React Query                                          |
| -------------------- | ----------------------------------------------- | ---------------------------------------------------- |
| **주된 목적**        | 클라이언트 측 상태 관리 (UI 상태, 인증 상태 등) | 서버 상태 관리 (API 호출, 데이터 캐싱, 동기화)       |
| **데이터 관리 대상** | 모든 종류의 상태 (동기/비동기, 내부 상태)       | 서버와 통신하는 비동기 데이터 (REST, GraphQL 등)     |
| **전역 상태 관리**   | 하나의 스토어에 저장, 전역 접근 가능            | 자동 캐싱/리패칭, 서버 상태 중심 관리                |
| **설정 복잡성**      | 전통 Redux는 복잡 → Redux Toolkit으로 단순화    | 상대적으로 단순, 기본 기능 내장                      |
| **비동기 처리**      | 미들웨어(thunk, saga) 필요, 로딩/에러 수동 관리 | 로딩/에러 처리 자동화, 무효화·백그라운드 리패칭 제공 |

- Redux: 앱 내부에서 생기는 클라이언트 상태 관리 (UI 중심)
- React Query: 서버에서 가져온 비동기 데이터(서버 상태) 관리

둘 다 쓸 수 있지만, Redux로 서버 상태를 관리하면 괜히 복잡해지고, React Query로 클라이언트 상태를 관리하려면 맞지 않지 않아서 같이 쓰는 게 좋음

---

### **학습에 참고한 자료**

- 위키독스 - https://wikidocs.net/273969
- 티스토리 - https://disco-biscuit.tistory.com/185
- 티스토리 - https://hsly22xk.tistory.com/m/379
- 티스토리 - https://sy-blog.tistory.com/210
- [velog.io](http://velog.io) - https://velog.io/@iberis/%EC%83%81%ED%83%9C%EA%B4%80%EB%A6%AC-%EB%9D%BC%EC%9D%B4%EB%B8%8C%EB%9F%AC%EB%A6%AC-%EB%B9%84%EA%B5%90-Redux-vs-Recoil-vs-Zustand-vs-Jotai
- npm trends - https://npmtrends.com/react-redux-vs-recoil-vs-zustand

---
