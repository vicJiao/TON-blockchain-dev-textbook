# TON 区块链开发教材 - 进阶篇

> 本文档包含 TON 区块链开发的高级主题，涵盖高级合约模式、DeFi 开发、节点运维和性能优化。

---

## 第五篇：进阶篇 —— 高级主题

---

## 第 16 章：高级合约模式

本章介绍 TON 智能合约开发中的高级设计模式，帮助开发者构建更复杂、更安全的去中心化应用。

### 16.1 代理合约模式

代理合约模式是一种常见的设计模式，用于实现合约升级、Gas 优化和批量操作等功能。

#### 16.1.1 代理合约设计原理

代理合约的核心思想是将业务逻辑与数据存储分离：

```
┌─────────────────────────────────────────────────────────────┐
│                   代理合约模式架构                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   用户/外部调用                                               │
│        │                                                    │
│        ↓                                                    │
│   ┌─────────────┐     ┌─────────────┐     ┌─────────────┐  │
│   │  代理合约    │────→│  实现合约 A  │     │  实现合约 B  │  │
│   │  (Proxy)    │     │ (Logic V1)  │     │ (Logic V2)  │  │
│   └─────────────┘     └─────────────┘     └─────────────┘  │
│        │                                                    │
│        │ 存储数据                                            │
│        ↓                                                    │
│   ┌─────────────┐                                          │
│   │  存储合约    │                                          │
│   │ (Storage)   │                                          │
│   └─────────────┘                                          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**代理合约的优势：**

1. **可升级性**：业务逻辑可以更新而无需迁移数据
2. **Gas 优化**：多个代理可以共享同一个实现合约
3. **代码复用**：通用逻辑可以集中管理
4. **安全隔离**：数据和逻辑分离降低风险

#### 16.1.2 消息转发与费用优化

代理合约最常见的用途是消息转发和 Gas 站模式：

```tact
import "@stdlib/deploy";

// Gas 站代理合约 - 为用户代付 Gas
contract GasStationProxy with Deployable {
    owner: Address;
    relayers: map<Address, Bool>;  // 授权的转发者
    
    init(owner: Address) {
        self.owner = owner;
    }
    
    // 添加转发者
    receive(msg: AddRelayer) {
        require(sender() == self.owner, "Only owner");
        self.relayers.set(msg.relayer, true);
    }
    
    // 代理转发消息
    receive(msg: ProxyMessage) {
        // 验证转发者
        let isRelayer: Bool = self.relayers.get(sender()) ?: false;
        require(isRelayer, "Unauthorized relayer");
        
        // 转发消息到目标合约
        send(SendParameters{
            to: msg.target,
            value: msg.value,
            mode: SendPayGasSeparately,
            body: msg.payload
        });
    }
    
    // 提取余额
    receive(msg: Withdraw) {
        require(sender() == self.owner, "Only owner");
        send(SendParameters{
            to: self.owner,
            value: myBalance() - ton("0.1"),  // 保留少量余额
            mode: SendPayGasSeparately
        });
    }
}

// 代理消息结构
message ProxyMessage {
    target: Address;
    value: Int as coins;
    payload: Cell;
}

message AddRelayer {
    relayer: Address;
}

message Withdraw {
    amount: Int as coins;
}
```

#### 16.1.3 应用场景

**场景一：批量操作代理**

```tact
// 批量转账代理
contract BatchTransferProxy with Deployable {
    owner: Address;
    
    init(owner: Address) {
        self.owner = owner;
    }
    
    receive(msg: BatchTransfer) {
        require(sender() == self.owner, "Only owner");
        
        // 验证总余额足够
        let totalAmount: Int = 0;
        foreach (recipient, amount in msg.transfers) {
            totalAmount = totalAmount + amount;
        }
        require(myBalance() >= totalAmount + ton("0.5"), "Insufficient balance");
        
        // 批量发送
        foreach (recipient, amount in msg.transfers) {
            send(SendParameters{
                to: recipient,
                value: amount,
                mode: SendPayGasSeparately
            });
        }
    }
}

// 批量转账消息
message BatchTransfer {
    transfers: map<Address, Int>;
}
```

**场景二：访问控制代理**

```tact
// 访问控制代理 - 限制对目标合约的访问
contract AccessControlProxy with Deployable {
    owner: Address;
    target: Address;
    whitelist: map<Address, Bool>;
    paused: Bool = false;
    
    init(owner: Address, target: Address) {
        self.owner = owner;
        self.target = target;
    }
    
    // 管理白名单
    receive(msg: UpdateWhitelist) {
        require(sender() == self.owner, "Only owner");
        self.whitelist.set(msg.user, msg.allowed);
    }
    
    // 暂停/恢复
    receive(msg: SetPaused) {
        require(sender() == self.owner, "Only owner");
        self.paused = msg.paused;
    }
    
    // 代理转发（带权限检查）
    receive(msg: ProxyCall) {
        require(!self.paused, "Contract paused");
        require(self.whitelist.get(sender()) ?: false, "Not whitelisted");
        
        send(SendParameters{
            to: self.target,
            value: msg.value,
            mode: SendPayGasSeparately,
            body: msg.payload
        });
    }
}

message UpdateWhitelist {
    user: Address;
    allowed: Bool;
}

message SetPaused {
    paused: Bool;
}

message ProxyCall {
    value: Int as coins;
    payload: Cell;
}
```

### 16.2 合约升级模式

合约升级是智能合约开发中的重要话题，TON 提供了多种升级策略。

#### 16.2.1 代码分离模式

```tact
// 可升级合约基类
trait Upgradable {
    owner: Address;
    codeVersion: Int = 1;
    
    // 升级代码
    receive(msg: UpgradeCode) {
        require(sender() == self.owner, "Only owner can upgrade");
        
        // 发送升级消息给自己
        send(SendParameters{
            to: myAddress(),
            value: 0,
            mode: SendRemainingValue + SendDestroyIfZero,
            code: msg.newCode,  // 新代码
            data: self.getStorageData()  // 当前数据
        });
    }
    
    // 获取存储数据（子类需要重写）
    virtual fun getStorageData(): Cell {
        return beginCell().endCell();
    }
    
    get fun getCodeVersion(): Int {
        return self.codeVersion;
    }
}

// 使用示例
contract MyUpgradableContract with Upgradable {
    owner: Address;
    codeVersion: Int = 1;
    counter: Int = 0;
    data: map<Address, Int>;
    
    init(owner: Address) {
        self.owner = owner;
    }
    
    // 重写获取存储数据方法
    override fun getStorageData(): Cell {
        return beginCell()
            .storeAddress(self.owner)
            .storeInt(self.codeVersion + 1, 32)  // 新版本号
            .storeInt(self.counter, 64)
            .endCell();
    }
    
    receive(msg: Increment) {
        self.counter = self.counter + 1;
    }
    
    get fun getCounter(): Int {
        return self.counter;
    }
}

message UpgradeCode {
    newCode: Cell;
}

message Increment {}
```

#### 16.2.2 存储布局兼容性

升级合约时，存储布局的兼容性至关重要：

```tact
// V1 版本存储布局
contract ContractV1 {
    owner: Address;      // 偏移 0
    counter: Int;        // 偏移 1
}

// V2 版本存储布局（向后兼容）
contract ContractV2 {
    owner: Address;      // 偏移 0（保持不变）
    counter: Int;        // 偏移 1（保持不变）
    newField: Int;       // 偏移 2（新增）
    
    init() {
        self.newField = 0;  // 新字段默认值
    }
}

// 不兼容的升级（错误示例）
contract ContractV2Bad {
    counter: Int;        // ❌ 偏移 0 变了！
    owner: Address;      // ❌ 偏移 1 变了！
}
```

**存储布局最佳实践：**

1. **只追加新字段**：永远在末尾添加新字段
2. **保留占位符**：为未来扩展预留空间
3. **使用版本号**：跟踪合约版本
4. **迁移函数**：必要时添加数据迁移逻辑

```tact
// 带迁移逻辑的可升级合约
contract UpgradableWithMigration {
    owner: Address;
    version: Int = 1;
    
    // 旧数据字段
    oldData: Int;
    
    // 新数据字段
    newData: Int = 0;
    migrated: Bool = false;
    
    // 迁移函数
    receive(msg: MigrateData) {
        require(sender() == self.owner, "Only owner");
        require(!self.migrated, "Already migrated");
        
        // 执行数据迁移
        self.newData = self.oldData * 2;  // 示例：数据转换
        self.migrated = true;
        self.version = 2;
    }
    
    get fun getVersion(): Int {
        return self.version;
    }
}

message MigrateData {}
```

#### 16.2.3 升级安全考量

```tact
// 安全升级合约 - 带时间锁
contract SecureUpgradableContract {
    owner: Address;
    pendingOwner: Address?;
    codeVersion: Int = 1;
    
    // 升级时间锁
    upgradeScheduledAt: Int = 0;
    pendingUpgradeCode: Cell?;
    
    const UPGRADE_DELAY: Int = 86400;  // 24小时延迟
    
    init(owner: Address) {
        self.owner = owner;
    }
    
    // 第一步：计划升级
    receive(msg: ScheduleUpgrade) {
        require(sender() == self.owner, "Only owner");
        require(self.pendingUpgradeCode == null, "Upgrade already scheduled");
        
        self.upgradeScheduledAt = now();
        self.pendingUpgradeCode = msg.newCode;
    }
    
    // 第二步：执行升级（需要等待延迟期）
    receive(msg: ExecuteUpgrade) {
        require(sender() == self.owner, "Only owner");
        require(self.pendingUpgradeCode != null, "No upgrade scheduled");
        require(now() >= self.upgradeScheduledAt + self.UPGRADE_DELAY, "Delay not passed");
        
        let newCode: Cell = self.pendingUpgradeCode!!;
        self.pendingUpgradeCode = null;
        
        // 执行升级
        send(SendParameters{
            to: myAddress(),
            value: 0,
            mode: SendRemainingValue + SendDestroyIfZero,
            code: newCode,
            data: self.getStorageData()
        });
    }
    
    // 取消升级
    receive(msg: CancelUpgrade) {
        require(sender() == self.owner, "Only owner");
        self.pendingUpgradeCode = null;
        self.upgradeScheduledAt = 0;
    }
    
    get fun getUpgradeStatus(): UpgradeStatus {
        return UpgradeStatus{
            scheduled: self.pendingUpgradeCode != null,
            scheduledAt: self.upgradeScheduledAt,
            canExecuteAt: self.upgradeScheduledAt + self.UPGRADE_DELAY
        };
    }
}

message ScheduleUpgrade {
    newCode: Cell;
}

message ExecuteUpgrade {}

message CancelUpgrade {}

struct UpgradeStatus {
    scheduled: Bool;
    scheduledAt: Int;
    canExecuteAt: Int;
}
```

### 16.3 高负载合约设计

高负载场景需要特殊的合约设计来优化性能和成本。

#### 16.3.1 高负载钱包

```tact
// 高负载钱包 - 支持批量操作
contract HighLoadWallet {
    owner: Address;
    publicKey: Int;
    subWalletId: Int;
    
    // 批量消息队列
    messageQueue: map<Int, QueuedMessage>;
    queueSize: Int = 0;
    
    init(publicKey: Int, subWalletId: Int) {
        self.owner = sender();
        self.publicKey = publicKey;
        self.subWalletId = subWalletId;
    }
    
    // 添加消息到队列
    receive(msg: QueueMessage) {
        require(sender() == self.owner, "Only owner");
        
        self.messageQueue.set(self.queueSize, QueuedMessage{
            target: msg.target,
            value: msg.value,
            payload: msg.payload,
            sendMode: msg.sendMode
        });
        self.queueSize = self.queueSize + 1;
    }
    
    // 批量发送队列中的消息
    receive(msg: FlushQueue) {
        require(sender() == self.owner, "Only owner");
        
        let processed: Int = 0;
        let maxBatch: Int = msg.batchSize;
        
        // 批量处理（限制每批数量以避免 Gas 超限）
        while (processed < maxBatch && self.queueSize > 0) {
            self.queueSize = self.queueSize - 1;
            let queuedMsg: QueuedMessage = self.messageQueue.get(self.queueSize)!!;
            
            send(SendParameters{
                to: queuedMsg.target,
                value: queuedMsg.value,
                mode: queuedMsg.sendMode,
                body: queuedMsg.payload
            });
            
            // 删除已处理的消息
            self.messageQueue.del(self.queueSize);
            processed = processed + 1;
        }
        
        // 如果还有消息，发送继续消息
        if (self.queueSize > 0) {
            send(SendParameters{
                to: myAddress(),
                value: ton("0.01"),
                mode: SendPayGasSeparately,
                body: FlushQueue{batchSize: maxBatch}.toCell()
            });
        }
    }
    
    get fun getQueueSize(): Int {
        return self.queueSize;
    }
}

struct QueuedMessage {
    target: Address;
    value: Int as coins;
    payload: Cell?;
    sendMode: Int;
}

message QueueMessage {
    target: Address;
    value: Int as coins;
    payload: Cell? = null;
    sendMode: Int = 0;
}

message FlushQueue {
    batchSize: Int = 100;
}
```

#### 16.3.2 消息队列与批处理

```tact
// 批处理器合约
contract BatchProcessor {
    owner: Address;
    
    // 批处理配置
    batchConfig: BatchConfig;
    
    // 待处理任务队列
    pendingTasks: map<Int, Task>;
    taskCounter: Int = 0;
    
    init(owner: Address) {
        self.owner = owner;
        self.batchConfig = BatchConfig{
            maxBatchSize: 50,
            minBatchSize: 10,
            processingInterval: 300  // 5分钟
        };
    }
    
    // 提交任务
    receive(msg: SubmitTask) {
        self.pendingTasks.set(self.taskCounter, Task{
            id: self.taskCounter,
            target: msg.target,
            data: msg.data,
            submittedAt: now(),
            priority: msg.priority
        });
        self.taskCounter = self.taskCounter + 1;
        
        // 检查是否达到批处理阈值
        self.checkAndProcessBatch();
    }
    
    // 手动触发批处理
    receive(msg: ProcessBatch) {
        require(sender() == self.owner, "Only owner");
        self.processBatch(msg.batchSize);
    }
    
    // 检查并处理批次
    fun checkAndProcessBatch() {
        let pendingCount: Int = self.taskCounter;
        if (pendingCount >= self.batchConfig.minBatchSize) {
            self.processBatch(self.batchConfig.maxBatchSize);
        }
    }
    
    // 处理批次
    fun processBatch(maxSize: Int) {
        let processed: Int = 0;
        let taskIds: map<Int, Int>;  // 存储要处理的任务ID
        let idCount: Int = 0;
        
        // 收集任务（简化示例，实际可能需要按优先级排序）
        foreach (id, task in self.pendingTasks) {
            if (processed < maxSize) {
                taskIds.set(idCount, id);
                idCount = idCount + 1;
                processed = processed + 1;
            }
        }
        
        // 处理任务
        foreach (idx, taskId in taskIds) {
            let task: Task = self.pendingTasks.get(taskId)!!;
            
            // 执行任务
            send(SendParameters{
                to: task.target,
                value: ton("0.01"),
                mode: SendPayGasSeparately,
                body: task.data
            });
            
            // 删除已处理的任务
            self.pendingTasks.del(taskId);
        }
    }
    
    get fun getPendingCount(): Int {
        return self.taskCounter;
    }
}

struct BatchConfig {
    maxBatchSize: Int;
    minBatchSize: Int;
    processingInterval: Int;
}

struct Task {
    id: Int;
    target: Address;
    data: Cell;
    submittedAt: Int;
    priority: Int;
}

message SubmitTask {
    target: Address;
    data: Cell;
    priority: Int = 0;
}

message ProcessBatch {
    batchSize: Int;
}
```

### 16.4 跨合约通信

#### 16.4.1 合约间消息传递模式

```tact
// 回调模式
contract ContractA {
    b: Address;
    pendingCallbacks: map<Int, CallbackInfo>;
    callbackId: Int = 0;
    
    init(b: Address) {
        self.b = b;
    }
    
    // 调用合约 B 并期待回调
    fun callBWithCallback(data: Cell) {
        let cbId: Int = self.callbackId;
        self.callbackId = self.callbackId + 1;
        
        // 记录回调信息
        self.pendingCallbacks.set(cbId, CallbackInfo{
            createdAt: now(),
            originalSender: sender()
        });
        
        // 发送消息给 B，包含回调ID
        send(SendParameters{
            to: self.b,
            value: ton("0.05"),
            mode: SendPayGasSeparately,
            body: RequestWithCallback{
                callbackId: cbId,
                data: data,
                callbackAddress: myAddress()
            }.toCell()
        });
    }
    
    // 处理回调
    receive(msg: CallbackResponse) {
        require(sender() == self.b, "Invalid sender");
        
        let callbackInfo: CallbackInfo = self.pendingCallbacks.get(msg.callbackId)!!;
        self.pendingCallbacks.del(msg.callbackId);
        
        // 处理回调结果
        self.handleCallbackResult(msg.result, callbackInfo);
    }
    
    fun handleCallbackResult(result: Cell, info: CallbackInfo) {
        // 处理结果
    }
}

struct CallbackInfo {
    createdAt: Int;
    originalSender: Address;
}

message RequestWithCallback {
    callbackId: Int;
    data: Cell;
    callbackAddress: Address;
}

message CallbackResponse {
    callbackId: Int;
    result: Cell;
}

// 合约 B
contract ContractB {
    receive(msg: RequestWithCallback) {
        // 处理请求
        let result: Cell = self.processRequest(msg.data);
        
        // 发送回调
        send(SendParameters{
            to: msg.callbackAddress,
            value: ton("0.01"),
            mode: SendPayGasSeparately,
            body: CallbackResponse{
                callbackId: msg.callbackId,
                result: result
            }.toCell()
        });
    }
    
    fun processRequest(data: Cell): Cell {
        // 处理逻辑
        return data;
    }
}
```

#### 16.4.2 错误传播与处理

```tact
// 带错误处理的跨合约调用
trait ErrorHandler {
    // 错误码定义
    const ERROR_INVALID_INPUT: Int = 100;
    const ERROR_INSUFFICIENT_FUNDS: Int = 101;
    const ERROR_UNAUTHORIZED: Int = 102;
    const ERROR_TIMEOUT: Int = 103;
    
    // 发送带错误处理的消息
    fun sendWithErrorHandling(
        target: Address,
        value: Int,
        body: Cell,
        errorHandler: Address
    ) {
        send(SendParameters{
            to: target,
            value: value,
            mode: SendPayGasSeparately,
            body: beginCell()
                .storeRef(body)
                .storeAddress(errorHandler)
                .endCell()
        });
    }
}

// 错误处理合约
contract ErrorHandlerContract with ErrorHandler {
    errorLog: map<Int, ErrorRecord>;
    errorCount: Int = 0;
    
    // 接收错误报告
    receive(msg: ErrorReport) {
        self.errorLog.set(self.errorCount, ErrorRecord{
            timestamp: now(),
            source: sender(),
            errorCode: msg.errorCode,
            errorMessage: msg.message,
            originalOperation: msg.originalOperation
        });
        self.errorCount = self.errorCount + 1;
        
        // 可以在这里添加告警逻辑
        self.handleError(msg);
    }
    
    fun handleError(msg: ErrorReport) {
        // 根据错误类型采取不同措施
        if (msg.errorCode == self.ERROR_INSUFFICIENT_FUNDS) {
            // 资金不足，可能需要充值提醒
        } else if (msg.errorCode == self.ERROR_TIMEOUT) {
            // 超时，可能需要重试
        }
    }
    
    get fun getErrorCount(): Int {
        return self.errorCount;
    }
}

struct ErrorRecord {
    timestamp: Int;
    source: Address;
    errorCode: Int;
    errorMessage: String;
    originalOperation: Cell;
}

message ErrorReport {
    errorCode: Int;
    message: String;
    originalOperation: Cell;
}
```

---

**本章小结：**

本章介绍了 TON 智能合约的高级设计模式：
- **代理合约模式**：实现 Gas 站、批量操作和访问控制
- **合约升级模式**：代码分离、存储兼容性、安全升级
- **高负载合约设计**：消息队列、批处理、高负载钱包
- **跨合约通信**：回调模式、错误处理、竞态条件防范

掌握这些高级模式可以帮助开发者构建更复杂、更可靠的 TON 应用。

---

## 第 17 章：DeFi 合约开发

本章介绍如何在 TON 上构建去中心化金融（DeFi）应用，包括 AMM、DEX 和质押协议。

### 17.1 AMM（自动做市商）原理

#### 17.1.1 恒定乘积做市商

AMM 的核心是恒定乘积公式：`x * y = k`

```
┌─────────────────────────────────────────────────────────────┐
│                  AMM 恒定乘积公式                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   流动性池：                                                │
│   ┌─────────────────────────────────────┐                  │
│   │  Token A (x)  │  Token B (y)        │                  │
│   │     1000      │     1000            │                  │
│   │              k = 1,000,000          │                  │
│   └─────────────────────────────────────┘                  │
│                                                             │
│   交易后：                                                  │
│   ┌─────────────────────────────────────┐                  │
│   │  Token A (x') │  Token B (y')       │                  │
│   │     1100      │     909.09          │                  │
│   │              k = 1,000,000          │                  │
│   └─────────────────────────────────────┘                  │
│                                                             │
│   计算：                                                    │
│   x' = x + Δx = 1000 + 100 = 1100                          │
│   y' = k / x' = 1,000,000 / 1100 = 909.09                  │
│   Δy = y - y' = 1000 - 909.09 = 90.91                      │
│                                                             │
│   价格：                                                    │
│   价格 = y / x = 909.09 / 1100 ≈ 0.826                     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### 17.1.2 流动性池设计

```tact
// 流动性池合约
contract LiquidityPool {
    // 池子状态
    tokenA: Address;  // 代币 A 地址
    tokenB: Address;  // 代币 B 地址
    
    reserveA: Int as coins = 0;  // 代币 A 储备
    reserveB: Int as coins = 0;  // 代币 B 储备
    
    totalSupply: Int as coins = 0;  // LP 代币总供应量
    lpBalances: map<Address, Int>;  // LP 代币余额
    
    // 费用参数
    feeNumerator: Int = 3;    // 0.3% 手续费
    feeDenominator: Int = 1000;
    
    init(tokenA: Address, tokenB: Address) {
        self.tokenA = tokenA;
        self.tokenB = tokenB;
    }
    
    // 添加流动性
    receive(msg: AddLiquidity) {
        let amountA: Int = msg.amountA;
        let amountB: Int = msg.amountB;
        let sender: Address = sender();
        
        // 计算 LP 代币数量
        let lpTokens: Int = 0;
        if (self.totalSupply == 0) {
            // 首次添加流动性
            lpTokens = sqrt(amountA * amountB);
        } else {
            // 按比例计算
            let lpA: Int = (amountA * self.totalSupply) / self.reserveA;
            let lpB: Int = (amountB * self.totalSupply) / self.reserveB;
            lpTokens = min(lpA, lpB);
        }
        
        require(lpTokens > 0, "Insufficient liquidity");
        
        // 更新储备
        self.reserveA = self.reserveA + amountA;
        self.reserveB = self.reserveB + amountB;
        
        // 铸造 LP 代币
        self.totalSupply = self.totalSupply + lpTokens;
        let currentBalance: Int = self.lpBalances.get(sender) ?: 0;
        self.lpBalances.set(sender, currentBalance + lpTokens);
        
        // 发送 LP 代币给用户（这里简化处理，实际可能需要转账）
    }
    
    // 移除流动性
    receive(msg: RemoveLiquidity) {
        let lpAmount: Int = msg.lpAmount;
        let sender: Address = sender();
        
        let userLp: Int = self.lpBalances.get(sender) ?: 0;
        require(userLp >= lpAmount, "Insufficient LP tokens");
        
        // 计算可提取的代币数量
        let amountA: Int = (lpAmount * self.reserveA) / self.totalSupply;
        let amountB: Int = (lpAmount * self.reserveB) / self.totalSupply;
        
        // 更新储备
        self.reserveA = self.reserveA - amountA;
        self.reserveB = self.reserveB - amountB;
        
        // 销毁 LP 代币
        self.totalSupply = self.totalSupply - lpAmount;
        self.lpBalances.set(sender, userLp - lpAmount);
        
        // 返还代币给用户
        send(SendParameters{
            to: sender,
            value: ton("0.01"),
            mode: SendPayGasSeparately
        });
    }
    
    // 交换代币
    receive(msg: Swap) {
        let amountIn: Int = msg.amountIn;
        let tokenIn: Address = msg.tokenIn;
        let minAmountOut: Int = msg.minAmountOut;
        
        require(tokenIn == self.tokenA || tokenIn == self.tokenB, "Invalid token");
        
        // 计算输出金额
        let amountOut: Int = 0;
        if (tokenIn == self.tokenA) {
            amountOut = self.getAmountOut(amountIn, self.reserveA, self.reserveB);
            require(amountOut >= minAmountOut, "Slippage exceeded");
            
            self.reserveA = self.reserveA + amountIn;
            self.reserveB = self.reserveB - amountOut;
        } else {
            amountOut = self.getAmountOut(amountIn, self.reserveB, self.reserveA);
            require(amountOut >= minAmountOut, "Slippage exceeded");
            
            self.reserveB = self.reserveB + amountIn;
            self.reserveA = self.reserveA - amountOut;
        }
        
        // 发送输出代币给用户
        // 实际实现需要调用代币合约转账
    }
    
    // 计算输出金额（考虑手续费）
    fun getAmountOut(amountIn: Int, reserveIn: Int, reserveOut: Int): Int {
        let amountInWithFee: Int = amountIn * (self.feeDenominator - self.feeNumerator);
        let numerator: Int = amountInWithFee * reserveOut;
        let denominator: Int = reserveIn * self.feeDenominator + amountInWithFee;
        return numerator / denominator;
    }
    
    // 获取价格
    get fun getPrice(): Int {
        if (self.reserveA == 0) return 0;
        return (self.reserveB * 1000000) / self.reserveA;  // 6位精度
    }
    
    get fun getReserves(): Reserves {
        return Reserves{
            reserveA: self.reserveA,
            reserveB: self.reserveB
        };
    }
}

struct Reserves {
    reserveA: Int as coins;
    reserveB: Int as coins;
}

message AddLiquidity {
    amountA: Int as coins;
    amountB: Int as coins;
}

message RemoveLiquidity {
    lpAmount: Int as coins;
}

message Swap {
    amountIn: Int as coins;
    tokenIn: Address;
    minAmountOut: Int as coins;
}
```

### 17.2 DEX 合约开发

#### 17.2.1 Router 合约

```tact
// DEX Router 合约
contract DexRouter {
    factory: Address;  // 工厂合约地址
    
    init(factory: Address) {
        self.factory = factory;
    }
    
    // 添加流动性（通过 Router）
    receive(msg: AddLiquidityViaRouter) {
        // 获取或创建流动性池
        let pair: Address = self.getPair(msg.tokenA, msg.tokenB);
        
        // 转发到流动性池合约
        send(SendParameters{
            to: pair,
            value: context().value,
            mode: SendPayGasSeparately,
            body: AddLiquidity{
                amountA: msg.amountA,
                amountB: msg.amountB
            }.toCell()
        });
    }
    
    // 执行交换（支持多跳）
    receive(msg: SwapExactTokensForTokens) {
        let path: map<Int, Address> = msg.path;
        let amountIn: Int = msg.amountIn;
        let amountOutMin: Int = msg.amountOutMin;
        let to: Address = msg.to;
        
        // 计算输出金额
        let amounts: map<Int, Int> = self.getAmountsOut(amountIn, path);
        let finalAmountOut: Int = amounts.get(mapSize(amounts) - 1)!!;
        
        require(finalAmountOut >= amountOutMin, "Insufficient output amount");
        
        // 执行交换
        self.executeSwap(amounts, path, to);
    }
    
    // 计算多跳交换的输出金额
    fun getAmountsOut(amountIn: Int, path: map<Int, Address>): map<Int, Int> {
        let amounts: map<Int, Int>;
        amounts.set(0, amountIn);
        
        // 简化示例，实际需要遍历路径
        for (let i: Int = 0; i < mapSize(path) - 1; i = i + 1) {
            let pair: Address = self.getPair(path.get(i)!!, path.get(i + 1)!!);
            // 获取储备并计算输出
            // amounts.set(i + 1, ...);
        }
        
        return amounts;
    }
    
    // 执行交换
    fun executeSwap(amounts: map<Int, Int>, path: map<Int, Address>, to: Address) {
        // 遍历路径执行交换
        for (let i: Int = 0; i < mapSize(path) - 1; i = i + 1) {
            let pair: Address = self.getPair(path.get(i)!!, path.get(i + 1)!!);
            
            send(SendParameters{
                to: pair,
                value: ton("0.01"),
                mode: SendPayGasSeparately,
                body: Swap{
                    amountIn: amounts.get(i)!!,
                    tokenIn: path.get(i)!!,
                    minAmountOut: amounts.get(i + 1)!!
                }.toCell()
            });
        }
    }
    
    // 获取交易对地址
    fun getPair(tokenA: Address, tokenB: Address): Address {
        // 调用工厂合约获取或创建交易对
        // 简化示例
        return newAddress(0, 0);
    }
}

message AddLiquidityViaRouter {
    tokenA: Address;
    tokenB: Address;
    amountA: Int as coins;
    amountB: Int as coins;
}

message SwapExactTokensForTokens {
    amountIn: Int as coins;
    amountOutMin: Int as coins;
    path: map<Int, Address>;
    to: Address;
}
```

### 17.3 质押（Staking）合约

```tact
// 质押合约
contract StakingContract {
    // 质押代币
    stakingToken: Address;
    
    // 奖励代币
    rewardToken: Address;
    
    // 奖励参数
    rewardRate: Int;  // 每秒奖励数量
    lastUpdateTime: Int;
    rewardPerTokenStored: Int;
    
    // 用户质押信息
    stakes: map<Address, StakeInfo>;
    
    // 总质押量
    totalStaked: Int as coins = 0;
    
    // 合约余额
    rewardBalance: Int as coins = 0;
    
    init(stakingToken: Address, rewardToken: Address, rewardRate: Int) {
        self.stakingToken = stakingToken;
        self.rewardToken = rewardToken;
        self.rewardRate = rewardRate;
        self.lastUpdateTime = now();
    }
    
    // 更新奖励参数
    fun updateReward(account: Address) {
        self.rewardPerTokenStored = self.rewardPerToken();
        self.lastUpdateTime = self.lastTimeRewardApplicable();
        
        if (account != newAddress(0, 0)) {
            let stake: StakeInfo = self.stakes.get(account) ?: StakeInfo{
                amount: 0,
                rewardDebt: 0,
                pendingRewards: 0
            };
            stake.pendingRewards = self.earned(account);
            stake.rewardDebt = (stake.amount * self.rewardPerTokenStored) / 1e18;
            self.stakes.set(account, stake);
        }
    }
    
    // 计算当前每代币奖励
    fun rewardPerToken(): Int {
        if (self.totalStaked == 0) {
            return self.rewardPerTokenStored;
        }
        return self.rewardPerTokenStored + 
               ((self.lastTimeRewardApplicable() - self.lastUpdateTime) * self.rewardRate * 1e18) / self.totalStaked;
    }
    
    // 计算用户应得奖励
    fun earned(account: Address): Int {
        let stake: StakeInfo = self.stakes.get(account) ?: StakeInfo{
            amount: 0,
            rewardDebt: 0,
            pendingRewards: 0
        };
        return ((stake.amount * (self.rewardPerToken() - stake.rewardDebt)) / 1e18) + stake.pendingRewards;
    }
    
    // 质押
    receive(msg: Stake) {
        let sender: Address = sender();
        self.updateReward(sender);
        
        let amount: Int = msg.amount;
        require(amount > 0, "Cannot stake 0");
        
        let stake: StakeInfo = self.stakes.get(sender) ?: StakeInfo{
            amount: 0,
            rewardDebt: 0,
            pendingRewards: 0
        };
        stake.amount = stake.amount + amount;
        self.stakes.set(sender, stake);
        
        self.totalStaked = self.totalStaked + amount;
        
        // 实际实现需要转移质押代币到合约
    }
    
    // 解除质押
    receive(msg: Withdraw) {
        let sender: Address = sender();
        self.updateReward(sender);
        
        let amount: Int = msg.amount;
        let stake: StakeInfo = self.stakes.get(sender)!!;
        require(stake.amount >= amount, "Insufficient stake");
        
        stake.amount = stake.amount - amount;
        self.stakes.set(sender, stake);
        self.totalStaked = self.totalStaked - amount;
        
        // 返还质押代币
        // 实际实现需要转账
    }
    
    // 领取奖励
    receive(msg: GetReward) {
        let sender: Address = sender();
        self.updateReward(sender);
        
        let stake: StakeInfo = self.stakes.get(sender)!!;
        let reward: Int = stake.pendingRewards;
        require(reward > 0, "No rewards");
        
        stake.pendingRewards = 0;
        self.stakes.set(sender, stake);
        
        // 发送奖励代币
        // 实际实现需要转账
    }
    
    // 退出（解除质押并领取奖励）
    receive(msg: Exit) {
        let sender: Address = sender();
        self.updateReward(sender);
        
        let stake: StakeInfo = self.stakes.get(sender)!!;
        let amount: Int = stake.amount;
        let reward: Int = stake.pendingRewards;
        
        require(amount > 0 || reward > 0, "Nothing to exit");
        
        // 解除质押
        if (amount > 0) {
            self.totalStaked = self.totalStaked - amount;
            // 返还质押代币
        }
        
        // 清空用户数据
        self.stakes.del(sender);
        
        // 发送奖励
        if (reward > 0) {
            // 发送奖励代币
        }
    }
    
    fun lastTimeRewardApplicable(): Int {
        return now();  // 简化，实际可能需要检查奖励结束时间
    }
    
    get fun getStakeInfo(account: Address): StakeInfo {
        return self.stakes.get(account) ?: StakeInfo{
            amount: 0,
            rewardDebt: 0,
            pendingRewards: 0
        };
    }
    
    get fun getTotalStaked(): Int {
        return self.totalStaked;
    }
}

struct StakeInfo {
    amount: Int as coins;
    rewardDebt: Int;
    pendingRewards: Int as coins;
}

message Stake {
    amount: Int as coins;
}

message Withdraw {
    amount: Int as coins;
}

message GetReward {}

message Exit {}
```

---

**本章小结：**

本章介绍了 TON 上的 DeFi 合约开发：
- **AMM 原理**：恒定乘积公式、流动性池设计
- **DEX 合约**：Router 合约、多跳交换
- **质押合约**：奖励计算、质押/解除质押逻辑

DeFi 是 TON 生态的重要组成部分，掌握这些合约开发技能可以构建复杂的金融产品。

---

## 第 18 章：节点运维与基础设施

本章介绍 TON 节点的部署、运维和基础设施搭建。

### 18.1 TON 节点类型

TON 网络中有多种类型的节点，各自承担不同的角色：

| 节点类型 | 功能 | 硬件要求 | 适用场景 |
|---------|------|---------|---------|
| **验证者节点** | 参与共识，生成区块 | 高（64GB+ RAM，SSD） | 验证者运营 |
| **全节点** | 存储完整区块链数据 | 中高（32GB+ RAM，大容量存储） | 数据服务 |
| **轻节点** | 验证区块头，不存储完整数据 | 低（4GB RAM） | 轻量级验证 |
| **Liteserver** | 提供轻客户端服务 | 中（16GB RAM） | API 服务 |

### 18.2 节点部署

#### 18.2.1 使用 mytonctrl 部署全节点

```bash
# 1. 系统要求检查
# - Ubuntu 20.04/22.04 LTS
# - 至少 16GB RAM（推荐 32GB+）
# - 至少 1TB SSD 存储
# - 稳定的网络连接

# 2. 安装 mytonctrl
cd /usr/src
sudo apt-get update
sudo apt-get install -y git

# 克隆 mytonctrl
git clone https://github.com/ton-blockchain/mytonctrl.git
cd mytonctrl

# 安装
sudo bash install.sh

# 3. 配置 mytonctrl
# 启动 mytonctrl 控制台
mytonctrl

# 在 mytonctrl 控制台中
# 查看状态
status

# 查看节点同步状态
status fast

# 查看验证者状态（如果是验证者）
validator_status
```

#### 18.2.2 节点配置

```bash
# 节点配置文件位置
/etc/ton/local.config.json

# 主要配置项
{
  "liteservers": [
    {
      "ip": 0,
      "port": 43679,
      "id": {
        "@type": "pub.ed25519",
        "key": "..."
      }
    }
  ],
  "validators": [
    {
      "id": "...",
      "temp_keys": [...],
      "adnl_addrs": [...]
    }
  ]
}
```

### 18.3 验证者运营

#### 18.3.1 参与验证者选举

```bash
# 在 mytonctrl 中参与选举

# 1. 创建钱包（如果还没有）
new_wallet my_wallet

# 2. 为钱包充值（至少 10,000 TON）
# 从交易所或其他钱包转账

# 3. 激活钱包
activate_wallet my_wallet

# 4. 参与选举
# mytonctrl 会自动处理选举流程

# 查看选举状态
election_status

# 查看验证者历史
validator_history
```

#### 18.3.2 监控和维护

```bash
# 创建监控脚本
#!/bin/bash
# monitor.sh

# 检查节点进程
if ! pgrep -x "validator-engine" > /dev/null; then
    echo "$(date): Validator engine not running!" >> /var/log/ton_monitor.log
    # 发送告警（邮件/电报）
    # 重启节点
    systemctl restart validator
fi

# 检查同步状态
SYNC_STATUS=$(mytonctrl -c "status" | grep "Synchronization")
if echo "$SYNC_STATUS" | grep -q "not synchronized"; then
    echo "$(date): Node not synchronized!" >> /var/log/ton_monitor.log
fi

# 检查磁盘空间
DISK_USAGE=$(df / | tail -1 | awk '{print $5}' | sed 's/%//')
if [ "$DISK_USAGE" -gt 90 ]; then
    echo "$(date): Disk usage critical: ${DISK_USAGE}%" >> /var/log/ton_monitor.log
fi
```

### 18.4 网络接入

#### 18.4.1 HTTP API（TON Center）

```typescript
// 使用 TON Center API
import { TonClient } from '@ton/ton';

const client = new TonClient({
  endpoint: 'https://toncenter.com/api/v2/jsonRPC',
  apiKey: 'your-api-key'  // 从 toncenter.com 获取
});

// 查询账户余额
const balance = await client.getBalance(address);

// 查询交易
const transactions = await client.getTransactions(address, { limit: 10 });
```

#### 18.4.2 Liteserver 客户端

```typescript
// 连接到自己的 Liteserver
import { TonClient } from '@ton/ton';

const client = new TonClient({
  endpoint: 'http://your-liteserver:43679',
  // 不需要 API key
});
```

---

**本章小结：**

本章介绍了 TON 节点运维：
- **节点类型**：验证者、全节点、轻节点、Liteserver
- **节点部署**：使用 mytonctrl 部署和管理
- **验证者运营**：参与选举、监控维护
- **网络接入**：HTTP API、Liteserver

节点运维是 TON 基础设施的重要组成部分，对于需要高性能、高可靠性的应用至关重要。

---

## 第 19 章：性能优化与 Gas 调优

本章介绍如何优化 TON 合约的性能和 Gas 消耗。

### 19.1 Gas 消费分析

#### 19.1.1 Gas 计量模型

```
Gas 消耗组成：
┌─────────────────────────────────────────────────────────────┐
│                     Gas 消耗结构                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  计算 Gas (Compute Gas)                                     │
│  ├── 指令执行：1-10 Gas/指令                                │
│  ├── 算术运算：5-20 Gas                                     │
│  ├── Cell 操作：10-100 Gas                                  │
│  └── 字典操作：50-500 Gas                                   │
│                                                             │
│  操作 Gas (Action Gas)                                      │
│  ├── 发送消息：500+ Gas                                     │
│  ├── 创建 Cell：50-200 Gas                                  │
│  └── 状态存储：按大小计费                                   │
│                                                             │
│  存储 Gas (Storage Gas)                                     │
│  ├── 按 bit 计费                                            │
│  ├── 按 cell 计费                                           │
│  └── 长期存储费用                                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### 19.1.2 常见操作的 Gas 成本

| 操作 | Gas 成本 | 说明 |
|-----|---------|------|
| 简单算术 | 5-10 | +, -, *, / |
| 比较 | 5 | ==, <, > |
| Cell 创建 | 50-100 | beginCell(), endCell() |
| 存储数据 | 1/bit | storeUint, storeAddress |
| 读取数据 | 1/bit | loadUint, loadAddress |
| 字典查找 | 100-300 | map.get() |
| 字典插入 | 200-500 | map.set() |
| 发送消息 | 500+ | send() |
| 消息转发 | 按大小 | 与消息大小相关 |

### 19.2 存储优化

#### 19.2.1 Cell 树结构优化

```tact
// ❌ 低效：扁平化存储
contract InefficientStorage {
    data1: Int;
    data2: Int;
    data3: Int;
    // ... 更多字段
    data100: Int;  // 所有数据在一个大 Cell 中
}

// ✅ 高效：分层存储
contract EfficientStorage {
    // 常用数据放在主 Cell
    activeUsers: Int;
    totalVolume: Int;
    
    // 不常用数据放在引用 Cell
    historicalData: Cell;  // 通过引用访问
    metadata: Cell;
}
```

#### 19.2.2 存储费用最小化

```tact
// 优化存储大小的技巧

// 1. 使用合适的整数类型
contract OptimizedIntegers {
    // ❌ 浪费空间
    smallValue: Int = 1;  // 使用 257 位存储小值
    
    // ✅ 指定合适的位大小
    smallValue: Int as uint8 = 1;   // 8 位
    mediumValue: Int as uint32 = 1000;  // 32 位
    timestamp: Int as uint64 = 1234567890;  // 64 位
}

// 2. 使用布尔标志位
contract OptimizedFlags {
    // ❌ 每个布尔值占用 1 字节
    isActive: Bool = true;
    isPaused: Bool = false;
    isPublic: Bool = true;
    
    // ✅ 使用位掩码
    flags: Int as uint8 = 0b00000101;  // bit 0: isActive, bit 2: isPublic
    
    fun isActive(): Bool {
        return (self.flags & 1) != 0;
    }
    
    fun setActive(value: Bool) {
        if (value) {
            self.flags = self.flags | 1;
        } else {
            self.flags = self.flags & ~1;
        }
    }
}

// 3. 压缩重复数据
contract CompressedData {
    // ❌ 存储完整地址列表
    userAddresses: map<Int, Address>;  // 每个地址 267 位
    
    // ✅ 如果地址有共同前缀，可以优化存储
    // 或者使用 Merkle 树存储
}
```

### 19.3 计算优化

#### 19.3.1 循环优化

```tact
// ❌ 低效：不必要的循环
contract InefficientLoop {
    items: map<Int, Item>;
    
    fun getTotalValue(): Int {
        var total: Int = 0;
        foreach (id, item in self.items) {
            total = total + item.value;
        }
        return total;
    }
}

// ✅ 高效：缓存计算结果
contract EfficientLoop {
    items: map<Int, Item>;
    cachedTotal: Int = 0;  // 缓存总值
    cacheValid: Bool = false;
    
    fun addItem(id: Int, item: Item) {
        self.items.set(id, item);
        self.cachedTotal = self.cachedTotal + item.value;
        // 缓存保持有效
    }
    
    fun removeItem(id: Int) {
        let item: Item = self.items.get(id)!!;
        self.items.del(id);
        self.cachedTotal = self.cachedTotal - item.value;
    }
    
    fun getTotalValue(): Int {
        return self.cachedTotal;  // O(1) 复杂度
    }
}
```

#### 19.3.2 字典操作优化

```tact
// 字典操作优化技巧
contract OptimizedDictionary {
    data: map<Address, UserData>;
    
    // ❌ 低效：多次字典访问
    fun updateUserInefficient(user: Address, newData: UserData) {
        if (self.data.exists(user)) {
            let oldData: UserData = self.data.get(user)!!;
            // 使用 oldData
            self.data.set(user, newData);
        }
    }
    
    // ✅ 高效：最小化字典访问
    fun updateUserEfficient(user: Address, newData: UserData) {
        // 直接设置，避免先读取
        self.data.set(user, newData);
    }
    
    // ❌ 低效：遍历大字典
    fun findUserByName(name: String): Address? {
        foreach (addr, data in self.data) {
            if (data.name == name) {
                return addr;
            }
        }
        return null;
    }
    
    // ✅ 高效：建立反向索引
    nameToAddress: map<String, Address>;
    
    fun findUserByNameOptimized(name: String): Address? {
        return self.nameToAddress.get(name);  // O(1) 查找
    }
}
```

### 19.4 消息优化

#### 19.4.1 消息合并与批处理

```tact
// 批处理合约
contract BatchOptimized {
    pendingOperations: map<Int, Operation>;
    operationCount: Int = 0;
    
    // ❌ 低效：单独发送每个操作
    fun executeIndividual(operations: map<Int, Operation>) {
        foreach (id, op in operations) {
            send(SendParameters{
                to: op.target,
                value: op.value,
                mode: SendPayGasSeparately
            });
        }
    }
    
    // ✅ 高效：批处理操作
    fun addToBatch(op: Operation) {
        self.pendingOperations.set(self.operationCount, op);
        self.operationCount = self.operationCount + 1;
        
        // 当批次达到一定大小时执行
        if (self.operationCount >= 10) {
            self.executeBatch();
        }
    }
    
    fun executeBatch() {
        // 使用单个消息处理多个操作
        // 或者发送到批处理合约
        send(SendParameters{
            to: self.batchProcessor,
            value: ton("0.1"),
            mode: SendPayGasSeparately,
            body: BatchExecute{
                operations: self.pendingOperations
            }.toCell()
        });
        
        // 清空批次
        self.pendingOperations = emptyMap();
        self.operationCount = 0;
    }
}

message BatchExecute {
    operations: map<Int, Operation>;
}

struct Operation {
    target: Address;
    value: Int as coins;
    data: Cell;
}
```

#### 19.4.2 转发费用优化

```tact
// 优化消息转发费用
contract ForwardOptimized {
    // ❌ 低效：携带不必要的数据
    fun sendLargeMessage() {
        let largePayload: Cell = beginCell()
            .storeRef(largeData1)
            .storeRef(largeData2)
            .storeRef(largeData3)
            .endCell();
        
        send(SendParameters{
            to: target,
            value: ton("0.05"),
            body: largePayload  // 大消息 = 高转发费
        });
    }
    
    // ✅ 高效：只发送必要数据
    fun sendOptimizedMessage() {
        // 只发送哈希或引用
        let optimizedPayload: Cell = beginCell()
            .storeUint(dataHash, 256)  // 只发送 256 位哈希
            .endCell();
        
        send(SendParameters{
            to: target,
            value: ton("0.01"),  // 更少的转发费
            body: optimizedPayload
        });
        
        // 完整数据可以通过其他方式获取（IPFS、链下存储等）
    }
}
```

### 19.5 Gas 基准测试

```typescript
// 使用 Sandbox 进行 Gas 基准测试
import { Blockchain } from '@ton-community/sandbox';
import { toNano } from '@ton/core';

describe('Gas Benchmarks', () => {
  let blockchain: Blockchain;
  let contract: MyContract;

  beforeEach(async () => {
    blockchain = await Blockchain.create();
    contract = blockchain.openContract(await MyContract.fromInit());
    // 部署合约...
  });

  it('should benchmark different operations', async () => {
    const operations = [
      { name: 'Simple transfer', op: 'Transfer' },
      { name: 'Complex calculation', op: 'ComplexCalc' },
      { name: 'Dictionary update', op: 'DictUpdate' },
    ];

    for (const op of operations) {
      const result = await contract.send(
        sender.getSender(),
        { value: toNano('0.1') },
        { $$type: op.op }
      );

      const tx = result.transactions.find(
        tx => tx.inMessage?.info.dest?.equals(contract.address)
      );

      if (tx) {
        console.log(`${op.name}: ${Number(tx.totalFees) / 1e9} TON`);
      }
    }
  });
});
```

---

**本章小结：**

本章介绍了 TON 合约的性能优化：
- **Gas 消费分析**：理解 Gas 计量模型和常见操作成本
- **存储优化**：Cell 树结构、数据压缩、位掩码
- **计算优化**：循环优化、字典操作优化、缓存策略
- **消息优化**：批处理、转发费用优化
- **基准测试**：使用 Sandbox 测量和优化 Gas 消耗

性能优化是生产级合约开发的关键，可以显著降低用户成本和提升用户体验。

---

**进阶篇小结：**

进阶篇涵盖了 TON 开发的高级主题：
- **第16章**：高级合约模式（代理、升级、高负载、跨合约通信）
- **第17章**：DeFi 合约开发（AMM、DEX、质押）
- **第18章**：节点运维与基础设施
- **第19章**：性能优化与 Gas 调优

掌握这些内容后，开发者可以构建企业级的 TON 应用，处理复杂的业务场景和高并发需求。

---
