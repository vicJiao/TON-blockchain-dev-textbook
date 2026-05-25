# 第14章：AppKit快速开发 - 播客文字稿

## 开场白

大家好，欢迎收听TON区块链开发教材第14章的播客讲解。我是你们的主播。

想象一下，你要盖一栋房子。你可以从打地基、烧砖块开始，一点一点亲手建造。但这需要大量的时间和精力。或者，你可以使用预制构件——工厂已经帮你做好了标准化的部件，你只需要组装起来就行了。

在TON DApp开发中，AppKit就是那些"预制构件"。它是一个专门为TON设计的React组件库，让你能够快速构建功能完善的去中心化应用。今天，我们就来深入了解这个强大的工具。

---

## 第一部分：AppKit概述

### 什么是AppKit？

AppKit是由TON社区开发的一套React组件库和工具集。你可以把它想象成一个"乐高套装"——里面包含了各种预先设计好的组件，从基础的按钮、卡片，到复杂的代币交换界面、流动性池管理，应有尽有。

使用AppKit，你不需要从零开始设计每一个UI元素，也不需要自己处理所有的区块链交互逻辑。它帮你封装好了这些复杂性，让你可以专注于应用的业务逻辑。

### AppKit的核心特性

AppKit提供了哪些功能呢？让我列举几个核心特性：

**1. 预构建的UI组件**
- 连接钱包按钮
- 代币选择器
- 交易确认弹窗
- 余额显示组件
- 等等...

**2. 内置的区块链交互**
- 自动处理钱包连接
- 代币余额查询
- 交易发送和状态追踪
- 智能合约交互

**3. 数据管理集成**
- 与TanStack Query（原React Query）集成
- 自动缓存和刷新数据
- 乐观更新

**4. DeFi功能支持**
- DEX交易功能
- 流动性管理
- 质押和收益农场

### AppKit vs 直接使用TON Connect

你可能会有疑问：我已经学会了TON Connect，为什么还要用AppKit？

让我用一个比喻来解释：

- **直接使用TON Connect**：就像你买了一堆原材料——木头、钉子、油漆。你可以做出任何东西，但需要自己测量、切割、组装。

- **使用AppKit**：就像你买了一套宜家家具。部件已经预制好了，有说明书，你只需要按照步骤组装即可。

如果你要做一个简单的功能，直接使用TON Connect可能更灵活。但如果你要构建一个功能完善的DeFi应用，AppKit能帮你节省大量的开发时间。

---

## 第二部分：React集成

### 安装AppKit

让我们看看如何在React项目中集成AppKit：

```bash
npm install @tonapp/sdk @tonapp/react
```

AppKit分为两个包：
- `@tonapp/sdk`：核心SDK，与框架无关
- `@tonapp/react`：React绑定，提供Hooks和组件

### 配置AppKitProvider

和TON Connect一样，AppKit也需要一个Provider：

```tsx
import { AppKitProvider } from '@tonapp/react';

function App() {
    return (
        <AppKitProvider
            config={{
                projectId: 'YOUR_PROJECT_ID',
                metadata: {
                    name: 'My DeFi App',
                    description: 'A TON DeFi application',
                    url: 'https://myapp.com',
                    icons: ['https://myapp.com/icon.png']
                },
                defaultChain: 'mainnet',
            }}
        >
            <YourApp />
        </AppKitProvider>
    );
}
```

### 使用内置组件

配置好后，你就可以使用AppKit提供的组件了：

```tsx
import { 
    ConnectButton, 
    AccountDisplay,
    TokenBalance,
    SendTransaction 
} from '@tonapp/react';

function MyPage() {
    return (
        <div>
            {/* 连接按钮 */}
            <ConnectButton />
            
            {/* 显示账户信息 */}
            <AccountDisplay />
            
            {/* 显示代币余额 */}
            <TokenBalance tokenAddress="EQD..." />
            
            {/* 发送交易组件 */}
            <SendTransaction 
                to="EQD..."
                amount="1"
                onSuccess={(tx) => console.log('成功:', tx)}
            />
        </div>
    );
}
```

看，多么简洁！你只需要几行代码，就能实现之前需要几十行代码才能完成的功能。

### 自定义主题

AppKit的组件支持主题定制，你可以让它们符合你的品牌风格：

```tsx
<AppKitProvider
    config={{
        // ...其他配置
        theme: {
            colors: {
                primary: '#0088CC',     // TON蓝色
                secondary: '#2C2C2C',   // 深色
                success: '#4CAF50',     // 成功绿
                error: '#F44336',       // 错误红
                background: '#FFFFFF',  // 背景色
                text: '#000000',        // 文字色
            },
            borderRadius: {
                small: '4px',
                medium: '8px',
                large: '16px',
            },
            fonts: {
                main: 'Inter, sans-serif',
            }
        }
    }}
>
```

### 使用Hooks

除了组件，AppKit还提供了丰富的Hooks：

```tsx
import { 
    useAccount, 
    useBalance, 
    useConnect,
    useDisconnect 
} from '@tonapp/react';

function CustomComponent() {
    // 获取账户信息
    const { address, isConnected } = useAccount();
    
    // 获取余额
    const { data: balance, isLoading } = useBalance();
    
    // 连接和断开
    const { connect } = useConnect();
    const { disconnect } = useDisconnect();
    
    return (
        <div>
            {isConnected ? (
                <>
                    <p>地址: {address}</p>
                    <p>余额: {isLoading ? '加载中...' : `${balance} TON`}</p>
                    <button onClick={disconnect}>断开</button>
                </>
            ) : (
                <button onClick={connect}>连接钱包</button>
            )}
        </div>
    );
}
```

---

## 第三部分：数据查询（TanStack Query）

### 为什么需要TanStack Query？

在Web应用中，数据管理是一个复杂的问题。你需要处理：
- 数据获取
- 缓存
- 错误处理
- 加载状态
- 自动刷新

TanStack Query（以前叫React Query）是一个专门解决这些问题的库。AppKit与它深度集成，让数据管理变得轻而易举。

### 基础查询

使用AppKit的`useQuery`钩子获取数据：

```tsx
import { useQuery } from '@tanstack/react-query';
import { useTonClient } from '@tonapp/react';

function TokenInfo({ tokenAddress }: { tokenAddress: string }) {
    const client = useTonClient();
    
    const { data, isLoading, error } = useQuery({
        queryKey: ['token', tokenAddress],
        queryFn: async () => {
            // 获取代币信息
            const result = await client.runMethod(
                Address.parse(tokenAddress),
                'get_token_data',
                []
            );
            return {
                name: result.stack.readString(),
                symbol: result.stack.readString(),
                totalSupply: result.stack.readBigNumber(),
            };
        },
        // 每30秒自动刷新
        refetchInterval: 30000,
    });
    
    if (isLoading) return <div>加载中...</div>;
    if (error) return <div>错误: {error.message}</div>;
    
    return (
        <div>
            <h2>{data?.name} ({data?.symbol})</h2>
            <p>总供应量: {data?.totalSupply.toString()}</p>
        </div>
    );
}
```

### 并行查询

有时候你需要同时获取多个数据：

```tsx
function Dashboard() {
    const { address } = useAccount();
    
    // 并行查询余额和交易历史
    const balanceQuery = useQuery({
        queryKey: ['balance', address],
        queryFn: () => fetchBalance(address),
    });
    
    const transactionsQuery = useQuery({
        queryKey: ['transactions', address],
        queryFn: () => fetchTransactions(address),
    });
    
    const isLoading = balanceQuery.isLoading || transactionsQuery.isLoading;
    
    if (isLoading) return <div>加载中...</div>;
    
    return (
        <div>
            <BalanceDisplay balance={balanceQuery.data} />
            <TransactionList transactions={transactionsQuery.data} />
        </div>
    );
}
```

### 依赖查询

有时候，一个查询依赖于另一个查询的结果：

```tsx
function TokenBalance({ tokenAddress }: { tokenAddress: string }) {
    const { address } = useAccount();
    
    // 先获取钱包地址
    const walletQuery = useQuery({
        queryKey: ['wallet', address],
        queryFn: () => fetchWalletData(address),
    });
    
    // 然后获取代币余额（依赖于钱包地址）
    const balanceQuery = useQuery({
        queryKey: ['tokenBalance', walletQuery.data?.address, tokenAddress],
        queryFn: () => fetchTokenBalance(walletQuery.data!.address, tokenAddress),
        // 只有钱包数据获取成功后才执行
        enabled: !!walletQuery.data,
    });
    
    // ...
}
```

### 数据修改（Mutations）

不只是查询，TanStack Query也处理数据修改：

```tsx
import { useMutation, useQueryClient } from '@tanstack/react-query';

function TransferButton({ to, amount }: { to: string; amount: string }) {
    const queryClient = useQueryClient();
    const { sendTransaction } = useSendTransaction();
    
    const mutation = useMutation({
        mutationFn: async () => {
            const result = await sendTransaction({
                to,
                amount,
            });
            return result;
        },
        onSuccess: () => {
            // 交易成功后，刷新余额
            queryClient.invalidateQueries({ queryKey: ['balance'] });
            alert('转账成功！');
        },
        onError: (error) => {
            alert('转账失败: ' + error.message);
        },
    });
    
    return (
        <button 
            onClick={() => mutation.mutate()}
            disabled={mutation.isPending}
        >
            {mutation.isPending ? '处理中...' : '转账'}
        </button>
    );
}
```

### 乐观更新

乐观更新是一种提升用户体验的技术。我们在请求发送后立即更新UI，如果请求失败再回滚：

```tsx
const mutation = useMutation({
    mutationFn: sendTransaction,
    onMutate: async (newTransaction) => {
        // 取消正在进行的重新获取
        await queryClient.cancelQueries({ queryKey: ['balance'] });
        
        // 保存当前值
        const previousBalance = queryClient.getQueryData(['balance']);
        
        // 乐观更新余额
        queryClient.setQueryData(['balance'], (old: bigint) => {
            return old - BigInt(newTransaction.amount);
        });
        
        // 返回上下文，用于回滚
        return { previousBalance };
    },
    onError: (err, newTransaction, context) => {
        // 出错时回滚
        queryClient.setQueryData(['balance'], context?.previousBalance);
    },
    onSettled: () => {
        // 无论成功失败，都重新获取最新数据
        queryClient.invalidateQueries({ queryKey: ['balance'] });
    },
});
```

---

## 第四部分：DeFi功能集成

### DEX交易组件

AppKit提供了现成的DEX交易组件：

```tsx
import { SwapWidget } from '@tonapp/react';

function SwapPage() {
    return (
        <SwapWidget
            defaultFromToken="TON"
            defaultToToken="USDT"
            onSwap={(tx) => console.log('交易完成:', tx)}
            slippage={0.5}  // 滑点容忍度 0.5%
        />
    );
}
```

这个组件包含了：
- 代币选择器
- 价格计算
- 滑点设置
- 交易确认
- 交易状态追踪

### 流动性管理

如果你要构建一个AMM（自动做市商）应用，AppKit也提供了流动性管理组件：

```tsx
import { LiquidityWidget } from '@tonapp/react';

function LiquidityPage() {
    return (
        <div>
            <h1>流动性池</h1>
            <LiquidityWidget
                poolAddress="EQD..."
                token0="TON"
                token1="USDT"
            />
        </div>
    );
}
```

### 质押功能

质押是DeFi的常见功能，AppKit同样提供了支持：

```tsx
import { useStaking } from '@tonapp/react';

function StakingComponent() {
    const { 
        stake, 
        unstake, 
        claimRewards,
        stakingInfo 
    } = useStaking('EQD...');  // 质押合约地址
    
    return (
        <div>
            <h2>质押池</h2>
            <p>已质押: {stakingInfo?.stakedAmount} TON</p>
            <p>待领取奖励: {stakingInfo?.pendingRewards} TOKEN</p>
            <p>APR: {stakingInfo?.apr}%</p>
            
            <button onClick={() => stake('100')}>
                质押 100 TON
            </button>
            <button onClick={() => unstake('50')}>
                解除质押 50 TON
            </button>
            <button onClick={claimRewards}>
                领取奖励
            </button>
        </div>
    );
}
```

### 价格图表

DeFi应用通常需要显示价格走势：

```tsx
import { PriceChart } from '@tonapp/react';

function TokenPage({ tokenAddress }: { tokenAddress: string }) {
    return (
        <div>
            <h1>代币详情</h1>
            <PriceChart
                tokenAddress={tokenAddress}
                timeframe="1D"  // 1天
                chartType="candlestick"
            />
        </div>
    );
}
```

### 自定义DeFi逻辑

如果AppKit提供的组件不能满足你的需求，你也可以基于它的基础Hooks构建自定义逻辑：

```tsx
import { useContract, useAccount } from '@tonapp/react';

function CustomYieldFarm() {
    const { address } = useAccount();
    const contract = useContract('EQD...');
    
    const deposit = async (amount: string) => {
        await contract.send({
            method: 'deposit',
            params: { amount },
            value: toNano('0.1'),  // gas费用
        });
    };
    
    const withdraw = async (amount: string) => {
        await contract.send({
            method: 'withdraw',
            params: { amount },
        });
    };
    
    // ...
}
```

---

## 总结

让我们回顾一下今天学到的内容。

首先，我们了解了AppKit是什么。它是一个专门为TON DApp开发设计的React组件库，提供了预构建的UI组件、区块链交互、数据管理和DeFi功能。

然后，我们学习了如何在React项目中集成AppKit。从安装配置到使用内置组件，再到自定义主题，我们覆盖了基础集成流程。

接着，我们深入探讨了数据查询，特别是与TanStack Query的集成。我们学习了基础查询、并行查询、依赖查询、数据修改和乐观更新等技术。

最后，我们了解了AppKit的DeFi功能，包括DEX交易、流动性管理、质押功能和价格图表。

## 下一章预告

在下一章，我们将进入一个非常激动人心的领域——Telegram Mini Apps开发。Mini Apps是运行在Telegram内部的Web应用，拥有超过8亿的潜在用户。

我们会学习Mini Apps的基础知识、开发环境的搭建、TON的集成，以及一个完整的游戏开发实战案例。想象一下，你开发的游戏可以直接在Telegram里玩，还能使用TON进行支付——这将是多么强大的组合！

好了，今天的播客就到这里。感谢收听，我们下一章再见！
