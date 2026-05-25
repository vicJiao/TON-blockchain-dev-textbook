# 第15章：Telegram Mini Apps开发 - 播客文字稿

## 开场白

大家好，欢迎收听TON区块链开发教材第15章的播客讲解。我是你们的主播。

想象一下，你开发了一个很棒的区块链应用，但发现用户获取成本极高——你需要做推广、买广告、优化SEO，还要和无数竞品争夺用户的注意力。这让人头疼，对吧？

现在，想象一下另一种场景：你的应用可以直接触达8亿用户，这些用户已经在使用一个超级App，你的应用就嵌在里面，用户无需下载、无需注册，点开就能用。听起来像梦想？这就是Telegram Mini Apps能为你做到的。

今天，我们将学习如何开发Telegram Mini Apps，并将其与TON区块链集成。这是本教材的最后一章，也是最具实战价值的一章。让我们开始吧！

---

## 第一部分：Mini Apps概述

### 什么是Telegram Mini Apps？

Telegram Mini Apps（以前叫Telegram Web Apps）是运行在Telegram内部的Web应用。你可以把它们想象成微信小程序的Telegram版本——轻量级、无需安装、即用即走。

用户可以在Telegram聊天界面中直接打开Mini App，无需跳转到外部浏览器或下载额外的应用。这种无缝的体验大大降低了用户的使用门槛。

### Mini Apps的优势

为什么要开发Mini Apps？让我列举几个核心优势：

**1. 庞大的用户基础**
- Telegram拥有超过8亿月活跃用户
- 你的应用可以直接触达这些用户

**2. 无缝的用户体验**
- 无需下载安装
- 无需单独注册账号（使用Telegram账号）
- 在聊天界面中直接打开

**3. 强大的传播能力**
- 可以轻松分享给好友和群组
- 支持机器人（Bot）集成
- 病毒式传播的潜力

**4. 与TON的深度集成**
- 原生支持TON钱包
- 内置支付功能
- 区块链游戏的最佳平台

### Mini Apps的应用场景

Mini Apps适合哪些类型的应用呢？

**游戏**：这是目前最流行的Mini Apps类型。从简单的点击游戏到复杂的策略游戏，都可以在Mini Apps中实现。

**电商**：用户可以在聊天中直接浏览商品、下单购买，使用TON支付。

**工具类**：投票、日程安排、任务管理等工具都很适合。

**金融服务**：钱包、DEX、质押平台等DeFi应用。

**社交娱乐**：表情包制作、头像生成器、小测试等。

### Mini Apps vs 传统Web App

Mini Apps和传统Web App有什么区别？

| 特性 | Mini Apps | 传统Web App |
|------|-----------|-------------|
| 入口 | Telegram内部 | 浏览器/独立App |
| 用户获取 | 通过Telegram生态 | 通过SEO/广告 |
| 身份认证 | Telegram自动提供 | 需要单独注册 |
| 支付 | TON原生支持 | 需要集成支付网关 |
| 分享传播 | 一键分享到聊天 | 需要复制链接 |

---

## 第二部分：开发基础

### 开发环境搭建

开发Mini Apps需要什么？其实非常简单：

**1. 基础工具**
- 一个文本编辑器（VS Code推荐）
- Node.js环境
- 一个Telegram账号

**2. 创建Bot**

首先，你需要创建一个Telegram Bot：

1. 在Telegram中搜索@BotFather
2. 发送`/newbot`命令
3. 按照提示设置Bot名称和用户名
4. 获得Bot Token（保存好，后面会用到）

**3. 设置Mini App**

在BotFather中设置Mini App：

```
/mybots -> 选择你的Bot -> Bot Settings -> Menu Button -> Configure menu button
```

设置菜单按钮的文本和URL，这个URL就是你的Mini App的地址。

### Mini Apps SDK

Telegram提供了JavaScript SDK，让你可以和Telegram客户端交互：

```html
<script src="https://telegram.org/js/telegram-web-app.js"></script>
```

在React项目中，你可以使用封装好的库：

```bash
npm install @vkruglikov/react-telegram-web-app
```

### 基础代码结构

让我们看一个简单的Mini App示例：

```tsx
import { useEffect } from 'react';
import { useTelegram } from '@vkruglikov/react-telegram-web-app';

function App() {
    const { 
        user,           // 用户信息
        ready,          // 是否准备就绪
        expand,         // 展开到全屏
        close,          // 关闭Mini App
        MainButton,     // 主按钮组件
        BackButton,     // 返回按钮组件
    } = useTelegram();

    useEffect(() => {
        // 通知Telegram客户端，Mini App已准备好
        ready();
        // 展开到全屏
        expand();
    }, [ready, expand]);

    return (
        <div className="app">
            <h1>你好, {user?.first_name}!</h1>
            <p>欢迎来到我的Mini App</p>
            
            <MainButton 
                text="确认" 
                onClick={() => {
                    // 处理确认逻辑
                    close();
                }}
            />
        </div>
    );
}
```

### 获取用户信息

Telegram会自动提供用户的基本信息：

```tsx
const { user } = useTelegram();

console.log(user);
// {
//     id: 123456789,
//     first_name: "张三",
//     last_name: "李四",
//     username: "zhangsan",
//     language_code: "zh",
//     is_premium: false
// }
```

注意：首次打开Mini App时，用户需要授权分享这些信息。

### 使用Telegram UI组件

Mini Apps提供了一套原生的UI组件，保持与Telegram一致的风格：

```tsx
import { 
    MainButton,
    BackButton,
    SettingsButton,
    HapticFeedback,
} from '@vkruglikov/react-telegram-web-app';

function MyComponent() {
    const { impactOccurred } = HapticFeedback();

    return (
        <div>
            {/* 主按钮 - 底部固定的主要操作按钮 */}
            <MainButton 
                text="提交订单"
                onClick={() => {
                    impactOccurred('light');  // 触觉反馈
                    submitOrder();
                }}
                progress={isLoading}  // 显示加载状态
            />
            
            {/* 返回按钮 - 左上角 */}
            <BackButton onClick={() => navigate(-1)} />
            
            {/* 设置按钮 */}
            <SettingsButton onClick={() => openSettings()} />
        </div>
    );
}
```

### 主题和样式

Mini Apps会自动适配Telegram的主题设置：

```tsx
const { colorScheme, themeParams } = useTelegram();

// colorScheme: 'light' | 'dark'
// themeParams: 包含各种颜色参数

console.log(themeParams);
// {
//     bg_color: '#ffffff',
//     text_color: '#000000',
//     hint_color: '#999999',
//     link_color: '#0088cc',
//     button_color: '#0088cc',
//     button_text_color: '#ffffff',
//     // ...
// }
```

你可以使用这些参数来保持与Telegram一致的视觉效果：

```css
.app {
    background-color: var(--tg-theme-bg-color);
    color: var(--tg-theme-text-color);
}

.button {
    background-color: var(--tg-theme-button-color);
    color: var(--tg-theme-button-text-color);
}
```

---

## 第三部分：TON集成

### 为什么在Mini Apps中使用TON？

Telegram和TON有着深厚的渊源。TON（The Open Network）最初就是由Telegram团队设计的。虽然现在TON是独立运营的区块链，但两者之间的集成非常紧密。

在Mini Apps中使用TON有天然的优势：
- 用户可能已经在Telegram中设置了TON钱包
- TON Connect协议在Telegram生态中得到了广泛支持
- 支付流程非常顺畅

### 集成TON Connect

在Mini Apps中集成TON Connect和普通的Web应用类似：

```tsx
import { TonConnectUIProvider } from '@tonconnect/ui-react';
import { useTelegram } from '@vkruglikov/react-telegram-web-app';

function App() {
    const { initData } = useTelegram();

    return (
        <TonConnectUIProvider
            manifestUrl="https://your-app.com/tonconnect-manifest.json"
            // 在Telegram中，可以使用特殊的返回URL
            actionsConfiguration={{
                returnStrategy: 'back',
            }}
        >
            <MiniApp />
        </TonConnectUIProvider>
    );
}
```

### 检测钱包支持

在Telegram环境中，用户可能使用不同的钱包：

```tsx
import { useTonConnectUI } from '@tonconnect/ui-react';

function WalletSection() {
    const [tonConnectUI] = useTonConnectUI();
    
    const isWalletConnected = tonConnectUI.connected;
    const walletInfo = tonConnectUI.wallet;
    
    // 检查是否是Telegram内置钱包
    const isTelegramWallet = walletInfo?.device.appName === 'TelegramWallet';
    
    return (
        <div>
            {isWalletConnected ? (
                <div>
                    <p>已连接: {walletInfo?.account.address}</p>
                    {isTelegramWallet && <span>Telegram钱包</span>}
                </div>
            ) : (
                <TonConnectButton />
            )}
        </div>
    );
}
```

### 发送支付

在Mini Apps中发起TON支付：

```tsx
import { useTonConnectUI } from '@tonconnect/ui-react';

function PaymentButton({ amount, itemName }: { amount: string; itemName: string }) {
    const [tonConnectUI] = useTonConnectUI();
    const { showPopup } = useTelegram();

    const handlePayment = async () => {
        try {
            const transaction = {
                validUntil: Math.floor(Date.now() / 1000) + 600,
                messages: [
                    {
                        address: 'YOUR_MERCHANT_ADDRESS',
                        amount: toNano(amount).toString(),
                        payload: beginCell()
                            .storeUint(0, 32)  // 操作码
                            .storeStringTail(itemName)  // 商品信息
                            .endCell()
                            .toBoc()
                            .toString('base64'),
                    }
                ]
            };

            const result = await tonConnectUI.sendTransaction(transaction);
            
            showPopup({
                title: '支付成功',
                message: `您已成功支付 ${amount} TON`,
            });
            
            return result;
        } catch (error) {
            showPopup({
                title: '支付失败',
                message: error.message,
            });
        }
    };

    return (
        <MainButton 
            text={`支付 ${amount} TON`}
            onClick={handlePayment}
        />
    );
}
```

### 使用Telegram Stars支付

除了TON，Telegram还推出了"Stars"——一种应用内虚拟货币：

```tsx
const { openInvoice } = useTelegram();

// 发起Stars支付
const buyWithStars = () => {
    openInvoice('https://t.me/$YOUR_BOT?start=invoice_payload', (status) => {
        if (status === 'paid') {
            console.log('Stars支付成功！');
        }
    });
};
```

---

## 第四部分：实战游戏开发

### 项目概述

让我们通过一个完整的游戏开发案例来巩固所学知识。我们要开发的游戏叫做"TON Clicker"——一个简单的点击赚钱游戏。

**游戏机制**：
- 玩家点击屏幕赚取金币
- 金币可以升级点击能力
- 玩家可以将金币兑换成TON代币
- 有排行榜显示顶级玩家

### 项目结构

```
ton-clicker/
├── src/
│   ├── components/
│   │   ├── ClickerButton.tsx    # 点击按钮
│   │   ├── UpgradeShop.tsx      # 升级商店
│   │   ├── Leaderboard.tsx      # 排行榜
│   │   └── WithdrawModal.tsx    # 提现弹窗
│   ├── hooks/
│   │   ├── useGameState.ts      # 游戏状态管理
│   │   ├── useTonPayment.ts     # TON支付
│   │   └── useTelegram.ts       # Telegram集成
│   ├── contracts/
│   │   └── clicker.fc           # FunC智能合约
│   ├── App.tsx
│   └── main.tsx
├── package.json
└── vite.config.ts
```

### 游戏状态管理

使用React Context管理游戏状态：

```tsx
// hooks/useGameState.ts
import { createContext, useContext, useState, useEffect } from 'react';

interface GameState {
    coins: number;
    clickPower: number;
    autoClickRate: number;
    totalClicks: number;
}

const GameContext = createContext<{
    state: GameState;
    click: () => void;
    upgrade: (type: 'click' | 'auto') => void;
} | null>(null);

export function GameProvider({ children }: { children: React.ReactNode }) {
    const [state, setState] = useState<GameState>({
        coins: 0,
        clickPower: 1,
        autoClickRate: 0,
        totalClicks: 0,
    });

    // 自动点击
    useEffect(() => {
        if (state.autoClickRate > 0) {
            const interval = setInterval(() => {
                setState(prev => ({
                    ...prev,
                    coins: prev.coins + prev.autoClickRate,
                }));
            }, 1000);
            return () => clearInterval(interval);
        }
    }, [state.autoClickRate]);

    const click = () => {
        setState(prev => ({
            ...prev,
            coins: prev.coins + prev.clickPower,
            totalClicks: prev.totalClicks + 1,
        }));
        
        // 触觉反馈
        window.Telegram.WebApp.HapticFeedback.impactOccurred('light');
    };

    const upgrade = (type: 'click' | 'auto') => {
        const cost = type === 'click' 
            ? state.clickPower * 100 
            : (state.autoClickRate + 1) * 500;

        if (state.coins >= cost) {
            setState(prev => ({
                ...prev,
                coins: prev.coins - cost,
                clickPower: type === 'click' 
                    ? prev.clickPower + 1 
                    : prev.clickPower,
                autoClickRate: type === 'auto' 
                    ? prev.autoClickRate + 1 
                    : prev.autoClickRate,
            }));
            
            window.Telegram.WebApp.HapticFeedback.notificationOccurred('success');
        }
    };

    return (
        <GameContext.Provider value={{ state, click, upgrade }}>
            {children}
        </GameContext.Provider>
    );
}

export const useGame = () => {
    const context = useContext(GameContext);
    if (!context) throw new Error('useGame must be used within GameProvider');
    return context;
};
```

### 点击按钮组件

```tsx
// components/ClickerButton.tsx
import { useGame } from '../hooks/useGameState';
import { useTelegram } from '@vkruglikov/react-telegram-web-app';

export function ClickerButton() {
    const { state, click } = useGame();
    const { impactOccurred } = useTelegram().HapticFeedback;

    const handleClick = () => {
        click();
        impactOccurred('medium');
    };

    return (
        <div className="clicker-container">
            <div className="stats">
                <h2>{Math.floor(state.coins).toLocaleString()} 金币</h2>
                <p>点击力量: {state.clickPower}</p>
                <p>自动收益: {state.autoClickRate}/秒</p>
            </div>
            
            <button 
                className="clicker-button"
                onClick={handleClick}
            >
                <span className="coin-icon">🪙</span>
                <span>+{state.clickPower}</span>
            </button>
            
            <p>总点击数: {state.totalClicks}</p>
        </div>
    );
}
```

### 升级商店

```tsx
// components/UpgradeShop.tsx
import { useGame } from '../hooks/useGameState';
import { MainButton } from '@vkruglikov/react-telegram-web-app';

export function UpgradeShop() {
    const { state, upgrade } = useGame();

    const clickUpgradeCost = state.clickPower * 100;
    const autoUpgradeCost = (state.autoClickRate + 1) * 500;

    return (
        <div className="upgrade-shop">
            <h3>升级商店</h3>
            
            <div className="upgrade-item">
                <div>
                    <h4>点击强化</h4>
                    <p>每次点击多获得1金币</p>
                    <p>价格: {clickUpgradeCost} 金币</p>
                </div>
                <MainButton
                    text="升级"
                    onClick={() => upgrade('click')}
                    disabled={state.coins < clickUpgradeCost}
                />
            </div>
            
            <div className="upgrade-item">
                <div>
                    <h4>自动点击器</h4>
                    <p>每秒自动获得1金币</p>
                    <p>价格: {autoUpgradeCost} 金币</p>
                </div>
                <MainButton
                    text="升级"
                    onClick={() => upgrade('auto')}
                    disabled={state.coins < autoUpgradeCost}
                />
            </div>
        </div>
    );
}
```

### TON提现功能

```tsx
// components/WithdrawModal.tsx
import { useState } from 'react';
import { useTonConnectUI } from '@tonconnect/ui-react';
import { useGame } from '../hooks/useGameState';
import { toNano, beginCell } from '@ton/core';

export function WithdrawModal({ onClose }: { onClose: () => void }) {
    const { state } = useGame();
    const [tonConnectUI] = useTonConnectUI();
    const [amount, setAmount] = useState('');

    // 假设1000金币 = 1 TON
    const tonAmount = parseFloat(amount) / 1000;

    const handleWithdraw = async () => {
        if (!tonConnectUI.connected) {
            alert('请先连接钱包');
            return;
        }

        try {
            const transaction = {
                validUntil: Math.floor(Date.now() / 1000) + 600,
                messages: [
                    {
                        address: 'GAME_CONTRACT_ADDRESS',
                        amount: toNano('0.05').toString(), // gas
                        payload: beginCell()
                            .storeUint(0x12345678, 32)  // 提现操作码
                            .storeCoins(BigInt(amount))
                            .endCell()
                            .toBoc()
                            .toString('base64'),
                    }
                ]
            };

            await tonConnectUI.sendTransaction(transaction);
            alert('提现请求已发送！');
            onClose();
        } catch (error) {
            alert('提现失败: ' + error.message);
        }
    };

    return (
        <div className="modal">
            <h3>提现到TON钱包</h3>
            <p>当前余额: {Math.floor(state.coins)} 金币</p>
            <p>汇率: 1000 金币 = 1 TON</p>
            
            <input
                type="number"
                value={amount}
                onChange={(e) => setAmount(e.target.value)}
                placeholder="输入金币数量"
            />
            
            {amount && (
                <p>将获得: {tonAmount.toFixed(4)} TON</p>
            )}
            
            <button onClick={handleWithdraw}>确认提现</button>
            <button onClick={onClose}>取消</button>
        </div>
    );
}
```

### 分享功能

利用Telegram的社交属性，添加分享功能：

```tsx
import { useTelegram } from '@vkruglikov/react-telegram-web-app';

function ShareButton({ score }: { score: number }) {
    const { shareURL } = useTelegram();

    const handleShare = () => {
        const text = `我在TON Clicker中赚了${score}金币！你能超过我吗？`;
        const url = `https://t.me/YOUR_BOT?start=ref_${userId}`;
        
        shareURL(url, text);
    };

    return (
        <button onClick={handleShare}>
            分享给好友
        </button>
    );
}
```

### 部署和测试

开发完成后，你需要：

1. **构建项目**：`npm run build`
2. **部署到服务器**：可以使用Vercel、Netlify等免费托管
3. **在BotFather中设置Mini App URL**
4. **在Telegram中测试**

测试时，你可以：
- 在桌面版Telegram中打开开发者工具（F12）
- 使用移动设备扫描二维码测试移动端体验
- 邀请朋友测试分享功能

---

## 总结

让我们回顾一下今天学到的内容，这也是本教材最后一章的总结。

首先，我们了解了Telegram Mini Apps是什么。它是运行在Telegram内部的Web应用，拥有8亿潜在用户，无需安装即可使用。

然后，我们学习了Mini Apps的开发基础。从创建Bot、设置开发环境，到使用Telegram SDK和UI组件，我们覆盖了完整的开发流程。

接着，我们深入探讨了TON在Mini Apps中的集成。TON和Telegram有着天然的亲和力，在Mini Apps中使用TON支付非常顺畅。

最后，我们通过一个完整的游戏开发案例——TON Clicker，将所学知识付诸实践。从游戏状态管理到TON提现，从升级商店到社交分享，我们构建了一个功能完整的区块链游戏。

## 结语

到这里，TON区块链开发教材的所有章节就讲完了。回顾一下我们的学习旅程：

- 我们了解了TON的基础架构和智能合约
- 学习了FunC语言和合约开发
- 掌握了Jetton代币标准和NFT开发
- 探索了DeFi协议和DEX实现
- 学会了前端集成和钱包连接
- 最后，我们学习了如何在Telegram Mini Apps中触达海量用户

区块链开发是一个快速发展的领域，TON生态也在不断创新。希望这本教材为你打下了坚实的基础，祝你在TON开发之路上越走越远！

感谢一路相伴，我们后会有期！
