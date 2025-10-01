# 면접 질문 day4

### 1. 이벤트 버블링, 캡쳐링, 위임에 대해 설명해 주세요.
DOM 이벤트는 발생 시점부터 처리까지 3단계의 전파 과정을 거치며, 이를 이해하면 효율적인 이벤트 처리가 가능합니다.

이벤트 전파의 3단계
1. 캡쳐링(Capturing Phase): 루트에서 타겟으로 내려감
2. 타겟(Target Phase): 실제 이벤트가 발생한 요소
3. 버블링(Bubbling Phase): 타겟에서 루트로 올라감
  
#### 1. 이벤트 버블링 (Event Bubbling)
이벤트가 발생한 요소에서 부모로 전파되는 과정입니다.
```js
// HTML 구조: outer > middle > inner(button)

document.getElementById('outer').addEventListener('click', () => {
  console.log('outer 클릭');
});

document.getElementById('middle').addEventListener('click', () => {
  console.log('middle 클릭');
});

document.getElementById('inner').addEventListener('click', () => {
  console.log('inner 클릭');
});

// button 클릭 시 출력 순서:
// "inner 클릭"
// "middle 클릭"  
// "outer 클릭"
```

#### 2. 이벤트 캡쳐링 (Event Capturing)
루트 요소에서 타겟 요소로 내려가면서 이벤트가 전파되는 과정입니다.
```js
// 세 번째 매개변수를 true로 설정하면 캡쳐링 단계에서 실행
document.getElementById('outer').addEventListener('click', () => {
  console.log('outer 캡쳐링');
}, true);

document.getElementById('middle').addEventListener('click', () => {
  console.log('middle 캡쳐링');
}, true);

document.getElementById('inner').addEventListener('click', () => {
  console.log('inner 캡쳐링');
}, true);

// button 클릭 시 출력 순서:
// "outer 캡쳐링"
// "middle 캡쳐링"
// "inner 캡쳐링"
```

#### 이벤트 전파 제어
전파 중단, 기본 동작 방지, 즉시 전파 중단 등의 메서드를 제공합니다.
```js
// 이벤트 전파 중단
document.getElementById('middle').addEventListener('click', (e) => {
  console.log('middle 클릭');
  e.stopPropagation(); // 여기서 전파 중단
});

// 기본 동작 방지
document.getElementById('inner').addEventListener('click', (e) => {
  e.preventDefault(); // 링크나 폼 제출 등 기본 동작 방지
  console.log('기본 동작 방지됨');
});

// 즉시 전파 중단 (같은 요소의 다른 리스너도 실행 안 됨)
document.getElementById('inner').addEventListener('click', (e) => {
  e.stopImmediatePropagation();
  console.log('첫 번째 리스너');
});

document.getElementById('inner').addEventListener('click', (e) => {
  console.log('두 번째 리스너'); // 실행되지 않음
});
```

#### 이벤트 위임 (Event Delegation)
부모 요소에서 자식 요소들의 이벤트를 처리하는 패턴입니다.
```js
// 비효율적인 방법: 각 항목에 개별 리스너
const items = document.querySelectorAll('.item');
items.forEach(item => {
  item.addEventListener('click', handleClick);
});

// 효율적인 방법: 이벤트 위임
document.getElementById('list').addEventListener('click', (e) => {
  // 실제 클릭된 요소가 .item인지 확인
  if (e.target.classList.contains('item')) {
    handleItemClick(e.target);
  }
  
  // 또는 closest 메서드 사용
  const item = e.target.closest('.item');
  if (item) {
    handleItemClick(item);
  }
});

function handleItemClick(item) {
  console.log('아이템 클릭:', item.textContent);
}
```
이벤트 위임의 장점
1. 성능 최적화: 리스너 개수 감소로 메모리 사용량 줄임
2. 동적 요소 처리: 나중에 추가된 요소도 자동으로 이벤트 처리
3. 코드 간소화: 하나의 리스너로 여러 요소 관리

### 2. 자바스크립트 this에 대해 설명해 주세요.
JavaScript의 this는 함수가 호출되는 방식에 따라 값이 결정되는 특별한 키워드입니다. 호출 컨텍스트에 따라 다른 객체를 참조합니다.

#### 1. 전역 컨텍스트에서의 this
전역 컨텍스트에서의 this는 브라우저에서는 window 객체, Node.js에서는 global 객체를 참조합니다.
```js
console.log(this); // 브라우저: window 객체, Node.js: global 객체

function globalFunction() {
  console.log(this); // 일반 모드: window, strict 모드: undefined
}

'use strict';
function strictFunction() {
  console.log(this); // undefined
}
```

#### 2. 객체 메서드에서의 this
메서드를 호출한 객체가 this가 됩니다.
```js
const person = {
  name: '김수민',
  age: 30,
  greet: function() {
    console.log(`안녕하세요, 저는 ${this.name}입니다.`); // this = person
  },
  getInfo: function() {
    return `${this.name}은 ${this.age}세입니다.`; // this = person
  }
};

person.greet(); // "안녕하세요, 저는 김수민입니다."

// 주의: 메서드를 변수에 할당하면 this가 바뀜
const greetFunc = person.greet;
greetFunc(); // this는 window (또는 undefined in strict mode)
```

#### 3. 생성자 함수와 클래스에서의 this
생성자 함수와 클래스에서의 this는 새로 생성되는 인스턴스가 됩니다.
```js
// 생성자 함수
function Person(name, age) {
  this.name = name;  // 새로 생성되는 인스턴스
  this.age = age;
  this.greet = function() {
    console.log(`저는 ${this.name}입니다.`);
  };
}

const kim = new Person('김수민', 30);
kim.greet(); // this = kim 인스턴스

// 클래스
class Developer {
  constructor(name, skill) {
    this.name = name;    // 인스턴스 객체
    this.skill = skill;
  }
  
  introduce() {
    console.log(`${this.name}은 ${this.skill} 개발자입니다.`);
  }
}

const dev = new Developer('김수민', 'Frontend');
dev.introduce(); // this = dev 인스턴스
```

#### 4. 명시적 this 바인딩 (call, apply, bind)
명시적으로 this를 바인딩하여 함수를 호출합니다.
```js
const person1 = { name: '김수민' };
const person2 = { name: '홍길동' };

function introduce(age, job) {
  console.log(`${this.name}은 ${age}세 ${job}입니다.`);
}

// call: 인수를 개별적으로 전달
introduce.call(person1, 30, '개발자');    // "김수민은 30세 개발자입니다."
introduce.call(person2, 25, '디자이너');  // "홍길동은 25세 디자이너입니다."

// apply: 인수를 배열로 전달
introduce.apply(person1, [30, '개발자']); // "김수민은 30세 개발자입니다."

// bind: 새로운 함수를 반환 (this가 고정됨)
const kimIntroduce = introduce.bind(person1);
kimIntroduce(30, '개발자'); // "김수민은 30세 개발자입니다."
```

#### 5. 화살표 함수에서의 this
화살표 함수는 자신만의 this를 갖지 않고, 정의된 시점의 상위 스코프(lexical scope)의 this를 사용합니다.
```js
const obj = {
  name: '김수민',
  regularMethod: function() {
    console.log('Regular:', this.name); // this = obj
    
    const arrowFunction = () => {
      console.log('Arrow:', this.name); // this = obj (상위 스코프의 this)
    };
    arrowFunction();
    
    function innerFunction() {
      console.log('Inner:', this.name); // this = window (또는 undefined)
    }
    innerFunction();
  }
};

obj.regularMethod();
// "Regular: 김수민"
// "Arrow: 김수민"  
// "Inner: undefined"
```

#### 6. 이벤트 핸들러에서의 this
```js
// 일반 함수: this는 이벤트가 발생한 요소
document.getElementById('button').addEventListener('click', function() {
  console.log(this); // button 요소
  this.style.color = 'red';
});

// 화살표 함수: this는 상위 스코프
document.getElementById('button').addEventListener('click', () => {
  console.log(this); // window 객체 (또는 상위 스코프)
});
```

#### this 결정 우선순위
1. new 키워드: 새로운 객체
2. call, apply, bind: 명시적으로 전달된 객체
3. 메서드 호출: 메서드를 호출한 객체
4. 일반 함수 호출: window (strict 모드에서는 undefined)

### 3. 렉시컬 스코프(Lexical scope)에 대해 설명해 주세요.
렉시컬 스코프는 함수가 선언된 위치에 따라 스코프가 결정되는 방식입니다. 함수가 호출되는 위치가 아닌, 코드에서 함수가 작성된 위치에 따라 변수 접근 범위가 정해집니다.
정적 스코프(Static scope)라고도 합니다.

#### 렉시컬 스코프의 장점
1. 예측 가능성: 코드만 보고도 변수 접근 범위를 알 수 있음
2. 디버깅 용이성: 스코프가 명확하여 오류 추적이 쉬움
3. 모듈화: 프라이빗 변수와 퍼블릭 인터페이스 구현 가능
4. 클로저 지원: 함수형 프로그래밍 패턴 활용 가능
렉시컬 스코프는 JavaScript의 핵심 개념으로, 함수형 프로그래밍과 모듈 시스템의 기반이 됩니다. 이를 이해하면 더 안전하고 예측 가능한 코드를 작성할 수 있습니다.

### 4. HTTP 메소드에 대해 설명해 주세요.
HTTP 메소드는 클라이언트가 서버에게 어떤 작업을 요청하는지 나타내는 방식입니다.
RESTful API에서는 각 메소드가 명확한 의미와 역할을 가지고 있습니다.

#### 주요 HTTP 메소드
1. GET: 데이터 조회
서버에서 데이터를 가져올 때 사용하며, 안전하고 멱등성을 가집니다.
2. POST: 새로운 데이터 생성
서버에 새로운 리소스를 생성할 때 사용합니다.
3. PUT: 전체 데이터 수정/생성
리소스의 전체 내용을 교체하거나 새로 생성합니다.
4. PATCH: 부분 데이터 수정
리소스의 일부분만 수정할 때 사용합니다.
5. DELETE: 데이터 삭제
서버에서 리소스를 삭제할 때 사용합니다.