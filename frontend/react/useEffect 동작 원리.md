# useEffect 동작 원리

## useEffect란?

> 💡 _useEffect는 컴포넌트를 외부 시스템과 동기화 할 수 있는 React Hook이다._
> — [React Docs: useEffect](https://react.dev/reference/react/useEffect)

- `useEffect`는 React 컴포넌트의 라이프사이클 시점에 맞춰 실행될 부수 효과를 등록하는 Hook이다.

## useEffect Hook의 동작 원리

### 기본 구조
`useEffect`는 두 개의 인자를 받는다:
```javascript
useEffect(effect, deps);
```

### 인자
1. **이펙트 함수 (setup 함수)**
   - Effect의 로직을 포함하는 함수
   - 렌더링이 끝난 후 실행되며, DOM 업데이트가 완료된 뒤 동작
   - cleanup 함수를 반환할 수 있음 (선택사항)

2. **dependency 배열 (deps)**
   - 코드 내부에서 참조되는 모든 값들을 배열로 전달
   - props, state, 그 외 참조되는 모든 값들 포함
   - React는 `Object.is`를 사용하여 이전 값과 비교

### 실행 시점과 조건
- **기본 동작**: 컴포넌트의 첫 번째 렌더링 이후 실행, 이후 모든 업데이트에서 수행
- **실행 시점**: DOM이 업데이트된 이후 (비동기적으로 실행)

### dependency 배열에 따른 동작
- **빈 배열 `[]`**: 컴포넌트 mount 시점에 한 번만 실행
- **배열 생략**: 컴포넌트가 리렌더링될 때마다 실행
- **값 포함**: 지정된 값이 변경될 때만 실행

### useEffect가 왜 필요할까?

React 함수 컴포넌트는 렌더링 될 때마다 함수가 다시 실행된다.
즉, 그 안에서 정의된 함수들은 그 시점의 props와 state를 캡처한다. ([클로저](../javascript/closure.md)로 인해 일어나는 현상)
React 컴포넌트 안의 props/state는 React 트리 내부 상태인데, `useEffect`를 사용하면 이 상태를 React에서 제어되지 않는 외부 시스템과 동기화 할 수 있다.
"외부 시스템"이란?

- DOM (스크롤, 포커스, 이벤트 리스너 등)
- 로컬 스토리지
- 네트워크 요청
- 타이머(setInterval, setTimeout)
- 외부 라이브러리(API, 차트, 지도 등)

이런 것들을 React state 변화에 맞춰 동기화하는 것이 `useEffect`의 역할이다.

```javascript
function Counter({ initial }) {
  const [count, setCount] = React.useState(initial);

  React.useEffect(() => {
    document.title = `현재 카운트: ${count}`; // document.title(외부 시스템)을,
  }, [count]); // count 값(dependency)이 변할 때 마다 바깥의 값을 맞춰줌 "동기화"

  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

## 정리(clean-up)를 이용하는 Effects

- cleanup 함수는 effect 함수에서 **리턴**되는 함수이며, 컴포넌트가 unmount될 때, 그리고 변경된 deps로 다시 리렌더링할 때마다 (이전 effect를 정리할 필요가 있을 때) 실행된다.
- 컴포넌트 unmount -> cleanup 함수 실행 -> 컴포넌트 mount -> effect 함수 실행
- React에서는 useEffect 안에서 <u>구독</u>을 설정하고, cleanup 함수에서 반드시 구독 해제를 해줘야 한다.

> 💡 구독(subscription)이란?
> 어떤 외부 소스에서 값이 변할 때마다 알림을 받는 것을 말한다.
> 이벤트 리스너, 브라우저 API, 데이터 스트림 등을 예시로 들 수 있다.
> 즉, 내 코드가 능동적으로 요청(fetch)하는 게 아니라,
> 외부에서 값이 바뀌면 자동으로 콜백이 실행되는 구조이다.

### cleanup을 하는 이유

1. React가 DOM을 업데이트한 뒤 추가로 코드를 실행하는 경우 cleanup이 필요하지 않다. (실행 이후 신경쓸 것이 없는 경우)
    - 네트워크 요청 → 요청 후 결과만 처리
    - 로깅, 알림 트리거 → 후속 관리 필요 없음
    - DOM 직접 조작 → 일회성 업데이트
2. Effect가 중복 실행되지 않도록 관리하기 위해
    - `setInterval`, `setTimeout`을 사용해 등록한 작업은 clear하지 않으면 중첩으로 실행되기 때문에 의도치 않은 동작을 발생시킬 수 있다.
    - 이벤트 리스너(`addEventListener`)를 제거하지 않으면 이벤트가 중복으로 실행된다.
3. 리렌더링 되었을 때 React가 effect를 실행하기 전에 이전의 렌더링에서 파생된 effect를 정리해주어야 한다.
4. 메모리 누수(Memory Leak)를 방지, 성능 최적화를 위해
    - 정리되지 않은 타이머, 이벤트 핸들러, 구독(subscription)이 계속 남아있으면 메모리를 점점 소비하게 된다.
    - 컴포넌트가 언마운트된 이후에도 남아있을 수 있음 -> 이미 사라진 컴포넌트에 접근하는 에러를 발생시킬 수 있다.
    - 특히 WebSocket, Firebase, RxJS 같은 스트림 구독을 해제하지 않으면 앱이 오래 켜질수록 성능 저하 발생한다.
5. 리소스 관리 차원
    - 외부 API 연결(WebSocket, 서버 Sent Events 등)을 해제하지 않으면 불필요한 네트워크 비용 발생한다.
    - 외부 라이브러리 인스턴스(API, 차트, 지도 등)를 cleanup 하지 않으면 중복으로 생성돼 렌더링 성능 저하를 발생시킨다.
  
정리하면, cleanup은 중복 실행 방지, 메모리 누수 방지, 불필요한 리소스 점유 방지라는 성능/안정성 이유 때문에 필요하다.

## useEffect Hook의 활용 예시

### 1. API 호출 & 상태 동기화 (데이터 패칭)

   ```tsx
   import { useEffect, useState } from 'react';

   function UserList() {
     const [users, setUsers] = useState([]);

     useEffect(() => {
       let ignore = false;

       fetch('/api/users')
         .then((res) => res.json())
         .then((data) => {
           if (!ignore) setUsers(data);
         });

       return () => {
         ignore = true; // 언마운트 시 setState 방지
       };
     }, []); // mount 시 1회 호출

     return (
       <ul>
         {users.map((u) => (
           <li key={u.id}>{u.name}</li>
         ))}
       </ul>
     );
   }
   ```

### 2. 이벤트 리스너 등록/해제 (스크롤, resize, 키보드)

   ```tsx
   import { useEffect, useState } from 'react';

   function ScrollTracker() {
     const [scrollY, setScrollY] = useState(0);

     useEffect(() => {
       const handleScroll = () => setScrollY(window.scrollY);

       window.addEventListener('scroll', handleScroll);

       return () => {
         window.removeEventListener('scroll', handleScroll);
       };
     }, []);

     return <div>현재 스크롤 위치: {scrollY}px</div>;
   }
   ```

### 3. 외부 시스템과 상태 동기화 (WebSocket, localStorage, 타이머)

   ```tsx
   import { useEffect, useState } from 'react';

   function Chat() {
     const [messages, setMessages] = useState<string[]>([]);

     useEffect(() => {
       const ws = new WebSocket('wss://example.com/chat');

       ws.onmessage = (event) => {
         setMessages((prev) => [...prev, event.data]);
       };

       return () => {
         ws.close(); // 연결 해제
       };
     }, []);

     return (
       <ul>
         {messages.map((m, i) => (
           <li key={i}>{m}</li>
         ))}
       </ul>
     );
   }
   ```


## 주의사항

### 1. 무한 루프 방지
의존성 배열에 객체나 함수를 직접 포함하면 매번 새로운 참조로 인식되어 무한 루프가 발생할 수 있다.

```javascript
useEffect(() => {
  fetchData(user);
}, [user]); // user 객체가 매번 새로 생성되면 무한 루프

// 해결방법
useEffect(() => {
  fetchData(user.id);
}, [user.id]); // 원시값만 의존성에 포함
```
예시처럼 primitive값만 의존성 배열에 포함하거나, useCallback, useMemo를 사용하여 메모이제이션을 할 수 있다.

### 2. 메모리 누수 방지
이벤트 리스너, 타이머, 구독 등을 정리하지 않으면 메모리 누수가 발생한다.

```javascript
useEffect(() => {
  const timer = setInterval(() => {
    console.log('타이머 실행');
  }, 1000);

  // cleanup 함수 반환
  return () => {
    clearInterval(timer);
  };
}, []);
```

### 3. 의존성 배열 관리
ESLint 경고를 무시하거나 의존성을 잘못 관리하면 예상치 못한 동작이 발생한다.

```javascript
// 의존성 누락
useEffect(() => {
  setCount(count + 1);
}, []); // count가 의존성에 없음

// 해결방법
useEffect(() => {
  setCount(prevCount => prevCount + 1);
}, []); // 함수형 업데이트 사용

// 또는
useEffect(() => {
  setCount(count + 1);
}, [count]); // 의존성에 포함
```

### 4. 조건부 Effect 실행
조건 없이 Effect를 실행하면 불필요한 API 호출이나 부수 효과가 발생할 수 있다.

```javascript
useEffect(() => {
  if (!userId) return; // 조건부 실행
  
  fetchUserData(userId);
}, [userId]);
```
useEffect 내부에서 특정 조건을 만족할 때만 부수 효과를 실행하도록 하여
불필요한 API 호출, 리소스 낭비, 예상치 못한 부수 효과를 방지할 수 있다.

### 5. 비동기 함수 처리
useEffect 내부에서 직접 async/await를 사용하면 cleanup이 제대로 동작하지 않을 수 있다.

```javascript
useEffect(async () => {
  const data = await fetchData();
  setData(data);
}, []);

useEffect(() => {
  let cancelled = false;
  
  const fetchData = async () => {
    const data = await api.getData();
    if (!cancelled) {
      setData(data);
    }
  };
  
  fetchData();
  
  return () => {
    cancelled = true;
  };
}, []);
```

## useEffect와 useLayoutEffect의 차이점
두 훅은 모두 부수 효과를 실행하기 위해 사용된다.
그러나 실행 시점과 용도가 다르다.
- **실행 시점**: `useEffect`는 렌더링 후 비동기적으로 실행되며, `useLayoutEffect`는 렌더링 후 DOM이 업데이트 되기 직전의 시점에 동기적으로 실행된다.
  > 
  > 💡 useEffect의 비동기적 실행
  > 이벤트 루프의 태스크큐에 콜백이 쌓이는 방식으로 동작 -> DOM이 업데이트된 후에 실행
  > 💡 useLayoutEffect의 동기적 실행
  > 렌더링 후, 페인팅 전에 실행 -> DOM이 업데이트되기 전에 실행

- **용도**: `useEffect`는 데이터를 가져오는 작업이나 이벤트 리스너 추가 등 렌더링 후에 화면에 직접적인 영향을 주지 않는 작업에 주로 사용되며, `useLayoutEffect`는 DOM의 크기를 측정하거나, 위치를 조정해야 할 때 사용하면 즉각적으로 변경사항이 반영되어 화면 깜빡임이나 불필요한 재렌더링을 방지할 수 있다.
```tsx
import { useEffect, useLayoutEffect } from 'react';

function App() {
  useEffect(() => {
    console.log('useEffect');
  }, []);
  
  useLayoutEffect(() => {
    console.log('useLayoutEffect');
  }, []);

  return <div>Hello</div>;
}
```
- 출력 결과: `useLayoutEffect` -> `useEffect`


## 실용적인 팁

### Effect가 너무 자주 실행되는 경우
- `useCallback`과 `useMemo`로 함수와 객체를 메모이제이션할 수 있다.
- 의존성 배열을 최소화한다.

### Effect가 실행되지 않는 경우  
- 의존성 배열에 필요한 값이 모두 포함되었는지 확인한다.
- ESLint 경고를 확인한다.

### 성능 최적화
- 불필요한 Effect는 제거한다.
- Effect 내부의 무거운 연산을 `useMemo`로 분리할 수 있다.
- Effect 내부에서 `setState` 호출 시 상태 업데이트가 과도하게 발생하면 불필요한 리렌더링이 반복되어 성능 저하가 발생할 수 있다.
- Effect 내부에서 DOM 읽기/쓰기 시 성능 면에서 `useLayoutEffect`를 선택하는 것이 유리하다.
