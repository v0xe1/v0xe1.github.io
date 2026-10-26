---
title: "CodeQL 공부하기 #실습편"
date: 2025-11-09 21:25:56 +0900
categories: [Research, Personal]
tags: [codeql]
---

> 이 글은 [티스토리](https://v0xe1.tistory.com/11)에서 옮겨온 글입니다.
{: .prompt-info }

## DataBase만들기

CodeQL을 사용하기 위해서는 먼저 DataBase를 만들어야 한다. CodeQL Database를 생성하는 기본 명령어는 다음과 같다.

```bash
codeql database create {database_name} -l {language} -c {build command}
```

- `{database_name}`으로 이름을 설정
- `-l {language}` 옵션으로 타겟의 언어(c/c++, java, python, …)을 설정
- `-c {build command}` 옵션으로 command(build, run)를 설정

간단한 예시로, 분석할 소스코드 example.c에 대한 DataBase를 만들고자 한다면 다음처럼 생성하면 된다.

```bash
codeql database create example.db -l cpp -c "gcc -o example example.c"
```

![](/assets/img/posts/codeql-practice/1.png)

성공적으로 `example.db`가 생성되었음을 알 수 있다. 그리고 Visual Studio Code에서 CodeQL 플러그인을 설치했다면, 이를 통해 CodeQL Database를 불러올 수 있다.

![](/assets/img/posts/codeql-practice/2.png)

## CodeQL Query

**CodeQL Database의 코드를 분석하려면 CodeQL QL이 필요하다. (.ql)**

그리고 ql파일을 실행하기 위해선 `qlpack.yml`이라는 codeQL 패키지의 메타데이터와 의존성을 정의한 파일이 같은 경로에 필요하다. 예시로 cpp라이브러리를 불러오는 구조는 아래와 같다.

```yaml
# Change 'getting-started' to the name of a user or organization that you have write access to.
name: getting-started/codeql-extra-queries-cpp
version: 0.0.0
dependencies:
  # This uses the latest version of the codeql/cpp-all library.
  # You may want to change to a more precise semver string.
  codeql/cpp-all: "*"
```

- **Name**: 패키지의 이름
- **Version**: 패키지의 버전
- **Dependencies**: 패키지가 의존하는 다른 CodeQL 라이브러리나 모듈

CodeQL repo에서는 기본적인 코드 분석에 대한 쿼리를 제공하고 있다. 필요하다면 아래 방법으로 복제해 사용해도 좋다.

```bash
git clone git@github.com:github/codeql.git codeql-repo
```

### Query 작성하기

실습을 위해 쿼리를 작성해 보자. 먼저 제작한 example.c는 UAF를 일으키는 소스코드이다. 내용은 다음과 같다.

```c
#include <stdio.h>
#include <stdlib.h> // malloc, free를 위해 필요
#include <string.h>

typedef struct {
    int id;
    char name[20];
} User;

int main() {
    User *userData;

    userData = (User *)malloc(sizeof(User));
    userData->id = 1001;
    strcpy(userData->name, "Alice");

    printf("데이터 생성: ID = %d, 이름 = %s\n", userData->id, userData->name);
    printf("메모리 주소: %p\n", (void *)userData);

    printf("\n메모리 해제...\n");
    free(userData);

    printf("\n--- UAF 발생 지점 ---\n");
    printf("해제된 메모리의 ID 읽기 시도: %d\n", userData->id); // vul

    return 0;
}
```

CodeQL Query를 사용해 해당 소스코드의 UAF취약점을 검사하도록 검사하는 쿼리문을 작성해 보자.

```
/**
 * @name Use After Free
 * @description Using memory after it has been freed can lead to
 * memory corruption or arbitrary code execution.
 * @kind problem
 * @problem.severity error
 * @precision high
 * @id cpp/use-after-free
 * @tags security
 * external/cwe/cwe-416
 */

import cpp
import semmle.code.cpp.security.Memory // UAF 탐지를 위한 핵심 라이브러리

from
  MemoryAccess use,  // 1. 메모리에 접근(use)하는 모든 표현식
  FunctionCall free  // 2. 메모리를 해제(free)하는 모든 함수 호출
where
  // 3. use가 free에 의해 해제된 메모리를 사용하는지 검사
  use.isUseAfterFree(free)
select
  use, // 4. 문제의 '사용' 지점을 결과로 선택
  "This expression accesses memory that was freed at $@.", free, free.getTarget().getName()
```

물론 이렇게 직접 쿼리문을 작성하지 않아도, CodeQL git repo에 등록되어 있는 쿼리문을 가져와 사용해도 좋다. 경로는 다음과 같다.

- `C:\hackTools\codeql\cpp\ql\src\Critical\UseAfterFree.ql`

## CodeQL database analyze

이제 PowerShell환경에서 먼저 Database Analyze를 수행해보자.

```powershell
codeql database analyze `
    C:\hackTools\tmp\example2.db `
    C:\hackTools\codeql\cpp\ql\src\Critical\UseAfterFree.ql `
    --format=sarif-latest `
    --output=uaf-results.sarif
```

- **분석에 사용할 데이터베이스 이름**: my_db
- **분석에 사용할 ql 파일 경로**: `C:~~\UseAfterFree.ql`
- **저장할 파일 형식**: sarif
- **저장할 파일의 이름과 경로**: 현재 디렉토리에 uaf-results.sarif

원래는 직접 작성한 쿼리문을 사용하려고 했으나, 라이브러리 종속성 문제로 기존의 쿼리문을 먼저 사용해 보았다. 분석 결과는 [sarif파일](https://docs.github.com/ko/code-security/code-scanning/integrating-with-code-scanning/sarif-support-for-code-scanning)로 저장된다. SARIF 표준은 JSON기반의 파일로, 정적 분석 도구가 결과를 공유하는 방법을 간소화하는 데 사용된다.

sarif파일 이외에도 다양하게 저장할 수 있는데, 대표적으로 .csv형식이 있다. 이는 엑셀 파일로 볼 수 있어서 좀 더 대량의 결과를 보기 편리하다. (참고: [https://geun-yeong.tistory.com/42](https://geun-yeong.tistory.com/42))

![](/assets/img/posts/codeql-practice/3.png)

![](/assets/img/posts/codeql-practice/4.png)

sarif파일은 json기반이기 때문에, 읽기 편하려면 개행 추가나 전용 Reader가 있어야 한다. 나는 Visual Studio Code에서 지원하는 SARIF Viewer를 사용했다.

![](/assets/img/posts/codeql-practice/5.png)

## 분석 결과 정리

SARIF View를 사용하면 다음과 같이 Page Viewer로 쉽게 정보를 열람할 수 있다. 의도한 대로 UAF취약점을 잘 감지한 모습이다.

![](/assets/img/posts/codeql-practice/6.png)

![](/assets/img/posts/codeql-practice/7.png)

뿐만 아니라 소스코드의 종속성 분석 결과도 링킹하여 실시간으로 분석해 준다. 정말 편리하다!

## Trying Idea…

- CodeQL repo를 가져와 사용하면 라이브러리 종속성 문제에서 해결된다.
- IDA와 같은 handray code는 DataBase생성 시 문법적인 방면에 어려움이 있을 것으로 예상된다. 따라서 가급적 Github에 존재하는 오픈소스 프로그램을 타겟으로 하는게 적합해 보인다.
- 다음에 모의해킹을 해야 한다면, 사용되고 있는 오픈소스 라이브러리를 대상으로 분석해 보아도 좋을 것 같다. (리액트, 그누보드 등)
