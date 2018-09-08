---
title: Lv.2
categories:
- BlockChain
- Crypto Zombies (Solidity)
tags:
- blockchain
- solidity
- cryptozombies
---

### 주소와 매핑
데이터베이스에 저장된 좀비들에게 주인을 설정하여 멀티 플레이어 게임으로 만든다.
`mapping`과 `address` 두 자료형이 필요하다.
#### 주소
이더리움 블록체인은 은행 계좌와 같은 **계정** 들로 이뤄져있다. 계정은 이더리움 블록체인상의 통화인 **이더** 의 잔액을 가진다. 계정을 통해 다른 계정과 이더를 주고 받을 수 있다.

각 계정은 은행 계쫘 번호와 같은 **주소** 를 가지고 있다. 주소는 특정 계정을 가리키는 고유 식별자이다.

*" 주소는 특정 유저(혹은 스마트 컨트랙트)가 소유한다. "*

#### 매핑
솔리디티에서 구조화된 데이터를 저장하는 또다른 방법이다.

다음과 같이 매핑을 정의할 수 있다.
`mapping (address => uint) public accountaBallance;`

매핑은 기본적으로 key-value 저장소로, 데이터를 저장하고 검색하는 데 이용된다.
`key => value`


``` javascript
pragma solidity ^0.4.19;

contract ZombieFactory {

    event NewZombie(uint zombieId, string name, uint dna);

    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;

    struct Zombie {
        string name;
        uint dna;
    }

    Zombie[] public zombies;

    mapping (uint => address) public zombieToOwner;
    // 좀비 id => 좀비 소유자 주소
    mapping (address => uint) ownerZombieCount;
    // 좀비 소유자 주소 => 좀비 수

    function _createZombie(string _name, uint _dna) private {
        uint id = zombies.push(Zombie(_name, _dna)) - 1;
        NewZombie(id, _name, _dna);
    }

    function _generateRandomDna(string _str) private view returns (uint) {
        uint rand = uint(keccak256(_str));
        return rand % dnaModulus;
    }

    function createRandomZombie(string _name) public {
        uint randDna = _generateRandomDna(_name);
        _createZombie(_name, randDna);
    }

}

```

### msg.sender
솔리디티에는 모든 함수에서 이용 가능한 특정 **전역 변수** 들이 있다.
* Msg.sender : 현재 함수를 호출한 사람(혹은 스마트 컨트랙트)의 주소(**address**)를 가리키는 변수

Tip. 솔리디티에서 함수 실행은 항상 외부 호출자가 한다. 따라서 항상 msg.sender의 값은 존재한다. 컨트랙트는 누군가가 컨트랙트의 함수를 호출할 때까지 블록체인 상에서 아무 것도 하지 않는다.

`msg.sender`를 활용하면, 이더리움 블록체인의 보안성을 이용할 수 있다. 누군가 다른 사람의 데이터를 변경하려면 해당 이더리움 주소와 관련된 개인키를 훔치는 방법 뿐이다.

### require
특정 조건이 참이 아닐 때 함수가 에러 메시지를 발생하고 실행을 멈춘다.

``` javascript
pragma solidity ^0.4.19;

contract ZombieFactory {

    event NewZombie(uint zombieId, string name, uint dna);

    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;

    struct Zombie {
        string name;
        uint dna;
    }

    Zombie[] public zombies;

    mapping (uint => address) public zombieToOwner;
    mapping (address => uint) ownerZombieCount;

    function _createZombie(string _name, uint _dna) private {
        uint id = zombies.push(Zombie(_name, _dna)) - 1;  // 새로운 좀비의 id
        zombieToOwner[id] = msg.sender; // 좀비 id => 좀비 소유자 주소
        ownerZombieCount[msg.sender]++; // 좀비 소유자가 가진 좀비 수 증가
        NewZombie(id, _name, _dna);
    }

    function _generateRandomDna(string _str) private view returns (uint) {
        uint rand = uint(keccak256(_str));
        return rand % dnaModulus;
    }

    function createRandomZombie(string _name) public {
        require(ownerZombieCount[msg.sender] == 0);
        // 유저가 군대에 좀비를 무제한으로 생성하는 것을 방지
        uint randDna = _generateRandomDna(_name);
        _createZombie(_name, randDna);
    }

}
```

### 상속
``` javascript 상속 예시코드
contract Doge {
  function catchphrase() public returns (string) {
    return "So Wow CryptoDoge";
  }
}

contract BabyDoge is Doge {
  function anotherCatchphrase() public returns (string) {
    return "Such Moon BabyDoge";
  }
}
```
위 코드에서 `BabyDoge` 컨트랙트는 `Doge` 컨트랙트를 상속한다.
`BabyDoge`가 컴파일되어 구축될 때, `Doge` 컨트랙트에 정의되는 어떤 public 함수에도 접근이 가능하다.

보통 **부분집합** 관계에서 **상속** 이 사용된다.

### Import
다수의 파일이 있고, 어떤 파일을 다른 파일로 불러오고 싶을 때 사용한다.
`import "./someothercontract.sol";`

코드가 길어져서 여러 파일로 나누어 정리하고, `import`라는 키워드로 다른 파일을 불러온다.

``` javascript
//zombiefeeding.sol
pragma solidity ^0.4.19;

import "./zombiefactory.sol";   // import

contract ZombieFeeding is ZombieFactory {
}
```

``` javascript
//zombiefactory.sol
pragma solidity ^0.4.19;

contract ZombieFactory {

    event NewZombie(uint zombieId, string name, uint dna);

    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;

    struct Zombie {
        string name;
        uint dna;
    }

    Zombie[] public zombies;

    mapping (uint => address) public zombieToOwner;
    mapping (address => uint) ownerZombieCount;

    function _createZombie(string _name, uint _dna) private {
        uint id = zombies.push(Zombie(_name, _dna)) - 1;
        zombieToOwner[id] = msg.sender;
        ownerZombieCount[msg.sender]++;
        NewZombie(id, _name, _dna);
    }

    function _generateRandomDna(string _str) private view returns (uint) {
        uint rand = uint(keccak256(_str));
        return rand % dnaModulus;
    }

    function createRandomZombie(string _name) public {
        require(ownerZombieCount[msg.sender] == 0);
        uint randDna = _generateRandomDna(_name);
        _createZombie(_name, randDna);
    }

}
```

### Storage & Memory
솔리디티에서 변수를 저장할 수 있는 공간
* **Storage**
블록체인 상에 영구적으로 저장되는 변수
컴퓨터의 하드 디스크와 비슷하다.
* **Memory**
임시적으로 저장되는 변수로, 컨트랙트 함수에 대한 외부 호출들이 일어나는 사이에 지워진다.
컴퓨터의 RAM과 비슷하다.

대부분의 경우, 이런 키워드를 사용할 필요가 없다.
솔리디티에서 자동으로
* **상태변수(함수 외부에 선언된 변수) : ** `storage` 로 선언되어, 영구적으로 저장
* **함수 내에 선언된 변수 : **`memory` 로 선언되어, 함수 호출이 종료되면 사라짐

하지만 키워드가 필요한 경우가 있다.
* **구조체와 배열**
``` javascript 키워드 사용 코드예시
contract SandwichFactory {
  struct Sandwich {
    string name;
    string status;
  } // 구조체

  Sandwich[] sandwiches;  // 구조체 배열

  function eatSandwich(uint _index) public {
    Sandwich storage mySandwich = sandwiches[_index];
    // `sandwiches[_index]`를 가리키는 포인터
    mySandwich.status = "Eaten!";
    // 따라서 `sandwiches[_index]`을 영구적으로 변경

    Sandwich memory anotherSandwich = sandwiches[_index + 1];
    // `anotherSandwich`는 단순히 메모리에 데이터를 복사

    anotherSandwich.status = "Eaten!";
    // 임시 변수인 `anotherSandwich`를 변경
    // `sandwiches[_index + 1]`에는 아무런 영향을 끼치지 않음
    sandwiches[_index + 1] = anotherSandwich;
    // 다른 활용 방안 : 임시 변경한 내용을 블록체인 저장소에 저장
  }
}
```

### 함수 접근 제어자
* `public`
모든 컨트랙트에서 사용 가능
* `private`
함수가 정의된 컨트랙트에서만 사용 가능
* `internal`
함수가 정의된 컨트랙트와 이를 상속하는 컨트랙트에서도 사용 가능
* `external`
컨트랙트 바깥에서만 호출될 수 있고, 컨트랙트 내의 다른 함수에서는 사용 불가.
나머지는 **public** 과 동일

### interface
블록체인 상에 있으면서 소유하지 않은 컨트랙트와 상호작용을 하려면 우선 **interface** 를 정의해야 한다.

``` javascript interface 정의 예시코드
contract NumberInterface {
  function getNum(address _myAddress) public view returns (uint);
}
```

dapp 코드에 이런 **interface** 를 포함하면 컨트랙트는 다른 컨트랙트에 정의된 함수의 특성, 호출 방법, 리턴 값에 대해 알 수 있다.

### interface 활용하기
**interface** 를 이용하여, 이더리움 블록체인 상의 다른 어떤 컨트랙트와도 상호작용할 수 있다. 이 때, 상호작용하는 함수가 `public`이나 `external`로 선언되어 있어야 한다.

``` javascript
pragma solidity ^0.4.19;
//zombiefeeding.sol
import "./zombiefactory.sol";

// 크립토좀비가 가장 좋아하는 먹이, 크립토키티와 상호작용할 수 있게하는 KittyInterface 정의
contract KittyInterface {
  function getKitty(uint256 _id) external view returns (
    bool isGestating,
    bool isReady,
    uint256 cooldownIndex,
    uint256 nextActionAt,
    uint256 siringWithId,
    uint256 birthTime,
    uint256 matronId,
    uint256 sireId,
    uint256 generation,
    uint256 genes
  );
}

contract ZombieFeeding is ZombieFactory {

  address ckAddress = 0x06012c8cf97BEaD5deAe237070F9587f8E7A266d;
  // 크립토키티 컨트렉트 주소
  KittyInterface kittyContract = KittyInterface(ckAddress);
  // 크립토키티 인터페이스 생성 및 초기화
  // KittyContract가 해당 주소 컨트랙트를 가리킴

  function feedAndMultiply(uint _zombieId, uint _targetDna) public {
    require(msg.sender == zombieToOwner[_zombieId]);
    // 좀비 주인만 먹이를 줄 수 있도록
    Zombie storage myZombie = zombies[_zombieId];
    // Zombie 구조체 변수를 storage로 선언, 영구 저장
    _targetDna = _targetDna % dnaModulus;
    uint newDna = (myZombie.dna + _targetDna) / 2;
    _createZombie("NoName", newDna);
    // _createZombie 함수는 ZombieFactory 컨트랙트의 internal 함수
  }

}

```

### 다수의 반환값 처리
``` javascript
function multipleReturns() internal returns(uint a, uint b, uint c) {
  return (1, 2, 3);
}
```

다음과 같이 3개의 값을 반환하는 함수가 있다.
이러한 다수의 반환값을 활용하는 방안에는
* 다수 값 할당
``` javascript
function processMultipleReturns() external {
  uint a;
  uint b;
  uint c;
  (a, b, c) = multipleReturns();
}
```
* 단 하나의 값에만 관심이 있는 경우
``` javascript
function getLastReturnValue() external {
  uint c;
  (,,c) = multipleReturns();
  // 다른 필드는 빈칸으로 놓으면 됨
}
```
등이 있다.

``` javascript
// zombiefeeding.sol
pragma solidity ^0.4.19;
import "./zombiefactory.sol";
contract KittyInterface {
  function getKitty(uint256 _id) external view returns (
    bool isGestating,
    bool isReady,
    uint256 cooldownIndex,
    uint256 nextActionAt,
    uint256 siringWithId,
    uint256 birthTime,
    uint256 matronId,
    uint256 sireId,
    uint256 generation,
    uint256 genes
  );
}
contract ZombieFeeding is ZombieFactory {

  address ckAddress = 0x06012c8cf97BEaD5deAe237070F9587f8E7A266d;
  KittyInterface kittyContract = KittyInterface(ckAddress);

  function feedAndMultiply(uint _zombieId, uint _targetDna, string _species) public {
    require(msg.sender == zombieToOwner[_zombieId]);
    Zombie storage myZombie = zombies[_zombieId];
    _targetDna = _targetDna % dnaModulus;
    uint newDna = (myZombie.dna + _targetDna) / 2;

    // 만약 kitty 라면, DNA 마지막 두 자리로 99 가짐
    if (keccak256(_species) == keccak256("kitty")) {
      newDna = newDna - newDna % 100 + 99;
    }
    _createZombie("NoName", newDna);
  }

  function feedOnKitty(uint _zombieId, uint _kittyId) public {
    uint kittyDna;
    (,,,,,,,,,kittyDna) = kittyContract.getKitty(_kittyId);
    // 다수의 반환값 중 마지막 반환값에만 관심
    // interface를 통해 getkitty 함수 접근
    feedAndMultiply(_zombieId, kittyDna, "kitty");
  }

}
```


### web.js 나중에 공부하기~!

``` solidity ex1
contract testContract{

}
```
