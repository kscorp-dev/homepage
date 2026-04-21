# Changelog

KS Corporation 홈페이지(https://www.ks-corporation.co.kr) 버전 이력입니다.
본 파일은 [Keep a Changelog](https://keepachangelog.com/ko/) 원칙을 따릅니다.

## [0.7.1] - 2026-04-21

### 변경
- Contact 섹션 Website 표기 `kscop.kr` → `https://www.ks-corporation.co.kr` (링크화)
- Contact 섹션 Location `대한민국` → `경기 부천시 소사구 부광로 228 511호`
- Footer의 `kscop.kr` 링크 → `www.ks-corporation.co.kr`

## [0.7.0] - 2026-04-21

### 추가
- 문의 폼 실제 발송 기능 구현 (AWS API Gateway + Lambda + SES)
- 엔드포인트: `https://emgs1jsqsh.execute-api.ap-northeast-2.amazonaws.com/contact`
- 봇 차단용 Honeypot(`_hp`) 필드
- 전송 중/성공/실패 상태 UI + 실패 시 fallback 이메일 안내
- 서버측 입력 검증(정규식, 길이 제한, Rate limit)

### 변경
- `handleSubmit()` mailto 방식 → `fetch()` POST 방식
- 폼 필드에 `maxlength` 속성 추가 (서버 검증과 일치)

### 인프라
- SES 도메인 identity `ks-corporation.co.kr` (DKIM verified, Route 53 연동)
- Lambda 함수 `ks-contact-form` (Python 3.12, arm64, 256MB)
- API Gateway HTTP API `emgs1jsqsh` + CORS 4개 Origin 허용

## [0.6.0] - 2026-04-21

### 추가
- `privacy.html` — 개인정보처리방침 페이지 (개인정보 보호법 기준 12개 조항)
- 개인정보 보호책임자 **고변승** 명시

### 변경
- Contact 섹션 이메일 `kscp@kscop.kr` → `contact@ks-corporation.co.kr` (mailto 링크)
- Footer 저작권 영역에 보호책임자 표기 + privacy.html 링크 연결

## [0.5.1] - 2026-04-08

### 수정
- 이메일 주소 및 연도 표기 수정

## [0.5.0] - 2026-04-08

### 추가
- KS Corporation 홈페이지 초기 버전 공개
- `index.html` — 히어로, 제품(SelOn, 공장 자동화), 로드맵, Contact 섹션
- `sellon.html` — 셀온 제품 상세 페이지
- Sky-blue 브랜드 컬러 팔레트 (`--sky1..4`, `--dark`, `--accent`)
- Pretendard 웹폰트 적용

---

### 인프라 요약 (2026-04-21 기준)

- **DNS**: AWS Route 53 (계정 026651348875)
- **정적 호스팅**: S3 `kscop-kr-website` (ap-northeast-2, 계정 724516859279)
- **CDN/HTTPS**: CloudFront `E1WGDQ5C00RORE` + ACM (와일드카드 `*.ks-corporation.co.kr`)
- **문의 폼 백엔드**: API Gateway → Lambda (Python 3.12) → SES (DKIM 서명)
- **수신함**: AWS WorkMail `contact@ks-corporation.co.kr`

### 링크

- Live: https://www.ks-corporation.co.kr
- 개인정보처리방침: https://www.ks-corporation.co.kr/privacy.html
- 저장소: https://github.com/kscorp-dev/homepage
