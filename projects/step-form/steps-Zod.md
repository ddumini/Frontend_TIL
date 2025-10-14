# Zod 사용 폼 검증

## Zod를 도입한 이유

스키마 선언 및 유효성 검사 라이브러리로, 컴파일 시점에서의 타입 에러만 잡아내는 TypeScript의 한계로 런타임 단계에서의 타입 에러를 잡아낸다.
네트워크 통신 시 외부 시스템으로부터의 데이터는 신뢰할 수 없기 때문에 그로 인해 발생하는 타입 에러를 잡아낼 수 있다.

- 원하는 숫자 범위, 정수/실수 구분, 원하는 문자열로 강제하는 검증 기능을 제공한다.

- parse, parseAsync 메서드를 사용하여 데이터를 스키마에 맞게 검증한다.
  유효성 검증을 통과할 경우 그대로 입력 값을 return하고, 유효성 검증을 통과하지 못할 경우 ZodError 객체를 throw한다.

- ZodError 객체는 유효성 검증을 통과하지 못한 필드와 그 필드의 에러 메시지를 포함하고 있다.

- Zod는 스키마를 기준으로 타입을 추론할 수 있다.
  이 기능을 통해 타입을 따로 작성할 필요가 없어진다.
  - `z.infer`: 스키마의 타입을 추론해주는 유틸리티 타입

## 1. 기본 스키마 정의 - 문자열 검증 (validation.ts)

```tsx
import { z } from 'zod';

const koreanNameSchema = z
  .string()
  .min(2, '이름은 2글자 이상이어야 합니다')
  .max(10, '이름은 10글자 이하여야 합니다')
  .regex(/^[가-힣]+$/, '완성된 한글만 입력해주세요');
```

- z.string(): 문자열 타입 정의
- .min(길이, 에러메시지): 최소 길이 제약
- .max(길이, 에러메시지): 최대 길이 제약
- .regex(정규식, 에러메시지): 정규식 패턴 검증
- 체이닝 방식으로 여러 검증 규칙 조합
- 한글 이름 검증 - 완성된 한글만 허용하고, 2-10자 제한

## 2. 정규식을 사용한 포멧 검증 (validation.ts)

```tsx
const phoneSchema = z.string().regex(/^010-\d{4}-\d{4}$/, '올바른 휴대폰 번호를 입력해주세요');
```

- .regex(): 특정 포멧 강제
- 010-xxxx-xxxx 형식만 허용
- 하이픈 포함한 정확한 포멧 검증
- 한국 휴대폰 번호 형식 강제

## 3. 내장 검증 타입 - UUID (validation.ts)

```tsx
const uuidSchema = z.string().uuid();
```

- .uuid(): Zod의 내장 UUID 검증기
- 별도 정규식 없이 간단하게 UUID 형식 검증
- 표준 UUID 포멧 자동 체크
- API에서 받은 ID 값의 유효성 검증

## 4. 객체 스키마 정의 (validation.ts)

```tsx
export const Step1Schema = z.object({
  kidName: koreanNameSchema,
  parentName: koreanNameSchema,
  parentPhoneNumber: phoneSchema,
});
```

```tsx
export const Step2Schema = z.object({
  kindergartenUUID: uuidSchema,
  kindergartenName: z.string().min(1, '기관명을 선택해주세요'),
  roomUUID: uuidSchema,
  roomName: z.string().min(1, '반을 선택해주세요'),
});
```

- z.object(): 객체 스키마 정의
- 각 키에 대한 검증 규칙 지정
- 재사용 가능한 스키마를 조합하여 사용
- .min(1): 빈 문자열 방지 (필수 필드)
- 단계별 폼 데이터 구조 정의 및 검증

## 5. 커스텀 검증 로직 (validation.ts)

```tsx
export const Step3Schema = z.object({
  packageUUID: uuidSchema,
  packageName: z.string().min(1, '패키지를 선택해주세요'),
});

export const Step4Schema = z.object({
  serviceStartDate: z
    .string()
    .regex(/^\d{4}-\d{2}-\d{2}$/, 'YYYY-MM-DD 형식으로 입력해주세요')
    .refine((date) => {
      const parsedDate = new Date(date);
      return !isNaN(parsedDate.getTime());
    }, '유효한 날짜를 입력해주세요')
    .refine((date) => new Date(date) > new Date(), '오늘 이후의 날짜로 입력해주세요'),
});
```

- .refine(콜백함수, 에러메시지): 커스텀 검증 로직 작성
- 첫 번째 refine: 실제로 유효한 날짜인지 확인
- 두 번째 refine: 비즈니스 로직 검증 (오늘 이후의 날짜만 허용)
- 여러 refine을 체이닝하여 순차적 검증 가능

## 6. 타입 추론 (validation.ts)

```tsx
// 타입 생성
export type Step1FormData = z.infer<typeof Step1Schema>;
export type Step2FormData = z.infer<typeof Step2Schema>;
export type Step3FormData = z.infer<typeof Step3Schema>;
export type Step4FormData = z.infer<typeof Step4Schema>;
export type ApplyFormData = Step1FormData & Step2FormData & Step3FormData & Step4FormData;
```

- z.infer<typeof Schema>: Zod 스키마에서 TypeScript 타입 자동 생성
- 스키마와 타입이 항상 동기화됨(Single Source of Truth)
- 타입 안전성과 런타임 검증을 동시에 확보
- &로 여러 타입 결합
- 중복 타입 정의 방지

## 7. 스키마 병합 (validation.ts)

```tsx
// 전체 폼 스키마
export const FullFormSchema = Step1Schema.merge(Step2Schema).merge(Step3Schema).merge(Step4Schema);
```

- .merge(): 여러 객체 스키마를 하나로 결합
- 체이닝으로 다중 스키마 병합 가능
- 멀티스텝 폼에서 전체 데이터 검증 시 유용
- 단계별로 나뉜 스키마를 최종 제출 시 전체 검증에 사용
- 코드 모듈화 및 재사용성 향상

## 8. 스키마 재사용 (validation.ts)

```tsx
// API 전송용 스키마
export const KidRegistrationSchema = z.object({
  kidName: koreanNameSchema,
  parentName: koreanNameSchema,
  parentPhoneNumber: phoneSchema,
  kindergartenUUID: uuidSchema,
  roomUUID: uuidSchema,
  packageUUID: uuidSchema,
  serviceStartDate: Step4Schema.shape.serviceStartDate,
});
```

- .shape.필드명: 기존 스키마의 특정 필드만 추출하여 재사용
- API 전송용으로 필요한 필드만 선별
- displayValue같은 UI에 표시될 필드 제거
- 프론트엔드 폼 데이터와 API 요청 데이터 구조가 다를 때
- 필요한 검증 규칙만 선택적으로 사용

## 9. React Hook Form과 통합 (apply-provider.tsx)

```tsx
import { zodResolver } from '@hookform/resolvers/zod'
// React Hook Form 초기화
  const form = useForm<ApplyFormData>({
    mode: 'onBlur',
    resolver: zodResolver(FullFormSchema),
    defaultValues: {...

```

- @hookform/resolvers/zod: React Hook Form과 Zod를 연결하는 패키지 사용
- zodResolver: Zod 스키마를 React Hook Form의 resolver로 변환
- 폼 제출/변경 시 자동으로 Zod 검증 실행
- 검증 실패 시 에러를 formState.errors에 자동 매핑
- 이는 RHF의 검증 엔진으로 Zod를 사용할 수 있게 해준다.
- 선언적 검증 및 타입 안전성이 확보된다.

## 10. 동기 검증 (form-layout.tsx)

```tsx
// 현재 스텝의 스키마로 검증
const schema = getCurrentStepSchema();

try {
  // 현재 스텝 필드만 추출해서 검증
  const currentStepData = currentFields.reduce((acc, field) => {
    acc[field] = formValues[field];
    return acc;
  }, {} as Partial<ApplyFormData>);

  schema.parse(currentStepData);
  return true;
} catch {
  return false;
}
```

- schema.parse(데이터): 동기식 방식 검증
- 검증 성공 시 파싱된 데이터 반환
- 검증 실패 시 ZodError 객체 발생
- try-catch로 에러 처리
- 버튼 활성화/비활성화 상태 실시간 체크
- UI 상태 제어를 위한 빠른 검증

## 11. 비동기 검증 및 ZodError 처리 (apply-provider.tsx)

```tsx
// 현재 스텝 검증
  const validateCurrentStep = async (): Promise<boolean> => {
    const schema = getCurrentStepSchema()
    const currentData = form.getValues()

    try {
      await schema.parseAsync(currentData) // 현재 데이터에 대한 비동기 검증
      return true
    } catch (error) {
      // 에러를 폼에 설정
      if (error instanceof ZodError) {
        // Zod 에러를 React Hook Form 에러로 변환
        error.issues.forEach((issue) => {
          form.setError(issue.path[0] as keyof ApplyFormData, {
            type: 'validation',
            message: issue.message
          })
        })
      }
      return false
    }
```

- schema.parseAsync(데이터): 비동기 방식 검증
- Promise 반환 - async/await 사용
- ZodError 인스턴스 체크로 타입 가드
  - error.issues: 모든 검증 에러 실패 항목 베열
  ```tsx
  // error (ZodError 객체 구조)
  {
    name: "ZodError",
    message: "Validation error",
    issues: [
      {
        path: ['kidName'],
        message: "이름은 2글자 이상이어야 합니다",
        code: "too_small",
        minimum: 2,
        type: "string",
        inclusive: true
      },
      {
        path: ['parentPhoneNumber'],
        message: "전화번호 형식 오류",
        code: "invalid_string",
        validation: "regex"
      }
    ]
  }
  ```
  - issue.path: 실패한 필드의 경로
  - issue.message: 에러 메시지
- 각 에러를 RHF의 setError로 전달
- 단계 이동 전 검증
- 서버 검증이 필요한 경우 대비
- 에러를 폼 상태에 통합
