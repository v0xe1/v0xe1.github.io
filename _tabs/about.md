---
# the default layout is 'page'
icon: fas fa-info-circle
order: 4
title: About
---

<!-- TODO 표시는 HTML 주석이라 사이트에는 보이지 않지만, 공개 저장소 소스에는 보입니다. 채운 뒤 주석을 지우세요. -->

## 배준서 (Junseo Bae) · v0xe1

**퍼징으로 0-day를 찾는 취약점 연구자**

퍼징과 리버싱으로 파서·코덱의 메모리 손상 취약점과 Windows 클라이언트의 로직 취약점을 찾아 제보해 왔습니다.
WebKit, 오픈소스 C 라이브러리, 국산 Windows 소프트웨어에서 직접 찾은 0-day로 CVE 5건을 받았고,
현재 Windows 권한 상승 취약점 1건을 제보해 검토를 기다리고 있습니다.
앞으로는 **브라우저와 Windows 시스템**의 취약점 연구와 익스플로잇 개발에 집중하려고 합니다.

- ✉️ [snakebjs0107@gmail.com](mailto:snakebjs0107@gmail.com)
- 🐙 [github.com/v0xe1](https://github.com/v0xe1)

| CVE | 국산 SW 제보 (반디뷰) | Windows EoP | 단독 발견 최고 등급 |
|:---:|:---:|:---:|:---:|
| **5** | **3** | **1** (검토 중) | **High** |

## 🐞 취약점 제보

최신순입니다. Write-up은 패치가 공개된 건부터 차례로 올리고 있습니다.

| 연도 | 대상 | 취약점 / 영향 | 참여 |
|:---:|---|---|---|
| 2026 | **Windows** | 로컬 권한 상승 (EoP) · MSRC 제보 완료, 검토 중 · 세부 내용은 패치 후 공개 | <!-- TODO: 단독/팀 --> |
| 2026 | **pjproject**<br>SIP·미디어 C 라이브러리 | [**CVE-2026-40614**](https://github.com/pjsip/pjproject/security/advisories/GHSA-j59p-4xrr-fp8g) `High`<br>Opus 코덱 디코딩 경로의 버퍼 크기 검증 미흡 → 조작된 수신 오디오 패킷으로 **힙 버퍼 오버플로우** (CWE-122) <!-- TODO: 발견 방법(퍼저·하네스) --> | **단독** (Finder: V0xe1) |
| 2025 | **Nextcloud Desktop**<br>동기화 클라이언트 | [**CVE-2025-47792**](https://github.com/nextcloud/security-advisories/security/advisories/GHSA-qm2f-959g-7p65) `Moderate`<br>소켓 API 접근 통제 미흡 → 같은 PC의 다른 프로그램이 사용자 데이터의 **공유 링크를 임의로 생성** (CWE-284) · HackerOne 제보 | <!-- TODO: 단독/팀 --> |
| 2024 | **pdfio**<br>PDF C 라이브러리 | [**CVE-2024-42358**](https://github.com/michaelrsweet/pdfio/security/advisories/GHSA-4hh9-j68x-8353) `Moderate`<br>TTF 파서 `read_cmap`의 nGroups 검증 미흡 → 무한 루프·메모리 고갈(DoS)과 **힙 버퍼 오버플로우** · 퍼징 + ASan | <!-- TODO: 팀명 / 맡은 역할 --> |
| 2024 | **반디뷰** (반디소프트)<br>Windows 이미지 뷰어 | ① PSD 파일 검증 미흡 → **버퍼 오버플로우**<br>② PSD 파일 검증 미흡 → DoS<br>③ JXR 파일 검증 미흡 → DoS <!-- TODO: KISA 신고 / KVE 번호, 공개 가능한 분석 링크 --> | <!-- TODO: 단독/팀 --> |
| 2023 | **Apple WebKit**<br>Safari 16.3 | [**CVE-2023-23517**, **CVE-2023-23518**](https://support.apple.com/ko-kr/102800)<br>메모리 처리 결함 → 조작된 웹 콘텐츠 처리 시 **임의 코드 실행** <!-- TODO: 맡은 역할 --> | 팀 **ApplePIE** (6인) |

## 🛠 기술

- **퍼징** — 파서·코덱 대상 하네스 작성, ASan 기반 크래시 분석 <!-- TODO: 사용한 퍼저 이름 -->
- **정적 분석** — CodeQL 쿼리 작성
- **리버싱** — IDA Pro, LLM(MCP) 연동 바이너리 분석 <!-- TODO: 디버거 등 추가 -->
- **분석 대상** — 브라우저(WebKit), Windows 클라이언트·시스템, C/C++ 오픈소스 라이브러리
- **언어** — C/C++, Python <!-- TODO: 확인 -->

## 🎓 교육 · 활동

| 기간 | 내용 |
|---|---|
| 2025 | **Hackyboiz** 팀 활동 — 취약점 연구 블로그·뉴스레터 팀 <!-- TODO: 정확한 활동 기간, 작성 글 링크 --> |
| 2025 | **HSPACE Knights Frontier** — CodeQL, Web3(Ethernaut), BoB 연구 프로젝트 분석 등 미션 수행 |
| 2022.06 ~ 2023.03 | **KITRI Best of the Best 11기** 수료 — 보안컨설팅 트랙 <!-- TODO: 프로젝트명 · 퍼징 팀에서 맡은 역할 --> |
| <!-- TODO: 입학 --> ~ <!-- TODO: 졸업 예정일 --> | **충남대학교 컴퓨터융합학부** · 보안 동아리 ARGOS |
| 2024.10 ~ 2026.07 | 병역: 사회복무요원 소집해제 |
