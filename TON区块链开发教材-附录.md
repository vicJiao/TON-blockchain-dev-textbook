# TON 区块链开发教材 - 附录

> 本附录包含 TON 开发的参考资料和速查表。

---

## 附录 A：TON 术语表

### 核心概念

| 术语 | 英文 | 说明 |
|------|------|------|
| 主链 | Masterchain | TON 区块链的核心链，存储全局配置和系统合约 |
| 工作链 | Workchain | 处理具体业务逻辑的链，如 Basechain（工作链0） |
| 分片链 | Shardchain | 工作链的分片，用于并行处理交易 |
| 单元格 | Cell | TON 的基本数据存储单元，最多1023位数据+4个引用 |
| 切片 | Slice | 用于读取 Cell 数据的游标 |
| 构建器 | Builder | 用于构建新 Cell 的写入游标 |
| BoC | Bag of Cells | Cell 的序列化格式，用于网络传输和存储 |
| TVM | TON Virtual Machine | TON 虚拟机，执行智能合约代码 |
| TON | The Open Network | 区块链网络名称 |
| Toncoin | Toncoin | TON 网络的原生代币 |

### 账户相关

| 术语 | 英文 | 说明 |
|------|------|------|
| 地址 | Address | TON 账户地址，格式为 `workchain:account_id` |
| 可退回 | Bounceable | 地址类型，交易失败时退回资金 |
| 不可退回 | Non-bounceable | 地址类型，交易失败时不退回资金 |
| 用户友好地址 | User-friendly Address | 带校验和的 Base64 编码地址 |
| 原始地址 | Raw Address | 十六进制格式的地址 |
| 账户状态 | Account State | 账户当前状态：nonexist/uninit/active/frozen |

### 消息相关

| 术语 | 英文 | 说明 |
|------|------|------|
| 内部消息 | Internal Message | 合约间发送的消息，携带价值 |
| 外部输入 | External In | 从外部（如用户）发送到合约的消息 |
| 外部输出 | External Out | 合约发送到外部的消息 |
| 退回消息 | Bounce | 交易失败时返回的错误消息 |
| 操作码 | Op Code | 消息类型标识符，32位整数 |
| 查询ID | Query ID | 用于追踪消息的唯一标识 |

### 合约相关

| 术语 | 英文 | 说明 |
|------|------|------|
| 智能合约 | Smart Contract | 部署在链上的可执行代码 |
| 存储 | Storage | 合约的持久化数据存储 |
| Getter | Getter | 只读函数，查询合约状态 |
| Receiver | Receiver | 消息接收处理函数 |
| Trait | Trait | Tact 语言中的可复用合约组件 |
| 初始化 | Init | 合约部署时的初始化代码 |

### 代币相关

| 术语 | 英文 | 说明 |
|------|------|------|
| Jetton | Jetton | TON 上的同质化代币标准（类似 ERC-20） |
| NFT | Non-Fungible Token | 非同质化代币 |
| SBT | Soulbound Token | 灵魂绑定代币，不可转让 |
| TEP | TON Enhancement Proposal | TON 改进提案 |
| 元数据 | Metadata | 代币的描述信息（名称、符号、图标等） |

### 开发工具

| 术语 | 英文 | 说明 |
|------|------|------|
| Blueprint | Blueprint | TON 官方推荐的合约开发框架 |
| Sandbox | Sandbox | 本地测试环境 |
| TON Connect | TON Connect | 钱包连接协议和 SDK |
| Fift | Fift | TON 的低级堆栈语言 |
| FunC | FunC | TON 的 C 风格合约语言（已逐步被 Tolk 取代） |
| Tolk | Tolk | TON 官方推荐的现代合约语言 |
| Tact | Tact | 社区开发的高级合约语言 |

### 网络和共识

| 术语 | 英文 | 说明 |
|------|------|------|
| 验证者 | Validator | 参与区块生产和共识的节点 |
| 提名者 | Nominator | 将代币委托给验证者的用户 |
| 权益证明 | PoS | Proof of Stake，TON 的共识机制 |
| 分片 | Sharding | 将网络分割为多个并行处理的分片 |
| 超立方体路由 | Hypercube Routing | TON 的高效消息路由机制 |
| ADNL | Abstract Datagram Network Layer | TON 的网络传输层协议 |

---

## 附录 B：常用 TEP 标准速查

### TEP-62：NFT 标准

**核心接口：**

```tact
// NFT Item 合约接口
message(0x5fcc3d14) Transfer {
    queryId: Int as uint64;
    newOwner: Address;
    responseDestination: Address;
    customPayload: Cell?;
    forwardAmount: Int as coins;
    forwardPayload: Slice as remaining;
}

// 获取 NFT 数据
get fun get_nft_data(): NftData {
    return NftData{
        init: Bool,
        index: Int,
        collection: Address?,
        owner: Address,
        content: Cell
    };
}
```

### TEP-64：元数据标准

**链上格式：**

```
content: {
  "uri": "https://example.com/metadata.json"  // 链下模式
}

// 或链上模式
content: {
  "name": "Token Name",
  "description": "Token Description",
  "image": "https://example.com/image.png"
}
```

### TEP-74：Jetton 标准

**核心消息：**

```tact
// 转账消息
message(0xf8a7ea5) TokenTransfer {
    queryId: Int as uint64;
    amount: Int as coins;
    destination: Address;
    responseDestination: Address;
    customPayload: Cell?;
    forwardTonAmount: Int as coins;
    forwardPayload: Slice as remaining;
}

// 转账通知
message(0x7362d09c) TokenNotification {
    queryId: Int as uint64;
    amount: Int as coins;
    sender: Address;
    forwardPayload: Slice as remaining;
}

// 销毁消息
message(0x595f07bc) TokenBurn {
    queryId: Int as uint64;
    amount: Int as coins;
    responseDestination: Address;
    customPayload: Cell?;
}
```

### TEP-66：NFT 版税标准

```tact
// 获取版税信息
get fun royalty_params(): RoyaltyParams {
    return RoyaltyParams{
        numerator: Int,      // 版税分子
        denominator: Int,    // 版税分母
        destination: Address // 收款地址
    };
}

// 常用版税率
// 2.5% = 25/1000
// 5%   = 50/1000
// 10%  = 100/1000
```

### TEP-81：TON DNS 标准

```tact
// DNS 解析请求
message(0x4eb1f0f9) DnsResolve {
    queryId: Int as uint64;
    domain: Slice;        // 域名切片
    category: Int;        // 查询类别
}

// 常见 DNS 类别
// 0x00000000 - 钱包地址
// 0xad01 - 存储桶地址
// 0xe8d4405088 - 站点内容
```

### TEP-85：SBT 标准

```tact
// SBT 与 NFT 类似，但禁止转让
// 典型实现会禁用 Transfer 消息处理

receive(msg: Transfer) {
    // SBT 不可转让
    throw(709); // 错误码：SBT 禁止转让
}
```

### TEP-89：钱包合约标准

**钱包版本演进：**

| 版本 | 特点 | 状态 |
|------|------|------|
| v1 | 基础版本 | 已弃用 |
| v2 | 添加 seqno | 维护中 |
| v3 | 添加 subwallet_id | 维护中 |
| v4 | 插件系统 | 推荐使用 |
| v5 | 最新版本 | 推荐使用 |

---

## 附录 C：Tolk / Tact 语法速查对照表

### 基础语法对比

| 特性 | Tolk | Tact |
|------|------|------|
| 文件扩展名 | `.tolk` | `.tact` |
| 变量声明 | `let x: int = 10;` | `let x: Int = 10;` |
| 可变变量 | `var x = 10;` | `let x: Int = 10;` (合约内可变) |
| 常量 | `const MAX: int = 100;` | `const MAX: Int = 100;` |
| 函数定义 | `fun add(a: int, b: int): int { ... }` | `fun add(a: Int, b: Int): Int { ... }` |
| 合约声明 | `contract MyContract { ... }` | `contract MyContract { ... }` |
| 消息定义 | `message Transfer { ... }` | `message Transfer { ... }` |

### 数据类型对比

| 类型 | Tolk | Tact |
|------|------|------|
| 整数 | `int` | `Int` |
| 布尔 | `bool` | `Bool` |
| 地址 | `address` | `Address` |
| Cell | `cell` | `Cell` |
| Slice | `slice` | `Slice` |
| Builder | `builder` | `Builder` |
| 字符串 | `string` | `String` |
| 映射 | `map<K, V>` | `map<K, V>` |
| 可选 | `T?` | `T?` |

### 控制流对比

| 特性 | Tolk | Tact |
|------|------|------|
| if/else | `if (cond) { ... } else { ... }` | `if (cond) { ... } else { ... }` |
| 循环 | `while (cond) { ... }` | `while (cond) { ... }` |
| 遍历 | `foreach (k, v in map) { ... }` | `foreach (k, v in map) { ... }` |
| 匹配 | `match (value) { ... }` | `if-let` 模式匹配 |
| 返回 | `return value;` | `return value;` |

### 合约特性对比

| 特性 | Tolk | Tact |
|------|------|------|
| 初始化 | `init() { ... }` | `init() { ... }` |
| 消息接收 | `receive(msg: Type) { ... }` | `receive(msg: Type) { ... }` |
| Getter | `get fun name(): Type { ... }` | `get fun name(): Type { ... }` |
| 发送消息 | `send(msg);` | `send(SendParameters{...});` |
| 获取发送者 | `sender()` | `sender()` |
| 获取地址 | `myAddress()` | `myAddress()` |
| 获取余额 | `myBalance()` | `myBalance()` |

### 常用函数对比

| 功能 | Tolk | Tact |
|------|------|------|
| 创建 Cell | `beginCell()` | `beginCell()` |
| 存储整数 | `.storeInt(value, bits)` | `.storeInt(value, bits)` |
| 存储地址 | `.storeAddress(addr)` | `.storeAddress(addr)` |
| 存储引用 | `.storeRef(cell)` | `.storeRef(cell)` |
| 结束 Cell | `.endCell()` | `.endCell()` |
| 解析地址 | `parseAddress("EQ...")` | `address("EQ...")` |
| 金额转换 | `ton("1.5")` | `ton("1.5")` |
| 抛出错误 | `throw(code)` | `throw(code)` |
| 要求条件 | `require(cond, "msg")` | `require(cond, "msg")` |

---

## 附录 D：常用开发命令速查

### Blueprint 命令

```bash
# 项目创建
npx create-ton@latest my-project
cd my-project
npm install

# 编译合约
npx blueprint build
npx blueprint build ContractName
npx blueprint build --force

# 运行测试
npx blueprint test
npx blueprint test --gas-report
npx blueprint test --coverage

# 部署合约
npx blueprint run
npx blueprint run --testnet
npx blueprint run --mainnet
npx blueprint run deploy.ts --testnet

# 创建新合约
npx blueprint create ContractName

# 验证合约
npx blueprint verify
```

### Tact 编译器命令

```bash
# 安装
npm install -g @tact-lang/compiler

# 编译
tact contracts/main.tact

# 带配置编译
tact --config tact.config.json
```

### Node.js 相关

```bash
# 初始化项目
npm init -y

# 安装依赖
npm install @ton/core @ton/ton @ton/blueprint
npm install --save-dev @ton-community/sandbox

# 运行 TypeScript
npx ts-node script.ts

# 运行测试
npx jest
npx jest --coverage
```

### Git 命令

```bash
# 初始化仓库
git init

# 添加文件
git add .
git add filename

# 提交更改
git commit -m "message"

# 推送到远程
git push origin main

# 拉取更新
git pull origin main
```

### Docker 命令（本地节点）

```bash
# 运行本地 TON 节点
docker run -d --name ton-node \
  -v ton-db:/var/ton-work/db \
  -p 43679:43679 \
  tonlabs/local-node:latest

# 查看日志
docker logs -f ton-node

# 停止节点
docker stop ton-node

# 删除容器
docker rm ton-node
```

---

## 附录 E：测试网与水龙头资源

### 测试网水龙头

| 资源 | 链接/方法 | 说明 |
|------|-----------|------|
| 官方水龙头 Bot | [@testgiver_ton_bot](https://t.me/testgiver_ton_bot) | 每次 2 TON，需要验证 |
| Testnet 浏览器 | [testnet.tonscan.org](https://testnet.tonscan.org) | 查看测试网交易 |
| Testnet API | [testnet.toncenter.com](https://testnet.toncenter.com) | 测试网 API 端点 |

### 测试网配置

```typescript
// 测试网客户端配置
const testnetClient = new TonClient({
  endpoint: 'https://testnet.toncenter.com/api/v2/jsonRPC',
  apiKey: 'your-api-key'
});

// 测试网合约地址示例
const testnetAddress = Address.parse('kQ...'); // 测试网地址以 kQ 开头
```

### 测试网钱包设置

**Tonkeeper：**
1. 打开 Tonkeeper 应用
2. 进入 Settings → Network
3. 选择 Testnet
4. 复制地址，从水龙头获取测试币

**MyTonWallet：**
1. 打开网页钱包
2. 点击网络切换按钮
3. 选择 Testnet
4. 复制地址获取测试币

---

## 附录 F：学习资源与社区链接

### 官方资源

| 资源 | 链接 | 说明 |
|------|------|------|
| TON 官方文档 | [docs.ton.org](https://docs.ton.org) | 最权威的技术文档 |
| Tact 文档 | [docs.tact-lang.org](https://docs.tact-lang.org) | Tact 语言官方文档 |
| Tolk 文档 | [docs.ton.org/develop/smart-contracts/tolk](https://docs.ton.org/develop/smart-contracts/tolk) | Tolk 语言文档 |
| TON Academy | [ton.academy](https://ton.academy) | 官方教程和课程 |
| TON 白皮书 | [ton.org/whitepaper](https://ton.org/whitepaper) | 技术白皮书 |

### GitHub 仓库

| 仓库 | 链接 | 说明 |
|------|------|------|
| TON 核心 | [github.com/ton-blockchain](https://github.com/ton-blockchain) | 官方代码库 |
| Tact 编译器 | [github.com/tact-lang/tact](https://github.com/tact-lang/tact) | Tact 语言编译器 |
| Blueprint | [github.com/ton-org/blueprint](https://github.com/ton-org/blueprint) | 开发框架 |
| TON Connect | [github.com/ton-connect](https://github.com/ton-connect) | 钱包连接 SDK |

### 社区资源

| 资源 | 链接 | 说明 |
|------|------|------|
| TON Dev (英文) | [t.me/tondev_eng](https://t.me/tondev_eng) | 开发者讨论群 |
| TON Dev (中文) | [t.me/tondev_zh](https://t.me/tondev_zh) | 中文开发者群 |
| TON 论坛 | [forum.ton.org](https://forum.ton.org) | 技术论坛 |
| TON 博客 | [blog.ton.org](https://blog.ton.org) | 官方博客 |

### 开发工具

| 工具 | 链接 | 说明 |
|------|------|------|
| TON Scan | [tonscan.org](https://tonscan.org) | 区块链浏览器 |
| TON Viewer | [tonviewer.com](https://tonviewer.com) | 替代浏览器 |
| Dton | [dton.io](https://dton.io) | 高级查询工具 |
| Toncenter | [toncenter.com](https://toncenter.com) | API 服务 |

### 学习教程

1. [TON Hello World](https://helloworld.tonstudio.io/) - 官方入门教程
2. [Tact Cookbook](https://docs.tact-lang.org/cookbook) - 代码示例合集
3. [TON Documentation](https://docs.ton.org/develop/smart-contracts/) - 智能合约开发指南
4. [TON FunC Lessons](https://github.com/romanovichim/TonFunClessons_Eng) - FunC 教程

---

## 附录 G：常见问题（FAQ）

### 环境配置问题

**Q: 安装 Blueprint 时提示权限错误？**

A: 使用 npx 运行，无需全局安装：
```bash
npx create-ton@latest my-project
```

**Q: Node.js 版本要求？**

A: 推荐 Node.js 18+，最低要求 Node.js 16。

**Q: Windows 上编译失败？**

A: 确保安装了 Windows Build Tools：
```bash
npm install --global windows-build-tools
```

### 合约开发问题

**Q: Tolk 和 Tact 应该选哪个？**

A: 
- 新项目推荐 Tact（语法更现代，工具链完善）
- 维护旧项目可能需要 FunC
- Tolk 是官方语言，但生态还在发展中

**Q: 合约编译后代码太大？**

A: 
- 优化代码逻辑，减少重复
- 使用更高效的算法
- 考虑将部分逻辑移到链下

**Q: Gas 费用太高？**

A:
- 优化存储结构
- 使用 lazy loading
- 减少不必要的计算
- 批量处理操作

### 测试问题

**Q: 测试时提示 "Contract not deployed"？**

A: 确保在测试前部署合约：
```typescript
beforeEach(async () => {
  contract = blockchain.openContract(await MyContract.fromInit());
  await contract.send(deployer.getSender(), { value: toNano('0.05') }, { $$type: 'Deploy' });
});
```

**Q: 如何模拟时间流逝？**

A: Sandbox 支持时间操控：
```typescript
// 推进时间 1 小时
await blockchain.setTimestamp(blockchain.now + 3600);
```

### 部署问题

**Q: 部署时提示余额不足？**

A: 确保钱包有足够余额，部署通常需要 0.05-0.1 TON。

**Q: 部署后合约地址不对？**

A: 合约地址由代码和初始数据决定，确保：
- 使用正确的编译代码
- 初始化参数正确
- 工作链 ID 正确

**Q: 如何在测试网和主网之间切换？**

A: 使用 Blueprint 的 `--testnet` 和 `--mainnet` 参数：
```bash
npx blueprint run --testnet  # 测试网
npx blueprint run --mainnet  # 主网
```

### 前端集成问题

**Q: TON Connect 连接失败？**

A: 检查：
- manifest.json 配置正确
- 域名白名单已配置
- 使用正确的网络（测试网/主网）

**Q: 如何获取测试网 API Key？**

A: 访问 [toncenter.com](https://toncenter.com)，注册账号获取 API Key。

### 安全相关问题

**Q: 如何防止重入攻击？**

A:
- 先更新状态，再发送消息
- 使用重入锁模式
- 验证消息来源

**Q: 合约如何升级？**

A: TON 合约一旦部署不可修改，但可以通过：
- 代理合约模式
- 数据迁移到新合约
- 设计可升级的架构

### 性能优化问题

**Q: 如何减少存储费用？**

A:
- 清理不必要的数据
- 使用更紧凑的数据结构
- 定期归档旧数据

**Q: 如何提高合约 TPS？**

A:
- 优化算法复杂度
- 减少存储操作
- 使用批处理

---

> **提示**：本附录会随 TON 生态发展持续更新，建议关注官方文档获取最新信息。
