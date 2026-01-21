# CLAUDE.md

이 파일은 Claude Code (claude.ai/code)가 이 저장소에서 작업할 때 필요한 가이드를 제공합니다.

## 프로젝트 개요

빌드 프로세스, 백엔드 서버, 외부 의존성 없이(Tailwind CSS CDN 제외) 브라우저에서 완전히 실행되는 **순수 JavaScript 버킷 리스트 웹 애플리케이션**입니다. LocalStorage를 사용하여 데이터를 영구 저장합니다.

## 애플리케이션 실행 방법

빌드 프로세스가 없는 정적 웹 애플리케이션이므로 여러 방법으로 실행할 수 있습니다:

1. **직접 파일 열기**: 파일 탐색기에서 `bucket-list-main/index.html`을 더블클릭
2. **Python 서버**: `cd bucket-list-main && python -m http.server 8000` (그런 다음 http://localhost:8000 접속)
3. **VS Code Live Server**: `index.html` 우클릭 → "Open with Live Server"

**중요**: 프로젝트 파일이 `bucket-list-main/bucket-list-main/`에 중첩되어 있으므로 항상 내부 디렉토리에서 작업하세요.

## 아키텍처

### 모듈 구조

애플리케이션은 관심사가 명확히 분리된 **2계층 아키텍처**를 따릅니다:

**데이터 레이어** (`js/storage.js`):
- `BucketStorage` 객체가 모든 LocalStorage 작업 관리
- LocalStorage에 `'bucketList'` 키로 데이터 저장
- CRUD 작업을 처리하고 앱 레이어에 데이터 반환
- 모든 스토리지 메서드는 동기식이며 즉시 반환됨

**프레젠테이션 레이어** (`js/app.js`):
- `BucketListApp` 클래스가 UI 상태 및 DOM 조작 관리
- `cacheElements()`에서 DOM 요소를 캐싱하여 반복적인 쿼리 방지
- 모든 사용자 상호작용이 `render()`를 트리거하여 전체 리스트 재구성
- 동적으로 생성된 요소에 인라인 `onclick` 핸들러 사용

### 데이터 흐름

```
사용자 액션 → 이벤트 핸들러 (app.js) → BucketStorage 메서드 (storage.js)
→ LocalStorage 업데이트 → render() → DOM 업데이트
```

모든 상태 변경은 버킷 리스트 컨테이너의 전체 재렌더링을 트리거합니다.

### 데이터 구조

각 버킷 리스트 항목은 다음과 같이 저장됩니다:
```javascript
{
  id: "timestamp-string",        // Date.now().toString()
  title: "목표 설명",
  completed: boolean,
  createdAt: "ISO-datetime",
  completedAt: "ISO-datetime or null"
}
```

배열은 LocalStorage에 `'bucketList'` 키로 JSON 형식으로 저장됩니다.

### 주요 구현 세부사항

**렌더링 전략**:
- 모든 변경 시 전체 리스트 재렌더링 (작은 리스트에서는 허용 가능)
- 템플릿 리터럴을 사용하여 HTML 문자열 생성
- `createBucketItemHTML()`이 Tailwind 클래스로 각 항목의 HTML 생성

**이벤트 처리**:
- 폼 제출과 필터 버튼은 `bindEvents()`에서 addEventListener 사용
- 동적 리스트 항목은 전역 `app` 인스턴스를 참조하는 인라인 onclick 핸들러 사용
- 모달 닫기는 취소 버튼, 모달 배경 클릭, 폼 제출 시 트리거됨

**보안**:
- `escapeHtml()` 메서드 (app.js:223)가 `textContent` 할당을 사용하여 XSS 방지
- 사용자 생성 콘텐츠를 HTML에 삽입하기 전에 항상 호출됨

**상태 관리**:
- 현재 필터는 `this.currentFilter`에 저장 ('all', 'active', 'completed')
- 현재 편집 중인 항목 ID는 `this.editingId`에 저장
- 다른 클라이언트 측 상태는 없음 - 모든 것이 LocalStorage에서 가져옴

## 파일 위치

```
bucket-list-main/bucket-list-main/
├── index.html           # Tailwind CDN이 포함된 메인 HTML 구조
├── css/styles.css       # 커스텀 애니메이션, 반응형 스타일, 다크 모드
└── js/
    ├── storage.js       # LocalStorage CRUD 작업
    └── app.js           # UI 로직 및 이벤트 핸들러
```

## 스타일링 시스템

- **주요 프레임워크**: Tailwind CSS (index.html:8에서 CDN으로 로드)
- **커스텀 스타일**: `css/styles.css`에서 애니메이션, 필터 버튼 상태, 반응형 조정 추가
- **빌드 프로세스 없음**: Tailwind 클래스를 컴파일 없이 HTML에서 직접 사용

Tailwind 클래스 적용 위치:
- 정적 요소는 `index.html`에 직접 작성
- 동적 요소는 `createBucketItemHTML()`의 템플릿 리터럴을 통해 작성

## 일반적인 수정 작업

**새로운 통계 추가**:
1. index.html의 통계 섹션에 HTML 요소 추가 (20-39줄)
2. `cacheElements()`에서 요소 캐싱 (app.js:21)
3. `BucketStorage.getStats()`에서 계산 로직 업데이트 (storage.js:111)
4. `updateStats()`에서 표시 로직 업데이트 (app.js:163)

**새로운 필터 추가**:
1. index.html에 `data-filter` 속성을 가진 필터 버튼 추가 (61-82줄)
2. `BucketStorage.getFilteredList()` switch 문에 case 추가 (storage.js:134)

**항목 속성 수정**:
1. `BucketStorage.addItem()`에서 항목 생성 로직 업데이트 (storage.js:41)
2. `createBucketItemHTML()`에서 HTML 생성 로직 업데이트 (app.js:175)
3. 기존 LocalStorage 데이터에 대한 마이그레이션 전략 고려

## 한국어 사용

모든 UI 텍스트, 주석, 문서가 한국어로 작성되어 있습니다. 기능을 수정하거나 추가할 때 다음 항목에 대해 한국어를 유지하세요:
- HTML의 사용자 대면 텍스트
- Alert/confirm 다이얼로그 메시지
- 코드 주석
- 콘솔 에러 메시지
