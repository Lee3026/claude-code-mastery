# Claude 프로젝트 가이드라인

## 1. 언어 및 커뮤니케이션 규칙

- **기본 응답 언어**: 한국어
- **코드 주석**: 한국어로 작성
- **커밋 메시지**: 한국어로 작성
- **문서화**: 한국어로 작성 (important)
- **변수명/함수명**: 영어 (코드 표준 준수)

## 2. 코딩 스타일

- **들여쓰기**: 2칸
- **선호 프레임워크**: React, Next.js

## 3. 코딩 컨벤션

### 네이밍 규칙
- **컴포넌트**: PascalCase (예: `UserProfile`)
- **함수/변수**: camelCase (예: `getUserData`, `isLoading`)
- **상수**: UPPER_SNAKE_CASE (예: `API_BASE_URL`)
- **타입/인터페이스**: PascalCase (예: `User`, `ButtonProps`)
- **Boolean**: `is`, `has`, `should`, `can` 접두사 (예: `isLoading`)

### 파일/폴더 명명
- 컴포넌트 파일: PascalCase (`Button.tsx`)
- 유틸 파일: camelCase (`formatDate.ts`)
- 폴더: kebab-case (`user-profile`)

### Import 순서
1. React 관련
2. 외부 라이브러리
3. 내부 모듈 (절대경로 → 상대경로)
4. 타입
5. 스타일

### 코드 작성 원칙
- 화살표 함수 사용
- 함수형 컴포넌트 사용
- Early return 패턴 권장
- 단일 책임 원칙 준수

## 4. 추가 선호사항

- **Tailwind CSS** 사용
- **TypeScript** 사용
