# 면접 질문 day2

## React/Next.js
### 1. Virtual DOM 동작 원리와 Reconciliation 과정
#### Virtual DOM
Virtual DOM은 React의 핵심 개념으로, 실제 DOM의 가상 표현을 메모리에 저장해서 성능을 최적화하는 기술입니다.
**Virtual DOM의 동작 원리**
Virtual DOM은 JavaScript 객체로 표현된 가상의 DOM 트리입니다. 컴포넌트의 상태가 변경되면 새로운 Virtual DOM 트리가 생성되고, 이전 Virtual DOM과 비교해서 실제로 변경된 부분만 실제 DOM에 반영합니다.
예를 들어 리스트에 아이템 하나가 추가되면, 전체 리스트를 다시 렌더링하는 것이 아니라 새로 추가된 하나의 아이템만 실제 DOM에 추가하는 방식입니다.
#### Reconciliation 과정
Reconciliation은 이전 Virtual DOM과 새로운 Virtual DOM을 비교해서 차이점을 찾는 과정입니다. 이 과정은 3단계로 진행됩니다.
**첫째, Diffing 알고리즘**입니다. 이전 Virtual DOM과 새로운 Virtual DOM을 비교해서 실제로 변경된 부분만 찾아내는 과정입니다. 
**둘째, Element Type 비교**입니다. 같은 위치의 요소가 다른 타입이면 이전 트리를 완전히 제거하고 새로운 트리를 구축합니다. 같은 타입이면 속성만 비교해서 변경된 속성만 업데이트합니다.
**셋째, Key를 이용한 리스트 최적화**입니다. 리스트의 자식 요소들에 key를 제공하면, React가 어떤 아이템이 추가, 제거, 수정되었는지 효율적으로 판별할 수 있습니다.
#### 성능상의 이점
실제 DOM 조작은 브라우저의 렌더링 엔진을 통해 레이아웃 재계산, 리페인트 등의 비용이 큰 작업이 발생합니다. 하지만 Virtual DOM은 메모리상의 JavaScript 객체 비교이므로 훨씬 빠릅니다.
또한 배치 업데이트를 통해 여러 변경사항을 모아서 한 번에 실제 DOM에 반영하므로 불필요한 렌더링을 줄일 수 있습니다.

### 2. useEffect 의존성 배열과 클린업 함수의 정확한 동작
useEffect의 의존성 배열은 언제 effect가 실행될지를 결정하는 중요한 요소입니다. 3가지 패턴이 있습니다.
**1. 의존성 배열이 없는 경우**
컴포넌트가 렌더링될 때마다 실행됩니다. 성능상 좋지 않아 거의 사용하지 않습니다.
**2. 빈 의존성 배열**
컴포넌트가 처음 마운트될 때만 실행됩니다. 주로 API 호출이나 이벤트 리스너 등록에 사용합니다.
**3. 특정 값이 있는 의존성 배열**
배열에 있는 값이 변경될 때만 실행됩니다. React는 `Object.is()`를 사용하여 이전 값과 비교합니다.

#### 클린업 함수의 동작 원리
클린업 함수는 두 가지 상황에서 실행됩니다.
**1. 컴포넌트가 언마운트될 때**
**2. 의존성이 변경되어 새로운 effect가 실행되기 직전**

**정확한 실행 순서**
1. 이전 effect의 클린업 함수 실행
2. 새로운 effect 함수 실행
3. 새로운 클린업 함수 등록

이를 통해 메모리 누수를 방지하고 불필요한 리소스를 정리할 수 있습니다.

### 3. React 18의 Concurrent Features
React 18에서 도입된 Concurrent Features는 사용자 경험을 향상시키기 위해 렌더링 과정을 더 유연하고 효율적으로 만드는 기능들입니다.
#### 핵심 개념 Concurrent Rendering
기존 React는 렌더링이 시작되면 완료될 때까지 중단할 수 없었습니다. 하지만 Concurrent Rendering에서는 렌더링 작업을 중단하고 재개할 수 있어, 더 중요한 업데이트를 우선 처리할 수 있습니다.
#### 주요 Concurrent Features
1. Automatic Batching: 여러 상태 업데이트를 자동으로 하나의 렌더링으로 묶어 성능을 최적화합니다.
2. Transitions: 긴급하지 않은 업데이트를 표시해서 더 중요한 업데이트가 우선 처리되도록 합니다.
3. Suspense 개선: 서버 사이드 렌더링과 데이터 페칭에서 더 강력한 기능을 제공합니다.
4. useDeferredValue: 값의 업데이트를 지연시켜 더 중요한 업데이트가 먼저 처리되도록 합니다.
#### 장점
첫째, 사용자 입력 반응성이 크게 향상됩니다. 검색어를 입력할 때 타이핑은 즉시 반영되고, 검색 결과는 타이핑이 끝난 후 업데이트되어 더 부드러운 사용자 경험을 제공합니다.
둘째, 대용량 데이터 처리 시 앱이 멈추지 않습니다. 큰 리스트를 렌더링할 때도 스크롤이나 다른 인터랙션이 끊기지 않습니다.
셋째, 자동 배칭으로 불필요한 렌더링이 줄어들어 전체적인 성능이 향상됩니다.

### 4. 컴포넌트 최적화 (memo, useMemo, useCallback)
React에서 성능 최적화를 위한 3가지 주요 도구 입니다.
#### React.memo - 컴포넌트 렌더링 최적화
컴포넌트의 props가 변경되지 않았을 때 리렌더링을 방지합니다.
```js
const ExpensiveComponent = React.memo(({ name, age }) => {
  console.log('ExpensiveComponent 렌더링');
  return <div>{name}님의 나이는 {age}세입니다.</div>;
});
```
얕은 비교를 기본으로 하므로, 객체나 배열 props는 참조가 바뀌면 리렌더링됩니다.

### useMemo - 값 계산 최적화
비용이 큰 계산 결과를 메모이제이션해서 불필요한 재계산을 방지합니다.
```js
function ProductList({ products, searchTerm }) {
  const filteredProducts = useMemo(() => {
    console.log('필터링 계산 실행');
    return products.filter(product => 
      product.name.toLowerCase().includes(searchTerm.toLowerCase())
    );
  }, [products, searchTerm]); // 의존성 배열 변경 시에만 재계산

  return (
    <ul>
      {filteredProducts.map(product => (
        <li key={product.id}>{product.name}</li>
      ))}
    </ul>
  );
}
```

### useCallback - 함수 참조 최적화
함수의 참조를 메모이제이션해서 불필요한 자식 컴포넌트 리렌더링을 방지합니다.
```js
function TodoApp() {
  const [todos, setTodos] = useState([]);
  const [filter, setFilter] = useState('all');

  const handleToggle = useCallback((id) => {
    setTodos(prev => prev.map(todo => 
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    ));
  }, []); // 의존성이 없으므로 함수 참조가 유지됨

  const handleDelete = useCallback((id) => {
    setTodos(prev => prev.filter(todo => todo.id !== id));
  }, []);

  return (
    <div>
      {todos.map(todo => (
        <TodoItem 
          key={todo.id}
          todo={todo}
          onToggle={handleToggle}
          onDelete={handleDelete}
        />
      ))}
    </div>
  );
}

const TodoItem = React.memo(({ todo, onToggle, onDelete }) => {
  console.log(`TodoItem ${todo.id} 렌더링`);
  return (
    <div>
      <span onClick={() => onToggle(todo.id)}>{todo.text}</span>
      <button onClick={() => onDelete(todo.id)}>삭제</button>
    </div>
  );
});
```
#### 최적화 전략과 주의사항

**언제 사용해야 하는가:**
1. React DevTools Profiler로 성능 병목이 확인된 경우
2. 자식 컴포넌트가 많고 자주 리렌더링되는 경우
3. 비용이 큰 계산이나 API 호출이 포함된 경우
4. 사용자 경험에 실제 영향을 주는 경우

**주의사항:**
1. **메모리 비용**: 모든 최적화는 메모리 사용량을 증가시킵니다.
2. **비교 비용**: 얕은 비교도 연산 비용이 발생합니다.
3. **의존성 관리**: 잘못된 의존성 배열은 버그를 야기할 수 있습니다.
4. **과도한 최적화**: 성능 향상보다 코드 복잡성만 증가할 수 있습니다.

### 5. 서버 컴포넌트 vs 클라이언트 컴포넌트 구분 기준
Next.js 13의 App Router에서는 서버 컴포넌트가 기본이고, 필요에 따라 클라이언트 컴포넌트를 사용하는 방식입니다.
#### 서버 컴포넌트 (Server Components)
서버에서 렌더링되어 HTML로 클라이언트에 전달되는 컴포넌트입니다.
**사용하면 좋은 경우:**
- 데이터베이스나 API에서 데이터를 직접 가져오는 경우
- 환경 변수나 서버 전용 라이브러리를 사용하는 경우
- SEO가 중요한 정적 콘텐츠
- 초기 로딩 성능이 중요한 페이지

**서버 컴포넌트의 장점:**
- 초기 로딩이 빠르고 SEO에 유리
- JavaScript 번들 크기가 줄어듦
- 서버 리소스에 직접 접근 가능

#### 클라이언트 컴포넌트 (Client Components)
'use client' 지시어를 사용해서 브라우저에서 실행되는 컴포넌트입니다.
**사용하면 좋은 경우:**
- 사용자 상호작용(onClick, onChange 등)이 필요한 경우
- React Hooks(useState, useEffect 등)를 사용하는 경우
- 브라우저 전용 API(localStorage, geolocation 등)를 사용하는 경우
- 실시간 업데이트가 필요한 경우

**클라이언트 컴포넌트의 장점:**
- 풍부한 인터랙티브 기능 제공
- 실시간 상태 관리 가능
- 사용자 경험이 더 동적

#### 실무에서의 구분 기준
**서버 컴포넌트로 만드는 것:**
- 페이지 레이아웃, 헤더, 푸터
- 상품 목록, 블로그 포스트 등 정적 콘텐츠
- 초기 데이터를 보여주는 컴포넌트
  
**클라이언트 컴포넌트로 만드는 것:**
- 검색 폼, 필터링 UI
- 모달, 드롭다운, 탭 등 인터랙티브 UI
- 좋아요, 장바구니 등 사용자 액션이 필요한 기능


### 6. Next.js Image 최적화와 코드 스플리팅 메커니즘
Next.js는 성능 최적화를 위해 자동으로 이미지 최적화와 코드 스플리팅을 제공합니다.

#### Next.js Image 최적화
Next.js의 Image 컴포넌트는 여러 단계의 최적화를 자동으로 수행합니다.
1. 자동 포맷 변환: 브라우저가 지원하는 최적의 이미지 포맷으로 자동 변환합니다.
2. 반응형 이미지 생성: 디바이스 크기에 맞는 여러 사이즈의 이미지를 자동 생성합니다.
3. Lazy Loading: 뷰포트에 들어올 때만 이미지를 로드합니다.
4. Placeholder 및 Blur 효과: 이미지 로딩 중 사용자 경험을 개선합니다.

#### 코드 스플리팅 메커니즘
Next.js는 페이지 기반과 동적 import 기반으로 코드를 자동 분할합니다.
1. 자동 페이지 스플리팅: 각 페이지는 별도의 JavaScript 번들로 분할됩니다.
2. 동적 Import: 필요할 때만 컴포넌트를 로드합니다.
3. 공통 청크 최적화: 여러 페이지에서 사용되는 공통 라이브러리는 별도 청크로 분리됩니다.

### 7. Next.js Middleware 활용법
Next.js Middleware는 요청이 페이지나 API 라우트에 도달하기 전에 실행되는 코드로, 인증, 리다이렉트, 헤더 조작 등 다양한 용도로 활용할 수 있습니다.
프로젝트 루트에 middleware.js 파일을 생성하면 자동으로 인식됩니다.
1. 인증 및 권한 검사
2. 국제화(i18n) 처리
3. API 레이트 리미팅
4. 요청/응답 헤더 조작

#### 장점과 주의사항
**장점:**
- 서버사이드에서 실행되어 클라이언트 보안 우회 불가
- 페이지 로드 전에 처리되어 불필요한 렌더링 방지
- Edge Runtime에서 실행되어 빠른 응답 속도

**주의사항:**
- Edge Runtime 제약으로 일부 Node.js API 사용 불가
- 복잡한 로직은 성능에 영향을 줄 수 있음
- 쿠키나 헤더 조작 시 브라우저 캐싱 고려 필요

Middleware는 클라이언트와 서버 사이의 중간 계층 역할을 하며, 보안, 성능, 사용자 경험 개선에 매우 유용한 도구입니다.