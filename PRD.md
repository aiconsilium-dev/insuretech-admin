# PRD — 보험 하자접수 AI (어드민 대시보드)

## 1. Product Overview

| 항목 | 내용 |
|------|------|
| **서비스명** | 보험 하자접수 AI — 어드민 대시보드 (InsureTech Admin) |
| **한 줄 설명** | 보험사/손해사정사가 전체 청구건을 TYPE별로 관리하는 데스크탑 시스템. AI 적산, 품셈 관리, 보험 연계, 구상권까지 통합 관리. |
| **대상 사용자** | 보험사 담당자, 손해사정사 |

## 2. Tech Stack

| 구분 | 기술 |
|------|------|
| 프레임워크 | 단일 HTML (Vanilla JS + CSS) |
| 빌드 도구 | 없음 (순수 HTML/JS/CSS) |
| 스타일링 | 인라인 CSS / `<style>` 태그 |
| 배포 | GitHub Pages (main 브랜치) |

## 3. Architecture

```
단일 HTML 파일
├── 좌측 사이드바 네비게이션
│   ├── TYPE A 관리
│   ├── TYPE B 관리
│   ├── TYPE C 관리
│   ├── 보험 관리
│   ├── 품셈/단가 관리
│   ├── 현장조사 관리
│   └── 구상권 관리
├── 메인 콘텐츠 영역
└── JavaScript (DOM 조작 기반)
```

## 4. Pages & Routes

단일 HTML 내 사이드바 메뉴 전환:

| 메뉴 | 기능 |
|------|------|
| TYPE A 관리 | 시공사 하자 — 하자보증기간 확인, 감정평가, 시공사 협상/청구 |
| TYPE B 관리 | 면책 — 면책 사유 기록, 의견서 발송, 이의신청 대응, 금감원 대응 |
| TYPE C 관리 | AI 적산 + 보험금 산출 — 공종별 산정, 소유자/임차인 분리, 보험 연계 |
| 보험 관리 | 단지별 가입 보험 현황 (5개 단지, CGL/재물/주택화재) |
| 품셈/단가 관리 | 2026년 표준품셈, M-bar 등 최신 공법 단가 |
| 현장조사 관리 | 현장조사 배정/결과 통합 관리 |
| 구상권 관리 | 도급업체 구상권 추적 + 보험 연계 |

## 5. Data Models

```typescript
// TYPE A: 시공사 하자
interface TypeAClaim {
  claimId: string;
  apartment: string;
  defectType: string;            // 하자 유형
  warrantyPeriod: {              // 하자보증기간
    start: string;
    end: string;
    isValid: boolean;
  };
  appraisal?: {                  // 감정평가
    appraiser: string;
    amount: number;
    date: string;
  };
  constructorNegotiation: {      // 시공사 협상
    constructor: string;
    status: 'pending' | 'negotiating' | 'agreed' | 'legal';
    agreedAmount?: number;
  };
}

// TYPE B: 면책
interface TypeBClaim {
  claimId: string;
  exemptionReason: string;       // 면책 사유
  opinionLetter?: {              // 의견서
    sentAt: string;
    content: string;
  };
  objection?: {                  // 이의신청
    filedAt: string;
    status: 'pending' | 'reviewing' | 'resolved';
  };
  fssResponse?: {                // 금감원 대응
    caseNo: string;
    status: string;
  };
}

// TYPE C: 보험금 산출
interface TypeCClaim {
  claimId: string;
  estimation: {                  // AI 적산
    trades: TradeEstimation[];   // 공종별 산정
    totalAmount: number;
  };
  ownerAmount: number;           // 소유자 보상액
  tenantAmount: number;          // 임차인 보상액
  insuranceLink: {               // 보험 연계
    policyType: string;
    policyNo: string;
    claimStatus: string;
  };
}

// 공종별 적산
interface TradeEstimation {
  tradeName: string;             // 공종명
  quantity: number;              // 수량
  unitPrice: number;             // 단가 (품셈 기준)
  amount: number;                // 금액
  method: string;                // 공법 (M-bar 등)
}

// 보험 정보
interface InsurancePolicy {
  apartment: string;             // 단지명
  policies: {
    type: 'CGL' | '재물' | '주택화재'; // 보험 유형
    insurer: string;             // 보험사
    policyNo: string;
    coverage: number;            // 보장한도
    deductible: number;          // 자기부담금
    period: { start: string; end: string };
  }[];
}

// 품셈 데이터
interface UnitPrice {
  code: string;                  // 품셈 코드
  name: string;                  // 공종명
  unit: string;                  // 단위
  price: number;                 // 단가
  year: number;                  // 기준연도 (2026)
  method?: string;               // 공법 (M-bar 등)
}
```

## 6. Key Features

### 6.1 TYPE A: 시공사 하자 관리
- 하자보증기간 자동 확인
- 감정평가 등록 및 관리
- 시공사 협상/청구 프로세스 트래킹

### 6.2 TYPE B: 면책 관리
- 면책 사유 기록 및 분류
- 의견서 발송 관리
- 이의신청 대응
- 금감원 민원 대응

### 6.3 TYPE C: AI 적산 + 보험금 산출
- **공종별 산정**: 품셈 기반 자동 적산
- **소유자/임차인 분리**: 보상 대상별 금액 분리
- **보험 연계**: 해당 보험 자동 매칭
- **재물보험/개인보험 우선 청구 전략** 지원
- **자기부담금 처리 기준** 적용

### 6.4 단지별 가입 보험 관리
- 5개 단지 보험 현황
- 보험 유형: CGL(종합배상책임) / 재물보험 / 주택화재보험
- 보장한도, 자기부담금, 보험기간 관리

### 6.5 품셈/단가 관리
- 2026년 표준품셈 데이터
- M-bar 등 최신 공법 단가 반영
- 공종별 단가 조회/수정

### 6.6 현장조사/구상권/보험연계 통합 관리
- 현장조사 결과 → TYPE 분류 → 보험 연계까지 일관된 워크플로우
- 도급업체 구상권 관리
- 보험사 청구 상태 트래킹

## 7. Design System

| 속성 | 값 |
|------|-----|
| 레이아웃 | **데스크탑 최적화** — 좌측 사이드바 (240px) + 메인 콘텐츠 |
| 배경색 | `#ffffff` |
| 사이드바 | 어두운 배경, 메뉴 하이라이트 |
| 테이블 | 상세 데이터 표시, 정렬/필터 지원 |
| TYPE 구분 | A: 주황/노랑, B: 회색, C: 파랑/초록 |
| 금액 | 우측 정렬, 천단위 콤마 |
| 폰트 | 시스템 폰트 |

## 8. External Integrations

| 서비스 | 연동 방식 |
|--------|-----------|
| 입주민 앱 (insuretech-resident) | 같은 생태계 — 접수 데이터 참조 (목업) |
| 관리사무소 앱 (insuretech-office) | 같은 생태계 — 현장조사 데이터 참조 (목업) |
| 표준품셈 DB | 하드코딩 (2026년 기준) |
| 보험사 API | 미연동 (목업) |

## 9. Deployment

| 항목 | 값 |
|------|-----|
| 호스팅 | GitHub Pages |
| 브랜치 | `main` |
| URL | `https://{owner}.github.io/insuretech-admin/` |

## 10. Known Limitations

- **모든 데이터는 목업** — 실제 보험사/품셈 DB 연동 없음
- **AI 적산은 규칙 기반** — 실제 AI/ML 모델 미연동
- **입주민 앱/관리사무소 앱과 데이터 연동 없음** — 각 앱 독립 실행
- **품셈 데이터는 하드코딩** — 실시간 업데이트 없음
- **보험 연계는 UI 시연용** — 실제 보험사 API 미연동
- **금감원 대응은 목업** — 실제 민원 시스템 연동 없음
- **자기부담금/우선 청구 전략은 정적 규칙** — 동적 계산 없음
