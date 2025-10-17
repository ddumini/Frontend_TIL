# Context API를 사용한 멀티스텝 폼 상태 관리

## Context API를 도입한 이유

Context API는 프로젝트의 상위/하위 컴포넌트 간 데이터 공유 방식으로 특정 컴포넌트에서 제공하는 데이터를 자식을 포함한 하위 컴포넌트에서 사용할 수 있게 해주는 React의 내장 기능이다.

해당 프로젝트는 단일 기능 범위로 상태가 /apply 페이지 내에서만 사용된다.
따라서 다른 페이지나 컴포넌트 트리 간 공유가 필요하지 않아 전역 상태 관리 라이브러리를 사용하지 않고 Context API를 사용하여 상태를 관리한다.

또 상태 구조가 단순한 스텝 네비게이션과 폼 상태 관리이므로 zustand의 셀렉터 패턴이 필요 없는 단순한 구조이다.

Context API는 의존성이 최소화되어 있어 번들 크기를 최적화할 수 있다. (최소 의존성 원칙)

Zustand가 필요한 경우는 다음과 같다.

1. 여러 페이지에서 공유되는 인증, 프로필 등 전역 사용자 상태가 필요할 때
2. 여러 API 호출과 상태 조합이 필요한 복잡한 비동기 로직이 필요할 때
3. 대규모 리스트나 잦은 업데이트로 인한 리렌더링 이슈에 대해 성능 최적화가 필요할 때
4. DevTools에서 복잡한 상태 디버깅이 필요한 경우

## 구현부

### 1. 커스텀 Hook: useStepNavigation

```tsx
import { useEffect, useState } from 'react';

export function useStepNavigation() {
  // 서버와 클라이언트 모두 동일한 초기값 사용
  const [currentStep, setCurrentStep] = useState(1);
  const totalSteps = 4;

  // 클라이언트에서만 sessionStorage 값 적용
  useEffect(() => {
    const saved = sessionStorage.getItem('currentStep');
    if (saved) {
      const savedStep = parseInt(saved, 10);
      if (savedStep >= 1 && savedStep <= totalSteps) {
        setCurrentStep(savedStep);
      }
    }
  }, [totalSteps]);

  // 단계를 sessionStorage에도 반영
  const goNext = () => {
    const nextStep = Math.min(currentStep + 1, totalSteps);
    setCurrentStep(nextStep);
    if (typeof window !== 'undefined') {
      sessionStorage.setItem('currentStep', nextStep.toString());
    }
  };

  const goPrev = () => {
    const prevStep = Math.max(currentStep - 1, 1);
    setCurrentStep(prevStep);
    if (typeof window !== 'undefined') {
      sessionStorage.setItem('currentStep', prevStep.toString());
    }
  };

  // 폼 작성 완료 후 처음으로 되돌릴 때
  // 상태와 sessionStorage 모두 초기화
  const resetStep = () => {
    setCurrentStep(1);
    if (typeof window !== 'undefined') {
      sessionStorage.setItem('currentStep', '1');
    }
  };

  return {
    currentStep,
    totalSteps,
    goNext,
    goPrev,
    resetStep,
    isFirstStep: currentStep === 1,
    isLastStep: currentStep === totalSteps,
  };
}
```

#### Next.js SSR 대응 패턴

```tsx
// 초기값은 서버와 클라이언트 동일하게
const [currentStep, setCurrentStep] = useState(1);

// 클라이언트에서만 복원
useEffect(() => {
  const saved = sessionStorage.getItem('currentStep');
  if (saved) {
    setCurrentStep(parseInt(saved, 10));
  }
}, []);
```

- Next.js는 서버에서 먼저 렌더링 후 클라이언트로 hydration
- 서버에는 window, sessionStorage 객체가 없음
- Hydration Mismatch 방지를 위해 서버와 클라이언트의 초기 렌더링 결과가 일치하게 한다.

#### 양방향 동기화 패턴

```tsx
const goNext = () => {
  const nextStep = Math.min(currentStep + 1, totalSteps);

  // 1. React 상태 업데이트
  setCurrentStep(nextStep);

  // 2. sessionStorage 동기화
  if (typeof window !== 'undefined') {
    sessionStorage.setItem('currentStep', nextStep.toString());
  }
};
```

- React 상태와 sessionStorage를 항상 동기화
- typeof window !== 'undefined' SSR 환경 안전 체크
- Math.min/max: 경계값 보호 (1~4 범위 강제)

#### Computed Values 패턴

```tsx
return {
  currentStep,
  totalSteps,
  goNext,
  goPrev,
  resetStep,
  isFirstStep: currentStep === 1, // ← 계산된 값
  isLastStep: currentStep === totalSteps, // ← 계산된 값
};
```

- 자주 사용되는 조건 로직을 미리 계산하여 제공
- 사용하는 쪽에서 currentStep === 1 같은 중복 코드 제거
- 가독성 향상

### 2. Context 정의 및 Provider: ApplyProvider

```tsx
'use client';

import { zodResolver } from '@hookform/resolvers/zod';
import { ApplyFormData, StepProps } from '@src/pages/apply';
import { createContext, ReactNode, useContext, useEffect } from 'react';
import { useForm, UseFormReturn } from 'react-hook-form';
import { ZodError, ZodSchema } from 'zod';

import { useStepNavigation } from './use-step-navigation';
import { FullFormSchema, Step1Schema, Step2Schema, Step3Schema, Step4Schema } from './validation'; // 위에서 생성한 스키마

// Context 타입 - React Hook Form 메서드들 추가
export interface ApplyContextType extends StepProps {
  form: UseFormReturn<ApplyFormData>;
  validateCurrentStep: () => Promise<boolean>;
  getCurrentStepSchema: () => ZodSchema<Partial<ApplyFormData>>;
}

const ApplyContext = createContext<ApplyContextType | undefined>(undefined);

export function ApplyProvider({ children }: { children: ReactNode }) {
  const stepNavigation = useStepNavigation();

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
      serviceStartDate: '',
    },
  });

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

  // 현재 스텝의 스키마 반환
  const getCurrentStepSchema = () => {
    switch (stepNavigation.currentStep) {
      case 1:
        return Step1Schema;
      case 2:
        return Step2Schema;
      case 3:
        return Step3Schema;
      case 4:
        return Step4Schema;
      default:
        return Step1Schema;
    }
  };

  // 현재 스텝 검증
  const validateCurrentStep = async (): Promise<boolean> => {
    const schema = getCurrentStepSchema();
    const currentData = form.getValues();

    try {
      await schema.parseAsync(currentData); // 현재 데이터에 대한 비동기 검증
      return true;
    } catch (error) {
      // 에러를 폼에 설정
      if (error instanceof ZodError) {
        // Zod 에러를 React Hook Form 에러로 변환
        error.issues.forEach((issue) => {
          form.setError(issue.path[0] as keyof ApplyFormData, {
            type: 'validation',
            message: issue.message,
          });
        });
      }
      return false;
    }
  };

  return (
    <ApplyContext.Provider
      value={{
        ...stepNavigation,
        form,
        validateCurrentStep,
        getCurrentStepSchema,
      }}>
      {children}
    </ApplyContext.Provider>
  );
}

// eslint-disable-next-line react-refresh/only-export-components
export function useApplyContext() {
  const context = useContext(ApplyContext);
  if (context === undefined) {
    throw new Error('useApplyContext must be used within an ApplyProvider');
  }
  return context;
}
```

#### Context 타입 정의

```tsx
export interface ApplyContextType extends StepProps {
  form: UseFormReturn<ApplyFormData>          // RHF 폼 인스턴스
  validateCurrentStep: () => Promise<boolean>  // 검증 함수
  getCurrentStepSchema: () => ZodSchema<...>   // 스키마 getter
}
```

- extends StepProps: 스텝 관련 타입 상속
- 폼 상태와 네비게이션 상태를 하나의 Context로 통합
- TypeScript 타입 안정성 확보

#### 여러 상태의 조합 패턴

```tsx
export function ApplyProvider({ children }: { children: ReactNode }) {
  const stepNavigation = useStepNavigation()  // 1. 스텝 상태
  const form = useForm<ApplyFormData>({...})  // 2. 폼 상태

  return (
    <ApplyContext.Provider
      value={{
        ...stepNavigation,  // 스텝 관련 모든 값/함수
        form,               // 폼 인스턴스
        validateCurrentStep,
        getCurrentStepSchema
      }}
    >
      {children}
    </ApplyContext.Provider>
  )
}
```

- Compound Provider 패턴: 여러 로직을 하나의 Provider로 통합
- ...stepNavigation: spread로 모든 프로퍼티 전달
- 관련있는 상태들을 논리적으로 그룹화

#### 폼 데이터 자동 저장

```tsx
useEffect(() => {
  const subscription = form.watch((data) => {
    if (typeof window !== 'undefined') {
      sessionStorage.setItem('applyFormData', JSON.stringify(data));
    }
  });
  return () => subscription.unsubscribe();
}, [form]);
```

- form.watch(): 폼의 모든 필드 변경을 구독
- 사용자가 타이핑할 때마다 자동 저장
- Subscription 정리: 컴포넌트 언마운트 시 구독 해제
- 새로고침해도 데이터 유지되는 사용자 경험

#### 폼 데이터 복원

```tsx
useEffect(() => {
  const saved = sessionStorage.getItem('applyFormData');
  if (saved) {
    try {
      const parsedData = JSON.parse(saved);
      form.reset(parsedData); // 폼 전체를 저장된 값으로 초기화
    } catch (error) {
      console.error('Failed to parse saved form data:', error);
    }
  }
}, [form]);
```

- form.reset(data): 폼의 모든 필드를 한 번에 설정
- try-catch로 잘못된 JSON 데이터 처리
- 새로고침시 작성하던 내용 자동 복원

#### 커스텀 Hook으로 Context 소비

```tsx
export function useApplyContext() {
  const context = useContext(ApplyContext);
  if (context === undefined) {
    throw new Error('useApplyContext must be used within an ApplyProvider');
  }
  return context;
}
```

- Context를 직접 사용하지 않고 커스텀 Hook으로 감싸기
- 에러 초기 발견: Provider 밖에서 사용 시 명확한 에러 메시지
- 타입 안전성: undefined 체크로 타입 가드

## 사용부

### 1. ApplyPage - Provider로 감싸기

```tsx
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

- Context를 사용할 최상위 레벨에서 Provider 설정
- `<ApplyProvider>` 내부의 모든 컴포넌트가 Context에 접근 가능
- Next.js App Router의 'use client' 경계 역할

### 2. ProgressBar - 진행바

```tsx
import { useApplyContext } from '../models/apply-provider';
import styles from './progress.module.css';

export default function Progress() {
  const { currentStep, totalSteps } = useApplyContext();
  const progress = currentStep && totalSteps ? (currentStep / totalSteps) * 100 : 0;

  return (
    <div className={styles.progress}>
      <div className={styles.progressBar} style={{ width: `${progress}%` }} />
    </div>
  );
}
```

- Context에서 currentStep, totalSteps만 선택적으로 추출
- 진행률 계산
- props drilling 없이 깊은 곳에서도 상태 접근

### 3. Navigation - 뒤로가기/메인으로

```tsx
export default function Navigation() {
  const { currentStep, goPrev } = useApplyContext();
  const router = useRouter();
  const isFirstStep = currentStep === 1;

  // 컨펌 모달 상태와 액션 타입 관리
  const [confirmModal, setConfirmModal] = useState<{
    isOpen: boolean;
    action: 'back' | 'home' | null;
  }>({
    isOpen: false,
    action: null,
  });

  // 뒤로가기 버튼 클릭
  const handleBackClick = () => {
    if (isFirstStep) {
      // 첫 번째 스텝에서는 메인으로 이동 컨펌
      setConfirmModal({
        isOpen: true,
        action: 'home',
      });
    } else {
      // 다른 스텝에서는 이전 스텝으로 이동 컨펌
      setConfirmModal({
        isOpen: true,
        action: 'back',
      });
    }
  };

  // 메인으로 버튼 클릭
  const handleMainClick = () => {
    setConfirmModal({
      isOpen: true,
      action: 'home',
    });
  };

  // 컨펌 확인 처리
  const handleConfirm = () => {
    if (confirmModal.action === 'back') {
      // 이전 단계로 이동 (데이터 유지)
      goPrev?.();
    } else if (confirmModal.action === 'home') {
      // 메인으로 이동 (데이터 삭제)
      if (typeof window !== 'undefined') {
        sessionStorage.removeItem('applyFormData');
        sessionStorage.removeItem('currentStep');
      }
      router.push('/');
    }

    setConfirmModal({ isOpen: false, action: null });
  };

  // 컨펌 취소 처리
  const handleCancel = () => {
    setConfirmModal({ isOpen: false, action: null });
  };

  return (
    <div className={styles.navigation}>
      <button className={styles.navigationButton} type='button' onClick={handleBackClick}>
        <span className='sr-only'>뒤로가기</span>
      </button>

      <button className={`${styles.navigationLink} textMedium`} type='button' onClick={handleMainClick}>
        메인으로
      </button>

      {confirmModal.isOpen && (
        <ConfirmModal isOpen={confirmModal.isOpen} onCancel={handleCancel} onConfirm={handleConfirm}>
          작성을 취소하고 이동할까요?
        </ConfirmModal>
      )}
    </div>
  );
}
```

- Context를 통해 상태변경 함수 goPrev, goNext, resetStep 등을 전달받아 사용
- 상위 컴포넌트에서 setState를 prop으로 전달할 필요 없음
- sessionStorage 수동 제거로 사용자 의도에 따른 명시적 처리

### 4. FormHeader - 현재 단계 제목 표시

```tsx
import { useApplyContext } from '../models/apply-provider';
import { STEP_CONFIG, StepNumber } from '../models/constants';
import styles from './form-header.module.css';
import Navigation from './navigation';
import Progress from './progress';

export default function FormHeader() {
  const { currentStep } = useApplyContext();
  return (
    <div>
      <Navigation />
      <Progress />
      <h2 className={`${styles.formHeaderTitle} textBold`}>{STEP_CONFIG[currentStep as StepNumber].title}</h2>
    </div>
  );
}
```

- currentStep을 기반으로 동적 제목 표시

### 5. FormLayout - 폼 제출 및 다음 단계 (steps.tsx)

#### Context에서 여러 값 추출

```tsx
const {
  currentStep, // 현재 단계
  goNext, // 다음 단계 함수
  form, // RHF 폼 인스턴스
  getCurrentStepSchema, // 스키마 getter
} = useApplyContext();
```

- 한 컴포넌트에서 여러 관련 상태/함수를 하나의 Context에서 추출하여 사용

#### 단계별 조건부 처리

```tsx
const handleButtonClick = async () => {
  if (isSubmitStep) {
    // 4단계: 최종 제출
    await registerKid(transformedData);
  } else {
    // 1-3단계: 다음으로
    goNext?.();
  }
};
```

- currentStep에 따른 버튼 동작 분기
- goNext 호출시 자동으로 React 상태 업데이트 + sessionStorage 동기화
- 모든 구독 컴포넌트 리렌더링

#### 입력 필드 리렌더링

```tsx
const { currentStep, form } = useApplyContext();
const stepConfig = STEP_CONFIG[currentStep as StepNumber];

const { data: packages, loading: packagesLoading } = usePackages();
const { data: kindergartens, loading: kindergartensLoading } = useKindergartens();

const [bottomSheetOpen, setBottomSheetOpen] = useState(false);
const [bottomSheetType, setBottomSheetType] = useState<BottomSheetType>(null);

const [isError, setIsError] = useState(false);

// React Hook Form의 watch로 현재 값들 가져오기
const formValues = form.watch();
const selectedKindergartenId = formValues.kindergartenUUID || '';

// 선택된 기관의 반 목록 가져오기
const { data: rooms, loading: roomsLoading } = useRooms(selectedKindergartenId);
```

- Context에서 form 인스턴스를 받아서 입력 필드에 연결
- currentStep에 따라 표시할 필드 결정
- 폼 상태와 UI 상태를 Context로 통합 관리
