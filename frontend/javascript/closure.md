# 클로저(Closure)

## 클로저란?

> 💡 _클로저는 주변 상태(어휘적 환경)에 대한 참조와 함께 묶인(포함된) 함수의 조합입니다._ 
>  _즉, 클로저는 내부 함수에서 외부 함수의 범위에 대한 접근을 제공합니다._ 
> _JavaScript에서 클로저는 함수 생성 시 함수가 생성될 때마다 생성됩니다._
>
> — [MDN Web Docs: Closures](https://developer.mozilla.org/ko/docs/Web/JavaScript/Closures)
>

클로저란 함수가 선언될 때의 스코프를 기억하여 함수가 생성된 이후에도 그 스코프에 접근할 수 있는 기능을 말한다.
함수가 <u>자신이 생성된 환경을 기억</u>하는 것.

```javascript
function outer() {
  const x = 10; // "선언될 때의 스코프"
  function inner() {
    console.log(x); // 10 (x 자신이 생성된 환경에 접근)
  }
  return inner;
}
```

#### 클로저는 자바스크립트의 함수가 일급 객체라는 특성과 렉시컬 스코프의 조합이다.

> 💡일급 객체란?
> 다음의 일급 객체의 조건을 만족하는 객체를 일급 객체라고 한다.
>
> - 변수나 데이터 구조(객체, 배열)에 담을 수 있다.
> - 함수의 매개변수로 전달할 수 있다.
> - 함수의 반환값으로 사용할 수 있다.
> - 런타임에 생성할 수 있다.
>
> 일급 객체의 예시
>
> - 콜백함수: 함수를 인자로 넘겨 실행할 수 있다.
> - 고차함수: 함수를 인자로 받거나 반환할 수 있다.
> - 함수형 프로그래밍: map, filter.. 등 메서드 구현이 가능하다.
> - 클로저: 함수가 다른 함수의 변수에 접근 가능하다.
>
> 💡렉시컬 스코프란?
>
> - 함수가 선언된 위치에 따라 상위 스코프를 결정하는 것을 말한다.

## 클로저는 언제 활용하나요? 🤷‍♀️

클로저는 변수와 함수의 접근 범위를 제어하고, 특정 데이터와 상태를 유지하기 위해 자주 활용된다.

### JavaScript에서 클로저의 활용 예시

#### 1. 데이터 은닉
외부에서 접근할 수 없는 비공개 변수와 함수를 만들 수 있다. 이를 통해 데이터를 은닉하여 외부 접근을 막고, 데이터 무결성을 유지할 수 있다. 예를 들어 특정 함수 내부에서만 접근 가능한 변수를 생성하고, 이를 조작할 수 있는 함수만 외부로 노출하여 안전하게 데이터를 관리할 수 있다.

```javascript
function createCounter() {
  let count = 0; // 은닉된 상태
  return function () {
    count++;
    return count;
  };
}

const counter = createCounter();
console.log(counter()); // 1
console.log(counter()); // 2
console.log(counter()); // 3
```

위 코드는 클로저를 사용하여 데이터를 은닉하는 예시이다.
`count`는 `createCounter` 실행이 끝났어도 내부 함수가 참조하고 있기 때문에 사라지지 않고 유지된다.
이를 이용해 상태를 은닉하고 안전하게 관리할 수 있다.

> 💡데이터 은닉 vs 상태 은닉?
> 두 개념은 큰 틀에서 같은 개념이지만 의미 차이를 두자면,
> - 데이터 은닉: 외부에서 직접 접근할 수 없는 변수 자체를 감추는 것
> - 상태 은닉: 단순히 데이터만 감추는 게 아니라, 그 데이터의 변화(상태 변화)를 특정 함수로만 제어 가능하게 하는 것
>
> 즉, 데이터 은닉 -> '감춘다'에 초점
> 상태 은닉 -> '관리한다'에 초점

#### 2. 비동기 작업에 활용
비동기 작업에서 이전의 실행 컨텍스트를 유지해야할 때 유용하다. 콜백 함수가 비동기적으로 실행될 때 클로저를 사용하면 함수 실행 시점의 변수를 참조할 수 있다.

```javascript
function setupButton(id) {
  const button = document.getElementById(id);

  function delayedLogs() {
    for (let i = 1; i <= 3; i++) {
      (function (n) {
        setTimeout(() => {
          console.log(n); // 클로저가 n을 기억
        }, n * 1000);
      })(i);
    }
  }

  button.addEventListener("click", delayedLogs);
  // 버튼 클릭 시 1초 뒤 1, 2초 뒤 2, 3초 뒤 3 출력
}

setupButton('btn1');
setupButton('btn2');
```
비동기 작업(`setTimeout`) 안에서 루프 변수(i)가 올바르게 기억되도록 클로저를 활용했다.
즉시 실행 함수(IIFE)를 사용하여 i값을 캡처 -> 클로저로 각 타이머에서 고유 값을 유지한다.

#### 3. 모듈 패턴 구현시 활용
모듈 패턴은 특정 기능을 캡슐화하고, 외부에 공개하고자 하는 부분만 선택적으로 노출하여 코드의 응집력을 높이고, 유지보수성을 향상시키는 패턴이다. 클로저를 활용하면 필요한 함수와 데이터만 외부로 노출함으로써 모듈 패턴을 쉽게 구현할 수 있다.

```javascript
const CounterModule = (function () {
  let count = 0; // private

  function increment() {
    count++;
    return count;
  }

  function reset() {
    count = 0;
  }

  return {
    increment, // 공개
    reset,     // 공개
  };
})();

console.log(CounterModule.increment()); // 1
console.log(CounterModule.increment()); // 2
CounterModule.reset();
console.log(CounterModule.increment()); // 1
```
count는 외부에서 접근할 수 없는 private 변수이다. -> 은닉
오직 `increment`와 `reset`으로만 제어 가능하다. -> 상태 관리

### React에서 클로저의 활용 예시
React에서는 훅(Hook)이나 이벤트 핸들러에서 클로저가 자주 활용된다.

#### 1. useEffect에서 클로저 활용
```jsx
import { useState, useEffect } from "react";

export default function Timer() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    // 클로저: 컴포넌트의 count 상태를 캡처
    const timer = setInterval(() => {
      setCount(prev => prev + 1); // 함수형 업데이트 사용
    }, 1000);

    return () => clearInterval(timer);
  }, []);

  return <div>카운트: {count}</div>;
}
```

#### 2. 이벤트 핸들러에서 클로저 활용
```jsx
import { useState } from "react";

export default function TodoList() {
  const [todos, setTodos] = useState([]);
  const [inputValue, setInputValue] = useState("");

  // 클로저: todos와 inputValue를 캡처
  const addTodo = () => {
    if (inputValue.trim()) {
      setTodos([...todos, { id: Date.now(), text: inputValue }]);
      setInputValue(""); // 클로저가 inputValue를 기억
    }
  };

  // 클로저: todos 배열을 캡처
  const deleteTodo = (id) => {
    setTodos(todos.filter(todo => todo.id !== id));
  };

  return (
    <div>
      <input 
        value={inputValue}
        onChange={(e) => setInputValue(e.target.value)}
      />
      <button onClick={addTodo}>추가</button>
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>
            {todo.text}
            <button onClick={() => deleteTodo(todo.id)}>삭제</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

#### 3. 커스텀 훅에서 클로저 활용
```jsx
import { useState, useCallback } from "react";

// 커스텀 훅: 클로저를 활용한 상태 관리
function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);

  const increment = useCallback(() => {
  setCount(prev => prev + 1);
}, []);

  const decrement = useCallback(() => {
    setCount(prev => prev - 1);
  }, []);

  const reset = useCallback(() => {
    setCount(initialValue); // 클로저가 initialValue를 기억
  }, [initialValue]);

  return { count, increment, decrement, reset };
}

export default function Counter() {
  const { count, increment, decrement, reset } = useCounter(10);

  return (
    <div>
      <p>카운트: {count}</p>
      <button onClick={increment}>+1</button>
      <button onClick={decrement}>-1</button>
      <button onClick={reset}>리셋</button>
    </div>
  );
}
```

> 💡 **클로저의 핵심**: 위 예시들에서 함수들이 자신이 선언된 환경의 변수들(`count`, `todos`, `inputValue` 등)을 기억하고 접근할 수 있는 것이 클로저이다.

## 클로저의 장단점

### 장점
- **데이터 은닉**: 외부에서 접근할 수 없는 private 변수 생성
- **상태 유지**: 함수 실행 후에도 변수 상태를 기억
- **모듈화**: 관련 기능을 하나의 단위로 묶어 관리
- **메모리 효율성**: 필요한 데이터만 메모리에 유지

### 단점
- **메모리 누수**: 클로저가 참조하는 변수는 가비지 컬렉션되지 않음
- **성능**: 과도한 클로저 사용은 메모리 사용량 증가 가능
- **디버깅 어려움**: 클로저가 참조하는 변수를 추적하기 어려울 수 있음

## 주의사항 및 성능 고려사항

### 메모리 관리
```javascript
// 메모리 누수 예시
function createHandler() {
  const largeData = new Array(1000000).fill('data');
  
  return function() {
    // largeData를 사용하지 않아도 클로저로 인해 메모리에 유지됨
    console.log('handler called');
  };
}

// 개선된 버전
function createOptimizedHandler() {
  const largeData = new Array(1000000).fill('data');
  
  return function() {
    console.log('handler called');
    // 사용 후 참조 해제
    largeData.length = 0;
  };
}
```
⚠️ **메모리 누수 문제**: 
- `createHandler()`: 반환된 함수가 `largeData`를 참조하지 않아도, 클로저로 인해 `largeData`가 메모리에 계속 유지된다.
- `createOptimizedHandler()`: 사용 후 `largeData.length = 0`으로 배열을 비워 메모리를 해제한다.

**메모리 관리 팁**:
1. **불필요한 참조 제거**: 클로저에서 사용하지 않는 변수는 참조하지 않도록 주의
2. **명시적 해제**: 큰 데이터 사용 후 `null` 할당이나 배열 길이를 0으로 설정
3. **이벤트 리스너 정리**: 컴포넌트 언마운트 시 이벤트 리스너 제거
4. **디버깅 도구 활용**: 브라우저 개발자 도구의 Memory 탭으로 메모리 사용량 모니터링

### this 바인딩 주의사항
```javascript
const obj = {
  name: 'JavaScript',
  getName: function() {
    return function() {
      return this.name; // undefined (this가 전역 객체를 가리킴)
    };
  },
  getNameWithClosure: function() {
    const self = this; // 클로저로 this 캡처
    return function() {
      return self.name; // 'JavaScript'
    };
  }
};
```
⚠️ **this 바인딩 문제**: 
- `getName()`: 반환된 함수가 독립적으로 실행될 때 `this`는 전역 객체(브라우저에서는 `window`, Node.js에서는 `global`)를 가리킨다.
- `getNameWithClosure()`: 클로저를 사용하여 `this`를 `self` 변수에 저장하고, 내부 함수에서 `self`를 참조한다.

**해결 방법**:
1. **클로저 활용**: `const self = this`로 this를 캡처
2. **bind() 사용**: `return function() { return this.name; }.bind(this)`
3. **화살표 함수**: `return () => this.name` (화살표 함수는 상위 스코프의 this를 가리킨다.)

## 마무리

클로저는 JavaScript의 핵심 개념 중 하나로, 다음과 같은 실무 활용을 제공한다.

### 실무에서의 활용
- React Hook 구현 (useState, useEffect 등)
- 이벤트 핸들러에서 상태 유지
- 모듈 패턴으로 코드 구조화
- 비동기 작업에서 컨텍스트 유지
- 함수 팩토리 패턴 구현

클로저를 이해하면 JavaScript의 함수형 프로그래밍과 React의 Hook 동작 원리를 더 깊이 이해할 수 있다.

