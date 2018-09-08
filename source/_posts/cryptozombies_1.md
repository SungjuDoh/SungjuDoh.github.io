---
title: Lv.1
categories:
- BlockChain
- Crypto Zombies (Solidity)
tags:
- blockchain
- solidity
- cryptozombies
---
### 컨트랙트
이더리움 애플리케이션의 기본적인 구성 요소 (모든 변수/함수 는 어느 한 컨트랙트에 속함)
모든 솔리디티 소스 코드는 해당 코드가 이용해야 하는 솔리디티 버전을 선언하며 시작한다.

### 상태변수
컨트랙트 저장소에 영구적으로 저장된다.
a.k.a. 이더리움 블록체인에 기록되는 것으로 데이터베이스에 데이터를 쓰는 것과 동일하다.

### 정수
* uint(부호 없는 정수, >= 0)
* int(부호 있는 정수

  
실제로 uint는 uint256를 의미한다. (256비트의 부호 없는 정수)
uint8, uint16, uint32 등 도 있다.


### 수학 연산
* 덧셈 : `x + y`
* 뺄셈 : `x - y`
* 곱셈 : `x * y`
* 나눗셈 : `x / y`
* 모듈러 (나머지) : `x % y`
* 지수연산 (x의 y제곱) : `x ** y`

``` javascript
pragma solidity ^0.4.19;

contract ZombieFactory {

    uint dnaDigits = 16;  // 좀비 DNA: 16자리
    uint dnaModulus = 10 ** dnaDigits;  // 16자리보다 큰 수를 16자리 숫자로 줄임

}
```

### 구조체
여러 특성을 가진 자료형을 생성할 수 있다.
참고) string: 임의의 길이를 가진 UTF-8 데이터

### 배열
* 정적 배열
`uint[2] fixedArray;`
* 동적 배열
`uint[] dynamicArray;`


tip.
구조체 배열도 생성 가능하다.
상태 변수는 컨트랙트 저장소에 영구적으로 저장되므로,
구조체 동적 배열은 마치 데이터베이스처럼 컨트랙트에 구조화된 데이터를 저장하는 데 유용하다.

public으로 배열 선언 시 getter 메소드가 자동적으로 생성 되어,
다른 컨트랙트들이 이 배열을 읽을 수 있게되므로 (read only), 컨트랙트에 공개 데이터를 저장할 때 유용하다.

``` javascript
pragma solidity ^0.4.19;

contract ZombieFactory {

    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;

    struct Zombie {
        string name;
        uint dna;
    } // 좀비의 특성을 멤버로 가지는 구조체

    Zombie[] public zombies;  // public 좀비 군대 저장소

}
```
### 함수 선언
tip.
관례적으로 함수 인자명을 언더바(\_)로 시작해서 전역 변수와 구별한다.

### 구조체 생성
새로운 구조체를 생성하고, 이를 구조체 배열에 추가할 수 있다.

### Private/ Public 함수
함수는 기본적으로 public으로 선언된다. 이는 누구나 해당 함수를 호출 및 실행할 수 있음을 의미한다.
따라서 보통의 경우 private으로 함수를 선언하는 것이 바람직하다. 이는 동일 컨트랙트 내의 다른 함수들만이 이 함수를 호출할 수 있음을 의미한다. 관례적으로 private 함수명은 언더바(\_)로 시작한다.


### 반환값
반환값의 자료형을 포함한다.
`function sayHello() public view returns (string) {`

### 함수 제어자
``` javascript 함수 제어자 예시코드
function _multiply(uint a, uint b) private pure returns (uint) {
  return a * b;
}
```
* view: 상태를 변화시키지 않는 함수 (a.k.a. 어떤 값을 변경하지 않음)
* pure: 앱에서 어떤 데이터도 접근하지 않는 함수 (ex. 앱에서 읽는 것도 없이, 반환값이 함수에 전달된 인자값에 따라 달라지는 함수)

### Keccak256
이더리움은 내장 해시 함수로 SHA3의 한 버전인 keccak256를 가진다. 해 함수는 기본적으로 입력 스트링을 랜덤 256비트 16진수로 매핑한다.

참고: 블록체인에서 안전한 의사 난수 발생기는 매우 어려운 문제다.

### 형 변환
``` javascript 형 변환 예시코드
uint8 a = 5;
uint b = 6;
uint8 c = a * b; // a * b가 uint8이 아닌 uint를 반환하기 때문에 에러
uint8 c = a * uint8(b); // 256비트 => 8비트 형 변환
```

``` javascript
pragma solidity ^0.4.19;

contract ZombieFactory {

    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;

    struct Zombie {
        string name;
        uint dna;
    }

    Zombie[] public zombies;

    function _createZombie(string _name, uint _dna) private {
        zombies.push(Zombie(_name, _dna));
    } // 좀비 생성하여 좀비군대 배열에 push하는 함수

    function _generateRandomDna(string _str) private view returns (uint) {
        uint rand = uint(keccak256(_str));
        return rand % dnaModulus;
    } // string으로 부터 랜덤 DNA를 생성하는 함수

    function createRandomZombie(string _name) public {
        uint randDna = _generateRandomDna(_name);
        _createZombie(_name, randDna);
    } // 좀비의 이름을 입력값으로 받아 랜덤 DNA를 가진 좀비를 만드는 함수

}
```
### 이벤트
컨트랙트가 블록체인 상의 앱 사용자 단에서 어떤 액션이 발생했을 때 의사소통하는 방법이다. 컨트랙트는 특정 이벤트가 일어나는 지 귀 기울이고, 그 이벤트가 발생하면 행동을 취한다.

``` javascript
pragma solidity ^0.4.19;

contract ZombieFactory {

    event NewZombie(uint zombieId, string name, uint dna);  
    // 좀비가 생성될 때 마다 앱의 사용자 단에서 이를 알고, 표시하도록하는 이벤트

    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;

    struct Zombie {
        string name;
        uint dna;
    }

    Zombie[] public zombies;

    function _createZombie(string _name, uint _dna) private {
        uint id = zombies.push(Zombie(_name, _dna)) - 1;
        NewZombie(id, _name, _dna); // 새로운 좀비가 배열에 추가된 후에 이벤트 실행
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

### web.js 나중에 공부하기~!

``` solidity ex1
contract testContract{

}
```
