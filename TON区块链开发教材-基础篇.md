# TON 区块链开发教材 - 基础篇

> 本文档包含 TON 区块链开发的基础知识，从认识 TON 区块链开始，逐步深入到核心概念。

---

## 第一篇：基础篇 —— 认识 TON 区块链

---

## 第 1 章：TON 区块链概述

**TON（The Open Network）** 是一个高性能、可扩展的区块链平台，最初由 Telegram 团队设计，现在由全球社区维护和发展。本章将带你了解 TON 的起源、核心设计理念以及其丰富的生态系统。

### 1.1 什么是 TON（The Open Network）

#### 1.1.1 起源与发展历程

TON 的历史可以追溯到 2018 年：

**Telegram Open Network 时期（2018-2020）**

```
2018年：Telegram 发布 TON 白皮书和路线图
2019年：通过 ICO 筹集 17 亿美元
2020年5月：SEC 起诉 Telegram，项目被迫中止
2020年6月：Telegram 宣布停止 TON 开发
```

**The Open Network 时期（2020至今）**

```
2020年：社区接管项目，更名为 The Open Network
2021年：主网正式上线
2022年：生态快速发展，DeFi、NFT 项目涌现
2023年：Telegram 重新支持 TON，推出 Wallet Bot
2024年：TON 成为增长最快的区块链之一
2025年：生态全面爆发，Mini Apps 生态繁荣
```

**关键转折点：**

- **2020年社区接管**：Telegram 将项目开源后，由 Anatoliy Makosov 和 Kirill Emelianenko 等开发者组成的社区继续开发
- **2023年 Telegram 回归**：Telegram 创始人 Pavel Durov 公开支持 TON，并在 Telegram 中集成 TON 钱包
- **2024-2025年生态爆发**：得益于 Telegram 的 9 亿用户基础，TON 生态快速增长

#### 1.1.2 核心设计理念

TON 的设计目标是解决传统区块链的"不可能三角"问题（去中心化、安全性、可扩展性）：

**1. 高吞吐量（High Throughput）**

```
┌─────────────────────────────────────────────────────────────┐
│                    TON 吞吐量对比                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Bitcoin        ████                              ~7 TPS    │
│  Ethereum       ████████                         ~15 TPS    │
│  Solana         ████████████████████████       ~65,000 TPS  │
│  TON            ████████████████████████████  ~100,000+ TPS │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

TON 通过以下技术实现高吞吐量：
- **动态分片（Dynamic Sharding）**：根据负载自动调整分片数量
- **并行处理**：多个分片可以同时处理交易
- **超立方体路由（Hypercube Routing）**：高效的消息传递机制

**2. 无限扩展（Infinite Scalability）**

```
传统区块链：
┌─────────┐
│ 单链    │  ← 所有交易在此处理
└─────────┘
         ↓
    性能瓶颈

TON 架构：
┌─────────┐     ┌─────────┐     ┌─────────┐
│ 分片 1  │     │ 分片 2  │     │ 分片 N  │  ← 并行处理
└─────────┘     └─────────┘     └─────────┘
         \           |           /
          \          |          /
           \    ┌─────────┐    /
            └──→│ 主链    │←──┘
                └─────────┘
```

**3. 异步执行（Asynchronous Execution）**

TON 采用异步消息模型：

```
同步执行（Ethereum）：
A → B → C → D  （顺序执行，等待每个步骤完成）

异步执行（TON）：
A ─┬→ B
   ├→ C
   └→ D        （并行发送消息，不等待响应）
```

异步模型的优势：
- 合约间调用无需等待，提高吞吐量
- 消息传递天然支持跨分片通信
- 更好的容错性

#### 1.1.3 与其他区块链的对比

| 特性 | TON | Ethereum | Solana |
|------|-----|----------|--------|
| **共识机制** | PoS + BFT | PoS | PoH + PoS |
| **TPS** | 100,000+ | 15-30 | 65,000 |
| **出块时间** | ~5秒 | ~12秒 | ~400ms |
| **交易费用** | ~0.001-0.01 TON | ~$1-50 | ~$0.001 |
| **智能合约语言** | Tolk, Tact, FunC | Solidity | Rust, C |
| **账户模型** | 基于 Cell | 基于账户 | 基于账户 |
| **消息模型** | 异步 | 同步 | 同步 |
| **分片** | 动态分片 | 计划分片 | 无 |
| **TVL（2025）** | ~$10B+ | ~$50B+ | ~$4B+ |

**TON 的独特优势：**

1. **Telegram 集成**：9亿用户基础，天然流量入口
2. **用户友好**：地址可阅读（.ton 域名），Gas 费用低且稳定
3. **开发者友好**：现代化合约语言（Tolk/Tact），工具链完善
4. **可扩展性**：动态分片真正实现无限扩展

### 1.2 TON 生态全景

TON 生态系统涵盖钱包、DeFi、NFT、游戏、社交等多个领域：

#### 1.2.1 主流钱包

| 钱包 | 类型 | 特点 | 适用场景 |
|------|------|------|----------|
| **Tonkeeper** | 移动/桌面 | 用户友好，支持 dApp | 日常使用 |
| **MyTonWallet** | 网页/扩展 | 功能丰富，开源 | 开发者 |
| **Tonhub** | 移动 | 安全性高 | 大额存储 |
| **Telegram Wallet** | 内置 | 无需安装，一键使用 | 新手入门 |
| **Ledger** | 硬件 | 最高安全性 | 长期持有 |

**钱包选择建议：**

```
新手用户 → Telegram Wallet（最简单）
日常用户 → Tonkeeper（功能全面）
开发者   → MyTonWallet（开发工具）
大额资产 → Ledger + Tonkeeper（安全）
```

#### 1.2.2 DeFi 协议

**DEX（去中心化交易所）：**

| 协议 | 特点 | TVL |
|------|------|-----|
| **STON.fi** | 原生 AMM，低滑点 | ~$500M+ |
| **DeDust** | 多链桥接，高流动性 | ~$300M+ |
| **Megaton Finance** | 跨链桥 + DEX | ~$100M+ |

**流动性质押：**

| 协议 | 特点 | APY |
|------|------|-----|
| **Tonstakers** | 最大 LST 协议 | ~4-5% |
| **bemo** | 流动性质押 | ~4% |

**借贷协议：**

| 协议 | 特点 |
|------|------|
| **Evaa Protocol** | 首个 TON 借贷协议 |

#### 1.2.3 NFT 与数字收藏品

**主要平台：**

| 平台 | 特点 |
|------|------|
| **Getgems** | 最大的 TON NFT 市场 |
| **Fragment** | Telegram 用户名和号码交易 |
| **TON Diamonds** | 高端 NFT 收藏 |

**TON NFT 特点：**
- 低 Gas 费用（约 0.01-0.05 TON）
- 与 Telegram 头像集成
- 支持版税（TEP-66）

#### 1.2.4 Telegram Mini Apps 生态

**什么是 Mini Apps？**

Mini Apps 是运行在 Telegram 内的轻量级 Web 应用，可以直接与 TON 区块链交互：

```
用户流程：
Telegram → 打开 Mini App → 使用 TON Connect 连接钱包 → 与合约交互
```

**热门 Mini Apps：**

| 应用 | 类型 | 用户量 |
|------|------|--------|
| **Hamster Kombat** | 游戏 | 3亿+ |
| **Notcoin** | 点击挖矿 | 3500万+ |
| **Catizen** | 游戏 | 数千万 |
| **Yescoin** | 点击挖矿 | 数千万 |
| **TapSwap** | 游戏 | 数千万 |

**Mini Apps 开发优势：**
- 9亿 Telegram 用户触达
- 无需安装，即点即用
- 内置支付和钱包集成
- 病毒式传播能力

#### 1.2.5 Toncoin 与代币经济模型

**Toncoin（TON）** 是 TON 区块链的原生代币：

**代币信息：**

| 属性 | 详情 |
|------|------|
| **代币符号** | TON |
| **总供应量** | ~50亿（有通胀） |
| **通胀率** | ~0.5-1%/年 |
| **用途** | Gas 费、质押、治理 |

**代币用途：**

1. **Gas 费用**：执行交易和合约需要支付 TON
2. **质押（Staking）**：验证者需要质押 TON 参与共识
3. **存储费用**：合约数据存储需要支付 TON
4. **治理**：未来可能用于协议治理投票

**经济模型特点：**

```
供应端：
- 初始供应：50亿 TON（Telegram ICO 时期）
- 通胀：每年新增约 0.5-1% 用于奖励验证者
- 通缩机制：部分 Gas 费用被销毁

需求端：
- 交易需求：生态活跃带来 Gas 需求
- 质押需求：验证者和委托者需要质押
- 投机需求：投资者持有
```

### 1.3 开发环境搭建

在开始 TON 开发之前，需要配置好开发环境。本节将引导你完成所有必要的安装和配置。

#### 1.3.1 Node.js 安装与配置

**系统要求：**
- Node.js v22.0.0 或更高版本
- npm v10.0.0 或更高版本

**Windows 安装：**

```bash
# 1. 访问 https://nodejs.org/ 下载 LTS 版本
# 2. 运行安装程序，按提示完成安装

# 验证安装
node --version  # 应显示 v22.x.x 或更高
npm --version   # 应显示 10.x.x 或更高
```

**macOS 安装（使用 Homebrew）：**

```bash
# 安装 Node.js
brew install node@22

# 验证
node --version
npm --version
```

**Linux（Ubuntu/Debian）：**

```bash
# 使用 NodeSource 安装
 curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
 sudo apt-get install -y nodejs

# 验证
node --version
npm --version
```

**使用 nvm 管理 Node.js 版本（推荐）：**

```bash
# 安装 nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# 安装 Node.js 22
nvm install 22
nvm use 22
nvm alias default 22

# 验证
node --version
```

#### 1.3.2 Blueprint 框架安装

**Blueprint** 是 TON 官方推荐的合约开发框架：

```bash
# 使用 npx 创建新项目（无需全局安装）
npx create-ton@latest my-first-ton-project

# 进入项目目录
cd my-first-ton-project

# 安装依赖
npm install
```

**创建项目时的选项：**

```
? Project name: my-first-ton-project
? Choose a template: (Use arrow keys)
❯ Tact language (recommended)  ← 推荐选择
  Tolk language
  FunC language
? Initialize Git repository? (Y/n) Y
```

**项目结构：**

```
my-first-ton-project/
├── contracts/          # 合约源代码
│   └── main.tact       # 主合约文件
├── tests/              # 测试文件
│   └── main.spec.ts    # 合约测试
├── scripts/            # 部署脚本
│   └── deploy.ts       # 部署逻辑
├── wrappers/           # 合约包装器
│   └── Main.ts         # TypeScript 接口
├── build/              # 编译输出
├── temp/               # 临时文件
├── package.json        # 项目配置
├── tsconfig.json       # TypeScript 配置
├── jest.config.js      # 测试配置
└── README.md           # 项目说明
```

**常用命令：**

```bash
# 编译合约
npx blueprint build

# 运行测试
npx blueprint test

# 部署到测试网
npx blueprint run --testnet

# 部署到主网
npx blueprint run --mainnet

# 创建新合约
npx blueprint create ContractName
```

#### 1.3.3 Acton 工具安装与配置

**Acton** 是 TON 官方的新一代开发工具链：

```bash
# 安装 Acton
npm install -g @ton-community/acton

# 验证安装
acton --version

# 创建新项目
acton new my-acton-project
cd my-acton-project

# 编译合约
acton build

# 运行测试
acton test

# 部署合约
acton deploy
```

**Acton vs Blueprint：**

| 特性 | Blueprint | Acton |
|------|-----------|-------|
| 定位 | 社区框架 | 官方工具 |
| 成熟度 | 高 | 发展中 |
| 推荐度 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| 适用场景 | 生产项目 | 尝鲜体验 |

**建议**：新手从 Blueprint 开始，熟悉后再尝试 Acton。

#### 1.3.4 IDE 配置（VSCode）

**推荐插件：**

| 插件名 | 用途 | 安装方法 |
|--------|------|----------|
| **Tact** | Tact 语言支持 | 搜索 "Tact" |
| **Tolk** | Tolk 语言支持 | 搜索 "Tolk" |
| **ESLint** | 代码检查 | 搜索 "ESLint" |
| **Prettier** | 代码格式化 | 搜索 "Prettier" |
| **GitLens** | Git 增强 | 搜索 "GitLens" |

**VS Code 设置（`.vscode/settings.json`）：**

```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "[tact]": {
    "editor.formatOnSave": true
  },
  "[tolk]": {
    "editor.formatOnSave": true
  },
  "typescript.preferences.importModuleSpecifier": "relative",
  "files.exclude": {
    "**/node_modules": true,
    "**/build": true,
    "**/temp": true
  }
}
```

**调试配置（`.vscode/launch.json`）：**

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug Tests",
      "type": "node",
      "request": "launch",
      "runtimeExecutable": "npx",
      "runtimeArgs": ["blueprint", "test"],
      "console": "integratedTerminal",
      "internalConsoleOptions": "neverOpen"
    },
    {
      "name": "Debug Deploy",
      "type": "node",
      "request": "launch",
      "runtimeExecutable": "npx",
      "runtimeArgs": ["blueprint", "run", "--testnet"],
      "console": "integratedTerminal"
    }
  ]
}
```

#### 1.3.5 TON 测试网与水龙头使用

**什么是测试网？**

测试网（Testnet）是 TON 的测试环境，使用没有实际价值的测试币：

```
主网（Mainnet）：真实资产，真实交易
测试网（Testnet）：测试资产，免费获取，用于开发测试
```

**获取测试网 TON：**

**方式一：官方水龙头（Bot）**

1. 打开 Telegram，搜索 `@testgiver_ton_bot`
2. 发送 `/start` 命令
3. 按照提示完成验证
4. 提供你的测试网钱包地址
5. 每次可获得 2 个测试网 TON

**方式二：Web 水龙头**

访问 [https://t.me/testgiver_ton_bot](https://t.me/testgiver_ton_bot) 或通过网页版申请。

**配置测试网钱包：**

**使用 Tonkeeper（推荐）：**

1. 下载 Tonkeeper 应用（iOS/Android/Chrome 插件）
2. 创建新钱包或导入现有钱包
3. 进入设置 → 网络 → 切换到 Testnet
4. 复制测试网地址，从水龙头获取测试币

**测试网区块链浏览器：**

| 浏览器 | 网址 |
|--------|------|
| TON Scan Testnet | https://testnet.tonscan.org |
| TON Viewer | https://testnet.tonviewer.com |

#### 1.3.6 第一个 Hello World 合约

现在让我们创建并部署第一个 TON 智能合约：

**步骤 1：创建项目**

```bash
npx create-ton@latest hello-ton
cd hello-ton
npm install
```

**步骤 2：查看合约代码（`contracts/main.tact`）**

```tact
import "@stdlib/deploy";

// 简单的计数器合约
contract Main with Deployable {
    // 持久化存储：计数器值
    counter: Int as uint32 = 0;
    
    // 初始化函数
    init() {
        self.counter = 0;
    }
    
    // 接收增加消息
    receive("increment") {
        self.counter = self.counter + 1;
    }
    
    // Getter 函数：获取当前计数
    get fun counter(): Int {
        return self.counter;
    }
}
```

**步骤 3：编译合约**

```bash
npx blueprint build
```

**步骤 4：运行测试**

```bash
npx blueprint test
```

**步骤 5：部署到测试网**

```bash
npx blueprint run --testnet
```

部署时会提示你：
1. 选择部署方式（选择 `tonconnect` 使用钱包）
2. 使用 Tonkeeper 扫描二维码连接
3. 确认部署交易

**步骤 6：与合约交互**

部署成功后，你可以在测试网浏览器查看合约：

```
https://testnet.tonscan.org/address/<你的合约地址>
```

**恭喜！** 你已经成功部署了第一个 TON 智能合约。

---

**本章小结：**

本章介绍了 TON 区块链的基础知识：
- **1.1 节**：了解了 TON 的起源、发展历程和核心设计理念（高吞吐量、无限扩展、异步执行）
- **1.2 节**：探索了 TON 丰富的生态系统，包括钱包、DeFi、NFT、Mini Apps 等
- **1.3 节**：完成了开发环境的搭建，并成功部署了第一个智能合约

接下来，我们将深入学习 TON 的核心架构，了解其多层链设计、账户模型和消息机制。

---
# 第 2 章：TON 核心架构

TON 区块链采用创新的多层链架构设计，实现了真正的无限扩展。本章将深入解析 TON 的核心架构，包括多层链设计、账户模型、消息机制和费用模型。

### 2.1 多层链架构

TON 采用了独特的多层链架构，包含 Masterchain、Workchains 和 Shardchains 三层结构。这种设计使得 TON 能够根据负载动态扩展处理能力。

#### 2.1.1 Masterchain（主链）

**Masterchain** 是 TON 网络的核心，负责整个网络的全局状态和治理。

**主要职责：**

```
┌─────────────────────────────────────────────────────────┐
│                    Masterchain                          │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. 全局配置                                            │
│     - 网络参数                                           │
│     - 验证者列表                                         │
│     - Gas 价格                                            │
│                                                         │
│  2. 系统合约                                            │
│     - 验证者选举合约                                     │
│     - 域名服务合约                                       │
│     - 元池合约（质押池）                                 │
│     - 收集者合约（Gas 费用收集）                         │
│                                                         │
│  3. Shard 状态证明                                       │
│     - 验证各分片的正确性                                  │
│     - 生成全局状态根哈希                                   │
│                                                         │
│  4. 验证者选举                                          │
│     - 管理验证者集合                                      │
│     - 分配验证任务                                        │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**Masterchain 区块结构：**

```tact
struct MasterchainBlock {
    shard_hashes: ShardHashes;      // 所有 Shardchain 的哈希
    validator_set_hash: int;         // 验证者集合哈希
    global_balance: int;              //  全局余额
    config_params: ConfigParams;     // 网络配置参数
    elect_addr: address;             // 选举地址
}
```

**Masterchain 特点：**
- 只有一个 Masterchain
- 每个区块包含所有 Shardchain 的状态哈希
- 验证者必须在 Masterchain 上运行

#### 2.1.2 Workchains（工作链）

**Workchains** 是实际处理用户交易和智能合约的链。

**设计理念：**

```
┌─────────────────────────────────────────────────────────┐
│                   Masterchain                           │
│                         │                               │
│          ┌──────────────┼──────────────┐                │
│          ↓              ↓              ↓                │
│    ┌──────────┐  ┌──────────┐  ┌──────────┐            │
│    │Workchain 0│  │Workchain 1│  │Workchain N│            │
│    │ (Basechain)│ │(自定义链) │  │(未来扩展) │            │
│    └──────────┘  └──────────┘  └──────────┘            │
└─────────────────────────────────────────────────────────┘
```

**Workchain 类型：**

| Workchain ID | 类型 | 说明 | 状态 |
|--------------|------|------|------|
| **0** | Basechain | 主工作链，支持智能合约 | 已上线 |
| **-1** | 士币链 | 简化链，无智能合约 | 规划中 |

**Workchain 配置参数：**

```tact
struct WorkchainDescr {
    workchain_id: int;                    // 链 ID
    enabled_since: int;                   // 启用时间
    min_split_bits: int;                  // 最小分片位数
    max_split_bits: int;                  // 最大分片位数
    basic: bool;                          // 是否为基础链
    active: bool;                          // 是否激活
    accept_msgs: bool;                      // 是否接受消息
    flags: int;                            // 配置标志
    zerostate_root_hash: int;             // 零状态根哈希
    zerostate_file_hash: int;             // 零状态文件哈希
}
```

**为什么只有一个 Basechain？**

TON 的扩展策略是通过 Shardchain（分片链）而不是增加 Workchain：
- 简化跨链通信
- 保持网络一致性
- 专注于单链的极致优化

#### 2.1.3 Shardchains（分片链）

**Shardchains** 是动态分片的执行单元，根据负载自动拆分和合并。

**动态分片原理：**

```
初始状态（1个 Shardchain）：
┌─────────────────────────────────┐
│     Shard 0 (所有账户)           │
│     (split depth = 0)           │
└─────────────────────────────────┘

负载增加时（自动拆分）：
┌────────────────┬────────────────┐
│  Shard 0       │  Shard 1       │
│  (accounts A-N)│  (accounts O-Z)│
│  (depth = 1)   │  (depth = 1)   │
└────────────────┴────────────────┘

负载继续增加（继续拆分）：
┌────────┬────────┬────────┬────────┐
│Shrd 0  │Shrd 1  │Shrd 2  │Shrd 3  │
│ (A-G)   │(H-N)   │(O-U)   │(V-Z)   │
└────────┴────────┴────────┴────────┘
```

**Shardchain 标识：**

```typescript
// Shardchain 使用(workchain_id, shard_prefix) 标识
interface ShardIdent {
    workchain_id: int16;      // 工作链 ID（通常是 0）
    shard_prefix: uint64;     // 分片前缀
}

// 示例
ShardIdent {
    workchain_id: 0,          // Basechain
    shard_prefix: 0x8000000000000000  // 0x8... = 前半部分
}
```

**分片前缀（Shard Prefix）：**

```
shard_prefix 是一个 64 位整数，高位表示分片范围：

shard_prefix = 0x8000000000000000（二进制：1000...）
表示：账户地址哈希的高位 >= 0x8000... 的账户在此分片

shard_prefix = 0x0000000000000000
表示：所有账户（未分片）

shard_prefix = 0x4000000000000000（二进制：0100...）
表示：账户地址哈希的高位 < 0x4000... 的账户在此分片
```

**分片深度（Split Depth）：**

| 深度 | 分片数 | shard_prefix |
|------|--------|--------------|
| 0 | 1 | 0x0000... |
| 1 | 2 | 0x8000..., 0x0000... |
| 2 | 4 | 0xC000..., 0x8000..., 0x4000..., 0x0000... |
| 3 | 8 | ... |

**自动拆分/合并条件：**

```typescript
// 自动拆分条件
if (shard_block_size > MAX_BLOCK_SIZE && split_depth < MAX_SPLIT_DEPTH) {
    split_shard();  // 拆分为两个子分片
}

// 自动合并条件
if (shard_block_size < MIN_BLOCK_SIZE && split_depth > 0) {
    merge_shard();  // 合并到父分片
}
```

#### 2.1.4 超立方体路由（Hypercube Routing）

超立方体路由是 TON 实现高效跨分片消息传递的核心机制。

**二维超立方体示例：**

```
         Shard 00 ──────── Shard 01
            │     \     /     │
            │       \   /       │
            │       /   \       │
            │     /     \       │
         Shard 10 ──────── Shard 11

每个节点连接到 N 个邻居（4个邻居 = 4维超立方体）
```

**消息传递路径：**

```
从 Shard 00 发送到 Shard 11（跨越2个维度）：

路径1（维度0 → 维度1）：
00 → 01 → 11

路径2（维度1 → 维度0）：
00 → 10 → 11

TON 选择最短路径（2跳）
```

**超立方体路由的优势：**

1. **路径短**：最多 log2(N) 跳
2. **可扩展**：维度增加时路径长度线性增长
3. **负载均衡**：消息分散到不同路径
4. **容错性**：多条路径可选

### 2.2 账户模型

TON 采用基于地址的账户模型，每个账户都有一个唯一的地址。

#### 2.2.1 账户地址格式

**地址结构：**

```
TON 地址由以下部分组成：

┌────────────────────────────────────────────┐
│            完整地址格式                      │
├────────────────────────────────────────────┤
│                                            │
│  [1 byte: flags]                            │
│  [1 byte: workchain_id]                    │
│  [32 bytes: account_id (hash)]               │
│  [4 bytes: CRC32 (可选)]                    │
│  [1 byte: version]                          │
│                                            │
└────────────────────────────────────────────┘
```

**两种地址格式：**

| 格式 | 说明 | 示例 |
|------|------|------|
| **Raw Address** | 原始二进制格式 | `0:83DFD552638C93F12826229F63F79E1D0F6B3A3B0B0D0D0D0D0D0D0D0D0D0D0D` |
| **User-friendly** | Base64 编码，可读 | `EQDt...xyz` |

**地址转换：**

```typescript
import { Address } from '@ton/core';

// Raw 转 User-friendly
const rawAddr = "0:83dfd552638c93f12826229f63f79e1d0f6b3a3b0b0d0d0d0d0d0d0d0d0d0d0d";
const userFriendly = Address.parse(rawAddr).toString();
// "EQDtFRljyJPxKCYin2N50ND2szoLCw0NDQ0NDQ0NDQ0NDQ0NDQ0NDQ0"

// User-friendly 转 Raw
const addr = Address.parse("EQDtFRljyJPxKCYin2N50ND2szoLCw0NDQ0NDQ0NDQ0NDQ0NDQ0NDQ0");
const raw = addr.toRawString();
// "0:83dfd552638c93f12826229f63f79e1d0f6b3a3b0b0d0d0d0d0d0d0d0d0d0d0d"
```

**Base64 地址变体：**

| 类型 | 前缀 | 说明 |
|------|------|------|
| **bounceable** | `EQ` | 可 bounce 的地址（默认） |
| **non-bounceable** | `UQ` | 不可 bounce 的地址 |
| **testnet bounceable** | `Ef` | 测试网可 bounce |
| **testnet non-bounceable** | `Ef` | 测试网不可 bounce |

#### 2.2.2 Bounceable 与 Non-bounceable 地址

**Bounce 机制：**

```
Bounceable 地址（默认）：
A → B（发送消息）
    ↓
    如果 B 不存在或消息失败
    ↓
    消息退回给 A（bounce）

Non-bounceable 地址：
A → B（发送消息）
    ↓
    如果 B 不存在
    ↓
    消息被销毁，余额不退回
```

**使用场景：**

| 地址类型 | 使用场景 | 示例 |
|----------|----------|------|
| **bounceable** | 普通合约和钱包 | 推荐使用 |
| **non-bounceable** | 特定系统合约 | 域名解析器、金库等 |

**生成不同类型的地址：**

```typescript
import { Address } from '@ton/core';

const addr = Address.parse("EQDtFRljyJPxKCYin2N50ND2szoLCw0NDQ0NDQ0NDQ0NDQ0NDQ0NDQ0");

// Bounceable（默认）
console.log(addr.toString());  // EQDt...

// Non-bounceable
console.log(addr.toString({ bounceable: false }));  // UQDt...

// 测试网
console.log(addr.toString({ testOnly: true }));  // EfDt...
```

#### 2.2.3 账户状态

TON 中的账户有四种可能的状态：

**状态转换图：**

```
┌────────────┐
│  nonexistent │ ← 新地址，未初始化
└──────┬─────┘
       │ 首次接收消息或部署合约
       ↓
┌────────────┐
│  uninit    │ ← 已创建，未初始化
└──────┬─────┘
       │ 执行 init 逻辑
       ↓
┌────────────┐
│  active    │ ← 正常运行
└──────┬─────┘
       │ 被冻结或账户删除
       ↓
┌────────────┐
│  frozen    │ ← 已冻结
└────────────┘
```

**详细状态说明：**

| 状态 | 代码 | 说明 | 存储状态 | 处理消息 |
|------|------|------|----------|----------|
| **nonexistent** | 0 | 账户不存在 | 无 | 创建账户 |
| **uninit** | 1 | 已创建但未初始化 | 有 | 执行 init |
| **active** | 2 | 正常运行 | 有 | 处理消息 |
| **frozen** | 3 | 已冻结 | 无 | 拒绝消息 |

**状态详解：**

**1. Nonexistent（不存在）**
```
账户余额：0
代码：无
数据：无

当收到第一条消息时：
- 如果是部署合约：创建新账户，状态变为 uninit
- 如果是转账：创建新账户（简单钱包），状态变为 active
```

**2. Uninit（未初始化）**
```
账户余额：>= 0
代码：有
数据：有（但未初始化）

必须执行 init 逻辑才能变为 active
```

**3. Active（活跃）**
```
账户余额：> 0
代码：合约代码
数据：持久化数据

正常处理所有消息
```

**4. Frozen（冻结）**
```
账户余额：0
代码：有
数据：有（哈希）

触发条件：
- 存储费用欠费
- 显式调用 freeze

恢复：发送消息触发 thaw
```

#### 2.2.4 智能合约的持久化存储

TON 合约使用 Cell 结构来存储持久化数据。

**存储结构：**

```tact
// Tact 合约的存储结构
contract MyContract {
    owner: Address;      // 存储在 c4寄存器的第一个字段
    counter: Int;        // 存储在 c4寄存器的第二个字段
    data: Cell;          // 存储在 c4寄存器的第三个字段
    
    init() {
        self.owner = sender();
        self.counter = 0;
        self.data = emptyCell();
    }
}
```

**存储布局（Storage Layout）：**

```
合约的持久化数据（c4 寄存器）：

┌─────────────────────────────────────┐
│          Storage Cell               │
├─────────────────────────────────────┤
│                                     │
│  ┌───────────────────────────────┐  │
│  │        Builder 数据            │  │
│  │                               │  │
│  │  [owner: Address]             │  │
│  │  [counter: Int]               │  │
│  │  [data_hash: Cell]            │  │
│  │                               │  │
│  └───────────────────────────────┘  │
│              │                       │
│              ↓                       │
│  ┌───────────────────────────────┐  │
│  │        引用 Cell               │  │
│  │                               │  │
│  │  [额外数据或嵌套结构]           │  │
│  │                               │  │
│  └───────────────────────────────┘  │
│                                     │
└─────────────────────────────────────┘
```

**存储费用计算：**

```
存储费用 = 存储单元数 × 单价 × 时间

- 每个 bit：0.000000001 TON/天
- 每个 cell：每个引用额外计费
- 最低费用：约 0.00000001 TON/天
```

### 2.3 消息机制

TON 的消息机制是其异步执行模型的核心。

#### 2.3.1 消息类型

TON 有三种类型的消息：

**消息类型对比：**

| 类型 | 方向 | 来源 | 目的地 | 特点 |
|------|------|------|--------|------|
| **Internal** | 合约 → 合约 | 合约 | 合约/钱包 | 携带 Toncoin，可触发执行 |
| **External In** | 外部 → 合约 | 外部 | 合约 | 无 Toncoin，可触发执行 |
| **External Out** | 合约 → 外部 | 合约 | 外部 | 通知外部系统 |

**Internal Message（内部消息）：**

```typescript
interface InternalMessage {
    header: {
        info: CommonMsgInfo;  // 公共信息
        ihr_disabled: boolean; // IHR（即时超路由）是否禁用
        bounce: boolean;        // 是否可 bounce
        bounced: boolean;      // 是否是 bounce 消息
        src: Address;          // 发送者地址
        dst: Address;         // 接收者地址
        value: CurrencyCollection; // 转账金额
        ihr_fee: Int;          // IHR 费用
        fwd_fee: Int;          // 转发费用
        created_lt: Int;      // 创建逻辑时间
        created_at: Int;       // 创建时间戳
    };
    body: Cell;                // 消息体
    state_init: Cell?;         // 初始化代码（部署时）
}
```

**External In Message（外部入站消息）：**

```typescript
interface ExternalInMessage {
    header: {
        src: Address;          // 发送者地址（可为任意）
        dst: Address;         // 接收者地址
        import_fee: Int;      // 导入费用
    };
    body: Cell;               // 消息体
}
```

**External Out Message（外部出站消息）：**

```typescript
interface ExternalOutMessage {
    header: {
        src: Address;          // 发送者地址
        dst: Address;         // 接收者地址（可为任意）
        created_lt: Int;      // 创建逻辑时间
        created_at: Int;      // 创建时间戳
    };
    body: Cell;               // 消息体
}
```

#### 2.3.2 消息结构

**TL-B 消息定义：**

```tLb
// 内部消息
internal_message$0
    msg:Message X
    = ExternalOrInternalMessage;

// 外部消息
external_message$1
    src:MsgAddressExt
    dst:MsgAddressInt
    import_fee:Grams
    body:StateInitLib
    = ExternalOrInternalMessage;

// 消息结构
message$0
    info:CommonMsgInfo
    init:StateInit?
    body:Cell
= Message X;
```

**消息体编码：**

```typescript
import { beginCell, storeUint } from '@ton/core';

// 编码消息体
const body = beginCell()
    .storeUint(0x12345678, 32)  // 操作码
    .storeUint(0, 64)            // query_id
    .storeCoins(1000000000n)     // 转账金额
    .storeAddress(recipient)      // 接收者
    .endCell();
```

#### 2.3.3 消息模式

**普通消息：**

```typescript
// 发送普通消息
await contract.send(
    sender,
    { value: toNano('0.05') },
    { $$type: 'Transfer', amount: toNano('1') }
);
```

**携带 Payload 的消息：**

```typescript
// 发送携带自定义 payload 的消息
const payload = beginCell()
    .storeUint(1, 32)           // 操作码
    .storeUint(0, 64)          // query_id
    .storeRef(additionalData) // 附加数据
    .endCell();

await contract.send(
    sender,
    { 
        value: toNano('0.05'),
        payload: payload
    },
    { $$type: 'CustomOp' }
);
```

**携带 StateInit 的消息（部署合约）：**

```typescript
import { StateInit } from '@ton/core';

// 创建合约的初始状态
const stateInit = contractStateInit();

await contract.send(
    sender,
    {
        value: toNano('0.1'),
        stateInit: stateInit
    },
    { $$type: 'Deploy' }
);
```

#### 2.3.4 异步消息模型

TON 的异步模型是其高性能的关键：

**同步 vs 异步执行：**

```
同步执行（Ethereum）：
┌──────────────────────────────────────────┐
│ 合约 A 调用合约 B                          │
│                                          │
│ A ──→ B ──→ C ──→ D                      │
│      ↑      ↑      ↑                      │
│      │      │      │                      │
│   等待    等待    等待                     │
│                                          │
│ 总时间 = T_A + T_B + T_C + T_D            │
└──────────────────────────────────────────┘

异步执行（TON）：
┌──────────────────────────────────────────┐
│ 合约 A 发送多条消息                        │
│                                          │
│ A ──┬─→ B                                 │
│     ├─→ C                                 │
│     └─→ D                                 │
│                                          │
│ A 执行完成后立即返回                        │
│ B、C、D 可并行或串行执行                    │
│                                          │
│ 总时间 = T_A + max(T_B, T_C, T_D)         │
└──────────────────────────────────────────┘
```

**异步执行示例：**

```tact
// Tact 合约示例
contract AsyncExample {
    owner: Address;
    
    // 同步执行：等待结果
    fun syncCall(): Int {
        // 这会等待远程调用完成
        let result = self.remoteContract.getValue();
        return result + 1;
    }
    
    // 异步执行：发送消息不等待
    receive(msg: TriggerBatch) {
        // 发送多个消息，不等待响应
        send(SendParameters{
            to: self.contract1,
            value: msg.amount / 3,
            body: "process".asComment()
        });
        
        send(SendParameters{
            to: self.contract2,
            value: msg.amount / 3,
            body: "process".asComment()
        });
        
        send(SendParameters{
            to: self.contract3,
            value: msg.amount / 3,
            body: "process".asComment()
        });
    }
}
```

**消息顺序保证：**

```
同一对账户间的消息顺序保证：
A → B 的消息顺序
Msg1 → Msg2 → Msg3  （按发送顺序执行）

但：
- A → B 和 C → B 的相对顺序不保证
- 跨分片消息顺序可能不同步
```

#### 2.3.5 Bounce 机制与错误处理

**Bounce 机制：**

当消息无法成功执行时，消息会被"弹回"（bounce）给发送者：

```typescript
// 发送 bounceable 消息（默认）
await contract.send(
    sender,
    {
        value: toNano('1'),
        bounce: true  // 可 bounce
    },
    { $$type: 'Transfer', amount: toNano('1') }
);

// 发送 non-bounceable 消息
await contract.send(
    sender,
    {
        value: toNano('1'),
        bounce: false  // 不可 bounce
    },
    { $$type: 'Transfer', amount: toNano('1') }
);
```

**Bounce 处理流程：**

```
1. A 发送消息给 B
   ↓
2. B 执行失败（exit_code != 0）
   ↓
3. B 创建 bounce 消息
   ↓
4. bounce 消息返回给 A
   ↓
5. A 的合约可以处理 bounce
```

**Tact 中的 Bounce 处理：**

```tact
contract BounceHandler {
    pendingTransfers: map<Int, TransferInfo>;
    
    // 正常处理消息
    receive(msg: Transfer) {
        // 处理转账
        self.processTransfer(msg);
    }
    
    // 处理 bounce 消息
    bounced(src: Address, msg: bounced<Transfer>) {
        // 从待处理列表中移除
        let queryId = msg.queryId;
        self.pendingTransfers.delete(queryId);
        
        // 恢复余额或重试逻辑
        // ...
    }
}
```

**Bounce 条件：**

| 条件 | 是否 Bounce | 说明 |
|------|-------------|------|
| 合约成功执行 | ❌ | 正常完成 |
| 合约抛出异常 | ✅ | 返回失败消息 |
| 目标不存在 | ✅ | 创建 bounce 消息 |
| Gas 耗尽 | ✅ | 返回未消费的余额 |
| Bounce=false | ❌ | 消息被销毁 |

### 2.4 Gas 与费用模型

TON 使用 Gas 来计量合约执行的资源消耗。

#### 2.4.1 Gas 计量与转换

**Gas 单位：**

```
Gas 是 TON 的计算单位，用于衡量合约执行的资源消耗。

1 Gas ≈ 执行一条简单指令的成本
复杂操作消耗更多 Gas
```

**Gas 到 Toncoin 的转换：**

```typescript
// Gas 转换公式
const gasPrice = 1024n;  // 每 1000 Gas 的价格（nanoTON）
const gasConsumed = 10000n;  // 消耗的 Gas

const fee = (gasConsumed * gasPrice) / 1000n;
console.log(`费用: ${fee} nanoTON = ${Number(fee) / 1e9} TON`);
```

**Gas 限制：**

```typescript
// 交易可以指定 Gas 限制
interface SendParameters {
    value: bigint;          // 发送金额
    sendMode?: number;      // 发送模式
    body?: Cell;            // 消息体
    code?: Cell;            // 新代码（升级）
    data?: Cell;            // 新数据
    stateInit?: Cell;      // 状态初始化
}

// 默认 Gas 限制
const defaultGasLimit = 10000000n;  // 10M Gas
```

**Gas 消耗来源：**

```
总 Gas = 计算 Gas + 存储 Gas + 转发 Gas

计算 Gas：
- 指令执行
- 存储读写

存储 Gas：
- 数据存储
- Cell 创建

转发 Gas：
- 消息转发
- 跨分片通信
```

#### 2.4.2 执行费用、存储费用、转发费用

**1. 执行费用（Compute Fees）**

```typescript
// 计算费用示例
const computeFee = (gasUsed: bigint): bigint => {
    const pricePerThousand = 1024n;  // nanoTON per 1000 gas
    return (gasUsed * pricePerThousand) / 1000n;
};

// 常见操作的 Gas 消耗
const operations = {
    'ADD': 10n,
    'MUL': 10n,
    'HASH': 100n,
    'SENDMSG': 5000n,
    'CREATECTOR': 10000n
};
```

**2. 存储费用（Storage Fees）**

```typescript
// 存储费用计算
const storageFee = (
    bits: number,        // 使用的位数
    cells: number,      // 使用的 Cell 数
    seconds: number     // 存储时间（秒）
): bigint => {
    const bitPrice = 1n;           // 每位每日的费用（nanoTON）
    const cellPrice = 100n;       // 每个 Cell 每日的费用
    const days = seconds / 86400;
    
    return BigInt(bits) * bitPrice * BigInt(days) +
           BigInt(cells) * cellPrice * BigInt(days);
};

// 示例
const fee = storageFee(1000, 5, 86400);  // 1天，1000位，5个Cell
console.log(`存储费用: ${fee} nanoTON`);
```

**存储费用详情：**

| 资源 | 单价 | 说明 |
|------|------|------|
| 每 bit | 1 nanoTON/天 | 数据位 |
| 每 cell | 100 nanoTON/天 | Cell 数量 |
| 每引用 | 500 nanoTON/天 | Cell 引用 |

**3. 转发费用（Forward Fees）**

```typescript
// 转发费用计算
const forwardFee = (
    msgSize: { bits: number; refs: number },
    dstWorkchain: number,
    isIhrEnabled: boolean
): bigint => {
    // 基础费用
    let baseFee = msgSize.bits * 1n + msgSize.refs * 5000n;
    
    // 跨分片费用加成
    if (dstWorkchain !== currentWorkchain) {
        baseFee *= 2n;
    }
    
    // IHR 费用（如果启用）
    if (isIhrEnabled) {
        baseFee += (msgSize.bits + msgSize.refs * 100) * 10n;
    }
    
    return baseFee;
};
```

**费用对比：**

| 费用类型 | 计算方式 | 支付方 |
|----------|----------|--------|
| **执行费用** | Gas 消耗 × 单价 | 消息发送者 |
| **存储费用** | 位数 × 时间 × 单价 | 合约所有者 |
| **转发费用** | 消息大小 × 距离 × 单价 | 消息发送者 |

#### 2.4.3 Gas 优化策略概述

**优化原则：**

```
1. 减少计算量
   ↓
   简化算法，减少指令数

2. 减少存储使用
   ↓
   优化数据结构，复用数据

3. 减少消息大小
   ↓
   压缩数据，减少引用
```

**优化技巧：**

```tact
// ❌ 低效：重复计算
fun getTotalBad(items: map<Int, Item>): Int {
    var sum = 0;
    foreach (id, item in items) {
        sum = sum + self.calculateValue(item);  // 每次调用都计算
    }
    return sum;
}

// ✅ 高效：缓存计算结果
fun updateItem(id: Int, item: Item) {
    let oldValue = self.itemValues.get(id) ?? 0;
    let newValue = self.calculateValue(item);
    
    self.items.set(id, item);
    self.totalValue = self.totalValue - oldValue + newValue;  // 增量更新
}

fun getTotalGood(): Int {
    return self.totalValue;  // O(1)
}
```

**数据结构优化：**

```tact
// ❌ 低效：使用多个单独的变量
contract BadContract {
    value1: Int;
    value2: Int;
    value3: Int;
    value4: Int;
    
    // 需要4个 Cell 引用
}

// ✅ 高效：打包到同一个 Cell
contract GoodContract {
    packedValues: Int;  // 将多个值打包到一个整数
    
    fun getValue1(): Int { return self.packedValues & 0xFF; }
    fun getValue2(): Int { return (self.packedValues >> 8) & 0xFF; }
    fun getValue3(): Int { return (self.packedValues >> 16) & 0xFF; }
    fun getValue4(): Int { return (self.packedValues >> 24) & 0xFF; }
}
```

**消息优化：**

```tact
// ❌ 低效：多个小消息
foreach (item in items) {
    send(SendParameters{ to: target, value: 0, body: item });
}

// ✅ 高效：合并为一个大消息
let batch = beginCell();
foreach (item in items) {
    batch = batch.storeRef(item);
}
send(SendParameters{ to: target, value: 0, body: batch.endCell() });
```

---

**本章小结：**

本章深入介绍了 TON 的核心架构：
- **2.1 节**：解析了 TON 的多层链架构（Masterchain、Workchain、Shardchain）和超立方体路由机制
- **2.2 节**：学习了 TON 的账户模型，包括地址格式、bounce 机制和账户状态
- **2.3 节**：掌握了 TON 的消息机制，包括消息类型、结构和异步执行模型
- **2.4 节**：了解了 Gas 与费用模型，包括费用计算和基本优化策略

这些核心概念是理解 TON 工作原理的基础，对于开发高效的智能合约至关重要。

---
# 第 3 章：TON 数据原语

TON 区块链使用独特的数据原语系统来存储和传输数据。本章将深入讲解 Cell、Slice、Builder 以及 Bag of Cells（BoC）和 TL-B 序列化方案。

### 3.1 Cell（单元格）

Cell 是 TON 区块链中最基础的数据存储单元，所有数据都以 Cell 的形式存储和传输。

#### 3.1.1 Cell 结构

**Cell 的基本结构：**

```
┌─────────────────────────────────────────────────────────┐
│                      Cell 结构                          │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │              数据部分 (Data)                     │   │
│  │                                                 │   │
│  │  最多 1023 bits（约 128 字节）                    │   │
│  │  存储实际数据（整数、地址、标志位等）              │   │
│  │                                                 │   │
│  └─────────────────────────────────────────────────┘   │
│                                                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │              引用部分 (References)               │   │
│  │                                                 │   │
│  │  最多 4 个引用                                    │   │
│  │  每个引用指向另一个 Cell                          │   │
│  │  形成 Cell 树结构                                 │   │
│  │                                                 │   │
│  └─────────────────────────────────────────────────┘   │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**Cell 容量限制：**

| 属性 | 最大值 | 说明 |
|------|--------|------|
| **数据位** | 1023 bits | 约 128 字节 |
| **引用数** | 4 个 | 指向其他 Cell |
| **总容量** | 1023 bits + 4 refs | 形成树状结构 |

**为什么 1023 位？**

```
Cell 大小设计考虑：
1. 1023 是质数，有利于哈希计算
2. 1023 = 1024 - 1，接近 2^10
3. 留出 1 bit 用于 Cell 类型标记
4. 4 个引用可以形成平衡的树结构
```

#### 3.1.2 Cell 树与有向无环图（DAG）

Cell 通过引用形成树状结构，多个 Cell 可以共享子 Cell 形成 DAG：

**树状结构示例：**

```
┌─────────┐
│  Root   │
│  Cell   │
└────┬────┘
     │
     ├────────┬────────┬────────┐
     │        │        │        │
     ▼        ▼        ▼        ▼
┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐
│ Child 1│ │ Child 2│ │ Child 3│ │ Child 4│
└────────┘ └────────┘ └────────┘ └────────┘
     │
     ├────────┬────────┐
     │        │        │
     ▼        ▼        ▼
┌────────┐ ┌────────┐ ┌────────┐
│GrandCh1│ │GrandCh2│ │GrandCh3│
└────────┘ └────────┘ └────────┘
```

**DAG 结构示例（共享子 Cell）：**

```
┌─────────┐         ┌─────────┐
│  Cell A │         │  Cell B │
└────┬────┘         └────┬────┘
     │                   │
     └─────────┬─────────┘
               │
               ▼
          ┌─────────┐
          │ Shared  │
          │  Cell   │
          └─────────┘
```

**DAG 的优势：**

1. **数据共享**：相同数据只需存储一次
2. **节省空间**：减少重复数据
3. **高效更新**：修改共享节点影响所有引用者

#### 3.1.3 Cell 哈希与 Merkle 证明

**Cell 哈希计算：**

```typescript
import { Cell } from '@ton/core';

// 创建 Cell
const cell = beginCell()
    .storeUint(123, 32)
    .storeAddress(address)
    .endCell();

// 计算哈希
const hash = cell.hash();  // 256-bit (32 bytes) SHA-256
console.log('Cell hash:', hash.toString('hex'));
```

**哈希计算过程：**

```
Cell Hash = SHA256(
    cell_type + 
    data_bits + 
    data_content + 
    ref_count + 
    ref_hashes...
)
```

**Merkle 证明：**

Merkle 证明用于验证 Cell 树中某部分数据的真实性，而无需暴露整个树：

```
原始 Cell 树：
┌─────────┐
│   Root  │  Hash = H(Root)
└────┬────┘
     │
     ├────────┬────────┐
     │        │        │
     ▼        ▼        ▼
┌────────┐ ┌────────┐ ┌────────┐
│   A    │ │   B    │ │   C    │
└────────┘ └────────┘ └────────┘

Merkle 证明（验证 B）：
提供：
1. B 的内容
2. A 的哈希
3. C 的哈希
4. Root 的哈希（已知）

验证：
计算 H(B) + H(A) + H(C) = H(Root) ?
```

**Merkle 证明应用：**

```typescript
// 创建 Merkle 证明
function createMerkleProof(cell: Cell, path: number[]): Cell {
    // path: 从根到目标节点的路径
    // 返回包含证明的 Cell
}

// 验证 Merkle 证明
function verifyMerkleProof(
    rootHash: Buffer,
    proof: Cell,
    targetData: Cell
): boolean {
    // 验证证明是否匹配根哈希
}
```

#### 3.1.4 异构 Cell（Exotic Cell）

除了普通 Cell，TON 还支持几种特殊的异构 Cell：

**异构 Cell 类型：**

| 类型 | 代码 | 用途 |
|------|------|------|
| **Pruned Branch** | 1 | 裁剪分支，用于轻节点 |
| **Library Reference** | 2 | 引用共享代码库 |
| **Merkle Proof** | 3 | Merkle 证明节点 |
| **Merkle Update** | 4 | Merkle 更新证明 |

**Pruned Branch（裁剪分支）：**

```
完整 Cell 树：
┌─────────┐
│  Root   │
└────┬────┘
     │
     ├────────┬────────┐
     │        │        │
     ▼        ▼        ▼
┌────────┐ ┌────────┐ ┌────────┐
│   A    │ │   B    │ │   C    │
└────────┘ └────────┘ └────────┘

轻节点视图（裁剪 C）：
┌─────────┐
│  Root   │
└────┬────┘
     │
     ├────────┬────────────────┐
     │        │                │
     ▼        ▼                ▼
┌────────┐ ┌────────┐ ┌────────────────┐
│   A    │ │   B    │ │ Pruned Branch  │
└────────┘ └────────┘ │  (C 的哈希)    │
                      └────────────────┘
```

**Library Reference（库引用）：**

```
共享代码库：
┌─────────────────┐
│  Library Cell   │
│  (共享代码)      │
└─────────────────┘
         ▲
         │ 引用
    ┌────┴────┐
    │         │
    ▼         ▼
┌───────┐ ┌───────┐
│Cell A │ │Cell B │
└───────┘ └───────┘
```

### 3.2 Slice 与 Builder

Slice 和 Builder 是操作 Cell 的两个核心工具：Slice 用于读取，Builder 用于写入。

#### 3.2.1 Slice：Cell 的读取游标

**Slice 概念：**

```
Cell 内容：
┌─────────────────────────────────────────┐
│ 10110001 00101100 11001100 10101010 ... │
└─────────────────────────────────────────┘

Slice（读取游标）：
┌─────────────────────────────────────────┐
│ ▶10110001 00101100 11001100 10101010 ...│
└─────────────────────────────────────────┘
  ↑
  当前读取位置

读取 8 位后：
┌─────────────────────────────────────────┐
│ 10110001 ▶00101100 11001100 10101010 ...│
└─────────────────────────────────────────┘
            ↑
            游标移动
```

**Slice 基本操作：**

```typescript
import { beginCell, Slice } from '@ton/core';

// 创建 Cell
const cell = beginCell()
    .storeUint(42, 32)           // 存储 32 位整数
    .storeAddress(address)        // 存储地址
    .storeRef(nestedCell)         // 存储引用
    .endCell();

// 创建 Slice
const slice = cell.beginParse();

// 读取数据
const value = slice.loadUint(32);      // 读取 32 位整数
const addr = slice.loadAddress();       // 读取地址
const ref = slice.loadRef();            // 读取引用

// 检查是否读完
const isEmpty = slice.remainingBits === 0;
const hasRefs = slice.remainingRefs > 0;
```

**Slice 读取方法：**

| 方法 | 说明 | 返回值 |
|------|------|--------|
| `loadUint(bits)` | 读取无符号整数 | bigint |
| `loadInt(bits)` | 读取有符号整数 | bigint |
| `loadBits(bits)` | 读取原始位 | Buffer |
| `loadAddress()` | 读取地址 | Address |
| `loadRef()` | 读取引用 | Cell |
| `loadString()` | 读取字符串 | string |
| `loadCoins()` | 读取金额（变长编码） | bigint |
| `skip(bits)` | 跳过指定位数 | void |

**读取示例：**

```typescript
// 复杂数据结构读取
const cell = beginCell()
    .storeUint(1, 8)              // 版本号
    .storeUint(123456789, 64)     // 时间戳
    .storeAddress(sender)         // 发送者
    .storeCoins(toNano('1.5'))    // 金额
    .storeRef(payload)            // 附加数据
    .endCell();

const slice = cell.beginParse();

const version = slice.loadUint(8);
const timestamp = slice.loadUint(64);
const sender = slice.loadAddress();
const amount = slice.loadCoins();
const payload = slice.loadRef();

console.log({
    version: Number(version),
    timestamp: new Date(Number(timestamp) * 1000),
    sender: sender.toString(),
    amount: Number(amount) / 1e9 + ' TON'
});
```

#### 3.2.2 Builder：构建新 Cell 的写入游标

**Builder 概念：**

```
Builder 构建过程：

初始状态：
┌─────────────────────────────────────────┐
│ ▶                                       │
└─────────────────────────────────────────┘
  ↑
  空 Builder

存储 8 位：
┌─────────────────────────────────────────┐
│ 10110001 ▶                              │
└─────────────────────────────────────────┘
            ↑
            写入后游标位置

存储 32 位整数：
┌─────────────────────────────────────────┐
│ 10110001 00000000 00000000 00000000 ▶   │
│ 00101010                                │
└─────────────────────────────────────────┘
                                      ↑

结束构建：
┌─────────────────────────────────────────┐
│ 10110001 00000000 00000000 00000000     │
│ 00101010                                │
└─────────────────────────────────────────┘
              ↓
         生成 Cell
```

**Builder 基本操作：**

```typescript
import { beginCell, Address } from '@ton/core';

// 创建 Builder
const builder = beginCell();

// 存储数据
builder.storeUint(42, 32);           // 存储 32 位整数
builder.storeInt(-100, 16);          // 存储 16 位有符号整数
builder.storeAddress(address);        // 存储地址
builder.storeRef(nestedCell);         // 存储引用
builder.storeCoins(toNano('1.5'));   // 存储金额

// 结束构建
const cell = builder.endCell();
```

**Builder 存储方法：**

| 方法 | 说明 | 参数 |
|------|------|------|
| `storeUint(value, bits)` | 存储无符号整数 | bigint/number, bits |
| `storeInt(value, bits)` | 存储有符号整数 | bigint/number, bits |
| `storeBits(buffer)` | 存储原始位 | Buffer |
| `storeAddress(address)` | 存储地址 | Address |
| `storeRef(cell)` | 存储引用 | Cell |
| `storeString(str)` | 存储字符串 | string |
| `storeCoins(amount)` | 存储金额 | bigint |
| `storeBuffer(buffer)` | 存储缓冲区 | Buffer |

**链式调用：**

```typescript
// 使用链式调用构建 Cell
const cell = beginCell()
    .storeUint(1, 8)              // 操作码
    .storeUint(Date.now(), 64)    // 时间戳
    .storeAddress(sender)         // 发送者
    .storeCoins(toNano('0.1'))    // 金额
    .storeRef(
        beginCell()
            .storeUint(123, 32)
            .storeString('data')
            .endCell()
    )                             // 嵌套 Cell
    .endCell();
```

#### 3.2.3 序列化与反序列化操作

**完整序列化示例：**

```typescript
// 定义数据结构
interface TransferMessage {
    opCode: number;
    queryId: bigint;
    amount: bigint;
    recipient: Address;
    payload?: Cell;
}

// 序列化
function serializeTransfer(msg: TransferMessage): Cell {
    const builder = beginCell()
        .storeUint(msg.opCode, 32)
        .storeUint(msg.queryId, 64)
        .storeCoins(msg.amount)
        .storeAddress(msg.recipient);
    
    if (msg.payload) {
        builder.storeRef(msg.payload);
    }
    
    return builder.endCell();
}

// 反序列化
function deserializeTransfer(cell: Cell): TransferMessage {
    const slice = cell.beginParse();
    
    return {
        opCode: Number(slice.loadUint(32)),
        queryId: slice.loadUint(64),
        amount: slice.loadCoins(),
        recipient: slice.loadAddress(),
        payload: slice.remainingRefs > 0 ? slice.loadRef() : undefined
    };
}
```

**处理可变长度数据：**

```typescript
// 存储变长数组
function storeArray(items: bigint[]): Cell {
    const builder = beginCell()
        .storeUint(items.length, 16);  // 先存储长度
    
    for (const item of items) {
        builder.storeUint(item, 64);
    }
    
    return builder.endCell();
}

// 读取变长数组
function loadArray(cell: Cell): bigint[] {
    const slice = cell.beginParse();
    const length = Number(slice.loadUint(16));
    const items: bigint[] = [];
    
    for (let i = 0; i < length; i++) {
        items.push(slice.loadUint(64));
    }
    
    return items;
}
```

### 3.3 Bag of Cells（BoC）

BoC 是 TON 中用于序列化和传输 Cell 树的标准格式。

#### 3.3.1 BoC 序列化格式

**BoC 结构：**

```
┌─────────────────────────────────────────────────────────┐
│                    BoC 格式                              │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │              头部 (Header)                       │   │
│  │                                                 │   │
│  │  [4 bytes: 魔数 "b5ee9c72"]                       │   │
│  │  [1 byte:  标志位]                               │   │
│  │  [1 byte:  Cell 数量（如果 > 1）]                │   │
│  │  [4 bytes: 根 Cell 索引]                         │   │
│  │                                                 │   │
│  └─────────────────────────────────────────────────┘   │
│                                                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │              Cell 数据区                         │   │
│  │                                                 │   │
│  │  Cell 1 数据                                     │   │
│  │  Cell 2 数据                                     │   │
│  │  ...                                            │   │
│  │  Cell N 数据                                     │   │
│  │                                                 │   │
│  └─────────────────────────────────────────────────┘   │
│                                                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │              索引区（可选）                       │   │
│  │                                                 │   │
│  │  Cell 哈希索引表                                 │   │
│  │                                                 │   │
│  └─────────────────────────────────────────────────┘   │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**BoC 头部格式：**

```typescript
interface BoCHeader {
    magic: 0xb5ee9c72;           // 魔数
    hasIndex: boolean;           // 是否有索引
    hasCRC32C: boolean;          // 是否有 CRC32C 校验
    hasCacheBits: boolean;       // 是否有缓存位
    flags: number;               // 标志位
    sizeBytes: number;           // 大小字节数
    offBytes: number;            // 偏移字节数
    cellsCount: number;          // Cell 数量
    rootsCount: number;          // 根 Cell 数量
    absentCount: number;         // 缺失 Cell 数量
    totalCellsSize: number;      // 总 Cell 大小
    rootIndexes: number[];       // 根 Cell 索引
}
```

#### 3.3.2 Cell 在网络传输与文件存储中的表示

**序列化为 BoC：**

```typescript
import { Cell } from '@ton/core';

// Cell 转 BoC（字节数组）
const cell: Cell = ...;
const bocBytes = cell.toBoc();  // Uint8Array

// BoC 转 Base64（用于传输）
const bocBase64 = cell.toBoc().toString('base64');

// BoC 转十六进制
const bocHex = cell.toBoc().toString('hex');
```

**从 BoC 反序列化：**

```typescript
import { Cell } from '@ton/core';

// 从字节数组解析
const bocBytes: Buffer = ...;
const cell = Cell.fromBoc(bocBytes)[0];

// 从 Base64 解析
const bocBase64: string = ...;
const cell = Cell.fromBase64(bocBase64);

// 从十六进制解析
const bocHex: string = ...;
const cell = Cell.fromHex(bocHex);
```

**文件存储：**

```typescript
import { writeFileSync, readFileSync } from 'fs';
import { Cell } from '@ton/core';

// 保存到文件
const cell: Cell = ...;
writeFileSync('contract.boc', cell.toBoc());

// 从文件读取
const bocBytes = readFileSync('contract.boc');
const cell = Cell.fromBoc(bocBytes)[0];
```

### 3.4 TL-B 序列化方案

TL-B（Type Language - Binary）是 TON 使用的类型定义和序列化语言。

#### 3.4.1 TL-B 语法与类型定义

**基本语法：**

```tlb
// 定义类型
name#hash fields = Type;

// 示例：简单消息
transfer#0f8a7ea5 
    query_id:uint64 
    amount:Coins 
    destination:MsgAddress 
    = TransferMsg;
```

**类型修饰符：**

| 修饰符 | 说明 | 示例 |
|--------|------|------|
| `#` | 十六进制前缀 | `transfer#0f8a7ea5` |
| `$` | 字符串 | `name:string` |
| `^` | Cell 引用 | `data:^Cell` |
| `?` | 可选字段 | `payload:Maybe ^Cell` |
| `*` | 变长数组 | `items:(List Int)` |

**基本类型：**

```tlb
// 整数类型
int8      // 8 位有符号整数
uint32    // 32 位无符号整数
int257    // 257 位有符号整数（TON 标准）

// 特殊类型
Bits      // 原始位数据
Cell      // Cell 类型
Address   // TON 地址
Coins     // 金额（变长编码）

// 复合类型
Maybe X   // 可选类型 X
Either X Y // X 或 Y
Both X Y   // X 和 Y
List X     // X 的列表
```

#### 3.4.2 常用 TL-B 结构

**消息结构：**

```tlb
// 内部消息
message$_ 
    info:CommonMsgInfo
    init:(Maybe (Either StateInit ^StateInit))
    body:(Either X ^X)
    = Message X;

// 消息信息
int_msg_info$0 
    ihr_disabled:Bool 
    bounce:Bool 
    bounced:Bool
    src:MsgAddressInt 
    dest:MsgAddressInt 
    value:CurrencyCollection 
    ihr_fee:Grams 
    fwd_fee:Grams 
    created_lt:uint64 
    created_at:uint32
    = CommonMsgInfo;
```

**账户状态：**

```tlb
// 账户状态
account_none$0 = Account;
account$1 
    addr:MsgAddressInt 
    storage_info:AccountStorageInfo 
    storage:AccountStorage 
    = Account;

// 账户存储
account_storage$_ 
    last_trans_lt:uint64 
    balance:CurrencyCollection 
    state:AccountState 
    = AccountStorage;

// 账户状态类型
account_uninit$00 = AccountState;
account_active$1 
    x:StateInit = AccountState;
account_frozen$01 
    state_hash:uint256 = AccountState;
```

**货币集合：**

```tlb
// 货币集合（支持多代币）
CurrencyCollection 
    grams:Grams 
    other:ExtraCurrencyCollection 
    = CurrencyCollection;

// Grams（TON 代币）
// 变长编码：前 4 位表示长度，后面是实际数值
// 0-4 位: 0-30 字节
// 5-248 位: 实际数值
Grams = uint16;  // 简化表示
```

#### 3.4.3 自定义 TL-B 类型编写

**定义自定义消息类型：**

```tlb
// 自定义转账消息
my_transfer#12345678 
    query_id:uint64 
    amount:Coins 
    recipient:MsgAddress 
    payload:(Maybe ^Cell)
    = MyTransfer;

// NFT 转移消息
transfer_nft#5fcc3d14 
    query_id:uint64 
    new_owner:MsgAddress 
    response_destination:MsgAddress 
    custom_payload:(Maybe ^Cell) 
    forward_amount:Coins 
    forward_payload:(Either Cell ^Cell)
    = NFTTransfer;
```

**在 Tact 中使用 TL-B：**

```tact
// Tact 自动处理 TL-B 序列化
message MyTransfer {
    queryId: Int as uint64;
    amount: Int as coins;
    recipient: Address;
    payload: Cell?;
}

contract MyContract {
    receive(msg: MyTransfer) {
        // Tact 自动解析 TL-B 格式的消息
        // ...
    }
}
```

**手动解析 TL-B 消息：**

```typescript
import { Slice } from '@ton/core';

// 解析自定义消息
function parseMyTransfer(slice: Slice): MyTransfer {
    const opCode = slice.loadUint(32);
    if (opCode !== 0x12345678) {
        throw new Error('Invalid op code');
    }
    
    return {
        queryId: slice.loadUint(64),
        amount: slice.loadCoins(),
        recipient: slice.loadAddress(),
        payload: slice.remainingRefs > 0 ? slice.loadRef() : null
    };
}
```

**TL-B 最佳实践：**

```tlb
// 1. 使用明确的操作码前缀
// 好的做法：
transfer#0f8a7ea5 ... = Transfer;

// 不好的做法：
transfer ... = Transfer;  // 没有前缀

// 2. 使用 Maybe 处理可选字段
// 好的做法：
payload:(Maybe ^Cell)

// 3. 使用 Either 处理互斥选项
// 好的做法：
body:(Either X ^X)

// 4. 使用描述性字段名
// 好的做法：
query_id:uint64 amount:Coins

// 不好的做法：
a:uint64 b:Coins
```

---

**本章小结：**

本章深入讲解了 TON 的数据原语系统：
- **3.1 节**：学习了 Cell 的结构、Cell 树与 DAG、哈希计算和 Merkle 证明，以及异构 Cell 类型
- **3.2 节**：掌握了 Slice（读取）和 Builder（写入）的使用方法，以及序列化与反序列化操作
- **3.3 节**：了解了 BoC 序列化格式，以及 Cell 在网络传输和文件存储中的表示
- **3.4 节**：学习了 TL-B 序列化方案的语法和类型定义，以及如何编写自定义 TL-B 类型

理解这些数据原语对于开发 TON 智能合约至关重要，因为所有合约数据都以这些格式存储和传输。

---

## 第 4 章：TON 虚拟机（TVM）

TON 虚拟机（TVM，TON Virtual Machine）是执行 TON 智能合约的核心引擎。本章将深入讲解 TVM 的架构、指令集和执行流程。

### 4.1 TVM 架构概述

TVM 是一个基于栈的虚拟机，专为区块链智能合约执行而设计，具有高效、确定性和可验证的特点。

#### 4.1.1 基于栈的执行模型

**栈式虚拟机特点：**

```
┌─────────────────────────────────────────────────────────┐
│                  TVM 执行模型                            │
├─────────────────────────────────────────────────────────┤
│                                                         │
│    ┌─────────────────────────────────────────────┐     │
│    │              操作数栈（Stack）                │     │
│    │                                             │     │
│    │    ┌─────┐                                  │     │
│    │    │  s0 │  ← 栈顶（最后入栈）               │     │
│    │    ├─────┤                                  │     │
│    │    │  s1 │                                  │     │
│    │    ├─────┤                                  │     │
│    │    │  s2 │                                  │     │
│    │    ├─────┤                                  │     │
│    │    │ ... │                                  │     │
│    │    ├─────┤                                  │     │
│    │    │ sn-1│  ← 栈底                          │     │
│    │    └─────┘                                  │     │
│    │                                             │     │
│    └─────────────────────────────────────────────┘     │
│                         │                               │
│                         ↓                               │
│    ┌─────────────────────────────────────────────┐     │
│    │              当前续体（Continuation）         │     │
│    │                                             │     │
│    │    待执行的指令序列                           │     │
│    │                                             │     │
│    └─────────────────────────────────────────────┘     │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**栈操作示例：**

```
初始状态：栈 = []

执行 PUSH 42：栈 = [42]

执行 PUSH 10：栈 = [42, 10]

执行 ADD：    栈 = [52]  （弹出 42 和 10，压入 52）

执行 PUSH 2： 栈 = [52, 2]

执行 MUL：    栈 = [104] （弹出 52 和 2，压入 104）
```

**栈式 vs 寄存器式虚拟机：**

| 特性 | 栈式（TVM） | 寄存器式（如 EVM） |
|------|-------------|-------------------|
| **指令长度** | 短（无需指定操作数位置） | 长（需指定寄存器） |
| **执行效率** | 较低（频繁栈操作） | 较高 |
| **代码密度** | 高（指令紧凑） | 低 |
| **编译复杂度** | 高（需优化栈使用） | 低 |
| **适用场景** | 区块链（代码大小敏感） | 通用计算 |

#### 4.1.2 TVM 数据类型

TVM 支持 7 种基本数据类型：

**数据类型概览：**

| 类型 | 说明 | 用途 |
|------|------|------|
| **Integer** | 257 位有符号整数 | 数值计算、地址、金额 |
| **Cell** | 数据单元 | 存储复杂数据结构 |
| **Slice** | Cell 的只读视图 | 解析 Cell 数据 |
| **Builder** | Cell 构造器 | 构建 Cell 数据 |
| **Tuple** | 元组（最多 255 元素） | 组合多个值 |
| **Continuation** | 代码续体 | 控制流、函数调用 |
| **Null** | 空值 | 表示缺失或无效 |

**Integer（整数）：**

```
TVM 整数特点：
- 257 位有符号整数（-2^256 到 2^256 - 1）
- 支持标准算术运算
- 支持位运算和逻辑运算
- 溢出时抛出异常

示例：
257 位范围：-57896044618658097711785492504343953926634992332820282019728792003956564819968
           到 57896044618658097711785492504343953926634992332820282019728792003956564819967
```

**Tuple（元组）：**

```
元组结构：
┌─────────────────────────────────┐
│          Tuple                  │
├─────────────────────────────────┤
│  [0]: 元素 0（任意类型）         │
│  [1]: 元素 1（任意类型）         │
│  [2]: 元素 2（任意类型）         │
│  ...                            │
│  [n]: 元素 n（最多 255 个）      │
└─────────────────────────────────┘

示例：
t = (42, "hello", cell1)  // 包含整数、字符串、Cell 的元组
t[0] = 42
t[1] = "hello"
t[2] = cell1
```

**Continuation（续体）：**

```
续体表示待执行的代码：

┌─────────────────────────────────┐
│       Continuation              │
├─────────────────────────────────┤
│  code: 指令序列                  │
│  stack: 关联的栈状态（可选）      │
│  cp: 当前位置指针                │
└─────────────────────────────────┘

用途：
1. 函数调用：保存返回地址
2. 异常处理：保存异常处理器
3. 循环：保存循环体
```

#### 4.1.3 控制寄存器（c0-c5、c7）

TVM 有 7 个控制寄存器，用于管理执行状态：

**控制寄存器列表：**

| 寄存器 | 名称 | 说明 |
|--------|------|------|
| **c0** | 当前续体 | 当前正在执行的代码 |
| **c1** | 调用栈 | 函数调用返回地址 |
| **c2** | 异常处理器 | 异常处理续体 |
| **c3** | 字典 | 合约代码字典 |
| **c4** | 持久数据 | 合约持久化存储 |
| **c5** | 动作列表 | 待执行的外部动作 |
| **c7** | 临时数据 | 合约临时数据（c7） |

**寄存器详解：**

**c0（当前续体）：**

```
c0 保存当前正在执行的代码续体：

┌─────────────────────────────────┐
│        c0（Current）            │
├─────────────────────────────────┤
│                                 │
│  待执行的指令序列                 │
│                                 │
│  例如：                          │
│  PUSH 1                         │
│  PUSH 2                         │
│  ADD                            │
│  ...                            │
│                                 │
└─────────────────────────────────┘
```

**c1（调用栈）：**

```
c1 保存函数调用的返回地址：

函数调用前：
c1 = [return_addr_1, return_addr_2, ...]

调用函数：
1. 将当前 c0 压入 c1
2. 将函数代码加载到 c0
3. 执行函数

函数返回：
1. 从 c1 弹出返回地址
2. 将返回地址加载到 c0
3. 继续执行
```

**c2（异常处理器）：**

```
c2 保存异常处理续体：

正常执行：
执行代码 → 完成

异常发生：
执行代码 → 异常 → 执行 c2 中的异常处理器

设置异常处理器：
c2 = exception_handler_code
```

**c3（字典）：**

```
c3 保存合约的代码字典：

┌─────────────────────────────────┐
│        c3（Dictionary）         │
├─────────────────────────────────┤
│                                 │
│  方法 ID → 方法代码              │
│                                 │
│  例如：                          │
│  0x12345678 → transfer_code     │
│  0x87654321 → withdraw_code     │
│  ...                            │
│                                 │
└─────────────────────────────────┘

用于合约方法分发
```

**c4（持久数据）：**

```
c4 保存合约的持久化存储：

┌─────────────────────────────────┐
│        c4（Persistent Data）    │
├─────────────────────────────────┤
│                                 │
│  合约状态数据（Cell）             │
│                                 │
│  例如：                          │
│  - owner: Address               │
│  - balance: Int                 │
│  - data: Cell                   │
│                                 │
│  交易结束后自动保存               │
│                                 │
└─────────────────────────────────┘
```

**c5（动作列表）：**

```
c5 保存待执行的外部动作：

┌─────────────────────────────────┐
│        c5（Actions）            │
├─────────────────────────────────┤
│                                 │
│  动作列表（Action List）         │
│                                 │
│  例如：                          │
│  - 发送消息                      │
│  - 设置代码                      │
│  - 部署新合约                    │
│  - ...                          │
│                                 │
│  在 action phase 执行            │
│                                 │
└─────────────────────────────────┘
```

**c7（临时数据）：**

```
c7 保存合约的临时数据：

┌─────────────────────────────────┐
│        c7（Temporary）          │
├─────────────────────────────────┤
│                                 │
│  临时数据（Tuple）                │
│                                 │
│  例如：                          │
│  [0]: 当前时间戳                 │
│  [1]: 当前区块高度               │
│  [2]: 配置参数                   │
│  ...                            │
│                                 │
│  交易结束后不保存                 │
│                                 │
└─────────────────────────────────┘
```

#### 4.1.4 Gas 计数器与执行中断

**Gas 计数器：**

TVM 使用 Gas 计数器来限制执行资源：

```
┌─────────────────────────────────────────────────────────┐
│                   Gas 计数器                             │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  gas_limit: 最大 Gas 限制                                │
│  gas_used: 已使用 Gas                                    │
│  gas_remaining = gas_limit - gas_used                   │
│                                                         │
│  每条指令执行前：                                         │
│  if gas_remaining < instruction_cost:                   │
│      throw "Out of gas"                                 │
│  else:                                                  │
│      gas_used += instruction_cost                       │
│      execute instruction                                │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**指令 Gas 成本：**

| 指令类型 | Gas 成本 | 说明 |
|----------|----------|------|
| **栈操作** | 1-10 | PUSH、DROP、DUP 等 |
| **算术运算** | 10-100 | ADD、MUL、DIV 等 |
| **Cell 操作** | 10-500 | NEWC、ENDC、STREF 等 |
| **Hash 计算** | 100-1000 | HASHCU、HASHEXT 等 |
| **字典操作** | 50-500 | DICTGET、DICTSET 等 |
| **消息发送** | 5000+ | SENDMSG |

**执行中断条件：**

```
TVM 执行中断的几种情况：

1. Gas 耗尽：
   gas_remaining < next_instruction_cost
   → exit_code = -14（out of gas）

2. 栈溢出：
   stack_depth > max_stack_depth
   → exit_code = -5（stack overflow）

3. 栈下溢：
   尝试弹出空栈
   → exit_code = -5（stack underflow）

4. 整数溢出：
   运算结果超出 257 位范围
   → exit_code = -4（integer overflow）

5. 类型错误：
   操作数类型不匹配
   → exit_code = -7（type check error）

6. 显式抛出：
   THROW 指令
   → exit_code = user_defined

7. 正常完成：
   所有指令执行完毕
   → exit_code = 0（success）
```

### 4.2 TVM 指令集概览

TVM 指令集包含数百条指令，本节介绍主要类别。

#### 4.2.1 栈操作指令

**基本栈操作：**

| 指令 | 栈效果 | 说明 |
|------|--------|------|
| `PUSH x` | → x | 压入常量 |
| `DROP` | x → | 丢弃栈顶 |
| `DUP` | x → x x | 复制栈顶 |
| `SWAP` | x y → y x | 交换栈顶两个 |
| `NIP` | x y → y | 丢弃次栈顶 |
| `ROT` | x y z → y z x | 循环旋转 |
| `ROTREV` | x y z → z x y | 反向旋转 |

**深度栈操作：**

| 指令 | 栈效果 | 说明 |
|------|--------|------|
| `PICK i` | ... → ... x_i | 取第 i 个元素到栈顶 |
| `ROLL i` | ... x_i → x_i ... | 将第 i 个元素滚到栈顶 |
| `ROLLREV i` | x_i ... → ... x_i | 反向滚动 |
| `DEPTH` | → n | 压入栈深度 |

**示例：**

```
初始栈：[1, 2, 3]（3 在栈顶）

DUP：    [1, 2, 3, 3]
DROP：   [1, 2, 3]
SWAP：   [1, 3, 2]
ROT：    [2, 3, 1]
PICK 1： [1, 2, 3, 2]（取索引 1 的元素，即 2）
```

#### 4.2.2 算术与逻辑指令

**算术运算：**

| 指令 | 栈效果 | 说明 |
|------|--------|------|
| `ADD` | a b → a+b | 加法 |
| `SUB` | a b → a-b | 减法 |
| `MUL` | a b → a*b | 乘法 |
| `DIV` | a b → a/b | 整数除法 |
| `MOD` | a b → a%b | 取模 |
| `DIVMOD` | a b → a/b a%b | 同时返回商和余数 |
| `NEG` | a → -a | 取负 |
| `ABS` | a → |a| | 绝对值 |
| `MAX` | a b → max(a,b) | 最大值 |
| `MIN` | a b → min(a,b) | 最小值 |

**位运算：**

| 指令 | 栈效果 | 说明 |
|------|--------|------|
| `AND` | a b → a&b | 按位与 |
| `OR` | a b → a\|b | 按位或 |
| `XOR` | a b → a^b | 按位异或 |
| `NOT` | a → ~a | 按位取反 |
| `LSHIFT` | a n → a<<n | 左移 |
| `RSHIFT` | a n → a>>n | 右移 |
| `POW2` | n → 2^n | 2 的幂 |

**比较运算：**

| 指令 | 栈效果 | 说明 |
|------|--------|------|
| `EQ` | a b → (a==b) | 相等 |
| `NEQ` | a b → (a!=b) | 不等 |
| `LT` | a b → (a<b) | 小于 |
| `GT` | a b → (a>b) | 大于 |
| `LEQ` | a b → (a<=b) | 小于等于 |
| `GEQ` | a b → (a>=b) | 大于等于 |
| `CMP` | a b → sign(a-b) | 比较（-1/0/1） |

**示例：**

```
计算 (10 + 20) * 2：

PUSH 10    // 栈：[10]
PUSH 20    // 栈：[10, 20]
ADD        // 栈：[30]
PUSH 2     // 栈：[30, 2]
MUL        // 栈：[60]
```

#### 4.2.3 Cell/Slice/Builder 操作指令

**Cell 操作：**

| 指令 | 栈效果 | 说明 |
|------|--------|------|
| `NEWC` | → b | 创建空 Builder |
| `ENDC` | b → c | 完成 Builder，生成 Cell |
| `HASHCU` | c → hash | 计算 Cell 哈希 |
| `CTOS` | c → s | Cell 转 Slice |
| `CLEVEL` | c → level | 获取 Cell 层级 |

**Slice 操作：**

| 指令 | 栈效果 | 说明 |
|------|--------|------|
| `ENDS` | s → | 结束 Slice（验证读完） |
| `LDREFS` | s → s' r | 加载引用 |
| `LDBITS n` | s → s' bits | 加载 n 位 |
| `LDUINT n` | s → s' x | 加载 n 位无符号整数 |
| `LDINT n` | s → s' x | 加载 n 位有符号整数 |
| `SKIPBITS n` | s → s' | 跳过 n 位 |
| `SBITS` | s → n | 获取剩余位数 |
| `SREFS` | s → n | 获取剩余引用数 |

**Builder 操作：**

| 指令 | 栈效果 | 说明 |
|------|--------|------|
| `STREF` | b c → b' | 存储引用 |
| `STSLICE` | b s → b' | 存储 Slice |
| `STBITS n` | b bits → b' | 存储 n 位 |
| `STUINT n` | b x → b' | 存储 n 位无符号整数 |
| `STINT n` | b x → b' | 存储 n 位有符号整数 |
| `STBREF` | b c → b' | 存储引用（带类型检查） |

**示例：构建一个包含整数和引用的 Cell：**

```
NEWC            // 创建空 Builder：[b]
PUSH 42         // 压入整数：[b, 42]
STUINT 32       // 存储 32 位整数：[b']
NEWC            // 创建嵌套 Builder：[b', b2]
PUSH 100        // 压入整数：[b', b2, 100]
STUINT 64       // 存储：[b', b2']
ENDC            // 完成嵌套 Cell：[b', c2]
STREF           // 存储引用：[b'']
ENDC            // 完成最终 Cell：[c]
```

#### 4.2.4 控制流指令

**条件跳转：**

| 指令 | 说明 |
|------|------|
| `IFTRUE cont` | 如果栈顶为真，跳转到 cont |
| `IFFALSE cont` | 如果栈顶为假，跳转到 cont |
| `IFNOT cont` | 如果栈顶为假，跳转到 cont |
| `IF cont1 cont2` | 条件选择执行 |

**循环：**

| 指令 | 说明 |
|------|------|
| `REPEAT n cont` | 重复执行 n 次 |
| `WHILE cond body` | 条件循环 |
| `UNTIL body` | 直到条件为真 |
| `LOOP cont` | 无限循环（需手动退出） |

**函数调用：**

| 指令 | 说明 |
|------|------|
| `CALL cont` | 调用续体 |
| `CALLX` | 调用栈顶的续体 |
| `JMP cont` | 跳转到续体 |
| `JMPX` | 跳转到栈顶的续体 |
| `RET` | 返回调用者 |

**示例：条件执行：**

```
// if (x > 0) { y = x * 2 } else { y = 0 }
PUSH x
PUSH 0
GT              // x > 0 ?
IF<
    PUSH x      // x > 0 分支
    PUSH 2
    MUL         // y = x * 2
>
<
    PUSH 0      // x <= 0 分支
                // y = 0
>
```

#### 4.2.5 字典操作指令

**字典（Dictionary）** 是 TON 中常用的数据结构，用于存储键值对。

**基本字典操作：**

| 指令 | 栈效果 | 说明 |
|------|--------|------|
| `NEWDICT` | → d | 创建空字典 |
| `DICTGET` | d key → d' value -1 或 d 0 | 获取值 |
| `DICTSET` | d key value → d' | 设置值 |
| `DICTDEL` | d key → d' -1 或 d 0 | 删除键 |
| `DICTREPLACE` | d key value → d' -1 或 d 0 | 替换值 |
| `DICTADD` | d key value → d' -1 或 d 0 | 添加值（键不存在时） |

**字典遍历：**

| 指令 | 说明 |
|------|------|
| `DICTFIRST` | 获取第一个键值对 |
| `DICTNEXT` | 获取下一个键值对 |
| `DICTMIN` | 获取最小键 |
| `DICTMAX` | 获取最大键 |

**示例：字典操作：**

```
// 创建字典并设置值
NEWDICT            // 栈：[d]
PUSH 1             // 键：[d, 1]
PUSH 100           // 值：[d, 1, 100]
DICTSET            // 设置：[d']（d'[1] = 100）

PUSH 2             // 键：[d', 2]
PUSH 200           // 值：[d', 2, 200]
DICTSET            // 设置：[d'']（d''[2] = 200）

// 获取值
PUSH 1             // 键：[d'', 1]
DICTGET            // 获取：[d'', 100, -1]（找到）
                   // 或 [d'', 0]（未找到）
```

### 4.3 TVM 执行流程

智能合约在 TVM 中的执行遵循严格的流程。

#### 4.3.1 智能合约的触发与执行

**触发方式：**

```
智能合约触发方式：

1. 内部消息（Internal Message）
   - 来自其他合约或钱包
   - 携带 Toncoin
   - 可触发合约逻辑

2. 外部消息（External In Message）
   - 来自外部（用户、应用）
   - 不携带 Toncoin
   - 需要验证签名

3. Tick/Tock 交易
   - 定时触发（每区块）
   - 用于维护任务
```

**执行入口：**

```
合约执行入口选择：

┌─────────────────────────────────────────────────────────┐
│                    消息类型判断                          │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  if (message.opcode == 0):                              │
│      → 执行 receive(string) 处理器                       │
│                                                         │
│  elif (message.opcode in known_ops):                    │
│      → 执行对应的 receive(msg) 处理器                    │
│                                                         │
│  else:                                                  │
│      → 执行 bounce 处理（如果 bounceable）               │
│      → 或抛出异常                        │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**执行准备：**

```
1. 加载合约代码
   - 从账户状态读取代码 Cell
   - 解析为 TVM 字节码

2. 初始化执行环境
   - 设置 c0 = 合约代码
   - 设置 c4 = 合约数据
   - 设置 c7 = 上下文信息（时间戳、区块高度等）

3. 准备消息
   - 将消息放入栈
   - 设置 Gas 限制

4. 开始执行
   - 从 c0 开始执行指令
```

#### 4.3.2 消息处理阶段（compute phase、action phase）

TON 智能合约执行分为两个主要阶段：

**执行阶段概览：**

```
┌─────────────────────────────────────────────────────────┐
│                   交易执行流程                           │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │         Compute Phase（计算阶段）                 │   │
│  │                                                 │   │
│  │  1. 执行合约代码                                 │   │
│  │  2. 更新合约状态（c4）                           │   │
│  │  3. 生成动作列表（c5）                           │   │
│  │  4. 计算 Gas 消耗                               │   │
│  │                                                 │   │
│  │  输出：exit_code, new_state, actions            │   │
│  └─────────────────────────────────────────────────┘   │
│                         │                               │
│                         ↓                               │
│  ┌─────────────────────────────────────────────────┐   │
│  │         Action Phase（动作阶段）                  │   │
│  │                                                 │   │
│  │  1. 处理 c5 中的动作列表                         │   │
│  │  2. 发送消息                                    │   │
│  │  3. 部署新合约                                  │   │
│  │  4. 更新代码                                    │   │
│  │                                                 │   │
│  │  输出：action_results                          │   │
│  └─────────────────────────────────────────────────┘   │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**Compute Phase（计算阶段）：**

```
计算阶段详细流程：

1. 初始化
   - 设置 Gas 限制
   - 加载合约代码和数据
   - 准备消息参数

2. 执行合约代码
   - TVM 执行指令
   - 处理消息
   - 更新内部状态

3. 状态更新
   - c4（持久数据）更新
   - c5（动作列表）填充

4. 结束计算
   - 返回 exit_code
   - 返回新的合约状态
   - 返回动作列表

可能的 exit_code：
- 0: 成功
- 1-127: 替代成功（特殊处理）
- 128-255: 保留
- < 0: 错误
```

**Action Phase（动作阶段）：**

```
动作阶段详细流程：

1. 验证动作列表
   - 检查动作数量限制
   - 检查消息大小限制
   - 检查余额充足

2. 处理每个动作
   动作类型：
   
   a) 发送消息（Send Message）
      - 创建消息 Cell
      - 扣除发送余额
      - 计算 fwd_fee
      - 将消息加入输出队列
   
   b) 设置代码（Set Code）
      - 更新合约代码
      - 用于合约升级
   
   c) 部署合约（Deploy）
      - 创建新账户
      - 设置初始状态
   
   d) 保留余额（Reserve）
      - 锁定部分余额
      - 用于存储费用

3. 验证最终状态
   - 检查余额非负
   - 检查存储费用充足

4. 提交交易
   - 保存新状态
   - 发出事件
```

**动作列表（c5）结构：**

```
c5 动作列表：

┌─────────────────────────────────────────────────────────┐
│                   Action List                           │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │  Action 1: Send Message                          │   │
│  │    - mode: 发送模式                              │   │
│  │    - msg: 消息 Cell                              │   │
│  └─────────────────────────────────────────────────┘   │
│                                                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │  Action 2: Set Code                              │   │
│  │    - new_code: 新代码 Cell                       │   │
│  └─────────────────────────────────────────────────┘   │
│                                                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │  Action 3: Reserve Balance                      │   │
│  │    - amount: 保留金额                            │   │
│  │    - mode: 保留模式                              │   │
│  └─────────────────────────────────────────────────┘   │
│                                                         │
│  ...                                                    │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**发送模式（Send Mode）：**

| 模式值 | 说明 |
|--------|------|
| 0 | 普通发送 |
| 64 | 携带全部剩余余额 |
| 128 | 携带全部余额并销毁账户 |
| 1 | 忽略错误（+其他模式） |
| 2 | 支付费用单独（+其他模式） |

#### 4.3.3 Exit code 与错误处理

**Exit Code 分类：**

```
Exit Code 范围：

┌─────────────────────────────────────────────────────────┐
│                  Exit Code 分类                          │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  成功（Success）：                                       │
│    0     - 正常成功                                      │
│    1-127 - 替代成功（特殊语义）                           │
│                                                         │
│  TVM 错误（TVM Errors）：                                │
│    -1    - 异常（通过 THROW 抛出）                        │
│    -2    - 栈下溢                                        │
│    -3    - 栈溢出                                        │
│    -4    - 整数溢出                                      │
│    -5    - 整数超出范围                                   │
│    -6    - 无效操作码                                    │
│    -7    - 类型检查错误                                  │
│    -8    - Cell 操作错误                                 │
│    -9    - 字典错误                                      │
│    -10   - 未知的错误                                    │
│    -11   - 不支持的序列化                                │
│    -12   - 未知的续体                                    │
│    -13   - Gas 用尽                                      │
│    -14   - 动作无效                                      │
│                                                         │
│  用户定义（User Defined）：                               │
│    128-255 - 保留给系统                                  │
│    其他    - 用户自定义错误码                             │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**常见 Exit Code 详解：**

| Exit Code | 名称 | 说明 | 处理建议 |
|-----------|------|------|----------|
| **0** | Success | 正常完成 | 无需处理 |
| **-1** | Exception | 显式抛出异常 | 检查 THROW 指令 |
| **-2** | Stack Underflow | 栈元素不足 | 检查栈操作顺序 |
| **-3** | Stack Overflow | 栈过深 | 优化栈使用 |
| **-4** | Integer Overflow | 整数溢出 | 检查运算范围 |
| **-5** | Integer Out of Range | 整数超出预期范围 | 检查位宽限制 |
| **-6** | Invalid Opcode | 无效操作码 | 检查代码生成 |
| **-7** | Type Check Error | 类型不匹配 | 检查数据类型 |
| **-8** | Cell Error | Cell 操作错误 | 检查 Cell 结构 |
| **-9** | Dictionary Error | 字典操作错误 | 检查键类型 |
| **-13** | Out of Gas | Gas 耗尽 | 增加 Gas 限制或优化代码 |
| **-14** | Action Invalid | 动作无效 | 检查动作列表 |

**错误处理示例：**

```tact
// Tact 中的错误处理
contract ErrorHandling {
    receive(msg: Transfer) {
        // 检查条件
        if (msg.amount <= 0) {
            // 抛出错误（exit_code = 123）
            throw(123);
        }
        
        // 检查余额
        if (self.balance < msg.amount) {
            // 抛出错误（exit_code = 124）
            throw(124);
        }
        
        // 正常处理
        self.processTransfer(msg);
    }
}
```

**Bounce 处理：**

```
当 exit_code != 0 且消息 bounceable 时：

1. 创建 bounce 消息
   - 反转 src 和 dst
   - 设置 bounced = true
   - 携带未消费的余额

2. 发送 bounce 消息
   - 通知发送者交易失败
   - 返还余额

3. 接收者处理 bounce
   - 可以在 bounced() 处理器中处理
   - 用于恢复状态或重试
```

**调试技巧：**

```
调试 TVM 执行：

1. 使用 TVM 模拟器
   - 在本地模拟执行
   - 查看每步状态

2. 检查日志
   - 查看 exit_code
   - 查看 Gas 消耗
   - 查看栈状态

3. 使用调试器
   - 设置断点
   - 单步执行
   - 查看变量

4. 单元测试
   - 测试各种场景
   - 验证 exit_code
   - 检查状态变化
```

---

**本章小结：**

本章深入讲解了 TON 虚拟机（TVM）：
- **4.1 节**：学习了 TVM 的基于栈的执行模型、数据类型、控制寄存器和 Gas 计数器
- **4.2 节**：掌握了 TVM 指令集的主要类别，包括栈操作、算术逻辑、Cell 操作、控制流和字典操作
- **4.3 节**：了解了智能合约的执行流程，包括触发方式、计算阶段、动作阶段和错误处理

理解 TVM 是深入理解 TON 智能合约执行机制的关键，对于优化合约性能和调试问题至关重要。

---
