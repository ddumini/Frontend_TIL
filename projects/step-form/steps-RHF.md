# RHF를 사용한 폼 제출 로직

## RHF를 사용하는 이유

유효성 검사를 쉽게 할 수 있는, 성능이 우수하고 유연하며 확장 가능한 form을 제공하는 라이브러리.

1. 성능:
   Formik이나 Redux Form보다 속도가 훨씬 빠르다.
2. 소스 코드가 깔끔하다:
   코드 길이가 다른 라이브러리보다 짧아 syntax가 깔끔하고, 리액트 본연의 syntax와 비슷하여 사용하기 편리하다.
3. 비제어 컴포넌트 방식의 사용으로 불필요한 렌더링을 막아주어 속도가 빠르다:
<details>
<summary>제어 컴포넌트?</summary>
제어 컴포넌트(Control Component)란 <strong>입력 폼 엘리먼트의 값이 React의 state에 의해 제어되는 컴포넌트</strong>를 말한다.  
즉, 사용자의 입력 값이 내부 state에 저장되고, 그 state로부터 입력값을 다시 폼 엘리먼트에 전달하는 방식이다.
onChange 방식이 제어 컴포넌트라고 할 수 있다.
사용자가 입력한 값과 저장되는 값이 실시간으로 동기화되며, 이러한 방식으로 데이터를 전부 받아올 수 있어 유효성 검사에 탁월하지만 데이터를 하나하나 다 받아오므로 비효율적이거나 속도가 느릴 수 있는 단점이 있다.
</details>
<details>
<summary>비제어 컴포넌트?</summary>
비제어 컴포넌트(Uncontrolled Component)란 <strong>DOM 자체에서 폼 데이터가 다루어지는 것</strong>을 말한다.
모든 state 업데이트에 대한 이벤트 핸들러를 작성하는 대신 비제어 컴포넌트를 만들려면 ref를 사용하여 DOM에서 폼 값을 가져올 수 있다.
ref는 값을 업데이트 하여도 리렌더링 되지 않는 특성으로, 입력이 모두 되고난 후 ref를 통해 값을 한번에 가져와서 활용한다.
state로 값을 관리하지 않기 때문에 값이 바뀔 때마다 리렌더링 하지 않고 값을 한번에 가져올 수 있는 성능상 이점이 있으나, 데이터를 완벽하게 가져올 수 없는 단점이 있다.
react hook form은 비제어 컴포넌트로 렌더링을 최적화할 수 있는 라이브러리이다.
단순한 form을 처리하기 위해  state로 모든 값을 검사하여 리렌더링하는 것 보다 입력이 끝난 후 유효성 검사를 보여주어도 되고, 더 빠른 검사를 할 수 있어 RHF가 많이 사용된다.
</details>

## 1. 폼 초기화 및 설정 (apply-provider.tsx)

```jsx
import { zodResolver } from '@hookform/resolvers/zod'
import { ApplyFormData, StepProps } from '@src/pages/apply'
import { createContext, ReactNode, useContext, useEffect } from 'react'
import { useForm, UseFormReturn } from 'react-hook-form'
import { ZodError, ZodSchema } from 'zod'

import { useStepNavigation } from './use-step-navigation'
import { FullFormSchema, Step1Schema, Step2Schema, Step3Schema, Step4Schema } from './validation' // 위에서 생성한 스키마

// Context 타입
export interface ApplyContextType extends StepProps {
  form: UseFormReturn<ApplyFormData>
  validateCurrentStep: () => Promise<boolean>
  getCurrentStepSchema: () => ZodSchema<Partial<ApplyFormData>>
}

const ApplyContext = createContext<ApplyContextType | undefined>(undefined)

export function ApplyProvider({ children }: { children: ReactNode }) {
  const stepNavigation = useStepNavigation()

  // React Hook Form 초기화
  const form = useForm<ApplyFormData>({
    mode: 'onBlur',
    resolver: zodResolver(FullFormSchema),
    defaultValues: {
      kidName: '',
      parentName: '',
      parentPhoneNumber: '',
      kindergartenUUID: '',
      kindergartenName: '',
      roomUUID: '',
      roomName: '',
      packageUUID: '',
      packageName: '',
      serviceStartDate: ''
    }
  })
```

- useForm 훅을 사용하여 폼 인스턴스 생성
- `mode: 'onBlur'`: 포커스를 잃을 때 유효성 검사 실행
- `resolver: zodResolver(FullFormSchema)`: Zod 스키마를 react-hook-form과 통합하여 타입 안전 검증
- `defaultValues`: 모든 필드의 초기값 설정

## 2. 폼 데이터 관찰 및 자동 저장 (apply-provider.tsx)

```jsx
// 클라이언트에서만 sessionStorage 값 복원
useEffect(() => {
  const saved = sessionStorage.getItem('applyFormData');
  if (saved) {
    try {
      const parsedData = JSON.parse(saved);
      form.reset(parsedData);
    } catch (error) {
      console.error('Failed to parse saved form data:', error);
    }
  }
}, [form]);

// 폼 데이터 변경 시 실시간 sessionStorage에 저장
useEffect(() => {
  const subscription = form.watch((data) => {
    if (typeof window !== 'undefined') {
      sessionStorage.setItem('applyFormData', JSON.stringify(data));
    }
  });
  return () => subscription.unsubscribe();
}, [form]);
```

- `form.watch()`: 폼의 모든 필드 변경사항 실시간 관찰
- `subscription` 패턴으로 메모리 누수 방지(언마운트 시 구독 해제)
- `form.reset()`: 저장된 데이터로 폼 전체 초기화
- `sessionStorage`를 활용한 페이지 새로고침 시 데이터 유지

## 3. 단계별 검증 로직 (apply-form.tsx)

```jsx
// 현재 스텝의 스키마 반환
  const getCurrentStepSchema = () => {
    switch (stepNavigation.currentStep) {
      case 1:
        return Step1Schema
      case 2:
        return Step2Schema
      case 3:
        return Step3Schema
      case 4:
        return Step4Schema
      default:
        return Step1Schema
    }
  }

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
  }
```

- form.getValue(): 현재 폼의 모든 값 가져오기
- form.setError(): 프로그래밍 방식으로 에러 설정(ZodError를 React Hook Form 에러로 변환)
- 멀티스텝 폼을 위한 단계별 검증 로직 구현
- 비동기 검증 지원 (parseAsync)

## 4. 기본 입력 필드 등록(steps.tsx)

```jsx
<FormInput
  key={index}
  required
  id={inputId}
  labelText={label}
  placeholder={placeholder}
  type={type}
  {...form.register(fieldName)}
  errorMessage={form.formState.errors[fieldName]?.message}
/>
```

- `form.register()`: 기본적인 필드 등록 방법
- spread 연산자로 onChange, onBlur, name, ref를 실제 FormInput 컴포넌트의 props로 자동 전달, HTML `<input>` 요소로 최종 전달
- form.formState.errors: 에러 객체에서 해당 필드의 에러 메시지 추출
- 텍스트 입력과 같은 단순 필드에 적합

-> RHF의 '비제어 컴포넌트 방식'으로, state를 통한 제어 없이도 폼 데이터를 관리하고, 불필요한 리렌더링을 방지하여 성능을 최적화한다.

## 5. Controller를 사용한 커스텀 입력 제어 (steps.tsx)

```jsx
return (
  // 전화번호 포멧팅
  <Controller
    key={index}
    control={form.control}
    name={fieldName}
    render={({ field }) => (
      <FormInput
        required
        errorMessage={form.formState.errors[fieldName]?.message}
        id={inputId}
        labelText={label}
        placeholder={placeholder}
        type={type}
        value={field.value || ''}
        onBlur={() => field.onBlur()}
        onChange={handleFormattedChange(formatPhoneNumber, field.onChange)}
      />
    )}
  />
);

// 날짜 포멧팅
<Controller
  key={index}
  control={form.control}
  name='serviceStartDate'
  render={({ field }) => (
    <FormInput
      required
      errorMessage={form.formState.errors.serviceStartDate?.message}
      id={inputId}
      labelText={label}
      placeholder={placeholder}
      type='text'
      value={field.value || ''}
      onBlur={() => field.onBlur()}
      onChange={handleFormattedChange(formatDateString, field.onChange)}
    />
  )}
/>;
```

- Controller 컴포넌트: 커스텀 입력 컴포넌트를 react-hook-form과 통합
- control={form.control}: 폼 제어 객체 전달
- render={({field})}: field 객체는 value, onChange, onBlur, name, ref 포함
- 포멧팅 로직을 onChange 핸들러에 주입하여 입력값 반환
- 전화번호(010-1234-5678), 날짜(yyyy-mm-dd)같은 특수 포멧에 사용

## 6. setValue를 사용한 프로그래밍 방식 값 설정 (step.tsx)

```jsx
const handleBottomSheetSelect = (value: string, id: string) => {
  switch (bottomSheetType) {
    case 'kindergarten':
      form.setValue('kindergartenName', value, { shouldValidate: true });
      form.setValue('kindergartenUUID', id, { shouldValidate: true });
      // 기관이 변경되면 반 선택 초기화
      form.setValue('roomName', '');
      form.setValue('roomUUID', '');
      break;
    case 'room':
      form.setValue('roomName', value, { shouldValidate: true });
      form.setValue('roomUUID', id, { shouldValidate: true });
      break;
  }
  setBottomSheetOpen(false);
};
```

```jsx
const handleRadioChange = (index: number, value: string) => {
  // 3단계에서 선택된 패키지 정보 저장
  if (currentStep === 3) {
    const selectedPackage = packages.find((pkg) => pkg.displayValue === value);
    if (selectedPackage) {
      form.setValue('packageName', selectedPackage.displayValue, { shouldValidate: true });
      form.setValue('packageUUID', selectedPackage.UUID, { shouldValidate: true });
    }
  }
};
```

- `form.setValue()`: 특정 필드의 값을 프로그래밍 방식으로 설정
  - BottomSheet, Radio 같은 사용자 입력 방식이 아닌 커스텀 UI에서 선택한 값을 프로그래밍 방식으로 폼에 반영
- `shouldValidate: true`: 값 설정 시 즉시 검증 실행
- 연관 필드 초기화 (기관 변경 시 반 정보 초기화)

## 7. watch를 사용한 실시간 값 관찰 (step.tsx)

```jsx
// React Hook Form의 watch로 현재 값들 가져오기
const formValues = form.watch();
const selectedKindergartenId = formValues.kindergartenUUID || '';

// 선택된 기관의 반 목록 가져오기
const { data: rooms, loading: roomsLoading } = useRooms(selectedKindergartenId);
```

```jsx
<FormInput.Radio
  key={index}
  checked={formValues.packageName === label} // 현재 선택한 값과 비교해서 체크 표시
  id={inputId}
  labelText={label}
  name={`step-${currentStep}-radio`}
  value={label}
  onChange={(e) => {
    handleRadioChange(index, e.target.value);
  }}
/>
```

- `form.watch()`: 모든 필드 값을 객체로 변환 (리액티브)
  - 리액티브: 폼 값이 변경되면 watch()가 감지하여 컴포넌트를 리렌더링하고, UI 자동 업데이트
- `form.watch('fieldName')`: 특정 필드만 관찰 가능
- 값 변경 시 컴포넌트 리렌더링 트리거
- 의존성 있는 데이터 페칭 (기관 선택 -> 해당 기관의 반 목록 로딩)
- 조건부 렌더링 (라디오 버튼 체크 상태)

## 8. 폼 검증 상태 확인 (form-layout.tsx)

```jsx
  // 현재 스텝의 검증 상태 확인
  const isCurrentStepValid = () => {
    const currentFields = getCurrentStepFields()
    const formValues = form.watch()

    // 현재 스텝의 스키마로 검증
    const schema = getCurrentStepSchema()

    try {
      // 현재 스텝 필드만 추출해서 검증
      const currentStepData = currentFields.reduce((acc, field) => {
        acc[field] = formValues[field]
        return acc
      }, {} as Partial<ApplyFormData>)

      schema.parse(currentStepData)
      return true
    } catch {
      return false
    }
  }

  // 버튼 비활성화 조건
  const isButtonDisabled = loading || !isCurrentStepValid()
```

- `form.watch()`와 Zod 스키마를 조합한 실시간 검증 상태 확인
- 단계별로 필요한 필드만 추출하여 검증 (예: Step1에서 2, 3, 4의 필드까지 검증되지 않게 하기 위함 -> 각 단계를 독립적으로 검증)

  - reduce 실행 순서 (누적 값을 acc에 객체 형태로 저장)

  ```jsx
  const currentStepData = currentFields.reduce((acc, field) => {
    acc[field] = formValues[field]
    return acc
  }, {} as Partial<ApplyFormData>)

  // 초기값: {} (빈 객체)
  let acc = {}

  // 1번째 반복: field = 'kidName'
  acc['kidName'] = formValues['kidName']  // "홍길동"
  // acc = { kidName: "홍길동" }

  // 2번째 반복: field = 'parentName'
  acc['parentName'] = formValues['parentName']  // "홍부모"
  // acc = { kidName: "홍길동", parentName: "홍부모" }

  // 3번째 반복: field = 'parentPhoneNumber'
  acc['parentPhoneNumber'] = formValues['parentPhoneNumber']  // "010-1234-5678"
  // acc = {
  //   kidName: "홍길동",
  //   parentName: "홍부모",
  //   parentPhoneNumber: "010-1234-5678"
  // }

  // 최종 결과 = currentStepData
  ```

  - `schema.parse(currentStepData)`: currentStepData를 검증하여 유효하지 않으면 catch로 `return false`

- 최종 결과 `currentStepData` 값을 버튼 활성화/비활성화 상태 결정에 활용

## 9. 폼 데이터 제출 (form-layout.tsx)

```jsx
// 버튼 클릭 핸들러 - form.handleSubmit 대신 직접 처리
const handleButtonClick = async () => {
  try {
    // throw new Error('에러 테스트')
    if (isSubmitStep) {
      // 4단계에서 최종 제출 처리
      const formValues = form.getValues();
      const transformedData: KidRegistration = {
        kidName: formValues.kidName,
        parentName: formValues.parentName,
        parentPhoneNumber: formValues.parentPhoneNumber,
        kindergartenUUID: formValues.kindergartenUUID,
        roomUUID: formValues.roomUUID,
        packageUUID: formValues.packageUUID,
        serviceStartDate: formValues.serviceStartDate,
      };
      await registerKid(transformedData); // 실제 API 전송
    } else {
      // 1-3단계에서는 다음 단계로 이동
      goNext?.();
    }
  } catch {
    setIsError(true);
  }
};
```

- `form.getValues()`: 폼의 현재 모든 값을 한 번에 가져오기
- `form.handleSubmit()`대신 커스텀 제출 로직 구현
- 멀티스텝 폼에서 마지막 단계에만 실제 제출 수행
  - `form.handleSubmit()`의 한계
    1. 버튼 클릭 시 항상 전체 폼 검증 시도
    2. 다음 버튼도 제출 동작으로 처리되어 불필요한 제출 발생
    3. 멀티스텝 로직을 구현하기 어려움
- API 전송을 위한 데이터 변환
