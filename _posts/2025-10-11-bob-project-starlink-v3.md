---
title: "신기한 BoB 프로젝트들: 스타링크 v3 취약점 분석"
date: 2025-10-11 23:54:04 +0900
categories: [Research, Report]
tags: []
---

> 이 글은 [티스토리](https://v0xe1.tistory.com/7)에서 옮겨온 글입니다.
{: .prompt-info }

## 1. 스타링크 v3?

**HSPACE Knights Frontier** **프로그램의 일환으로 BoB에서 진행한 프로젝트 주제들에 대해 내 생각을 공유하는 시간을 가져보려고 한다.** 흥미로운 주제를 찾아 보다가, 13기 항목에 이목을 끄는 주제가 있어 정보를 공유하고자 한다. 프로젝트 이름은 ‘스타링크 v3 취약점 분석’이다. 이름의 [스타링크 v3](https://www.starlink.com/kr/technology)는 스페이스X가 운용하는 스타링크 위성의 3세대 버전을 의미하는 것으로 추측된다.

스타링크의 보안 취약점 연구는 출시 당시에도 활발했었으며, 공식 버그바운티 시스템으로 수차례 제보된 기록이 존재했다. 하지만 대부분의 취약점 제보는 스타링크의 사용자 단말(지상 국지 장비, dish + 라우터 쪽)에 한하여 이뤄진 것이 다수이다. 필자가 조사한 바에 따르면, 최근 스타링크의 CVE로 언급된 취약점은 라우터 GEN2장비에서 발견되었었다. 이는 다음과 같다.

- [CVE-2023-49965](https://hackintoanetwork.com/blog/2023-starlink-router-gen2-xss-kor/)
- [CVE-2023-52235](https://hackintoanetwork.com/blog/2023-starlink-dns-rebinding-attack-kor/)

필자 또한 졸업 프로젝트의 일환으로 스타링크의 라우터 Gen2의 분석을 수행해 본 경험이 있었기에 해당 과정이 얼마나 번거롭고 복잡한지 알고 있다. 그러나 실제 인공위성을 타겟으로 하는 연구는 감히 상상하기 쉽지 않았을 것이다. 당장 어떻게 공격할지 생각만 해도 막막하기도 하다. 또한 프로젝트의 핵심 활동이 ‘취약점 분석’이였으므로, 어쩌면 기존의 1-day를 활용한 케이스일 수도 있다.

## 2. 예상해보기

해당 연구 과정을 자세히 알아보고 싶지만, 비교적 최근의 프로젝트 였기 때문에 자료를 쉽게 찾을 수 없었다. 따라서 필자가 생각한 프로젝트 진행 과정을 조사에 의거하여 추측해 보았다.

### 2.1. 공격 가능성 식별

대부분의 취약점 분석의 시작이라고 볼 수 있다. 스타링크 V3 위성이 사용하는 통신, 소프트웨어, 하드웨어는 어떻게 구성되어 있는지 조사하고, 이를 관리하는 단말은 무엇인지 알아야 할 것이다. 스타링크는 저궤도 위성통신을 이용한 인터넷 서비스로, 사용자의 단말(dish + 라우터)이 근처의 스타링크 위성과 통신하고, 그 위성은 게이트웨이/백홀을 통해 인터넷과 연결된다.

따라서 현실적으로 직접 분석 가능한 요소는 아래와 같을 것이다.

- [**스타링크 라우터, 안테나 장비**](https://www.starlink.com/specifications?srsltid=AfmBOooqsrqXldCP1m-cXXUuvJoZW1_HM55BSrGVjc9iQId0ffrp2cFI)
- **스타링크 서비스 어플리케이션**
- **스타링크 웹 서비스**

이외에 여건이 된다면 스페이스X의 제품군들 모두가 위성으로의 예상 공격로라고 생각할 수 있을 것이다. 이외에 스타링크 V3의 위협 모델링을 수행했을 가능성이 높다.

### 2.2. 시도할만한 수단들?

프로젝트에서 시도했을 것 같은 수단을 정리해 보았다.

- **단말 장비 해부(물리분해)**: PCB·JTAG·UART 포트 식별
  - 하드웨어의 펌웨어 추출 시도
- **정적/동적 펌웨어 분석**: Ghidra/IDA, binwalk, strings, firmware-mod-kit
  - 추출한 펌웨어를 QEMU등으로 에뮬레이팅하는 과정이 필요할 것으로 보임.
- **네트워크·프로토콜 분석**: Wireshark, tcpdump, scapy, mitmproxy(관리 UI/앱)
  - 네트워크 프로토콜 형식을 파악할 수 있다면, 이를 이용해 퍼징 테스트를 수행할 수 있음.
  - 웹 서비스가 Attack Surface라면 고전적인 BurpSuit를 활용한 패킷 분석도 효과적일 것으로 추측됨.
- **무선/물리분석**: SD R (GNU Radio, HackRF/USRP), 스펙트럼 분석기(합법적 테스트 대역에서), 신호 재생·스푸핑 실험(격리 환경)

## 3. 예상 산출물

1. 분석한 취약점에 대한 전반적인 보고서들(CVE, PoC, TestCase, report timeline 등)
2. 스타링크 V3의 위협 모델링
3. 최신 스타링크 단말기의 무선/물리 취약점 분석 보고서
4. 관련 1-day분석 보고서
5. 위성 대상 보안 분석 방법론 제시(패킷 캡처 방법 등…)

## 4. 시행착오는 없었을까?

### 4.1. 스타링크의 한국 서비스 진출 연기

당시 프로젝트가 진행된 2024년도는 한국에서 스타링크 서비스를 이용할 수 없는 상태였다.

따라서 아무리 단말 및 이용 서비스를 대상으로 보안 취약점을 발견했다 하더라도, 이를 증명하는 과정을 거치려면 서비스가 실제로 시행되고 있는 해외에서만 가능할 것이다.

### 4.2. 스타링크 v3 사용 가능 여부 불확실

필자가 조사한 바에 따르면, spaceX는 스타링크 v3위성을 2026년 상반기를 목표로 하고 있는 것으로 알고 있다. 그렇다면 어떻게 스타링크 v3를 분석할 수 있을까? 이 부분만큼은 정말 미지수이다...

## 5. 참고문서

- [https://bugcrowd.com/engagements/spacex](https://bugcrowd.com/engagements/spacex)
- [https://www.kitribob.kr/trainee_walk/projects](https://www.kitribob.kr/trainee_walk/projects)
- [https://www.pcmag.com/news/spacex-starlink-has-crossed-2-million-active-customers-in-us#:~:text=%60%60SpaceX%20is%20targeting%20to%20begin%20launching%20its,first%20half%20of%202026%2C''%20the%20company%20announced](https://www.pcmag.com/news/spacex-starlink-has-crossed-2-million-active-customers-in-us#:~:text=%60%60SpaceX%20is%20targeting%20to%20begin%20launching%20its,first%20half%20of%202026%2C''%20the%20company%20announced)
- [https://blog.quarkslab.com/starlink.html](https://blog.quarkslab.com/starlink.html)
