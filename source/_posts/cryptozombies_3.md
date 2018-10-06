---
title: Lv.3
categories:
- BlockChain
- Crypto Zombies (Solidity)
tags:
- blockchain
- solidity
- cryptozombies
---

### 컨트랙트의 불변성
이더리움 DApp의 다른 애플리캐이션과는 다른 특징
* **Immutable**
  * 컨트랙트 배포 후, 컨트랙트를 수정하거나 업데이트 불가하다.
  * 컨트랙트로 배포한 최초의 코드는 항상 블록체인에 영구적으로 존재한다.
  * 코드가 곧 법인이 되는 격. 어떤 컨트랙트의 코드를 읽고 검증했다면 이후에 그 누구도 함수를 수정하거나 예상치 못한 결과를 발생시키지 못한다.

* **외부 의존성**
따라서, 대개의 경우 DApp의 **중요한 일부를 수정** 할 수 있도록 하는 **함수** 를 만들어놓는 것이 합리적이다.

### 소유 가능한 컨트랙트
모든 사람이 우리의 컨트랙트를 업데이트할 수 없도록, 컨트랙트를 대상으로 특별한 권리를 가지는 소유자가 있음을 의미한다.

**OpenZeppelin** 솔리디티 라이브에서 가져온 `Ownable` 컨트랙트
``` javascript Ownable 컨트랙트
/**
 * @title Ownable
 * @dev The Ownable contract has an owner address, and provides basic authorization control
 * functions, this simplifies the implementation of "user permissions".
 */
contract Ownable {
  address public owner;
  event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);

  /**
   * @dev The Ownable constructor sets the original `owner` of the contract to the sender
   * account.
   */
  function Ownable() public {
    owner = msg.sender;
  }

  /**
   * @dev Throws if called by any account other than the owner.
   */
  modifier onlyOwner() {
    require(msg.sender == owner);
    _;
  }

  /**
   * @dev Allows the current owner to transfer control of the contract to a newOwner.
   * @param newOwner The address to transfer ownership to.
   */
  function transferOwnership(address newOwner) public onlyOwner {
    require(newOwner != address(0));
    OwnershipTransferred(owner, newOwner);
    owner = newOwner;
  }
}
```
* 컨트랙트가 생성되면 `owner`에 컨트랙트를 **배포한 사람** 을 대입한다.
* 특정 함수들에 대해 오직 **소유자만 접근** 할 수 있도록 제한한다.
* 새로운 소유자에게 해당 컨트랙트의 **소유권을 옮길 수 있도록** 한다.

대부분의 솔리디티 DApp들은 `Ownable`컨트랙트를 복사/붙여넣기 하면서 시작한다. 그리고 첫 컨트랙트는 이 컨트랙트를 상속하여 만든다.

참고
* **생성자(Constructor) :** 컨트랙트가 생성될 때 딱 한 번만 실행된다. 컨트랙트와 동일 이름을 가진다.
* **함수 제어자(Function Modifier) :** 다른 함수들에 대한 접근을 제어하기 위한 일종의 유사 함수다. 보통 함수 실행 전의 요구사항 충족 여부를 확인하는 데에 사용한다.

### 가스(Gas)
*"이더리움 DApp이 사용하는 연료"*

솔리디티에서는 사용자들이 DApp의 함수를 실행할 때마다 **가스** 라는 화폐를 지불해야한다. 사용자는 **이더(ETH, 이더리움 화폐)** 를 이용해서 가스를 사기 때문에, DApp 함수를 실행하려면 사용자들은 **ETH** 를 소모해야한다.

함수를 실행하는 데에 필요한 가스량은 그 함수의 **로직** 이 얼마나 복잡한지에 따라 달라진다.

**1. 가스 비용(gas cost)**
각 연산은 소모되는 가스비용을 가지고, 해당 연산 수행에 소모되는 컴퓨팅 자원의 양이 이 비용을 결정한다.
예를 들어, **storage에 값을 쓰는 것 > 두 정수를 더하는 것**

이더리움에서 코드 최적화는 다른 프로그래밍 언더들에 비해 훨씬 중요하다. 코드가 엉망이라면, 사용자들은 함수를 실행하기 위해 일종의 할증료를 더 내야하기 때문이다.

**2. 가스는 왜 필요한가?**
이더리움은 크고 느리지만 굉장히 안전한 컴퓨터와 같다.
어떤 함수를 실행할 때, 네트워크상의 모든 개별 노드가 함수의 출력값을 검증하기 위해 그 함수를 실행해야 한다. **모든 함수의 실행을 검증하는 수천 개의 노드** 가 이더리움을 분산화하고, 데이터를 보존하며 누군가 검열할 수 없도록 하는 요소이다.

이더리움을 만든 사람들은 누군가가 무한 반복문을 써서 네트워크를 방해하거나, 자원 소모가 큰 연산으로 네트워크 자원을 모두 사용하지 못하도록 만들길 원했다. 그래서 연산 처리에 비용이 들도록 만들었고, 사용자들은 **저장 공간과 연산 사용 시간** 에 따라서 비용을 지불해야 한다.

참고: 다른 합의 알고리즘을 가진 **사이드체인** 에서는 반드시 이렇지는 않다. 따라서 DApp을 사이드체인과 이더리움 메인넷 중 어디에 올릴 지 판단하는 방법들도 추후에 배우게 된다.

**3. 가스를 아끼기 위한 구조체 압축**
`uint`는 크기에 상관없이 256비트의 저장 공간을 미리 잡아 놓는다.
하지만 `struct` 안에서는 변수들을 더 적은 공간을 차지하도록 압축할 것이므로, 가능한 더 작은 크기의 `uint`를 쓰는 것이 좋다.
또한, 동일한 데이터 타입은 하나로 묶어놓는 것이 좋다.
`uint c; uint32 a; uint32 b;` 의 경우, `uint32` 필드가 묶여있다.

### 시간 단위
솔리디티는 시간을 다룰 수 있는 단위계를 기본적으로 제공한다.

`now` 변수를 사용하면 현재의 유닉스 **타임스탬프**(1970.01.01부터 지금까지의 초 단위 합)값을 얻을 수 있다.

참고
유닉스 타임은 전통적으로 32비트 숫자로 저장된다. 따라서 2038년 부터 문제를 일으킬 것이므로, 20년 이상 운영되길 원한다면 64비트 숫자를 써야하고, 이때 가스 비용 또한 고려해야한다.

솔리디티는 또한 `seconds`, `minutes`, `hours`, `days`, `weeks`, `years` 같은 **시간 단위** 를 포함한다. 이들은 그에 해당하는 길이 만큼의 초 단위 `uint` 숫자로 변환된다. `1 hours`와 같이 사용한다.

### 구조체를 인수로 전달
`private` 또는 `internal` 함수에 인수로서 **구조체의 storage 포인터** 를 전달할 수 있다.

``` javascript 구조체 인수 전달 예시코드
function _doStuff(Zombie storage _zombie) internal {
} // 여기서 Zombie 는 구조체명
```

### Public 함수와 보안
`public`과 `external` 함수는 특수한 제어자를 갖지 않는 이상, 어떤 사용자든 이 함수들을 호출하고 자신들이 원하는 모든 데이터를 함수에 전달할 수 있다.
따라서 보안 점검을 위해 이 함수들을 검사하고, 사용자들이 이들을 남용할 수 있는 방법을 생각해봐야한다. 함수를 `internal`로 만드는 것도 한 가지 예다.

### 인수를 가지는 함수 제어자
``` javascript 예시코드
// 사용자의 나이를 저장하기 위한 매핑
mapping (uint => uint) public age;

// 사용자가 특정 나이 이상인지 확인하는 제어자
modifier olderThan(uint _age, uint _userId) {
  require (age[_userId] >= _age);
  _;
}

// `olderThan` 제어자를 인수와 함께 호출
function driveCar(uint _userId) public olderThan(16, _userId) {
}
```

### View 함수를 사용해 가스 절약하기
`view` 함수는 사용자에 의해 외부에서 호출되었을 때 가스를 전혀 소모하지 않는다. 이는 `view`함수가 블록체인 상에서 실제로 어떤 것도 수정하지 않고 데이터를 **읽기만** 하기 때문이다. 이 함수는 실행할 때 **로컬 이더리움 노드** 에 질의만 하고, 블록체인에 어떤 트랜잭션도 만들지 않는다.

**트랜잭션 :** 모든 개별 노드에서 실행되어야 하고, 가스를 소모한다.  

따라서, 가스 사용을 최적화하는 비결은 가능한 모든 곳에 **읽기 전용** 의 `external view` 함수를 쓰는 것이다.

참고
`view`함수가 동일 컨트랙트 내에 있는 다른 함수에서 **내부적** 으로 호출되는 경우에는 가스가 소모된다. 다른 함수가 이더리움에 **트랜잭션** 을 생성하기 때문이다.

### 메모리에 배열 선언하기
`storage`데이터를 사용하는 것은 비싼 연산이다. (특히 쓰기 연산) 해당 데이터를 쓰거나 바꿀 때마다 수천 ㄱ의 노드들이 그들의 하드 드라이브에 데이터를 저장해야하기 때문이다.

비용을 최소화 하기위해, 정말 필요한 경우가 아니면 `storage`데이터를 사용하지 않는 것이 좋다. 이를 위해 때때로는 겉보기에 비효율적으로 보이는 프로그래밍 구성이 필요하다. 어떤 배열에서 내용을 빠르게 찾기 위해, 단순히 변수에 저장하는 것 대신 **함수가 호출될 때마다 배열을 `memory`에 다시 만드는 것** 이 한 가지 예시이다.

`uint[] memory values = new uint[](3);`
`memory` 키워드를 사용하면 되고, 반드시 **길이 인수** 와 함께 생성되어야 한다. 이 배열은 함수가 끝날 때까지만 존재할 것이고, 만약 이 함수가 외부에서 호출되는 `view`함수라면 비용은 무료이다.

### For 반복문
때때로 함수 내에서 배열을 다룰 때, `storage`에 해당 배열을 저장하지 않고 `for`반복문을 사용해서 구성해야 할 필요가 있다.

우리 코드를 예시로 생각해보자.
`getZombiesByOwner` 함수를 구현할 때, 가장 기초적인 구현 방법은 `ZombieFactory`컨트랙트에서 소유자의 좀비군대에 대한 `mapping`을 만들어서 저장하는 것이다.
`mapping (address => uint[]) public ownerToZombies;`

이후에 새로운 좀비를 만들 때마다, 해당 소유자의 좀비 배열에 추가하는 것이다.
`ownerToZombies[owner].push(zombieId);`
이렇게 되면 `getZombiesByOwner` 함수는 다음과 같이 아주 이해하기 쉬운 함수가 된다.
``` javascript
function getZombiesByOwner(address _owner) external view returns (uint[]) {
  return ownerToZombies[_owner];
}
```

* **이 방식의 문제점**
만약 나중에 한 좀비를 원래 소유자에서 다른 사람에게 전달하는 함수를 구현하게 되면 문제가 생긴다. 기존 소유자의 좀비 배열에서 해당 좀비를 지운 후, 좀비가 지워진 구멍을 메우기 위해 기존 소유자의 배열에서 **모든 좀비를 한 칸씩 움직여서** 배열의 길이를 지워야한다.

  `storage` 연산의 가스 비용이 아주 높고, 함수가 실행될 때마다 다른 양의 가스가 소모되기 때문에 이를 예측할 수 도 없다.

* **Solution**
`view` 함수는 외부 호출 시 가스를 소모하지 않기 때문에, `for`반복문을 사용하여 좀비 배열의 모든 요소에 접근하여 **특정 사용자의 좀비들로 구성된 배열** 을 만들 수 있을 것이다. 이렇게 되면 `transfer` 함수는 훨씬 비용을 적게 쓰게된다.


### web.js 나중에 공부하기~!

``` solidity ex1
contract testContract{

}
```
