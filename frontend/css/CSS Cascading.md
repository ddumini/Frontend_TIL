# CSS Cascading

## Cascading이란?
여러 CSS 규칙이 동일한 요소에 적용될 때, 최종적으로 어떤 스타일을 적용할지 결정하는 알고리즘이다.
"Cascade"는 폭포수를 의미하며, 우선순위에 따라 스타일이 흘러내려가면서 적용되는 모습을 비유한 용어이다.

## Cascade 우선순위 (4단계)

### 1. 출처와 중요도 (Origin and Importance)
**!important가 없는 경우:**
1. Author stylesheets (개발자가 작성한 스타일)
2. User stylesheets (사용자가 설정한 스타일)  
3. User Agent stylesheets (브라우저 기본 스타일)

**!important가 있는 경우:**
1. User !important
2. Author !important
3. User Agent !important

### 2. 명시도 (Specificity)
명시도는 점수로 계산되며, 높은 점수일수록 우선 적용된다.

**명시도 점수 계산:**
- 인라인 스타일: **1000점**
- ID 선택자 (#id): **100점**
- 클래스 (.class), 속성 ([type="text"]), 가상클래스 (:hover): **10점**
- 요소 (div), 가상요소 (::before): **1점**
- 전체 선택자 (*): **0점**

**계산 예시:**
```css
div.content #header    /* 1 + 10 + 100 = 111점 */
#header                /* 100점 */
.content               /* 10점 */
div                    /* 1점 */
```

### 3. 선언 순서 (Order of Appearance)
동일한 명시도를 가진 규칙이 여러 개일 경우, **나중에 선언된 규칙**이 우선 적용된다.

### 4. 상속 (Inheritance)
부모 요소에서 상속받은 속성은 가장 낮은 우선순위를 가진다.
- 상속되는 속성: color, font-family, line-height 등
- 상속되지 않는 속성: margin, padding, border 등

## 실무 주의사항
- **!important 사용 금지**: 디버깅을 어렵게 하고 유지보수성을 해친다
- **인라인 스타일 지양**: HTML과 CSS의 분리 원칙에 위배된다
- **ID 선택자보다 클래스 선택자 권장**: 재사용성과 유연성을 위해
- **명시도를 너무 높이지 않기**: 나중에 재정의하기 어려워진다
