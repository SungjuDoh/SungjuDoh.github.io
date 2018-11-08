---
title: Lv.4
categories:
- BlockChain
- Crypto Zombies (Solidity)
tags:
- blockchain
- solidity
- cryptozombies
---
### Payable
* **함수제어자**
  * **접근 제어자 (Visibility Modifier) :** 함수가 언제, 어디서 호출될 수 있는지 제어
    * private
    * internal
    * public
    * external
  * **상태 제어자 (Visibility Modifier) :** 블록체인과 상호작용하는 방법
    * View : 어떤 데이터도 저장/변경되지 않음
    * pure : 어떤 데이터도 블록체인에 저장하지 않고, 데이터를 읽지도 않음
    * 두 함수 모두 컨트랙트 외부에서 호출 시 가스 소모가 없다.
  * **사용자 정의 제어자**

**Payable 제어자**
일반적인 웹 서버에서 API 함수를 실행할 때에는, 함수 호출을 통해서 US달러나 비트코인을 보낼 수 없다.

하지만 이더리움에서는 돈(ether), 데이터(Transaction Payload), 컨트랙트 코드 자체 모두가 이더리움 위에 존재하기 때문에, 함수를 실행하는 동시에 컨트랙트에 돈을 지불하는 것이 가능하다.

함수를 실행하기 위해 컨트랙트에 일정 금액을 지불하는 예시 코드를 살펴보자.
``` javascript payable 예시코드
contract OnlineStore {
  function buySomething() external payable {
    // 함수 실행에 0.001이더가 보내졌는지 확실히 하기 위해 확인:
    require(msg.value == 0.001 ether);
    // 이더가 보내졌다면, 함수를 호출한 자에게 디지털 아이템을 전달하기 위한 내용 구성
    transferThing(msg.sender);
  }
}
```
`msg.value`는 컨트랙트로 이더가 얼마나 보내졌는 지 확인하는 방법이다.
DApp 의 자바스크립트 프론트앤드인 web3.js 에서 다음과 같이 함수를 실행할 때 해당 payable 함수가 실행된다.
``` javascript
OnlineStore.buySomething({from: web3.eth.defaultAccount, value: web3.utils.toWei(0.001)})
```
`value` 필드에서 얼마의 이더를 보낼 지 결정하게된다.
참고.
만약 함수가 `payable`로 표시되지 않았을때 이더를 보내려 하면, 함수에서 트랜잭션을 거부할 것이다.

### 출금하기
컨트랙트로 이더를 보내면, 해당 컨트랙트의 이더리움 계좌에 이더가 저장된다. 컨트랙트에서 이더를 인출하는 함수를 작성할 수 있다.
```javascript
contract GetPaid is Ownable {
  function withdraw() external onlyOwner {
    owner.transfer(this.balance);
  }
}
```
`Ownable` 컨트랙트를 import 했다고 가정하고, `owner`와 `onlyOwner`를 사용하고 있다.
* `transfer` : 이더를 특정 주소로 전달할 수 있다.
* `this.balance` : 컨트랙트에 저장되어있는 전체 잔액을 반환한다.

누군가 한 아이템에 대해 초과 지불을 하면, 이더를 되돌려주는 함수도 만들 수 있다.
```javascript
uint itemFee = 0.001 ether;
msg.sender.transfer(msg.value - itemFee);
```

혹은 구매자와 판매자가 존재하는 컨트랙트에서, 누군가 판매자의 아이템을 구매하면 구매자로부터 받은 요금을 판매자에게 전달할 수 있다.
```javascript
seller.transfer(msg.value);
```

```javascript
////zombiehelper.sol
pragma solidity ^0.4.19;

import "./zombiefeeding.sol";

contract ZombieHelper is ZombieFeeding {

  //레벨 업을 위해 지불해야하는 비용
  uint levelUpFee = 0.001 ether;

  modifier aboveLevel(uint _level, uint _zombieId) {
    require(zombies[_zombieId].level >= _level);
    _;
  }

  //우리 컨트랙트의 이더리움 계좌에 저장된 이더를 인출하는 함수
  function withdraw() external onlyOwner{
    owner.transfer(this.balance);
  }
  //현재는 0.001 ether로 지정되어 있는 levelUpFee를 변경하는 함수
  function setLevelUpFee(uint _fee)external onlyOwner{
    levelUpFee = _fee;
  }
  //ether를 지불하여 levelup을 가능하게하는 함수
  function levelUp(uint _zombieId) external payable {
    require(msg.value == levelUpFee);
    zombies[_zombieId].level++;
  }

  function changeName(uint _zombieId, string _newName) external aboveLevel(2, _zombieId) {
    require(msg.sender == zombieToOwner[_zombieId]);
    zombies[_zombieId].name = _newName;
  }

  function changeDna(uint _zombieId, uint _newDna) external aboveLevel(20, _zombieId) {
    require(msg.sender == zombieToOwner[_zombieId]);
    zombies[_zombieId].dna = _newDna;
  }

  function getZombiesByOwner(address _owner) external view returns(uint[]) {
    uint[] memory result = new uint[](ownerZombieCount[_owner]);
    uint counter = 0;
    for (uint i = 0; i < zombies.length; i++) {
      if (zombieToOwner[i] == _owner) {
        result[counter] = i;
        counter++;
      }
    }
    return result;
  }

}
```

### 난수
솔리디티에서 난수를 만들기에 가장 좋은 방법을 `keccak256` 해시 함수를 쓰는 것이다.
``` javascript
uint random = uint(keccak256(now, msg.sender, randNonce)) % 100;
```
`now`의 타임스탬프 값, `msg.sender`, 증가하는 `nounce`를 받고 있다. `nonce`는 딱 한 번만 사용되는 숫자로 똑같은 입력으로 두 번 이상 동일한 해시함수를 실행할 수 없게 한다. `%100` 을 통해 마지막 2자리 숫자만 받고, 이를 통해 0과 99사이의 완전한 난수를 얻을 수 있다.

하지만 이 메소드는 정직하지 않은 노드의 공격에 취약하다.

이더리움에서는 컨트랙트의 함수를 실행하면 **트랜잭션(transaction)** 으로서 네트워크의 노드 하나 혹은 여러 노드에 실행을 알린다. 그 후 네트워크의 노드들은 여러 트랜잭션을 모으고. **작업증명(PoW)** 이라는 매우 복잡한 수학적 문제를 풀기 위한 시도를 한다. 그리고 **블럭(해당 트랜잭션 그룹 + 작업증명)** 으로 네트워크에 배포한다.

한 노드가 어떤 PoW를 풀면, 다른 노드들은 그 PoW를 풀려는 시도를 멈추고, 해당 노드가 보낸 **트랜잭션 목록이 유효한 것인지** 검증한다. 유효하다면 해당 블록을 받아들이고 **다음 블록** 을 풀기 시작한다.

**이 과정이 우리의 난수 함수를 취약하게 만든다.**

내가 만약 노드를 실행하고 있다면, 나는 **오직 나의 노드에만** 트랜잭션을 알리고 이것을 공유하지 않을 수 있다. 그 후 함수를 실행하고, 유리하지 않은 결과가 나오면, 내가 풀고 있는 다음 블록에 **해당 트랜잭션을 포함하지 않는 것** 을 선택한다. 원하는 결과를 얻을 때까지 무한대로 반복할 수 있고, 이득을 볼 수 있다.

이더리움에서 난수를 안전하게 만드는 한 가지 방법은 외부의 난수 함수에 접근할 수 있도록 **오라클** (이더리움 외부에서 데이터를 받아오는 안전한 방법 중 하나)을 사용하는 것이다.

```javascript
//zombieattack.sol
import "./zombiehelper.sol";

contract ZombieBattle is ZombieHelper {

  uint randNonce = 0;
  //공격하는 좀비라면 승률 70%
  uint attackVictoryProbability = 70;

  //난수를 생성하는 함수
  function randMod(uint _modulus) internal returns(uint) {
    randNonce++;  //매번 증가해서 같은 입력으로 동일한 해시함수를 두 번 이상 실행하지 않음
    return uint(keccak256(now, msg.sender, randNonce)) % _modulus;
  }

  // 공격하는 함수
  function attack(uint _zombieId, uint _targetId) external{

  }
}

```


### 공통 로직 구조 개선하기 6,7 공통구조는 require
작성한 코드 중 `changeName()`, `changeDna()`, `feedMultiply` 에 호출자와 `_zombieId`의 소유자인지 확인하는 코드가 공통적으로 들어간다. 이를 `modifer`를 추가 정의하여 코드를 **refactoring** 한다.
```javascript
modifier ownerOf(uint _zombieId) {
   require(msg.sender == zombieToOwner[_zombieId]);
   _;
 }
```


### 공격 기능 추가하기 - 승리, 패배
공격 기능을 추가함과 동시에, 좀비들이 얼마나 이기고 졌는지 추적함으로서 "좀비 순위표"를 유지할 수 있다.
DApp에서는 다양한 방식으로 해당 데이터를 저장할 수 있다.
* 개별적인 매핑
* 순위표 구조체
* `Zombie` 구조체 자체에 추가

간결함을 유지하기 위해 `Zombie`구조체에 상태를 저장하는 변수를 추가한다.
```javascript
//zombiefactory.sol
struct Zombie {
  string name;
  uint dna;
  uint32 level;
  uint32 readyTime;
  uint16 winCount;
  uint16 lossCount;
}
```

다음으로 `attack`함수를 통해
```javascript
// zombieattack.sol
function attack(uint _zombieId, uint _targetId) external ownerOf(_zombieId) {
  // Zombie storage의 포인터를 얻어서 나와 적 좀비의 상호작용이 쉽도록한다.
  Zombie storage myZombie = zombies[_zombieId];
  Zombie storage enemyZombie = zombies[_targetId];
  uint rand = randMod(100); // 전투 결과를 결정하는 난수
  // 승리
  if (rand <= attackVictoryProbability) {
    myZombie.winCount++;
    myZombie.level++;
    enemyZombie.lossCount++;
    feedAndMultiply(_zombieId, enemyZombie.dna, "zombie");  // 섭취한 적의 dna와 자신의 dna가 결합되어 새로운 좀비탄생
  } else {  // 패배
    myZombie.lossCount++;
    enemyZombie.winCount++;
  }
  _triggerCooldown(myZombie); // 해당 좀비는 하루에 한 번만 공격 가능
}

```

### web.js 나중에 공부하기~!

``` solidity ex1
contract testContract{

}
```
