# 第七章：Tact语言开发实战

## 开场白

大家好，欢迎收听TON区块链开发教材的第七章。我是你们的主播。

上一章我们学习了Tolk语言，掌握了TON官方语言的基础和进阶用法。今天这一章，我们要进入一个更现代化的世界——Tact语言。

如果说Tolk像是C语言，那么Tact就像是TypeScript。它保留了强大的功能，同时提供了更友好的语法和更丰富的抽象。对于习惯了现代Web开发的程序员来说，Tact会让你有一种"回家的感觉"。

准备好了吗？让我们一起探索Tact的魅力。

## 第一节：基础语法

首先，我们来看看Tact的基础语法。好消息是，如果你学过TypeScript，这部分内容会非常熟悉。

### 数据类型

Tact有完善的数据类型系统。首先是基本类型：

Int表示整数，可以是任意大小的整数。这和JavaScript的number不同，Tact的Int可以精确表示非常大的数字，不会有精度问题。

Bool表示布尔值，只有true和false两个值。

Address表示区块链地址，这是TON特有的类型。你可以用它来存储用户地址或者合约地址。

Cell是TON的核心数据类型，就像我们在Tolk中学到的那样，它是一个可以嵌套的数据容器。

还有String用于文本，Builder和Slice用于操作Cell。

### 常量与变量

在Tact中，声明变量使用let关键字：

let count: Int = 0;
let name: String = "Hello Tact";
let isActive: Bool = true;

注意Tact有类型推断，如果你赋值的时候类型很明确，可以省略类型声明：

let count = 0;  // 编译器知道这是Int
let name = "Hello";  // 编译器知道这是String

常量的声明使用const：

const MAX_SUPPLY: Int = 1000000;

常量在编译时就确定了，不能修改。这适合定义一些配置参数，比如最大供应量、手续费比例等。

### 控制流

Tact的控制流和大多数现代语言类似。

if语句用于条件判断：

if (balance > 0) {
    // 有余额时执行
} else if (balance == 0) {
    // 余额为零时执行
} else {
    // 其他情况
}

while循环用于重复执行：

let i: Int = 0;
while (i < 10) {
    // 重复10次
    i = i + 1;
}

repeat循环是TON特有的：

repeat(5) {
    // 重复5次
}

还有try-catch用于错误处理，这个我们后面会详细讲。

## 第二节：合约结构

了解了基础语法，我们来看看Tact合约的结构。

### contract声明

Tact合约以contract关键字开头，后面是合约名：

contract SimpleWallet {
    // 合约内容
}

这和Tolk很像，但Tact的合约可以包含更多的声明，比如状态变量、初始化函数、消息处理器等。

### init函数

init函数是合约的构造函数，在合约部署时执行一次。你可以在这里初始化状态变量。

比如：

contract SimpleWallet {
    owner: Address;
    balance: Int;

    init(owner: Address) {
        self.owner = owner;
        self.balance = 0;
    }
}

这里我们定义了两个状态变量owner和balance，在init函数中初始化它们。self关键字表示合约实例本身，类似于其他语言中的this。

### receive函数

receive函数用于接收和处理消息。在Tact中，你可以为不同类型的消息定义不同的接收函数。

比如：

message Deposit {
    amount: Int;
}

message Withdraw {
    amount: Int;
    to: Address;
}

contract SimpleWallet {
    // ... 状态变量和init函数

    receive(msg: Deposit) {
        self.balance = self.balance + msg.amount;
    }

    receive(msg: Withdraw) {
        require(self.balance >= msg.amount, "Insufficient balance");
        self.balance = self.balance - msg.amount;
        // 发送资金
    }
}

这里我们定义了两种消息类型：Deposit表示存款，Withdraw表示取款。合约根据收到的消息类型，自动调用对应的receive函数。

这种写法比Tolk清晰多了，你不需要手动解析消息体，Tact会自动帮你处理。

### getter函数

getter函数用于查询合约状态，和Tolk的method_id函数类似：

get fun getBalance(): Int {
    return self.balance;
}

get fun getOwner(): Address {
    return self.owner;
}

get fun关键字表示这是一个getter函数，可以从外部调用，不会修改合约状态。

## 第三节：消息与交互

消息是智能合约之间交互的基础。Tact让消息处理变得非常简单。

### message类型

在Tact中，你可以用message关键字定义消息类型：

message Transfer {
    to: Address;
    amount: Int;
    memo: String?;  // ?表示可选字段
}

message Mint {
    recipient: Address;
    amount: Int;
}

这就像是在定义数据结构，清晰明了。

### SendParameters

当你需要向其他合约发送消息时，使用SendParameters：

cell msg = Transfer{
    to: recipient,
    amount: 100,
    memo: "Payment"
}.toCell();

send(SendParameters{
    to: recipient,
    value: ton("0.05"),
    body: msg
});

这里ton("0.05")表示0.05个TON币。Tact提供了ton函数来方便地表示金额。

SendParameters还有很多其他参数，比如：
- bounce：如果目标合约不存在，是否退回消息
- mode：发送模式，比如是否携带剩余gas
- code和data：如果是部署合约，可以在这里提供合约代码和初始数据

### context对象

在处理消息时，你可以通过context对象获取消息的上下文信息：

receive(msg: SomeMessage) {
    let ctx = context();
    let sender = ctx.sender;  // 发送者地址
    let value = ctx.value;    // 随消息发送的TON币数量
    
    // 处理消息
}

context对象包含了消息的各种元信息，比如发送者地址、消息价值、原始消息Cell等。

## 第四节：Trait与组合

Trait是Tact最强大的特性之一，它让代码复用变得非常简单。

### 什么是Trait

Trait可以理解为一种"能力"或者"特性"。比如，一个合约可以有"可拥有"的特性，可以有"可暂停"的特性，可以有"可升级"的特性。

在Tact中，你可以定义trait，然后在合约中使用它：

trait Ownable {
    owner: Address;

    fun requireOwner() {
        require(context().sender == self.owner, "Not owner");
    }

    get fun getOwner(): Address {
        return self.owner;
    }
}

这里我们定义了一个Ownable trait，它包含一个owner状态变量，一个requireOwner函数用于检查调用者是否是所有者，还有一个getter函数。

### 使用Trait

在合约中使用trait很简单：

contract MyToken with Ownable {
    totalSupply: Int;

    init(owner: Address) {
        self.owner = owner;
        self.totalSupply = 0;
    }

    receive(msg: Mint) {
        self.requireOwner();  // 使用trait中的函数
        self.totalSupply = self.totalSupply + msg.amount;
    }
}

contract关键字后面的with Ownable表示这个合约使用了Ownable trait。合约自动获得了trait中定义的所有状态变量和函数。

这就像是在说：我的代币合约具有"可拥有"的特性，所以它有owner，可以检查所有者权限。

### 组合多个Trait

一个合约可以使用多个trait：

trait Pausable {
    paused: Bool;

    fun requireNotPaused() {
        require(!self.paused, "Contract is paused");
    }

    receive("pause") {
        self.requireOwner();
        self.paused = true;
    }

    receive("unpause") {
        self.requireOwner();
        self.paused = false;
    }
}

contract MyContract with Ownable, Pausable {
    // 这个合约既有Ownable特性，又有Pausable特性
}

这种组合式编程让代码非常模块化。你可以把通用的功能封装成trait，然后在不同的合约中复用。

## 第五节：高级特性

掌握了基础，我们来看看Tact的一些高级特性。

### Map操作

Map是Tact中的字典类型，用于存储键值对：

contract Token {
    balances: map<Address, Int>;
    totalSupply: Int;

    fun balanceOf(account: Address): Int {
        if (self.balances.exists(account)) {
            return self.balances.get(account)!!;
        }
        return 0;
    }

    fun transfer(from: Address, to: Address, amount: Int) {
        let fromBalance = self.balanceOf(from);
        require(fromBalance >= amount, "Insufficient balance");
        
        self.balances.set(from, fromBalance - amount);
        let toBalance = self.balanceOf(to);
        self.balances.set(to, toBalance + amount);
    }
}

Map提供了set、get、exists等方法，操作非常直观。!!操作符表示"我确定这个值存在"，如果不存在会抛出错误。

### 序列化

在TON中，数据需要被序列化成Cell才能存储和传输。Tact会自动处理大部分序列化工作，但你也可以自定义。

比如：

message CustomMessage {
    id: Int as uint32;
    data: Cell;
    flags: Int as uint8;
}

as关键字用于指定序列化格式。uint32表示32位无符号整数，uint8表示8位无符号整数。

Tact会自动把CustomMessage序列化成Cell，你只需要调用toCell()方法。

### 合约升级

合约升级是一个高级话题。在TON中，合约代码一旦部署就不能修改，但你可以通过代理模式实现升级。

基本思路是：用户和一个代理合约交互，代理合约把调用转发给实际的逻辑合约。当需要升级时，只需要部署新的逻辑合约，然后让代理合约指向新的地址。

Tact可以通过trait来封装这种升级逻辑：

trait Upgradable {
    implementation: Address;

    fun setImplementation(newImpl: Address) {
        self.requireOwner();
        self.implementation = newImpl;
    }

    receive(msg: ForwardMessage) {
        // 转发消息到实现合约
        send(SendParameters{
            to: self.implementation,
            value: context().value,
            body: msg.toCell()
        });
    }
}

这样，任何使用这个trait的合约都具有了升级能力。

## 第六节：实战多签钱包案例

好了，学了这么多，让我们来实现一个实用的多签钱包合约。

多签钱包的意思是：需要多个人的签名才能执行转账。这在企业资金管理、团队项目中非常有用。

我们的多签钱包功能：
- 可以设置多个所有者
- 转账需要一定数量的签名
- 任何人都可以发起转账请求
- 其他所有者可以批准或拒绝

首先，定义消息类型：

message SubmitTransaction {
    to: Address;
    value: Int;
}

message ConfirmTransaction {
    txId: Int;
}

message ExecuteTransaction {
    txId: Int;
}

然后，定义合约：

contract MultiSigWallet {
    owners: map<Address, Bool>;
    requiredConfirmations: Int;
    transactions: map<Int, Transaction>;
    confirmations: map<Int, map<Address, Bool>>;
    transactionCount: Int;

    struct Transaction {
        to: Address;
        value: Int;
        confirmations: Int;
        executed: Bool;
    }

    init(owners: map<Address, Bool>, required: Int) {
        self.owners = owners;
        self.requiredConfirmations = required;
        self.transactionCount = 0;
    }

    fun isOwner(addr: Address): Bool {
        return self.owners.exists(addr) && self.owners.get(addr)!!;
    }

    receive(msg: SubmitTransaction) {
        require(self.isOwner(context().sender), "Not owner");
        
        let txId = self.transactionCount;
        self.transactions.set(txId, Transaction{
            to: msg.to,
            value: msg.value,
            confirmations: 1,
            executed: false
        });
        
        // 记录第一个确认
        let confs: map<Address, Bool> = emptyMap();
        confs.set(context().sender, true);
        self.confirmations.set(txId, confs);
        
        self.transactionCount = self.transactionCount + 1;
        
        // 如果只需要一个确认，直接执行
        if (self.requiredConfirmations == 1) {
            self.executeTransaction(txId);
        }
    }

    receive(msg: ConfirmTransaction) {
        require(self.isOwner(context().sender), "Not owner");
        require(self.transactions.exists(msg.txId), "Transaction not found");
        
        let tx = self.transactions.get(msg.txId)!!;
        require(!tx.executed, "Already executed");
        
        let confs = self.confirmations.get(msg.txId)!!;
        require(!confs.exists(context().sender), "Already confirmed");
        
        confs.set(context().sender, true);
        self.confirmations.set(msg.txId, confs);
        
        tx.confirmations = tx.confirmations + 1;
        self.transactions.set(msg.txId, tx);
        
        // 如果确认数达到要求，执行交易
        if (tx.confirmations >= self.requiredConfirmations) {
            self.executeTransaction(msg.txId);
        }
    }

    fun executeTransaction(txId: Int) {
        let tx = self.transactions.get(txId)!!;
        require(!tx.executed, "Already executed");
        require(tx.confirmations >= self.requiredConfirmations, "Not enough confirmations");
        
        tx.executed = true;
        self.transactions.set(txId, tx);
        
        // 发送资金
        send(SendParameters{
            to: tx.to,
            value: tx.value,
            mode: SendRemainingValue
        });
    }

    get fun getTransaction(txId: Int): Transaction? {
        return self.transactions.get(txId);
    }

    get fun getTransactionCount(): Int {
        return self.transactionCount;
    }
}

这个多签钱包展示了Tact的许多特性：Map的使用、结构体定义、消息处理、条件判断等。

## 总结与预告

今天我们深入学习了Tact语言，从基础语法到高级特性，最后实现了一个完整的多签钱包合约。

我们学习了Tact的数据类型、变量声明和控制流；了解了合约的基本结构，包括init函数、receive函数和getter函数；掌握了消息定义、SendParameters和context对象的使用；学习了强大的Trait机制，让代码复用变得简单；还了解了Map操作、序列化和合约升级等高级特性。

Tact的现代化语法和丰富的抽象，让智能合约开发变得更加高效和愉快。如果你是Web开发者，Tact会让你快速进入区块链开发的世界。

在下一章，我们将学习TON合约的测试和部署。我们会了解如何编写测试用例，如何使用Blueprint框架，以及如何将合约部署到主网。这是从开发到生产的关键一步，千万不要错过。

感谢收听，我们下一章再见。
