---
title: "1. Hello Ethernaut"
date: 2025-10-02 18:14:22 +0900
categories: [CTF & Wargame, Ethernaut]
tags: [wargame]
---

> 이 글은 [티스토리](https://v0xe1.tistory.com/5)에서 옮겨온 글입니다.
{: .prompt-info }

HSPACE 나이츠 프론티어 프로젝트의 일환으로, 평소 궁금했던 Web3에 대한 공부를 수행하기 위해 OpenZepplin에서 만든 Web3 wargame인 The Ethernaut를 풀어보았습니다. 아직은 Solidity문법이 뭔지 잘 모르지만, 일단 풀어 보면서 알아볼 예정입니다.

---

## Hello Ethernaut

이 wargame이 어떻게 돌아가는지 기본적인 방법을 알아가는 문제입니다. 1번~9번까지 안내에 따라 설정하면 됩니다.  
MetaMask를 처음 시작하여 지갑에 이더가 없는 경우, 이더 파우셋으로 무료 이더를 받을 수 있습니다. 제 경우 기존 MetaMask가 지원하는 파우셋 기능이 찾아봐도 안보여서(신규 패치 후 사라졌을 수 있습니다.) 다른 방법으로 Sepolia Ether를 확보해야 했습니다.

## 테스트용 Sepolia Ether 채굴하기

다음 소개할 사이트는 테스트용 Sepolia Ether를 본인의 컴퓨터 자원을 사용해 채굴해 입금받을 수 있는 사이트입니다. 채굴 시작 후 최소 0.05ETH부터 입금이 가능합니다.  
[https://sepolia-faucet.pk910.de/#/](https://sepolia-faucet.pk910.de/#/)  
alchemy같은 faucet사이트에서는 이제 기본 ETH를 일정량 보유하고 있어야 파우셋을 받을 수 있기 때문에 위 방법을 사용하면 조금 걸리더라도 확실하게 받을 수 있을 것입니다.

---

## 풀이

지갑에 성공적으로 테스트 이더를 보유하게 되었다면, wargame사이트에 재접속했을 때 MetaMask와 사이트가 연결을 시도합니다. 이 때 지갑에서 연결 수수료를 요구하기 때문에 소량의 이더가 필요한 것입니다. 이후 사이트에서 레벨 인스턴스 생성 버튼을 클릭해 인스턴스를 생성해달라고 요청하고, 승인하여 문제의 컨트랙트를 조사합니다.

![](/assets/img/posts/ethernaut-01-hello-ethernaut/1.png)

연결 후 문제의 설명대로 console에 명령들을 입력해보면, 다음과 같은 현재 연결된 메인 스마트 컨트랙트의 메서드를 볼 수 있습니다. 여러 메서드들을 실행해 보면서 문제를 조사합니다.

![](/assets/img/posts/ethernaut-01-hello-ethernaut/2.png)

몇몇 메소드를 실행해 보면, 위 내용처럼 문제를 어떻게 풀어야 하는지 설명이 존재합니다. 위와 같이 알아낸 password를 인증하면, MetaMask에 트랜젝션 요청이 왔음을 확인할 수 있습니다.

![](/assets/img/posts/ethernaut-01-hello-ethernaut/3.png)

컨펌 버튼을 누르고, 문제 사이트에서 레벨 완료하기 버튼을 누르면 됩니다.

## 소스코드

문제를 다 풀고 나서야 보여줍니다. Solidity문법을 처음 보기 때문에 여태 사용했던 메서드가 실제로는 어떻게 생겼는지 알 수 있어서, 다음 문제 풀 때 참고합시다.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Instance {
    string public password;
    uint8 public infoNum = 42;
    string public theMethodName = "The method name is method7123949.";
    bool private cleared = false;

    // constructor
    constructor(string memory _password) {
        password = _password;
    }

    function info() public pure returns (string memory) {
        return "You will find what you need in info1().";
    }

    function info1() public pure returns (string memory) {
        return 'Try info2(), but with "hello" as a parameter.';
    }

    function info2(string memory param) public pure returns (string memory) {
        if (keccak256(abi.encodePacked(param)) == keccak256(abi.encodePacked("hello"))) {
            return "The property infoNum holds the number of the next info method to call.";
        }
        return "Wrong parameter.";
    }

    function info42() public pure returns (string memory) {
        return "theMethodName is the name of the next method.";
    }

    function method7123949() public pure returns (string memory) {
        return "If you know the password, submit it to authenticate().";
    }

    function authenticate(string memory passkey) public {
        if (keccak256(abi.encodePacked(passkey)) == keccak256(abi.encodePacked(password))) {
            cleared = true;
        }
    }

    function getCleared() public view returns (bool) {
        return cleared;
    }
}
```
