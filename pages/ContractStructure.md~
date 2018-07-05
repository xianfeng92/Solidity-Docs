# 合约的结构

在Solidity中，Contract类似于面向对象编程语言中的Object。每个Contract中可以包含  State Variables（状态变量）, Functions（函数）, Function Modifiers（函数修饰器）, Events（事件）, Struct Types（结构体）and Enum Types（枚举类型）。

## State Variables

状态变量是永久地存储在storage中的值, storage 所存储的值是会保存在区块中的。

```
pragma solidity ^0.4.11;

contract SimpleStorage{

     uint storedData; // State variable
     // ...

 }

```

## Functions

函数是Contract中可执行的代码单元。

```

pragma solidity ^0.4.0;

contract SimpleAuction {
    function bid() public payable { // Function
        // ...
    }
}

```

通过设置函数的可见性，我们可以在一个合约中调用另外一个合约的相关函数。


## Function Modifiers

函数修饰器可以用来以声明的方式改良函数语义,类似于python中的装饰器。

```
pragma solidity ^0.4.22;

contract Purchase {
    address public seller;

    modifier onlySeller() { // 修饰器
        require(
            msg.sender == seller,
            "Only seller can call this."
        );
        _;
    }

    function abort() public onlySeller { // Modifier usage
        // ...
    }
}
```

## Events

事件是与以太坊虚拟机(EVM)日志工具的接口。

```
pragma solidity ^0.4.21;
contract SimpleAuction {
    event HighestBidIncreased(address bidder, uint amount); // 事件

    function bid() public payable {
        // ...
        emit HighestBidIncreased(msg.sender, msg.value); // 触发事件
    }
}
```

## Struct Types

结构体是自定义类型，可以对多个变量进行分组。

```
pragma solidity ^0.4.0;

contract Ballot {
    struct Voter { // 结构
        uint weight;
        bool voted;
        address delegate;
        uint vote;
    }
}
```

## Enum Types

枚举可用来创建由一定数量的“常量值”构成的自定义类型。

```
pragma solidity ^0.4.0;

contract Purchase {
    enum State { Created, Locked, Inactive } // Enum
}
```











