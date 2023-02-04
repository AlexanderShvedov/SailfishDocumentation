<h1 align="center">SailfishDocumentation</h1>


Все графы будут сгенерированы для следубщего котракта и функции withdrawBalance (если не приведён какой-то другой пример):

pragma solidity ^0.4.24;

contract Reentrancy {
    mapping (address => uint) userBalance;
   
    function getBalance(address u) view public returns(uint){
        return userBalance[u];
    }

    function addToBalance() payable public{
        userBalance[msg.sender] += msg.value;
    }   

    function withdrawBalance() public{
        // send userBalance[msg.sender] ethers to msg.sender
        // if mgs.sender is a contract, it will call its fallback function
        if( ! (msg.sender.call.value(userBalance[msg.sender])() ) ){
            revert();
        }
        userBalance[msg.sender] = 0;
    }   
}

## Начало

Идёт инициализация Slither. Из него берут информацию о функциях и переменных для построения зависимостей.

slither_obj = Slither(contract_path, solc=solc_path)

## Callgraph

Для каждой функции из каждого контракта добавляется вершина для функции в граф. В конце все вершины складываются в общий граф. Рёбер на этом этапе нет, функции на отдельные иструкции не раскладываются. 

(Замечание: есть ещё проверки на interanl и external call для функций, которые так же влияют на граф, но в моём примере эти проверки не прошли)

<img src="./graphsPictures/callgraph.png" width="100%">

## generate_icfg

Для начала генерируется граф cfg для каждой функции. На этом этапе функция разбивается на блоки с интсрукциями. 

<img src="./graphsPictures/withdrawBalance_cfg.png" width="100%">

Далее cfg используется для icfg. В данном примере разницы между ними нет.

<img src="./graphsPictures/withdrawBalance_icfg.png" width="100%">

Однако, если внурти одной функции есть вызов другой функции контракта (неважно, публичной или приватной), этот вызов разоюбъётся на отдельные блоки. Это можно видеть в данном примере:


pragma solidity ^0.4.21;
contract Foo {
    mapping (address => uint256) public balance;
    mapping (address => uint256) public randomValues;

    
    function functionWithPublicFunction(uint256 value) public payable {
       balance[msg.sender] += msg.value;
       publicFunction(value, msg.sender);
    }

    function functionWithPrivateFunction(uint256 value) public payable {
       balance[msg.sender] += msg.value;
       privateFunction(value, msg.sender);
    }

    function publicFunction(uint256 value, address to) public {
        privateFunction(value, msg.sender);
        randomValues[to] *= value;
    }

    function privateFunction(uint256 value, address to) private {
        randomValues[to] += value;
    }
}


Будут сгенерированы такие cfg и icfg:

<img src="./graphsPictures/functionWithPublicFunction_cfg.png" width="100%">
<img src="./graphsPictures/functionWithPublicFunction_icfg.png" width="100%">