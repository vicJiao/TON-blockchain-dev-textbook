# TON 区块链开发教材 - 合约篇

> 本文档包含 TON 智能合约开发的核心内容，涵盖 Tolk/Tact 语言、设计模式和安全最佳实践。

---

## 第二篇：合约篇 —— 智能合约开发

---

## 第 5 章：Tolk 语言基础

**Tolk** 是 TON 区块链官方推荐的智能合约开发语言，由 TON 核心团队开发。它是 FunC 语言的现代化继任者，提供了更简洁的语法、更好的类型系统和更强的开发体验。

### 5.1 Tolk 语言概述

#### 5.1.1 Tolk 与 FunC 的关系

Tolk 是从 FunC 语言发展而来的新一代智能合约语言：

| 特性 | FunC | Tolk |
|------|------|------|
| 语法风格 | C 风格 | 类似 TypeScript/Rust |
| 类型系统 | 弱类型 | 强类型 |
| 类型推断 | 有限 | 强大 |
| 标准库 | 基础 | 丰富 |
| 开发体验 | 较复杂 | 现代化 |
| 官方支持 | 维护模式 | 积极开发 |

**为什么选择 Tolk？**

```
1. 官方推荐：TON 基金会主推的开发语言
2. 现代化语法：更易于学习和使用
3. 类型安全：编译时捕获更多错误
4. 工具链完善：更好的 IDE 支持和调试工具
5. 性能优化：生成的 TVM 字节码更高效
```

#### 5.1.2 Tolk 开发环境搭建

**安装 Tolk 编译器：**

```bash
# 通过 npm 安装（推荐）
npm install -g @ton/tolk

# 验证安装
tolk --version

# 编译合约
tolk compile contract.tolk -o contract.fc
```

**VS Code 插件：**

安装 "Tolk" 插件获得语法高亮、代码补全和错误检查。

**项目结构：**

```
my-tolk-project/
├── contracts/
│   ├── main.tolk          # 主合约
│   ├── imports/
│   │   └── stdlib.tolk    # 标准库
│   └── utils.tolk         # 工具函数
├── tests/
│   └── contract.spec.ts   # 测试文件
├── build/
│   └── contract.cell      # 编译输出
└── tolk.config.json       # 配置文件
```

**tolk.config.json 配置：**

```json
{
  "compilerVersion": "0.6.0",
  "entryPoints": ["contracts/main.tolk"],
  "outputDir": "build",
  "optimizationLevel": 2
}
```

---

### 5.2 Tolk 基础语法

#### 5.2.1 变量与数据类型

**基本数据类型：**

```tolk
// 整数（257 位有符号整数）
let a: int = 42;
let b: int = -100;
let c: int = 0xFF;        // 十六进制
let d: int = 1_000_000;   // 数字分隔符

// 布尔值（实际是整数 0 和 -1）
let flag: bool = true;
let isValid: bool = false;

// 切片（Slice）- 用于读取 Cell 数据
let slice: slice = cs;    // cs 是传入的消息体

// 构建器（Builder）- 用于构建 Cell
let builder: builder = beginCell();

// Cell - 数据单元
let cell: cell = beginCell().endCell();

// 地址
let addr: address = sender();
```

**类型推断：**

```tolk
// Tolk 支持类型推断
let x = 42;           // 推断为 int
let y = true;         // 推断为 bool
let z = beginCell();  // 推断为 builder

// 显式类型声明（推荐用于函数参数和返回值）
let explicit: int = 42;
```

**变量可变性：**

```tolk
// 不可变变量（默认）
let immutable = 10;
// immutable = 20;  // 错误！不可重新赋值

// 可变变量
var mutable = 10;
mutable = 20;         // 正确
```

#### 5.2.2 运算符与表达式

**算术运算符：**

```tolk
let a = 10;
let b = 3;

let sum = a + b;      // 13
let diff = a - b;     // 7
let prod = a * b;     // 30
let quot = a / b;     // 3（整数除法）
let rem = a % b;      // 1（取模）

// 复合赋值
var x = 10;
x += 5;               // x = 15
x -= 3;               // x = 12
x *= 2;               // x = 24
x /= 4;               // x = 6
```

**位运算符：**

```tolk
let a = 0b1100;       // 12
let b = 0b1010;       // 10

let and = a & b;      // 0b1000 = 8
let or = a | b;       // 0b1110 = 14
let xor = a ^ b;      // 0b0110 = 6
let not = ~a;         // 按位非
let left = a << 2;    // 左移
let right = a >> 1;   // 右移
```

**比较运算符：**

```tolk
let a = 10;
let b = 20;

let eq = a == b;      // false
let ne = a != b;      // true
let lt = a < b;       // true
let le = a <= b;      // true
let gt = a > b;       // false
let ge = a >= b;      // false
```

**逻辑运算符：**

```tolk
let a = true;
let b = false;

let and = a && b;     // false
let or = a || b;      // true
let not = !a;         // false

// 短路求值
let result = a && expensiveCall();  // 如果 a 为 false，不会调用 expensiveCall
```

#### 5.2.3 控制流

**条件语句：**

```tolk
// if-else
if (condition) {
    // 执行代码
} else if (anotherCondition) {
    // 执行代码
} else {
    // 执行代码
}

// 示例
fun checkValue(x: int): void {
    if (x > 100) {
        // 大于 100
    } else if (x > 50) {
        // 50-100
    } else {
        // 小于等于 50
    }
}
```

**循环：**

```tolk
// while 循环
var i = 0;
while (i < 10) {
    // 执行代码
    i += 1;
}

// repeat 循环（重复 n 次）
repeat (5) {
    // 执行 5 次
}

// until 循环（先执行后检查）
var x = 0;
until (x >= 10) {
    x += 1;
}
```

**match 表达式（模式匹配）：**

```tolk
// 匹配整数
let result = match (value) {
    1 => "one",
    2 => "two",
    3 => "three",
    _ => "other",     // 默认分支
};

// 匹配消息操作码
match (op) {
    0x12345678 => handleTransfer(),
    0x87654321 => handleDeposit(),
    _ => throw(0xffff),  // 未知操作码
}
```

---

### 5.3 函数定义与调用

#### 5.3.1 函数基础

**函数定义：**

```tolk
// 无返回值函数
fun greet(name: slice): void {
    // 函数体
}

// 有返回值函数
fun add(a: int, b: int): int {
    return a + b;
}

// 多参数函数
fun transfer(to: address, amount: int, payload: cell): void {
    // 转账逻辑
}
```

**函数调用：**

```tolk
// 直接调用
let sum = add(10, 20);

// 命名参数调用（更清晰）
transfer(to: recipient, amount: 1000, payload: emptyCell());

// 嵌套调用
let result = add(add(1, 2), add(3, 4));
```

**默认参数：**

```tolk
// 带默认值的参数
fun sendMessage(to: address, amount: int, mode: int = 0): void {
    // 如果 mode 未提供，默认为 0
}

// 调用时可以省略默认参数
sendMessage(addr, 1000);        // mode = 0
sendMessage(addr, 1000, 1);     // mode = 1
```

#### 5.3.2 特殊函数修饰符

**getter 函数（只读查询）：**

```tolk
// getter 函数可以被外部查询，不消耗 Gas
get fun balance(): int {
    return myBalance();
}

get fun getData(key: int): cell? {
    return storage.data.get(key);
}
```

**impure 函数（有副作用）：**

```tolk
// impure 函数会修改状态或发送消息
impure fun updateState(newValue: int): void {
    storage.value = newValue;  // 修改状态
}

impure fun sendFunds(to: address, amount: int): void {
    sendMessage(to, amount);   // 发送消息
}
```

**inline 函数（内联展开）：**

```tolk
// inline 函数在编译时展开，减少调用开销
inline fun max(a: int, b: int): int {
    return a > b ? a : b;
}

// 使用
let m = max(10, 20);  // 编译后相当于：let m = 10 > 20 ? 10 : 20;
```

#### 5.3.3 匿名函数与闭包

```tolk
// 匿名函数
let multiply = fun (a: int, b: int): int { return a * b; };
let result = multiply(3, 4);  // 12

// 作为参数传递
fun applyOperation(a: int, b: int, op: fun(int, int) -> int): int {
    return op(a, b);
}

let sum = applyOperation(10, 20, fun (x, y) { return x + y; });
```

---

### 5.4 复合数据类型

#### 5.4.1 元组（Tuple）

```tolk
// 创建元组
let point: [int, int] = [10, 20];
let person: [slice, int, address] = [name, age, addr];

// 访问元素
let x = point.0;      // 第一个元素
let y = point.1;      // 第二个元素

// 解构
let [name, age, addr] = person;

// 嵌套元组
let nested: [[int, int], [int, int]] = [[1, 2], [3, 4]];
```

#### 5.4.2 结构体（Struct）

```tolk
// 定义结构体
struct Point {
    x: int,
    y: int,
}

struct User {
    name: slice,
    balance: int,
    address: address,
}

// 创建实例
let p = Point { x: 10, y: 20 };
let user = User { name: "Alice", balance: 1000, address: sender() };

// 访问字段
let xCoord = p.x;
let userBalance = user.balance;

// 更新字段（创建新实例）
let newUser = User { ...user, balance: 2000 };
```

#### 5.4.3 枚举（Enum）

```tolk
// 定义枚举
enum MessageType {
    Transfer,
    Deposit,
    Withdraw,
}

// 带关联值的枚举
enum Message {
    Transfer { to: address, amount: int },
    Deposit { from: address, amount: int },
    Withdraw { amount: int },
}

// 使用枚举
let msg = Message.Transfer { to: recipient, amount: 100 };

// 模式匹配
match (msg) {
    Message.Transfer { to, amount } => {
        // 处理转账
    },
    Message.Deposit { from, amount } => {
        // 处理存款
    },
    Message.Withdraw { amount } => {
        // 处理提款
    },
}
```

#### 5.4.4 字典（Map）

```tolk
// 声明字典
let balances: map<address, int> = emptyMap();
let metadata: map<int, cell> = emptyMap();

// 设置值
balances.set(addr1, 1000);
balances.set(addr2, 2000);

// 获取值
let balance1 = balances.get(addr1);        // 返回 int?（可选类型）
let balance2 = balances.getOrDefault(addr2, 0);  // 返回 int，默认值 0

// 检查存在
let exists = balances.exists(addr1);       // true/false

// 删除
balances.del(addr1);

// 遍历
foreach (addr, balance in balances) {
    // 处理每个键值对
}
```

**可选类型（Optional）：**

```tolk
// 可选类型表示值可能存在也可能不存在
let maybeValue: int? = someValue;     // 可能有值
let noValue: int? = null;             // 无值

// 安全解包
if (maybeValue != null) {
    let value = maybeValue!!;         // !! 强制解包（确定非空时使用）
}

// 提供默认值
let value = maybeValue ?? 0;          // 如果为 null，使用 0
```

---

### 5.5 合约开发基础

#### 5.5.1 合约结构

```tolk
// 导入标准库
import "stdlib.tolk";

// 定义合约状态
contract MyContract {
    // 状态变量
    owner: address;
    balance: int;
    counter: int;
    
    // 初始化函数（部署时执行一次）
    init(owner: address) {
        self.owner = owner;
        self.balance = 0;
        self.counter = 0;
    }
    
    // 接收消息的函数
    receive(msg: TransferMessage) {
        // 处理转账消息
    }
    
    // 接收简单转账（无消息体）
    receive() {
        // 处理纯 TON 转账
    }
    
    // Getter 函数
    get fun getBalance(): int {
        return self.balance;
    }
    
    get fun getCounter(): int {
        return self.counter;
    }
}
```

#### 5.5.2 消息处理

**定义消息类型：**

```tolk
// 消息结构体
message TransferMessage {
    to: address,
    amount: int,
    payload: cell?,
}

message DepositMessage {
    from: address,
    amount: int,
}

// 在合约中处理
contract TokenContract {
    // ...
    
    receive(msg: TransferMessage) {
        // 验证发送者
        require(sender() == self.owner, "Unauthorized");
        
        // 执行转账逻辑
        sendMessage(msg.to, msg.amount, msg.payload);
    }
    
    receive(msg: DepositMessage) {
        // 更新余额
        self.balance += msg.amount;
    }
}
```

**消息操作码：**

```tolk
// 手动解析消息（用于与旧合约兼容）
receive(msg: slice) {
    let op = msg.loadInt(32);    // 加载 32 位操作码
    
    match (op) {
        0x12345678 => handleTransfer(msg),
        0x87654321 => handleDeposit(msg),
        _ => throw(0xffff),       // 未知操作码
    }
}

fun handleTransfer(msg: slice): void {
    let to = msg.loadAddress();
    let amount = msg.loadCoins();
    // ...
}
```

#### 5.5.3 发送消息

```tolk
// 发送简单消息
sendMessage(to: address, amount: int): void;

// 发送带参数的消息
sendMessage(to: address, amount: int, mode: int): void;

// 构建并发送复杂消息
fun sendComplexMessage(to: address, amount: int): void {
    let msg = beginCell()
        .storeInt(0x12345678, 32)      // 操作码
        .storeAddress(to)               // 目标地址
        .storeCoins(amount)             // 金额
        .storeRef(payloadCell)          // 引用附加数据
        .endCell();
    
    sendMessage(to, amount, 1, msg);   // mode = 1（单独支付 Gas）
}
```

**发送模式：**

```tolk
// 发送模式常量
const SEND_PAY_GAS_SEPARATELY = 1;     // 单独支付 Gas
const SEND_IGNORE_ERRORS = 2;          // 忽略发送错误
const SEND_DESTROY_ACCOUNT = 32;       // 发送后销毁账户
const SEND_CARRY_ALL_REMAINING = 64;   // 携带所有剩余余额
const SEND_CARRY_ALL_BALANCE = 128;    // 携带全部余额

// 常用组合
const SEND_MODE_DEFAULT = 0;
const SEND_MODE_PAY_GAS = 1;
const SEND_MODE_CARRY_ALL = 64;
const SEND_MODE_CARRY_ALL_PAY_GAS = 65;  // 64 + 1
```

---

### 5.6 标准库常用函数

#### 5.6.1 上下文函数

```tolk
// 获取当前消息上下文
let ctx = context();

// 常用上下文属性
let senderAddr = sender();              // 发送者地址
let myAddr = myAddress();               // 本合约地址
let currentBalance = myBalance();       // 当前余额（包括未处理的消息金额）
let msgValue = msgValue();              // 收到的消息金额
let now = now();                        // 当前区块时间戳
let blockLt = blockLt();                // 区块逻辑时间
let transLt = transLt();                // 交易逻辑时间
let randSeed = randomize();             // 随机种子
```

#### 5.6.2 Cell 操作

```tolk
// 创建 Builder
let b = beginCell();

// 存储数据
b = b.storeInt(value, bits);            // 存储有符号整数
b = b.storeUint(value, bits);           // 存储无符号整数
b = b.storeBool(value);                 // 存储布尔值
b = b.storeCoins(amount);               // 存储金额（变长编码）
b = b.storeAddress(addr);               // 存储地址
b = b.storeRef(cell);                   // 存储 Cell 引用

// 结束构建
let cell = b.endCell();

// 读取 Slice
let s = cell.beginParse();
let value = s.loadInt(32);              // 加载 32 位整数
let addr = s.loadAddress();             // 加载地址
let coins = s.loadCoins();              // 加载金额
let ref = s.loadRef();                  // 加载引用
let remaining = s.remainingBits();      // 剩余位数
```

#### 5.6.3 数学函数

```tolk
// 数学运算
let abs = math::abs(x);                 // 绝对值
let min = math::min(a, b);              // 最小值
let max = math::max(a, b);              // 最大值
let pow = math::pow(base, exp);         // 幂运算
let sqrt = math::sqrt(x);               // 平方根
let log2 = math::log2(x);               // 以 2 为底的对数

// 位操作
let bitCount = math::bitCount(x);       // 1 的位数
let bitLength = math::bitLength(x);     // 最高位位置
```

#### 5.6.4 异常处理

```tolk
// 抛出异常
throw(code);                            // 抛出指定错误码
require(condition, "Error message");    // 条件不满足时抛出

// 常用错误码
const ERROR_INVALID_ADDRESS = 100;
const ERROR_INSUFFICIENT_BALANCE = 101;
const ERROR_UNAUTHORIZED = 102;
const ERROR_INVALID_MESSAGE = 103;

// 使用示例
require(sender() == self.owner, ERROR_UNAUTHORIZED);
require(amount > 0, ERROR_INVALID_MESSAGE);
require(balance >= amount, ERROR_INSUFFICIENT_BALANCE);
```

---

### 5.7 完整示例：简单钱包合约

```tolk
import "stdlib.tolk";

// 消息定义
message TransferRequest {
    to: address,
    amount: int,
}

// 合约定义
contract SimpleWallet {
    // 状态
    owner: address;
    
    // 初始化
    init(owner: address) {
        self.owner = owner;
    }
    
    // 接收转账请求
    receive(msg: TransferRequest) {
        // 验证所有者
        require(sender() == self.owner, "Only owner can transfer");
        
        // 验证余额
        require(myBalance() >= msg.amount, "Insufficient balance");
        
        // 发送转账
        sendMessage(msg.to, msg.amount, SEND_MODE_PAY_GAS);
    }
    
    // 接收存款（任何人都可以存款）
    receive() {
        // 自动接受 TON 转账
    }
    
    // Getter 函数
    get fun getBalance(): int {
        return myBalance();
    }
    
    get fun getOwner(): address {
        return self.owner;
    }
}
```

---

**本章小结：**

本章介绍了 Tolk 语言的基础知识，包括：
- Tolk 语言概述和开发环境搭建
- 基础语法（变量、运算符、控制流）
- 函数定义与调用（包括 getter、impure、inline 等特殊修饰符）
- 复合数据类型（元组、结构体、枚举、字典）
- 合约开发基础（合约结构、消息处理、发送消息）
- 标准库常用函数

Tolk 作为 TON 官方推荐的智能合约语言，提供了现代化的语法和强大的类型系统，是开发 TON 智能合约的首选语言。下一章我们将学习另一种流行的合约语言 —— Tact。

---

## 第 6 章：Tact 语言基础

**Tact** 是专为 TON 区块链设计的高级智能合约编程语言，由 TON 社区开发。它以 TypeScript 风格的语法、强大的类型系统和优秀的开发者体验而闻名，是目前 TON 生态中最流行的智能合约语言之一。

### 6.1 Tact 语言概述

#### 6.1.1 Tact 的特点与优势

Tact 语言设计目标是让智能合约开发更加简单、安全和高效：

| 特性 | 说明 |
|------|------|
| **TypeScript 风格** | 熟悉的语法，降低学习成本 |
| **强类型系统** | 编译时捕获类型错误 |
| **面向合约设计** | 原生支持合约、消息、状态等概念 |
| **自动序列化** | 消息和状态的序列化/反序列化自动处理 |
| **内置安全机制** | 防止常见的合约漏洞 |
| **丰富标准库** | 提供常用功能的现成实现 |
| **优秀工具链** | 完整的测试、部署工具支持 |

**Tact vs Tolk：**

| 对比项 | Tact | Tolk |
|--------|------|------|
| 开发者 | 社区驱动 | 官方团队 |
| 语法风格 | TypeScript | Rust/TypeScript |
| 成熟度 | 非常成熟 | 较新 |
| 生态工具 | 非常丰富 | 正在完善 |
| 学习曲线 | 平缓 | 中等 |
| 推荐场景 | 快速开发、团队项目 | 性能敏感、底层开发 |

#### 6.1.2 开发环境搭建

**安装 Tact 编译器：**

```bash
# 通过 npm 安装
npm install -g @tact-lang/compiler

# 验证安装
tact --version

# 创建新项目
npm create tact@latest my-project
cd my-project
npm install
```

**VS Code 支持：**

安装 "Tact" 插件获得完整的语言支持：
- 语法高亮
- 代码补全
- 错误检查
- 代码导航

**项目结构：**

```
my-tact-project/
├── contracts/
│   ├── contract.tact      # 主合约文件
│   └── imports/
│       └── stdlib.tact    # 标准库
├── tests/
│   └── contract.spec.ts   # 测试文件
├── scripts/
│   └── deploy.ts          # 部署脚本
├── build/
│   └── contract/          # 编译输出
│       ├── Contract.compiled.json
│       └── Contract.abi.json
├── tact.config.json       # Tact 配置
└── package.json
```

**tact.config.json 配置：**

```json
{
  "projects": [
    {
      "name": "my-contract",
      "path": "./contracts/contract.tact",
      "output": "./build",
      "options": {
        "debug": false,
        "masterchain": false,
        "external": false
      }
    }
  ]
}
```

---

### 6.2 Tact 基础语法

#### 6.2.1 变量与类型

**基本类型：**

```tact
// 整数类型
let a: Int = 42;
let b: Int = -100;
let c: Int = 0xFF;        // 十六进制
let d: Int = ton("1.5");  // TON 金额（nanoTON）

// 布尔类型
let flag: Bool = true;
let isValid: Bool = false;

// 地址类型
let addr: Address = address("EQD...");
let senderAddr: Address = sender();

// Cell 类型
let cell: Cell = emptyCell();
let builder: Builder = beginCell();
let slice: Slice = cell.beginParse();

// 字符串（实际是 Slice）
let name: String = "Hello Tact";

// 字节数组
let data: Bytes = "binary data".asBytes();
```

**整数类型详解：**

```tact
// Int 是 257 位有符号整数
// 可以指定位大小（用于序列化）
let small: Int as uint8 = 255;     // 8 位无符号
let medium: Int as int32 = -1000;  // 32 位有符号
let large: Int as uint256 = 0;     // 256 位无符号

// 常用整数类型别名
type Coins = Int as coins;         // 变长编码的金额
type Uint32 = Int as uint32;
type Int32 = Int as int32;
```

**常量定义：**

```tact
// 全局常量
const MAX_SUPPLY: Int = 1000000;
const DECIMALS: Int = 9;
const TOKEN_NAME: String = "MyToken";

// 在合约中使用
contract Token {
    supply: Int = MAX_SUPPLY;
}
```

#### 6.2.2 运算符

**算术运算符：**

```tact
let a: Int = 10;
let b: Int = 3;

let sum: Int = a + b;      // 13
let diff: Int = a - b;     // 7
let prod: Int = a * b;     // 30
let quot: Int = a / b;     // 3（整数除法）
let rem: Int = a % b;      // 1（取模）

// 复合赋值
var x: Int = 10;
x += 5;                    // x = 15
x -= 3;                    // x = 12
x *= 2;                    // x = 24
```

**比较运算符：**

```tact
let a: Int = 10;
let b: Int = 20;

let eq: Bool = a == b;     // false
let ne: Bool = a != b;     // true
let lt: Bool = a < b;      // true
let le: Bool = a <= b;     // true
let gt: Bool = a > b;      // false
let ge: Bool = a >= b;     // false
```

**逻辑运算符：**

```tact
let a: Bool = true;
let b: Bool = false;

let and: Bool = a && b;    // false
let or: Bool = a || b;     // true
let not: Bool = !a;        // false
```

#### 6.2.3 控制流

**条件语句：**

```tact
// if-else
if (condition) {
    // 执行代码
} else if (anotherCondition) {
    // 执行代码
} else {
    // 执行代码
}

// 示例
fun checkValue(x: Int): String {
    if (x > 100) {
        return "Large";
    } else if (x > 50) {
        return "Medium";
    } else {
        return "Small";
    }
}
```

**循环：**

```tact
// while 循环
var i: Int = 0;
while (i < 10) {
    // 执行代码
    i = i + 1;
}

// repeat 循环
repeat (5) {
    // 执行 5 次
}

// 遍历范围
foreach (key, value in self.map) {
    // 处理每个键值对
}
```

**try-catch 异常处理：**

```tact
try {
    // 可能抛出异常的代码
    let result = riskyOperation();
} catch (e) {
    // 处理异常
    dump("Error occurred");
}
```

---

### 6.3 合约定义

#### 6.3.1 基本合约结构

```tact
import "@stdlib/deploy";

// 定义合约
contract SimpleContract with Deployable {
    // 状态变量
    counter: Int = 0;
    owner: Address;
    
    // 构造函数（初始化）
    init(owner: Address) {
        self.owner = owner;
    }
    
    // 接收消息的函数
    receive(msg: IncrementMessage) {
        require(sender() == self.owner, "Only owner");
        self.counter = self.counter + msg.amount;
    }
    
    // 接收简单转账
    receive() {
        // 处理纯 TON 转账
    }
    
    // Getter 函数（只读）
    get fun getCounter(): Int {
        return self.counter;
    }
    
    get fun getOwner(): Address {
        return self.owner;
    }
}
```

#### 6.3.2 状态变量

```tact
contract StateExample {
    // 基本类型状态
    intValue: Int = 0;
    boolValue: Bool = false;
    addressValue: Address;
    
    // 映射（字典）
    balances: map<Address, Int>;
    metadata: map<Int, Cell>;
    
    // 可选类型
    optionalValue: Int? = null;
    
    // 结构体
    user: UserStruct?;
    
    // 常量
    const MAX_VALUE: Int = 1000000;
}

// 结构体定义
struct UserStruct {
    name: String;
    balance: Int;
    isActive: Bool;
}
```

**映射（Map）操作：**

```tact
contract MapExample {
    balances: map<Address, Int>;
    
    fun setBalance(addr: Address, amount: Int) {
        self.balances.set(addr, amount);
    }
    
    fun getBalance(addr: Address): Int {
        // 使用 ?? 提供默认值
        return self.balances.get(addr) ?? 0;
    }
    
    fun hasBalance(addr: Address): Bool {
        return self.balances.exists(addr);
    }
    
    fun removeBalance(addr: Address) {
        self.balances.del(addr);
    }
}
```

#### 6.3.3 Trait（特质/接口）

```tact
// 定义 Trait
trait Ownable {
    owner: Address;
    
    fun requireOwner() {
        require(sender() == self.owner, "Not owner");
    }
    
    get fun getOwner(): Address {
        return self.owner;
    }
}

trait Pausable {
    paused: Bool = false;
    
    fun whenNotPaused() {
        require(!self.paused, "Paused");
    }
    
    fun whenPaused() {
        require(self.paused, "Not paused");
    }
}

// 使用 Trait
contract MyToken with Ownable, Pausable {
    owner: Address;
    paused: Bool = false;
    
    init(owner: Address) {
        self.owner = owner;
    }
    
    receive(msg: TransferMessage) {
        self.requireOwner();
        self.whenNotPaused();
        // 转账逻辑
    }
}
```

---

### 6.4 消息处理

#### 6.4.1 消息定义

```tact
// 定义消息结构
message TransferMessage {
    to: Address;
    amount: Int as coins;
    payload: Cell? = null;
}

message MintMessage {
    to: Address;
    amount: Int as coins;
}

message BurnMessage {
    amount: Int as coins;
}

// 带操作码的消息（用于兼容性）
message(0x12345678) CustomMessage {
    field1: Int;
    field2: Address;
}
```

#### 6.4.2 接收消息

```tact
contract MessageHandler {
    // 接收特定类型的消息
    receive(msg: TransferMessage) {
        // 处理转账消息
        self.transfer(msg.to, msg.amount);
    }
    
    // 接收文本消息
    receive("deposit") {
        // 处理 "deposit" 文本消息
        self.deposit();
    }
    
    // 接收任意 Slice 数据
    receive(msg: Slice) {
        // 手动解析消息
        let op: Int = msg.loadUint(32);
        // ...
    }
    
    // 接收空消息（纯 TON 转账）
    receive() {
        // 处理纯转账
    }
}
```

#### 6.4.3 发送消息

```tact
import "@stdlib/deploy";

contract MessageSender with Deployable {
    // 发送简单消息
    fun sendSimple(to: Address, amount: Int) {
        send(SendParameters{
            to: to,
            value: amount,
            mode: SendIgnoreErrors,
            body: null
        });
    }
    
    // 发送带消息体的消息
    fun sendWithMessage(to: Address, amount: Int) {
        send(SendParameters{
            to: to,
            value: amount,
            mode: SendPayGasSeparately,
            body: TransferMessage{
                to: sender(),
                amount: amount,
                payload: null
            }.toCell()
        });
    }
    
    // 发送带引用的消息
    fun sendWithRef(to: Address, data: Cell) {
        send(SendParameters{
            to: to,
            value: ton("0.05"),
            mode: SendPayGasSeparately,
            body: beginCell()
                .storeUint(0x12345678, 32)
                .storeRef(data)
                .endCell()
        });
    }
}
```

**发送模式常量：**

```tact
// 发送模式
const SendDefault: Int = 0;
const SendPayGasSeparately: Int = 1;      // 单独支付 Gas
const SendIgnoreErrors: Int = 2;          // 忽略发送错误
const SendDestroyAccount: Int = 32;       // 发送后销毁账户
const SendRemainingValue: Int = 64;       // 携带所有剩余余额
const SendAllBalance: Int = 128;          // 携带全部余额

// 常用组合
const SendRemainingBalance: Int = 64 + 1; // 携带剩余余额 + 单独支付 Gas
```

---

### 6.5 标准库使用

#### 6.5.1 上下文信息

```tact
import "@stdlib/context";

contract ContextExample {
    fun showContext() {
        // 发送者地址
        let senderAddr: Address = sender();
        
        // 本合约地址
        let myAddr: Address = myAddress();
        
        // 收到的金额
        let value: Int = context().value;
        
        // 原始消息体
        let body: Slice = context().raw;
        
        // 当前时间戳
        let now: Int = now();
        
        // 区块逻辑时间
        let lt: Int = curLt();
        
        // 随机数
        let random: Int = random();
        let randomInRange: Int = randomInt(100);  // 0-99
    }
}
```

#### 6.5.2 地址操作

```tact
import "@stdlib/address";

contract AddressExample {
    fun addressOperations() {
        // 创建地址
        let addr: Address = address("EQD...");
        
        // 检查地址是否为空
        let isEmpty: Bool = addr.isEmpty();
        
        // 地址转字符串
        let addrString: String = addr.toString();
        
        // 获取工作链 ID
        let wc: Int = addr.workchain();
        
        // 新地址（用于部署）
        let newAddr: Address = newAddress(0, hash);
    }
}
```

#### 6.5.3 Cell 操作

```tact
contract CellExample {
    fun cellOperations() {
        // 创建 Builder
        let b: Builder = beginCell();
        
        // 存储数据
        b = b.storeUint(42, 32);           // 存储无符号整数
        b = b.storeInt(-10, 32);           // 存储有符号整数
        b = b.storeBool(true);             // 存储布尔值
        b = b.storeCoins(ton("1.5"));      // 存储金额
        b = b.storeAddress(sender());      // 存储地址
        b = b.storeRef(emptyCell());       // 存储 Cell 引用
        
        // 结束构建
        let c: Cell = b.endCell();
        
        // 读取 Slice
        let s: Slice = c.beginParse();
        let value: Int = s.loadUint(32);
        let addr: Address = s.loadAddress();
        let ref: Cell = s.loadRef();
        
        // 检查剩余数据
        let remainingBits: Int = s.bits();
        let remainingRefs: Int = s.refs();
    }
}
```

#### 6.5.4 数学函数

```tact
import "@stdlib/math";

contract MathExample {
    fun mathOperations() {
        // 基本运算
        let abs: Int = abs(-10);           // 10
        let min: Int = min(10, 20);        // 10
        let max: Int = max(10, 20);        // 20
        
        // 幂运算和对数
        let pow: Int = pow(2, 10);         // 1024
        let log2: Int = log2(1024);        // 10
        
        // 位运算
        let and: Int = 12 & 10;            // 8
        let or: Int = 12 | 10;             // 14
        let xor: Int = 12 ^ 10;            // 6
        let not: Int = ~12;                // 按位非
        
        // 检查位
        let isBitSet: Bool = checkBit(8, 3);  // true (8 = 1000)
    }
}
```

---

### 6.6 完整示例：ERC20 风格代币合约

```tact
import "@stdlib/deploy";
import "@stdlib/ownable";

// 消息定义
message Mint {
    amount: Int;
    receiver: Address;
}

message Burn {
    amount: Int;
}

message Transfer {
    to: Address;
    amount: Int;
}

// 事件（通过消息模拟）
message TransferEvent {
    from: Address;
    to: Address;
    amount: Int;
}

// 代币合约
contract Token with Deployable, Ownable {
    // 状态变量
    totalSupply: Int as coins = 0;
    balances: map<Address, Int>;
    owner: Address;
    
    // 代币元数据
    name: String = "MyToken";
    symbol: String = "MTK";
    decimals: Int = 9;
    
    // 初始化
    init(owner: Address) {
        self.owner = owner;
    }
    
    // 铸造代币（仅所有者）
    receive(msg: Mint) {
        self.requireOwner();
        
        // 更新总供应量
        self.totalSupply = self.totalSupply + msg.amount;
        
        // 更新接收者余额
        let currentBalance: Int = self.balances.get(msg.receiver) ?: 0;
        self.balances.set(msg.receiver, currentBalance + msg.amount);
    }
    
    // 销毁代币
    receive(msg: Burn) {
        let sender: Address = sender();
        let currentBalance: Int = self.balances.get(sender) ?: 0;
        
        require(currentBalance >= msg.amount, "Insufficient balance");
        
        // 更新余额
        self.balances.set(sender, currentBalance - msg.amount);
        
        // 更新总供应量
        self.totalSupply = self.totalSupply - msg.amount;
    }
    
    // 转账
    receive(msg: Transfer) {
        let sender: Address = sender();
        let senderBalance: Int = self.balances.get(sender) ?: 0;
        
        require(senderBalance >= msg.amount, "Insufficient balance");
        require(msg.to != sender(), "Cannot transfer to self");
        
        // 扣除发送者余额
        self.balances.set(sender, senderBalance - msg.amount);
        
        // 增加接收者余额
        let receiverBalance: Int = self.balances.get(msg.to) ?: 0;
        self.balances.set(msg.to, receiverBalance + msg.amount);
    }
    
    // 接收存款
    receive() {
        // 接受 TON 转账
    }
    
    // Getter 函数
    get fun getBalance(addr: Address): Int {
        return self.balances.get(addr) ?: 0;
    }
    
    get fun getTotalSupply(): Int {
        return self.totalSupply;
    }
    
    get fun getName(): String {
        return self.name;
    }
    
    get fun getSymbol(): String {
        return self.symbol;
    }
    
    get fun getDecimals(): Int {
        return self.decimals;
    }
}
```

---

### 6.7 测试与部署

#### 6.7.1 编写测试

```typescript
import { Blockchain } from "@ton-community/sandbox";
import { Token } from "../build/Token";
import { toNano } from "@ton/core";

describe("Token Contract", () => {
    let blockchain: Blockchain;
    let token: Token;
    let owner: any;
    let user: any;
    
    beforeEach(async () => {
        blockchain = await Blockchain.create();
        owner = await blockchain.treasury("owner");
        user = await blockchain.treasury("user");
        
        // 部署合约
        token = blockchain.openContract(
            await Token.fromInit(owner.address)
        );
        
        await token.send(
            owner.getSender(),
            { value: toNano("0.05") },
            { $$type: "Deploy", queryId: 0n }
        );
    });
    
    it("should mint tokens", async () => {
        // 铸造代币
        await token.send(
            owner.getSender(),
            { value: toNano("0.05") },
            {
                $$type: "Mint",
                amount: 1000n,
                receiver: user.address
            }
        );
        
        // 验证余额
        const balance = await token.getGetBalance(user.address);
        expect(balance).toBe(1000n);
    });
    
    it("should transfer tokens", async () => {
        // 先铸造
        await token.send(
            owner.getSender(),
            { value: toNano("0.05") },
            {
                $$type: "Mint",
                amount: 1000n,
                receiver: owner.address
            }
        );
        
        // 转账
        await token.send(
            owner.getSender(),
            { value: toNano("0.05") },
            {
                $$type: "Transfer",
                to: user.address,
                amount: 500n
            }
        );
        
        // 验证余额
        const ownerBalance = await token.getGetBalance(owner.address);
        const userBalance = await token.getGetBalance(user.address);
        
        expect(ownerBalance).toBe(500n);
        expect(userBalance).toBe(500n);
    });
});
```

#### 6.7.2 部署脚本

```typescript
import { toNano } from "@ton/core";
import { Token } from "../build/Token";
import { NetworkProvider } from "@ton/blueprint";

export async function run(provider: NetworkProvider) {
    // 创建合约实例
    const token = provider.open(
        await Token.fromInit(provider.sender().address!!)
    );
    
    // 部署合约
    await token.send(
        provider.sender(),
        { value: toNano("0.05") },
        { $$type: "Deploy", queryId: 0n }
    );
    
    // 等待部署完成
    await provider.waitForDeploy(token.address);
    
    console.log("Token deployed at:", token.address.toString());
    
    // 铸造初始代币
    await token.send(
        provider.sender(),
        { value: toNano("0.05") },
        {
            $$type: "Mint",
            amount: 1000000n,
            receiver: provider.sender().address!!
        }
    );
    
    console.log("Initial tokens minted");
}
```

---

**本章小结：**

本章介绍了 Tact 语言的基础知识，包括：
- Tact 语言概述和开发环境搭建
- 基础语法（变量、类型、运算符、控制流）
- 合约定义（状态变量、Trait、构造函数）
- 消息处理（消息定义、接收、发送）
- 标准库使用（上下文、地址、Cell、数学函数）
- 完整示例（ERC20 风格代币合约）
- 测试与部署

Tact 以其 TypeScript 风格的语法、强大的类型系统和丰富的标准库，成为 TON 生态中最受欢迎的智能合约语言之一。无论是初学者还是有经验的开发者，都能快速上手并构建安全的智能合约。

---

## 第 7 章：智能合约设计模式

设计模式是解决特定问题的经过验证的解决方案。在 TON 区块链上开发智能合约时，正确的设计模式可以提高代码的可维护性、安全性和效率。本章介绍 TON 生态中最常用的智能合约设计模式。

### 7.1 所有权模式（Ownership Pattern）

#### 7.1.1 单一所有权

最常见的模式，合约有一个明确的所有者，拥有特殊权限。

```tact
import "@stdlib/ownable";

// 使用标准库的 Ownable Trait
contract OwnedContract with Ownable {
    owner: Address;
    
    init(owner: Address) {
        self.owner = owner;
    }
    
    // 只有所有者可以调用的函数
    receive(msg: AdminAction) {
        self.requireOwner();
        // 执行管理操作
    }
}
```

**Tolk 实现：**

```tolk
// 所有权模式
trait Ownable {
    owner: address;
    
    fun requireOwner(): void {
        require(sender() == self.owner, "Not owner");
    }
}

contract OwnedContract with Ownable {
    owner: address;
    
    init(owner: address) {
        self.owner = owner;
    }
    
    receive(msg: AdminMessage) {
        self.requireOwner();
        // 执行管理操作
    }
}
```

#### 7.1.2 多签名所有权

需要多个地址共同授权才能执行敏感操作。

```tact
contract MultiSig {
    owners: map<Address, Bool>;
    requiredConfirmations: Int;
    pendingTransactions: map<Int, PendingTx>;
    
    struct PendingTx {
        to: Address;
        value: Int;
        confirmations: Int;
        confirmed: map<Address, Bool>;
    }
    
    // 提交交易
    receive(msg: SubmitTransaction) {
        require(self.owners.get(sender()) == true, "Not owner");
        
        let txId = self.nextTxId;
        self.pendingTransactions.set(txId, PendingTx{
            to: msg.to,
            value: msg.value,
            confirmations: 1,
            confirmed: emptyMap()
        });
        
        // 自动确认提交者
        let tx = self.pendingTransactions.get(txId)!!;
        tx.confirmed.set(sender(), true);
        
        // 检查是否达到确认数
        if (tx.confirmations >= self.requiredConfirmations) {
            self.executeTransaction(txId);
        }
    }
    
    // 确认交易
    receive(msg: ConfirmTransaction) {
        require(self.owners.get(sender()) == true, "Not owner");
        
        let tx = self.pendingTransactions.get(msg.txId)!!;
        require(tx.confirmed.get(sender()) != true, "Already confirmed");
        
        tx.confirmations = tx.confirmations + 1;
        tx.confirmed.set(sender(), true);
        
        if (tx.confirmations >= self.requiredConfirmations) {
            self.executeTransaction(msg.txId);
        }
    }
}
```

#### 7.1.3 可转移所有权

允许将所有权转移给其他地址。

```tact
contract TransferableOwnership {
    owner: Address;
    pendingOwner: Address? = null;
    
    // 开始所有权转移
    receive(msg: TransferOwnership) {
        require(sender() == self.owner, "Not owner");
        self.pendingOwner = msg.newOwner;
    }
    
    // 接受所有权
    receive(msg: AcceptOwnership) {
        require(sender() == self.pendingOwner, "Not pending owner");
        self.owner = sender();
        self.pendingOwner = null;
    }
    
    // 取消转移
    receive(msg: CancelTransfer) {
        require(sender() == self.owner, "Not owner");
        self.pendingOwner = null;
    }
}
```

---

### 7.2 可暂停模式（Pausable Pattern）

允许在紧急情况下暂停合约功能。

```tact
import "@stdlib/ownable";

trait Pausable {
    paused: Bool = false;
    
    fun whenNotPaused() {
        require(!self.paused, "Contract is paused");
    }
    
    fun whenPaused() {
        require(self.paused, "Contract is not paused");
    }
    
    fun pause() {
        self.paused = true;
    }
    
    fun unpause() {
        self.paused = false;
    }
}

contract PausableToken with Ownable, Pausable {
    owner: Address;
    paused: Bool = false;
    
    // 可暂停的转账功能
    receive(msg: Transfer) {
        self.whenNotPaused();  // 检查是否暂停
        // 执行转账
    }
    
    // 只有所有者可以暂停
    receive("pause") {
        self.requireOwner();
        self.pause();
    }
    
    receive("unpause") {
        self.requireOwner();
        self.unpause();
    }
}
```

---

### 7.3 重入防护模式（Reentrancy Guard）

防止重入攻击，确保函数在执行期间不能被再次调用。

```tact
contract ReentrancyGuard {
    locked: Bool = false;
    
    fun nonReentrant() {
        require(!self.locked, "Reentrant call");
        self.locked = true;
    }
    
    fun release() {
        self.locked = false;
    }
    
    // 使用示例
    receive(msg: Withdraw) {
        self.nonReentrant();
        
        // 执行提款逻辑
        let amount = self.balances.get(sender())!!;
        require(amount >= msg.amount, "Insufficient balance");
        
        // 先更新状态
        self.balances.set(sender(), amount - msg.amount);
        
        // 再发送消息（检查-生效-交互模式）
        send(SendParameters{
            to: sender(),
            value: msg.amount,
            mode: SendPayGasSeparately
        });
        
        self.release();
    }
}
```

**检查-生效-交互（Checks-Effects-Interactions）模式：**

```tact
contract SecureContract {
    // ❌ 错误顺序：先交互，后生效
    receive(msg: WithdrawBad) {
        send(SendParameters{  // 1. 交互（危险！）
            to: sender(),
            value: msg.amount
        });
        self.balances.set(sender(), 0);  // 2. 生效（太晚）
    }
    
    // ✅ 正确顺序：检查 -> 生效 -> 交互
    receive(msg: WithdrawGood) {
        // 1. 检查
        let balance = self.balances.get(sender())!!;
        require(balance >= msg.amount, "Insufficient");
        
        // 2. 生效（先更新状态）
        self.balances.set(sender(), balance - msg.amount);
        
        // 3. 交互（最后执行）
        send(SendParameters{
            to: sender(),
            value: msg.amount,
            mode: SendPayGasSeparately
        });
    }
}
```

---

### 7.4 代理模式（Proxy Pattern）

允许在不改变合约地址的情况下升级合约逻辑。

#### 7.4.1 简单代理

```tact
// 代理合约（保持不变）
contract Proxy {
    implementation: Address;
    admin: Address;
    
    init(admin: Address, implementation: Address) {
        self.admin = admin;
        self.implementation = implementation;
    }
    
    // 转发所有消息到实现合约
    receive(msg: Slice) {
        // 使用 rawReserve 保留余额
        nativeReserve(0, ReserveExact);
        
        // 转发消息
        send(SendParameters{
            to: self.implementation,
            value: 0,
            mode: SendRemainingValue + SendIgnoreErrors,
            body: msg
        });
    }
    
    // 升级实现合约
    receive(msg: Upgrade) {
        require(sender() == self.admin, "Not admin");
        self.implementation = msg.newImplementation;
    }
}
```

#### 7.4.2 钻石代理（Diamond Proxy）

支持多个实现合约，按功能选择不同的实现。

```tact
contract DiamondProxy {
    facets: map<Int, Address>;  // 选择器 -> 实现合约
    admin: Address;
    
    // 根据函数选择器路由到对应的实现
    receive(msg: Slice) {
        let selector: Int = msg.preloadUint(32);
        let facet = self.facets.get(selector);
        
        require(facet != null, "Function not found");
        
        send(SendParameters{
            to: facet!!,
            value: 0,
            mode: SendRemainingValue,
            body: msg
        });
    }
    
    // 添加/更新 facet
    receive(msg: AddFacet) {
        require(sender() == self.admin, "Not admin");
        
        foreach (selector in msg.selectors) {
            self.facets.set(selector, msg.facet);
        }
    }
    
    // 移除 facet
    receive(msg: RemoveFacet) {
        require(sender() == self.admin, "Not admin");
        
        foreach (selector in msg.selectors) {
            self.facets.del(selector);
        }
    }
}
```

---

### 7.5 工厂模式（Factory Pattern）

用于创建和管理多个合约实例。

```tact
// 代币合约（由工厂创建）
contract Token {
    factory: Address;
    name: String;
    symbol: String;
    totalSupply: Int;
    
    init(factory: Address, name: String, symbol: String) {
        self.factory = factory;
        self.name = name;
        self.symbol = symbol;
        self.totalSupply = 0;
    }
}

// 工厂合约
contract TokenFactory {
    tokens: map<Address, TokenInfo>;
    tokenCode: Cell;  // Token 合约的代码
    
    struct TokenInfo {
        name: String;
        symbol: String;
        creator: Address;
        createdAt: Int;
    }
    
    // 创建新代币
    receive(msg: CreateToken) {
        // 计算新地址
        let stateInit = StateInit{
            code: self.tokenCode,
            data: beginCell()
                .storeAddress(myAddress())
                .storeRef(beginCell().storeString(msg.name).endCell())
                .storeRef(beginCell().storeString(msg.symbol).endCell())
                .endCell()
        };
        
        let tokenAddress = contractAddress(stateInit);
        
        // 记录代币信息
        self.tokens.set(tokenAddress, TokenInfo{
            name: msg.name,
            symbol: msg.symbol,
            creator: sender(),
            createdAt: now()
        });
        
        // 部署代币合约
        send(SendParameters{
            to: tokenAddress,
            value: msg.deployValue,
            mode: SendPayGasSeparately,
            code: stateInit.code,
            data: stateInit.data,
            body: "deploy".asComment()
        });
    }
    
    // 查询代币信息
    get fun getTokenInfo(addr: Address): TokenInfo? {
        return self.tokens.get(addr);
    }
    
    get fun getAllTokens(): map<Address, TokenInfo> {
        return self.tokens;
    }
}
```

---

### 7.6 拉取模式（Pull Pattern）

用户主动领取资金，而不是合约主动推送。

```tact
// ❌ 推模式（Push）- 可能失败
contract PushPattern {
    receive(msg: DistributeRewards) {
        foreach (user, amount in self.rewards) {
            // 如果某个发送失败，整个交易可能失败
            send(SendParameters{
                to: user,
                value: amount
            });
        }
    }
}

// ✅ 拉模式（Pull）- 更安全
contract PullPattern {
    pendingWithdrawals: map<Address, Int>;
    
    // 管理员设置可提取金额
    receive(msg: SetWithdrawal) {
        self.requireOwner();
        self.pendingWithdrawals.set(msg.user, msg.amount);
    }
    
    // 用户主动提取
    receive("withdraw") {
        let amount = self.pendingWithdrawals.get(sender())!!;
        require(amount > 0, "No pending withdrawal");
        
        // 清除待提取金额
        self.pendingWithdrawals.del(sender());
        
        // 发送资金
        send(SendParameters{
            to: sender(),
            value: amount,
            mode: SendPayGasSeparately
        });
    }
    
    get fun getPendingWithdrawal(user: Address): Int {
        return self.pendingWithdrawals.get(user) ?: 0;
    }
}
```

---

### 7.7 状态机模式（State Machine）

使用有限状态机管理合约生命周期。

```tact
contract StateMachine {
    // 状态定义
    const STATE_CREATED: Int = 0;
    const STATE_ACTIVE: Int = 1;
    const STATE_PAUSED: Int = 2;
    const STATE_CLOSED: Int = 3;
    
    state: Int = STATE_CREATED;
    
    // 状态转换函数
    fun transitionTo(newState: Int) {
        // 验证状态转换是否合法
        require(self.isValidTransition(self.state, newState), "Invalid transition");
        self.state = newState;
    }
    
    fun isValidTransition(from: Int, to: Int): Bool {
        // 定义允许的状态转换
        if (from == STATE_CREATED && to == STATE_ACTIVE) { return true; }
        if (from == STATE_ACTIVE && to == STATE_PAUSED) { return true; }
        if (from == STATE_PAUSED && to == STATE_ACTIVE) { return true; }
        if (from == STATE_ACTIVE && to == STATE_CLOSED) { return true; }
        if (from == STATE_PAUSED && to == STATE_CLOSED) { return true; }
        return false;
    }
    
    // 激活合约
    receive("activate") {
        self.requireOwner();
        self.transitionTo(STATE_ACTIVE);
    }
    
    // 暂停合约
    receive("pause") {
        self.requireOwner();
        self.transitionTo(STATE_PAUSED);
    }
    
    // 关闭合约
    receive("close") {
        self.requireOwner();
        self.transitionTo(STATE_CLOSED);
    }
    
    // 业务函数检查状态
    receive(msg: DoSomething) {
        require(self.state == STATE_ACTIVE, "Not active");
        // 执行业务逻辑
    }
    
    get fun getState(): Int {
        return self.state;
    }
}
```

---

### 7.8 访问控制模式（Access Control）

基于角色的权限管理。

```tact
contract AccessControl {
    roles: map<Address, Int>;  // 地址 -> 角色位掩码
    
    // 角色定义
    const ROLE_NONE: Int = 0;
    const ROLE_ADMIN: Int = 1;      // 0001
    const ROLE_MINTER: Int = 2;     // 0010
    const ROLE_BURNER: Int = 4;     // 0100
    const ROLE_PAUSER: Int = 8;     // 1000
    
    // 检查角色
    fun hasRole(user: Address, role: Int): Bool {
        let userRoles = self.roles.get(user) ?: ROLE_NONE;
        return (userRoles & role) != 0;
    }
    
    // 授予角色
    fun grantRole(user: Address, role: Int) {
        self.requireAdmin();
        let currentRoles = self.roles.get(user) ?: ROLE_NONE;
        self.roles.set(user, currentRoles | role);
    }
    
    // 撤销角色
    fun revokeRole(user: Address, role: Int) {
        self.requireAdmin();
        let currentRoles = self.roles.get(user) ?: ROLE_NONE;
        self.roles.set(user, currentRoles & ~role);
    }
    
    // 修饰器函数
    fun requireAdmin() {
        require(self.hasRole(sender(), ROLE_ADMIN), "Not admin");
    }
    
    fun requireMinter() {
        require(self.hasRole(sender(), ROLE_MINTER), "Not minter");
    }
    
    // 使用示例
    receive(msg: Mint) {
        self.requireMinter();
        // 铸造逻辑
    }
    
    receive(msg: Pause) {
        require(self.hasRole(sender(), ROLE_PAUSER | ROLE_ADMIN), "Not authorized");
        // 暂停逻辑
    }
}
```

---

### 7.9 防溢出模式（Overflow Protection）

TON 的整数运算默认会溢出，需要手动检查。

```tact
contract SafeMath {
    // 安全加法
    fun safeAdd(a: Int, b: Int): Int {
        let c = a + b;
        require(c >= a, "Addition overflow");
        return c;
    }
    
    // 安全减法
    fun safeSub(a: Int, b: Int): Int {
        require(b <= a, "Subtraction underflow");
        return a - b;
    }
    
    // 安全乘法
    fun safeMul(a: Int, b: Int): Int {
        if (a == 0) return 0;
        let c = a * b;
        require(c / a == b, "Multiplication overflow");
        return c;
    }
    
    // 安全除法
    fun safeDiv(a: Int, b: Int): Int {
        require(b > 0, "Division by zero");
        return a / b;
    }
    
    // 使用示例
    receive(msg: Transfer) {
        let senderBalance = self.balances.get(sender())!!;
        let newBalance = self.safeSub(senderBalance, msg.amount);
        self.balances.set(sender(), newBalance);
        
        let receiverBalance = self.balances.get(msg.to) ?: 0;
        let newReceiverBalance = self.safeAdd(receiverBalance, msg.amount);
        self.balances.set(msg.to, newReceiverBalance);
    }
}
```

---

### 7.10 设计模式选择指南

| 模式 | 适用场景 | 复杂度 | Gas 开销 |
|------|----------|--------|----------|
| 单一所有权 | 简单合约、管理员功能 | 低 | 低 |
| 多签名 | 资金管理、重要操作 | 高 | 中 |
| 可暂停 | 需要紧急停止的合约 | 低 | 低 |
| 重入防护 | 涉及外部调用的合约 | 中 | 低 |
| 代理 | 需要升级的合约 | 高 | 中 |
| 工厂 | 创建多个相似合约 | 中 | 中 |
| 拉取 | 批量分发、奖励发放 | 中 | 低 |
| 状态机 | 复杂生命周期的合约 | 中 | 低 |
| 访问控制 | 多角色权限管理 | 中 | 低 |
| 防溢出 | 所有涉及计算的合约 | 低 | 低 |

---

**本章小结：**

本章介绍了 TON 智能合约开发中最常用的设计模式：
- 所有权模式（单一所有权、多签名、可转移）
- 可暂停模式（紧急停止机制）
- 重入防护模式（检查-生效-交互）
- 代理模式（简单代理、钻石代理）
- 工厂模式（合约创建管理）
- 拉取模式（用户主动领取）
- 状态机模式（生命周期管理）
- 访问控制模式（基于角色的权限）
- 防溢出模式（安全数学运算）

正确应用这些设计模式可以显著提高合约的安全性和可维护性。在实际开发中，应根据具体需求选择合适的模式组合使用。

---

## 第 8 章：合约安全与最佳实践

智能合约一旦部署就难以修改，且直接管理资产，因此安全性至关重要。本章介绍 TON 智能合约开发中的常见安全风险和最佳实践。

### 8.1 常见安全漏洞

#### 8.1.1 重入攻击（Reentrancy）

**攻击原理：**
攻击者合约在接收资金时回调目标合约，重复执行提款逻辑。

```tact
// ❌ 易受攻击的代码
contract Vulnerable {
    balances: map<Address, Int>;
    
    receive(msg: Withdraw) {
        let balance = self.balances.get(sender())!!;
        require(balance >= msg.amount, "Insufficient");
        
        // 先发送资金（危险！）
        send(SendParameters{
            to: sender(),
            value: msg.amount
        });
        
        // 后更新状态（太晚）
        self.balances.set(sender(), balance - msg.amount);
    }
}
```

**防护措施：**

```tact
// ✅ 安全的代码
contract Secure {
    balances: map<Address, Int>;
    locked: Bool = false;
    
    fun nonReentrant() {
        require(!self.locked, "Reentrant");
        self.locked = true;
    }
    
    fun release() {
        self.locked = false;
    }
    
    receive(msg: Withdraw) {
        self.nonReentrant();
        
        // 1. 检查
        let balance = self.balances.get(sender())!!;
        require(balance >= msg.amount, "Insufficient");
        
        // 2. 生效（先更新状态）
        self.balances.set(sender(), balance - msg.amount);
        
        // 3. 交互（最后执行）
        send(SendParameters{
            to: sender(),
            value: msg.amount,
            mode: SendPayGasSeparately
        });
        
        self.release();
    }
}
```

#### 8.1.2 整数溢出/下溢

**攻击原理：**
TON 的整数运算默认会溢出，攻击者可以利用这一点操纵计算结果。

```tact
// ❌ 易受攻击的代码
contract VulnerableOverflow {
    balances: map<Address, Int>;
    
    receive(msg: Transfer) {
        let senderBalance = self.balances.get(sender())!!;
        // 如果 senderBalance < msg.amount，会下溢变成很大的数
        self.balances.set(sender(), senderBalance - msg.amount);
        
        let receiverBalance = self.balances.get(msg.to) ?: 0;
        // 可能溢出
        self.balances.set(msg.to, receiverBalance + msg.amount);
    }
}
```

**防护措施：**

```tact
// ✅ 安全的代码
contract SafeOverflow {
    balances: map<Address, Int>;
    
    fun safeSub(a: Int, b: Int): Int {
        require(b <= a, "Subtraction underflow");
        return a - b;
    }
    
    fun safeAdd(a: Int, b: Int): Int {
        let c = a + b;
        require(c >= a, "Addition overflow");
        return c;
    }
    
    receive(msg: Transfer) {
        let senderBalance = self.balances.get(sender())!!;
        require(senderBalance >= msg.amount, "Insufficient");
        
        self.balances.set(sender(), self.safeSub(senderBalance, msg.amount));
        
        let receiverBalance = self.balances.get(msg.to) ?: 0;
        self.balances.set(msg.to, self.safeAdd(receiverBalance, msg.amount));
    }
}
```

#### 8.1.3 访问控制缺失

**攻击原理：**
敏感函数没有适当的权限检查，任何人都可以调用。

```tact
// ❌ 易受攻击的代码
contract MissingAccessControl {
    owner: Address;
    
    // 任何人都可以调用，销毁合约并取走所有资金
    receive("destroy") {
        send(SendParameters{
            to: sender(),
            value: myBalance(),
            mode: SendDestroyAccount
        });
    }
    
    // 任何人都可以铸造代币
    receive(msg: Mint) {
        self.totalSupply = self.totalSupply + msg.amount;
        self.balances.set(msg.to, msg.amount);
    }
}
```

**防护措施：**

```tact
// ✅ 安全的代码
contract ProperAccessControl with Ownable {
    owner: Address;
    
    receive("destroy") {
        self.requireOwner();  // 只有所有者可以销毁
        send(SendParameters{
            to: sender(),
            value: myBalance(),
            mode: SendDestroyAccount
        });
    }
    
    receive(msg: Mint) {
        self.requireOwner();  // 只有所有者可以铸造
        self.totalSupply = self.totalSupply + msg.amount;
        self.balances.set(msg.to, msg.amount);
    }
}
```

#### 8.1.4 拒绝服务（DoS）

**攻击原理：**
通过消耗所有 Gas 或制造错误条件，阻止合约正常功能。

```tact
// ❌ 易受攻击的代码
contract DoSVulnerable {
    users: map<Address, Int>;
    
    // 遍历所有用户发送资金，Gas 可能耗尽
    receive("distribute") {
        let total = self.totalRewards;
        foreach (user, share in self.users) {
            let reward = total * share / self.totalShares;
            send(SendParameters{
                to: user,
                value: reward
            });
        }
    }
}
```

**防护措施：**

```tact
// ✅ 使用拉取模式
contract DoSSafe {
    users: map<Address, Int>;
    pendingRewards: map<Address, Int>;
    
    // 计算奖励但不发送
    receive("calculateRewards") {
        self.requireOwner();
        foreach (user, share in self.users) {
            let reward = self.totalRewards * share / self.totalShares;
            self.pendingRewards.set(user, reward);
        }
    }
    
    // 用户自行领取
    receive("claim") {
        let reward = self.pendingRewards.get(sender())!!;
        require(reward > 0, "No reward");
        
        self.pendingRewards.del(sender());
        
        send(SendParameters{
            to: sender(),
            value: reward,
            mode: SendPayGasSeparately
        });
    }
}
```

#### 8.1.5 伪随机数漏洞

**攻击原理：**
使用可预测的随机数源，攻击者可以预测结果。

```tact
// ❌ 易受攻击的代码
contract BadRandom {
    // 使用区块哈希作为随机数，可以被矿工操纵
    fun random(): Int {
        return blockLt() % 100;  // 可预测！
    }
    
    receive("play") {
        if (self.random() > 50) {
            // 发送奖励
        }
    }
}
```

**防护措施：**

```tact
// ✅ 安全的随机数
contract SecureRandom {
    // 使用 commit-reveal 方案
    commits: map<Address, Int>;      // 用户提交的哈希
    reveals: map<Address, Int>;      // 用户揭示的数值
    
    // 第一步：提交哈希
    receive(msg: Commit) {
        self.commits.set(sender(), msg.hash);
    }
    
    // 第二步：揭示数值
    receive(msg: Reveal) {
        let commit = self.commits.get(sender())!!;
        require(hash(msg.value) == commit, "Invalid reveal");
        
        self.reveals.set(sender(), msg.value);
    }
    
    // 第三步：计算结果（所有揭示后）
    receive("finalize") {
        // 组合所有揭示值生成随机数
        var seed: Int = 0;
        foreach (user, value in self.reveals) {
            seed = seed ^ value;
        }
        
        let random = seed % 100;
        // 使用随机数...
    }
}
```

---

### 8.2 安全编码最佳实践

#### 8.2.1 输入验证

```tact
contract InputValidation {
    // 验证地址
    fun validateAddress(addr: Address): Bool {
        return addr != emptyAddress();
    }
    
    // 验证金额
    fun validateAmount(amount: Int): Bool {
        return amount > 0 && amount <= self.maxAmount;
    }
    
    // 验证消息体
    receive(msg: Transfer) {
        // 检查地址
        require(self.validateAddress(msg.to), "Invalid address");
        require(msg.to != sender(), "Cannot transfer to self");
        
        // 检查金额
        require(self.validateAmount(msg.amount), "Invalid amount");
        
        // 检查余额
        let balance = self.balances.get(sender())!!;
        require(balance >= msg.amount, "Insufficient balance");
        
        // 执行转账
        // ...
    }
}
```

#### 8.2.2 错误处理

```tact
contract ErrorHandling {
    // 定义错误码
    const ERROR_INVALID_ADDRESS: Int = 100;
    const ERROR_INSUFFICIENT_BALANCE: Int = 101;
    const ERROR_UNAUTHORIZED: Int = 102;
    const ERROR_INVALID_AMOUNT: Int = 103;
    const ERROR_CONTRACT_PAUSED: Int = 104;
    
    receive(msg: Transfer) {
        // 使用具体的错误码
        require(msg.to != emptyAddress(), ERROR_INVALID_ADDRESS);
        require(msg.amount > 0, ERROR_INVALID_AMOUNT);
        require(!self.paused, ERROR_CONTRACT_PAUSED);
        
        let balance = self.balances.get(sender())!!;
        require(balance >= msg.amount, ERROR_INSUFFICIENT_BALANCE);
        
        // ...
    }
}
```

#### 8.2.3 事件日志

```tact
contract EventLogging {
    // 定义事件（通过消息模拟）
    message TransferEvent {
        from: Address;
        to: Address;
        amount: Int;
        timestamp: Int;
    }
    
    message ApprovalEvent {
        owner: Address;
        spender: Address;
        amount: Int;
    }
    
    receive(msg: Transfer) {
        // 执行业务逻辑
        // ...
        
        // 发送事件消息到零地址（记录日志）
        send(SendParameters{
            to: emptyAddress(),
            value: 0,
            mode: SendIgnoreErrors,
            body: TransferEvent{
                from: sender(),
                to: msg.to,
                amount: msg.amount,
                timestamp: now()
            }.toCell()
        });
    }
}
```

#### 8.2.4 Gas 优化

```tact
contract GasOptimization {
    // ❌ 低效：存储大量数据
    history: map<Int, Transaction>;  // 所有历史记录
    
    // ✅ 高效：只存储必要的状态
    balances: map<Address, Int>;
    totalSupply: Int;
    
    // ❌ 低效：循环遍历
    fun getTotalBalance(): Int {
        var total: Int = 0;
        foreach (addr, balance in self.balances) {
            total = total + balance;
        }
        return total;
    }
    
    // ✅ 高效：维护累计值
    fun mint(to: Address, amount: Int) {
        self.totalSupply = self.totalSupply + amount;
        let current = self.balances.get(to) ?: 0;
        self.balances.set(to, current + amount);
    }
    
    // ❌ 低效：重复计算
    fun calculateFee(amount: Int): Int {
        return amount * self.feePercent / 10000;
    }
    
    // ✅ 高效：缓存计算结果
    fun transferWithFee(to: Address, amount: Int) {
        let fee = amount * self.feePercent / 10000;  // 只计算一次
        let netAmount = amount - fee;
        // ...
    }
}
```

---

### 8.3 测试策略

#### 8.3.1 单元测试

```typescript
import { Blockchain } from "@ton-community/sandbox";
import { MyContract } from "../build/MyContract";
import { toNano } from "@ton/core";

describe("MyContract", () => {
    let blockchain: Blockchain;
    let contract: MyContract;
    let owner: any;
    let user: any;
    
    beforeEach(async () => {
        blockchain = await Blockchain.create();
        owner = await blockchain.treasury("owner");
        user = await blockchain.treasury("user");
        
        contract = blockchain.openContract(
            await MyContract.fromInit(owner.address)
        );
        
        await contract.send(
            owner.getSender(),
            { value: toNano("0.05") },
            { $$type: "Deploy" }
        );
    });
    
    it("should handle basic transfer", async () => {
        // 测试正常转账
    });
    
    it("should reject insufficient balance", async () => {
        // 测试余额不足
        const result = await contract.send(
            user.getSender(),
            { value: toNano("0.05") },
            { $$type: "Transfer", amount: 1000000n }
        );
        
        expect(result.transactions).toHaveTransaction({
            exitCode: 101  // ERROR_INSUFFICIENT_BALANCE
        });
    });
    
    it("should reject unauthorized access", async () => {
        // 测试未授权访问
        const result = await contract.send(
            user.getSender(),
            { value: toNano("0.05") },
            { $$type: "AdminAction" }
        );
        
        expect(result.transactions).toHaveTransaction({
            exitCode: 102  // ERROR_UNAUTHORIZED
        });
    });
});
```

#### 8.3.2 模糊测试

```typescript
import { fc, it } from "@fast-check/jest";

describe("Fuzzing Tests", () => {
    it("should handle random amounts", async () => {
        await fc.assert(
            fc.asyncProperty(
                fc.bigInt({ min: 0n, max: 1000000000000n }),
                async (amount) => {
                    const result = await contract.send(
                        user.getSender(),
                        { value: toNano("0.05") },
                        { $$type: "Transfer", amount }
                    );
                    
                    // 验证不会崩溃
                    expect(result.transactions).toBeDefined();
                }
            )
        );
    });
});
```

#### 8.3.3 安全审计检查清单

```
□ 访问控制
  □ 所有敏感函数都有适当的权限检查
  □ 没有遗漏的 require 验证
  □ 所有者变更使用两步验证

□ 重入防护
  □ 遵循检查-生效-交互顺序
  □ 使用重入锁保护外部调用
  □ 状态更新在发送消息之前

□ 数学运算
  □ 所有加减乘除都有溢出检查
  □ 除法前有适当的余数处理
  □ 使用安全的数学库

□ 输入验证
  □ 验证所有外部输入
  □ 检查地址有效性
  □ 验证金额范围

□ Gas 优化
  □ 避免无界循环
  □ 使用拉取模式代替推送
  □ 优化存储使用

□ 错误处理
  □ 使用有意义的错误码
  □ 所有错误情况都被处理
  □ 不会静默失败

□ 事件和日志
  □ 所有状态变更都有事件
  □ 事件包含足够信息
  □ 使用 indexed 字段便于查询
```

---

### 8.4 部署前检查清单

#### 8.4.1 代码审查

```
□ 功能正确性
  □ 业务逻辑符合需求
  □ 边界条件处理正确
  □ 状态转换正确

□ 安全性
  □ 通过所有安全测试
  □ 无已知漏洞
  □ 访问控制正确

□ 性能
  □ Gas 消耗合理
  □ 不会耗尽 Gas
  □ 存储使用优化

□ 可维护性
  □ 代码清晰易读
  □ 有适当的注释
  □ 遵循编码规范
```

#### 8.4.2 测试覆盖

```
□ 单元测试覆盖率 > 90%
□ 集成测试通过
□ 模糊测试无崩溃
□ 主网分叉测试（如可能）
□ 安全审计完成
```

#### 8.4.3 部署准备

```
□ 选择正确的网络（testnet/mainnet）
□ 准备足够的部署资金
□ 验证构造函数参数
□ 准备监控和报警
□ 制定应急响应计划
```

---

### 8.5 应急响应

#### 8.5.1 暂停机制

```tact
contract EmergencyStop with Ownable, Pausable {
    owner: Address;
    paused: Bool = false;
    
    // 紧急暂停
    receive("emergencyStop") {
        self.requireOwner();
        self.pause();
        
        // 发送通知
        send(SendParameters{
            to: self.emergencyContact,
            value: 0,
            body: "Contract paused".asComment()
        });
    }
    
    // 所有业务函数检查暂停状态
    receive(msg: Transfer) {
        self.whenNotPaused();
        // ...
    }
}
```

#### 8.5.2 资金回收

```tact
contract FundRecovery with Ownable {
    owner: Address;
    
    // 紧急提取所有资金
    receive("emergencyWithdraw") {
        self.requireOwner();
        
        send(SendParameters{
            to: self.owner,
            value: myBalance() - ton("0.1"),  // 保留少量 Gas
            mode: SendPayGasSeparately
        });
    }
    
    // 迁移到新合约
    receive(msg: Migrate) {
        self.requireOwner();
        
        // 转移所有权
        self.owner = msg.newContract;
        
        // 转移资金
        send(SendParameters{
            to: msg.newContract,
            value: myBalance() - ton("0.1"),
            mode: SendPayGasSeparately
        });
    }
}
```

---

**本章小结：**

本章介绍了 TON 智能合约开发中的安全知识：
- 常见安全漏洞（重入攻击、整数溢出、访问控制缺失、拒绝服务、伪随机数）
- 安全编码最佳实践（输入验证、错误处理、事件日志、Gas 优化）
- 测试策略（单元测试、模糊测试、安全审计检查清单）
- 部署前检查清单
- 应急响应机制

安全是智能合约开发的重中之重。开发者应该始终保持安全意识，遵循最佳实践，进行充分的测试和审计，确保合约的安全性和可靠性。

---

