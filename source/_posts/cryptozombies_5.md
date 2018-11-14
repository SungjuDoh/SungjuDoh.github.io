---
title: Lv.5
thumbnail: https://mybroadband.co.za/news/wp-content/uploads/2017/05/Etherium-1.jpg
categories:
- BlockChain
- Crypto Zombies (Solidity)
tags:
- blockchain
- solidity
- cryptozombies
- token
- ERC721
- Crypto-collectible assets
---
### 이더리움 상의 토큰
**토큰**
공통 규약을 따르는 스마트 컨트랙트
다른 모든 토큰 컨트랙트가 사용하는 표준함수 집합을 구현하는 것
example : `transfer`, `balanceOf`, `mapping` 등의 함수와 매핑

* 하나의 컨트랙트와 같고, 그 안에서 누가 얼마나 많은 토큰을 가지고 있는지 기록하고, 함수를 이용하여 토큰을 다른 주소로 전송가능하게 한다.

### ERC20 토큰
**ERC20** 토큰들이 똑같은 이르의 동일한 함수 집합을 공유하므로, 똑같은 방식으로 상호작용이 가능하다.

**ERC20 토큰과 상호작용할 수 있는 애플리케이션** 은
* 어떤 ERC20 토큰과도 상호 작용 가능
* 추가 커스텀 코드 없이, 앱에 더 많은 토큰 추가 가능
  (새로운 토큰의 컨트랙트 주소만 끼워넣으면 됨)

### 다른 토큰 표준
**ERC20 토큰**
화폐처럼 사용되는 토큰으로 적절

**좀비 게임의 경우?**
* 좀비는 화폐처럼 분할 불가
* 모든 좀비가 똑같지는 않음
=> *ERC721 토큰*

**ERC721 토큰**
크립토 수집품에 더 적절한 토큰 표준
* 교체 불가 : 각 토큰이 유일하고 분할 불가
* 전체 단위로만 거래 가능하고, 각 토큰은 유일한 ID를 가짐

**표준토큰 사용의 장점**
* 좀비 거래/판매를 위한 경매나 중계 로직을 직접 구현할 필요가 없다.
* 스펙을 맞추고, ERC721 자산 거래의 플랫폼이 있다면 좀비들을 해당 플랫폼에서 사용 가능

### ERC721 표준, 다중 상속
```javascript
contract ERC721 {
  event Transfer(address indexed _from, address indexed _to, uint256 _tokenId);
  event Approval(address indexed _owner, address indexed _approved, uint256 _tokenId);

  function balanceOf(address _owner) public view returns (uint256 _balance);
  function ownerOf(uint256 _tokenId) public view returns (address _owner);
  function transfer(address _to, uint256 _tokenId) public;
  function approve(address _to, uint256 _tokenId) public;
  function takeOwnership(uint256 _tokenId) public;
}
```
현재 초안인 상태의 표준일 뿐이고, 포스팅 작성 당시 공식으로 채택된 구현 버전은 없다. 하나의 구현 가능한 버전 정도로만 살펴본고, 해당 메소드들을 하나씩 구현해보기로 한다.

* **토큰 컨트랙트 구현 시작**
1. 인터페이스를 솔리디티 파일로 복사하여 저장후 Import
2. 해당 컨트랙트를 상속하는 우리의 컨트랙트를 만들고, 각 메소드를 오버라이딩하여 정의
* 참고: 솔리디티는 컨트랙트의 다중 상속이 가능하다.


### 토큰 구현 - balanceOf, ownerOf
* **balanceOf**
`function balanceOf(address _owner) public view returns (uint256 _balance);`
해당 `address`가 토큰을 얼마나 가지고 있는 지 반환
  * 참고: 우리의 토큰은 좀비에 해당

* **ownerOf**
`function ownerOf(uint256 _tokenId) public view returns (address _owner);`
토큰 ID를 받아 소유자의 `address`를 반환
  * 참고: 토큰 ID는 좀비 ID에 해당하고, 우리는 이미 이 정보를 `mapping`으로 저장중이므로 쉽게 구현 가능
  ```javascript
  //zombiefactory.sol
  mapping (uint => address) public zombieToOwner;
  mapping (address => uint) ownerZombieCount;
  ```
  * **refactoring** : 같은 이름의 `ownerOf modifier`가 있으므로, 해당 제어자의 이름을 변경해야한다.

```javascript
//zombieownership.sol
function balanceOf(address _owner) public view returns (uint256 _balance) {
   return ownerZombieCount[_owner];
 }

 function ownerOf(uint256 _tokenId) public view returns (address _owner) {
   return zombieToOwner[_tokenId];
 }
```


### 토큰 구현 - 전송 로직
한 사람이 다른 사람에게 소유권을 넘기는 것을 구현
```javascript
function transfer(address _to, uint256 _tokenId) public;
function approve(address _to, uint256 _tokenId) public;
function takeOwnership(uint256 _tokenId) public;
```
*ERC721* 스펙에서는 토큰 전송 시 2개의 다른 방식이 있다.
  1. `transfer` : 전송 상대의 `address`, 전송하고자 하는 `_tokenId`로 호출
  2. `approve` : 토큰의 소유자가 새로운 소유자의 정보를 이용하여 해당 토큰을 가질 수 있도록 허가하여 저장 (보통 `mapping` 이용)
  이후 새로운 소유자가 `takeOwnership` 호출 시, 해당 컨트랙트는 `msg.sender`가 소유자로부터 토큰을 받을 수 있게 허가를 받은 지 확인 후 해당 토큰을 전송

  * `transfer`와 `takeOwnership` 은 동일 로직을 이용하지만 함수호출자가 각각 토큰 보내는 사람, 받는 사람이라는 점이 다름
  => 로직 private 함수 `_transfer`로 추상화

```javascript
//zombieownership.sol
  mapping (uint => address) zombieApprovals;
 // 공통 로직 함수
function _transfer(address _from, address _to, uint256 _tokenId) private {
    ownerZombieCount[_to]++;
    ownerZombieCount[_from]--;
    zombieToOwner[_tokenId] = _to;
    Transfer(_from, _to, _tokenId);
  }

function transfer(address _to, uint256 _tokenId) public onlyOwnerOf(_tokenId) {
    _transfer(msg.sender, _to, _tokenId);
}

// 새로운 소유자를 허가
function approve(address _to, uint256 _tokenId) public onlyOwnerOf(_tokenId) {
    zombieApprovals[_tokenId] = _to;
    Approval(msg.sender, _to, _tokenId);
}

//msg.sender가 토큰을 가질 수 있도록 승인된 지 확인 후, _transfer 호출
function takeOwnership(uint256 _tokenId) public {
  require(zombieApprovals[_tokenId] == msg.sender);
  address owner = ownerOf(_tokenId);
  // _to 인자, 즉 받는 사람이 msg.sender, 즉 해당함수의 호출자
  _transfer(owner, msg.sender, _tokenId);
}
 ```


### web.js 나중에 공부하기~!

``` solidity ex1
contract testContract{

}
```
