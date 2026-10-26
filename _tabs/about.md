---
# the default layout is 'page'
icon: fas fa-info-circle
order: 4
title: About
---

<!-- TODO 표시는 HTML 주석이라 사이트에는 보이지 않지만, 공개 저장소 소스에는 보입니다. 채운 뒤 주석을 지우세요. -->

## 배준서 (Junseo Bae) · v0xe1

**보안 취약점 연구자**

리버싱과 코드 오디팅으로 소프트웨어의 취약점을 찾고 분석하는 일을 좋아합니다.
어떤 타겟을 고르고 어떻게 파고들지 분석 방법론을 고민하는 과정을 특히 즐기며,
공부하고 분석한 내용을 이곳에 꾸준히 기록하고 있습니다.

- ✉️ [snakebjs0107@gmail.com](mailto:snakebjs0107@gmail.com)
- 🐙 [github.com/v0xe1](https://github.com/v0xe1)

## 🐞 취약점 연구

### 개인 연구

| 연도 | 대상 | 취약점 / 영향 |
|:---:|---|---|
| 2026 | **Windows** | 로컬 권한 상승 (EoP)<br>MSRC 제보, 검토 중 |
| 2026 | **pjproject**<br>SIP·미디어 C 라이브러리 | [**CVE-2026-40614**](https://github.com/pjsip/pjproject/security/advisories/GHSA-j59p-4xrr-fp8g) `High`<br>Opus 코덱 디코딩 경로의 버퍼 크기 검증 미흡 → 조작된 수신 오디오 패킷으로 **힙 버퍼 오버플로우** (CWE-122) <!-- TODO: 발견 방법 --> |
| 2025 | **Nextcloud Desktop**<br>동기화 클라이언트 | [**CVE-2025-47792**](https://github.com/nextcloud/security-advisories/security/advisories/GHSA-qm2f-959g-7p65) `Moderate`<br>소켓 API 접근 통제 미흡 → 같은 PC의 다른 프로그램이 사용자 데이터의 **공유 링크를 임의로 생성** (CWE-284)<br>HackerOne 버그바운티 · 직접 타겟을 선정하고 리버싱으로 발견 |

### 팀 프로젝트

**CVE 원정대** · 파인더갭 HuntingMaster 1기 (2024.07 ~ 2024.08, 3인)

KISA 타겟 분석이 성과를 내지 못하자 퍼징으로 방향을 바꾼 프로젝트입니다.
WinAFL persistent mode와 커버리지 분석으로 반디뷰가 직접 구현한 이미지 파서를 노렸고,
오픈소스 대상으로는 ASan 퍼징과 크래시 자동 분류 환경을 구축했습니다.
팀원 모두가 하네스 설계·개선과 크래시 분석을 함께 진행했습니다. (제보 12건, 승인 4건)

- **반디뷰** (반디소프트, Windows 이미지 뷰어) — [v7.06 릴리스 노트](https://kr.bandisoft.com/bandiview/history/)에서 수정
  - PSD 파일 검증 미흡 → **버퍼 오버플로우**, PSD·JXR 파일 검증 미흡 → DoS (3건) · 반디소프트에 직접 제보, 릴리스 노트 크레딧
  - EXR 포맷 취약점 · KISA 제보 · **KVE-2024-0416**
- **pdfio** (PDF C 라이브러리) — [**CVE-2024-42358**](https://github.com/michaelrsweet/pdfio/security/advisories/GHSA-4hh9-j68x-8353) `Moderate` · TTF 파서 `read_cmap`의 nGroups 검증 미흡 → 무한 루프·메모리 고갈과 **힙 버퍼 오버플로우** (추가 버그 2건 패치)

**ApplePIE** · KITRI Best of the Best 11기 (2022 ~ 2023, 6인)

- **Apple WebKit** (Safari 16.3) — [**CVE-2023-23517**, **CVE-2023-23518**](https://support.apple.com/ko-kr/102800) · 메모리 처리 결함 → 조작된 웹 콘텐츠 처리 시 **임의 코드 실행**
- 팀원이 코드 오디팅으로 발견했고, 본인은 퍼징 파트를 맡았습니다.

## 🛠 기술

- **리버싱** — IDA Pro 기반 바이너리 분석, LLM(MCP) 연동 분석 <!-- TODO: 디버거 등 -->
- **코드 오디팅** — 소스코드 기반 취약점 분석, CodeQL 쿼리 작성
- **분석 방법론** — 타겟 선정, 공격 표면 분석, 정적·동적 분석 전략 설계
- **동적 분석** — 디버깅, 크래시 원인 분석 <!-- TODO: 사용하는 디버거 -->
- **퍼징** — WinAFL·ASan 기반 하네스 작성, 커버리지 분석, 크래시 분류
- **분석 대상** — Windows, 브라우저(WebKit), C/C++ 오픈소스 라이브러리
- **언어** — C/C++, Python <!-- TODO: 확인 -->

## 🎓 교육 · 활동

| 기간 | 내용 |
|---|---|
| 2025 | **HSPACE Knights Frontier** — CodeQL, Web3(Ethernaut), BoB 연구 프로젝트 분석 등 미션 수행 |
| 2024.07 ~ 2024.08 | **파인더갭 HuntingMaster 1기** — 팀 프로젝트 "CVE 원정대" (3인) |
| 2022.06 ~ 2023.03 | **KITRI Best of the Best 11기** 수료 — 보안컨설팅 트랙 · 팀 프로젝트 ApplePIE |
| <!-- TODO: 입학 --> ~ <!-- TODO: 졸업 예정일 --> | **충남대학교 컴퓨터융합학부** · 보안 동아리 ARGOS |
| 2024.10 ~ 2026.07 | 병역: 사회복무요원 소집해제 |
