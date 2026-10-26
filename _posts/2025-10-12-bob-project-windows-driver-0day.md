---
title: "신기한 BoB 프로젝트들: Windows Driver 0-day Research"
date: 2025-10-12 21:00:04 +0900
categories: [Research, Report]
tags: []
---

> 이 글은 [티스토리](https://v0xe1.tistory.com/9)에서 옮겨온 글입니다.
{: .prompt-info }

## 서론

**HSPACE Knights Frontier** **프로그램의 일환으로 BoB에서 진행한 프로젝트 주제들에 대해 내 생각을 공유하는 시간을 가져보려고 한다.** 이번에 알아볼 프로젝트 이름은 ‘Windows Driver 0-day Research’이다.

또한, 설명에 앞서 해당 프로젝트는 Best of the Best 12th Team &lt;우리 오늘부터 0-day?&gt;에서 진행했으며, 프로젝트에 대한 진행 과정과 시행착오, 성과 등이 매우 고퀄리티로 이미 정리되어 있다. 보다 자세한 정보를 알아보고 싶다면, 해당 연구 글들을 열람해 보면 좋을 것이다.

- [https://today-0day.gitbook.io/main](https://today-0day.gitbook.io/main)
- [https://today-0day.framer.website/](https://today-0day.framer.website/)

또한 이 글도 원래는 이 연구가 어떻게 이뤄졌을까와 내 생각을 공유해보는 차원에서 진행했지만, 프로젝트 진행자 분들이 연구 내용을 인터넷에 자세히 공개해 주셨기 때문에 해당 연구를 간략히 리뷰해 보는 형식으로 진행하려고 한다.

## Windows Driver?

드라이버는 컴퓨터의 운영 체제가 하드웨어 장치와 상호 작용할 수 있도록 하는 소프트웨어이다. 이를 이용하여 커널 모드에서만 접근할 수 있는 자원을 사용하거나, 명령을 내릴 수 있다.

해당 프로젝트에서 발견한 Windows kernel Driver를 사용하는 분야는 5가지였으며, 비교적 취약점이 유력하게 발생할만한 프로그램을 타겟으로 선정해 진행했다고 한다. 공개한 선정 기준은 다음과 같다.

- CVE 발급 이력이 존재할 것
- 취약점을 Handling 할 수 있는 PSIRT팀이 존재하거나, 창구가 존재할 것
- 분야 별 최소 20위 이내 기업일 것
- 국내외 영향력을 고려
- 공신력 있는 기관이 인증한 프로그램일 것

## 퍼징 테스트를 이용한 자동화

퍼징 테스트는 취약점 분석에 있어서 이젠 빠질 수 없는 기술이다. 이 프로젝트에서도 마찬가지로 직접 제작한 퍼저를 통해 자동화를 통해 분석가의 능력으로 인한 편차를 줄이고, 빠른 시간 안에 많은 버그를 찾을 수 있었다고 한다.

직접 설계한 퍼저는 아래와 같이 Static Analysis -> Fuzzing -> Call Stack 확인 순으로 진행해 드라이버 분석부터 크래시 발견 및 디버깅 과정을 모두 일련의 과정으로 자동화하였다.

![](/assets/img/posts/bob-project-windows-driver-0day/1.png)
_https://today-0day.gitbook.io/main/our-approach/undefined_

단계 구분상 1차적으로 타겟에 대한 바이너리를 정적분석을 수행하고, 퍼저의 크래시 결과와 함께 독자적인 Parser로 취약점 여부를 분석했을 것으로 추측된다. 결과적으로 이 퍼저 자동화를 통해 Window Kernel Driver에서 100+개의 Security Bugs를 찾을 수 있었다고 밝혔다.

## 정정 분석 방법: angr-PT

해당 문서를 읽으면서 매우 놀라운 부분이 있었는데, 연구자가 직접 읽어보면서 분석해야 하는 정적 분석에서도 매우 효과적인 자동화 솔루션을 적용했다는 것이다. 해당 연구진들이 독자적으로 개발한 AngrPT는 다음의 솔루션을 제공하고 있다.

- **IoControl Code RIP범위 추적**
- **IOCTL 제약조건 분석**
- **전역변수 종속성 분석**

해당 솔루션은 Kernel Driver의 유저 상호작용 특징인 IoControl Code를 타겟으로 드라이버가 수행하는 동작을 식별하는데에 주력하고 있다.

![](/assets/img/posts/bob-project-windows-driver-0day/2.png)
_https://today-0day.gitbook.io/main/our-approach/angr-pt_

정적분석에 사용되는 IDA Hex Ray는 코드의 전체 형식을 이해하는데 도움을 주지만, 대부분의 Driver코드가 다음과 같이 각 IoControl Code가 특정 변수에서 Offset을 빼는식으로 계산이 되어 분석에 큰 어려움이 있다. 따라서 Hex Ray결과에 IoControl Code별 바이너리상의 Offset을 제시하여 분석가의 편의를 제공하는 것이다.

또한 서로 다른 제약 조건을 갖는 IoControl Code에도 대응할 수 있도록 IRP구조체의 IoStatus Code를 Symbolic Variable로 만들어 각 IOCTL의 제약조건을 자동으로 분석하도록 하였다.

이외에 IOCTL간 종속성으로 인한 버그를 탐지했고, 이를 후술할 MS Fuzzer에 고려하여 적용했다 한다.

## MS Fuzzer?

독자적이고 효율적인 정적 분석 환경에서 나온 정보들을 보다 효율적으로 사용하기 위해 전용 Fuzzer를 개발한 것으로 보인다. 퍼저의 구조도는 다음과 같다.

![](/assets/img/posts/bob-project-windows-driver-0day/3.png)
_https://today-0day.gitbook.io/main/our-approach/ms-fuzzer_

외관상으로 정리해 보자면, Nyx-Qemu를 사용해 다수의 Driver를 퍼징하는 독립실행 환경을 구측하였음을 알 수 있다.

![](/assets/img/posts/bob-project-windows-driver-0day/4.png)
_https://today-0day.gitbook.io/main/our-approach/ms-fuzzer/playmaker-mode_

또한 앞서 말한 AngrPT의 분석 내용(IOCTL 종속성 버그 등)를 실별하는 Play maker mode를 뮤테이션에 적용해서 커버리지 / 크래시 발견 면에서 좋은 성능을 얻었다고 한다. 이외에 퍼징 테스트에 필요한 고전적인 AFK단축, AFL스타일 문제 해결 등 다양한 문제들을 해결하는 과정이 문서에 적혀 있으니 궁금하다면 참고해 보길 바란다.

## 예상 시행착오?

- 처음 Attack Surface의 선정 방식이 매우 궁금하다. 문서에서는 분야별로 프로그램을 조사하여 100개의 벤더사를 대상으로 128개의 프로그램을 Target으로 선정하였다고 한줄 적혀있는데, 이 많은 타겟을 촉박한 프로젝트 기간에 어떻게 조사할 수 있었는지 궁금하다.
- 퍼저를 살펴본 결과, Nyx-Fuzzer와 kAFL을 기반을 개발한 것으로 추측되는데, 두 퍼저를 어떻게 섞을지 많은 고민을 해본 흔적이 상당히 보인다. 여기에 RedQueen등 여러 솔루션을 추가해 수정하는데 많은 어려움을 겪었을 것이다. (나라면 그랬다)

## 참고

- [https://today-0day.gitbook.io/main](https://today-0day.gitbook.io/main)
- [https://today-0day.framer.website/](https://today-0day.framer.website/)
- [https://jeongzero.oopy.io/8d9187bc-63c8-4ed1-8bbc-653c16584249](https://jeongzero.oopy.io/8d9187bc-63c8-4ed1-8bbc-653c16584249)
- [https://github.com/0dayResearchLab/msFuzz](https://github.com/0dayResearchLab/msFuzz)
