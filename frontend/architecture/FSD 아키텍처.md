# Feature-Sliced Design (FSD) 아키텍처

프론트엔드 프로젝트를 위한 아키텍처 방법론
코드를 어떻게 분리하고 구성할지를 명확히 정의하여, 변화하는 비즈니스 요구 속에서도 프로젝트를 이해하기 쉽고 안정적으로 유지할 수 있다.

- 프론트엔드(웹, 모바일, 데스크톱 등)을 개발할 때
- 라이브러리가 아닌 애플리케이션을 개발할 때

## 특징

1. 명시적 비즈니스 로직
   도메인별로 코드를 구분해 필요한 로직을 즉시 찾을 수 있다.
2. 유연성
   요구사항이 바뀌어도 구성 요소를 간편하게 교체, 확장한다.
3. 기술 부채 최소화
   모듈을 독립적으로 수정, 리팩토링해 부작용을 없앤다.
4. 명시적 코드 재사용
   DRY(Don't Repeat Yourself) 원칙과 로컬 커스터마이징 사이에서 균형을 잡는다.

## 개념

1. 공용 API
   모든 모듈은 최상위 수준에 공용 API를 노출해야 한다.
2. 격리
   같은 Layer 또는 상위 Layer에 직접 의존하지 않는다.
3. 요구사항 중심
   비즈니스와 사용자 요구를 설계의 기준으로 삼는다.

## 구조

Feature-Sliced Design: Concept and Importance

![FSD 아키텍처 개념도](/images/image_fsd-architecture-diagram.png)

## 구조 예시

간단한 FSD 구조는 다음과 같다.

- `📁 app`
- `📁 pages`
- `📁 shared`
  이 상위 폴더들은 각각 **Layer**에 해당한다.

- `📂 app`

  - `📁 routes`
  - `📁 analytics`

- `📂 pages`

  - `📁 home`
  - `📂 article-reader`
    - `📁 ui`
    - `📁 api`
  - `📁 settings`

- `📂 shared`

  - `📁 ui`
  - `📁 api`

- `📂 pages` 내부 폴더들은 **Slice**이다.
  일반적으로 도메인(이 예시에서는 페이지) 기준으로 구분된다.

- `📂 app`, `📂 shared`, `📂 pages/article-reader` 내의 하위 폴더들은 **Segment**이다.
  Segment는 해당 코드의 기능 목적(UI, API 통신 등)에 따라 분류한다.

## Layer

Layer는 모든 FSD 프로젝트의 표준 최상위 폴더이다.

1. **APP** - Routing, Entrypoint, Global Styles, Provider 등 앱을 실행하는 모든 요소
2. **Pages** - 전체 page 또는 중첩 Routing의 핵심 영역
3. **Widgets** - 독립적으로 동작하는 대형 UI, 기능 블록
4. **Features** - 제품 전반에서 재사용되는 비즈니스 기능
5. **Entities** - user, product 같은 핵심 도메인 Entity
6. **Shared** - 프로젝트 전반에서 재사용되는 일반 유틸리티

App, Shared Layer는 Slice 없이 곧바로 Segment로 구성된다.
상위 Layer의 모듈은 **자신보다 하위 Layer만 참조**할 수 있다.

## Slice

Slice는 Layer 내부를 비즈니스 도메인별로 나눈다.
이름, 개수에 제한이 없으며, **같은 Layer내 다른 Slice를 참조할 수 없다.**
이 규칙이 **높은 응집도와 낮은 결합도**를 보장한다.

> 💡높은 응집도와 낮은 결합도?
>
> - **높은 응집도(High Cohesion)**: 한 모듈(또는 Slice)이 하나의 목적, 하나의 책임에 집중되어 있는 정도를 말한다.
>   즉, 관련 기능끼리 모여있다는 것을 말한다.
>   예시)
>   `user` Slice 안에는 회원가입, 로그인, 프로필 수정 등 "유저"와 관련된 로직만,
>   `product` Slice 안에는 상품 조회, 상품 등록, 상품 수정 등 "상품"과 관련된 로직만 있다.
>
> - **낮은 결합도(Low Coupling)**: 한 모듈이 다른 모듈에 의존하는 정도
>   즉, 다른 Slice에 의존하지 않아도 동작할 수 있는 상태를 말한다.
>   예시)
>   `user` Slice 내부 로직이 `product` Slice의 내부 함수나 상태를 가져다 쓰면 결합도가 높음.
>   반대로, `user`와 `product`가 서로의 내부를 모른 채 공용 인터페이스로만 소통한다면 결합도가 낮음.

## Segment

Slice와 App, Shared Layer는 Segment로 세분화되어, 기술적 목적에 따라 코드를 그룹화한다.

- `ui` - UI components, date formatter, styles 등 UI 표현과 직접 관련된 코드
- `api` - request functions, data types, mapper 등 백엔드 통신 및 데이터 로직
- `model` - schema, interfaces, store, business logic 등 애플리케이션 도메인 모델
- `lib` - 해당 Slice에서 여러 모듈이 함께 사용하는 공통 library code
- `config` - configuration files, feature flags 등 환경, 기능 설정

대부분의 Layer에서는 위 다섯 Segment로 충분하다.
필요하다면 App 또는 Shared Layer에서만 추가 Segment를 정의한다.
