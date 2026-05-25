# TON 区块链开发教材 - 实战篇

> 本文档包含三个完整的实战项目，从需求分析到部署上线，帮助开发者掌握 TON 开发的完整流程。

---

## 第六篇：实战篇 —— 综合项目

---

## 第 20 章：实战项目一 —— Jetton 代币发行平台

本章将带领你完成一个完整的 Jetton 代币发行平台，包括智能合约开发、前端界面和部署流程。

### 20.1 项目概述与架构设计

#### 20.1.1 项目需求

**功能需求：**
- 用户可以发行自定义 Jetton 代币
- 支持代币元数据（名称、符号、图标、描述）
- 支持代币铸造和销毁
- 提供代币管理界面
- 支持代币转账

**技术栈：**
- 智能合约：Tact 语言
- 开发框架：Blueprint
- 前端：React + TypeScript + TON Connect
- 部署：Vercel

#### 20.1.2 架构设计

```
┌─────────────────────────────────────────────────────────────┐
│                  Jetton 代币发行平台架构                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                   前端层                             │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  │   │
│  │  │ 发行页面     │  │ 管理页面     │  │ 转账页面     │  │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  │   │
│  │  ┌─────────────────────────────────────────────────┐  │   │
│  │  │           TON Connect 钱包连接                   │  │   │
│  │  └─────────────────────────────────────────────────┘  │   │
│  └─────────────────────────────────────────────────────┘   │
│                            │                                │
│                            ▼                                │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                   合约层                             │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  │   │
│  │  │ Jetton 主合约│  │ Jetton 钱包  │  │ 工厂合约     │  │   │
│  │  │  (Master)   │  │  (Wallet)   │  │  (Factory)  │  │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  │   │
│  └─────────────────────────────────────────────────────┘   │
│                            │                                │
│                            ▼                                │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                   存储层                             │   │
│  │  ┌─────────────┐  ┌─────────────┐                   │   │
│  │  │ 链上存储     │  │ IPFS 存储    │                   │   │
│  │  │ (合约数据)   │  │ (元数据/图标)│                   │   │
│  │  └─────────────┘  └─────────────┘                   │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 20.2 智能合约开发

#### 20.2.1 Jetton 主合约

```tact
// contracts/jetton_master.tact
import "@stdlib/deploy";
import "@stdlib/ownable";

// Jetton 标准消息
message Mint {
    amount: Int;
    receiver: Address;
}

message Burn {
    amount: Int;
}

message Transfer {
    to: Address;
    amount: Int;
}

// Jetton 主合约
contract JettonMaster with Deployable, Ownable {
    // 代币元数据
    name: String;
    symbol: String;
    decimals: Int;
    description: String?;
    image: String?;
    
    // 代币状态
    totalSupply: Int as coins = 0;
    maxSupply: Int as coins;
    mintable: Bool;
    burnable: Bool;
    
    // 管理
    owner: Address;
    walletCode: Cell;
    
    // 用户钱包地址映射
    wallets: map<Address, Address>;
    
    init(
        owner: Address,
        name: String,
        symbol: String,
        decimals: Int,
        maxSupply: Int,
        mintable: Bool,
        burnable: Bool,
        description: String?,
        image: String?,
        walletCode: Cell
    ) {
        self.owner = owner;
        self.name = name;
        self.symbol = symbol;
        self.decimals = decimals;
        self.maxSupply = maxSupply;
        self.mintable = mintable;
        self.burnable = burnable;
        self.description = description;
        self.image = image;
        self.walletCode = walletCode;
    }
    
    // 铸造代币
    receive(msg: Mint) {
        require(self.mintable, "Token not mintable");
        require(sender() == self.owner, "Only owner can mint");
        require(self.totalSupply + msg.amount <= self.maxSupply, "Exceeds max supply");
        
        // 更新总供应量
        self.totalSupply = self.totalSupply + msg.amount;
        
        // 获取或创建用户钱包
        let walletAddress: Address = self.getWalletAddress(msg.receiver);
        
        // 发送铸造消息到用户钱包
        send(SendParameters{
            to: walletAddress,
            value: ton("0.02"),
            mode: SendPayGasSeparately,
            body: InternalMint{
                amount: msg.amount,
                receiver: msg.receiver
            }.toCell()
        });
    }
    
    // 获取钱包地址
    fun getWalletAddress(owner: Address): Address {
        // 计算钱包地址
        let walletInit: StateInit = initOf JettonWallet(
            myAddress(),
            owner,
            self.walletCode
        );
        return contractAddress(walletInit);
    }
    
    // 发现钱包（用户首次使用时调用）
    receive(msg: DiscoverWallet) {
        let walletAddress: Address = self.getWalletAddress(msg.owner);
        self.wallets.set(msg.owner, walletAddress);
        
        // 发送确认消息
        send(SendParameters{
            to: sender(),
            value: 0,
            mode: SendRemainingValue,
            body: WalletDiscovered{
                owner: msg.owner,
                wallet: walletAddress
            }.toCell()
        });
    }
    
    // Getter 函数
    get fun getJettonData(): JettonData {
        return JettonData{
            totalSupply: self.totalSupply,
            mintable: self.mintable,
            owner: self.owner,
            content: self.buildContentCell(),
            walletCode: self.walletCode
        };
    }
    
    get fun getWalletAddressQuery(owner: Address): Address {
        return self.getWalletAddress(owner);
    }
    
    // 构建内容 Cell（符合 TEP-64）
    fun buildContentCell(): Cell {
        let content: Builder = beginCell();
        
        // 存储元数据
        content = content.storeString(self.name);
        content = content.storeString(self.symbol);
        content = content.storeInt(self.decimals, 8);
        
        if (self.description != null) {
            content = content.storeString(self.description!!);
        }
        
        if (self.image != null) {
            content = content.storeString(self.image!!);
        }
        
        return content.endCell();
    }
}

// 内部铸造消息
message InternalMint {
    amount: Int;
    receiver: Address;
}

message DiscoverWallet {
    owner: Address;
}

message WalletDiscovered {
    owner: Address;
    wallet: Address;
}

struct JettonData {
    totalSupply: Int as coins;
    mintable: Bool;
    owner: Address;
    content: Cell;
    walletCode: Cell;
}
```

#### 20.2.2 Jetton 钱包合约

```tact
// contracts/jetton_wallet.tact
import "@stdlib/deploy";

// Jetton 钱包合约
contract JettonWallet {
    master: Address;      // 主合约地址
    owner: Address;      // 钱包所有者
    balance: Int as coins = 0;
    
    init(master: Address, owner: Address, walletCode: Cell) {
        self.master = master;
        self.owner = owner;
    }
    
    // 接收铸造的代币
    receive(msg: InternalMint) {
        require(sender() == self.master, "Only master can mint");
        self.balance = self.balance + msg.amount;
    }
    
    // 接收转账
    receive(msg: JettonTransfer) {
        require(sender() == self.owner, "Only owner can transfer");
        require(self.balance >= msg.amount, "Insufficient balance");
        require(msg.amount > 0, "Amount must be positive");
        
        // 扣除余额
        self.balance = self.balance - msg.amount;
        
        // 获取接收者钱包地址
        let recipientWallet: Address = self.getWalletAddress(msg.to);
        
        // 发送代币到接收者
        send(SendParameters{
            to: recipientWallet,
            value: msg.forwardTonAmount,
            mode: SendPayGasSeparately,
            body: JettonInternalTransfer{
                amount: msg.amount,
                from: self.owner,
                responseAddress: msg.responseAddress,
                forwardTonAmount: msg.forwardTonAmount,
                forwardPayload: msg.forwardPayload
            }.toCell()
        });
        
        // 发送通知给发送者
        if (msg.responseAddress != null) {
            send(SendParameters{
                to: msg.responseAddress!!,
                value: ton("0.01"),
                mode: SendPayGasSeparately,
                body: JettonTransferNotification{
                    amount: msg.amount,
                    to: msg.to
                }.toCell()
            });
        }
    }
    
    // 接收内部转账
    receive(msg: JettonInternalTransfer) {
        // 验证发送者是另一个 Jetton 钱包
        let senderWallet: Address = self.getWalletAddress(msg.from);
        require(sender() == senderWallet, "Invalid sender");
        
        // 增加余额
        self.balance = self.balance + msg.amount;
        
        // 转发消息给所有者
        if (msg.forwardTonAmount > 0) {
            send(SendParameters{
                to: self.owner,
                value: msg.forwardTonAmount,
                mode: SendPayGasSeparately,
                body: msg.forwardPayload
            });
        }
        
        // 发送接收通知
        if (msg.responseAddress != null) {
            send(SendParameters{
                to: msg.responseAddress!!,
                value: ton("0.01"),
                mode: SendPayGasSeparately,
                body: JettonReceipt{
                    amount: msg.amount,
                    from: msg.from
                }.toCell()
            });
        }
    }
    
    // 销毁代币
    receive(msg: Burn) {
        require(sender() == self.owner, "Only owner can burn");
        require(self.balance >= msg.amount, "Insufficient balance");
        
        self.balance = self.balance - msg.amount;
        
        // 通知主合约销毁
        send(SendParameters{
            to: self.master,
            value: ton("0.01"),
            mode: SendPayGasSeparately,
            body: JettonBurnNotification{
                amount: msg.amount,
                owner: self.owner
            }.toCell()
        });
    }
    
    // 获取钱包地址
    fun getWalletAddress(owner: Address): Address {
        let walletInit: StateInit = initOf JettonWallet(
            self.master,
            owner,
            emptyCell()  // 简化，实际需要 walletCode
        );
        return contractAddress(walletInit);
    }
    
    // Getter 函数
    get fun getWalletData(): WalletData {
        return WalletData{
            balance: self.balance,
            owner: self.owner,
            master: self.master,
            walletCode: emptyCell()
        };
    }
    
    get fun getBalance(): Int {
        return self.balance;
    }
}

// 转账消息
message JettonTransfer {
    to: Address;
    amount: Int as coins;
    responseAddress: Address?;
    forwardTonAmount: Int as coins = 0;
    forwardPayload: Cell? = null;
}

// 内部转账消息
message JettonInternalTransfer {
    amount: Int as coins;
    from: Address;
    responseAddress: Address?;
    forwardTonAmount: Int as coins;
    forwardPayload: Cell?;
}

// 转账通知
message JettonTransferNotification {
    amount: Int as coins;
    to: Address;
}

// 接收确认
message JettonReceipt {
    amount: Int as coins;
    from: Address;
}

// 销毁通知
message JettonBurnNotification {
    amount: Int as coins;
    owner: Address;
}

struct WalletData {
    balance: Int as coins;
    owner: Address;
    master: Address;
    walletCode: Cell;
}
```

#### 20.2.3 工厂合约

```tact
// contracts/token_factory.tact
import "@stdlib/deploy";
import "@stdlib/ownable";

// 代币创建参数
message CreateToken {
    name: String;
    symbol: String;
    decimals: Int;
    maxSupply: Int;
    mintable: Bool;
    burnable: Bool;
    description: String?;
    image: String?;
}

// 工厂合约
contract TokenFactory with Deployable, Ownable {
    owner: Address;
    
    // 创建费用
    creationFee: Int as coins;
    
    // 已创建的代币列表
    tokens: map<Int, Address>;
    tokenCount: Int = 0;
    
    // 用户创建的代币
    userTokens: map<Address, map<Int, Address>>;
    
    // Jetton 钱包代码
    walletCode: Cell;
    
    init(owner: Address, creationFee: Int, walletCode: Cell) {
        self.owner = owner;
        self.creationFee = creationFee;
        self.walletCode = walletCode;
    }
    
    // 创建新代币
    receive(msg: CreateToken) {
        // 验证支付
        require(context().value >= self.creationFee, "Insufficient creation fee");
        
        // 验证参数
        require(msg.decimals <= 18, "Decimals too high");
        require(msg.maxSupply > 0, "Max supply must be positive");
        
        let creator: Address = sender();
        
        // 部署 Jetton 主合约
        let tokenInit: StateInit = initOf JettonMaster(
            creator,  // 创建者作为所有者
            msg.name,
            msg.symbol,
            msg.decimals,
            msg.maxSupply,
            msg.mintable,
            msg.burnable,
            msg.description,
            msg.image,
            self.walletCode
        );
        
        let tokenAddress: Address = contractAddress(tokenInit);
        
        // 记录代币
        self.tokens.set(self.tokenCount, tokenAddress);
        self.tokenCount = self.tokenCount + 1;
        
        // 记录到用户代币列表
        let userTokenList: map<Int, Address> = self.userTokens.get(creator) ?: emptyMap();
        let userTokenCount: Int = mapSize(userTokenList);
        userTokenList.set(userTokenCount, tokenAddress);
        self.userTokens.set(creator, userTokenList);
        
        // 发送部署资金
        send(SendParameters{
            to: tokenAddress,
            value: ton("0.1"),
            mode: SendPayGasSeparately,
            code: tokenInit.code,
            data: tokenInit.data
        });
        
        // 发送确认
        send(SendParameters{
            to: creator,
            value: 0,
            mode: SendRemainingValue,
            body: TokenCreated{
                tokenAddress: tokenAddress,
                name: msg.name,
                symbol: msg.symbol
            }.toCell()
        });
    }
    
    // 更新创建费用
    receive(msg: UpdateCreationFee) {
        require(sender() == self.owner, "Only owner");
        self.creationFee = msg.newFee;
    }
    
    // 提取费用
    receive(msg: WithdrawFees) {
        require(sender() == self.owner, "Only owner");
        send(SendParameters{
            to: self.owner,
            value: myBalance() - ton("0.1"),
            mode: SendPayGasSeparately
        });
    }
    
    // Getter 函数
    get fun getTokenCount(): Int {
        return self.tokenCount;
    }
    
    get fun getToken(index: Int): Address? {
        return self.tokens.get(index);
    }
    
    get fun getUserTokens(user: Address): map<Int, Address>? {
        return self.userTokens.get(user);
    }
    
    get fun getCreationFee(): Int {
        return self.creationFee;
    }
}

message TokenCreated {
    tokenAddress: Address;
    name: String;
    symbol: String;
}

message UpdateCreationFee {
    newFee: Int as coins;
}

message WithdrawFees {}
```

### 20.3 前端开发

#### 20.3.1 项目初始化

```bash
# 创建前端项目
npx create-react-app jetton-platform --template typescript
cd jetton-platform

# 安装依赖
npm install @tonconnect/ui-react @ton/core @ton/ton
npm install @ton/blueprint @ton-community/sandbox --save-dev

# 安装 UI 库
npm install @mui/material @emotion/react @emotion/styled
npm install react-router-dom axios
```

#### 20.3.2 TON Connect 配置

```tsx
// src/App.tsx
import { TonConnectUIProvider } from '@tonconnect/ui-react';
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import { ThemeProvider, createTheme } from '@mui/material';
import { Home } from './pages/Home';
import { CreateToken } from './pages/CreateToken';
import { ManageToken } from './pages/ManageToken';
import { Navbar } from './components/Navbar';

const theme = createTheme({
  palette: {
    primary: {
      main: '#0088CC',  // TON 蓝色
    },
  },
});

function App() {
  return (
    <TonConnectUIProvider 
      manifestUrl="https://your-app.com/tonconnect-manifest.json"
    >
      <ThemeProvider theme={theme}>
        <Router>
          <Navbar />
          <Routes>
            <Route path="/" element={<Home />} />
            <Route path="/create" element={<CreateToken />} />
            <Route path="/manage/:tokenAddress" element={<ManageToken />} />
          </Routes>
        </Router>
      </ThemeProvider>
    </TonConnectUIProvider>
  );
}

export default App;
```

#### 20.3.3 代币创建页面

```tsx
// src/pages/CreateToken.tsx
import { useState } from 'react';
import { useTonConnectUI, useTonAddress } from '@tonconnect/ui-react';
import { Address, toNano } from '@ton/core';
import {
  Container,
  Typography,
  TextField,
  Button,
  Box,
  Switch,
  FormControlLabel,
  Alert,
  CircularProgress,
} from '@mui/material';
import { TokenFactory } from '../contracts/TokenFactory';

export function CreateToken() {
  const [tonConnectUI] = useTonConnectUI();
  const userAddress = useTonAddress();
  
  const [formData, setFormData] = useState({
    name: '',
    symbol: '',
    decimals: 9,
    maxSupply: '',
    mintable: true,
    burnable: true,
    description: '',
  });
  
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState('');
  const [success, setSuccess] = useState('');

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    if (!userAddress) {
      setError('Please connect your wallet first');
      return;
    }
    
    setLoading(true);
    setError('');
    setSuccess('');
    
    try {
      // 构建创建代币消息
      const createTokenMsg = {
        name: formData.name,
        symbol: formData.symbol,
        decimals: formData.decimals,
        maxSupply: toNano(formData.maxSupply),
        mintable: formData.mintable,
        burnable: formData.burnable,
        description: formData.description || null,
        image: null,
      };
      
      // 发送交易
      await tonConnectUI.sendTransaction({
        messages: [
          {
            address: process.env.REACT_APP_FACTORY_ADDRESS!,
            amount: toNano('0.5').toString(), // 创建费用
            payload: buildCreateTokenPayload(createTokenMsg),
          },
        ],
        validUntil: Date.now() + 5 * 60 * 1000,
      });
      
      setSuccess('Token creation transaction sent! Please wait for confirmation.');
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Failed to create token');
    } finally {
      setLoading(false);
    }
  };

  return (
    <Container maxWidth="md" sx={{ mt: 4 }}>
      <Typography variant="h4" gutterBottom>
        Create New Token
      </Typography>
      
      {error && <Alert severity="error" sx={{ mb: 2 }}>{error}</Alert>}
      {success && <Alert severity="success" sx={{ mb: 2 }}>{success}</Alert>}
      
      <Box component="form" onSubmit={handleSubmit}>
        <TextField
          fullWidth
          label="Token Name"
          value={formData.name}
          onChange={(e) => setFormData({ ...formData, name: e.target.value })}
          margin="normal"
          required
        />
        
        <TextField
          fullWidth
          label="Token Symbol"
          value={formData.symbol}
          onChange={(e) => setFormData({ ...formData, symbol: e.target.value })}
          margin="normal"
          required
          helperText="e.g., BTC, ETH, TON"
        />
        
        <TextField
          fullWidth
          label="Decimals"
          type="number"
          value={formData.decimals}
          onChange={(e) => setFormData({ ...formData, decimals: parseInt(e.target.value) })}
          margin="normal"
          inputProps={{ min: 0, max: 18 }}
        />
        
        <TextField
          fullWidth
          label="Max Supply"
          value={formData.maxSupply}
          onChange={(e) => setFormData({ ...formData, maxSupply: e.target.value })}
          margin="normal"
          required
          helperText="Maximum number of tokens that can be minted"
        />
        
        <TextField
          fullWidth
          label="Description"
          multiline
          rows={3}
          value={formData.description}
          onChange={(e) => setFormData({ ...formData, description: e.target.value })}
          margin="normal"
        />
        
        <Box sx={{ mt: 2 }}>
          <FormControlLabel
            control={
              <Switch
                checked={formData.mintable}
                onChange={(e) => setFormData({ ...formData, mintable: e.target.checked })}
              />
            }
            label="Mintable (allow future minting)"
          />
        </Box>
        
        <Box sx={{ mt: 1 }}>
          <FormControlLabel
            control={
              <Switch
                checked={formData.burnable}
                onChange={(e) => setFormData({ ...formData, burnable: e.target.checked })}
              />
            }
            label="Burnable (allow token burning)"
          />
        </Box>
        
        <Button
          type="submit"
          variant="contained"
          size="large"
          disabled={loading || !userAddress}
          sx={{ mt: 3 }}
        >
          {loading ? <CircularProgress size={24} /> : 'Create Token (0.5 TON)'}
        </Button>
      </Box>
    </Container>
  );
}

// 辅助函数：构建创建代币的 payload
function buildCreateTokenPayload(msg: any): string {
  // 使用 @ton/core 构建 Cell
  // 这里简化处理，实际需要根据合约 ABI 编码
  return '';
}
```

### 20.4 测试、部署与上线

#### 20.4.1 合约测试

```typescript
// tests/TokenFactory.spec.ts
import { Blockchain } from '@ton-community/sandbox';
import { TokenFactory } from '../build/TokenFactory';
import { toNano } from '@ton/core';

describe('TokenFactory', () => {
  let blockchain: Blockchain;
  let factory: TokenFactory;
  let deployer: any;

  beforeEach(async () => {
    blockchain = await Blockchain.create();
    deployer = await blockchain.treasury('deployer');
    
    factory = blockchain.openContract(
      await TokenFactory.fromInit(deployer.address, toNano('0.5'), emptyCell())
    );
    
    await factory.send(deployer.getSender(), { value: toNano('0.1') }, 'Deploy');
  });

  it('should create token', async () => {
    const result = await factory.send(
      deployer.getSender(),
      { value: toNano('0.5') },
      {
        $$type: 'CreateToken',
        name: 'Test Token',
        symbol: 'TEST',
        decimals: 9n,
        maxSupply: 1000000n,
        mintable: true,
        burnable: true,
        description: 'Test description',
        image: null,
      }
    );

    expect(result.transactions).toHaveTransaction({
      from: factory.address,
      success: true,
    });

    const tokenCount = await factory.getTokenCount();
    expect(tokenCount).toBe(1n);
  });
});
```

#### 20.4.2 部署脚本

```typescript
// scripts/deploy.ts
import { toNano } from '@ton/core';
import { TokenFactory } from '../build/TokenFactory';
import { NetworkProvider } from '@ton/blueprint';

export async function run(provider: NetworkProvider) {
  const walletCode = await compile('JettonWallet');
  
  const tokenFactory = provider.open(
    await TokenFactory.fromInit(
      provider.sender().address!,
      toNano('0.5'),  // 创建费用
      walletCode
    )
  );

  await tokenFactory.send(
    provider.sender(),
    { value: toNano('0.1') },
    { $$type: 'Deploy' }
  );

  await provider.waitForDeploy(tokenFactory.address);

  console.log('TokenFactory deployed at:', tokenFactory.address.toString());
}
```

---

**项目一总结：**

通过这个项目，你学会了：
- 实现符合 TEP-74 标准的 Jetton 代币合约
- 构建代币工厂合约
- 开发完整的前端界面
- 测试和部署智能合约

---

## 第 21 章：实战项目二 —— NFT 市场

本章将开发一个完整的 NFT 市场，支持 NFT 的铸造、交易和版税分配。

### 21.1 项目概述与架构设计

#### 21.1.1 功能需求

- NFT 集合创建和管理
- NFT 铸造（支持限量和无限量）
- NFT 挂单和购买
- 版税自动分配（TEP-66）
- 支持多种支付方式

#### 21.1.2 架构设计

```
┌─────────────────────────────────────────────────────────────┐
│                     NFT 市场架构                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │ NFT 集合合约 │  │ NFT Item 合约│  │ 市场合约     │         │
│  │ (Collection)│  │   (Item)    │  │ (Marketplace)│         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │ 版税合约     │  │ 支付合约     │  │ 工厂合约     │         │
│  │ (Royalty)   │  │  (Payment)  │  │  (Factory)  │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 21.2 智能合约开发

#### 21.2.1 NFT 集合合约

```tact
// contracts/nft_collection.tact
import "@stdlib/deploy";
import "@stdlib/ownable";

// NFT 集合合约
contract NFTCollection with Deployable, Ownable {
    // 集合信息
    name: String;
    description: String?;
    image: String?;
    
    // 铸造参数
    maxSupply: Int;
    mintPrice: Int as coins;
    mintable: Bool = true;
    
    // 状态
    nextItemIndex: Int = 0;
    owner: Address;
    
    // 版税信息
    royaltyNumerator: Int;
    royaltyDenominator: Int;
    royaltyDestination: Address;
    
    // NFT Item 代码
    itemCode: Cell;
    
    init(
        owner: Address,
        name: String,
        description: String?,
        image: String?,
        maxSupply: Int,
        mintPrice: Int,
        royaltyNumerator: Int,
        royaltyDenominator: Int,
        royaltyDestination: Address,
        itemCode: Cell
    ) {
        self.owner = owner;
        self.name = name;
        self.description = description;
        self.image = image;
        self.maxSupply = maxSupply;
        self.mintPrice = mintPrice;
        self.royaltyNumerator = royaltyNumerator;
        self.royaltyDenominator = royaltyDenominator;
        self.royaltyDestination = royaltyDestination;
        self.itemCode = itemCode;
    }
    
    // 铸造 NFT
    receive(msg: MintNFT) {
        require(self.mintable, "Minting is disabled");
        require(self.nextItemIndex < self.maxSupply, "Max supply reached");
        require(context().value >= self.mintPrice, "Insufficient payment");
        
        let itemIndex: Int = self.nextItemIndex;
        self.nextItemIndex = self.nextItemIndex + 1;
        
        // 计算 NFT 地址
        let itemAddress: Address = self.getNFTAddress(itemIndex);
        
        // 部署 NFT Item
        let itemInit: StateInit = StateInit{
            code: self.itemCode,
            data: beginCell()
                .storeAddress(myAddress())
                .storeInt(itemIndex, 64)
                .storeAddress(sender())
                .storeRef(msg.content)
                .endCell()
        };
        
        // 发送部署消息
        send(SendParameters{
            to: itemAddress,
            value: ton("0.05"),
            mode: SendPayGasSeparately,
            code: itemInit.code,
            data: itemInit.data
        });
        
        // 发送多余资金给铸造者
        let excess: Int = context().value - self.mintPrice;
        if (excess > ton("0.01")) {
            send(SendParameters{
                to: sender(),
                value: excess,
                mode: SendPayGasSeparately
            });
        }
    }
    
    // 批量铸造
    receive(msg: BatchMint) {
        require(sender() == self.owner, "Only owner can batch mint");
        require(self.nextItemIndex + msg.count <= self.maxSupply, "Exceeds max supply");
        
        // 简化示例，实际应该分批处理
        // ...
    }
    
    // 获取 NFT 地址
    fun getNFTAddress(index: Int): Address {
        let init: StateInit = StateInit{
            code: self.itemCode,
            data: beginCell()
                .storeAddress(myAddress())
                .storeInt(index, 64)
                .storeAddress(newAddress(0, 0))  // 占位
                .storeRef(emptyCell())
                .endCell()
        };
        return contractAddress(init);
    }
    
    // 关闭铸造
    receive(msg: DisableMinting) {
        require(sender() == self.owner, "Only owner");
        self.mintable = false;
    }
    
    // Getter 函数
    get fun getCollectionData(): CollectionData {
        return CollectionData{
            nextItemIndex: self.nextItemIndex,
            collectionContent: self.buildContentCell(),
            owner: self.owner
        };
    }
    
    get fun getNFTAddressQuery(index: Int): Address {
        return self.getNFTAddress(index);
    }
    
    get fun getRoyaltyParams(): RoyaltyParams {
        return RoyaltyParams{
            numerator: self.royaltyNumerator,
            denominator: self.royaltyDenominator,
            destination: self.royaltyDestination
        };
    }
    
    fun buildContentCell(): Cell {
        return beginCell()
            .storeString(self.name)
            .storeString(self.description ?: "")
            .storeString(self.image ?: "")
            .endCell();
    }
}

// 铸造消息
message MintNFT {
    content: Cell;  // NFT 元数据
}

message BatchMint {
    count: Int;
    contents: map<Int, Cell>;
}

message DisableMinting {}

struct CollectionData {
    nextItemIndex: Int;
    collectionContent: Cell;
    owner: Address;
}

struct RoyaltyParams {
    numerator: Int;
    denominator: Int;
    destination: Address;
}
```

#### 21.2.2 NFT Item 合约

```tact
// contracts/nft_item.tact
import "@stdlib/deploy";

// NFT Item 合约
contract NFTItem {
    collection: Address;
    index: Int;
    owner: Address;
    content: Cell;
    
    // 市场信息
    isListed: Bool = false;
    listPrice: Int as coins = 0;
    marketplace: Address? = null;
    
    init(collection: Address, index: Int, owner: Address, content: Cell) {
        self.collection = collection;
        self.index = index;
        self.owner = owner;
        self.content = content;
    }
    
    // 转移 NFT
    receive(msg: Transfer) {
        require(sender() == self.owner, "Only owner can transfer");
        require(!self.isListed, "NFT is listed, cancel listing first");
        
        let oldOwner: Address = self.owner;
        self.owner = msg.newOwner;
        
        // 发送确认给新所有者
        if (msg.responseDestination != null) {
            send(SendParameters{
                to: msg.responseDestination!!,
                value: ton("0.01"),
                mode: SendPayGasSeparately,
                body: OwnershipAssigned{
                    queryId: msg.queryId,
                    prevOwner: oldOwner
                }.toCell()
            });
        }
        
        // 转发 payload
        if (msg.forwardAmount > 0) {
            send(SendParameters{
                to: msg.newOwner,
                value: msg.forwardAmount,
                mode: SendPayGasSeparately,
                body: msg.forwardPayload
            });
        }
    }
    
    // 挂单
    receive(msg: ListForSale) {
        require(sender() == self.owner, "Only owner can list");
        require(!self.isListed, "Already listed");
        require(msg.price > 0, "Price must be positive");
        
        self.isListed = true;
        self.listPrice = msg.price;
        self.marketplace = sender();
    }
    
    // 取消挂单
    receive(msg: CancelListing) {
        require(sender() == self.owner || sender() == self.marketplace, "Unauthorized");
        require(self.isListed, "Not listed");
        
        self.isListed = false;
        self.listPrice = 0;
        self.marketplace = null;
    }
    
    // 购买
    receive(msg: BuyNFT) {
        require(self.isListed, "Not for sale");
        require(context().value >= self.listPrice, "Insufficient payment");
        
        let buyer: Address = sender();
        let seller: Address = self.owner;
        let price: Int = self.listPrice;
        
        // 获取版税信息
        let royalty: RoyaltyParams = self.getRoyaltyParams();
        let royaltyAmount: Int = (price * royalty.numerator) / royalty.denominator;
        let sellerAmount: Int = price - royaltyAmount;
        
        // 转移所有权
        self.owner = buyer;
        self.isListed = false;
        self.listPrice = 0;
        self.marketplace = null;
        
        // 支付卖家
        send(SendParameters{
            to: seller,
            value: sellerAmount,
            mode: SendPayGasSeparately
        });
        
        // 支付版税
        if (royaltyAmount > 0) {
            send(SendParameters{
                to: royalty.destination,
                value: royaltyAmount,
                mode: SendPayGasSeparately
            });
        }
        
        // 通知买家
        send(SendParameters{
            to: buyer,
            value: ton("0.01"),
            mode: SendPayGasSeparately,
            body: NFTPurchased{
                nft: myAddress(),
                price: price,
                seller: seller
            }.toCell()
        });
    }
    
    // 获取版税参数
    fun getRoyaltyParams(): RoyaltyParams {
        // 调用集合合约获取版税参数
        // 简化示例
        return RoyaltyParams{
            numerator: 5,  // 5%
            denominator: 100,
            destination: self.collection
        };
    }
    
    // Getter 函数
    get fun getNFTData(): NFTData {
        return NFTData{
            isInitialized: true,
            index: self.index,
            collection: self.collection,
            owner: self.owner,
            content: self.content
        };
    }
    
    get fun getListingInfo(): ListingInfo {
        return ListingInfo{
            isListed: self.isListed,
            price: self.listPrice,
            marketplace: self.marketplace
        };
    }
}

// 转移消息
message Transfer {
    queryId: Int;
    newOwner: Address;
    responseDestination: Address?;
    customPayload: Cell?;
    forwardAmount: Int as coins;
    forwardPayload: Cell?;
}

message OwnershipAssigned {
    queryId: Int;
    prevOwner: Address;
}

message ListForSale {
    price: Int as coins;
}

message CancelListing {}

message BuyNFT {}

message NFTPurchased {
    nft: Address;
    price: Int as coins;
    seller: Address;
}

struct NFTData {
    isInitialized: Bool;
    index: Int;
    collection: Address;
    owner: Address;
    content: Cell;
}

struct ListingInfo {
    isListed: Bool;
    price: Int as coins;
    marketplace: Address?;
}

struct RoyaltyParams {
    numerator: Int;
    denominator: Int;
    destination: Address;
}
```

### 21.3 前端开发

#### 21.3.1 NFT 展示组件

```tsx
// src/components/NFTCard.tsx
import { Card, CardMedia, CardContent, Typography, Button, Box } from '@mui/material';
import { Address } from '@ton/core';

interface NFTCardProps {
  nft: {
    address: string;
    index: number;
    name: string;
    image: string;
    owner: string;
    isListed: boolean;
    price?: string;
  };
  onBuy?: () => void;
  onList?: () => void;
}

export function NFTCard({ nft, onBuy, onList }: NFTCardProps) {
  return (
    <Card sx={{ maxWidth: 345, m: 1 }}>
      <CardMedia
        component="img"
        height="200"
        image={nft.image || '/placeholder-nft.png'}
        alt={nft.name}
      />
      <CardContent>
        <Typography gutterBottom variant="h6" component="div">
          {nft.name}
        </Typography>
        <Typography variant="body2" color="text.secondary">
          #{nft.index}
        </Typography>
        <Typography variant="body2" color="text.secondary">
          Owner: {nft.owner.slice(0, 6)}...{nft.owner.slice(-4)}
        </Typography>
        
        {nft.isListed && (
          <Box sx={{ mt: 1 }}>
            <Typography variant="h6" color="primary">
              {nft.price} TON
            </Typography>
          </Box>
        )}
        
        <Box sx={{ mt: 2 }}>
          {nft.isListed && onBuy && (
            <Button variant="contained" fullWidth onClick={onBuy}>
              Buy Now
            </Button>
          )}
          {!nft.isListed && onList && (
            <Button variant="outlined" fullWidth onClick={onList}>
              List for Sale
            </Button>
          )}
        </Box>
      </CardContent>
    </Card>
  );
}
```

---

**项目二总结：**

通过 NFT 市场项目，你学会了：
- 实现符合 TEP-62 标准的 NFT 合约
- 实现版税机制（TEP-66）
- 构建 NFT 交易功能
- 开发 NFT 市场前端

---

## 第 22 章：实战项目三 —— DeFi 质押协议

本章将开发一个完整的 DeFi 质押协议，支持 TON 质押、奖励分配和治理功能。

### 22.1 项目概述与架构设计

#### 22.1.1 功能需求

- TON 代币质押
- 动态 APY 计算
- 奖励自动复利
- 治理代币分发
- 紧急暂停机制

#### 22.1.2 架构设计

```
┌─────────────────────────────────────────────────────────────┐
│                   DeFi 质押协议架构                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │ 质押池合约   │  │ 奖励合约     │  │ 治理合约     │         │
│  │ (Staking)   │  │  (Rewards)  │  │ (Governance)│         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │ 策略合约     │  │ 保险合约     │  │ 价格预言机   │         │
│  │ (Strategy)  │  │  (Insurance)│  │   (Oracle)  │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 22.2 智能合约开发

#### 22.2.1 质押池合约

```tact
// contracts/staking_pool.tact
import "@stdlib/deploy";
import "@stdlib/ownable";
import "@stdlib/pausable";

// 质押池合约
contract StakingPool with Deployable, Ownable, Pausable {
    // 质押代币
    stakingToken: Address;
    
    // 奖励参数
    rewardPerSecond: Int;
    lastUpdateTime: Int;
    accRewardPerShare: Int;  // 累计每份额奖励（放大 1e12 倍）
    
    // 总质押量
    totalStaked: Int as coins = 0;
    
    // 用户质押信息
    userInfo: map<Address, UserInfo>;
    
    // 管理
    owner: Address;
    paused: Bool = false;
    
    // 紧急提款费用（百分比，如 100 = 1%）
    emergencyWithdrawFee: Int = 100;
    feeRecipient: Address;
    
    init(stakingToken: Address, rewardPerSecond: Int, feeRecipient: Address) {
        self.owner = sender();
        self.stakingToken = stakingToken;
        self.rewardPerSecond = rewardPerSecond;
        self.lastUpdateTime = now();
        self.feeRecipient = feeRecipient;
    }
    
    // 更新奖励参数
    fun updatePool() {
        if (now() <= self.lastUpdateTime) {
            return;
        }
        
        if (self.totalStaked == 0) {
            self.lastUpdateTime = now();
            return;
        }
        
        let multiplier: Int = now() - self.lastUpdateTime;
        let reward: Int = multiplier * self.rewardPerSecond;
        self.accRewardPerShare = self.accRewardPerShare + (reward * 1e12) / self.totalStaked;
        self.lastUpdateTime = now();
    }
    
    // 计算待领取奖励
    fun pendingReward(user: Address): Int {
        let info: UserInfo = self.userInfo.get(user) ?: UserInfo{
            amount: 0,
            rewardDebt: 0
        };
        
        let accReward: Int = self.accRewardPerShare;
        if (now() > self.lastUpdateTime && self.totalStaked != 0) {
            let multiplier: Int = now() - self.lastUpdateTime;
            let reward: Int = multiplier * self.rewardPerSecond;
            accReward = accReward + (reward * 1e12) / self.totalStaked;
        }
        
        return (info.amount * accReward) / 1e12 - info.rewardDebt;
    }
    
    // 质押
    receive(msg: Deposit) {
        self.whenNotPaused();
        self.updatePool();
        
        let sender: Address = sender();
        let info: UserInfo = self.userInfo.get(sender) ?: UserInfo{
            amount: 0,
            rewardDebt: 0
        };
        
        // 领取之前的奖励
        if (info.amount > 0) {
            let pending: Int = (info.amount * self.accRewardPerShare) / 1e12 - info.rewardDebt;
            if (pending > 0) {
                self.sendReward(sender, pending);
            }
        }
        
        // 更新质押量
        info.amount = info.amount + msg.amount;
        info.rewardDebt = (info.amount * self.accRewardPerShare) / 1e12;
        self.userInfo.set(sender, info);
        
        self.totalStaked = self.totalStaked + msg.amount;
        
        // 实际实现需要转移质押代币
    }
    
    // 提款
    receive(msg: Withdraw) {
        self.updatePool();
        
        let sender: Address = sender();
        let info: UserInfo = self.userInfo.get(sender)!!;
        require(info.amount >= msg.amount, "Insufficient staked amount");
        
        // 计算并发送奖励
        let pending: Int = (info.amount * self.accRewardPerShare) / 1e12 - info.rewardDebt;
        if (pending > 0) {
            self.sendReward(sender, pending);
        }
        
        // 更新质押信息
        info.amount = info.amount - msg.amount;
        info.rewardDebt = (info.amount * self.accRewardPerShare) / 1e12;
        
        if (info.amount == 0) {
            self.userInfo.del(sender);
        } else {
            self.userInfo.set(sender, info);
        }
        
        self.totalStaked = self.totalStaked - msg.amount;
        
        // 返还质押代币
        // 实际实现需要转账
    }
    
    // 紧急提款（带惩罚）
    receive(msg: EmergencyWithdraw) {
        let sender: Address = sender();
        let info: UserInfo = self.userInfo.get(sender)!!;
        let amount: Int = info.amount;
        
        require(amount > 0, "No staked amount");
        
        // 计算费用
        let fee: Int = (amount * self.emergencyWithdrawFee) / 10000;
        let receiveAmount: Int = amount - fee;
        
        // 删除用户信息
        self.userInfo.del(sender);
        self.totalStaked = self.totalStaked - amount;
        
        // 发送费用到费用接收地址
        if (fee > 0) {
            self.sendReward(self.feeRecipient, fee);
        }
        
        // 返还剩余代币给用户
        // 实际实现需要转账
    }
    
    // 领取奖励
    receive(msg: Harvest) {
        self.updatePool();
        
        let sender: Address = sender();
        let info: UserInfo = self.userInfo.get(sender)!!;
        
        let pending: Int = (info.amount * self.accRewardPerShare) / 1e12 - info.rewardDebt;
        require(pending > 0, "No pending rewards");
        
        info.rewardDebt = (info.amount * self.accRewardPerShare) / 1e12;
        self.userInfo.set(sender, info);
        
        self.sendReward(sender, pending);
    }
    
    // 发送奖励
    fun sendReward(to: Address, amount: Int) {
        // 实际实现需要发送奖励代币
        send(SendParameters{
            to: to,
            value: amount,
            mode: SendPayGasSeparately
        });
    }
    
    // 更新奖励速率
    receive(msg: UpdateRewardPerSecond) {
        require(sender() == self.owner, "Only owner");
        self.updatePool();
        self.rewardPerSecond = msg.newRate;
    }
    
    // 更新紧急提款费用
    receive(msg: UpdateEmergencyFee) {
        require(sender() == self.owner, "Only owner");
        require(msg.newFee <= 500, "Fee too high");  // 最大 5%
        self.emergencyWithdrawFee = msg.newFee;
    }
    
    // Getter 函数
    get fun getPoolInfo(): PoolInfo {
        return PoolInfo{
            totalStaked: self.totalStaked,
            rewardPerSecond: self.rewardPerSecond,
            accRewardPerShare: self.accRewardPerShare,
            lastUpdateTime: self.lastUpdateTime
        };
    }
    
    get fun getUserInfo(user: Address): UserInfo {
        return self.userInfo.get(user) ?: UserInfo{
            amount: 0,
            rewardDebt: 0
        };
    }
    
    get fun pendingRewardQuery(user: Address): Int {
        return self.pendingReward(user);
    }
    
    get fun getAPY(): Int {
        // 简化 APY 计算
        // 实际应该基于奖励代币价格和质押代币价格
        if (self.totalStaked == 0) return 0;
        
        let yearlyReward: Int = self.rewardPerSecond * 365 * 24 * 3600;
        return (yearlyReward * 10000) / self.totalStaked;  // 返回百分比 * 100
    }
}

// 消息定义
message Deposit {
    amount: Int as coins;
}

message Withdraw {
    amount: Int as coins;
}

message EmergencyWithdraw {}

message Harvest {}

message UpdateRewardPerSecond {
    newRate: Int;
}

message UpdateEmergencyFee {
    newFee: Int;
}

// 结构体
struct UserInfo {
    amount: Int as coins;
    rewardDebt: Int;
}

struct PoolInfo {
    totalStaked: Int as coins;
    rewardPerSecond: Int;
    accRewardPerShare: Int;
    lastUpdateTime: Int;
}
```

### 22.3 前端开发

#### 22.3.1 质押界面

```tsx
// src/pages/Staking.tsx
import { useState, useEffect } from 'react';
import { useTonConnectUI, useTonAddress } from '@tonconnect/ui-react';
import { toNano, fromNano } from '@ton/core';
import {
  Container,
  Typography,
  Card,
  CardContent,
  TextField,
  Button,
  Box,
  Grid,
  LinearProgress,
} from '@mui/material';

export function Staking() {
  const [tonConnectUI] = useTonConnectUI();
  const userAddress = useTonAddress();
  
  const [stakeAmount, setStakeAmount] = useState('');
  const [poolInfo, setPoolInfo] = useState({
    totalStaked: '0',
    apy: '0',
    userStaked: '0',
    pendingRewards: '0',
  });
  
  // 加载池子信息
  useEffect(() => {
    if (userAddress) {
      loadPoolInfo();
    }
  }, [userAddress]);
  
  const loadPoolInfo = async () => {
    // 调用合约获取信息
    // const pool = new StakingPool(...);
    // const info = await pool.getPoolInfo();
    // setPoolInfo(...);
  };
  
  const handleStake = async () => {
    if (!stakeAmount || parseFloat(stakeAmount) <= 0) return;
    
    await tonConnectUI.sendTransaction({
      messages: [
        {
          address: process.env.REACT_APP_STAKING_POOL!,
          amount: toNano(stakeAmount).toString(),
          payload: buildDepositPayload(toNano(stakeAmount)),
        },
      ],
    });
  };
  
  const handleHarvest = async () => {
    await tonConnectUI.sendTransaction({
      messages: [
        {
          address: process.env.REACT_APP_STAKING_POOL!,
          amount: toNano('0.05').toString(),
          payload: buildHarvestPayload(),
        },
      ],
    });
  };

  return (
    <Container maxWidth="lg" sx={{ mt: 4 }}>
      <Typography variant="h3" gutterBottom>
        TON Staking Pool
      </Typography>
      
      <Grid container spacing={3}>
        {/* 池子统计 */}
        <Grid item xs={12} md={4}>
          <Card>
            <CardContent>
              <Typography color="textSecondary" gutterBottom>
                Total Staked
              </Typography>
              <Typography variant="h4">
                {poolInfo.totalStaked} TON
              </Typography>
            </CardContent>
          </Card>
        </Grid>
        
        <Grid item xs={12} md={4}>
          <Card>
            <CardContent>
              <Typography color="textSecondary" gutterBottom>
                APY
              </Typography>
              <Typography variant="h4" color="primary">
                {poolInfo.apy}%
              </Typography>
            </CardContent>
          </Card>
        </Grid>
        
        <Grid item xs={12} md={4}>
          <Card>
            <CardContent>
              <Typography color="textSecondary" gutterBottom>
                Your Stake
              </Typography>
              <Typography variant="h4">
                {poolInfo.userStaked} TON
              </Typography>
            </CardContent>
          </Card>
        </Grid>
        
        {/* 质押操作 */}
        <Grid item xs={12} md={6}>
          <Card>
            <CardContent>
              <Typography variant="h6" gutterBottom>
                Stake TON
              </Typography>
              
              <TextField
                fullWidth
                label="Amount"
                type="number"
                value={stakeAmount}
                onChange={(e) => setStakeAmount(e.target.value)}
                margin="normal"
                InputProps={{
                  endAdornment: <Typography>TON</Typography>,
                }}
              />
              
              <Button
                variant="contained"
                fullWidth
                size="large"
                onClick={handleStake}
                disabled={!userAddress}
                sx={{ mt: 2 }}
              >
                Stake
              </Button>
            </CardContent>
          </Card>
        </Grid>
        
        {/* 奖励管理 */}
        <Grid item xs={12} md={6}>
          <Card>
            <CardContent>
              <Typography variant="h6" gutterBottom>
                Your Rewards
              </Typography>
              
              <Typography variant="h4" color="success.main" gutterBottom>
                {poolInfo.pendingRewards} Reward Tokens
              </Typography>
              
              <Button
                variant="contained"
                color="secondary"
                fullWidth
                size="large"
                onClick={handleHarvest}
                disabled={!userAddress || parseFloat(poolInfo.pendingRewards) <= 0}
              >
                Harvest Rewards
              </Button>
            </CardContent>
          </Card>
        </Grid>
      </Grid>
    </Container>
  );
}

// 辅助函数
function buildDepositPayload(amount: bigint): string {
  // 构建 deposit payload
  return '';
}

function buildHarvestPayload(): string {
  // 构建 harvest payload
  return '';
}
```

### 22.4 测试、部署与上线

#### 22.4.1 安全审计检查清单

```markdown
## 质押协议安全审计清单

### 访问控制
- [ ] 关键函数有适当的权限检查
- [ ] 管理员权限可以转移
- [ ] 紧急暂停机制正常工作

### 数学运算
- [ ] 没有整数溢出/下溢
- [ ] 除法运算前进行适当的乘法
- [ ] 奖励计算精度足够

### 重入攻击
- [ ] 使用 Checks-Effects-Interactions 模式
- [ ] 状态更新在转账之前
- [ ] 没有外部调用后的状态依赖

### 经济安全
- [ ] 奖励速率有上限
- [ ] 紧急提款费用合理
- [ ] 没有无限铸造风险

### 测试覆盖
- [ ] 单元测试 > 90%
- [ ] 集成测试覆盖主要流程
- [ ] 边界条件测试
- [ ] 异常处理测试
```

---

**项目三总结：**

通过 DeFi 质押协议项目，你学会了：
- 实现复杂的奖励计算机制
- 构建安全的质押合约
- 开发 DeFi 前端界面
- 进行安全审计和测试

---

**实战篇小结：**

实战篇通过三个完整的项目，将前面学习的知识应用到实际开发中：
- **第20章**：Jetton 代币发行平台 - 学习代币合约开发和前端集成
- **第21章**：NFT 市场 - 学习 NFT 合约和交易市场开发
- **第22章**：DeFi 质押协议 - 学习复杂的 DeFi 合约和奖励机制

完成这三个项目后，你已经具备了独立开发完整 TON DApp 的能力。建议继续探索更多实际项目，参与 TON 生态建设。

---

**教材完整总结：**

本教材从基础到实战，系统地介绍了 TON 区块链开发：

1. **基础篇**：理解 TON 核心概念和架构
2. **合约篇**：掌握 Tolk/Tact 智能合约开发
3. **工具篇**：熟练使用开发工具和框架
4. **应用篇**：学会构建 DApp 和 Telegram Mini Apps
5. **进阶篇**：掌握高级合约模式和优化技巧
6. **实战篇**：通过三个完整项目巩固所学知识

祝你在 TON 开发之旅中取得成功！

---
