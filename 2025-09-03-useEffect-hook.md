# 09-03 기술 면접 주제: useEffect

---

### **Q. useEffect는 무엇인가요?**

`useEffect`는 컴포넌트가 렌더링된 이후에 실행되어야 하는 부수 효과를 처리하는 훅이다.

React는 전달된 함수를 기억했다가, DOM 업데이트가 끝난 후 호출한다.

- 첫 번째 렌더링 후와, 의존성 배열(dependencies)이 변경될 때 실행된다.
- 의존성 배열을 `[]`로 두면 마운트 시 한 번만 실행된다.
- 주로 데이터 페칭, 구독 설정/해제, DOM 직접 조작, 외부 라이브러리 연동 같은 작업에 사용된다.

### **사진 변경 버튼 예시**

- 버튼을 눌러 프로필 사진을 바꾸면, 서버에서 새 이미지를 가져와야 함 → 서버 호출은 `useEffect` 안에서 실행.

```jsx
useEffect(() => {
  fetch("/api/profile-img")
    .then(res => res.json())
    .then(data => setImg(data.url));
}, [userId]); // userId가 바뀔 때마다 새로 불러옴
```

### **회원정보 수정 화면**

- 수정 페이지 들어오면 처음에 기존 회원정보를 서버에서 불러와야 함 → 이 로딩 로직을 `useEffect`에서 실행.

```jsx
useEffect(() => {
  fetch(`/api/user/${userId}`)
    .then(res => res.json())
    .then(data => setForm(data));
}, [userId]);
```

### **DOM 직접 조작 (예: 포커스 이동)**

- 화면이 뜨면 자동으로 입력창에 커서가 가야 함 → DOM 접근은 렌더링 후라서 `useEffect`에서 실행.

```jsx
useEffect(() => {
  inputRef.current.focus();
}, []); // 처음 한 번만 실행
```

---

### **Q. useEffect는 언제 실행되나요? (렌더링 과정에서 실행 순서 포함)**

`useEffect`는 컴포넌트가 실제 DOM에 반영된 이후 실행된다. 즉, 렌더링 과정 중에 바로 실행되는 것이 아니라, 화면 업데이트가 완료된 다음 단계에서 호출된다.

### 실행 순서

1. 컴포넌트 함수 실행 → JSX 반환
2. 가상 DOM 생성 → 실제 DOM 업데이트
3. 그 다음에 useEffect 내부 로직 실행
4. 다음 렌더링 전에는, 이전 effect의 cleanup 함수가 먼저 실행됨

```jsx
useEffect(() => {
  console.log("DOM 업데이트 후 실행");

  return () => {
    console.log("다음 렌더링 전이나 컴포넌트 사라질 때 실행 (cleanup)");
  };
}, [count]);
```

- 첫 렌더링 직후: 의존성 배열이 없거나 `[]`일 때는 마운트 시 한 번 실행된다.
- 업데이트 시: 의존성 배열에 지정한 값이 바뀌면, cleanup → 새 effect 순으로 실행된다.
- 언마운트 시: cleanup 함수가 실행되어 이벤트 해제, 타이머 정리 등을 한다.

- 렌더링 / 언마운트 / 클린업
    
    ### 렌더링 (Rendering)
    
    React에서 렌더링은 “컴포넌트 함수가 실행돼서 JSX가 실제 화면(DOM)에 반영되는 것”을 말한다.
    
    - 예: 버튼을 클릭해서 `count`가 0 → 1로 바뀌면, React가 다시 컴포넌트 함수를 실행 → 새로운 DOM 만들어서 바뀐 부분만 업데이트해준다.
    
    ### 언마운트 (Unmount)
    
    언마운트는 말 그대로 “컴포넌트가 화면에서 사라지는 순간”이다.
    
    - 예:조건부 렌더링에서 `isOpen ? <Modal /> : null` → `isOpen`이 false 되면 `<Modal />`이 화면에서 사라짐 → 이게 언마운트.
    - 페이지 이동하면서 다른 컴포넌트로 바뀌는 경우도 언마운트.
    
    ### 클린업 (Cleanup)
    
    클린업은 “useEffect 안에서 등록해둔 부수 효과를 정리(clean up)하는 것”이다.
    
    왜 필요하냐면, 안 그러면 메모리 누수나 중복 실행 같은 문제가 생긴다.
    
    ```jsx
    useEffect(() => {
      const id = setInterval(() => {
        console.log("매초 실행됨");
      }, 1000);
    
      // cleanup 함수
      return () => {
        clearInterval(id); // 타이머 정리
        console.log("타이머 해제됨");
      };
    }, []);
    
    ```
    
    - 마운트 → effect 실행 → 타이머 시작
    - 언마운트 → cleanup 실행 → 타이머 해제

---

### **Q. dependency array가 없을 때, 빈 배열일 때, 특정 값이 들어갔을 때 각각 어떻게 동작하나요?**

- 배열 없음 **(`useEffect(() => {...})`)**
    
    → 컴포넌트가 마운트된 직후 + 이후 모든 리렌더링마다 실행된다.
    
- 빈 배열 **(`useEffect(() => {...}, [])`)**
    
    → 컴포넌트가 마운트될 때 한 번만 실행된다. (언마운트 시 cleanup은 실행됨)
    
- 특정 값 있음 **(`useEffect(() => {...}, [value])`)**
    
    → 컴포넌트가 마운트될 때 한 번 실행되고, 그 후에는 해당 값(`value`)이 바뀔 때마다 실행된다.
    

```jsx
// case 1: 배열 없음
useEffect(() => {
  console.log("모든 렌더링마다 실행됨");
});

// case 2: 빈 배열
useEffect(() => {
  console.log("마운트될 때 한 번만 실행");
}, []);

// case 3: 특정 값
useEffect(() => {
  console.log("count가 바뀔 때마다 실행");
}, [count]);
```

- 배열 없음 → 매 렌더링마다
- 빈 배열 → 처음 한 번만
- 값 있음 → 해당 값이 바뀔 때마다

- **🍔 햄버거집 비유**
    
    
    - 컴포넌트 = 햄버거 가게
    - 렌더링 = 손님 들어오고, 주문 받고, 음식이 나옴 (화면에 뭔가 새로 그림)
    - useEffect = 가게 끝나고 하는 뒷정리 작업 (청소, 재고 체크 등)
    - dependency array = “언제 뒷정리 할지” 조건표
    
    ### 1. dependency array 없음
    
    ```jsx
    useEffect(() => {
      console.log("청소!");
    });
    ```
    
    - 가게 문 열고 손님 올 때마다 청소를 함.
    - 즉, 처음 문 열 때도 하고, 손님 한 명 올 때마다 계속 또 함.
    
    → “마운트될 때 + 모든 리렌더링마다 실행”
    
    ### 2. 빈 배열 `[]`
    
    ```jsx
    useEffect(() => {
      console.log("청소!");
    }, []);
    ```
    
    - 가게 오픈 첫날에만 청소 딱 한 번 함.
    - 이후에 손님 몇 번 와도 추가로 청소 안 함.
    
    → “마운트될 때 한 번만 실행”
    
    ### 3. 특정 값 있을 때 `[count]`
    
    ```jsx
    useEffect(() => {
      console.log("청소!");
    }, [count]);
    ```
    
    - 특정 상황(count 값이 변할 때)만 청소
    - 예: `count` = 손님 수
    - 손님 수가 늘어나거나 줄어들면 “아 손님 수 바뀌었네?” 하고 청소 실행.
    
    → “마운트될 때 한 번 실행 + count 값이 바뀔 때마다 실행”
    
    ### 정리
    
    - 배열 없음 → 계속 실행 (모든 렌더링마다)
    - 빈 배열 → 처음에만 한 번
    - 값 있음 → 그 값이 바뀔 때만
- 헷갈리는 부분 정리
    
    ### 1️⃣ 렌더링 vs 실행
    
    - `useEffect` 안에 있는 코드는 **렌더링이 끝난 후에 실행되는 것이디**.
    - 그러니까 **렌더링이 많아진다**라고 하기보다는 **effect 실행이 많아진다**라고 하는 게 정확함.
    - 렌더링은 state/props가 바뀔 때 일어나는 거고, 그때마다 effect도 같이 실행될 수 있음.
    
    ### 2️⃣ 의존성 배열 없는 경우
    
    - 맞아, **모든 렌더링마다 effect 실행**이라서 필요 이상으로 많이 실행될 수 있음.
    - 데이터 요청이나 이벤트 등록 같은 무거운 작업을 매번 실행하면 비효율적.
    
    ### 3️⃣ 빈 배열인 경우
    
    - **처음 마운트 때만 실행**이라서, 변화가 생겨도 effect가 반응하지 않는다.
    - 그래서 “변화를 감지 못한다”라는 표현이 맞다.
    - 예: 회원정보 페이지 들어올 때 최초 1번만 API 요청 → 이후엔 반응 없음.
    
    ### 4️⃣ 특정 값 있는 경우
    
    - **그 값이 변할 때마다 실행**돼서 실제로 가장 많이 쓰인다.
    - 예:
        - `userId`가 바뀌면 → 해당 유저 프로필 다시 불러오기
        - `keyword`가 바뀌면 → 검색 API 다시 호출하기
    
    ```jsx
    useEffect(() => {
      fetch(`/api/user/${userId}`)
        .then(res => res.json())
        .then(data => setProfile(data));
    }, [userId]);
    ```
    
    ### 결론
    
    - 배열 없음 → 거의 안 씀 (비효율적)
    - 빈 배열 → “처음에 한 번만” 필요한 경우 (ex. 초기 데이터 불러오기)
    - 특정 값 → 가장 실용적, 값이 바뀔 때마다 실행되니까 동적인 상황 처리 가능
    
    **“**프로필 사진이 변경됨 → userId나 photoVersion 같은 값 바뀜 → 그걸 dependency array에 넣음 → 자동으로 API 다시 호출**”** 이런 패턴이 바로 실전에서 제일 많이 쓰는 흐름
    

---

### **Q. dependency array에서 객체, 배열, 함수를 직접 넣으면 왜 문제가 생길 수 있나요?**

React에서 의존성 배열은 내부적으로 얕은 비교만 한다.

`===` 비교로 “같은 값인가?” 확인하는데, 객체/배열/함수는 참조값(주소)가 매번 달라져서 비교에 걸려버린다.

- 객체/배열/함수는 리렌더링 때마다 새로 생성되기 때문에, 실제 내용이 같더라도 `prev !== next`로 판정됨.
- 그 결과, 의존성 배열에 넣으면 매번 값이 바뀐 걸로 인식 → `useEffect`가 매 렌더링마다 실행됨.
- 이게 데이터 요청이나 이벤트 등록 같은 무거운 작업일 경우 무한 실행 / 성능 저하 문제가 생길 수 있다.

```jsx
// ❌ 문제되는 코드
useEffect(() => {
  console.log("매번 실행됨!");
}, [{}]); // 객체를 직접 넣음
```

- `{}`는 렌더링 때마다 새로운 객체라서 매번 바뀌었다고 인식 → 무한 실행

```jsx
// ✅ 해결법: useMemo / useCallback으로 메모이제이션
const memoizedObj = useMemo(() => ({}), []);
useEffect(() => {
  console.log("처음 한 번만 실행됨!");
}, [memoizedObj]);
```

```jsx
const handleClick = useCallback(() => {
  console.log("클릭");
}, []);
useEffect(() => {
  console.log("함수가 바뀔 때만 실행됨");
}, [handleClick]);
```

---

### **Q. 여러 개의 useEffect가 있을 때 실행 순서는 어떻게 되나요?**

`useEffect`는 렌더링 직후, 비동기적으로 실행된다.

`setState`로 상태를 바꾸면 바로 실행되는 게 아니라, DOM 업데이트가 끝난 뒤에 실행된다.

여러 개의 `useEffect`가 있다면, 작성된 순서대로(위 → 아래) 실행된다.

이 순서는 컴포넌트 파일 안에 적힌 코드 순서를 따른다.

```jsx
useEffect(() => {
  console.log("첫 번째 useEffect");
});

useEffect(() => {
  console.log("두 번째 useEffect");
});

// 출력 결과

첫 번째 useEffect
두 번째 useEffect
```

- `useEffect`는 비동기적으로 실행된다.
- 여러 개 있을 경우 작성 순서대로 실행된다.
- 따라서 순서가 중요한 로직이라면 의존성 배열과 분리된 `useEffect`를 잘 관리하거나, 필요 시 하나의 `useEffect` 안에서 실행 순서를 제어해야 한다.

---

### **Q. useEffect의 cleanup 함수는 무엇이고 언제 실행되나요?**

`useEffect`의 cleanup 함수는 effect에서 등록한 부수 효과(side effect)를 정리하는 함수이다.

쉽게 말해, 컴포넌트가 사라지거나, 다음 effect가 실행되기 전에 이전 effect를 정리해주는 함수이다.

### **실행 시점**

1. 컴포넌트가 언마운트될 때 → 메모리 누수, 이벤트 중복 방지
2. 의존성 배열 값이 바뀔 때 → 새 effect 실행 전에 이전 effect를 정리

### **cleanup이 필요한 경우**

- `setInterval`, `setTimeout` 등 등록한 작업을 해제할 때 (`clearInterval`, `clearTimeout`)
- `window.addEventListener` 같은 이벤트 리스너 제거
- 외부 라이브러리 인스턴스 해제

### **cleanup이 필요 없는 경우**

- 한 번 실행만 하면 되는 작업 (DOM 수동 조작, 네트워크 요청, 로깅 등)

```jsx
useEffect(() => {
  const id = setInterval(() => {
    console.log("매초 실행");
  }, 1000);

  // cleanup
  return () => {
    clearInterval(id);
    console.log("타이머 해제됨");
  };
}, []);
```

- cleanup 함수 = effect의 뒷정리 역할
- 실행 시점 = 언마운트 시 + 의존성 변경 시 새 effect 실행 직전
- 주로 타이머, 이벤트 리스너, 구독/라이브러리 해제에 사용

---

### **Q. useEffect와 useLayoutEffect의 차이는 무엇인가요?**

둘 다 렌더링 후 실행되는 훅이지만, 실행 시점이 다르다.

- **useEffect**
    - 비동기적으로 실행됨 → DOM 업데이트와 브라우저 화면 그리기가 끝난 뒤 실행
    - 주로 데이터 요청, 구독 설정, 로깅 같은 작업에 적합
- **useLayoutEffect**
    - 동기적으로 실행됨 → DOM 업데이트가 끝나자마자, 브라우저가 화면을 그리기 전에 실행
    - 주로 DOM을 측정하거나, DOM 스타일을 조작해야 하는 경우에 적합

```jsx
useEffect(() => {
  console.log("화면이 그려진 뒤 실행");
});

useLayoutEffect(() => {
  console.log("화면이 그려지기 직전에 실행");
});
```

- useEffect → 비동기, “보이는 화면이 먼저 → 성능 유리
- useLayoutEffect → 동기, 보이기 전에 조정 → 깜빡임 방지

---

### **학습에 참고한 자료**

- [velog.io](http://velog.io) - https://velog.io/@clydehan/React-useEffect-%EC%96%B8%EC%A0%9C-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B3%A0-%EC%96%B8%EC%A0%9C-%ED%94%BC%ED%95%B4%EC%95%BC-%ED%95%A0%EA%B9%8C
- 티스토리 - https://shinbian11.tistory.com/103
- 티스토리 - https://funveloper.tistory.com/193#google_vignette
- 티스토리 - https://wnsdufdl.tistory.com/454
- f-lab - https://f-lab.kr/insight/understanding-useeffect-and-uselayouteffect-in-react-20240618?gad_source=1&gclid=Cj0KCQiA7NO7BhDsARIsADg_hIZz6PPo1nVm9w17vgSNIBBnU419jlhhpFgzupigi-fiTVnmreaPuKYaAlWcEALw_wcB

---