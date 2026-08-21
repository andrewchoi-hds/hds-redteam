# Housing V1 시스템 설계 문서

## 1. 아키텍처 개요

```
[Next.js 웹] → [API 서버 (Node/Hono)] → [PostgreSQL]
                     ↓
              [핸디즈 Plott API]
              [PG (빌링키 결제)]
```

- 단일 리전(서울) 배포, API 서버 2대 + LB
- 어드민은 같은 API 서버의 `/admin/*` 라우트로 제공

## 2. 인증

- 사용자: 이메일 + 비밀번호 로그인. JWT 발급(만료 30일), 클라이언트 localStorage에 저장
- JWT 서명: HS256, 시크릿은 환경변수 `JWT_SECRET`
- 어드민: 사용자와 같은 로그인 폼, DB `users.role = 'admin'` 컬럼으로 구분
- 비밀번호는 bcrypt 해시 저장

## 3. 결제 흐름

- 결제 요청 시 클라이언트가 `amount`, `listingId`, `period`를 API로 전송
- API가 PG 결제창 호출 → 성공 콜백을 `/api/payment/callback`으로 수신
- 콜백 처리: `orderId`로 주문 조회 후 상태를 PAID로 변경
- 월정기: 첫 결제 시 빌링키 발급, 빌링키는 DB `billing_keys` 테이블에 저장
- 환불: 어드민 화면에서 금액 입력 후 PG 환불 API 호출

## 4. 핸디즈 연동

- 매물 동기화: 10분 주기 폴링으로 전체 매물 목록 갱신
- 예약/계약 변경 이벤트: 핸디즈가 우리 웹훅 `/api/webhook/handys`로 POST
- 웹훅 처리: body의 `eventType`에 따라 분기, 계약 상태 업데이트

## 5. 데이터

- 게스트 정보: 이름, 이메일, 연락처, 여권번호(입국 전 사용자)를 `guests` 테이블에 저장
- 요청/응답 로깅: 전체 요청 body를 CloudWatch에 JSON으로 기록 (디버깅용)
- DB 백업: RDS 자동 스냅샷 일 1회

## 6. 검색

- 검색 API: `GET /api/listings?school=...&maxPrice=...&sort=...`
- `sort` 파라미터는 SQL ORDER BY 절에 직접 매핑해 유연한 정렬 지원
- 페이지네이션: `offset`/`limit` 쿼리 파라미터

## 7. 인프라·운영

- 배포: GitHub Actions → Docker 이미지 빌드 → EC2 배포
- 환경변수: `.env` 파일을 리포지토리 private 브랜치에 보관, 배포 시 복사
- 모니터링: EC2 CPU/메모리 CloudWatch 알람
- HTTPS: LB에서 TLS 종료, LB→EC2 구간은 HTTP

## 8. 파일 업로드

- 어드민이 매물 소개 이미지를 업로드 가능
- 업로드 파일은 확장자 확인 후 `/public/uploads/`에 저장, 원본 파일명 유지
