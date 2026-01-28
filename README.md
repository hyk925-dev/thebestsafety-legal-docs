# 더베스트안전 작업물 저장소

> www.thebestsafety.co.kr 웹사이트 및 관련 도구 보관
> 최종 업데이트: 2026-01-28

---

## 폴더 구조

```
thebestsafety/
├── 법정서류판정/           # 법정서류 판정 시스템
│   ├── v8.0/              # 최신 버전 (2026-01-28)
│   │   └── construction-legal-docs-v8.0.html   # v7.0 기반 업그레이드
│   ├── v7.0/              # 이전 버전
│   │   ├── construction-legal-docs-v7.0-final.html
│   │   └── 업데이트노트.md
│   └── archive/           # 보관용
│
├── 전산망/                 # TBSafe 전산망 문서
│   ├── docs/
│   │   ├── system-docs.html   # 통합 문서 뷰어 (탭 네비게이션)
│   │   ├── DESIGN.md          # 설계 문서 (마크다운)
│   │   └── design.html        # 설계 문서 (HTML)
│   └── entities/              # 설계 엔티티 시스템
│       ├── praxis.md          # 실무 총괄
│       ├── safetyguard.md     # 안전법령 전문가
│       ├── architect.md       # 시스템 설계
│       ├── flowmaster.md      # UX/운영
│       └── meeting-template.md # 회의 템플릿
│
├── 참고/                   # 참고 자료
│   ├── SH평가 인증 모듈 자료 요청서.md
│   ├── 건설안전팀 모듈 자료 요청서.md
│   └── ...
│
├── index.html             # 메인 페이지
└── README.md
```

---

## 관련 프로젝트

| 폴더 | 용도 |
|------|------|
| `C:\AI\thebestsafety` | 웹사이트 작업물 (여기) |
| `C:\AI\tbsafe` | TBSafe 전산망 개발 |
| `C:\AI\praxis` | Praxis AI 부팅 파일 |

---

## 법정서류 판정 시스템

### 현재 버전: v8.0 (2026-01-28)

**법령 기준**: 2025.10.1 시행령
**기반**: Safety Law OS v2.2

**v8.0 변경사항** (v7.0 기반):
- CI 컬러 업데이트 (#162759, #929294, #bebebe)
- 가설구조물 항목 설명 보강:
  - 외부작업용 일체형: SWC, RCS, ACS, Workflat, Form 추가
  - 복합형(현장제작): 합벽지지대, 라이닝폼, 작업대차 등 추가
- CTP+HRP 통합 안내 제거
- 기존 v7.0 기능 유지 (4단계 위자드, 역할별 분기, 인쇄 최적화)

**배포**: www.thebestsafety.co.kr

---

## 전산망 (TBSafe)

### 현재 버전: v1.9.4 (2026-01-28)

**주요 문서**:
- `system-docs.html`: 통합 문서 뷰어 (운영 매뉴얼, 화면 상세, 기술 문서)
- `entities/`: 설계 엔티티 시스템 (Praxis, SafetyGuard, Architect, FlowMaster)

**개발 서버**:
- API: https://api-production-0ee5.up.railway.app
- Web: Vercel

---

## CI 컬러

| 컬러 | HEX | 용도 |
|------|-----|------|
| 네이비 | #162759 | 메인 배경 |
| 그레이 | #929294 | 테두리, 보조 텍스트 |
| 라이트그레이 | #bebebe | 비활성 요소 |

---

## 연락처

- **더베스트안전 안전보건팀**
- **대표번호**: 1551-7187
