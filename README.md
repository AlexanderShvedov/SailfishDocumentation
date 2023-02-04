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

<img src="./graphsPictures/callgrapth.png" width="100%">