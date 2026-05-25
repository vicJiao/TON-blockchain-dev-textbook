# TON 区块链开发教材 - 工具篇

> 本文档包含 TON 区块链开发的工具链和环境配置，帮助开发者快速搭建完整的开发环境。

---

## 第三篇：工具篇 —— 开发工具与框架

---

## 第 9 章：开发环境搭建

本章介绍 TON 区块链开发所需的工具链和环境配置，帮助开发者快速搭建完整的开发环境。

### 9.1 开发工具概述

TON 生态系统提供了丰富的开发工具，涵盖合约编译、测试、部署和交互等各个环节。

**核心工具链：**

| 工具 | 用途 | 推荐程度 |
|------|------|----------|
| **BluePrint** | 合约开发框架 | ⭐⭐⭐⭐⭐ |
| **Tact Compiler** | Tact 语言编译器 | ⭐⭐⭐⭐⭐ |
| **Tolk Compiler** | Tolk 语言编译器 | ⭐⭐⭐⭐⭐ |
| **Sandbox** | 本地测试环境 | ⭐⭐⭐⭐⭐ |
| **TON Connect** | 钱包连接 SDK | ⭐⭐⭐⭐⭐ |
| **Toncenter API** | 区块链数据访问 | ⭐⭐⭐⭐ |
| **TON Scan** | 区块链浏览器 | ⭐⭐⭐⭐ |

**开发流程：**

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   编写合约   │ → │   本地测试   │ → │  部署到测试网 │
│  (Tact/Tolk) │    │  (Sandbox)  │    │  (Testnet)  │
└─────────────┘    └─────────────┘    └─────────────┘
                                              ↓
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   监控维护   │ ← │  部署到主网  │ ← │   最终测试   │
│             │    │  (Mainnet)  │    │             │
└─────────────┘    └─────────────┘    └─────────────┘
```

---

### 9.2 Node.js 环境配置

TON 开发工具主要基于 Node.js 生态，因此首先需要配置 Node.js 环境。

#### 9.2.1 安装 Node.js

**Windows/macOS：**

1. 访问 [Node.js 官网](https://nodejs.org/)
2. 下载 LTS 版本（推荐 v18 或更高）
3. 运行安装程序，按提示完成安装

**Linux (Ubuntu/Debian)：**

```bash
# 使用 NodeSource 安装
 curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
 sudo apt-get install -y nodejs

# 验证安装
 node --version  # 应显示 v18.x.x 或更高
 npm --version   # 应显示 9.x.x 或更高
```

**使用 nvm 管理 Node.js 版本（推荐）：**

```bash
# 安装 nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# 安装 Node.js 18
nvm install 18
nvm use 18
nvm alias default 18

# 验证
node --version
```

#### 9.2.2 配置 npm 镜像（可选）

国内用户可配置淘宝镜像加速下载：

```bash
# 设置淘宝镜像
npm config set registry https://registry.npmmirror.com

# 验证
npm config get registry

# 恢复官方镜像
npm config set registry https://registry.npmjs.org/
```

#### 9.2.3 安装全局工具

```bash
# 安装 TypeScript 编译器
npm install -g typescript

# 安装 ts-node（直接运行 TypeScript）
npm install -g ts-node

# 验证安装
tsc --version
ts-node --version
```

---

### 9.3 BluePrint 框架安装

BluePrint 是 TON 官方推荐的合约开发框架，提供项目模板、编译、测试和部署的一站式解决方案。

#### 9.3.1 创建新项目

```bash
# 使用 npx 创建项目（无需全局安装）
npx create-ton@latest my-ton-project

# 进入项目目录
cd my-ton-project

# 安装依赖
npm install
```

**创建项目时的选项：**

```
? Project name: my-ton-project
? Choose a template: (Use arrow keys)
❯ Tact language (recommended)  
  Tolk language  
  FunC language  
? Initialize Git repository? (Y/n) Y
```

#### 9.3.2 项目结构

```
my-ton-project/
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

#### 9.3.3 常用命令

```bash
# 编译合约
npx blueprint build

# 运行测试
npx blueprint test

# 部署到测试网
npx blueprint run --testnet --mainnet

# 部署到主网
npx blueprint run --mainnet

# 创建新合约
npx blueprint create ContractName
```

---

### 9.4 VS Code 配置

Visual Studio Code 是 TON 开发的首选 IDE，配合合适的插件可以大幅提升开发效率。

#### 9.4.1 推荐插件

| 插件名 | 用途 | 安装命令 |
|--------|------|----------|
| **Tact** | Tact 语言支持 | 搜索 "Tact" |
| **Tolk** | Tolk 语言支持 | 搜索 "Tolk" |
| **ESLint** | JavaScript/TypeScript 代码检查 | 搜索 "ESLint" |
| **Prettier** | 代码格式化 | 搜索 "Prettier" |
| **GitLens** | Git 增强 | 搜索 "GitLens" |

#### 9.4.2 VS Code 设置

创建 `.vscode/settings.json`：

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

#### 9.4.3 调试配置

创建 `.vscode/launch.json`：

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

---

### 9.5 测试网配置

在部署到主网之前，必须在测试网进行充分测试。

#### 9.5.1 获取测试网 TON

**方式一：官方水龙头（Bot）**

1. 打开 Telegram，搜索 `@testgiver_ton_bot`
2. 发送 `/start` 命令
3. 按照提示完成验证
4. 提供你的测试网钱包地址
5. 每次可获得 2 个测试网 TON

**方式二：Web 水龙头**

访问 [https://t.me/testgiver_ton_bot](https://t.me/testgiver_ton_bot) 或通过网页版申请。

#### 9.5.2 配置测试网钱包

**使用 Tonkeeper（推荐）：**

1. 下载 Tonkeeper 应用（iOS/Android/Chrome 插件）
2. 创建新钱包或导入现有钱包
3. 进入设置 → 网络 → 切换到 Testnet
4. 复制测试网地址，从水龙头获取测试币

**使用 Tonhub：**

1. 下载 Tonhub 应用
2. 创建或导入钱包
3. 在设置中启用 Testnet 模式

#### 9.5.3 测试网区块链浏览器

| 浏览器 | 网址 | 用途 |
|--------|------|------|
| **TON Scan Testnet** | https://testnet.tonscan.org | 查看交易、合约 |
| **TON Viewer** | https://testnet.tonviewer.com | 替代浏览器 |
| **Dton** | https://dton.io | 高级查询 |

---

### 9.6 主网准备

当合约在测试网验证通过后，可以准备部署到主网。

#### 9.6.1 主网钱包设置

**安全建议：**

1. **使用硬件钱包**：Ledger 或 Tonkeeper 的硬件支持
2. **多重签名**：重要合约使用多签钱包管理
3. **小额测试**：首次部署先用小额资金测试
4. **备份助记词**：安全存储钱包助记词

**推荐钱包：**

| 钱包 | 类型 | 特点 |
|------|------|------|
| **Tonkeeper** | 软件 | 用户友好，支持 dApp |
| **Tonhub** | 软件 | 开源，安全性高 |
| **Ledger** | 硬件 | 最高安全性 |
| **MyTonWallet** | 网页 | 功能丰富 |

#### 9.6.2 获取主网 TON

**交易所购买：**

- Binance、OKX、Bybit 等主要交易所都支持 TON
- 购买后提现到你的 TON 钱包地址

**注意：** 部署合约需要支付 Gas 费用，建议准备至少 0.5-1 TON。

#### 9.6.3 主网区块链浏览器

| 浏览器 | 网址 | 特点 |
|--------|------|------|
| **TON Scan** | https://tonscan.org | 最常用 |
| **TON Viewer** | https://tonviewer.com | 界面简洁 |
| **Dton** | https://dton.io | API 支持 |
| **Tonalytica** | https://tonalytica.redoubt.online | 数据分析 |

---

### 9.7 其他开发工具

#### 9.7.1 TON API 服务

**Toncenter（免费）：**

```typescript
import { TonClient } from '@ton/ton';

const client = new TonClient({
  endpoint: 'https://testnet.toncenter.com/api/v2/jsonRPC',
  apiKey: 'your-api-key'  // 从 toncenter.com 获取
});

// 查询账户余额
const balance = await client.getBalance(address);
```

**GetBlock（付费）：**

```typescript
const client = new TonClient({
  endpoint: 'https://go.getblock.io/YOUR_API_KEY'
});
```

#### 9.7.2 本地节点（高级）

对于需要完整区块链数据的开发场景，可以运行本地 TON 节点：

```bash
# 使用 Docker 运行本地节点
docker run -d --name ton-node \
  -v ton-db:/var/ton-work/db \
  -p 43679:43679 \
  tonlabs/local-node:latest
```

**注意：** 运行全节点需要大量存储空间（>100GB）和带宽。

#### 9.7.3 合约验证工具

**验证合约源码：**

1. 访问 [TON Scan](https://tonscan.org)
2. 找到你的合约地址
3. 点击 "Source" 标签
4. 上传源码并验证

**验证的好处：**
- 增加合约透明度
- 便于第三方审计
- 提升用户信任

---

### 9.8 开发工作流示例

完整的开发流程示例：

```bash
# 1. 创建项目
npx create-ton@latest my-token
cd my-token
npm install

# 2. 编写合约（编辑 contracts/main.tact）
# ...

# 3. 编译合约
npx blueprint build

# 4. 编写测试（编辑 tests/main.spec.ts）
# ...

# 5. 运行测试
npx blueprint test

# 6. 配置部署脚本（编辑 scripts/deploy.ts）
# ...

# 7. 部署到测试网
npx blueprint run --testnet

# 8. 在测试网验证
# - 检查合约状态
# - 测试所有功能
# - 检查 Gas 消耗

# 9. 部署到主网
npx blueprint run --mainnet

# 10. 验证合约源码
# - 在 TON Scan 上传源码
# - 确认验证通过
```

---

### 9.9 常见问题排查

#### 9.9.1 编译错误

**问题：** `Error: Cannot find module '@tact-lang/compiler'`

**解决：**
```bash
npm install
# 或
npm install @tact-lang/compiler
```

**问题：** `Type error in contract`

**解决：**
- 检查 Tact 语法是否正确
- 确保所有变量都已声明类型
- 检查导入路径是否正确

#### 9.9.2 测试失败

**问题：** `Contract not deployed`

**解决：**
```typescript
// 确保在测试前部署合约
beforeEach(async () => {
  blockchain = await Blockchain.create();
  contract = blockchain.openContract(await MyContract.fromInit());
  
  // 部署合约
  await contract.send(
    deployer.getSender(),
    { value: toNano('0.05') },
    { $$type: 'Deploy' }
  );
});
```

**问题：** `Gas limit exceeded`

**解决：**
- 优化合约代码，减少 Gas 消耗
- 在测试中使用更大的 Gas 限制
- 检查是否有无限循环

#### 9.9.3 部署失败

**问题：** `Not enough balance`

**解决：**
- 确保钱包有足够余额
- 测试网：从水龙头获取更多测试币
- 主网：从交易所购买 TON

**问题：** `Contract code hash mismatch`

**解决：**
- 重新编译合约
- 确保部署的是最新编译的代码
- 检查初始化参数是否正确

---

### 9.10 学习资源

**官方文档：**

| 资源 | 链接 | 说明 |
|------|------|------|
| TON 文档 | https://docs.ton.org | 官方文档 |
| Tact 文档 | https://docs.tact-lang.org | Tact 语言文档 |
| TON Academy | https://ton.academy | 教程和课程 |

**社区资源：**

| 资源 | 链接 | 说明 |
|------|------|------|
| TON Dev Chat | https://t.me/tondev_eng | 开发者群组 |
| TON Dev 中文 | https://t.me/tondev_zh | 中文开发者群 |
| GitHub | https://github.com/ton-org | 官方代码库 |

**推荐教程：**

1. [TON Hello World](https://helloworld.tonstudio.io/) - 入门教程
2. [Tact Cookbook](https://docs.tact-lang.org/cookbook) - 代码示例
3. [TON Documentation](https://docs.ton.org/develop/smart-contracts/) - 官方文档

---

**本章小结：**

本章介绍了 TON 区块链开发环境的搭建：
- Node.js 环境配置和 npm 设置
- BluePrint 框架的安装和使用
- VS Code 编辑器的配置
- 测试网和主网的配置方法
- 其他开发工具（API、浏览器等）
- 完整的开发工作流示例
- 常见问题排查方法

搭建好开发环境后，下一章将介绍合约的编译、测试和调试技巧。

---

## 第 10 章：合约编译与测试

本章深入介绍 TON 智能合约的编译过程、测试方法和调试技巧，帮助开发者编写高质量的合约代码。

### 10.1 合约编译流程

#### 10.1.1 编译原理

TON 合约编译将高级语言（Tact/Tolk）转换为 TVM 字节码：

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  源代码文件  │ → │   语法分析   │ → │   代码生成   │ → │  TVM 字节码  │
│ (.tact/.tolk)│    │   (Parser)   │    │  (Compiler)  │    │  (.boc/.cell)│
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
                                                              ↓
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  部署到链上  │ ← │  生成地址   │ ← │  计算 Code Hash │
│             │    │             │    │               │
└─────────────┘    └─────────────┘    └─────────────┘
```

**编译输出文件：**

| 文件 | 扩展名 | 说明 |
|------|--------|------|
| 合约代码 | `.cell` / `.boc` | TVM 字节码 |
| ABI 文件 | `.abi` / `.json` | 接口定义 |
| 包装器 | `.ts` | TypeScript 接口 |

#### 10.1.2 Tact 编译

**命令行编译：**

```bash
# 使用 blueprint 编译
npx blueprint build

# 编译特定合约
npx blueprint build ContractName

# 强制重新编译
npx blueprint build --force
```

**编译配置（`tact.config.json`）：**

```json
{
  "projects": [
    {
      "name": "my-contract",
      "path": "./contracts/main.tact",
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

**编译选项说明：**

| 选项 | 类型 | 说明 |
|------|------|------|
| `debug` | boolean | 启用调试信息 |
| `masterchain` | boolean | 支持主链部署 |
| `external` | boolean | 支持外部消息 |
| `ipfsAbiGetter` | boolean | 生成 IPFS ABI getter |

#### 10.1.3 编译输出分析

编译成功后会生成以下文件：

```
build/
├── MyContract.compiled.json    # 编译后的代码（Base64）
├── MyContract.code.boc         # 字节码文件
├── MyContract.abi              # ABI 接口定义
├── MyContract.ts               # TypeScript 包装器
└── MyContract.md               # 生成的文档
```

**查看编译后的代码大小：**

```typescript
import { readFileSync } from 'fs';

const compiled = JSON.parse(
  readFileSync('./build/MyContract.compiled.json', 'utf-8')
);

const codeSize = Buffer.from(compiled.hex, 'hex').length;
console.log(`合约代码大小: ${codeSize} bytes`);
console.log(`预估部署费用: ${(codeSize * 0.001).toFixed(4)} TON`);
```

---

### 10.2 测试框架介绍

#### 10.2.1 Sandbox 测试环境

Sandbox 是 TON 官方提供的本地测试框架，模拟区块链环境进行合约测试。

**核心特性：**

- ✅ 本地运行，无需连接真实网络
- ✅ 快速执行，秒级反馈
- ✅ 完整状态检查
- ✅ 消息流追踪
- ✅ Gas 消耗统计

**安装：**

```bash
npm install --save-dev @ton-community/sandbox @ton-community/test-utils
```

#### 10.2.2 测试结构

```typescript
import { Blockchain } from '@ton-community/sandbox';
import { MyContract } from '../build/MyContract';
import { toNano, beginCell } from '@ton/core';

describe('MyContract', () => {
  let blockchain: Blockchain;
  let contract: MyContract;
  let deployer: any;
  let user: any;

  // 每个测试前执行
  beforeEach(async () => {
    // 创建区块链实例
    blockchain = await Blockchain.create();
    
    // 创建测试账户
    deployer = await blockchain.treasury('deployer');
    user = await blockchain.treasury('user');
    
    // 部署合约
    contract = blockchain.openContract(
      await MyContract.fromInit(deployer.address)
    );
    
    await contract.send(
      deployer.getSender(),
      { value: toNano('0.05') },
      { $$type: 'Deploy' }
    );
  });

  // 测试用例
  it('should deploy successfully', async () => {
    // 测试逻辑
  });
});
```

#### 10.2.3 常用断言

```typescript
import { expect } from '@ton-community/test-utils';

// 检查交易是否成功
expect(result.transactions).toHaveTransaction({
  from: deployer.address,
  to: contract.address,
  success: true
});

// 检查特定 exit code
expect(result.transactions).toHaveTransaction({
  exitCode: 0  // 成功
});

// 检查金额转移
expect(result.transactions).toHaveTransaction({
  value: toNano('1.0')
});

// 检查合约状态
const data = await contract.getData();
expect(data.counter).toBe(1n);
```

---

### 10.3 编写单元测试

#### 10.3.1 基础功能测试

```typescript
describe('Counter Contract', () => {
  it('should increment counter', async () => {
    // 初始状态
    const initial = await contract.getCounter();
    expect(initial).toBe(0n);

    // 发送增加消息
    await contract.send(
      user.getSender(),
      { value: toNano('0.01') },
      { $$type: 'Increment' }
    );

    // 验证结果
    const updated = await contract.getCounter();
    expect(updated).toBe(1n);
  });

  it('should handle multiple increments', async () => {
    for (let i = 0; i < 5; i++) {
      await contract.send(
        user.getSender(),
        { value: toNano('0.01') },
        { $$type: 'Increment' }
      );
    }

    const counter = await contract.getCounter();
    expect(counter).toBe(5n);
  });
});
```

#### 10.3.2 错误处理测试

```typescript
describe('Error Handling', () => {
  it('should reject unauthorized access', async () => {
    const result = await contract.send(
      user.getSender(),  // 非所有者
      { value: toNano('0.01') },
      { $$type: 'AdminAction' }
    );

    expect(result.transactions).toHaveTransaction({
      exitCode: 102,  // ERROR_UNAUTHORIZED
      success: false
    });
  });

  it('should reject insufficient value', async () => {
    const result = await contract.send(
      user.getSender(),
      { value: toNano('0.001') },  // 金额不足
      { $$type: 'BuyItem' }
    );

    expect(result.transactions).toHaveTransaction({
      exitCode: 103,  // ERROR_INSUFFICIENT_VALUE
      success: false
    });
  });

  it('should handle invalid input', async () => {
    const result = await contract.send(
      user.getSender(),
      { value: toNano('0.01') },
      { $$type: 'SetValue', value: -1 }  // 无效值
    );

    expect(result.transactions).toHaveTransaction({
      success: false
    });
  });
});
```

#### 10.3.3 状态验证测试

```typescript
describe('State Verification', () => {
  it('should update balance correctly', async () => {
    // 初始余额
    const initialBalance = await contract.getBalance();
    
    // 存款
    await contract.send(
      user.getSender(),
      { value: toNano('1.0') },
      { $$type: 'Deposit' }
    );

    // 验证余额
    const newBalance = await contract.getBalance();
    expect(newBalance).toBe(initialBalance + toNano('1.0'));
  });

  it('should track ownership transfer', async () => {
    const originalOwner = await contract.getOwner();
    expect(originalOwner.equals(deployer.address)).toBe(true);

    // 转移所有权
    await contract.send(
      deployer.getSender(),
      { value: toNano('0.01') },
      { $$type: 'TransferOwnership', newOwner: user.address }
    );

    const newOwner = await contract.getOwner();
    expect(newOwner.equals(user.address)).toBe(true);
  });
});
```

---

### 10.4 集成测试

#### 10.4.1 多合约交互测试

```typescript
describe('Multi-Contract Integration', () => {
  let tokenContract: TokenContract;
  let vaultContract: VaultContract;

  beforeEach(async () => {
    // 部署代币合约
    tokenContract = blockchain.openContract(
      await TokenContract.fromInit(deployer.address, 1000000n)
    );
    
    // 部署金库合约
    vaultContract = blockchain.openContract(
      await VaultContract.fromInit(tokenContract.address)
    );

    await tokenContract.send(deployer.getSender(), ...);
    await vaultContract.send(deployer.getSender(), ...);
  });

  it('should handle deposit and withdraw flow', async () => {
    // 用户存入代币到金库
    await tokenContract.send(
      user.getSender(),
      { value: toNano('0.05') },
      { 
        $$type: 'Transfer',
        to: vaultContract.address,
        amount: 1000n
      }
    );

    // 验证金库余额
    const vaultBalance = await tokenContract.getBalanceOf(vaultContract.address);
    expect(vaultBalance).toBe(1000n);

    // 用户从金库提取
    await vaultContract.send(
      user.getSender(),
      { value: toNano('0.05') },
      { $$type: 'Withdraw', amount: 500n }
    );

    // 验证最终余额
    const finalVaultBalance = await tokenContract.getBalanceOf(vaultContract.address);
    expect(finalVaultBalance).toBe(500n);
  });
});
```

#### 10.4.2 消息流追踪

```typescript
it('should trace message flow', async () => {
  const result = await contract.send(
    user.getSender(),
    { value: toNano('0.1') },
    { $$type: 'ComplexOperation' }
  );

  // 打印所有交易
  console.log('Transactions:', result.transactions.length);
  
  for (const tx of result.transactions) {
    console.log(`From: ${tx.inMessage?.info.src}`);
    console.log(`To: ${tx.inMessage?.info.dest}`);
    console.log(`Value: ${tx.inMessage?.info.value.coins}`);
    console.log(`Exit Code: ${tx.exitCode}`);
    console.log('---');
  }

  // 验证消息链
  expect(result.transactions.length).toBeGreaterThan(1);
});
```

---

### 10.5 Gas 消耗分析

#### 10.5.1 测量 Gas 消耗

```typescript
describe('Gas Analysis', () => {
  it('should measure gas consumption', async () => {
    const result = await contract.send(
      user.getSender(),
      { value: toNano('0.1') },
      { $$type: 'ComplexOperation' }
    );

    // 获取合约执行的交易
    const contractTx = result.transactions.find(
      tx => tx.inMessage?.info.dest?.equals(contract.address)
    );

    if (contractTx) {
      const gasUsed = contractTx.totalFees;
      console.log(`Gas used: ${gasUsed} nanoTON`);
      console.log(`Gas used: ${Number(gasUsed) / 1e9} TON`);
      
      // 设置 Gas 上限
      expect(Number(gasUsed)).toBeLessThan(0.01 * 1e9); // < 0.01 TON
    }
  });

  it('should compare different operations', async () => {
    const operations = [
      { type: 'SimpleTransfer', gas: 0n },
      { type: 'ComplexCalculation', gas: 0n },
      { type: 'StorageUpdate', gas: 0n }
    ];

    for (const op of operations) {
      const result = await contract.send(
        user.getSender(),
        { value: toNano('0.1') },
        { $$type: op.type }
      );

      const tx = result.transactions.find(
        tx => tx.inMessage?.info.dest?.equals(contract.address)
      );
      
      op.gas = tx?.totalFees || 0n;
      console.log(`${op.type}: ${Number(op.gas) / 1e9} TON`);
    }
  });
});
```

#### 10.5.2 优化 Gas 消耗

```typescript
// ❌ 低效：每次计算
contract Inefficient {
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
contract Efficient {
  items: map<Int, Item>;
  totalValue: Int = 0;  // 缓存总值
  
  fun addItem(id: Int, item: Item) {
    self.items.set(id, item);
    self.totalValue = self.totalValue + item.value;  // 更新缓存
  }
  
  fun getTotalValue(): Int {
    return self.totalValue;  // O(1) 直接返回
  }
}
```

---

### 10.6 调试技巧

#### 10.6.1 日志输出

```tact
// 在合约中使用日志
contract Debuggable {
  receive(msg: DebugMessage) {
    // 使用 comment 消息输出日志
    send(SendParameters{
      to: sender(),
      value: 0,
      mode: SendIgnoreErrors,
      body: "Debug: Processing message".asComment()
    });
    
    // 继续处理...
  }
}
```

#### 10.6.2 状态检查

```typescript
// 在测试中检查合约状态
it('should debug contract state', async () => {
  // 获取合约数据
  const data = await contract.getData();
  console.log('Contract data:', data);

  // 获取合约余额
  const balance = await contract.getBalance();
  console.log('Contract balance:', balance);

  // 获取合约状态
  const state = await blockchain.getContract(contract.address);
  console.log('Account state:', state.accountState);
  console.log('Storage used:', state.storageStats);
});
```

#### 10.6.3 断点调试

```typescript
// 使用 debugger 语句
it('should debug step by step', async () => {
  debugger;  // 在这里设置断点
  
  const result = await contract.send(
    user.getSender(),
    { value: toNano('0.01') },
    { $$type: 'Test' }
  );
  
  debugger;  // 检查 result
  
  expect(result.transactions).toHaveTransaction({
    success: true
  });
});
```

---

### 10.7 测试覆盖率

#### 10.7.1 配置覆盖率报告

```javascript
// jest.config.js
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  collectCoverage: true,
  coverageDirectory: 'coverage',
  coverageReporters: ['text', 'lcov', 'html'],
  collectCoverageFrom: [
    'contracts/**/*.tact',
    'wrappers/**/*.ts'
  ]
};
```

#### 10.7.2 运行覆盖率测试

```bash
# 运行测试并生成覆盖率报告
npx jest --coverage

# 查看 HTML 报告
open coverage/lcov-report/index.html
```

---

### 10.8 持续集成

#### 10.8.1 GitHub Actions 配置

```yaml
# .github/workflows/test.yml
name: Contract Tests

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Compile contracts
      run: npx blueprint build
    
    - name: Run tests
      run: npx blueprint test
    
    - name: Generate coverage report
      run: npx jest --coverage
    
    - name: Upload coverage
      uses: codecov/codecov-action@v3
```

---

**本章小结：**

本章介绍了 TON 智能合约的编译与测试：
- 合约编译流程和编译选项
- Sandbox 测试框架的使用
- 单元测试和集成测试的编写方法
- Gas 消耗分析和优化技巧
- 调试技巧和状态检查
- 测试覆盖率和持续集成配置

掌握这些测试技能可以确保合约的正确性和安全性，是智能合约开发的重要环节。

---

## 第 11 章：部署与交互

本章介绍 TON 智能合约的部署流程、与合约交互的方法以及生产环境的注意事项。

### 11.1 合约部署原理

#### 11.1.1 部署流程

TON 合约部署是一个多步骤过程：

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  准备代码    │ → │  计算地址   │ → │  发送部署消息 │
│  和初始数据  │    │             │    │             │
└─────────────┘    └─────────────┘    └─────────────┘
                                              ↓
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  合约激活   │ ← │  验证部署   │ ← │  网络处理   │
│  可交互     │    │             │    │             │
└─────────────┘    └─────────────┘    └─────────────┘
```

**部署要素：**

| 要素 | 说明 | 来源 |
|------|------|------|
| **Code** | 合约字节码 | 编译输出 |
| **Data** | 初始存储数据 | 构造函数参数 |
| **Balance** | 初始余额 | 部署者发送 |

#### 11.1.2 地址计算

TON 合约地址由 Code 和初始 Data 决定：

```
地址 = hash(code_cell, data_cell, workchain_id)
```

**地址结构：**

```
<workchain_id>:<account_id>

示例：
EQD...xyz (Base64 编码)
  ↓
workchain: 0 (基础链)
account_id: hash 值
```

**计算地址（代码示例）：**

```typescript
import { Address, Cell, contractAddress } from '@ton/core';

// 加载编译后的代码
const code = Cell.fromBoc(compiledCode)[0];

// 准备初始数据
const data = beginCell()
  .storeAddress(ownerAddress)
  .storeUint(0, 64)  // 初始计数器
  .endCell();

// 计算地址
const address = contractAddress(0, { code, data });
console.log('Contract address:', address.toString());
```

---

### 11.2 部署脚本编写

#### 11.2.1 基础部署脚本

```typescript
// scripts/deploy.ts
import { toNano } from '@ton/core';
import { MyContract } from '../build/MyContract';
import { NetworkProvider } from '@ton/blueprint';

export async function run(provider: NetworkProvider) {
  // 创建合约实例
  const myContract = provider.open(
    await MyContract.fromInit(
      provider.sender().address!,  // 所有者地址
      0n                           // 初始计数器值
    )
  );

  // 发送部署消息
  await myContract.send(
    provider.sender(),
    { value: toNano('0.05') },
    { $$type: 'Deploy' }
  );

  // 等待部署确认
  await provider.waitForDeploy(myContract.address);

  console.log('Contract deployed at:', myContract.address.toString());
}
```

#### 11.2.2 高级部署选项

```typescript
// 带自定义配置的部署
export async function run(provider: NetworkProvider) {
  const myContract = provider.open(
    await MyContract.fromInit(
      provider.sender().address!,
      100n,  // 初始值
      true   // 启用某些功能
    )
  );

  // 发送更多初始资金
  await myContract.send(
    provider.sender(),
    {
      value: toNano('0.5'),  // 更多初始余额
      bounce: false          // 不接收 bounce 消息
    },
    {
      $$type: 'Deploy',
      queryId: 0n
    }
  );

  await provider.waitForDeploy(myContract.address);

  // 部署后初始化
  await myContract.send(
    provider.sender(),
    { value: toNano('0.01') },
    { $$type: 'Initialize', params: {...} }
  );
}
```

#### 11.2.3 多合约部署

```typescript
// 部署相互依赖的多个合约
export async function run(provider: NetworkProvider) {
  // 1. 先部署主合约
  const mainContract = provider.open(
    await MainContract.fromInit(provider.sender().address!)
  );

  await mainContract.send(
    provider.sender(),
    { value: toNano('0.1') },
    { $$type: 'Deploy' }
  );

  await provider.waitForDeploy(mainContract.address);
  console.log('Main contract:', mainContract.address.toString());

  // 2. 部署依赖合约（传入主合约地址）
  const childContract = provider.open(
    await ChildContract.fromInit(mainContract.address)
  );

  await childContract.send(
    provider.sender(),
    { value: toNano('0.1') },
    { $$type: 'Deploy' }
  );

  await provider.waitForDeploy(childContract.address);
  console.log('Child contract:', childContract.address.toString());

  // 3. 在主合约中注册子合约
  await mainContract.send(
    provider.sender(),
    { value: toNano('0.01') },
    {
      $$type: 'RegisterChild',
      childAddress: childContract.address
    }
  );
}
```

---

### 11.3 网络部署

#### 11.3.1 测试网部署

```bash
# 部署到测试网
npx blueprint run --testnet

# 指定特定脚本
npx blueprint run deploy.ts --testnet
```

**测试网配置（`blueprint.config.ts`）：**

```typescript
import { Config } from '@ton/blueprint';

export const config: Config = {
  network: {
    testnet: {
      endpoint: 'https://testnet.toncenter.com/api/v2/jsonRPC',
      apiKey: process.env.TESTNET_API_KEY,
    },
    mainnet: {
      endpoint: 'https://toncenter.com/api/v2/jsonRPC',
      apiKey: process.env.MAINNET_API_KEY,
    },
  },
};
```

#### 11.3.2 主网部署

```bash
# 部署到主网（谨慎操作！）
npx blueprint run --mainnet
```

**主网部署检查清单：**

```
□ 合约已在测试网充分测试
□ 所有测试用例通过
□ 代码已审计（如需要）
□ 部署钱包有足够余额（>0.5 TON）
□ 助记词已安全备份
□ 部署脚本已验证
□ 初始化参数正确
□ 紧急停止机制已配置（如需要）
```

#### 11.3.3 部署验证

部署后在区块链浏览器验证：

```typescript
// 验证部署
export async function verifyDeployment(provider: NetworkProvider, address: Address) {
  const contract = provider.open(MyContract.fromAddress(address));

  // 检查合约是否存在
  const state = await contract.getState();
  console.log('Contract state:', state);

  // 验证初始数据
  const owner = await contract.getOwner();
  console.log('Owner:', owner.toString());

  const counter = await contract.getCounter();
  console.log('Initial counter:', counter);

  // 检查余额
  const balance = await contract.getBalance();
  console.log('Balance:', balance);
}
```

---

### 11.4 合约交互

#### 11.4.1 读取合约状态

```typescript
// 查询合约数据
async function readContractState(contract: MyContract) {
  // 获取所有者
  const owner = await contract.getOwner();
  console.log('Owner:', owner.toString());

  // 获取计数器值
  const counter = await contract.getCounter();
  console.log('Counter:', counter);

  // 获取用户余额
  const userBalance = await contract.getBalanceOf(userAddress);
  console.log('User balance:', userBalance);

  // 获取合约配置
  const config = await contract.getConfig();
  console.log('Config:', config);
}
```

#### 11.4.2 发送消息

```typescript
// 发送简单消息
await contract.send(
  sender,
  { value: toNano('0.01') },
  { $$type: 'Increment' }
);

// 发送带参数的消息
await contract.send(
  sender,
  { value: toNano('0.05') },
  {
    $$type: 'Transfer',
    to: recipientAddress,
    amount: 1000n
  }
);

// 发送复杂消息
await contract.send(
  sender,
  { value: toNano('0.1') },
  {
    $$type: 'ComplexOperation',
    param1: 123n,
    param2: true,
    param3: beginCell()
      .storeAddress(someAddress)
      .storeUint(456, 64)
      .endCell()
  }
);
```

#### 11.4.3 监听事件

```typescript
// 监听合约事件
import { TonClient } from '@ton/ton';

const client = new TonClient({
  endpoint: 'https://toncenter.com/api/v2/jsonRPC'
});

// 获取合约交易历史
async function getTransactionHistory(address: Address) {
  const transactions = await client.getTransactions(address, {
    limit: 10
  });

  for (const tx of transactions) {
    console.log('Transaction:', {
      hash: tx.hash,
      time: new Date(tx.time * 1000),
      value: tx.inMessage?.value,
      from: tx.inMessage?.src
    });
  }
}

// 轮询检查新事件
async function pollEvents(contract: MyContract, interval: number = 5000) {
  let lastCounter = await contract.getCounter();

  setInterval(async () => {
    const currentCounter = await contract.getCounter();

    if (currentCounter > lastCounter) {
      console.log('Counter increased:', currentCounter);
      lastCounter = currentCounter;
      // 处理事件...
    }
  }, interval);
}
```

---

### 11.5 TON Connect 集成

#### 11.5.1 前端集成

```typescript
// 使用 TON Connect 连接钱包
import { TonConnect } from '@tonconnect/sdk';

const tonConnect = new TonConnect({
  manifestUrl: 'https://your-app.com/tonconnect-manifest.json'
});

// 连接钱包
async function connectWallet() {
  const wallets = await tonConnect.getWallets();

  // 显示钱包选择界面
  const wallet = await showWalletSelector(wallets);

  await tonConnect.connect({
    universalLink: wallet.universalLink,
    bridgeUrl: wallet.bridgeUrl
  });
}

// 发送交易
async function sendTransaction() {
  const transaction = {
    validUntil: Math.floor(Date.now() / 1000) + 60,
    messages: [
      {
        address: contractAddress.toString(),
        amount: toNano('0.05').toString(),
        payload: beginCell()
          .storeUint(1, 32)  // op code
          .storeUint(0, 64)  // query id
          .endCell()
          .toBoc()
          .toString('base64')
      }
    ]
  };

  const result = await tonConnect.sendTransaction(transaction);
  console.log('Transaction sent:', result);
}
```

#### 11.5.2 React 集成示例

```tsx
// components/TonConnectButton.tsx
import { TonConnectButton } from '@tonconnect/ui-react';

export function App() {
  return (
    <TonConnectUIProvider manifestUrl="https://your-app.com/tonconnect-manifest.json">
      <div>
        <TonConnectButton />
        <ContractInteraction />
      </div>
    </TonConnectUIProvider>
  );
}

// components/ContractInteraction.tsx
import { useTonConnectUI } from '@tonconnect/ui-react';
import { Address, toNano } from '@ton/ton';

export function ContractInteraction() {
  const [tonConnectUI] = useTonConnectUI();

  const handleIncrement = async () => {
    await tonConnectUI.sendTransaction({
      messages: [
        {
          address: contractAddress,
          amount: toNano('0.01').toString(),
          payload: 'te6cckEBAQEAAgAAAEysuc0='
        }
      ],
      validUntil: Date.now() + 5 * 60 * 1000
    });
  };

  return <button onClick={handleIncrement}>Increment</button>;
}
```

---

### 11.6 生产环境注意事项

#### 11.6.1 安全管理

```
□ 使用硬件钱包管理生产环境密钥
□ 多签钱包管理重要合约
□ 密钥分散存储
□ 定期轮换密钥
□ 限制合约管理员权限
```

#### 11.6.2 监控和报警

```typescript
// 合约监控系统
class ContractMonitor {
  constructor(private client: TonClient, private address: Address) {}

  async monitor() {
    // 检查合约余额
    const balance = await this.client.getBalance(this.address);
    if (balance < toNano('0.1')) {
      this.alert('Contract balance low!');
    }

    // 检查异常交易
    const transactions = await this.client.getTransactions(this.address, { limit: 10 });
    for (const tx of transactions) {
      if (tx.exitCode !== 0) {
        this.alert(`Transaction failed: ${tx.hash}`);
      }
    }
  }

  private alert(message: string) {
    // 发送报警通知
    console.error(`[ALERT] ${message}`);
    // 集成 Telegram/Email/Slack 通知
  }
}
```

#### 11.6.3 升级策略

```tact
// 可升级合约示例
contract UpgradableContract with Ownable {
  owner: Address;
  code: Cell;  // 存储新代码

  receive(msg: Upgrade) {
    self.requireOwner();

    // 设置新代码
    self.code = msg.newCode;

    // 发送升级消息给自己
    send(SendParameters{
      to: myAddress(),
      value: 0,
      mode: SendRemainingValue + SendDestroyIfZero,
      code: msg.newCode,  // 新代码
      data: msg.newData   // 新数据
    });
  }
}
```

---

### 11.7 故障排查

#### 11.7.1 部署失败

| 错误 | 原因 | 解决 |
|------|------|------|
| `exit_code: -14` | 余额不足 | 增加发送的 TON 金额 |
| `exit_code: 0` | 代码哈希不匹配 | 重新编译合约 |
| `Account not found` | 地址错误 | 检查地址计算 |
| `Action list invalid` | 动作列表错误 | 检查 send 参数 |

#### 11.7.2 交互失败

```typescript
// 调试交易
async function debugTransaction(hash: string) {
  const tx = await client.getTransaction(hash);

  console.log('Transaction details:');
  console.log('- Exit code:', tx.exitCode);
  console.log('- Gas used:', tx.gasUsed);
  console.log('- Storage fee:', tx.storageFee);

  if (tx.exitCode !== 0) {
    // 查找错误码含义
    const errorCodes = {
      1: 'Action list invalid',
      2: 'Stack underflow',
      3: 'Stack overflow',
      4: 'Integer overflow',
      5: 'Integer out of range',
      6: 'Invalid opcode',
      7: 'Type check error',
      8: 'Cell overflow',
      9: 'Cell underflow',
      10: 'Dictionary error'
    };

    console.log('Error:', errorCodes[tx.exitCode] || 'Unknown error');
  }
}
```

---

**本章小结：**

本章介绍了 TON 智能合约的部署与交互：
- 合约部署原理和地址计算
- 部署脚本的编写方法
- 测试网和主网部署流程
- 合约交互和状态读取
- TON Connect 钱包集成
- 生产环境的安全管理
- 监控报警和故障排查

掌握这些技能后，开发者可以独立完成合约的部署和运维工作。

---

