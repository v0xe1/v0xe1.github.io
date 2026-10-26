---
title: "[DailyNews]AI가 내 OneDrive를 마음대로 열람한다고?"
date: 2025-05-29 14:56:46 +0900
categories: [News]
tags: [dailynews, microsoft, oauth, onedrive, vulnerability]
---

> 이 글은 [티스토리](https://v0xe1.tistory.com/3)에서 옮겨온 글입니다.
{: .prompt-info }

## 개요

![](/assets/img/posts/onedrive-file-picker-flaw/1.png)

MS의 Windows가 강력하게 지원하는 OneDrive는 세계적으로 높은 신뢰를 받는 클라우드 스토리지로, 국내에서도 많은 Windows사용자들이 백업 등 다양한 목적으로 애용하고 있습니다. 하지만 이 OneDrive가 chatGPT 등의 AI에게까지 열람 권한을 과도하게 부여하고 있다면, 믿기시나요?

## OneDrive File Picker의 결함으로 웹 앱들에게 사용자 드라이브에 대한 **전체 액세스 권한을 부여합니다.**

![](/assets/img/posts/onedrive-file-picker-flaw/2.png)
_출처:  https://thehackernews.com/2025/05/microsoft-onedrive-file-picker-flaw.html_

2025년 5월 28일, Oasis Security의 연구팀은 웹사이트가 OneDrive File Picker를 통해 업로드하기 위해 선택한 특정 파일뿐만 아니라 사용자의 전체 OneDrive 콘텐츠에 액세스 할 수 있도록 하는 Microsoft의 OneDrive File Picker의 결함을 발견했다고 밝혔습니다.

연구원들은 ChatGPT, Slack, Trello 및 ClickUp을 포함한 수백 개의 앱이 영향을 받고 있다고 추정하며, 이는 수백만 명의 사용자가 이미 이러한 앱에 OneDrive에 대한 액세스 권한을 부여했을 수 있음을 의미합니다. 이 결함은 고객 데이터 유출 및 규정 준수 규정 위반을 포함하여 심각한 결과를 초래할 수 있다고 알리고 있습니다.

## 원인

#### OAuth. Background

[OAuth](https://www.oasis.security/resources/blog/microsoft-oauth-start-here)는 웹사이트의 사용자의 데이터를 다른 사이트에 호스팅 된 웹 애플리케이션과 공유하기 위한 산업 표준 프로토콜입니다. 때문에 다양한 플랫폼의 수많은 서비스에서 지원되고 있습니다.

OneDrive는 사용자가 자신의 OneDrive에서 웹 애플리케이션(예: 파일 업로드)에 파일을 공유하고자 할 때, **OAuth를 사용해 사용자가 애플리케이션에 필요한 권한을 부여할 수 있도록 허용**합니다. 그리고 사용자가 동의하면 웹 애플리케이션에 접근 토큰이 발급되어 사용자의 데이터에 대한 제한된 접근이 허용됩니다.

#### OneDrive File Picker**의 과도한 권한**

공식 OneDrive File Picker 구현은 OneDrive에 대한 세분화된 OAuth 범위가 없기 때문에 단일 파일만 업로드하는 경우에도 전체 드라이브에 대한 읽기 액세스를 요청합니다.

![](/assets/img/posts/onedrive-file-picker-flaw/3.png)
_출처: OASIS onedrive-file-access-warning.pdf_

**업로드를 완료하기 전에 사용자에게 동의를 제공하라는 메시지가 표시되지만, 프롬프트의 모호하고 불분명한 언어는 부여되는 액세스 수준을 전달하지 못하므로 사용자는 예기치 않은 보안 위험에 노출**될 수 있습니다.

세분화된 범위가 없기 때문에 사용자는 다른 보안 옵션이 없다는 이유로 모든 파일을 대상으로 하는 악성 앱과 과도한 권한을 요청하는 합법적인 앱을 구별할 수 없습니다.

[연구팀의 공개 보고서의 내용](https://pages.oasis.security/rs/106-PZV-596/images/onedrive-file-access-warning.pdf?version=0)에 따르면 모든 버전의 OneDrive File Picker는 다음을 허용하는 권한을 요청합니다.

- 파일 업로드 시, **OneDrive 전체를 read 가능.**
- 파일 다운로드 시, **OneDrive 전체에 write 가능.**

게다가, 특정 요청에 대한 File Picker의 7.0 버전은 업로드 과정에 대해 읽기 및 쓰기 권한을 모두 요구하며, 이러한 권한들은 최소 1시간 동안 유효합니다.

이 내용들을 요약하자면, **OneDrive File Picker를 사용하는 웹 애플리케이션은 여러분이 업로드/다운로드하기 위해 선택한 파일뿐만 아니라 전체 OneDrive에 접근할 수 있습니다.** 더 나쁜 것은, 파일 업로드가 완료된 후에도 이 접근 권한이 계속 남아 있을 수 있다는 것입니다.

#### 민감한 요청 정보를 안전하지 못하게 저장한다

최신 버전의 OneDrive File Picker(8.0)에서는 개발자가 일반적으로 MSAL(Microsoft Authentication Library)을 사용하고, 대부분 [Authorization Flow](https://www.oasis.security/resources/blog/microsoft-oauth-start-here#authorization)를 사용하여 인증을 직접 처리해야 합니다.

하지만 File Picker는 이 과정에서 다음의 보안 위험을 가지고 있음이 확인되었습니다.

- MSAL은 브라우저의 세션 스토리지에 **중요한 토큰(OAuth 토큰)을 일반 텍스트로 저장**
- 권한 부여 흐름 사용 [새로 고침 토큰(Refresh Token)](https://www.oasis.security/resources/blog/microsoft-oauth-start-here#access-duration) 액세스 기간을 연장하여 **사용자 데이터에 대한 지속적인 액세스를 제공하는 발급도 할 수 있음.**

![](/assets/img/posts/onedrive-file-picker-flaw/4.png)
_출처: OASIS onedrive-file-access-warning.pdf_

그리고 이 8.0 버전은, OpenAI도 사용하고 있음을 보여줍니다.

## **Mitigation Steps**

Oasis security의 제보에 따라 Microsoft는 아직 수정 사항이 없지만 이 문제를 인정했습니다. 때문에 이런 위험에 민감한 독자분들은 그동안 안전한 대안이 마련될 때까지 OAuth를 통해 OneDrive를 사용하여 파일을 업로드하는 옵션을 일시적으로 제거하는 것도 고려해봐야 할 수도 있습니다.

Oasis는 이 위험에 대해 다음의 Mitigation Steps를 소개하고 있습니다.

- **이전에 Vendor에 대한 액세스 권한을 부여했는지 여부를 확인**
  - 개인: MS 계정에서 앱 액세스 세부 정보로 부여된 범위 및 권한 확인.
  - 기업 및 조직: [https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/view-applications-portal#view-a-list-of-applications](https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/view-applications-portal#view-a-list-of-applications)
- **사이트에서 OneDrive File Picker를 사용하는지 확인**
  - 사이트의 인터페이스를 통해 OneDrive에 파일을 업로드하거나 다운로드.
  - 동의 프롬프트(consent prompt)가 나타나면 OneDrive 권한이 요청되고 있는지 확인.
- **Web Apps에 대한 완화 방법**
  - **새로 고침 토큰 사용 방지**
    - "오프라인 액세스" 범위를 요청하지 마세요.
    - 새로 고침 토큰 유지 및 사용과 관련된 코드 또는 스토리지 논리를 제거합니다.
    - 새로 고침 토큰이 현재 저장되어 있는 경우 안전한 방식으로 폐기합니다.
  - **액세스 토큰을 안전한 방식으로 저장하고** 더 이상 필요하지 않을 때 폐기하십시오.
    - 액세스 토큰을 처리하는 코드를 검토하여 제삼자에게 노출되지 않는지(예: 세션 또는 로컬 저장소에 저장됨) 확인합니다.

이 보안위험에 대해서 조사 중, 사이버 보안 뉴스 플랫폼 [“The Hacker News”](https://thehackernews.com/)에서는 추가 의견을 위해 Microsoft에 연락을 취했으며, 회신이 있으면 관련 기사를 업데이트하겠다고 알렸습니다. 여러분들도 이 위험에 대해 더 관심 있으신 분들은 아래 레퍼런스 항목을 참고하시면 좋을 것 같습니다.

## Reference

- [https://thehackernews.com/2025/05/microsoft-onedrive-file-picker-flaw.html](https://thehackernews.com/2025/05/microsoft-onedrive-file-picker-flaw.html)
- [https://www.oasis.security/resources/blog/onedrive-file-picker-security-flaw-oasis-research](https://www.oasis.security/resources/blog/onedrive-file-picker-security-flaw-oasis-research)
- [https://pages.oasis.security/rs/106-PZV-596/images/onedrive-file-access-warning.pdf?version=0](https://pages.oasis.security/rs/106-PZV-596/images/onedrive-file-access-warning.pdf?version=0)
- [https://www.oasis.security/resources/blog/microsoft-oauth-start-here](https://www.oasis.security/resources/blog/microsoft-oauth-start-here)

> 이 글에서 궁금하신 내용이나, 의견이 있으시다면 마음껏 댓글로 남겨주세요.  
> DailyNews의 궁극적인 목표는 기사에 관심 있는 독자와 소통하고, 다른 시선의 의견들을 꾸준히 수집하는 것입니다.
