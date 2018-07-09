## 单位和全局变量

### Ether Units

Ether 单位之间的换算就是在数字后边加上 wei、 finney、 szabo 或 ether 来实现的，其默认单位为 Wei。例如： 2 ether == 2000 finney 的逻辑判断值为 true。

### Time Units

时间单位默认为秒。在时间单位之间，数字后面带有 seconds、 minutes、 hours、 days、 weeks 和 years 的可以进行换算，基本换算关系如下：

* 1 == 1 seconds
* 1 minutes == 60 seconds
* 1 hours == 60 minutes
* 1 days == 24 hours
* 1 weeks == 7 days
* 1 years == 365 days

由于闰秒造成的每年不都是 365 天、每天不都是 24 小时 leap seconds，所以如果你要使用这些单位计算日期和时间，请注意这个问题。因为闰秒是无法预测的，所以需要借助外部的预言机（oracle，是一种链外数据服务）来对一个确定的日期代码库进行时间矫正。

## Special Variables and Functions

在全局命名空间中已经预设了一些特殊的变量和函数，他们主要用来提供关于区块链的信息和一些通用的工具函数。

###　Block and Transaction Properties


* block.blockhash(uint blockNumber) returns (bytes32): 指定区块的hash值，仅可用于最新的 256 个区块且不包括当前区块；而 blocks 从 0.4.22 版本开始已经不推荐使用，由 blockhash(uint blockNumber) 代替

* block.coinbase (address): 当前区块的旷工地址

* block.difficulty (uint): 当前区块的难度

* block.gaslimit (uint): 当前区块的 gaslimit

* block.number (uint): 当前区块号

* block.timestamp (uint): 自 unix epoch 起始当前区块以秒计的时间戳

* msg.data (bytes): 完整的 calldata　数据

* msg.gas (uint): 剩余 gas

* msg.sender (address): 消息发送者（当前调用）

* msg.sig (bytes4): calldata 的前 4 字节（也就是函数标识符）

* msg.value (uint): 随消息发送的 wei 的数量

* now (uint): 当前区块时间戳（block.timestamp）

* tx.gasprice (uint): 交易的 gas 价格

* tx.origin (address): 交易发起者（完全的调用链）


__Note:__

不要依赖 block.timestamp、 now 和 blockhash 产生随机数，除非你知道自己在做什么。

__时间戳和区块哈希在一定程度上都可能受到挖矿矿工影响。__例如，挖矿社区中的恶意矿工可以用某个给定的哈希来运行赌场合约的 payout 函数，而如果他们没收到钱，还可以用一个不同的哈希重新尝试。

当前区块的时间戳必须严格大于最后一个区块的时间戳，但这里唯一能确保的只是它会是在权威链上的两个连续区块的时间戳之间的数值。

## Error Handling

* assert(bool condition):

如果条件不满足，抛出异常 — 用于检查内部参数是否满足一定条件。

* require(bool condition):

如果条件不满足，抛出异常 - 用于检查由输入或者外部组件引起的错误。

* require(bool condition, string message):

如果条件不满足则撤销状态更改 - 用于检查由输入或者外部组件引起的错误，可以同时提供一个错误消息。

* revert():

终止运行并撤销状态更改。

* revert(string reason):

终止运行并撤销状态更改，可以同时提供一个解释性的字符串。


## Mathematical and Cryptographic Functions

* addmod(uint x, uint y, uint k) returns (uint):

  计算 (x + y) % k，加法会在任意精度下执行，并且加法的结果即使超过 2**256 也不会被截取。从 0.5.0 版本的编译器开始会加入对 k != 0 的校验（assert）。


* mulmod(uint x, uint y, uint k) returns (uint):

  计算 (x * y) % k，乘法会在任意精度下执行，并且乘法的结果即使超过 2**256 也不会被截取。从 0.5.0 版本的编译器开始会加入对 k != 0 的校验（assert）。

* keccak256(...) returns (bytes32):

  计算 (tightly packed) arguments 的 Ethereum-SHA-3 （Keccak-256）哈希。

* sha256(...) returns (bytes32):

  计算 (tightly packed) arguments 的 SHA-256 哈希。

* sha3(...) returns (bytes32):

  等价于 keccak256。


* ripemd160(...) returns (bytes20):
 
  计算 (tightly packed) arguments 的 RIPEMD-160 哈希。


* ecrecover(bytes32 hash, uint8 v, bytes32 r, bytes32 s) returns (address) ：

  利用椭圆曲线签名恢复与公钥相关的地址，错误返回零值。



上文中的“tightly packed”是指不会对参数值进行 padding 处理（就是说所有参数值的字节码是连续存放的），这意味着下边这些调用都是等价的：

* keccak256("ab", "c") 和　keccak256("abc")

请注意，__常量值会使用存储它们所需要的最少字节数进行打包__。例如：keccak256(0) == keccak256(uint8(0))，keccak256(0x12345678) == keccak256(uint32(0x12345678))。

在一个私链上，你很有可能碰到由于 sha256、ripemd160 或者 ecrecover 引起的 Out-of-Gas。这个原因就是他们被当做所谓的预编译合约而执行，并且在第一次收到消息后这些合约才真正存在（尽管合约代码是硬代码）。发送到不存在的合约的消息非常昂贵，所以实际的执行会导致 Out-of-Gas 错误。在你的合约中实际使用它们之前，给每个合约发送一点儿以太币，比如 1 Wei。这在官方网络或测试网络上不是问题。

## Address Related

* <address>.balance (uint256):

查看　address　的余额

* <address>.transfer(uint256 amount):

向 address 发送数量为 amount 的 Wei，失败时抛出异常，发送 2300 gas 的矿工费，不可调节。


* <address>.send(uint256 amount) returns (bool):

向 address 发送数量为 amount 的 Wei，失败时返回 false，发送 2300 gas 的矿工费用，不可调节。

* <address>.call(...) returns (bool):

issue low-level CALL, returns false on failure, forwards all available gas, adjustable

* <address>.callcode(...) returns (bool):

issue low-level CALLCODE, returns false on failure, forwards all available gas, adjustable

* <address>.delegatecall(...) returns (bool):

issue low-level DELEGATECALL, returns false on failure, forwards all available gas, adjustable


__NOte__:

使用 send 有很多危险：如果调用栈深度已经达到 1024，此时转账会失败；当如果接收者用光了 gas，转账同样会失败。为了保证以太币转账安全，我们需要检查 send 的返回值，利用 transfer 或者让接收者主动　withdraws 的模式会更好。


## Contract Related

* this (current contract's type):

  当前合约，可以明确转换为 address。

* selfdestruct(address recipient):

  销毁合约，并把余额发送到指定 address。

此外，当前合约内的所有函数都可以被直接调用，包括当前函数。















































