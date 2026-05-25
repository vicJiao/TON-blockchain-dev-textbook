# TON 区块链开发教材 - 应用篇

> 本文档包含 TON DApp 开发的核心内容，涵盖 SDK 使用、钱包连接、前端集成和 Telegram Mini Apps 开发。

---

## 第四篇：应用篇 —— DApp 开发

---

## 第 12 章：TON SDK 与前端集成

本章介绍如何使用 TON SDK 与区块链进行交互，包括核心库的使用、合约调用和数据查询。

### 12.1 @ton/core —— 核心数据结构

**@ton/core** 是 TON 开发的基础库，提供了 Cell、Slice、Builder、Address 等核心数据结构的实现。

#### 12.1.1 安装与导入

```bash
npm install @ton/core
```

```typescript
import {
  Cell,
  Slice,
  Builder,
  Address,
  beginCell,
  toNano,
  fromNano
} from '@ton/core';
```

#### 12.1.2 Cell 操作

**创建 Cell：**

```typescript
import { beginCell } from '@ton/core';

// 创建简单的 Cell
const cell = beginCell()
  .storeUint(42, 32)                    // 存储 32 位无符号整数
  .storeInt(-10, 32)                    // 存储 32 位有符号整数
  .storeBool(true)                      // 存储布尔值
  .storeCoins(toNano('1.5'))           // 存储金额（变长编码）
  .storeAddress(Address.parse('EQD...')) // 存储地址
  .storeRef(beginCell().storeUint(1, 8).endCell()) // 存储引用
  .endCell();

console.log('Cell hash:', cell.hash().toString('hex'));
console.log('Cell size:', cell.bits.length, 'bits');
```

**读取 Cell：**

```typescript
const slice = cell.beginParse();

const num = slice.loadUint(32);        // 读取 32 位无符号整数
const addr = slice.loadAddress();      // 读取地址
const coins = slice.loadCoins();       // 读取金额
const ref = slice.loadRef();           // 读取引用

// 检查剩余数据
console.log('Remaining bits:', slice.remainingBits);
console.log('Remaining refs:', slice.remainingRefs);
```

#### 12.1.3 Address 处理

**创建地址：**

```typescript
import { Address } from '@ton/core';

// 从字符串解析
const addr1 = Address.parse('EQD...');
const addr2 = Address.parseFriendly('EQD...');

// 从原始数据创建
const addr3 = new Address(0, Buffer.from('...', 'hex'));

// 零地址
const nullAddr = new Address(0, Buffer.alloc(32));
```

**地址转换：**

```typescript
const addr = Address.parse('EQD...');

// 转换为不同格式
console.log('User-friendly:', addr.toString());           // 用户友好格式
console.log('Raw:', addr.toRawString());                  // 原始格式
console.log('Buffer:', addr.toBuffer().toString('hex')); // Buffer 格式

// 获取工作链 ID
console.log('Workchain:', addr.workChain);
```

#### 12.1.4 金额处理

```typescript
import { toNano, fromNano } from '@ton/core';

// TON 转换为 nanoTON
const nanoAmount = toNano('1.5');      // 1500000000n
const nanoAmount2 = toNano(2.5);       // 2500000000n

// nanoTON 转换为 TON
const tonAmount = fromNano(1500000000n);  // "1.5"
const tonAmount2 = fromNano('1500000000'); // "1.5"

// 格式化显示
function formatTon(amount: bigint): string {
  return `${fromNano(amount)} TON`;
}
```

---

### 12.2 @ton/ton —— 区块链交互

**@ton/ton** 提供了与 TON 区块链进行交互的客户端实现。

#### 12.2.1 安装与配置

```bash
npm install @ton/ton
```

```typescript
import { TonClient } from '@ton/ton';

// 创建客户端（测试网）
const testnetClient = new TonClient({
  endpoint: 'https://testnet.toncenter.com/api/v2/jsonRPC',
  apiKey: 'your-api-key'  // 从 toncenter.com 获取
});

// 创建客户端（主网）
const mainnetClient = new TonClient({
  endpoint: 'https://toncenter.com/api/v2/jsonRPC',
  apiKey: 'your-api-key'
});
```

#### 12.2.2 查询账户信息

```typescript
// 查询账户余额
async function getBalance(address: Address) {
  const balance = await client.getBalance(address);
  console.log('Balance:', fromNano(balance), 'TON');
  return balance;
}

// 查询账户状态
async function getAccountState(address: Address) {
  const state = await client.getContractState(address);
  
  console.log('State:', state.state);  // 'active', 'uninit', 'frozen', 'nonexist'
  console.log('Balance:', fromNano(state.balance));
  console.log('Code hash:', state.codeHash?.toString('hex'));
  console.log('Data hash:', state.dataHash?.toString('hex'));
  
  return state;
}
```

#### 12.2.3 调用 Getter 函数

```typescript
// 创建合约包装器
class MyContract {
  constructor(readonly address: Address) {}
  
  async getCounter(provider: TonClient): Promise<bigint> {
    const result = await provider.runMethod(
      this.address,
      'getCounter',
      []
    );
    return result.stack.readBigNumber();
  }
  
  async getBalanceOf(provider: TonClient, owner: Address): Promise<bigint> {
    const result = await provider.runMethod(
      this.address,
      'getBalanceOf',
      [
        { type: 'slice', cell: beginCell().storeAddress(owner).endCell() }
      ]
    );
    return result.stack.readBigNumber();
  }
}

// 使用
const contract = new MyContract(Address.parse('EQD...'));
const counter = await contract.getCounter(client);
const balance = await contract.getBalanceOf(client, userAddress);
```

#### 12.2.4 获取交易历史

```typescript
async function getTransactions(address: Address, limit: number = 10) {
  const transactions = await client.getTransactions(address, {
    limit,
    archival: true  // 查询归档数据
  });
  
  for (const tx of transactions) {
    console.log('Transaction:', {
      hash: tx.hash().toString('hex'),
      time: new Date(tx.time * 1000),
      from: tx.inMessage?.info.src?.toString(),
      to: tx.inMessage?.info.dest?.toString(),
      value: tx.inMessage?.info.value.coins,
      exitCode: tx.exitCode
    });
  }
  
  return transactions;
}
```

---

### 12.3 多语言 SDK 生态

TON 生态提供了多种编程语言的 SDK，方便不同背景的开发者使用。

#### 12.3.1 Python SDK（tonutils）

```bash
pip install tonutils
```

```python
from tonutils.client import ToncenterClient
from tonutils.wallet import WalletV4

# 创建客户端
client = ToncenterClient(api_key='your-api-key')

# 创建钱包
wallet = await WalletV4.from_mnemonic(
    client=client,
    mnemonic=['word1', 'word2', ...]
)

# 查询余额
balance = await wallet.get_balance()
print(f'Balance: {balance} TON')

# 发送交易
await wallet.transfer(
    destination='EQD...',
    amount=0.1,  # TON
    body='Hello TON!'
)
```

#### 12.3.2 Go SDK（tonutils-go）

```go
package main

import (
    "context"
    "fmt"
    "github.com/xssnick/tonutils-go/ton"
    "github.com/xssnick/tonutils-go/liteclient"
)

func main() {
    // 连接 Liteserver
    client := liteclient.NewConnectionPool()
    err := client.AddConnection(context.Background(), "...", "...")
    if err != nil {
        panic(err)
    }
    
    api := ton.NewAPIClient(client)
    
    // 查询账户
    block, err := api.CurrentMasterchainInfo(context.Background())
    if err != nil {
        panic(err)
    }
    
    addr, _ := ton.AddrFromBase64("EQD...")
    account, err := api.GetAccount(context.Background(), block, addr)
    if err != nil {
        panic(err)
    }
    
    fmt.Printf("Balance: %d nanoTON\n", account.State.Balance)
}
```

#### 12.3.3 Rust SDK（ton-rs）

```rust
use ton_client::Client;
use ton_types::AccountId;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // 创建客户端
    let client = Client::new("https://toncenter.com/api/v2/jsonRPC").await?;
    
    // 查询账户
    let address = "EQD...".parse()?;
    let account = client.get_account(&address).await?;
    
    println!("Balance: {}", account.balance);
    
    Ok(())
}
```

#### 12.3.4 Java SDK（ton4j）

```java
import org.ton.TonClient;
import org.ton.Address;

public class Main {
    public static void main(String[] args) {
        // 创建客户端
        TonClient client = TonClient.builder()
            .endpoint("https://toncenter.com/api/v2/jsonRPC")
            .apiKey("your-api-key")
            .build();
        
        // 查询账户
        Address address = Address.of("EQD...");
        Account account = client.getAccount(address);
        
        System.out.println("Balance: " + account.getBalance());
    }
}
```

---

### 12.4 TypeScript Wrapper 模式

Wrapper 模式是 TON 开发中常用的设计模式，用于封装合约交互逻辑。

#### 12.4.1 基础 Wrapper 结构

```typescript
import { Address, Cell, Contract, TonClient, beginCell } from '@ton/core';

export class TokenContract implements Contract {
  constructor(readonly address: Address) {}
  
  // 静态方法：从初始化数据创建
  static createFromConfig(config: TokenConfig, code: Cell, workchain = 0) {
    const data = beginCell()
      .storeAddress(config.owner)
      .storeCoins(config.totalSupply)
      .storeRef(config.metadata)
      .endCell();
    
    const init = { code, data };
    const address = contractAddress(workchain, init);
    
    return new TokenContract(address, init);
  }
  
  // 发送消息
  async sendTransfer(
    provider: TonClient,
    via: Sender,
    params: {
      to: Address;
      amount: bigint;
      value?: bigint;
    }
  ) {
    await provider.internal(via, {
      value: params.value || toNano('0.05'),
      body: beginCell()
        .storeUint(0x12345678, 32)  // op code
        .storeUint(0, 64)           // query id
        .storeAddress(params.to)
        .storeCoins(params.amount)
        .endCell()
    });
  }
  
  // Getter 方法
  async getBalance(provider: TonClient, owner: Address): Promise<bigint> {
    const result = await provider.get('getBalance', [
      { type: 'slice', cell: beginCell().storeAddress(owner).endCell() }
    ]);
    return result.stack.readBigNumber();
  }
  
  async getTotalSupply(provider: TonClient): Promise<bigint> {
    const result = await provider.get('getTotalSupply', []);
    return result.stack.readBigNumber();
  }
}
```

#### 12.4.2 使用 Wrapper

```typescript
import { TokenContract } from './wrappers/TokenContract';

// 创建合约实例
const token = new TokenContract(Address.parse('EQD...'));

// 查询数据
const totalSupply = await token.getTotalSupply(client);
const myBalance = await token.getBalance(client, myAddress);

console.log('Total Supply:', fromNano(totalSupply), 'tokens');
console.log('My Balance:', fromNano(myBalance), 'tokens');

// 发送交易
await token.sendTransfer(client, sender, {
  to: recipientAddress,
  amount: toNano('100')
});
```

#### 12.4.3 完整 Wrapper 示例

```typescript
import {
  Address,
  Cell,
  Contract,
  ContractProvider,
  Sender,
  beginCell,
  toNano,
  contractAddress
} from '@ton/core';

export interface JettonConfig {
  admin: Address;
  content: Cell;
  walletCode: Cell;
}

export interface JettonMint {
  to: Address;
  amount: bigint;
  responseAddress?: Address;
}

export class JettonMaster implements Contract {
  constructor(
    readonly address: Address,
    readonly init?: { code: Cell; data: Cell }
  ) {}
  
  static createFromConfig(config: JettonConfig, code: Cell, workchain = 0) {
    const data = beginCell()
      .storeCoins(0)  // total_supply
      .storeAddress(config.admin)
      .storeRef(config.content)
      .storeRef(config.walletCode)
      .endCell();
    
    const init = { code, data };
    return new JettonMaster(contractAddress(workchain, init), init);
  }
  
  async sendDeploy(provider: ContractProvider, via: Sender, value: bigint) {
    await provider.internal(via, {
      value,
      bounce: false
    });
  }
  
  async sendMint(
    provider: ContractProvider,
    via: Sender,
    params: JettonMint
  ) {
    const message = beginCell()
      .storeUint(21, 32)  // op::mint
      .storeUint(0, 64)   // query_id
      .storeAddress(params.to)
      .storeCoins(toNano('0.02'))  // forward_ton_amount
      .storeRef(
        beginCell()
          .storeUint(0x178d4519, 32)  // internal_transfer
          .storeUint(0, 64)
          .storeCoins(params.amount)
          .storeAddress(this.address)  // from
          .storeAddress(params.responseAddress || params.to)
          .storeCoins(0)
          .storeUint(0, 1)
          .endCell()
      )
      .endCell();
    
    await provider.internal(via, {
      value: toNano('0.05'),
      body: message
    });
  }
  
  async getJettonData(provider: ContractProvider) {
    const result = await provider.get('get_jetton_data', []);
    return {
      totalSupply: result.stack.readBigNumber(),
      mintable: result.stack.readBoolean(),
      admin: result.stack.readAddress(),
      content: result.stack.readCell(),
      walletCode: result.stack.readCell()
    };
  }
  
  async getWalletAddress(provider: ContractProvider, owner: Address) {
    const result = await provider.get('get_wallet_address', [
      { type: 'slice', cell: beginCell().storeAddress(owner).endCell() }
    ]);
    return result.stack.readAddress();
  }
}
```

---

**本章小结：**

本章介绍了 TON SDK 的核心功能：
- **@ton/core**：Cell、Slice、Builder、Address 等核心数据结构
- **@ton/ton**：区块链客户端，用于查询和交互
- **多语言 SDK**：Python、Go、Rust、Java 等语言的 SDK
- **Wrapper 模式**：封装合约交互逻辑的最佳实践

掌握这些 SDK 工具是开发 TON DApp 的基础。下一章将介绍如何使用 TON Connect 连接钱包。

---

## 第 13 章：TON Connect 钱包连接

TON Connect 是 TON 生态的标准钱包连接协议，允许 DApp 安全地连接用户钱包并发送交易。

### 13.1 TON Connect 协议概述

#### 13.1.1 协议特点

```
┌─────────────────────────────────────────────────────────────┐
│                    TON Connect 架构                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   DApp (前端)              桥接服务               钱包       │
│   ┌─────────┐            ┌─────────┐          ┌─────────┐   │
│   │         │ ─────────→│         │────────→│         │   │
│   │  应用   │   连接    │  Bridge │  通信   │  钱包   │   │
│   │         │ ←─────────│         │←────────│         │   │
│   └─────────┘            └─────────┘          └─────────┘   │
│                                                             │
│   特点：                                                      │
│   • 端到端加密                                               │
│   • 无需信任第三方                                           │
│   • 支持多钱包                                               │
│   • 跨平台（Web/移动端）                                      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**核心优势：**

1. **安全性**：端到端加密，私钥永不离开钱包
2. **便捷性**：扫码或点击即可连接
3. **兼容性**：支持所有主流 TON 钱包
4. **标准化**：统一的接口规范

#### 13.1.2 支持的钱包

| 钱包 | 平台 | 特点 |
|------|------|------|
| **Tonkeeper** | iOS/Android/Extension | 最流行，功能全面 |
| **MyTonWallet** | Web/Extension | 开发者友好 |
| **Tonhub** | iOS/Android | 开源，安全性高 |
| **Telegram Wallet** | Telegram 内置 | 无需安装 |
| **Ledger** | 硬件 | 最高安全性 |

---

### 13.2 基础集成

#### 13.2.1 安装 SDK

```bash
npm install @tonconnect/sdk
```

#### 13.2.2 创建 Manifest

创建 `tonconnect-manifest.json` 文件：

```json
{
  "url": "https://your-dapp.com",
  "name": "Your DApp Name",
  "iconUrl": "https://your-dapp.com/icon.png",
  "termsOfUseUrl": "https://your-dapp.com/terms",
  "privacyPolicyUrl": "https://your-dapp.com/privacy"
}
```

**Manifest 要求：**

- `url`：DApp 的完整 URL
- `name`：应用名称（显示在钱包中）
- `iconUrl`：应用图标（建议 512x512 PNG）
- `termsOfUseUrl`：服务条款链接（可选）
- `privacyPolicyUrl`：隐私政策链接（可选）

#### 13.2.3 初始化连接器

```typescript
import { TonConnect } from '@tonconnect/sdk';

// 创建连接器实例
const connector = new TonConnect({
  manifestUrl: 'https://your-dapp.com/tonconnect-manifest.json'
});

// 监听连接状态变化
connector.onStatusChange((wallet) => {
  if (wallet) {
    console.log('Connected:', wallet.account.address);
    console.log('Chain:', wallet.account.chain);  // '-239' 为主网, '-3' 为测试网
    console.log('Wallet:', wallet.device.appName);
  } else {
    console.log('Disconnected');
  }
});
```

#### 13.2.4 连接钱包

```typescript
// 获取可用钱包列表
const wallets = await connector.getWallets();

// 连接特定钱包（例如 Tonkeeper）
const tonkeeper = wallets.find(w => w.appName === 'tonkeeper');

if (tonkeeper) {
  await connector.connect({
    universalLink: tonkeeper.universalLink,
    bridgeUrl: tonkeeper.bridgeUrl
  });
}

// 或者显示钱包选择界面让用户选择
function showWalletSelector(wallets: WalletInfo[]) {
  // 渲染钱包列表 UI
  wallets.forEach(wallet => {
    console.log(wallet.name, wallet.imageUrl);
  });
}
```

#### 13.2.5 断开连接

```typescript
// 断开当前连接
await connector.disconnect();

// 清理状态
connector.pauseConnection();  // 暂停连接（页面隐藏时）
connector.unpauseConnection(); // 恢复连接
```

---

### 13.3 发送交易

#### 13.3.1 基础交易

```typescript
// 发送简单转账
const transaction = {
  validUntil: Math.floor(Date.now() / 1000) + 60,  // 60秒后过期
  messages: [
    {
      address: 'EQD...',  // 目标地址
      amount: toNano('0.1').toString(),  // 金额（nanoTON）
      payload: ''  // 可选的消息体
    }
  ]
};

const result = await connector.sendTransaction(transaction);
console.log('Transaction hash:', result.boc);
```

#### 13.3.2 调用合约

```typescript
import { beginCell, toNano } from '@ton/core';

// 构建消息体
const messageBody = beginCell()
  .storeUint(0x12345678, 32)  // 操作码
  .storeUint(0, 64)           // 查询 ID
  .storeAddress(Address.parse('EQD...'))  // 参数
  .storeCoins(toNano('100'))  // 参数
  .endCell()
  .toBoc()
  .toString('base64');

const transaction = {
  validUntil: Math.floor(Date.now() / 1000) + 300,
  messages: [
    {
      address: contractAddress.toString(),
      amount: toNano('0.05').toString(),
      payload: messageBody
    }
  ]
};

const result = await connector.sendTransaction(transaction);
```

#### 13.3.3 批量交易

```typescript
// 一次发送多笔交易
const transaction = {
  validUntil: Math.floor(Date.now() / 1000) + 300,
  messages: [
    {
      address: 'EQD...',
      amount: toNano('0.1').toString()
    },
    {
      address: 'EQD...',
      amount: toNano('0.2').toString()
    },
    {
      address: contractAddress.toString(),
      amount: toNano('0.05').toString(),
      payload: messageBody
    }
  ]
};

const result = await connector.sendTransaction(transaction);
```

---

### 13.4 React 集成

#### 13.4.1 安装 UI 库

```bash
npm install @tonconnect/ui-react
```

#### 13.4.2 配置 Provider

```tsx
// App.tsx
import { TonConnectUIProvider } from '@tonconnect/ui-react';

function App() {
  return (
    <TonConnectUIProvider 
      manifestUrl="https://your-dapp.com/tonconnect-manifest.json"
      actionsConfiguration={{
        twaReturnUrl: 'https://t.me/your_bot/your_app'  // Telegram Mini App 返回 URL
      }}
    >
      <YourApp />
    </TonConnectUIProvider>
  );
}
```

#### 13.4.3 使用 UI 组件

```tsx
// components/WalletButton.tsx
import { TonConnectButton } from '@tonconnect/ui-react';

export function WalletButton() {
  return <TonConnectButton />;  // 自动显示连接/断开按钮
}
```

#### 13.4.4 使用 Hooks

```tsx
// components/SendTransaction.tsx
import { useTonConnectUI, useTonAddress } from '@tonconnect/ui-react';
import { beginCell, toNano } from '@ton/core';

export function SendTransaction() {
  const [tonConnectUI] = useTonConnectUI();
  const userAddress = useTonAddress();
  
  const handleSend = async () => {
    const transaction = {
      messages: [
        {
          address: 'EQD...',
          amount: toNano('0.1').toString()
        }
      ]
    };
    
    try {
      const result = await tonConnectUI.sendTransaction(transaction);
      console.log('Success:', result);
    } catch (error) {
      console.error('Error:', error);
    }
  };
  
  return (
    <div>
      <p>Connected: {userAddress}</p>
      <button onClick={handleSend}>Send 0.1 TON</button>
    </div>
  );
}
```

#### 13.4.5 自定义连接界面

```tsx
import { useTonConnectUI, useTonConnectModal } from '@tonconnect/ui-react';

export function CustomConnectButton() {
  const [tonConnectUI] = useTonConnectUI();
  const { state, open, close } = useTonConnectModal();
  
  const handleConnect = () => {
    open();  // 打开连接模态框
  };
  
  const handleDisconnect = async () => {
    await tonConnectUI.disconnect();
  };
  
  return (
    <div>
      {tonConnectUI.connected ? (
        <button onClick={handleDisconnect}>
          Disconnect
        </button>
      ) : (
        <button onClick={handleConnect}>
          Connect Wallet
        </button>
      )}
    </div>
  );
}
```

---

### 13.5 高级功能

#### 13.5.1 监听交易状态

```typescript
// 监听连接状态
connector.onStatusChange((wallet) => {
  if (wallet) {
    // 保存钱包信息
    localStorage.setItem('ton-wallet', JSON.stringify({
      address: wallet.account.address,
      chain: wallet.account.chain
    }));
  } else {
    // 清除保存的信息
    localStorage.removeItem('ton-wallet');
  }
});

// 恢复连接
async function restoreConnection() {
  const saved = localStorage.getItem('ton-wallet');
  if (saved) {
    await connector.restoreConnection();
  }
}
```

#### 13.5.2 网络切换

```typescript
// 检查当前网络
function checkNetwork(wallet: ConnectedWallet) {
  const isMainnet = wallet.account.chain === '-239';
  const isTestnet = wallet.account.chain === '-3';
  
  if (!isMainnet && !isTestnet) {
    console.warn('Unknown network');
  }
  
  return { isMainnet, isTestnet };
}

// 请求切换网络
async function switchNetwork(connector: TonConnect) {
  // 断开当前连接
  await connector.disconnect();
  
  // 重新连接（用户选择网络）
  // 实际网络由用户选择的钱包决定
}
```

#### 13.5.3 错误处理

```typescript
async function safeSendTransaction(
  connector: TonConnect,
  transaction: SendTransactionRequest
) {
  try {
    const result = await connector.sendTransaction(transaction);
    return { success: true, result };
  } catch (error) {
    if (error instanceof UserRejectedError) {
      return { success: false, error: 'User rejected' };
    }
    if (error instanceof TonConnectError) {
      return { success: false, error: error.message };
    }
    return { success: false, error: 'Unknown error' };
  }
}
```

---

**本章小结：**

本章介绍了 TON Connect 的使用方法：
- TON Connect 协议概述和特点
- 基础集成（安装、Manifest、连接）
- 发送交易（转账、合约调用、批量交易）
- React 集成（UI 组件、Hooks）
- 高级功能（状态监听、网络切换、错误处理）

TON Connect 是连接 DApp 和用户钱包的标准方式，掌握其使用是开发 TON DApp 的关键。下一章将介绍 AppKit，一个更高级的 DApp 开发工具包。

---

## 第 14 章：AppKit 快速开发

AppKit 是一个高级 DApp 开发工具包，集成了 TON Connect、数据查询和 UI 组件，可以快速构建 TON DApp。

### 14.1 AppKit 概述

#### 14.1.1 功能特点

```
┌─────────────────────────────────────────────────────────────┐
│                      AppKit 功能架构                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │ TON Connect │  │ 数据查询    │  │  UI 组件    │         │
│  │   集成      │  │ (TanStack) │  │             │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │ 余额查询    │  │ 交易历史    │  │  DeFi 集成  │         │
│  │ Jetton/NFT  │  │ 价格数据    │  │  (Swap等)  │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**核心功能：**

1. **TON Connect 集成**：内置钱包连接功能
2. **数据查询**：集成 TanStack Query，自动缓存和刷新
3. **UI 组件**：预制的 React 组件
4. **DeFi 集成**：内置 Swap、Staking 等功能

#### 14.1.2 安装

```bash
npm install @ton/appkit @tanstack/react-query
```

---

### 14.2 基础配置

#### 14.2.1 初始化 AppKit

```tsx
// main.tsx
import { AppKitProvider } from '@ton/appkit';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 30 * 1000,  // 30秒
      refetchInterval: 60 * 1000  // 每分钟刷新
    }
  }
});

ReactDOM.createRoot(document.getElementById('root')!).render(
  <QueryClientProvider client={queryClient}>
    <AppKitProvider
      manifestUrl="https://your-dapp.com/tonconnect-manifest.json"
      defaultNetwork="mainnet"
    >
      <App />
    </AppKitProvider>
  </QueryClientProvider>
);
```

#### 14.2.2 使用 Hooks

```tsx
import {
  useWallet,
  useBalance,
  useTonPrice,
  useJettonBalance
} from '@ton/appkit';

function WalletInfo() {
  const { data: wallet } = useWallet();
  const { data: balance } = useBalance();
  const { data: price } = useTonPrice();
  const { data: usdtBalance } = useJettonBalance(
    'EQCxE6mUtQJKFnGfaROTKOt1lZbDiiX1kCixRv7Nw2Id_sDs'  // USDT 合约地址
  );
  
  if (!wallet) return <div>Connect wallet</div>;
  
  return (
    <div>
      <p>Address: {wallet.account.address}</p>
      <p>Balance: {balance?.formatted} TON</p>
      <p>Value: ${balance && price ? (parseFloat(balance.formatted) * price).toFixed(2) : '-'}</p>
      <p>USDT: {usdtBalance?.formatted}</p>
    </div>
  );
}
```

---

### 14.3 数据查询

#### 14.3.1 余额查询

```tsx
import { useBalance, useJettonBalance, useNftItems } from '@ton/appkit';

function Balances() {
  // TON 余额
  const { data: tonBalance, isLoading: tonLoading } = useBalance();
  
  // Jetton 余额
  const { data: jettonBalance } = useJettonBalance(jettonAddress);
  
  // NFT 列表
  const { data: nfts } = useNftItems(walletAddress);
  
  return (
    <div>
      <h3>Balances</h3>
      {tonLoading ? (
        <p>Loading...</p>
      ) : (
        <p>TON: {tonBalance?.formatted}</p>
      )}
      <p>Jetton: {jettonBalance?.formatted}</p>
      <p>NFTs: {nfts?.length}</p>
    </div>
  );
}
```

#### 14.3.2 交易历史

```tsx
import { useTransactions } from '@ton/appkit';

function TransactionHistory() {
  const { data: transactions, isLoading } = useTransactions({
    limit: 10,
    refetchInterval: 30000  // 每30秒刷新
  });
  
  if (isLoading) return <div>Loading...</div>;
  
  return (
    <div>
      <h3>Recent Transactions</h3>
      {transactions?.map((tx) => (
        <div key={tx.hash}>
          <p>Hash: {tx.hash.slice(0, 16)}...</p>
          <p>Amount: {tx.value}</p>
          <p>Time: {new Date(tx.time * 1000).toLocaleString()}</p>
        </div>
      ))}
    </div>
  );
}
```

---

### 14.4 DeFi 集成

#### 14.4.1 Swap 功能

```tsx
import { useSwap, useSwapQuote } from '@ton/appkit';

function SwapComponent() {
  const [fromAmount, setFromAmount] = useState('');
  const [fromToken, setFromToken] = useState('TON');
  const [toToken, setToToken] = useState('USDT');
  
  // 获取报价
  const { data: quote } = useSwapQuote({
    fromToken,
    toToken,
    amount: fromAmount
  });
  
  // 执行交换
  const { mutate: swap, isPending } = useSwap({
    onSuccess: (result) => {
      console.log('Swap successful:', result);
    },
    onError: (error) => {
      console.error('Swap failed:', error);
    }
  });
  
  return (
    <div>
      <input
        value={fromAmount}
        onChange={(e) => setFromAmount(e.target.value)}
        placeholder="Amount"
      />
      <p>Expected: {quote?.expectedOutput}</p>
      <button
        onClick={() => swap({ fromToken, toToken, amount: fromAmount })}
        disabled={isPending}
      >
        {isPending ? 'Swapping...' : 'Swap'}
      </button>
    </div>
  );
}
```

#### 14.4.2 Staking 功能

```tsx
import { useStakingInfo, useStake, useUnstake } from '@ton/appkit';

function StakingComponent() {
  const { data: stakingInfo } = useStakingInfo();
  const { mutate: stake, isPending: isStaking } = useStake();
  const { mutate: unstake, isPending: isUnstaking } = useUnstake();
  
  return (
    <div>
      <h3>Staking</h3>
      <p>APY: {stakingInfo?.apy}%</p>
      <p>Staked: {stakingInfo?.stakedAmount} TON</p>
      <p>Rewards: {stakingInfo?.pendingRewards} TON</p>
      
      <button
        onClick={() => stake({ amount: '100' })}
        disabled={isStaking}
      >
        Stake 100 TON
      </button>
      
      <button
        onClick={() => unstake({ amount: '50' })}
        disabled={isUnstaking}
      >
        Unstake 50 TON
      </button>
    </div>
  );
}
```

---

**本章小结：**

本章介绍了 AppKit 的使用方法：
- AppKit 概述和功能特点
- 基础配置和初始化
- 数据查询（余额、交易历史）
- DeFi 集成（Swap、Staking）

AppKit 大大简化了 TON DApp 的开发流程，提供了开箱即用的功能。下一章将介绍 Telegram Mini Apps 开发。

---

## 第 15 章：Telegram Mini Apps 开发

Telegram Mini Apps 是运行在 Telegram 内的 Web 应用，可以无缝集成 TON 区块链，触达 Telegram 的 9 亿用户。

### 15.1 Mini Apps 概述

#### 15.1.1 什么是 Mini Apps

```
┌─────────────────────────────────────────────────────────────┐
│                   Telegram Mini Apps 架构                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                  Telegram App                        │   │
│  │  ┌─────────────────────────────────────────────┐   │   │
│  │  │           Mini App (WebView)                 │   │   │
│  │  │  ┌─────────┐  ┌─────────┐  ┌─────────┐      │   │   │
│  │  │  │  UI     │  │ TON SDK │  │ TG API  │      │   │   │
│  │  │  └─────────┘  └─────────┘  └─────────┘      │   │   │
│  │  └─────────────────────────────────────────────┘   │   │
│  │                                                      │   │
│  │  • 无需安装，即点即用                                 │   │
│  │  • 内置 TON 钱包                                     │   │
│  │  • 病毒式传播能力                                     │   │
│  │                                                      │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**核心优势：**

1. **庞大用户基础**：Telegram 9 亿用户
2. **无需安装**：Web 技术，即点即用
3. **内置钱包**：Telegram Wallet 集成
4. **社交传播**：易于分享和邀请
5. **跨平台**：iOS、Android、桌面端

#### 15.1.2 开发环境

```bash
# 创建 Mini App 项目
npx create-ton-app@latest my-mini-app
cd my-mini-app
npm install

# 安装 Telegram SDK
npm install @telegram-apps/sdk
```

---

### 15.2 Telegram SDK 使用

#### 15.2.1 初始化 SDK

```tsx
// App.tsx
import { useEffect } from 'react';
import { init, miniApp, viewport } from '@telegram-apps/sdk';

function App() {
  useEffect(() => {
    // 初始化 Telegram SDK
    init();
    
    // 扩展视口到全屏
    viewport.expand();
    
    // 设置 Mini App 标题
    miniApp.setHeaderColor('#000000');
    miniApp.setBackgroundColor('#000000');
    miniApp.ready();
  }, []);
  
  return <YourApp />;
}
```

#### 15.2.2 获取用户信息

```tsx
import { useLaunchParams, useInitData } from '@telegram-apps/sdk-react';

function UserInfo() {
  const initData = useInitData();
  const lp = useLaunchParams();
  
  return (
    <div>
      <p>User: {initData?.user?.firstName}</p>
      <p>Username: @{initData?.user?.username}</p>
      <p>Language: {initData?.user?.languageCode}</p>
      <p>Platform: {lp?.platform}</p>  // ios, android, web, etc.
    </div>
  );
}
```

#### 15.2.3 主按钮交互

```tsx
import { useMainButton } from '@telegram-apps/sdk-react';

function GameComponent() {
  const mainButton = useMainButton();
  
  useEffect(() => {
    mainButton.setParams({
      text: 'Claim Reward',
      isVisible: true,
      isEnabled: true
    });
    
    const handleClick = () => {
      claimReward();
    };
    
    mainButton.on('click', handleClick);
    
    return () => {
      mainButton.off('click', handleClick);
    };
  }, [mainButton]);
  
  return <div>Game Content</div>;
}
```

#### 15.2.4 主题适配

```tsx
import { useThemeParams } from '@telegram-apps/sdk-react';

function ThemedComponent() {
  const theme = useThemeParams();
  
  return (
    <div
      style={{
        backgroundColor: theme.bgColor,
        color: theme.textColor,
        buttonColor: theme.buttonColor,
        buttonTextColor: theme.buttonTextColor
      }}
    >
      <p>This text adapts to Telegram theme</p>
    </div>
  );
}
```

---

### 15.3 TON 集成

#### 15.3.1 连接 Telegram Wallet

```tsx
import { TonConnectUIProvider, useTonConnectUI } from '@tonconnect/ui-react';

function MiniApp() {
  return (
    <TonConnectUIProvider
      manifestUrl="https://your-app.com/tonconnect-manifest.json"
      actionsConfiguration={{
        twaReturnUrl: 'https://t.me/your_bot/your_app'
      }}
    >
      <AppContent />
    </TonConnectUIProvider>
  );
}

function AppContent() {
  const [tonConnectUI] = useTonConnectUI();
  
  const handleConnect = async () => {
    await tonConnectUI.openModal();
  };
  
  return (
    <div>
      <button onClick={handleConnect}>
        Connect Telegram Wallet
      </button>
    </div>
  );
}
```

#### 15.3.2 发送交易

```tsx
import { useTonConnectUI } from '@tonconnect/ui-react';
import { toNano } from '@ton/core';

function SendTon() {
  const [tonConnectUI] = useTonConnectUI();
  
  const sendTransaction = async () => {
    await tonConnectUI.sendTransaction({
      messages: [
        {
          address: 'EQD...',
          amount: toNano('0.1').toString()
        }
      ]
    });
  };
  
  return (
    <button onClick={sendTransaction}>
      Send 0.1 TON
    </button>
  );
}
```

---

### 15.4 实战：开发 Mini App 游戏

#### 15.4.1 项目结构

```
my-mini-app/
├── src/
│   ├── components/
│   │   ├── Game.tsx
│   │   ├── Wallet.tsx
│   │   └── Leaderboard.tsx
│   ├── hooks/
│   │   ├── useGame.ts
│   │   └── useTon.ts
│   ├── contracts/
│   │   └── game.tact
│   ├── App.tsx
│   └── main.tsx
├── public/
│   └── tonconnect-manifest.json
├── package.json
└── vite.config.ts
```

#### 15.4.2 游戏合约

```tact
// contracts/game.tact
import "@stdlib/deploy";

message Play {
    bet: Int as coins;
}

message ClaimReward {
    amount: Int as coins;
}

contract Game with Deployable {
    owner: Address;
    players: map<Address, Int>;  // 玩家分数
    totalBets: Int as coins = 0;
    
    init(owner: Address) {
        self.owner = owner;
    }
    
    receive(msg: Play) {
        let player = sender();
        
        // 记录投注
        self.totalBets = self.totalBets + msg.bet;
        
        // 生成随机结果（简化示例）
        let score = randomInt(100);
        self.players.set(player, score);
        
        // 发送结果通知
        send(SendParameters{
            to: player,
            value: 0,
            mode: SendIgnoreErrors,
            body: beginCell()
                .storeUint(0x12345678, 32)
                .storeUint(score, 64)
                .endCell()
        });
    }
    
    receive(msg: ClaimReward) {
        let player = sender();
        let score = self.players.get(player) ?: 0;
        
        require(score > 80, "Score too low");
        
        // 计算奖励
        let reward = msg.amount;
        require(self.totalBets >= reward, "Insufficient pool");
        
        self.totalBets = self.totalBets - reward;
        
        // 发送奖励
        send(SendParameters{
            to: player,
            value: reward,
            mode: SendPayGasSeparately
        });
    }
    
    get fun getScore(player: Address): Int {
        return self.players.get(player) ?: 0;
    }
    
    get fun getTotalBets(): Int {
        return self.totalBets;
    }
}
```

#### 15.4.3 前端实现

```tsx
// src/components/Game.tsx
import { useState, useEffect } from 'react';
import { useTonConnectUI } from '@tonconnect/ui-react';
import { useMainButton } from '@telegram-apps/sdk-react';
import { toNano, Address } from '@ton/core';
import { GameContract } from '../contracts/GameContract';

export function Game() {
  const [score, setScore] = useState(0);
  const [isPlaying, setIsPlaying] = useState(false);
  const [tonConnectUI] = useTonConnectUI();
  const mainButton = useMainButton();
  
  const gameContract = new GameContract(
    Address.parse('EQD...')
  );
  
  useEffect(() => {
    mainButton.setParams({
      text: isPlaying ? 'Playing...' : 'Play (0.1 TON)',
      isVisible: true,
      isEnabled: !isPlaying && tonConnectUI.connected
    });
    
    const handleClick = async () => {
      if (!tonConnectUI.connected) {
        tonConnectUI.openModal();
        return;
      }
      
      setIsPlaying(true);
      
      try {
        // 发送游戏交易
        await tonConnectUI.sendTransaction({
          messages: [
            {
              address: gameContract.address.toString(),
              amount: toNano('0.1').toString(),
              payload: gameContract.createPlayPayload(toNano('0.1'))
            }
          ]
        });
        
        // 等待结果
        await new Promise(resolve => setTimeout(resolve, 5000));
        
        // 查询分数
        const newScore = await gameContract.getScore(
          Address.parse(tonConnectUI.account!.address)
        );
        setScore(Number(newScore));
        
        if (newScore > 80n) {
          alert(`Congratulations! Score: ${newScore}. You can claim reward!`);
        }
      } catch (error) {
        console.error('Game error:', error);
      } finally {
        setIsPlaying(false);
      }
    };
    
    mainButton.on('click', handleClick);
    
    return () => {
      mainButton.off('click', handleClick);
    };
  }, [isPlaying, tonConnectUI.connected]);
  
  return (
    <div className="game-container">
      <h1>Mini Game</h1>
      <div className="score">Score: {score}</div>
      {!tonConnectUI.connected && (
        <p>Connect wallet to play</p>
      )}
    </div>
  );
}
```

#### 15.4.4 部署和发布

**1. 部署合约：**

```bash
npx blueprint build
npx blueprint run --mainnet
```

**2. 配置 Bot：**

```bash
# 使用 BotFather 创建 Bot
# 1. 发送 /newbot
# 2. 设置名称和用户名
# 3. 启用 Mini App 模式
```

**3. 设置 Web App：**

```bash
# 在 BotFather 中
# 1. 发送 /mybots
# 2. 选择你的 Bot
# 3. 选择 Bot Settings -> Menu Button -> Configure menu button
# 4. 设置 Web App URL
```

**4. 构建和部署：**

```bash
# 构建
npm run build

# 部署到 Vercel/Netlify
npx vercel --prod
```

---

**本章小结：**

本章介绍了 Telegram Mini Apps 的开发：
- Mini Apps 概述和优势
- Telegram SDK 的使用
- TON 钱包集成
- 实战：开发 Mini App 游戏
- 部署和发布流程

Telegram Mini Apps 是 TON 生态的重要组成部分，为开发者提供了触达海量用户的机会。应用篇到此结束，接下来将进入进阶篇，学习更高级的合约开发技术。

---

**应用篇小结：**

应用篇涵盖了 TON DApp 开发的完整流程：
- **第12章**：TON SDK 核心功能（@ton/core、@ton/ton）
- **第13章**：TON Connect 钱包连接协议
- **第14章**：AppKit 快速开发工具包
- **第15章**：Telegram Mini Apps 开发

掌握这些内容后，开发者可以独立构建完整的 TON DApp，包括前端界面、钱包连接和合约交互。

---
