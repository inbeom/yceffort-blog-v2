---
title: Ethereum) 이더리움으로 구현한 선거 시스템
date: 2018-06-21 04:39:57
published: true
mathjax: true
tags:
  - blockchain
  - smart-contract
  - ethereum
description: 유권자의 개인정보보호를 극대화 하는 스마트 컨트랙트를 활용한 이사회 투표
  [원문](https://eprint.iacr.org/2017/110.pdf)  이더리움은 비트코인 다음가는 두번째로 유명한 암호화폐다.
  이더리움은 기본적으로 비트코인의 블록체인을 기반으로 하고 있으며, 이 블록체인은 탈중앙화 되고 개방되어 있는 P2P 네트워크를 기반으로 유지
  되고 ...
category: blockchain
slug: /2018/06/21/A-smart-contract-for-boadroom-voting-with-maximum-voter-privacy/
template: post
---
유권자의 개인정보보호를 극대화 하는 스마트 컨트랙트를 활용한 이사회 투표

[원문](https://eprint.iacr.org/2017/110.pdf)

이더리움은 비트코인 다음가는 두번째로 유명한 암호화폐다. 이더리움은 기본적으로 비트코인의 블록체인을 기반으로 하고 있으며, 이 블록체인은 탈중앙화 되고 개방되어 있는 P2P 네트워크를 기반으로 유지 되고 있다. 블록체인은 중앙에서 금융 원장을 관리하는 것을 제거하기 위해 만들어 졌다. 오늘날 많은 연구원들은 블록체인을 IOT, 헬스케어 와 같은 다양한 문제를 해결하는데 적용하고자 노력하고 있다.

이 논문에서는, 블록체인을 활용한 탈중앙화 인터넷 투표를 구현하려고 한다. 검증 가능성을 제공하는 E-voting 프로토콜은 일반적으로 모든 유권자들에게 일관된 견해를 제공하는 공개 게시판의 존재를 가정한다. 현실적인 예로  International Association of Cryptologic Research(국제 암호학회)에서 하는 선거를 들 수 있다. 이들은 Helois 투표 시스템이라고 하는, 단일 웹서버에 구현된 투표 시스템을 사용한다. 이 서버는 신뢰할 수 있는 정보를 모든 유권자에게 제공한다. 이러한 '신뢰를 가정하는' 것이 아닌, 블록체인에서 실제로 구현가능한 공공 게시판을 구현해보고자 한다. 더 나아가, 유권자들 사이에서 의사 소통을 조정할 책임 이 있는 분산된 선거 환경 또한 고려 해본다. 이를 위해, 블록체인의 기본인 P2P네트워크가 인증된 브로드캐스트 채널로서 적절한지도 탐구하였다.

그러나 현재까지는, 비트코인과 이더리움 모두 확장성 이슈가 발목잡고 있다. 비트코인은 초당 최대 7개의 트랜잭션만 처리가 가능하며, 각 트랜잭션은 임의의 데이터를 저장하는데 80바이트만 사용할 수 있다. 반면에 이더리움은 Gas metric을 사용하여 연산과 저장을 명시적으로 실행하고, 네트워크는 사용자가 사용할 수 있는 가스를 제한해 두었다. 이러한 이유 때문에, 이들 블록체인은 국가적인 크기의 선거를 수행하기에 적합하지 않다. 이러한 이유로, 블록체인을 활용한 작은 그룹의 투표 (40명정도)를 블록체인으로 구현할 수 있는 방안에 대해 연구하였다.

비트코인이 아닌 이더리움이 선택된 이유는 간단하다. 이더리움의 스마트 컨트랙트는 프로그래밍언어를 표현할 수 있으며, 이 코드들은 블록체인에 곧바로 저장될 수 있다. 더욱 중요한 것은, 모든 참가자들이 P2P네트워크 상에서 독립적으로 계약 코드를 실행하고 그 결과를 통해 합의에 도달하게 되는 것이다. 이 뜻은, 유권자가 프로토콜의 정상적인 실행을 확인하기 위해 모든 연산을 각자가 할 필요가 없다는 것이다. 대신 유권자들은 이더리움이 제공하는 합의 컴퓨팅을 신뢰하여 프로토콜의 올바른 실행을 보장받을 수 있는 것이다.

## 이더리움을 활용한 공개 투표 네트워크

이 논문에서 실제로 해당 시스템을 만들어서 깃헙에 공개하였다. [링크](https://github.com/stonecoldpat/anonymousvoting) 모든 유권자가 접근하여 투표하는 인터페이스를 웹 기반으로 제작하였다. 브라우저는 서버에서 실행중인 이더리움 데몬과 상호작용하며, 이러한 프로토콜은 5가지의 스테이지로 구성되어 있고 유권자는 두개에서 최대 3개의 액션을 취해야 한다.

### 구조

여기에는 두 종류의 스마트 컨트랙트가 있으며 모두 이더리움의 solidity language로 구현되어 있다. 첫번째 컨트랙트는 voting contract다. 이는 투표를 위한 프로토콜로, 투표과정을 컨트롤 하고 Open Vote Network상에 시는 두종류의 [영지식증명](https://ko.wikipedia.org/wiki/%EC%98%81%EC%A7%80%EC%8B%9D_%EC%A6%9D%EB%AA%85) 을 검증한다. 이는 모든 유권자들에게 동일한 암호화 코드를 주어 이더리움 네트워크과 연결될 필요없이 로컬 환경에서 사용할 수 있게 해준다. 그리고 사용자들에게는 아래의 3가지 페이지가 주어진다

![Ethereum-election-5-stages](../images/Ethereum-election-5-stages.png)

- 선거 관리자 (admin.html): 여기에는 유권자 목록과 선거에 필요한 질문, 그리고 투표가 제시간에 끝날 수 있도록 설정할 수 있다.
- 유권자 (voter.html): 선거에 등록하고, 단 한번 투표를 할 수 있다.
- 관찰자 (livefeed.html): 선거에서 발생하는 일련의 과정을 지켜 볼 수 있음.

이를 위해서는 선거 관리자와 유권자들은 각각 이더리움 어카운트를 갖고 있다는 것을 가정했다. 유저는 자신의 이더리움 어카운트를 자신의 프라이빗 키로 잠금을 해제할 수 있고, 웹브라우저를 통해 바로 인증할 수 있다. 유저는 이더리움 월렛을 활용할 필요가 없고, 이런 이더리움 클라이언트는 데몬에서 백그라운드로 이루어진다.

### 투표단계

#### SETUP

선거 관리자는 유권자들의 계정을 인증하고, 투표가능한 유권자 정보를 얻기 위해 voting contract를 업데이트 한다. 그리고 투표가 가능한 시간도 설정한다.

- $$t_{finishRegistration}$$: 모든 유권자들은 그들의 voting key $$g^{x_i}$$를 이 시간내에 등록해야 한다.
- $$t_{beginElection}$$: 이더리움에 알릴 투표 시작 시간
- $$t_{finishCommit}$$: 모든 유권자들은 반드시 그들의 투표 $$H(g^{x_iy_i}g^{v_i})$$
를 이 시 간내에 커밋해야 한다.(옵션)
- $$t_{finishVote}$$: 모든 유권자들은 $$g^{x_iy_i}g^{v_i}$$를 이 시간내에 투표해야한다.
- $$\pi$$: 유권자에게 투표시간을 주기위해 투표단계가 활성화 되어야 하는 최소 기간

관리자는 또한 투표용 질문인 $$d$$ 를 설정할 수 있으며, 커밋스테이지가 필요한지도 결정할 수 있다. 마지막으로, 이더리움에 SINGUP 스테이지로 넘어감을 알려야한다.

#### SINGUP

모든 가능한 유권자들은 투표용 질문과 관리자가 설정한 파라미터들을 본뒤에 등록을 해야한다. 등록을 위해서는 투표용 키잉ㄴ $$g^{x_i}$$와 $$ZKP(x_i)$$가 필요하다. 이 키와 proof $$d$$와 함께 이더리움으로 보내진다. 이더리움은 $$t_{finishRegistration}$$ 이 지난 뒤에는 이를 허용하지 않는다. 관리자sms COMMIT 이나 VOTE 단계로 넘어감을 이더리움에 알려야 하고, 모든 유권자들의 재생성된 키 $$g^{y_0}, g^{y_1}, g^{y_2} ... g^{y_n}$$ 는 이러한 전환 되는 동안 이더리움에서 계산된다.


#### COMMIT (옵션)

모든 유권자들은 이더리움 블록체인에 그들의 해쉬를 $$H(g^{x_iy_i}g^{v_i})$$ 올린다. 블록체인 이 이를 최종적으로 받아드리면, contract는 자동적으로 VOTE stage로 넘어간다.


#### VOTE

모든 유권자들은 그들의 암호화된 투표권 $$g^{x_iy_i}g^{v_i}$$ 를 올리고 이를 영지식증명을 통해 증명한다. 투표가 이더리움에 의해 받아지면 deposit $$d$$가 유권자에게 환불된다. 투표 관리자는 이더리움이 최종적인 투표권이 행사되었음을 알리게 되면 이를 인지하게 된다.

#### TALLY

투표관리자는 이더리움에게 개표를 알린다. 이더리움은 이산대수를 무작위로 대입하여 yes vote의 개수를 구한다.

이전에 언급했듯, Open Vote Network는 모든 유권자들로 하여금 그들의 표에대해 개표연산을 요구한다. 이과정에서 deposit $$d$$는 등록된 유권자들에게 투표를 할 수 있는 경제적인 인센티브를 의미한다. 이는 voting protocol에 정상적으로 투표가 진행되었다면 되돌아오고, 그렇지 않다면 돌려받지 못한다.


이러한 방식을 활용하여 투표를 구현한 결과, 유권자당 최소 약 $0.73이 필요한 것이 밝혀졌다. 이는 유권자에게 최대한의 개인정보보호를 제공함과 동시에 공개적으로 검증가능하다는 것을 보았을때, 충분히 합리적인 비용으로 간주 될 수 있다. 이 논문은 블록체인을 활용한 최초의 탈중앙화된 인터넷 투표 프로토콜을 구현한 것이다. 단순히 이더리움 블록체인을 공공 게시판 성격으로 활용한 것 뿐 만 아니라 투표 프로토콜의 올바른 실행을 보장하는 컨센서스 플랫폼으로 사용한 것에 의의가 있다.
향후 블록체인을 활용하여 국가단위의 선거를 치룰 수 있는 지 또한 알아야할 논제로 남게 되었다. 만약 이것이 가능하다면, 이 구현을 위한 전용 블록체인이 필요해 질 수도 있다. 예를 들어, 오로지 e-voting 스마트 컨트랙트만을 담는 이더리움 스타일의 블록체인이 될 수 도 있다. 이 새로운 블록체인은 더 많은 정보를 담기 위해 큰 블록사이즈가 필요할 것이고, 아마도 RSCoin과 비슷한 중앙 집중형식으로 유지 될 수도 있다.

 > RSCoin은 중앙집중화된 코인으로, 중앙은행에서 완전히 통제할 수 있는 비트코인이라고 볼 수 있습니다. 자세한 내용은 [여기](https://totalbitcoin.org/rscoin-a-centralized-version-of-bitcoin/)를 참조해주세요.
