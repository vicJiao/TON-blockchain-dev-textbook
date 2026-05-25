# 第13章：TON Connect钱包连接 - 播客文字稿

## 开场白

大家好，欢迎收听TON区块链开发教材第13章的播客讲解。我是你们的主播。

想象一下，你开了一家网店，顾客浏览商品、加入购物车都很顺利，但到了付款环节却发现——没有支付方式！这就是没有钱包连接功能的DApp面临的尴尬处境。

今天我们要聊的TON Connect，就是解决这个问题的关键。它是TON生态的标准钱包连接协议，让用户能够安全、便捷地使用他们的钱包与你的应用交互。让我们开始吧！

---

## 第一部分：TON Connect协议概述

### 什么是TON Connect？

TON Connect是TON区块链的官方钱包连接协议。你可以把它想象成一座桥梁——一座连接你的应用（DApp）和用户钱包的桥梁。

没有这座桥，用户就像站在河对岸，看得见你的应用，却无法进行任何需要钱包操作的功能。有了TON Connect，用户可以轻松地从他们的钱包"走到"你的应用里，完成转账、签名等各种操作。

### TON Connect vs 其他连接方式

你可能听说过其他区块链的钱包连接方式，比如MetaMask的浏览器注入方式。TON Connect有什么不同呢？

**MetaMask模式**：钱包作为浏览器扩展，直接往网页里"注入"一个全局对象。这种方式简单，但有一些问题：
- 只能用于浏览器环境
- 移动端支持不好
- 安全性依赖于扩展本身

**TON Connect模式**：基于二维码扫描和深度链接的协议。这种方式的优势：
- 跨平台：支持Web、iOS、Android
- 钱包无关：用户可以选择任何支持TON Connect的钱包
- 安全性高：私钥永远不会离开钱包应用
- 用户体验好：扫码即可连接

### TON Connect的工作原理

让我用一个购物的场景来解释TON Connect的工作流程：

**第一步：建立连接（扫码进店）**
- 你的应用生成一个连接二维码
- 用户打开钱包App，扫描二维码
- 钱包和应用之间建立一个加密通道

**第二步：发起请求（挑选商品）**
- 用户在应用中选择要执行的操作（比如转账）
- 应用生成一个请求，发送给钱包

**第三步：用户确认（付款）**
- 钱包收到请求，显示给用户确认
- 用户在钱包中查看详情并确认

**第四步：返回结果（交易完成）**
- 钱包执行操作，返回结果给应用
- 应用更新状态，显示成功信息

整个过程就像你在商店购物：进店（连接）→ 选商品（发起请求）→ 付款（确认）→ 拿收据（返回结果）。

### 安全性设计

TON Connect在设计上非常注重安全性：

1. **端到端加密**：应用和钱包之间的通信是加密的，中间人无法窃听
2. **私钥不出钱包**：用户的私钥永远保存在钱包里，不会暴露给应用
3. **用户确认**：每笔交易都需要用户在钱包中手动确认
4. **域名验证**：钱包会显示发起请求的域名，防止钓鱼攻击

---

## 第二部分：@tonconnect/ui-react集成

好了，理论讲完，让我们看看实际怎么写代码。对于React开发者，@tonconnect/ui-react是最方便的集成方式。

### 安装和配置

首先，安装必要的包：

```bash
npm install @tonconnect/ui-react
```

然后，在你的应用根组件中配置TON Connect：

```tsx
import { TonConnectUIProvider } from '@tonconnect/ui-react';

function App() {
    return (
        <TonConnectUIProvider 
            manifestUrl="https://your-app.com/tonconnect-manifest.json"
        >
            <YourApp />
        </TonConnectUIProvider>
    );
}
```

这里的`manifestUrl`指向一个JSON文件，包含你的应用信息：

```json
{
    "url": "https://your-app.com",
    "name": "My TON App",
    "iconUrl": "https://your-app.com/icon.png",
    "termsOfUseUrl": "https://your-app.com/terms",
    "privacyPolicyUrl": "https://your-app.com/privacy"
}
```

这就像给你的应用办了一张"身份证"，钱包可以看到你是谁、来自哪里。

### 添加连接按钮

配置好后，添加连接按钮非常简单：

```tsx
import { TonConnectButton } from '@tonconnect/ui-react';

function Header() {
    return (
        <header>
            <h1>我的TON应用</h1>
            <TonConnectButton />
        </header>
    );
}
```

`TonConnectButton`是一个现成的组件，它会自动处理：
- 显示"连接钱包"按钮
- 点击后弹出钱包选择界面
- 连接后显示钱包地址和断开按钮

### 获取连接状态

想知道用户是否已经连接了钱包？使用`useTonConnectUI`钩子：

```tsx
import { useTonConnectUI } from '@tonconnect/ui-react';

function UserProfile() {
    const [tonConnectUI] = useTonConnectUI();
    
    // 检查是否已连接
    const isConnected = tonConnectUI.connected;
    
    // 获取钱包地址
    const walletAddress = tonConnectUI.account?.address;
    
    return (
        <div>
            {isConnected ? (
                <p>已连接: {walletAddress}</p>
            ) : (
                <p>请先连接钱包</p>
            )}
        </div>
    );
}
```

### 自定义UI

如果你想要更个性化的界面，可以使用底层的`@tonconnect/sdk`：

```tsx
import { TonConnect } from '@tonconnect/sdk';

const connector = new TonConnect({
    manifestUrl: 'https://your-app.com/tonconnect-manifest.json'
});

// 监听连接状态变化
connector.onStatusChange(wallet => {
    if (wallet) {
        console.log('已连接:', wallet.account.address);
    } else {
        console.log('已断开');
    }
});

// 生成连接二维码
const universalLink = connector.connect({
    universalLink: 'https://app.tonkeeper.com/ton-connect',
    bridgeUrl: 'https://bridge.tonapi.io/bridge'
});

// 显示二维码给用户扫描
```

---

## 第三部分：交易发送

连接钱包的最终目的是进行交易。让我们看看如何发送交易。

### 发送简单转账

最常见的操作是发送TON代币：

```tsx
import { useTonConnectUI } from '@tonconnect/ui-react';
import { Address } from '@ton/core';

function SendTon() {
    const [tonConnectUI] = useTonConnectUI();
    
    const handleSend = async () => {
        // 构建交易
        const transaction = {
            validUntil: Math.floor(Date.now() / 1000) + 600, // 10分钟后过期
            messages: [
                {
                    address: 'EQD...',  // 接收地址
                    amount: '1000000000', // 1 TON (单位是nanoTON)
                    payload: '', // 可选的消息体
                }
            ]
        };
        
        try {
            // 发送交易
            const result = await tonConnectUI.sendTransaction(transaction);
            console.log('交易成功:', result);
        } catch (error) {
            console.error('交易失败:', error);
        }
    };
    
    return <button onClick={handleSend}>发送1 TON</button>;
}
```

### 发送带消息体的交易

有时候，你需要在转账的同时发送一些数据给智能合约：

```tsx
import { beginCell, toNano } from '@ton/core';

function SendWithMessage() {
    const [tonConnectUI] = useTonConnectUI();
    
    const handleDeposit = async () => {
        // 构建消息体
        const body = beginCell()
            .storeUint(0x12345678, 32)  // 操作码
            .storeUint(0, 64)            // query_id
            .storeAddress(myAddress)     // 用户地址
            .endCell()
            .toBoc()
            .toString('base64');
        
        const transaction = {
            validUntil: Math.floor(Date.now() / 1000) + 600,
            messages: [
                {
                    address: 'EQD...',  // 合约地址
                    amount: toNano('2').toString(), // 2 TON
                    payload: body,      // 消息体
                }
            ]
        };
        
        await tonConnectUI.sendTransaction(transaction);
    };
    
    return <button onClick={handleDeposit}>存款到合约</button>;
}
```

### 批量转账

TON Connect还支持一次发送多笔交易：

```tsx
const transaction = {
    validUntil: Math.floor(Date.now() / 1000) + 600,
    messages: [
        {
            address: 'EQD...',
            amount: '1000000000',
        },
        {
            address: 'EQD...',
            amount: '2000000000',
        },
        {
            address: 'EQD...',
            amount: '3000000000',
        }
    ]
};

await tonConnectUI.sendTransaction(transaction);
```

这就像一次填写多张支票，一次性处理多个转账。

### 交易状态监听

发送交易后，你可能想知道交易是否被确认：

```tsx
import { useEffect } from 'react';
import { useTonConnectUI } from '@tonconnect/ui-react';

function TransactionStatus({ txHash }: { txHash: string }) {
    const [tonConnectUI] = useTonConnectUI();
    
    useEffect(() => {
        const checkStatus = async () => {
            // 使用TON API查询交易状态
            const response = await fetch(
                `https://toncenter.com/api/v2/getTransactions?address=${txHash}`
            );
            const data = await response.json();
            
            if (data.result && data.result.length > 0) {
                console.log('交易已确认!');
            }
        };
        
        const interval = setInterval(checkStatus, 5000); // 每5秒检查一次
        return () => clearInterval(interval);
    }, [txHash]);
    
    return <div>交易状态: 确认中...</div>;
}
```

---

## 第四部分：钱包兼容性

### 支持TON Connect的钱包

TON Connect是一个开放协议，很多钱包都支持它。主流的支持钱包包括：

1. **Tonkeeper** - 最流行的TON钱包，功能全面
2. **Tonhub** - 老牌钱包，安全可靠
3. **MyTonWallet** - 浏览器扩展钱包，类似MetaMask
4. **OpenMask** - 另一个流行的浏览器扩展
5. **SafePal** - 硬件钱包，支持TON

### 处理不同钱包的差异

虽然TON Connect是标准协议，但不同钱包可能有一些细微差异：

```tsx
import { useTonConnectUI } from '@tonconnect/ui-react';

function WalletInfo() {
    const [tonConnectUI] = useTonConnectUI();
    const wallet = tonConnectUI.wallet;
    
    if (!wallet) return null;
    
    return (
        <div>
            <p>钱包名称: {wallet.device.appName}</p>
            <p>平台: {wallet.device.platform}</p>
            <p>版本: {wallet.device.appVersion}</p>
            
            {/* 根据钱包类型显示不同提示 */}
            {wallet.device.appName === 'Tonkeeper' && (
                <p>提示：Tonkeeper支持所有功能</p>
            )}
            {wallet.device.appName === 'MyTonWallet' && (
                <p>提示：浏览器扩展钱包，请保持扩展开启</p>
            )}
        </div>
    );
}
```

### 处理连接错误

在实际应用中，各种错误都可能发生。良好的错误处理很重要：

```tsx
async function handleTransaction() {
    try {
        const result = await tonConnectUI.sendTransaction(transaction);
        // 成功处理
    } catch (error) {
        if (error.message.includes('Rejected by user')) {
            alert('您取消了交易');
        } else if (error.message.includes('timeout')) {
            alert('连接超时，请重试');
        } else {
            alert('发生错误: ' + error.message);
        }
    }
}
```

### 最佳实践

最后，分享一些使用TON Connect的最佳实践：

1. **始终检查连接状态**：在执行操作前，确认用户已连接钱包
2. **设置合理的过期时间**：交易的有效期不要太短也不要太长，5-10分钟合适
3. **提供清晰的UI反馈**：让用户知道交易正在进行中
4. **处理错误情况**：网络问题、用户取消等情况都要处理
5. **测试多个钱包**：确保你的应用在不同钱包中都能正常工作

---

## 总结

让我们回顾一下今天学到的内容。

首先，我们了解了TON Connect协议。它是TON生态的标准钱包连接协议，通过二维码和深度链接实现应用和钱包的安全连接。

然后，我们学习了如何在React应用中集成@tonconnect/ui-react。从安装配置到添加连接按钮，再到获取连接状态，整个过程非常流畅。

接着，我们深入探讨了交易发送。从简单的TON转账，到带消息体的合约调用，再到批量转账，我们覆盖了各种常见场景。

最后，我们聊了钱包兼容性和最佳实践。TON Connect支持多种钱包，作为开发者，我们需要确保应用在各种环境下都能正常工作。

## 下一章预告

在下一章，我们将学习AppKit——这是一个更高级的React组件库，专为TON DApp开发设计。我们会了解AppKit的架构，学习如何进行数据查询（使用TanStack Query），以及如何集成DeFi功能。

如果说TON Connect是连接用户钱包的桥梁，那么AppKit就是一座完整的"开发大厦"，它提供了从UI组件到数据管理的全套解决方案。想要快速构建专业的TON应用？下一章你一定不能错过！

好了，今天的播客就到这里。感谢收听，我们下一章再见！
