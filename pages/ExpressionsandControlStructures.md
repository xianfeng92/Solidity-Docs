# Expressions and Control Structures

## Input Parameters and Output Parameters

与在Javascript中一样，函数可以作为输入的参数； 与Javascript和C不同，函数也可以返回任意数量的参数作为输出。

## Input Parameters

输入参数的声明方式与变量相同。有个例外，未使用的参数可以省略变量名称。例如，假设我们的合约接受带有两个整数参数的外部调用，函数如下：

```
pragma solidity ^0.4.16;

contract Simple {
    function taker(uint _a, uint _b) public pure {
        // do something with _a and _b.
    }
}

```

## Output Parameters

在 returns 关键字之后，可以用相同的语法声明输出参数。例如，我们希望返回两个结果：两个给定整数的和和乘积，可以这么写：

```
pragma solidity ^0.4.16;

contract Simple {
    function arithmetics(uint _a, uint _b)
        public
        pure
        returns (uint o_sum, uint o_product)
    {
        o_sum = _a + _b;
        o_product = _a * _b;
    }
}

```

可以省略输出参数的名称,输出值也可以使用　return 语句指定。return 语句还能够返回多个值。返回参数初始化为零；如果未显式设置，则它们保持为零。

输入参数和输出参数可以用作函数体中的表达式。在那里，它们也可以在赋值的左手边使用。

## Control Structures

除了　swich　和　goto, JavaScript　中的控制结构都可以在　Solidity　中使用。一般有：if, else, while, do, for, break, continue, return, ? :

注意，在C和JavaScript中没有从非布尔到布尔类型的类型转换，所以如果（1）{…}　在Solidity　是无效的。

## Returning Multiple Values

当函数具有多个输出参数时，返回（V0，V1，…，VN）可以返回多个值。组件的数量必须与输出参数的数目相同。


## Function Calls

### Internal Function Calls

当前合约的功能可以直接调用（“Internal”），也可以递归地调用，如在下面这个例子中所示：

```
pragma solidity ^0.4.16;

contract C {
    function g(uint a) public pure returns (uint ret) { return f(); }
    function f() internal pure returns (uint ret) { return g(7) + f(); }
}

```
这些函数调用被转换成EVM内的简单跳转。这样做的结果是当前的内存不被清除，即__传递内存引用到内部调用的函数是非常有效的__。只有同一合约的功能才能在内部调用。


### External Function Calls

this.g(8) 和　c.g(2)　（c 为一个合约实例）这些表达式也是一个有效的函数调用，这种函数调用是外部的（externally），是通过消息传递而不是简单的EVM跳转实现的。


请注意，关于 this 的调用不能在构造函数中使用，因为此时合约实例并未创建。其他合约的函数必须通过外部调用，对于外部函数的调用，所有的参数是复制到内存中的。

当调用其它合约的函数时，调用所发送的Wei的数量以及gas值都是可以指定的。

```
pragma solidity ^0.4.0;

contract InfoFeed {
    function info() public payable returns (uint ret) { return 42; }
}

contract Consumer {
    InfoFeed feed;
    function setFeed(address addr) public { feed = InfoFeed(addr); }
    function callFeed() public { feed.info.value(10).gas(800)(); }
}
```

info 函数必须要有 payable 修饰词。注意，表达式infoFEED（ADDR）执行显式类型转换，说明“我们知道给定地址的合约是InfoFEED”，这里不会执行构造函数。必须非常谨慎地处理显式类型转换。不要在你不确定其类型的合约中调用函数。

我们可以直接使用 setFeed(InfoFeed _feed) { feed = _feed; }。feed.info.value(10).gas(800) 仅（本地）设置与函数调用一起发送的value的值和gas的量，并且只有末端的圆括号执行实际调用。如果调用的合约不存在，或者被调用的合约本身抛出异常或没有gas了，此时函数的调用会导致异常。


任何与另一合约的相互作用都会带来潜在的危险，特别是如果合约的源代码事先不知道。当前的合同将控制权移交给另一个合约（该合约做任何事都是有可能的）。即使被调用的合约从已知的父合约继承，继承合约只需要具有正确的接口。然而，合约的实现却可以是完全随意的，因而构成危险。另外，注意其可能会调用你系统的其他合约，甚至在第一次调用返回之前就返回到所调用的合约。这意味着被调用的合约可以通过其功能改变调用的合约的状态变量。编写您的函数，例如，在你的合约中对状态变量进行任何更改之后，调用外部函数，这样您的合同就不易受到重用。


## Named Calls and Anonymous Function Parameters

函数调用参数可以以任何顺序给出，如果它们被包含在{}中，如下例所示。参数列表必须与函数声明中的参数列表重合，但可以按任意顺序排列。

```
pragma solidity ^0.4.0;

contract C {
    function f(uint key, uint value) public {
        // ...
    }

    function g() public {
        // named arguments
        f({value: 2, key: 3});
    }
}
```

## Omitted Function Parameter Names

可以省略未使用的参数（特别是返回参数）的名称。这些参数仍然存在于堆栈上，但它们是不可访问的。

```
pragma solidity ^0.4.16;

contract C {
    // omitted name for parameter
    function func(uint k, uint) public pure returns(uint) {
        return k;
    }
}

```

## Creating Contracts via new

可以使用new关键字创建合约实例，要创建的合约的完整代码必须事先知道，因此递归依赖创建是不可能的。

```
pragma solidity ^0.4.0;

contract D {
    uint x;
    function D(uint a) public payable {
        x = a;
    }
}

contract C {
    D d = new D(4); // will be executed as part of C's constructor

    function createD(uint arg) public {
        D newD = new D(arg);
    }

    function createAndEndowD(uint arg, uint amount) public payable {
        // Send ether along with the creation
        D newD = (new D).value(amount)(arg);
    }
}
```

正如在这个例子中所看到的，在使用.Value（）选项创建D的实例时，可以向其发生ether，但是不能限制 gas 量。如果创建失败（由于out-of-stack、余额不足或其他问题），则会引发异常。



## Order of Evaluation of Expressions

表达式的求值顺序是不确定的（更正式地说，未指定表达式树中一个节点的子节点的计算顺序，但它们当然在节点本身之前进行求值）。 只保证语句按顺序执行，并且完成布尔表达式的短路。



# Assignment

##　Destructuring Assignments and Returning Multiple Values


Solidity内部支持元组类型，即不同类型的对象列表，其大小在编译时是常量。这些元组可以用于同时返回多个值，并同时将它们分配给多个变量：

```
pragma solidity ^0.4.0;

contract C {
    uint[] data;

    function f() returns (uint, bool, uint) {
        return (7, true, 2);
    }

    function g() {
        // Declares and assigns the variables. Specifying the type explicitly is not possible.
        var (x, b, y) = f();
        // Assigns to a pre-existing variable.
        (x, y) = (2, 7);
        // Common trick to swap values -- does not work for non-value storage types.
        (x, y) = (y, x);
        // Components can be left out (also for variable declarations).
        // If the tuple ends in an empty component,
        // the rest of the values are discarded.
        (data.length,) = f(); // Sets the length to 7
        // The same can be done on the left side.
        (,data[3]) = f(); // Sets data[3] to 2
        // Components can only be left out at the left-hand-side of assignments, with
        // one exception:
        (x,) = (1,);
        // (1,) is the only way to specify a 1-component tuple, because (1) is
        // equivalent to 1.
    }
}
```

## Complications for Arrays and Structs

对于数组和结构等非值类型，赋值语义有点复杂。__赋值给状态变量总是创建一个独立的副本__。另一方面，赋值给局部变量只为基本类型创建独立的副本，如：32字节的静态类型。如果将结构或数组（包括　bytes　和　string）从状态变量分配给局部变量，则本地变量保存对原始状态变量的引用。对局部变量的第二次赋值只更改引用，不修改状态。对局部变量的成员（或元素）的赋值会改变其状态。


## Scoping and Declarations

声明的变量将具有初始默认值，其字节表示全为零。 变量的“默认值”是任何类型的典型“zero-state”。例如，bool的默认值为false。 uint或int类型的默认值为0.对于静态大小的数组和bytes1到bytes32，每个单独的元素将初始化为与其类型对应的默认值。最后，对于动态大小的数组，bytes 和　string，默认值是空数组或空字符串。

无论在函数内任何地方声明的变量其作用域都将是整个函数范围。这是因为Solidity从JavaScript继承了它的范围规则。这与许多语言形成对比，在这些语言中，变量仅在声明它们的范围内，直到语义块结束。 因此，以下代码是非法的并导致编译器抛出标识符已经声明（Identifier already declared）的错误：

```
// This will not compile

pragma solidity ^0.4.0;

contract ScopingErrors {
    function scoping() {
        uint i = 0;

        while (i++ < 1) {
            uint same1 = 0;
        }

        while (i++ < 2) {
            uint same1 = 0;// Illegal, second declaration of same1
        }
    }

    function minimalScoping() {
        {
            uint same2 = 0;
        }

        {
            uint same2 = 0;// Illegal, second declaration of same2
        }
    }

    function forLoopScoping() {
        for (uint same3 = 0; same3 < 1; same3++) {
        }

        for (uint same3 = 0; same3 < 1; same3++) {// Illegal, second declaration of same3
        }
    }
}

```

除此之外，如果声明了变量，它将在函数的开头初始化为其默认值。因此，以下代码是合法的，尽管写得不好：

```
function foo() returns (uint) {
    // baz is implicitly initialized as 0
    uint bar = 5;
    if (true) {
        bar += baz;
    } else {
        uint baz = 10;// never executes
    }
    return bar;// returns 5
}
```

## Error handling: Assert, Require, Revert and Exceptions

Solidity使用 state-reverting 异常来处理错误。这样的异常将撤消对当前调用（及其所有子调用）中的状态所做的所有更改，并且还向调用者标记错误。__函数assert和require可用于检查条件并在不满足条件时抛出异常__。 assert函数只应用于测试内部错误，并检查不变量。 require函数应用于确保满足有效条件（如输入或合约状态变量），或验证调用外部合约的返回值。如果使用得当，分析工具可以评估我们的合约，以确定将达到失败断言的条件和函数调用。正常运行的代码永远不会满足assert语句; 如果发生这种情况，即表明我们的合约中有bug。

还有两种触发异常的方法：revert 函数可以用来标记错误并恢复当前调用。在将来，还可能包含关于调用回复的错误的详细信息。抛出关键字也可以用作revert函数的替代。

当异常发生在子调用中时，它们会自动“冒泡”（即异常被重新抛出）。这个规则的例外是　send　和　the low-level functions call, delegatecall and callcode——在异常情况下返回false，而不是冒泡。

__Solidity 不支持异常的捕获__。


在以下示例中，我们可以看到如何使用require来轻松检查输入条件以及　assert　如何用于内部错误检查：

```
pragma solidity ^0.4.0;

contract Sharer {
    function sendHalf(address addr) payable returns (uint balance) {
        require(msg.value % 2 == 0); // Only allow even numbers
        uint balanceBeforeTransfer = this.balance;
        addr.transfer(msg.value / 2);
        // Since transfer throws an exception on failure and
        // cannot call back here, there should be no way for us to
        // still have half of the money.
        assert(this.balance == balanceBeforeTransfer - msg.value / 2);
        return this.balance;
    }
}

```

在以下情况下生成 assert-style 异常：

* 数组访问越界时

* 字节数组访问越界时

* 用０来做除数或取模时

* 左右做负数移位时

* 将过大的值或负值转换为枚举类型

* 调用内部函数类型的零初始化变量。

* assert 函数的参数表达式为false

在以下情况下生成 require-style 异常：

* 调用到throw

* 调用的　require　函数的参数表达式为false

* 如果通过消息调用去调用函数，但它没有完成正常调用（即，runs out of gas, has no matching function, or throws an exception itself），除了使用 a low level operation　call、send、delegatecall或callcode。The low level operations 从不抛出异常，而是通过返回false指示失败。

* 如果使用 new 关键字创建合约失败

* 如果执行一个外部函数调用，它指向一个不包含代码的合约

* 如果合约通过public函数接收 ether，而该函数没有　payable 修饰（包括　constructor　函数和　fallback　函数）

* 如果public 的getter 方法接收 ether

* 如果 a .transfer() 函数调用失败

在内部，Solidity 对 require-style 异常执行 revert 操作（指令0xFD）；对于 assert-style 异常会执行一个无效的操作（指令 0xfe）来抛出该异常。在这两种情况下，EVM会还原对状态所做出的所有更改。这么做的原因是程序的执行没有达到预期的效果，此时方法不应该继续执行下去。因为我们想保留交易的原子性，最安全的做法是恢复所有的改变，并使整个交易（或至少调用）没有效果。注意，assert-style 异常消耗所有可用的gas，而require-style 异常不会消耗任何 gas（从Meavigas发布开始）。





































