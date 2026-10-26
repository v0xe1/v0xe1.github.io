---
title: "CodeQL 공부하기 #이론편"
date: 2025-11-09 21:22:23 +0900
categories: [Research, Personal]
tags: [codeql]
---

> 이 글은 [티스토리](https://v0xe1.tistory.com/10)에서 옮겨온 글입니다.
{: .prompt-info }

Knights Frontier 미션으로, BoB시절 프로젝트에 활용했었던 CodeQL을 좀 더 자세히 다뤄보는 시간을 갖는다. 당시 나는 Fuzzing팀으로 활동했기 때문에 CodeAudit에 비교적 시간을 투자하지 못해서 아쉬웠다. 이 기회에 CodeQL을 사용해서 어떻게 취약점을 탐지할 수 있을지 공부하려고 한다.

## CodeQL이란?

GitHub에서 보안 검사를 자동화하기 위해 개발한 코드 분석 엔진.

- 코드베이스를 관계형 데이터베이스로 변환해, SQL과 유사한 QL 쿼리로 정적 분석을 수행하는 도구.
- 보안 취약점 자동 탐지 및 코드 품질을 개선에 효과적.
- GitHub CI/CD 파이프라인을 통해 자동화 가능.

## 왜 CodeQL을 공부해야 할까?

내가 생각하는 CodeAudit을 통해 손수 보안 취약점을 탐지하고자 하는 방법의 단점은 ‘시간’이다. 정적분석 과정은 타겟의 코드베이스 규모에 따라 다르지만, 대체로 모두 고려하기 힘든 거대한 베이스가 많다. 만약 좋은 눈썰미로 보안 취약점이 내제된 코드를 발견했다고 하더라도 이 버그가 어떤 기능에 영향을 줄 지 심층 분석할 필요가 있다.

CodeQL은 코드를 데이터처럼 처리하는 프로그래밍 언어 및 관련 도구다. 기존 정적 분석기보다 훨씬 더 확실하게 코드를 분석하고 잠재적인 취약점을 찾아낼 수 있도록 설계되었다. 이유는 코드를 단순 텍스트나 구문 트리로 보는 것이 아니라, 관계형 데이터베이스로 변환해 코드의 모든 데이터 흐름 경로와 관계를 참고할 수 있기 때문이다.

결과적으로 도구를 통해 더 많은 취약점 클래스를 탐지하고, 발견된 취약점에 대한 모든 위험 경로를 추적하는데 큰 도움을 줄 수 있다. 이로 인해 사용자는 방대한 코드 분석에 실패해 미탐지 확률을 낮추거나 복잡한 취약점을 놓치지 않고 더 적은 시간으로 빠르게 취약점의 위험 여부를 파악할 수 있다. 실제로 2024년 [GitHub Security Lab](https://github.com/github/securitylab/discussions/828)은 CodeQL 쿼리와 함께 취약점을 제보할 경우 추가 바운티를 제공하기도 했으니, 해커로서 연구의 효율성과 영향력을 극대화할 수 있을 것이다.

### 요약

- CodeQL은 자동화된 보안 검사 및 Variant analysis를 수행하는데 사용되는 분석 엔진
- 코드베이스를 데이터베이스로 변환하고, 쿼리문을 사용해 분석
- 알려진 취약점을 기반으로 유사한 유형의 새로운 취약점을 발견하기 적합함.
- [다양한 프로그래밍 언어 지원](https://codeql.github.com/docs/codeql-overview/supported-languages-and-frameworks/)
  - C/C++
  - C#
  - Go
  - Java/Kotlin
  - JavaScript/TypeScript
  - Python
  - Ruby
  - Rust
  - Swift
  - GitHub Actions workflows

---

## 동작 원리

![](/assets/img/posts/codeql-theory/1.png)
_https://github.blog/developer-skills/github/codeql-zero-to-hero-part-1-the-fundamentals-of-static-analysis-for-vulnerability-research/_

CodeQL은 `Source → Flow → Sink` 모델을 사용해 데이터 오염 추적을 수행한다.

- **Source**: 오염된 데이터가 처음으로 시스템에 들어오는 지점. 즉, 취약점이 발생하는 곳.
- **Flow**: 오염된 데이터가 프로그램 내부에서 변수에 할당되고, 함수 인자로 전달되는 등 이동하는 경로.
- **Sink**: 오염된 데이터가 도달했을 때 실제 문제를 일으키는 위험한 함수나 기능들.

![](/assets/img/posts/codeql-theory/2.png)
_https://github.blog/developer-skills/github/codeql-zero-to-hero-part-1-the-fundamentals-of-static-analysis-for-vulnerability-research/_

따라서 CodeQL의 취약점 탐지는 `Source → Flow → Sink`를 하나로 묶어서 Database로 변환한 다음, 분석자가 비교적 간편한 명령을 내릴 수 있는 쿼리문을 사용해 취약점을 탐지하는 것이다.

## CodeQL 이용방법

[GitHub 문서](https://docs.github.com/ko/code-security/code-scanning/introduction-to-code-scanning/about-code-scanning-with-codeql)에서 소개하는 CodeQL을 이용한 분석 과정은 다음과 같다.

1. 코드베이스를 표현하기 위해 CodeQL 데이터베이스를 생성합니다.
2. 그런 다음 해당 데이터베이스에서 CodeQL 쿼리를 실행하여 코드베이스의 문제를 식별합니다.
3. CodeQL을 코드 스캐닝과 함께 사용하면 쿼리 결과가 GitHub의 코드 스캐닝 알림으로 표시됩니다.

### **CodeQL Database 생성하기**

CodeQL Database를 생성하는 과정은 다음과 같이 진행한다.

1. **Init** : CodeQL Database 디렉토리 구조를 생성합니다.
   - codeql database init …
2. **Extraction** : 대상 code로부터 관계 표현들을 추출합니다.
   - codeql database trace-command …
3. **Finalize** : 생성된 모든 관계 표현들을 import하여 하나의 Database로 만듭니다.
   - codeql database finalize …

위의 모든 일련의 과정은 후술할 `codeql database create …` 명령어를 통해서 한번에 수행 가능하다.

## **Querying?**

- CodeQL에서 사용하는 객체 지향형 쿼리 언어. 구문은 SQL과 유사.
- Datalog(Declarative logic programming language)에 기반을 두고 있기때문에 QL의 모든 연산이 논리연산으로 취급.

예시) QL 라이브러리에서 제공하는 `ExecTainted.ql`

```
/**
 * @name Uncontrolled data used in OS command
 * @description Using user-supplied data in an OS command, without
 *              neutralizing special elements, can make code vulnerable
 *              to command injection.
 * @kind problem
 * @problem.severity error
 * @precision low
 * @id cpp/command-line-injection
 * @tags security
 *       external/cwe/cwe-078
 *       external/cwe/cwe-088
 */

import cpp
import semmle.code.cpp.security.CommandExecution
import semmle.code.cpp.security.Security
import semmle.code.cpp.security.TaintTracking

from Expr taintedArg, Expr taintSource, string taintCause, string callChain
where
  shellCommand(taintedArg, callChain) and
  tainted(taintSource, taintedArg) and
  isUserInput(taintSource, taintCause)
select taintedArg,
  "This argument to an OS command is derived from $@ and then passed to " + callChain, taintSource,
  "user input (" + taintCause + ")"
```

실제로 기본 틀인 `import`, `form`, `where`, `select`안에서 `and`와 같은 논리 연산자가 사용되는 모습을 볼 수 있다. 상단의 주석은 [MetaData](https://codeql.github.com/docs/writing-codeql-queries/metadata-for-codeql-queries/)로, 해당 쿼리에 대한 중요한 정보를 제공한다. 나중에 쿼리를 포함한 보안 취약점을 제보해야 하는 일이 생긴다면 이 메타데이터를 첨부하는 것 같다.

다음은 예시 쿼리문의 설명이다.

### 라이브러리 임포트 (Import)

쿼리 로직을 작성하기 위해 미리 정의된 CodeQL 라이브러리(모듈)를 가져온다.

- `import cpp`: C/C++ 코드를 분석하기 위한 핵심 라이브러리(AST, 함수, 표현식 등)를 가져온다.
- `import semmle.code.cpp.security.CommandExecution`: `system()`, `exec()` 등 OS 명령어를 실행하는 함수(즉, '싱크')가 무엇인지 미리 정의해둔 라이브러리.
- `import semmle.code.cpp.security.Security`: `argv`, `getenv()` 등 '사용자 입력'(즉, '소스')이 무엇인지 정의하는 등, 보안 분석에 공통으로 필요한 유틸리티를 가져옴.
- `import semmle.code.cpp.security.TaintTracking`: **가장 중요한 라이브러리입니다.** '데이터 오염 추적(Taint Tracking)' 기능을 제공. 이 라이브러리가 '소스'에서 '싱크'까지 데이터가 흘러가는 경로를 추적해 준다.

### from 절

`from`절에서는 쿼리에서 사용할 변수들을 자료형과 함께 선언한다.

```
from Expr taintedArg, Expr taintSource, string taintCause, string callChain
```

### where 절

`from`에서 선언한 변수들이 만족해야 할 구체적인 논리 조건을 정의한다.

```
where
  shellCommand(taintedArg, callChain) and //'taintedArg'가 'Sink'(OS 명령어)인가?
  tainted(taintSource, taintedArg) and    //'Source'에서 'Sink'까지 'Flow'가 존재하는가?
  isUserInput(taintSource, taintCause)    //'taintSource'가 'Source'(사용자 입력)인가?
```

### Select 절

`where`의 모든 조건을 통과한 대상(`from`에서 선언한 변수들)을 어떤 형식으로 사용자에게 보여줄지 선택한다.

```
//조건을 만족하면 'Sink' 위치와 경고 메시지를 출력한다.
select taintedArg,
  "This argument to an OS command is derived from $@ and then passed to " + callChain, taintSource,
  "user input (" + taintCause + ")"
```

---

## 실습 준비 - 설치하기

CodeQL을 설치하려고 여러 자료들을 조사해 보았는데, 문서마다 사용 환경 및 목적성이 제각각이라 혼란스러웠다. 최종적으로 다음 방법을 사용해서 windows 11환경에 구축을 수행했다.

첫번째는 Visual Studio Code에서 CodeQL 플러그인 확장자를 사용하는 것이다. 훨씬 직관적으로 CodeQL의 쿼리 실행과 결과 분석들을 진행할 수 있는게 장점이다. 하지만 작동을 위해서는 CodeQL CLI 바이너리가 필요하다.

![](/assets/img/posts/codeql-theory/3.png)

두번째로 [CodeQL CLI 바이너리](https://github.com/github/codeql-cli-binaries?tab=readme-ov-file)는 최신 릴리즈 버전의 `codeql-win64.zip`를 설치하였다. 압축을 풀고, CMD에서 프로그램을 실행할 수 있음을 알 수 있다.

![](/assets/img/posts/codeql-theory/4.png)

원활한 사용을 위해 다음과 같이 Windows 환경변수를 추가해서 명령어로 사용 가능하다.

![](/assets/img/posts/codeql-theory/5.png)

![](/assets/img/posts/codeql-theory/6.png)

## 참고문헌

- [https://codeql.github.com/docs/](https://codeql.github.com/docs/)
- [https://www.youtube.com/playlist?list=PLKseYrrlvWNrtUVn1_oaLbp5gojinNOs3](https://www.youtube.com/playlist?list=PLKseYrrlvWNrtUVn1_oaLbp5gojinNOs3)
- [https://github.blog/author/sylwiabudzynska/](https://github.blog/author/sylwiabudzynska/)
- [https://manal-dev.tistory.com/83](https://manal-dev.tistory.com/83)
- [https://github.blog/developer-skills/github/codeql-zero-to-hero-part-1-the-fundamentals-of-static-analysis-for-vulnerability-research/](https://github.blog/developer-skills/github/codeql-zero-to-hero-part-1-the-fundamentals-of-static-analysis-for-vulnerability-research/)
