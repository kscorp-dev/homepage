# KS Corporation Homepage

> 감동을 담은 소프트웨어를 만드는 스타트업, KS Corporation의 공식 웹사이트.

[![Live](https://img.shields.io/badge/live-ks--corporation.co.kr-0ea5e9?style=flat-square)](https://www.ks-corporation.co.kr)
[![Release](https://img.shields.io/github/v/release/kscorp-dev/homepage?style=flat-square&color=0284c7)](https://github.com/kscorp-dev/homepage/releases)
[![License](https://img.shields.io/badge/license-proprietary-475569?style=flat-square)](#라이선스)

---

## 🌐 Live

- **메인**: https://www.ks-corporation.co.kr
- **개인정보처리방침**: https://www.ks-corporation.co.kr/privacy.html
- **셀온(SelOn) 상세**: https://www.ks-corporation.co.kr/sellon.html

## 🧱 기술 스택

| 영역 | 구성 |
|---|---|
| **프론트엔드** | Vanilla HTML + CSS + JS (프레임워크 없음), Pretendard 웹폰트 |
| **정적 호스팅** | AWS S3 (`kscop-kr-website`, `ap-northeast-2`) |
| **CDN / HTTPS** | AWS CloudFront (`E1WGDQ5C00RORE`), ACM 와일드카드 인증서 |
| **DNS** | AWS Route 53 (호스티드 존 `Z02474923EI6E5MU7UFC6`) |
| **문의 폼 백엔드** | API Gateway HTTP API → Lambda (Python 3.12, arm64) → SES |
| **메일 수신** | AWS WorkMail `contact@ks-corporation.co.kr` |

## 🗺️ 아키텍처

```
브라우저
  │
  │  HTTPS  ─────────────────────────►  CloudFront (E1WGDQ5C00RORE)
  │                                       │
  │                                       ▼
  │                                     S3 kscop-kr-website (정적 파일)
  │
  │  POST /contact (JSON)  ────────►  API Gateway (emgs1jsqsh)
                                        │
                                        ▼
                                      Lambda ks-contact-form
                                        │ sesv2.send_email
                                        ▼
                                      SES (ks-corporation.co.kr, DKIM)
                                        │
                                        ▼
                                      WorkMail contact@ks-corporation.co.kr
```

## 📁 디렉터리

```
homepage/
├── index.html          # 메인 랜딩 (Hero / 제품 / 로드맵 / Contact)
├── sellon.html         # 셀온(SelOn) 제품 상세 페이지
├── privacy.html        # 개인정보처리방침 (12조, 보호책임자 고변승)
├── KS_LOGO.png         # 로고 (바이너리)
├── KS_LOGO_b64.txt     # 로고 Base64 (inline 데이터 URI용)
├── serve.py            # 로컬 개발 서버 (Python 단순 정적)
├── CHANGELOG.md        # 버전 이력
└── README.md           # 본 문서
```

## 🧑‍💻 로컬 개발

```bash
git clone https://github.com/kscorp-dev/homepage.git
cd homepage
python3 serve.py                 # http://localhost:8000
```

브라우저에서 `http://localhost:8000/index.html` 열면 라이브와 동일하게 동작합니다.

> 문의 폼만 예외입니다. Lambda 엔드포인트 CORS가 운영 도메인만 허용하기 때문에, 로컬에서는 폼 제출이 CORS로 차단됩니다. UI 확인은 로컬에서, 실제 발송 테스트는 라이브에서 진행하세요.

## 🚀 배포

### 빠른 배포 (단일 파일 수정 후)

```bash
# 1. 파일 수정, 커밋, 태그
git add <file>
git commit -m "v0.7.2 — <변경 요약>"

# 2. S3 업로드
aws s3 cp index.html s3://kscop-kr-website/index.html \
  --content-type 'text/html; charset=utf-8' --profile kscorp-new

# 3. CloudFront 캐시 무효화
aws cloudfront create-invalidation \
  --distribution-id E1WGDQ5C00RORE \
  --paths '/index.html' '/' \
  --profile kscorp-new
```

### 전체 동기화

```bash
aws s3 sync . s3://kscop-kr-website/ \
  --exclude '.*' --exclude 'README.md' --exclude 'CHANGELOG.md' \
  --exclude 'serve.py' --exclude 'KS_LOGO_b64.txt' \
  --profile kscorp-new

aws cloudfront create-invalidation \
  --distribution-id E1WGDQ5C00RORE --paths '/*' \
  --profile kscorp-new
```

## 🏷️ 버전 관리 정책

**Semantic Versioning 유사**한 흐름을 따릅니다 (`MAJOR.MINOR.PATCH`).

| 변경 종류 | 버전 증가 | 예시 |
|---|---|---|
| 버그 수정, 문구 정정, 이미지 교체 | PATCH (`0.7.1` → `0.7.2`) | 오타 수정, 링크 교체 |
| 섹션 추가, UI 개선, 새 페이지 | MINOR (`0.7.2` → `0.8.0`) | 신규 제품 섹션, 새 페이지 |
| 대규모 리디자인, 구조 변경 | MAJOR (`0.x` → `1.0`) | 프레임워크 도입, 전면 재설계 |

### 버전 업데이트 체크리스트 (매 릴리즈)

- [ ] `CHANGELOG.md` 상단에 새 버전 섹션 추가 (추가/변경/수정/제거 기록)
- [ ] `README.md` 본 파일의 해당 섹션 최신화 (필요 시)
- [ ] 커밋 메시지 `vX.Y.Z — <요약>` 형식
- [ ] `git tag -a vX.Y.Z -m "..."` 로 annotated 태그 생성
- [ ] `git push origin main && git push origin vX.Y.Z`
- [ ] `gh release create vX.Y.Z --notes "..."` 로 GitHub Release 발행
- [ ] 필요 시 `gh repo edit --description "... vX.Y.Z ..."` 업데이트
- [ ] S3 업로드 + CloudFront 무효화
- [ ] 라이브 검증 (`curl -I https://www.ks-corporation.co.kr/`)

## 📋 버전 이력

전체 이력은 **[CHANGELOG.md](CHANGELOG.md)** 참조.

최근 3개 버전:

- **v0.7.1** (2026-04-21) — Contact 정보 최신화 (Website, Location)
- **v0.7.0** (2026-04-21) — 문의 폼 실제 발송 구현 (API Gateway + Lambda + SES)
- **v0.6.0** (2026-04-21) — 개인정보처리방침 페이지 + 컨택 메일 변경

## 📬 문의

- **Email**: [contact@ks-corporation.co.kr](mailto:contact@ks-corporation.co.kr)
- **Website 문의 폼**: https://www.ks-corporation.co.kr/#contact
- **주소**: 경기 부천시 소사구 부광로 228 511호
- **개인정보 보호책임자**: 고변승 / contact@ks-corporation.co.kr

## 🔒 라이선스

© 2026 KS Corporation. All rights reserved.  
본 저장소의 모든 콘텐츠(코드, 디자인, 로고, 텍스트)는 KS Corporation의 소유이며 명시적 허가 없이 복제, 배포, 상업적 사용을 금합니다.
