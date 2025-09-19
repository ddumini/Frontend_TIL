# 면접 질문 day3

### 1. 자바스크립트에서 == 와 === 가 어떻게 다른지 설명해 주세요.
JavaScript에서 == (동등 연산자)와 === (일치 연산자)는 값을 비교하는 방식이 다릅니다.
**=== (일치 연산자) - Strict Equality**
타입과 값이 모두 같을 때만 true를 반환합니다. 타입 변환이 일어나지 않습니다.
```javascript
console.log(5 === 5);        // true (같은 타입, 같은 값)
console.log(5 === '5');      // false (다른 타입)
console.log(true === 1);     // false (다른 타입)
console.log(null === undefined); // false (다른 타입)
console.log(0 === false);    // false (다른 타입)
```
**== (동등 연산자) - Loose Equality**
값을 비교하기 전에 타입 변환(Type Coercion)을 시도합니다.
== 연산자의 복잡한 변환 과정
```javascript
console.log('0' == false);   // true 
// 1. false → 0 (불린을 숫자로 변환)
// 2. '0' == 0 (문자열과 숫자 비교)
// 3. '0' → 0 (문자열을 숫자로 변환)
// 4. 0 == 0 → true
```

실제 개발에서는 === 연산자를 사용하는 것이 좋습니다.
예외적으로 == 를 사용하는 경우는 null과 undefined를 함께 체크할 때입니다.
```javascript
function isNullOrUndefined(value) {
  return value == null;  // null 또는 undefined 모두 true
}
// 아래와 동일하게 작동
function isNullOrUndefined(value) {
  return value === null || value === undefined;
}
```
== 는 암묵적 형변환이이 일어나는데, 암묵적 형변환이 일어날 것을 모두 기억하거나 예측하는 것이 어렵기 때문에, 의도하지 않은 결과를 얻게 될 가능성이 크고 이로 인해 오류 가능성이 커집니다. 
=== 을 사용하면 코드의 의도가 명확해지고 예상치 못한 버그를 방지할 수 있습니다. ESLint 같은 도구에서도 == 사용을 경고하는 이유입니다.

### 2. 자바스크립트에서 얕은 복사(Shallow Copy)와 깊은 복사(Deep Copy)에 대해 설명해 주세요.
실제값을 복사 하는 것을 깊은 복사, 객체의 메모리 주소값을 복사하는 것을 얕은 복사라고 합니다.
JavaScript에서 객체나 배열을 복사할 때 참조 관계에 따라 얕은 복사와 깊은 복사로 구분됩니다.
**얕은 복사 (Shallow Copy)**
최상위 레벨의 속성만 복사하고, 중첩된 객체나 배열은 참조를 공유하는 방식입니다.
```javascript
const original = {
  name: '김수민',
  age: 30,
  hobbies: ['독서', '영화감상'],
  address: {
    city: '서울',
    district: '양천구'
  }
};

// 얕은 복사 방법들
const shallow1 = { ...original };           // 스프레드 연산자
const shallow2 = Object.assign({}, original); // Object.assign
const shallow3 = Array.from(original);      // 배열의 경우

// 문제점: 중첩된 객체는 참조 공유
shallow1.name = '홍길동';           // 원본에 영향 없음
shallow1.hobbies.push('운동');      // 원본에도 영향 있음!
shallow1.address.city = '부산';     // 원본에도 영향 있음!

console.log(original.name);         // '김수민'
console.log(original.hobbies);      // ['독서', '영화감상', '운동']
console.log(original.address.city); // '부산'
```
**깊은 복사 (Deep Copy)**
모든 레벨의 속성을 완전히 새로운 메모리 공간에 복사하는 방식입니다.
```javascript
const original = {
  name: '김수민',
  age: 30,
  hobbies: ['독서', '영화감상'],
  address: {
    city: '서울',
    district: '양천구'
  }
};

// 깊은 복사 방법들
const deep1 = JSON.parse(JSON.stringify(original)); // JSON.parse/stringify
const deep2 = structuredClone(original); // structuredClone(최신 브라우저 지원)
const deep3 = lodash.cloneDeep(original); // lodash(라이브러리)
//재귀 함수를 이용한 깊은 복사
function deepCopy(obj) {
  if (typeof obj !== 'object' || obj === null) {
    return obj;
  }
  const copy = Array.isArray(obj) ? [] : {};
  for (let key in obj) {
    copy[key] = deepCopy(obj[key]);
  }
  return copy;
}
const deepCopyObj = deepCopy(original);
```

**활용 방법**
얕은 복사는 빠르지만 중첩된 구조에서 주의가 필요하고, 깊은 복사는 안전하지만 성능 비용이 있습니다.
- 단순한 객체: 얕은 복사 사용
- 중첩이 깊은 객체: 필요한 부분만 선택적으로 깊은 복사
- 변경이 필요한 객체: 깊은 복사 사용
- 불변성을 유지해야 하는 객체: 깊은 복사 사용

얕은 복사는 최상위 속성만 복사해서 중첩 객체는 참조를 공유하고, 깊은 복사는 모든 레벨을 완전히 새로 복사해서 원본과 독립적입니다. React에서 상태 업데이트 시 얕은 복사를 주로 사용하되, 중첩 구조 변경이 필요할 때는 깊은 복사나 불변성 라이브러리를 활용합니다.

### 3. var, let, const 를 중복 선언 허용, 스코프, 호이스팅 관점에서 서로 비교해 주세요.
"ES6에서 도입된 let과 const는 기존 var의 문제점들을 해결하기 위해 만들어졌습니다. 세 가지 관점에서 비교해보겠습니다.
1. 중복 선언 허용 여부
   - var: 중복 선언 허용
   - let, const: 중복 선언 금지
2. 스코프
   - var: 함수 스코프
   - let, const: 블록 스코프
3. 호이스팅
   - var: 호이스팅 후 undefined로 초기화
   - let, const: 호이스팅되지만 TDZ에 걸려 접근 불가
   - const: 선언과 동시에 초기화 필요(syntax error),
   재할당 불가(객체/배열의 경우 내용 변경 가능)

var는 함수 스코프, 중복 선언 허용, 호이스팅 혼란으로 안정성이 떨어져 사용하지 않는 것이 좋습니다.
코드의 안전성과 가독성을 높이기 위해서는 let과 const를 사용하는 것이 좋습니다.

### 4. 브라우저가 어떻게 동작하는지 설명해 주세요.
브라우저는 사용자가 웹 페이지를 요청했을 때부터 화면에 렌더링하기까지 여러 단계를 거쳐 동작합니다.

#### 1. 네트워크 요청 단계
사용자가 URL을 입력하면 DNS 조회를 통해 IP 주소를 찾고, HTTP/HTTPS 요청을 서버에 보냅니다.

#### 2. HTML 파싱과 DOM 생성
서버로부터 받은 HTML을 파싱해서 DOM 트리를 생성합니다.

#### 3. CSS 파싱과 CSSOM 생성
CSS 파일을 파싱해서 CSSOM 트리를 생성합니다.

#### 4. 렌더 트리 생성
DOM과 CSSOM을 결합하여 실제 화면에 표시될 요소들만으로 렌더 트리를 생성합니다.

#### 5. 레이아웃 단계
각 요소의 정확한 위치와 크기를 계산합니다. 이 과정에서 리플로우가 발생할 수 있습니다.

#### 6. 페인팅 단계
실제 픽셀로 요소들을 그리는 단계입니다. 배경색, 텍스트, 테두리 등을 렌더링합니다.

#### 7. 컴포지팅 단계
여러 레이어를 합성해서 최종 화면을 만듭니다.

**JavaScript 실행과 렌더링 차단**
HTML 파싱 중 `<script>` 태그를 만나면 렌더링이 차단되므로
async/defer 속성을 사용하여 비동기로 실행하거나 DOM 파싱 완료 후 실행해야 합니다.

**Critical Rendering Path 최적화**
CSS는 `<head>`에 배치하여 렌더링 차단 방지
JavaScript는 `async`/`defer` 속성 활용 (프리로드 스캐너에 의해 먼저 실행되어 렌더링 차단 방지)

**리플로우와 리페인트**
리플로우를 발생시키는 작업은 비용이 크기 때문에 최소화하는 것이 중요합니다.
리페인트만 발생하는 작업은 상대적으로 가볍습니다.
최적화를 위해 컴포지팅 단계에서만 처리하는 transform과 opacity 속성을 사용하는 것이 좋습니다.
컴포지팅 단계에서 처리되는 작업은 GPU를 활용하여 빠르게 처리됩니다.
