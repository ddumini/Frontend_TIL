# 성능 최적화

## 1. 번들링 & 빌드 최적화

### Turbopack 사용

- Next.js 15의 공식 번들러인 Turbopack은 고성능 빌드 환경을 제공하며 개발 서버 실행, 핫 리로딩, 코드 번들링 속도가 최적화되어 있다.
- Webpack 대비 메모리 사용량이 적으며 최대 700배 빠른 번들링 속도를 제공한다.

### Incremental Compilation

TypeScript의 증분 컴파일을 활성화하여 이전 빌드 정보를 재사용, 빌드 시간을 단축한다.

### Path Alias

절대 경로 별칭으로 import 경로를 단순화하고 트리 쉐이킹을 개선한다.

## 2. 이미지 & 리소스 최적화

- Next.js Image 컴포넌트 사용
  - 자동 이미지 최적화 (WebP, AVIF 변환)
  - priority prop으로 LCP 개선
  - fill로 반응형 이미지 처리
  - Lazy Loading 기본 적용
- CDN, Font preconnect 사용
  - CDN은 빠른 로딩 속도, 번들 사이즈 감소, 캐싱 효율성에서 유리하다.
  - preconnect 설정을 통해 외부 폰트 리소스에 대한 사전 연결로 TTFB(Time to First Byte)를 단축하고 Hydration 오류를 방지한다.

## 3. 컴포넌트 & 렌더링 최적화

- Server Component 기본 사용
  - 상태관리나 브라우저 API가 필요한 경우에만 Client Component로 지정하여 하이드레이션 비용을 최소화합니다. (번들 크기 감소)

## 4. 데이터 페칭 최적화

- 조건부 데이터 페칭

  ```tsx
  export function useRooms(kindergartenUUID?: string) {
  const [data, setData] = useState<BaseEntity[]>([])
  const [loading, setLoading] = useState(false)
  const [error, setError] = useState<string | null>(null)

  useEffect(() => {
    let isCancelled = false

    if (!kindergartenUUID) {
      setData([])
      return
    }

    // ...
  ```

  - 필요한 경우에만 API 호출 (kindergartenUUID가 있을 때만)
  - 불필요한 네트워크 요청 방지

- Memory Leak 방지

  ```tsx
  let isCancelled = false;

  const fetchData = async () => {
    setLoading(true);
    setError(null);

    try {
      const response = await fetch('/api/kindergartens');

      if (!response.ok) {
        throw new Error('데이터를 불러오는데 실패했습니다.');
      }

      const result = await response.json();

      if (!isCancelled) {
        setData(result.kindergartens || []);
      }
    } catch (err) {
      if (!isCancelled) {
        setError(err instanceof Error ? err.message : '알 수 없는 오류가 발생했습니다.');
      }
    } finally {
      if (!isCancelled) {
        setLoading(false);
      }
    }
  };

  fetchData();

  return () => {
    isCancelled = true;
  };
  ```

  - cleanup 함수로 언마운트 후 상태 업데이트 방지
  - Race condition 방지 (동시성 이슈 방지)

## 5. 폼 & 상태 관리 최적화

- React Hook Form + Zod
  - onBlur 모드로 불필요한 재렌더링 최소화
  - Zod 스키마로 타입 안정성과 유효성 검사 동시에 처리
  - 비제어 컴포넌트 패턴으로 성능 향상
- SessionStorage 영속화
  - 새로고침 시 데이터 유지로 UX 개선
  - API 재요청 최소화
  - form.watch()로 효율적인 상태 구독
    - 변경된 필드만 리렌더링 (비제어 컴포넌트)
    - 의존성 관리 간단, 코드 간결성
- 단계별 검증
  - 전체 폼이 아닌 현재 단계만 검증하여 검증 비용 최소화

## 6. UI/UX 최적화

- Skeleton UI
  - 데이터 로딩 중 Skeleton UI로 인지된 성능 향상
  - Layout Shift 방지
- 커서 위치 복원

  ```tsx
  // 커서 위치 복원
  requestAnimationFrame(() => {
    let newPos = 0;
    let numCount = 0;

    for (let i = 0; i < formatted.length; i++) {
      if (/\d/.test(formatted[i])) {
        numCount++;
        if (numCount <= numbersBeforeCursor) newPos = i + 1;
        else break;
      } else if (numCount < numbersBeforeCursor) {
        newPos = i + 1;
      }
    }

    input.setSelectionRange(newPos, newPos);
  });
  ```

  - 포멧팅 시 커서 위치 유지로 UX 개선
  - requestAnimationFrame로 부드러운 처리

## 7. 키보드 접근성 & Body Scroll Lock

```tsx
export function useModalKeyboard({
  isOpen,
  onClose,
  enableEscapeKey = true,
  preventBodyScroll = true,
}: UseModalKeyboardProps) {
  useEffect(() => {
    const handleKeyDown = (e: KeyboardEvent) => {
      if (enableEscapeKey && e.key === 'Escape' && isOpen) {
        e.preventDefault();
        onClose();
      }
    };

    if (isOpen) {
      document.addEventListener('keydown', handleKeyDown);

      if (preventBodyScroll) {
        document.body.style.overflow = 'hidden';
      }
    }

    return () => {
      document.removeEventListener('keydown', handleKeyDown);

      if (preventBodyScroll) {
        document.body.style.overflow = '';
      }
    };
  }, [isOpen, onClose, enableEscapeKey, preventBodyScroll]);
}
```

- ESC 키로 모달 닫기
- 모달 열릴 때 배경 스크롤 방지

## 9. CSS 최적화

- CSS Modules
  - 스코프된 CSS로 글로벌 네임스페이스 오염 방지
  - 사용하지 않는 CSS 자동 제거
  - 빌드 타임에서 최적화된 클래스명 생성
- Responsive Font Sizing
  - 작은 화면에서 vw 단위로 폰트 크기를 조정하여 가독성 유지
- CSS Custom Properties
  - 재사용 가능한 디자인 토큰
  - 추후 테마 변경 시 효율적인 스타일 업데이트

## 10. 접근성 & SEO 최적화

- Sementic HTML
  - 스크린 리더 전용 텍스트로 접근성 향상
  - 시멘틱 태그 및 aria-label 사용
- Metadata 최적화
  - Next.js의 Metadata API로 SEO 최적화

## 정리

1. Turbopack 사용과 증분 컴파일, Path Alias 사용으로 **번들링 & 빌드 최적화**
2. Next.js Image 컴포넌트 사용과 CDN, Font preconnect 사용으로 **이미지 & 리소스 최적화**
3. Server Component 기본 사용과 조건부 데이터 페칭, Memory Leak 방지로 **데이터 페칭 최적화**
4. React Hook Form + Zod, SessionStorage 영속화, 단계별 검증으로 **폼 & 상태 관리 최적화**
5. Skeleton UI, 커서 위치 복원으로 **UI/UX 최적화**
6. useModalKeyboard 사용과 CSS Modules, Responsive Font Sizing, CSS Custom Properties 사용으로 **CSS 최적화**
7. Semantic HTML, Metadata 최적화로 **접근성 & SEO 최적화**
