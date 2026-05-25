# 第六章：Tolk语言开发实战

## 开场白

大家好，欢迎收听TON区块链开发教材的第六章。我是你们的主播。

上一章我们了解了TON的三种合约语言，今天这一章，我们要深入Tolk语言，从基础语法到高级特性，一步步掌握这门TON官方推荐的编程语言。

Tolk就像是区块链世界的C语言，它既给你足够的控制力，又不会让你陷入繁琐的底层细节。无论你是想开发DeFi协议、NFT市场，还是游戏合约，Tolk都能胜任。好了，让我们开始这段学习之旅吧。

## 第一节：基础语法

首先，我们来看看Tolk的基础语法。如果你学过C语言或者JavaScript，这部分会很容易理解。

### 数据类型

Tolk有几种基本的数据类型。首先是整数类型int，这是Tolk中最常用的类型。TON虚拟机处理的都是整数，可以是正数、负数，也可以是非常大的数。

然后是Cell类型。你可以把Cell想象成一个数据盒子，它可以装下各种数据，包括其他Cell。TON的所有数据最终都是存储在Cell中的，这就像是俄罗斯套娃，大盒子里可以装小盒子。

还有Slice和Builder类型。Slice用于从Cell中读取数据，Builder用于向Cell中写入数据。这就像是Cell是信封，Slice是拆信刀，Builder是写信的笔。

### 变量

在Tolk中声明变量很简单。你可以写：

int count = 0;

这就声明了一个整数变量count，初始值为0。

Tolk是强类型语言，每个变量都有明确的类型。你不能把一个整数赋值给Cell类型的变量，编译器会报错。这就像是你不能把苹果放进装橘子的箱子里。

### 运算符

Tolk的运算符和C语言很像。有加法、减法、乘法、除法，还有取模运算。比如：

int a = 10;
int b = 3;
int sum = a + b;      // 13
int product = a * b;  // 30
int remainder = a % b; // 1

还有比较运算符，比如等于、不等于、大于、小于。逻辑运算符包括与、或、非。这些都很直观，和大多数编程语言一样。

### 控制流

控制流就是程序的执行顺序。Tolk有if语句、while循环和repeat循环。

if语句用来做条件判断：

if (balance > 0) {
    // 有余额时执行这里
} else {
    // 没有余额时执行这里
}

while循环用于重复执行代码：

int i = 0;
while (i < 10) {
    // 重复执行10次
    i = i + 1;
}

repeat循环是TON特有的，用于固定次数的循环：

repeat(5) {
    // 重复执行5次
}

这些控制流语句就像是交通信号灯，控制程序该往哪里走。

## 第二节：合约结构

了解了基础语法，我们来看看Tolk合约的基本结构。

### contract声明

一个Tolk合约以contract关键字开头，后面跟着合约的名字。比如：

contract SimpleWallet {
    // 合约的内容写在这里
}

这就像是给合约起名字，告诉别人这个合约是做什么的。

### storage

storage是合约的永久存储空间。合约部署后，storage中的数据会一直保存在区块链上，直到合约被销毁。

在Tolk中，storage通常是一个Cell。你可以在里面存储各种状态，比如用户的余额、合约的配置参数等。

举个例子：

cell storage;

int owner;
int balance;

() load_data() {
    slice s = storage.begin_parse();
    owner = s~load_uint(256);
    balance = s~load_coins();
}

() save_data() {
    builder b = begin_cell();
    b = b.store_uint(owner, 256);
    b = b.store_coins(balance);
    storage = b.end_cell();
}

这里load_data函数从storage中读取数据，save_data函数把数据写回storage。每次修改状态后，都要调用save_data，否则修改不会保存。

### 消息处理

智能合约的核心功能是接收和处理消息。在Tolk中，消息处理函数是合约的入口点。

TON有两种主要的消息类型：内部消息和外部消息。

内部消息来自其他合约，外部消息来自外部世界，比如用户的交易。

### getter函数

getter函数用于查询合约的状态。它们不会修改storage，只是读取数据并返回。

比如：

int get_balance() method_id {
    load_data();
    return balance;
}

这个函数返回合约的余额。method_id标记表示这是一个getter函数，可以从外部调用。

## 第三节：消息处理详解

消息处理是智能合约最重要的部分。让我们深入了解onInternalMessage和onExternalMessage。

### onInternalMessage

onInternalMessage处理来自其他合约的消息。当有人向你的合约发送TON币，或者其他合约调用你的合约时，就会触发这个函数。

函数签名是这样的：

() onInternalMessage(int my_balance, int msg_value, cell in_msg_full, slice in_msg_body) {
    // 处理消息
}

参数解释：
- my_balance：合约当前的余额
- msg_value：随消息发送的TON币数量
- in_msg_full：完整的消息Cell
- in_msg_body：消息的主体内容

在函数内部，你可以解析消息内容，执行相应的逻辑，比如转账、更新状态等。

举个例子，假设我们收到一个转账消息：

() onInternalMessage(int my_balance, int msg_value, cell in_msg_full, slice in_msg_body) {
    load_data();
    
    int op = in_msg_body~load_uint(32);
    
    if (op == 1) {
        // 处理转账操作
        int amount = in_msg_body~load_coins();
        int recipient = in_msg_body~load_uint(256);
        
        // 执行转账逻辑
        send_transaction(recipient, amount);
    }
    
    save_data();
}

这里我们首先从消息体中读取操作码op，然后根据操作码执行不同的逻辑。

### onExternalMessage

onExternalMessage处理来自外部的消息，通常是由用户签名的交易触发。

函数签名：

() onExternalMessage(slice in_msg) {
    // 处理外部消息
}

外部消息通常用于初始化合约或者执行需要用户授权的操作。

需要注意的是，处理外部消息时要验证消息的签名，确保消息确实来自声称的发送者。这就像是检查身份证，确认对方的身份。

## 第四节：高级特性

掌握了基础，我们来看看Tolk的一些高级特性。

### Lazy Loading

Lazy Loading，也就是延迟加载，是一种优化技术。它的核心思想是：只在需要的时候才加载数据，而不是一开始就全部加载。

在TON中，存储是有成本的。如果你的合约存储了大量数据，每次调用都要加载全部数据，会很浪费gas。Lazy Loading让你可以按需加载，提高效率。

举个例子，假设你的合约存储了一个大数组。使用Lazy Loading，你可以只加载数组中需要的部分，而不是整个数组。

### 字典操作

字典，或者说哈希表，是一种常用的数据结构。在Tolk中，字典也是用Cell来存储的。

Tolk提供了丰富的字典操作函数：

- dict_set：设置键值对
- dict_get：根据键获取值
- dict_delete：删除键值对

比如：

cell balances_dict;

() set_balance(int user_id, int amount) {
    balances_dict = dict_set(balances_dict, 256, user_id, begin_cell().store_coins(amount).end_cell());
}

(int, slice) get_balance(int user_id) {
    (int success, slice value) = dict_get(balances_dict, 256, user_id);
    if (success) {
        return (true, value);
    }
    return (false, null());
}

这里我们用字典来存储用户的余额，键是用户ID，值是余额。

### 错误处理

错误处理对于智能合约非常重要，因为合约一旦部署就不能修改，bug可能会造成严重的损失。

Tolk提供了throw函数来抛出错误：

throw_if(100, condition);
throw_unless(101, condition);

throw_if在条件为真时抛出错误，throw_unless在条件为假时抛出错误。数字是错误码，方便调试时定位问题。

还有throw函数直接抛出错误：

if (balance < amount) {
    throw(102); // 余额不足
}

良好的错误处理可以让合约更安全，也更容易调试。

## 第五节：实战投票合约案例

好了，学了这么多，让我们来实战一下，实现一个简单的投票合约。

这个合约的功能是：用户可以投票给不同的选项，合约记录每个选项的票数，任何人都可以查询投票结果。

首先，定义合约结构：

contract VotingContract {
    cell storage;
    
    // 状态变量
    int owner;
    int voting_end_time;
    cell votes_dict;
    int total_votes;
}

然后，实现加载和保存数据的函数：

() load_data() {
    slice s = storage.begin_parse();
    owner = s~load_uint(256);
    voting_end_time = s~load_uint(64);
    votes_dict = s~load_ref();
    total_votes = s~load_uint(64);
}

() save_data() {
    builder b = begin_cell();
    b = b.store_uint(owner, 256);
    b = b.store_uint(voting_end_time, 64);
    b = b.store_ref(votes_dict);
    b = b.store_uint(total_votes, 64);
    storage = b.end_cell();
}

接下来，实现投票功能：

() onInternalMessage(int my_balance, int msg_value, cell in_msg_full, slice in_msg_body) {
    load_data();
    
    // 检查投票是否结束
    if (now() > voting_end_time) {
        throw(100); // 投票已结束
    }
    
    int op = in_msg_body~load_uint(32);
    
    if (op == 1) { // 投票操作
        int option_id = in_msg_body~load_uint(64);
        int voter_id = in_msg_body~load_uint(256);
        
        // 检查是否已经投票
        (int has_voted, _) = dict_get(votes_dict, 256, voter_id);
        throw_if(101, has_voted); // 已经投过票了
        
        // 记录投票
        votes_dict = dict_set(votes_dict, 256, voter_id, begin_cell().store_uint(option_id, 64).end_cell());
        total_votes = total_votes + 1;
    }
    
    save_data();
}

最后，实现查询函数：

(int) get_total_votes() method_id {
    load_data();
    return total_votes;
}

(int) has_voted(int voter_id) method_id {
    load_data();
    (int success, _) = dict_get(votes_dict, 256, voter_id);
    return success;
}

这个简单的投票合约展示了Tolk的基本用法：状态管理、消息处理、字典操作和错误处理。

## 总结与预告

今天我们深入学习了Tolk语言，从基础语法到高级特性，最后实现了一个完整的投票合约。

我们学习了Tolk的数据类型、变量、运算符和控制流；了解了合约的基本结构，包括storage、消息处理和getter函数；掌握了onInternalMessage和onExternalMessage的用法；还学习了Lazy Loading、字典操作和错误处理等高级特性。

在下一章，我们将学习Tact语言。Tact提供了更高级的抽象，让合约开发更加简单高效。我们会学习Tact的基础语法、合约结构、消息系统、Trait机制，最后实现一个多签钱包合约。

感谢收听，我们下一章再见。
