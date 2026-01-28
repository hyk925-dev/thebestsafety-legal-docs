# TBSafe 시스템 설계 문서

> 더베스트안전 통합 전산망 v1.9.4

## 1. 시스템 개요

### 1.1 목적
건설현장 안전관리 컨설팅 업무를 위한 통합 전산 시스템. 고객사/현장 관리, 월간 안전평가, 법정서류 판정, 알림 발송 등 핵심 업무를 디지털화.

### 1.2 주요 사용자
| 역할 | 권한 | 설명 |
|------|------|------|
| admin | 전체 | 시스템 관리자, 모든 기능 접근 |
| user | 업무 | 더베스트안전 직원, 고객사/현장 관리 |
| client | 제한 | 고객사 담당자, 자사 데이터만 조회 |

---

## 2. 기술 스택

### 2.1 프론트엔드
- **Framework**: Next.js 14 (App Router)
- **Language**: TypeScript
- **Styling**: Tailwind CSS
- **State**: React useState/useEffect
- **Deployment**: Vercel

### 2.2 백엔드
- **Framework**: Express.js
- **Language**: TypeScript
- **Database**: PostgreSQL (Railway)
- **Authentication**: JWT (jsonwebtoken)
- **File Upload**: Multer
- **Excel**: ExcelJS
- **Email**: Nodemailer
- **SMS**: Twilio
- **Scheduler**: node-cron
- **Deployment**: Railway

### 2.3 인프라
```
[Vercel] ─── HTTPS ───> [Railway API] ───> [Railway PostgreSQL]
   │                          │
   │                          ├── Nodemailer (SMTP)
   │                          └── Twilio (SMS)
   │
   └── Static Assets (Next.js)
```

---

## 3. 데이터베이스 스키마

### 3.1 핵심 테이블

```
┌─────────────┐      ┌─────────────┐      ┌──────────────────┐
│   users     │      │   clients   │      │      sites       │
├─────────────┤      ├─────────────┤      ├──────────────────┤
│ id (PK)     │      │ id (PK)     │◄────►│ id (PK)          │
│ email       │      │ name        │      │ client_id (FK)   │
│ password    │      │ business    │      │ name             │
│ name        │      │ represent.  │      │ address          │
│ role        │      │ address     │      │ start_date       │
│ client_id   │──────│ phone       │      │ end_date         │
│ is_active   │      │ email       │      │ contract_amount  │
│ last_login  │      │ contract_*  │      │ safety_manager   │
│ created_at  │      │ status      │      │ status           │
└─────────────┘      └─────────────┘      └──────────────────┘
                                                   │
                     ┌─────────────────────────────┼─────────────────────────────┐
                     │                             │                             │
                     ▼                             ▼                             ▼
          ┌──────────────────┐          ┌─────────────────┐          ┌─────────────────┐
          │ monthly_assess.  │          │     alerts      │          │    uploads      │
          ├──────────────────┤          ├─────────────────┤          ├─────────────────┤
          │ id (PK)          │          │ id (PK)         │          │ id (PK)         │
          │ site_id (FK)     │          │ site_id (FK)    │          │ site_id (FK)    │
          │ year_month       │          │ alert_type      │          │ uploaded_by     │
          │ area1~5_rate     │          │ deadline_day    │          │ year_month      │
          │ area1~5_grade    │          │ timing[]        │          │ file_type       │
          │ tbm_data         │          │ contact_*       │          │ file_name       │
          │ notes            │          │ is_active       │          │ file_path       │
          └──────────────────┘          └─────────────────┘          │ status          │
                                                 │                   └─────────────────┘
                                                 ▼
                                        ┌─────────────────┐
                                        │ alert_history   │
                                        ├─────────────────┤
                                        │ id (PK)         │
                                        │ alert_id (FK)   │
                                        │ sent_at         │
                                        │ sent_type       │
                                        │ recipient       │
                                        │ status          │
                                        └─────────────────┘
```

### 3.2 전체 테이블 목록

| 테이블 | 설명 | 주요 컬럼 |
|--------|------|-----------|
| users | 사용자 | email, password_hash, role, client_id |
| clients | 고객사 | name, business_type, contract_start/end |
| sites | 현장 | client_id, name, address, contract_amount |
| monthly_assessments | 월간평가 | site_id, year_month, area1~5_rate |
| check_items | 체크항목 | assessment_id, area, item_no, is_done |
| quotes | 견적서 | client_id, site_id, total, status |
| quote_items | 견적항목 | quote_id, description, amount |
| alerts | 알림설정 | site_id, deadline_day, timing, contact_* |
| alert_history | 알림이력 | alert_id, sent_type, recipient, status |
| alert_templates | 알림템플릿 | name, template_type, subject, body |
| uploads | 업로드 | site_id, file_type, file_path, status |
| legal_document_checks | 법정서류판정 | site_id, required_documents, triggers |
| sh_evaluations | SH평가 | client_id, score_category1~4, grade |
| sh_company_info | SH본사정보 | client_id, organization_chart |
| sh_site_info | SH현장정보 | site_id, hazardous_materials |
| sh_documents | SH서류 | client_id, doc_type, content |
| sh_checklist | SH체크리스트 | evaluation_id, item_name, is_checked |

---

## 4. API 엔드포인트

### 4.1 인증 (Auth)
| Method | Endpoint | 설명 | 권한 |
|--------|----------|------|------|
| POST | /api/auth/login | 로그인 | Public |
| POST | /api/auth/register | 회원가입 | Admin |
| POST | /api/auth/init-admin | 초기 관리자 생성 | Public (1회) |
| GET | /api/auth/me | 내 정보 조회 | Auth |
| PUT | /api/auth/password | 비밀번호 변경 | Auth |

### 4.2 고객사 (Clients)
| Method | Endpoint | 설명 | 권한 |
|--------|----------|------|------|
| GET | /api/clients | 목록 조회 | Auth |
| GET | /api/clients/:id | 상세 조회 | Auth |
| POST | /api/clients | 생성 | Admin/User |
| PUT | /api/clients/:id | 수정 | Admin/User |
| DELETE | /api/clients/:id | 삭제 | Admin |

### 4.3 현장 (Sites)
| Method | Endpoint | 설명 | 권한 |
|--------|----------|------|------|
| GET | /api/sites | 목록 조회 | Auth |
| GET | /api/sites/:id | 상세 조회 | Auth |
| POST | /api/sites | 생성 | Admin/User |
| PUT | /api/sites/:id | 수정 | Admin/User |
| DELETE | /api/sites/:id | 삭제 | Admin |

### 4.4 월간평가 (Assessments)
| Method | Endpoint | 설명 | 권한 |
|--------|----------|------|------|
| GET | /api/assessments | 목록 조회 | Auth |
| GET | /api/assessments/:id | 상세 조회 | Auth |
| POST | /api/assessments | 생성 | Admin/User |
| PUT | /api/assessments/:id | 수정 | Admin/User |

### 4.5 알림 (Alerts)
| Method | Endpoint | 설명 | 권한 |
|--------|----------|------|------|
| GET | /api/alerts | 목록 조회 | Auth |
| POST | /api/alerts | 생성 | Auth |
| PUT | /api/alerts/:id | 수정 | Auth |
| GET | /api/alerts/:id/history | 이력 조회 | Auth |
| POST | /api/alerts/:id/send | 개별 발송 | Auth |
| POST | /api/alerts/send-due | 일괄 발송 | Admin |

### 4.6 알림 템플릿 (Alert Templates)
| Method | Endpoint | 설명 | 권한 |
|--------|----------|------|------|
| GET | /api/alert-templates | 목록 조회 | Auth |
| POST | /api/alert-templates | 생성 | Admin |
| PUT | /api/alert-templates/:id | 수정 | Admin |
| DELETE | /api/alert-templates/:id | 삭제 | Admin |
| POST | /api/alert-templates/:id/set-default | 기본 설정 | Admin |
| POST | /api/alert-templates/preview | 미리보기 | Auth |

### 4.7 데이터 내보내기 (Export)
| Method | Endpoint | 설명 | 권한 |
|--------|----------|------|------|
| GET | /api/export/assessments/excel | 평가 Excel | Auth |
| GET | /api/export/sites/excel | 현장 Excel | Auth |
| GET | /api/export/clients/excel | 고객사 Excel | Auth |
| GET | /api/export/site-report/:siteId/excel | 현장보고서 | Auth |

### 4.8 기타
| Method | Endpoint | 설명 | 권한 |
|--------|----------|------|------|
| GET | /api/dashboard/stats | 대시보드 통계 | Auth |
| GET | /api/activity | 활동 로그 | Admin |
| GET | /api/users | 사용자 목록 | Admin |
| PUT | /api/users/:id/status | 상태 변경 | Admin |
| POST | /api/users/:id/reset-password | 비밀번호 초기화 | Admin |
| DELETE | /api/users/:id | 사용자 삭제 | Admin |

---

## 5. 주요 기능

### 5.1 5대 영역 월간평가
```
영역1: 안전보건 조직 구성
영역2: 근로자 참여
영역3: 위험성 평가
영역4: 안전보건 교육
영역5: 비상 대응 체계
```

각 영역별 시행률(0~1)을 입력하면 자동으로 등급 산정:
- 우수: 80% 이상
- 양호: 60% 이상
- 보통: 40% 이상
- 미흡: 40% 미만

### 5.2 법정서류 판정
공사 조건(금액, 층수, 높이, 면적 등)에 따라 필수 법정서류 자동 판정:
- 안전관리계획서
- 유해위험방지계획서
- 공사안전보건대장
- 건설안전점검표
- 등등

### 5.3 알림 시스템
- **타이밍**: D-3, D-1, 당일, 초과
- **채널**: 이메일 (SMTP), SMS (Twilio)
- **템플릿**: 변수 치환 지원 ({{site_name}}, {{urgency}} 등)
- **스케줄러**: 매일 오전 9시 KST 자동 발송

### 5.4 SH평가 (안전보건 경영시스템)
- 본사/현장 정보 관리
- 4대 영역 + 가점 평가
- 등급 산정 (S, A, B, C, D)
- 체크리스트 및 서류 관리

---

## 6. 화면 구성

### 6.1 페이지 목록
| 경로 | 페이지 | 설명 |
|------|--------|------|
| /login | 로그인 | 인증 |
| /dashboard | 대시보드 | 메인, 통계, 빠른메뉴 |
| /clients | 고객사 | 목록, 등록, 수정 |
| /sites | 현장 | 목록, 등록, 수정 |
| /sites/[id] | 현장 상세 | 분석, 추세, 알림 |
| /assessments | 월간평가 | 목록, 입력 |
| /legal-check | 법정서류 | 판정 입력, 결과 |
| /quotes | 견적 | 목록, 작성 |
| /alerts | 알림 | 설정, 템플릿, 발송 |
| /uploads | 업로드 | 파일 관리 |
| /reports | 보고서 | 월간 보고서 |
| /sh-evaluation | SH평가 | 평가 입력, 조회 |
| /users | 사용자 | 관리 (관리자) |
| /activity | 활동로그 | 내역 (관리자) |
| /settings | 설정 | 비밀번호, 내보내기 |

---

## 7. 배포 정보

### 7.1 환경변수

**API (Railway)**
```env
DATABASE_URL=postgresql://...
JWT_SECRET=your-jwt-secret
MIGRATE_SECRET=tbsafe-init-2026

# 이메일 (선택)
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email
SMTP_PASS=your-app-password
SMTP_FROM=TBSafe <noreply@tbsafe.com>

# SMS (선택)
TWILIO_ACCOUNT_SID=your-sid
TWILIO_AUTH_TOKEN=your-token
TWILIO_FROM_NUMBER=+1234567890
```

**Web (Vercel)**
```env
NEXT_PUBLIC_API_URL=https://api-production-0ee5.up.railway.app
```

### 7.2 URL
- API: https://api-production-0ee5.up.railway.app
- Web: (Vercel 배포 주소)
- Health Check: GET /health

### 7.3 초기 설정
```bash
# 1. 마이그레이션 실행
curl -X POST https://api-production-0ee5.up.railway.app/migrate \
  -H "Content-Type: application/json" \
  -d '{"secret":"tbsafe-init-2026"}'

# 2. 관리자 계정 생성
curl -X POST https://api-production-0ee5.up.railway.app/api/auth/init-admin \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@example.com","password":"your-password","name":"관리자"}'
```

---

## 8. 버전 히스토리

| 버전 | 날짜 | 주요 변경 |
|------|------|----------|
| 1.9.4 | 2026-01 | 사용자 관리 강화 (비밀번호 초기화/삭제) |
| 1.9.3 | 2026-01 | 활동 로그 API |
| 1.9.2 | 2026-01 | 대시보드 향상 (추세/알림 위젯) |
| 1.9.1 | 2026-01 | 알림 템플릿 관리 |
| 1.9.0 | 2026-01 | Excel 내보내기, 현장 상세 분석 |
| 1.8.0 | 2026-01 | API 모듈화 리팩토링 |
| 1.7.0 | 2026-01 | SH평가, 법정서류 판정 |

---

## 9. 연락처

- **개발**: Claude Code (AI Assistant)
- **의뢰**: 더베스트안전
- **Repository**: https://github.com/hyk925-dev/tbsafe
