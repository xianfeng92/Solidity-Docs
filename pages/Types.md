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

函数类型是一种表示函数的类型。可以将一个函数赋值给另一个函数类型的变量，也可以将一个函数作为参数进行传递，还能在函数调用中返回函数类型变量。函数类型有两类：内部（internal）函数和外部（external） 函数：

内部函数只能在当前合约内被调用（更具体来说，其包括内部库函数和所继承的函数），因为它们不能在当前合约上下文的外部被执行。调用一个内部函数是通过跳转到它的入口标签来实现的，就像在当前合约的内部调用一个函数。

外部函数是通过其地址和函数签名来调用。

函数类型表示成如下的形式：

> function (<parameter types>) {internal|external} [pure|constant|view|payable] [returns (<return types>)]


与参数类型相反，__返回类型不能为空__ —— 如果函数类型不需要返回，则需要删除整个 returns (<return types>) 部分。


函数类型默认是 internal。 而合约中的函数本身默认是 public 的，只有当它被当做类型名称时，默认才是内部函数。

函数声明时默认为public类型，和显示声明为public类型的函数一样，都可供外部访问。

有两种方法可以访问当前合约中的函数：一种是直接使用它的名字，f ，另一种是使用 this.f 。 前者适用于内部函数，后者适用于外部函数。

如果当函数类型的变量还没有初始化时就调用它的话会引发一个异常。如果在一个函数被 delete 之后调用它也会发生相同的情况。

如果外部函数类型在 Solidity 的上下文环境以外的地方使用，它们会被视为 function 类型。 该类型将函数地址紧跟其函数标识一起编码为一个 bytes24 类型。。

请注意，当前合约的 public 函数既可以被当作内部函数也可以被当作外部函数使用。 如果想将一个函数当作内部函数使用，就用 f 调用，如果想将其当作外部函数，使用 this.f 。

> Note 关于　internal，external，public,private

1. 调用方式

* internal

internal调用，实现时转为简单的EVM跳转，所以它能直接使用上下文环境中的数据，对于引用传递时将会变得非常高效（不用拷贝数据）。

* external

external调用，实现为合约的外部消息调用。所以在合约初始化时不能external的方式调用自身函数，因为合约还未初始化完成。external调用时，实际是向目标合约发送一个消息调用。消息中的函数定义部分是一个24字节大小的消息体，20字节为地址，4字节为函数签名。

2. 函数的可见性

* external

声明为external的可以从其它合约或通过Transaction进行调用，所以声明为external的函数是合约对外接口的一部分。__不能以internal的方式进行调用__,你在一个合约中申明一个external函数f时，你不能在本合约中采用internal方式调用，只能this.f。

* public

函数默认声明为public,public的函数既允许以internal的方式调用，也允许以external的方式调用。


* internal

在当前的合约或继承的合约中，只允许以internal的方式调用。

* private

只能在当前合约中被访问（不可在被继承的合约中访问）。


[参考](https://blog.csdn.net/aaa19890808/article/details/79343036)


如果使用内部函数类型的例子:

```

pragma solidity ^0.4.16;

library ArrayUtils {
  // internal functions can be used in internal library functions because
  // they will be part of the same code context
  function map(uint[] memory self, function (uint) pure returns (uint) f)
    internal
    pure
    returns (uint[] memory r)
  {
    r = new uint[](self.length);
    for (uint i = 0; i < self.length; i++) {
      r[i] = f(self[i]);
    }
  }
  function reduce(
    uint[] memory self,
    function (uint, uint) pure returns (uint) f
  )
    internal
    pure
    returns (uint r)
  {
    r = self[0];
    for (uint i = 1; i < self.length; i++) {
      r = f(r, self[i]);
    }
  }
  function range(uint length) internal pure returns (uint[] memory r) {
    r = new uint[](length);
    for (uint i = 0; i < r.length; i++) {
      r[i] = i;
    }
  }
}

contract Pyramid {
  using ArrayUtils for *;
  function pyramid(uint l) public pure returns (uint) {
    return ArrayUtils.range(l).map(square).reduce(sum);
  }
  function square(uint x) internal pure returns (uint) {
    return x * x;
  }
  function sum(uint x, uint y) internal pure returns (uint) {
    return x + y;
  }
}

```

另外一个使用外部函数类型的例子:

```
pragma solidity ^0.4.11;

contract Oracle {
  struct Request {
    bytes data;
    function(bytes memory) external callback;
  }
  Request[] requests;
  event NewRequest(uint);
  function query(bytes data, function(bytes memory) external callback) public {
    requests.push(Request(data, callback));
    NewRequest(requests.length - 1);
  }
  function reply(uint requestID, bytes response) public {
    // 这里要验证 reply 来自可信的源
    requests[requestID].callback(response);
  }
}

contract OracleUser {
  Oracle constant oracle = Oracle(0x1234567); // 已知的合约
  function buySomething() {
    oracle.query("USD", this.oracleResponse);
  }
  function oracleResponse(bytes response) public {
    require(msg.sender == address(oracle));
    // 使用数据
  }
}
```

## Reference Types

在处理复杂的类型（即占用的空间超过 256 位的类型）时，我们需要更加谨慎。由于拷贝这些类型变量需要很大的开销，我们不得不考虑它的存储位置，是将它们保存在 ** memory **中， 还是 **storage **中。

### Data location

每个复杂类型，如数组和结构，都需要指定一个额外属性，即“数据位置”，关于它是存储在memory中还是storage中。 根据上下文，数据存储的位置始终存在默认值，但可以通过向类型附加关键字memory或storage来修改。 函数参数（包括返回参数）的默认值是memory，局部变量的默认值是storage，显然状态变量的数据位置是storage。

还有一种数据位置，calldata，这是一块只读的，且不会永久存储的位置，用来存储函数参数。外部函数的参数（非返回参数）的数据位置被强制指定为 calldata，效果跟memory很像。

数据位置的指定是非常重要，因为它们影响着赋值行为： 在storage 和　memory 之间两两赋值，或storage 向状态变量（甚至是从其它状态变量）赋值都会创建一份独立的拷贝。然而状态变量向局部变量赋值时仅仅传递一个引用，而且这个引用总是指向状态变量，因此后者改变的同时前者也会发生改变。另一方面，从一个memory 存储的引用类型向另一个　memory 存储的引用类型赋值并不会创建拷贝。

```
pragma solidity ^0.4.0;

contract C {
    uint[] x; // x 的数据存储位置是 storage

    // memoryArray 的数据存储位置是 memory
    function f(uint[] memoryArray) public {
        x = memoryArray; // 将整个数组拷贝到 storage 中，可行
        var y = x;  // 分配一个指针（其中 y 的数据存储位置是 storage），可行
        y[7]; // 返回第 8 个元素，可行
        y.length = 2; // 通过 y 修改 x，可行
        delete x; // 清除数组，同时修改 y，可行
        // 下面的就不可行了；需要在 storage 中创建新的未命名的临时数组， /
        // 但 storage 是“静态”分配的：
        // y = memoryArray;
        // 下面这一行也不可行，因为这会“重置”指针，
        // 但并没有可以让它指向的合适的存储位置。
        // delete y;

        g(x); // 调用 g 函数，同时移交对 x 的引用
        h(x); // 调用 h 函数，同时在 memory 中创建一个独立的临时拷贝
    }

    function g(uint[] storage storageArray) internal {}
    function h(uint[] memoryArray) public {}
}

```

### 总结

#### 强制指定的数据位置：

* 外部函数的参数（不包括返回参数）： calldata

* 状态变量： storage

#### 默认数据位置：

* 函数参数（包括返回参数）： memory

* 所有其它局部变量： storage


## Arrays

数组我们可以在声明时指定其长度，也可以动态调整大小。 对于　storage 的数组来说，元素类型可以是任意的（即元素也可以是数组类型，映射类型或者结构体）。 对于 memory 的数组来说，元素类型不能是映射类型，如果作为 public 函数的参数，它只能是 ABI 类型。

一个元素类型为 T，固定长度为 k 的数组可以声明为 T[k]，而动态数组声明为 T[]。 举个例子，一个长度为 5，元素类型为 uint 的动态数组的数组，应声明为 uint[][5]。 要访问第三个动态数组的第二个元素，你应该使用 x[2][1]。

bytes 和 string 类型的变量是特殊的数组。 bytes 类似于 byte[]，但它在 calldata 中会被“紧打包”（将元素连续地存在一起，不会按每 32 字节一单元的方式来存放）。 string 与 bytes 相同，但（暂时）不允许用长度或索引来访问。

可以将数组标识为 public，从而让 Solidity 创建一个 getter。 之后必须使用数字下标作为参数来访问 getter。


## Allocating Memory Arrays


可使用 new 关键字在内存中创建变长数组。与storage 数组不同，你不能通过修改成员变量 .length 改变内存memory数组的大小。

```
pragma solidity ^0.4.16;

contract C {
    function f(uint len) public pure {
        uint[] memory a = new uint[](7);
        bytes memory b = new bytes(len);
        // 这里我们有 a.length == 7 以及 b.length == len
        a[6] = 8;
    }
}
```


## Array Literals / Inline Arrays

数组字面是作为表达式编写的数组，不会立即分配给变量。

```
pragma solidity ^0.4.16;

contract C {
    function f() public pure {
        g([uint(1), 2, 3]);
    }
    function g(uint[3] _data) public pure {
        // ...
    }
}
```

数组字面常数是一种定长的 memory 数组类型，它的基础类型由其中元素的类型决定。例如，[1, 2, 3] 的类型是 uint8[3] memory，因为其中的每个字面常数的类型都是 uint8。 正因为如此，有必要将上面这个例子中的第一个元素转换成 uint 类型。 目前需要注意的是，定长的 memory 数组并不能赋值给变长的 memory 数组，下面是个反例：

```
// 这段代码并不能编译。

pragma solidity ^0.4.0;

contract C {
    function f() public {
        // 这一行引发了一个类型错误，因为 unint[3] memory
        // 不能转换成 uint[] memory。
        uint[] x = [uint(1), 3, 4];
    }
}

```


## Members

* length

数组中的 length 成员变量表示其长度。动态数组如果是　storage 的，可以通过改变成员变量 .length 改变数组大小。并不能通过访问超出当前数组长度的方式实现自动扩展数组的长度。一经创建，memory 数组的大小就是固定的（但却是动态的，也就是说，它依赖于运行时的参数）。


* push

变长的storage 数组以及 bytes 类型（而不是 string 类型）都有一个叫做 push 的成员函数，它用来附加新的元素到数组末尾。这个函数将返回新的数组长度。

```
pragma solidity ^0.4.16;

contract ArrayContract {
    uint[2**20] m_aLotOfIntegers;
    // 注意下面的代码并不是一对动态数组，
    // 而是一个数组元素为一对变量的动态数组（也就是数组元素为长度为 2 的定长数组的动态数组）。
    bool[2][] m_pairsOfFlags;
    // newPairs 存储在 memory 中 —— 函数参数默认的存储位置

    function setAllFlagPairs(bool[2][] newPairs) public {
        // 向一个 storage 的数组赋值会替代整个数组
        m_pairsOfFlags = newPairs;
    }

    function setFlagPair(uint index, bool flagA, bool flagB) public {
        // 访问一个不存在的数组下标会引发一个异常
        m_pairsOfFlags[index][0] = flagA;
        m_pairsOfFlags[index][1] = flagB;
    }

    function changeFlagArraySize(uint newSize) public {
        // 如果 newSize 更小，那么超出的元素会被清除
        m_pairsOfFlags.length = newSize;
    }

    function clear() public {
        // 这些代码会将数组全部清空
        delete m_pairsOfFlags;
        delete m_aLotOfIntegers;
        // 这里也是实现同样的功能
        m_pairsOfFlags.length = 0;
    }

    bytes m_byteData;

    function byteArrays(bytes data) public {
        // 字节的数组（语言意义中的 byte 的复数 ``bytes``）不一样，因为它们不是填充式存储的，
        // 但可以当作和 "uint8[]" 一样对待
        m_byteData = data;
        m_byteData.length += 7;
        m_byteData[3] = byte(8);
        delete m_byteData[2];
    }

    function addFlag(bool[2] flag) public returns (uint) {
        return m_pairsOfFlags.push(flag);
    }

    function createMemoryArray(uint size) public pure returns (bytes) {
        // 使用 `new` 创建动态 memory 数组：
        uint[2][] memory arrayOfPairs = new uint[2][](size);
        // 创建一个动态字节数组：
        bytes memory b = new bytes(200);
        for (uint i = 0; i < b.length; i++)
            b[i] = byte(i);
        return b;
    }
}

```


## Structs

Solidity 支持通过构造结构体的形式定义新的类型，以下是一个结构体使用的示例：

```

pragma solidity ^0.4.11;

contract CrowdFunding {
    // 定义的新类型包含两个属性。
    struct Funder {
        address addr;
        uint amount;
    }

    struct Campaign {
        address beneficiary;
        uint fundingGoal;
        uint numFunders;
        uint amount;
        mapping (uint => Funder) funders;
    }

    uint numCampaigns;
    mapping (uint => Campaign) campaigns;

    function newCampaign(address beneficiary, uint goal) public returns (uint campaignID) {
        campaignID = numCampaigns++; // campaignID 作为一个变量返回
        // 创建新的结构体示例，存储在 storage 中。我们先不关注映射类型。
        campaigns[campaignID] = Campaign(beneficiary, goal, 0, 0);
    }

    function contribute(uint campaignID) public payable {
        Campaign storage c = campaigns[campaignID];
        // 以给定的值初始化，创建一个新的临时 memory 结构体，
        // 并将其拷贝到 storage 中。
        // 注意你也可以使用 Funder(msg.sender, msg.value) 来初始化。
        c.funders[c.numFunders++] = Funder({addr: msg.sender, amount: msg.value});
        c.amount += msg.value;
    }

    function checkGoalReached(uint campaignID) public returns (bool reached) {
        Campaign storage c = campaigns[campaignID];
        if (c.amount < c.fundingGoal)
            return false;
        uint amount = c.amount;
        c.amount = 0;
        c.beneficiary.transfer(amount);
        return true;
    }
}

```

上面的合约只是一个简化版的众筹合约，但它已经足以让我们理解结构体的基础概念。 结构体类型可以作为元素用在映射和数组中，其自身也可以包含映射和数组作为成员变量。

尽管结构体本身可以作为映射的值类型成员，但它并不能包含自身。 这个限制是有必要的，因为结构体的大小必须是有限的。


## Mappings

映射类型在声明时的形式为 mapping(_KeyType => _ValueType)。 其中 _KeyType 可以是__除了映射、变长数组、合约、枚举以及结构体__以外的几乎所有类型。 _ValueType 可以是包括映射类型在内的任何类型。


映射可以视作哈希表，它们在初始化过程中创建每个可能的 key，并将其映射到字节形式全是零的值：一个类型的默认值。然而：在映射中，实际上并不存储 key，而是存储它的 keccak256 哈希值，从而便于查询实际的值。

正因为如此，映射是没有长度的，也没有 key 的集合或 value 的集合的概念。

只有状态变量（或者在 internal 函数中的对于存储变量的引用）可以使用映射类型。

可以将映射声明为 public，然后来让 Solidity 创建一个 getter。 _KeyType 将成为 getter的参数，并且 getter 会返回 _ValueType。

_ValueType 也可以是一个映射。这时在使用 getter 时需要递归地传入每个 _KeyType 参数。

```
pragma solidity ^0.4.0;

contract MappingExample {
    mapping(address => uint) public balances;

    function update(uint newBalance) public {
        balances[msg.sender] = newBalance;
    }
}

contract MappingUser {
    function f() public returns (uint) {
        MappingExample m = new MappingExample();
        m.update(100);
        return m.balances(this);
    }
}

```

## delete

delete a 的结果是__将 a 的类型在初始化时的值赋值给 a__。即对于整型变量来说，相当于 a = 0， 但 delete 也适用于数组，__对于动态数组来说，是将数组的长度设为 0，而对于静态数组来说，是将数组中的所有元素重置__。如果对象是结构体，则将结构体中的所有属性重置。

__delete 对整个映射是无效的（因为映射的键可以是任意的，通常也是未知的）__。 因此在你删除一个结构体时，结果将重置所有的非映射属性，这个过程是递归进行的，除非它们是映射。 然而，单个的键及其映射的值是可以被删除的。

理解 delete a 的效果就像是给 a 赋值很重要，换句话说，这相当于在 a 中存储了一个新的对象。

```
pragma solidity ^0.4.0;

contract DeleteExample {
    uint data;
    uint[] dataArray;

    function f() public {
        uint x = data;
        delete x; // 将 x 设为 0，并不影响数据
        delete data; // 将 data 设为 0，并不影响 x，因为它仍然有个副本
        uint[] storage y = dataArray;
        delete dataArray;
        // 将 dataArray.length 设为 0，但由于 uint[] 是一个复杂的对象，y 也将受到影响，
        // 因为它是一个存储位置是 storage 的对象的别名。
        // 另一方面："delete y" 是非法的，引用了 storage 对象的局部变量只能由已有的 storage 对象赋值。
    }
}

```

## Conversions between Elementary Types

### Implicit Conversions

如果一个运算符用在两个不同类型的变量之间，那么编译器将隐式地将其中一个类型转换为另一个类型（不同类型之间的赋值也是一样）。 一般来说，只要值类型之间的转换在语义上行得通，而且转换的过程中没有信息丢失，那么隐式转换基本都是可以实现的： uint8 可以转换成 uint16，int128 转换成 int256，但 int8 不能转换成 uint256 （因为 uint256 不能涵盖某些值，例如，-1）。 更进一步来说，无符号整型可以转换成跟它大小相等或更大的字节类型，但反之不能。 任何可以转换成 uint160 的类型都可以转换成 address 类型。

### Explicit Conversions

如果某些情况下编译器不支持隐式转换，但是你很清楚你要做什么，这种情况可以考虑显式转换。注意这可能会发生一些无法预料的后果，因此一定要进行测试，确保结果是你想要的！ 下面的示例是将一个 int8 类型的负数转换成 uint：

> int8 y = -3;
> uint x = uint(y);

这段代码的最后，x 的值将是 0xfffff..fd （64 个 16 进制字符），因为这是 -3 的 256 位补码形式。

### Type Deduction

为了方便起见，没有必要每次都精确指定一个变量的类型，__编译器会根据分配给该变量的第一个表达式的类型自动推断该变量的类型__。

> uint24 x = 0x123;
> var y = x;


这里 y 的类型将是 uint24。__不能对函数参数或者返回参数使用 var。__

































































