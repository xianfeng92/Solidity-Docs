#　Types

Solidity是一种静态类型语言，这意味着每个变量（状态变量和局部变量）都需要在编译时指定变量的类型（或可以根据上下文推导出变量类型）。Solidity 提供了几种基本类型，可以用来组合出复杂类型。

## Value Types

以下类型我们可以称为值类型，因为这些类型的变量将始终__按值__来传递。 也就是说，__当这些变量被用作函数参数或者用在赋值语句中时，总会进行值拷贝__。

### Booleans

bool：可能的取值为true和false。

运算符：

* !（逻辑非）

* &&（逻辑与，"and"）

* ||（逻辑或， "or"）

* == （等于）

* != （不等于）

运算符||和&&都遵循同样的短路（short-circuiting）规则。就是说在表达式 f(x) || g(y) 中， 如果 f(x) 的值为 true ，那么 g(y) 就不会被执行，即使会出现一些副作用。


## Integers

int/uint:分别表示有符号和无符号的不同位数的整型变量。支持关键字 uint8 到 uint256（无符号，从8位到256位）以及int8到int256，以8位为步长递增。uint 和 int 默认情况下分别是 uint256 和 int256。

运算符：

* 比较运算符： <= ， < ， == ， != ， >= ， > （返回布尔值）

* 位运算符： & ， | ， ^ （异或）， ~ （位取反）

* 算数运算符： + ， - ， 一元运算 - ， 一元运算 + ， * ， / ， % （取余） ， ** （幂）， << （左移位） ， >> （右移位）


除以零或者模零运算都会引发运行时异常。

移位运算的结果取决于运算符左边的类型。 表达式 x << y 与 x * 2**y 是等价的， x >> y 与 x / 2**y 是等价的。这意味对一个负数进行移位会导致其符号消失。按负数位移动会引发运行时异常。

## Address

address：地址类型存储一个 20 字节的值（以太坊地址的大小）。地址类型也有成员变量，并作为所有合约的基础。

Note:从 0.5.0 版本开始，合约不会从地址类型派生，但仍然可以显式地转换成地址类型。


## Members of Addresses

* balance 和 transfer

可以使用 balance 属性来查询一个地址的余额，也可以使用 transfer 函数向一个地址发送Ether（以 wei 为单位）:

```
address x = 0x123;
address myAddress = this;
if (x.balance < 10 && myAddress.balance >= 10) x.transfer(10);

```

如果 x 是一个合约地址，它的代码（更具体来说是　fallback 函数，如果有的话）会跟 transfer 函数调用一起执行（这是 EVM 的一个特性，无法阻止）。 如果在执行过程中用光了 gas 或者因为任何原因执行失败，以太币Ether交易会被打回，当前的合约也会由于异常而终止执行。


* send

send 是 transfer 的低级版本。__如果执行失败，当前的合约不会因为异常而终止，但 send 会返回 false__。


在使用 send 的时候会有些风险：如果调用栈深度是 1024 或者接收者用光了 gas 都会导致发送失败。所以为了保证以太币Ether转移的安全，一定要检查 send 的返回值。建议使用 transfer或者用一种接收者可以取回资金的模式。


* call, callcode and delegatecall

与那些没有附带ABI的合约交互时，可以使用接受任意类型和任意数量参数的 call 函数。call函数的__参数会被打包到以 32 字节为单位的连续区域中存放__。当第一个参数被编码成正好4个字节时，其后面不会填充后续参数编码，以允许使用函数签名。

```

address nameReg = 0x72ba7d8e73fe8eb666ea66babc8116a41bfb10e2;
nameReg.call("register", "MyName");
nameReg.call(bytes4(keccak256("fun(uint256)")), a);

```

使用 .gas() 修饰器来调整本次call调用可使用的gas数量

> namReg.call.gas(1000000)("register", "MyName");


也能控制提供的以太币Ether的值

> nameReg.call.value(1 ether)("register", "MyName");


这些修饰器可以联合使用,每个修改器出现的顺序不重要

> nameReg.call.gas(1000000).value(1 ether)("register", "MyName");


类似地，也可以使用 delegatecall：区别在于只使用给定地址的代码，其它属性（存储，余额，……）都取自当前合约。 delegatecall 的目的是使用存储在另外一个合约中的库代码。用户必须确保两个合约中的存储结构都适用于 delegatecall。在 homestead 版本之前，只有一个功能类似但作用有限的 callcode 的函数可用，但它不能获取委托方的 msg.sender 和 msg.value。不鼓励使用 callcode，在未来也会将其移除。

__所有合约都继承了地址（address）的成员变量，因此可以使用 this.balance 查询当前合约的余额__。

这三个函数都属于低级函数，需要谨慎使用。 具体来说，任何未知的合约都可能是恶意的。你在调用一个合约的同时就将控制权交给了它，它可以反过来调用你的合约， 因此，当调用返回时要为你的状态变量的改变做好准备。


## Fixed-size byte arrays

关键字有：bytes1， bytes2， bytes3， ...， bytes32。byte 是 bytes1 的别名。

成员变量：

* .length 表示这个字节数组的长度（只读）.


## Dynamically-sized byte array



## Address Literals


## Rational and Integer Literals


## String Literals


## Hexadecimal Literals


## Enums


## Function Types

函数类型是一种表示函数的类型。可以将一个函数赋值给另一个函数类型的变量，也可以将一个函数作为参数进行传递，还能在函数调用中返回函数类型变量。S函数类型有两类：内部（internal）函数和外部（external） 函数：

内部函数只能在当前合约内被调用（更具体来说，其包括内部库函数和所继承的函数），因为它们不能在当前合约上下文的外部被执行。调用一个内部函数是通过跳转到它的入口标签来实现的，就像在当前合约的内部调用一个函数。

外部函数是通过其地址和函数签名来调用。

函数类型表示成如下的形式：

> function (<parameter types>) {internal|external} [pure|constant|view|payable] [returns (<return types>)]


与参数类型相反，__返回类型不能为空__ —— 如果函数类型不需要返回，则需要删除整个 returns (<return types>) 部分。























