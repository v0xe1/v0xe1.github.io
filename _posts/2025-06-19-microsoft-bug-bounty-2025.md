---
title: "2025 Microsoft 공식 버그바운티 정리"
date: 2025-06-19 20:37:35 +0900
categories: [News]
tags: [bugbounty, microsoft, windows]
---

> 이 글은 [티스토리](https://v0xe1.tistory.com/4)에서 옮겨온 글입니다.
{: .prompt-info }

Windows의 보안 취약점을 발견하고자 하는 분들이라면 한번쯤은 그 보상에 대해 생각해볼만 합니다. Microsoft는 COM을 포함한 Windows 플랫폼 전반에 대한 보안 취약점 제보를 공식적으로 받고 있으며, 이를 통해 보상(버그바운티)을 제공하는 체계를 운영하고 있습니다. 이 글에는 Microsoft 버그바운티를 알기 쉽게 요점정리한 내용이 담겨 있습니다.

---

## 🧭 공식 버그바운티 프로그램

Microsoft는 다양한 제품군에 대해 연속형 버그바운티 프로그램을 운영하고 있습니다 :

- **Microsoft Applications & On‑Premises Servers**: COM, ActiveX, IIS, Exchange 등 서버/애플리케이션 대상. 최대 **30,000 USD** 지급. ([microsoft.com](https://www.microsoft.com/en-us/msrc/bounty-applications))
- **Windows Insider Preview**: Windows 보안 이슈. 최대 **100,000 USD**. ([microsoft.com](https://www.microsoft.com/en-us/msrc/bounty))
- **Hyper‑V / Kernel‑level**: 고심각도 취약점 시 **250,000 USD**까지. ([microsoft.com](https://www.microsoft.com/en-us/msrc/bounty))
- **M365 / Azure / Identity / Copilot** 등 다수 플랫폼 별 상이한 상금 규모 존재. ([microsoft.com](https://www.microsoft.com/en-us/msrc/bounty))

---

## 🛠 제보 절차 및 대상

1. **MSRC Researcher Portal** 또는 이메일(secure@microsoft.com) 통해 제출
   - 제품명(예: Windows COM, “Microsoft Applications and On-Premises Servers”) 명시
   - CVE 형태의 정식 협조형 책임 공개(Coordinated Vulnerability Disclosure) 조건 ([microsoft.com](https://www.microsoft.com/en-us/msrc/bounty-terms), [microsoft.com](https://www.microsoft.com/en-us/msrc/bounty-applications))
2. 제출 내용:
   - 취약점 개요 및 영향도
   - 재현 가능한 PoC 코드
   - 대상 시스템/환경 정보
3. 제출 후 MSRC가 검토, 인정 시 보상 지급. 보상금은 MSRC 권한에 따라 결정 ([en.wikipedia.org](https://en.wikipedia.org/wiki/Zero_Day_Initiative), [wired.com](https://www.wired.com/story/windows-defender-vulnerability-twelve-years), [microsoft.com](https://www.microsoft.com/en-us/msrc/bounty-terms))

---

## ⚖️ 보상 및 법적 보호(샌크터리)

- 최대 **100k~250k USD**까지 보상이 이루어질 수 있음
- **Safe Harbor** 정책에 따라, 연구자가 MSRC 가이드에 맞춰 보고할 경우 **민형사 책임 면책** 보장 ([microsoft.com](https://www.microsoft.com/en-us/msrc/bounty-applications))

---

## ✅ 요약표

| 항목 | 정보 |
| --- | --- |
| 제보 위치 | MSRC 포털 또는 secure@microsoft.com |
| 대상 제품 | COM 포함 Applications, Windows, Hyper-V 등 |
| 최대 보상 | 최대 250,000 USD |
| 법적 보호 | MSRC Safe Harbor에 의해 보장 |
| 제출 시 요구 사항 | 재현 PoC + CVE 방식 책임 공개 |

더욱 자세한 취약점 제보 절차는 [공식문서](https://www.microsoft.com/en-us/msrc/bounty-applications)에서 더 확인하실 수 있을 것입니다. 저는 군 복무 중이기 때문에 당장은 보상을 받을 수 없지만, 전역 후 버그바운티 진행을 위해 미리 계획해 보고자 정리해 보았습니다.
