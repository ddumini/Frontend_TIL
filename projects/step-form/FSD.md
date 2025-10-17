# FSD

## FSD 아키텍처

[FSD](frontend/architecture/FSD%20아키텍처.md)는 프론트엔드 프로젝트를 위한 아키텍처 방법론으로, Layer(최상위 폴더), Slice(비즈니스 도메인 기준으로 분류한 하위 폴더), Segment(해당 코드의 기능 목적에 따라 분류한 하위 폴더) 로 코드를 분리하고 구성할 수 있다.

- 도메인별로 코드를 구분해 필요한 로직을 즉시 찾을 수 있다.
- 구성 요소를 간편하게 교체, 확장한다.
- 모듈을 독립적으로 수정, 리팩토링해 부작용을 없앤다
- 명시적 코드 재사용: DRY(Don't Repeat Yourself) 원칙과 로컬 커스터마이징 사이에서 균형을 잡는다.

이런 특징을 통해 이해하기 쉬운 코드로 유지보수하기 쉽고, 확장성과 재사용성이 높은 코드를 작성할 수 있다.

의존성, 캡슐화, 계층적 흐름을 통한 견고한 설계가 가능하다.

## 전체 아키텍쳐 개요

```
프로젝트 루트/
├── app/                    # Next.js App Router (라우팅 레이어)
│   ├── page.ts            # 라우트 진입점
│   ├── apply/page.ts
│   └── complete/page.ts
│
└── src/                    # FSD 구조 (비즈니스 로직 레이어)
    ├── app/               # App 레이어 (전역 설정)
    │   ├── layouts/
    │   └── styles/
    │
    ├── pages/             # Pages 레이어 (페이지별 기능)
    │   ├── home/
    │   ├── apply/
    │   └── complete/
    │
    └── shared/            # Shared 레이어 (공통 기능)
        ├── ui/
        ├── models/
        ├── utils/
        └── mock/
```

## FSD 레이어 구조와 역할 (with Next.js App Router)

### Layer1: App(src/app/)

- 역할: 전역 레이아웃, 스타일, 프로바이더 등 애플리케이션 전반에 영향을 미치는 설정
- 특징: 다른 레이어에 의존할 수 었음

```
src/app/
├── layouts/
│   ├── index.ts          # Public API
│   └── layout.tsx        # 전역 레이아웃 구현
└── styles/
    └── globals.css
```

### Layer2: Pages(src/pages/)

- 역할: 각 페이지별 기능을 독립적으로 관리
- 특징: Shared Layer에만 의존, 다른 Pages와는 독립적

```
src/pages/
├── home/
│   ├── index.ts          # Public API (reexport)
│   └── ui/
│       ├── home-page.tsx
│       ├── home-page.module.css
│       ├── apply-button.tsx
│       └── apply-button.module.css
│
├── apply/
│   ├── index.ts          # Public API (reexport)
│   ├── models/           # 비즈니스 로직, 상태 관리
│   │   ├── apply-provider.tsx
│   │   ├── types.ts
│   │   ├── validation.ts
│   │   ├── constants.ts
│   │   └── use-step-navigation.ts
│   └── ui/               # UI 컴포넌트
│       ├── apply-page.tsx
│       ├── form-layout.tsx
│       ├── steps.tsx
│       └── [각 컴포넌트의 CSS 모듈]
│
└── complete/
    ├── index.ts          # Public API (reexport)
    └── ui/
        ├── complete-page.tsx
        ├── complete-title.tsx
        └── apply-info.tsx
```

### Layer3: Shared(src/shared/)

- 역할: 프로젝트 전반에서 재사용되는 공통 기능 관리
- 특징: 다른 Layers에 의존할 수 없음, 순수한 공통 모듈

```
src/shared/
├── ui/                   # 공통 UI 컴포넌트
│   ├── index.ts
│   ├── button/
│   ├── bottom-sheet/
│   ├── form-inputs/
│   │   ├── index.ts      # 복합 컴포넌트 조합
│   │   ├── input.tsx
│   │   ├── radio.tsx
│   │   └── select.tsx
│   └── [기타 UI 컴포넌트들]
│
├── models/               # 공통 비즈니스 로직
│   ├── index.ts
│   ├── types.ts
│   └── use-api.ts
│
├── utils/                # 유틸리티 함수
│   ├── index.ts
│   ├── format.ts
│   └── use-modal-keyboard.ts
│
└── mock/                 # Mock 데이터
    ├── index.ts
    ├── kindergartens.ts
    ├── packages.ts
    └── rooms.ts
```

## Public API 패턴 (Reexport)

FSD의 핵심 원칙은 각 슬라이스는 Public API를 노출하여 외부에서 사용할 수 있도록 하는 것이다.

### 기본 Reexport 패턴

예시: src/pages/apply/index.ts

```jsx
// Public API - 외부에서 사용할 수 있는 것만 노출
export { ApplyProvider, useApplyContext } from './models/apply-provider';
export type { ApplyFormData, StepProps } from './models/types';
export { default as ApplyPage } from './ui/apply-page';
```

사용 방법

```jsx
// 올바른 사용 (Public API를 통한 import)
import { ApplyPage, ApplyProvider, useApplyContext } from '@src/pages/apply';

// 잘못된 사용 (내부 구조 직접 접근)
import ApplyPage from '@src/pages/apply/ui/apply-page';
```

### 복합 컴포넌트

예시: src/shared/ui/form-inputs/index.ts

```jsx
import Input from './input';
import Radio from './radio';
import Select from './select';

// Object.assign으로 복합 컴포넌트 생성
export const FormInput = Object.assign(Input, {
  Radio,
  Select,
});
```

- Object.assign으로 네이밍스페이스 패턴을 적용하여 컴포넌트를 생성한다.

```jsx
//Object.assign 동작 원리
// 1. 기본 Input 컴포넌트
function Input(props) {
  return <input {...props} />;
}

// 2. Object.assign으로 속성 추가
export const FormInput = Object.assign(Input, {
  Radio: RadioComponent,
  Select: SelectComponent,
});

// 3. 결과적으로 생성되는 객체
FormInput = {
  // Input 함수 자체가 기본 호출 가능
  (...props) => <input {...props} />,

  // 추가된 속성들
  Radio: RadioComponent,
  Select: SelectComponent,
}
```

사용 방법:

```jsx
import { FormInput } from '@src/shared/ui'

// 기본 Input
<FormInput name="name" />

// Radio (복합)
<FormInput.Radio name="gender" />

// Select (복합)
<FormInput.Select name="room" />
```

## Next.js App Router와의 통합

### 라우팅 레이어와 FSD 레이어 분리

- Next 라우트 파일 (src/app/page.tsx)

```jsx
// 라우트 파일은 단순히 FSD 페이지를 reexport
export { ApplyPage as default } from '@src/pages/apply';
```

- FSD 페이지 파일 (src/pages/apply/ui/apply-page.tsx)

```jsx
import { Container } from '@src/shared/ui';
import { ApplyProvider } from '../models/apply-provider';
import FormLayout from './form-layout';

export default function ApplyPage() {
  return (
    <ApplyProvider>
      <main>
        <h1 className='sr-only'>신청 정보 입력 페이지</h1>
        <Container>
          <FormLayout />
        </Container>
      </main>
    </ApplyProvider>
  );
}
```

- Next.js 라우팅 시스템과 FSD 구조가 독립적으로 유지되어 라우트 변경 시 비즈니스 로직에 영향 없음

### Path Alias 설정

- tsconfig.js

```js
{
  "compilerOptions": {
    "paths": {
      "@/*": ["./app/*"],       // Next.js 라우트 레이어
      "@src/*": ["./src/*"]     // FSD 레이어
    }
  }
}
```
