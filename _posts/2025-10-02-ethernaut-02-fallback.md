---
title: "2. Fallback"
date: 2025-10-02 23:47:17 +0900
categories: [CTF & Wargame, Ethernaut]
tags: [wargame]
---

> 이 글은 [티스토리](https://v0xe1.tistory.com/6)에서 옮겨온 글입니다.
{: .prompt-info }

주어진 컨트랙트의 소유권을 탈취하고, 모든 자금을 인출해야 한다. 제공된 소스코드는 다음과 같다.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Fallback {
    mapping(address => uint256) public contributions;
    address public owner;

    constructor() {
        owner = msg.sender;
        contributions[msg.sender] = 1000 * (1 ether);
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "caller is not the owner");
        _;
    }

    function contribute() public payable {
        require(msg.value < 0.001 ether);
        contributions[msg.sender] += msg.value;
        if (contributions[msg.sender] > contributions[owner]) {
            owner = msg.sender;
        }
    }

    function getContribution() public view returns (uint256) {
        return contributions[msg.sender];
    }

    function withdraw() public onlyOwner {
        payable(owner).transfer(address(this).balance);
    }

    receive() external payable {
        require(msg.value > 0 && contributions[msg.sender] > 0);
        owner = msg.sender;
    }
}
```

문제 해결을 위해 wargame사이트에서는 몇가지 방법을 고려해 보라고 한다.

- ABI를 통해 이더를 전송하는 방법
- ABI 외부에서 이더를 전송하는 방법
- wei와 ether 단위 간의 변환 (콘솔에서 `help()` 명령어 참고)
- Fallback 함수의 작동 방식

## 문제 분석

주어진 컨트랙트 Fallback의 소스코드는 2개의 변수를 갖는 것으로 보인다. Solidity문법을 검색해 보면서 한줄씩 해석해 보았다.

![](/assets/img/posts/ethernaut-02-fallback/1.png)

- `mapping(address => uint256) public contributions;`
  - 각 주소(`address`)가 컨트랙트에 얼마나 기여했는지(`uint256`)를 기록하는 변수.
- `address public owner;`
  - 컨트랙트 소유자 주소로 추측.

![](/assets/img/posts/ethernaut-02-fallback/2.png)

그 다음에는 생성자가 나오며, 여기서 `owner = 컨트랙트 배포자(msg.sender)`를 수행. `msg.sender`에게 1000ETH만큼의 `contributions` 부여한다. `modifier onlyOwner()`는 소유자와 배포자가 같은지 확인하는 용도로 보인다.

> modifier(제어자)는 함수의 행동을 바꾸거나 조건을 추가하는 재사용 가능한 코드 조각이라고 한다. 주로 함수 실행 전에 특정 조건을 검사하는 용도로 사용된다.

![](/assets/img/posts/ethernaut-02-fallback/3.png)

핵심 기능으로 보이는 `contribute()`함수는 다음처럼 해석했다.

1. `require()`를 이용해 호출 시 전송한 이더가 0.001 ETH 미만이어야 함.
2. 보낸 이더만큼 배포자의 `contributions`을 증가
3. 만약 배포자의 총 기여금이 현재 `owner`의 기여금(초기값 1000 ETH)보다 커지면, **배포자가 새로운 `owner`가 된다.**

> payable은 함수가 이더(ETH)를 받을 수 있도록 만들어주는 키워드다. 이 키워드가 붙은 함수는 호출될 때 이더를 함께 전송받을 수 있다.

`getContribution()`는 저장된 `contributions`를 반환하는 함수이고, `withdraw()`는 말 그대로 컨트랙트에 쌓인 모든 이더를 인출하는 함수로 보인다.

`receive()`함수는 다음처럼 해석했다.

![](/assets/img/posts/ethernaut-02-fallback/4.png)

1. **배포자는 반드시 0보다 큰 이더를 보내야 한다. 그리고,**
2. **배포자의 `contributions`이 0보다 커야 한다.(이전에 아주 작은 금액이라도 한번 입금해 기여한 기록이 있어야 한다는 뜻.)**
3. 두 조건을 만족하면 이더 배포자가 owner가 된다.

> external은 함수를 오직 컨트랙트 외부에서만 호출할 수 있도록 지정하는 가시성(visibility) 키워드라고 한다. 이외에 public과 달리, external 함수는 인자값을 메모리에 복사하지 않고 바로 calldata에서 읽어오기 때문에 가스비가 더 저렴하다고 한다.  
> 만약 내부에서 호출해야 한다면, this.myExternalFunction()처럼 this 키워드를 사용해야 한다.

이쯤에서 Fallback이 무엇인지 궁금해, 관련 내용을 검색해 보았다. Solidity에서 자주 사용되는 함수라는 듯 하다.

- [https://log4day.tistory.com/12](https://log4day.tistory.com/12)
- [https://medium.com/gopax/smart-contract%EC%97%90%EC%84%9C-%EB%B0%9C%EC%83%9D%ED%95%98%EB%8A%94-%EB%B3%B4%EC%95%88-%EC%B7%A8%EC%95%BD%EC%A0%90-%EB%A6%AC%EC%84%9C%EC%B9%98-ethernaut-0cb8a51fd97d](https://medium.com/gopax/smart-contract%EC%97%90%EC%84%9C-%EB%B0%9C%EC%83%9D%ED%95%98%EB%8A%94-%EB%B3%B4%EC%95%88-%EC%B7%A8%EC%95%BD%EC%A0%90-%EB%A6%AC%EC%84%9C%EC%B9%98-ethernaut-0cb8a51fd97d)

이전에는 receive함수와 결합된 형태였으나, 세분화된 것으로 보인다. 또한 관련 실제 보안 취약점들도 찾아보았는데, 핵심은 로직 버그를 잘 찾는게 요건인 듯 하다.

## 풀이 과정

분석한 내용을 정리하자면, 우선 컨트랙트의 소유권을 탈취하기 위해서 다음의 과정을 거쳐야 할 것 같다.

1. 배포자로서 `contribute()`함수를 이용해 0.001미만의 이더를 입금한다.
   1. `contributions`는 입금한 이더 만큼 기록된다.
2. `contributions`이 0 이상이므로, `recevie()`함수를 이용할 수 있다.
   1. 배포자로서 0.001미만 이더를 한번 더 입금한다.
   2. 두 조건을 만족해 `recevie()`로 컨트랙트의 owner를 배포자로 변경한다.

따라서 콘솔 명령어로 다음처럼 입력하면 0.001미만의 매우 작은 이더를 전송한다.

- `await contract.contribute({value: 1})`

![](/assets/img/posts/ethernaut-02-fallback/5.png)

그 다음, external payable로 명시되어 있는 recevie함수를 호출하기 위해 문제 컨트랙트의 주소로 데이터 없이 직접 전송하면 recevie함수가 자동으로 호출된다. 이를 이용해서 다음의 명령을 수행한다.

- `sendTransaction({from: player, to: instance, value: 1})`

![](/assets/img/posts/ethernaut-02-fallback/6.png)

전달 결과, contract.owner변수가 배포자인 내 계정으로 변경됨을 확인해 볼 수 있다.

![](/assets/img/posts/ethernaut-02-fallback/7.png)

![](/assets/img/posts/ethernaut-02-fallback/8.png)

owner가 되었기 때문에, witdraw함수를 호출할 수 있는 권한이 생겼다. 이를 이용해 다음과 같이 모든 잔액을 호출하도록 contract.withdraw()함수를 콘솔에 입력한다. MetaMask에 온 출금 알림을 승인하면 모든 클리어 조건을 만족할 수 있다.

> Note.
>
> 컨트랙트로 이더가 들어오고 나가는 기본 원리와 fallback 메서드 사용법을 알고 있습니다.
>
> 또한 OpenZeppelin의 Ownable 컨트랙트를 학습했으며, 이를 통해 일부 메서드의 사용을 권한 있는 주소로 제한하는 방법도 배웠습니다.
