# TON 区块链开发实战教材

> **版本**：v1.0 | **更新日期**：2026-05-20
> **定位**：从入门到进阶的全面系统教材，兼顾初学者与有经验的开发者

---

## 目录大纲

---

### 第一篇：基础篇 —— 认识 TON 区块链

#### 第 1 章：TON 区块链概述

- 1.1 什么是 TON（The Open Network）
  - 起源与发展历程（Telegram Open Network → The Open Network）
  - 核心设计理念：高吞吐量、无限扩展、异步执行
  - 与其他区块链（Ethereum、Solana）的对比
- 1.2 TON 生态全景
  - 主流钱包（Tonkeeper、MyTonWallet 等）
  - DeFi 协议（STON.fi、DeDust、Tonstakers 等）
  - NFT 与数字收藏品
  - Telegram Mini Apps 生态
  - Toncoin 与代币经济模型
- 1.3 开发环境搭建
  - Node.js 安装与配置（要求 v22.0.0+）
  - Blueprint 框架安装（`npm create ton@latest`）
  - Acton 工具安装与配置
  - IDE 配置（VSCode 插件：Tolk、Tact 语法支持）
  - TON 测试网与水龙头使用
  - 第一个 Hello World 合约

#### 第 2 章：TON 核心架构

- 2.1 多层链架构
  - Masterchain（主链）：全局配置与系统合约
  - Workchains（工作链）：Basechain 与未来扩展
  - Shardchains（分片链）：动态分片机制
  - 超立方体路由（Hypercube Routing）与消息传递
- 2.2 账户模型
  - 账户地址格式（raw address 与 user-friendly address）
  - bounceable 与 non-bounceable 地址
  - 账户状态（nonexist / uninit / active / frozen）
  - 智能合约的持久化存储（persistent storage）
- 2.3 消息机制
  - 消息类型：Internal / External In / External Out
  - 消息结构：header（flags、src、dst、value）与 body
  - 消息模式：普通消息与携带 payload 的消息
  - 异步消息模型与消息顺序保证
  - bounce 机制与错误处理
- 2.4 Gas 与费用模型
  - Gas 计量与转换（gas → Toncoin）
  - 执行费用、存储费用、转发费用
  - Gas 优化策略概述

#### 第 3 章：TON 数据原语

- 3.1 Cell（单元格）
  - Cell 结构：最多 1023 bits 数据 + 最多 4 个引用
  - Cell 树与有向无环图（DAG）
  - Cell 哈希与 Merkle 证明
  - 异构 Cell（Exotic Cell）：Pruned、Library、Merkle Proof/Update
- 3.2 Slice 与 Builder
  - Slice：Cell 的读取游标
  - Builder：构建新 Cell 的写入游标
  - 序列化与反序列化操作
- 3.3 Bag of Cells（BoC）
  - BoC 序列化格式
  - Cell 在网络传输与文件存储中的表示
- 3.4 TL-B 序列化方案
  - TL-B 语法与类型定义
  - 常用 TL-B 结构（消息、账户状态等）
  - 自定义 TL-B 类型编写

#### 第 4 章：TON 虚拟机（TVM）

- 4.1 TVM 架构概述
  - 基于栈的执行模型
  - TVM 数据类型（Integer、Cell、Slice、Builder、Tuple、Continuation、Null）
  - 控制寄存器（c0-c5、c7）
  - Gas 计数器与执行中断
- 4.2 TVM 指令集概览
  - 栈操作指令
  - 算术与逻辑指令
  - Cell/Slice/Builder 操作指令
  - 控制流指令
  - 字典操作指令
- 4.3 TVM 执行流程
  - 智能合约的触发与执行
  - 消息处理阶段（compute phase、action phase）
  - Exit code 与错误处理

---

### 第二篇：合约篇 —— 智能合约开发

#### 第 5 章：合约语言总览与选择

- 5.1 TON 合约语言体系
  - 语言层级：Fift（汇编）→ Tolk/Tact（高级）
  - 编译流程：高级语言 → Fift → TVM bytecode
- 5.2 Tolk 语言（官方推荐）
  - 设计理念与核心特性
  - 自动 cell 序列化与 lazy loading
  - 合约声明与 ABI 自动导出
  - TypeScript wrapper 自动生成
- 5.3 Tact 语言（社区推荐）
  - 设计理念与核心特性
  - 简洁语法与强类型系统
  - 内置安全特性（cashback 等）
  - Blueprint 集成与自动 wrapper 生成
- 5.4 语言选择建议
  - 新项目推荐路径
  - 各语言适用场景对比

#### 第 6 章：Tolk 语言开发实战

- 6.1 基础语法
  - 数据类型（int、cell、slice、builder、tuple）
  - 变量声明与赋值（val、var）
  - 运算符与表达式
  - 控制流（if/else、match、while、loop）
  - 函数定义与调用
- 6.2 合约结构
  - `contract` 声明与存储定义
  - `storage` 类型与持久化
  - `incomingMessages` 消息类型声明
  - `init` 函数（合约部署/初始化）
  - getter 函数（`get fun`）
- 6.3 消息处理
  - `onInternalMessage` 处理内部消息
  - `onExternalMessage` 处理外部消息
  - 消息体解析（`fromSlice`）
  - 消息发送（`send`、`emit`）
- 6.4 高级特性
  - lazy loading 与 gas 优化
  - 字典（Dict）操作
  - 错误处理与 require
  - 合约升级模式
- 6.5 实战：用 Tolk 开发一个投票合约
  - 需求分析与设计
  - 合约代码编写
  - 编译与部署
  - 测试与验证

#### 第 7 章：Tact 语言开发实战

- 7.1 基础语法
  - 数据类型（Int、Bool、Address、Cell、Slice、Builder、Map）
  - 常量与变量（`let`、`const`）
  - 运算符与表达式
  - 控制流（if/else、if-let、while、repeat、foreach）
  - 函数定义
- 7.2 合约结构
  - `contract` 声明与参数
  - `init()` 初始化函数
  - `receive()` 消息接收器
  - `get fun` getter 函数
  - `bounced()` bounce 处理器
- 7.3 消息与交互
  - `message` 类型定义
  - `SendParameters` 与消息发送
  - `context` 对象与交易上下文
  - `sender()` 与调用者识别
  - `cashback()` 自动退款机制
- 7.4 Trait 与组合
  - `trait` 定义与使用
  - `with` 关键字与合约组合
  - `mutates` 与 `virtual`/`override`
  - 常用内置 Trait（Ownable、Deployable 等）
- 7.5 高级特性
  - Map 字典操作
  - 序列化（`toCell`、`fromCell`）
  - 合约升级（`upgrade()`）
  - `__init__` 与 `initOf` 部署模式
- 7.6 实战：用 Tact 开发一个多签钱包
  - 需求分析与设计
  - 合约代码编写
  - Blueprint 项目配置
  - 编译、测试与部署

#### 第 8 章：标准合约与 TEP 协议

- 8.1 TEP 标准体系
  - TEP 提案流程与分类
  - 核心 TEP 标准概览
- 8.2 Jettons（TEP-74）—— 同质化代币
  - Jetton 架构设计（主合约 + 用户钱包合约）
  - Jetton 合约接口规范
  - Jetton 元数据标准（TEP-64）
  - 发行与管理 Jetton
- 8.3 NFT（TEP-62）—— 非同质化代币
  - NFT 合约接口规范
  - NFT 集合（Collection）合约
  - NFT 元数据与内容解析
  - NFT 版税（TEP-66）
  - 压缩 NFT（TEP-126）
- 8.4 TON DNS（TEP-81）
  - 域名注册与解析机制
  - DNS 合约接口
- 8.5 SBT（TEP-85）—— 灵魂绑定代币
  - SBT 合约接口规范
  - SBT 应用场景
- 8.6 钱包合约标准
  - 钱包版本演进（v1 → v5）
  - v4r2 与 v5r1 钱包合约详解
  - 高负载钱包（High-load Wallet v2/v3）

---

### 第三篇：工具篇 —— 开发工具链

#### 第 9 章：开发环境搭建

- 9.1 开发工具概述
  - TON 开发工具链全景
  - 开发流程介绍
- 9.2 Node.js 环境配置
  - Node.js 安装与配置
  - npm 镜像配置
  - 全局工具安装
- 9.3 BluePrint 框架安装
  - 项目创建（`npm create ton@latest`）
  - 项目结构详解
  - 常用命令
- 9.4 VS Code 配置
  - 推荐插件
  - 编辑器设置
  - 调试配置
- 9.5 测试网配置
  - 获取测试网 TON
  - 测试网钱包配置
  - 测试网浏览器
- 9.6 主网准备
  - 主网钱包设置
  - 获取主网 TON
  - 主网浏览器
- 9.7 其他开发工具
  - TON API 服务
  - 本地节点
  - 合约验证工具
- 9.8 开发工作流示例
- 9.9 常见问题排查
- 9.10 学习资源

#### 第 10 章：合约编译与测试

- 10.1 合约编译流程
  - 编译原理
  - Tact 编译
  - 编译输出分析
- 10.2 测试框架介绍
  - Sandbox 测试环境
  - 测试结构
  - 常用断言
- 10.3 编写单元测试
  - 基础功能测试
  - 错误处理测试
  - 状态验证测试
- 10.4 集成测试
  - 多合约交互测试
  - 消息流追踪
- 10.5 Gas 消耗分析
  - 测量 Gas 消耗
  - 优化 Gas 消耗
- 10.6 调试技巧
  - 日志输出
  - 状态检查
  - 断点调试
- 10.7 测试覆盖率
- 10.8 持续集成

#### 第 11 章：部署与交互

- 11.1 合约部署原理
  - 部署流程
  - 地址计算
- 11.2 部署脚本编写
  - 基础部署脚本
  - 高级部署选项
  - 多合约部署
- 11.3 网络部署
  - 测试网部署
  - 主网部署
  - 部署验证
- 11.4 合约交互
  - 读取合约状态
  - 发送消息
  - 监听事件
- 11.5 TON Connect 集成
  - 前端集成
  - React 集成示例
- 11.6 生产环境注意事项
  - 安全管理
  - 监控和报警
  - 升级策略
- 11.7 故障排查

---

### 第四篇：应用篇 —— DApp 开发

#### 第 12 章：TON SDK 与前端集成

- 12.1 @ton/core —— 核心数据结构
  - Cell、Slice、Builder 操作
  - 地址处理（Address）
  - 加密与签名
  - BOC 编解码
- 12.2 @ton/ton —— 区块链交互
  - TonClient 配置与使用
  - 合约调用与状态查询
  - 交易发送与确认
  - 区块链数据查询
- 12.3 多语言 SDK 生态
  - Python SDK（tonutils）
  - Go SDK（tonutils-go、tongo）
  - Rust SDK（ton-rs）
  - Java SDK（ton4j）
- 12.4 TypeScript Wrapper 模式
  - 合约 Wrapper 类编写
  - ABI 编码与解码
  - Getter 调用封装
  - 消息发送封装

#### 第 13 章：TON Connect 钱包连接

- 13.1 TON Connect 协议概述
  - 协议架构与安全模型
  - 端到端加密会话
  - 连接流程详解
- 13.2 @tonconnect/ui-react 集成
  - TonconnectUIProvider 配置
  - useTonConnect hook 使用
  - 连接/断开钱包
  - 读取钱包地址与网络信息
- 13.3 交易发送
  - 构建交易参数
  - 发送 Toncoin
  - 发送 Jetton
  - 调用智能合约
- 13.4 钱包兼容性
  - 支持的钱包列表
  - 钱包注册表管理
  - 兼容性处理

#### 第 14 章：AppKit 快速开发

- 14.1 AppKit 概述
  - AppKit 的定位与核心功能
  - 安装与初始化
- 14.2 React 集成
  - AppKitProvider 配置
  - 内置 UI 组件使用
  - 自定义主题与样式
- 14.3 数据查询
  - TanStack Query 集成
  - 余额查询
  - 交易历史查询
  - Jetton/NFT 数据查询
- 14.4 DeFi 功能集成
  - Swap 集成（Omniston、DeDust）
  - Staking 集成（Tonstakers）
  - 自定义 DeFi 提供者

#### 第 15 章：Telegram Mini Apps 开发

- 15.1 Telegram Mini Apps 概述
  - Mini Apps 架构与运行环境
  - Telegram Web App API
  - TON 生态中的 Mini Apps 定位
- 15.2 Mini Apps 开发基础
  - 项目初始化与配置
  - Telegram 主题适配
  - 用户数据获取
  - 主按钮与交互组件
- 15.3 TON 集成
  - TON Connect 在 Mini Apps 中的使用
  - AppKit 在 Mini Apps 中的集成
  - 内置钱包（Telegram Wallet）
  - 交易发起与确认
- 15.4 实战：开发一个 Telegram Mini App 游戏
  - 需求分析与设计
  - 前端开发
  - 合约开发与部署
  - 集成与测试

---

### 第五篇：进阶篇 —— 高级主题

#### 第 16 章：高级合约模式

- 16.1 代理合约模式
  - 代理合约设计原理
  - 消息转发与费用优化
  - 应用场景（批量操作、Gas 站）
- 16.2 合约升级模式
  - 代码分离模式（code separation）
  - 存储布局兼容性
  - 升级安全考量
- 16.3 高负载合约设计
  - 高负载钱包（High-load Wallet）
  - 消息队列与批处理
  - 并发处理策略
- 16.4 跨合约通信
  - 合约间消息传递模式
  - 回调模式（callback pattern）
  - 错误传播与处理
  - 竞态条件与防范

#### 第 17 章：DeFi 合约开发

- 17.1 AMM（自动做市商）原理
  - 恒定乘积做市商（x * y = k）
  - 流动性池设计
  - 滑点与价格影响
- 17.2 DEX 合约开发
  - 流动性池合约
  - Router 合约
  - Jetton Swap 实现
- 17.3 质押（Staking）合约
  - 质押池设计
  - 奖励分配机制
  - 提款与惩罚机制
- 17.4 治理合约
  - 提案创建与投票
  - 时间锁与执行
  - 权重计算

#### 第 18 章：节点运维与基础设施

- 18.1 TON 节点类型
  - 验证者节点（Validator）
  - 全节点（Full Node）
  - 轻节点（Light Node）
  - Liteserver
- 18.2 节点部署
  - 硬件与网络要求
  - mytonctrl 安装与配置
  - 节点启动与同步
- 18.3 验证者运营
  - 质押与选举
  - 奖励机制
  - 监控与维护
- 18.4 网络接入
  - HTTP API（TON Center）
  - ADNL 协议
  - Liteserver 客户端

#### 第 19 章：性能优化与 Gas 调优

- 19.1 Gas 消费分析
  - Gas 计量模型详解
  - 常见操作的 Gas 成本
  - Gas 基准测试工具使用
- 19.2 存储优化
  - Cell 树结构优化
  - 存储费用计算与最小化
  - 数据压缩策略
- 19.3 计算优化
  - 循环优化
  - 字典操作优化
  - Lazy loading 策略
- 19.4 消息优化
  - 消息合并与批处理
  - 转发费用优化
  - bounce 消息处理优化

---

### 第六篇：实战篇 —— 综合项目

#### 第 20 章：实战项目一 —— Jetton 代币发行平台

- 20.1 项目概述与架构设计
- 20.2 智能合约开发
  - Jetton 主合约
  - Jetton 钱包合约
  - 管理功能合约
- 20.3 前端开发
  - 项目初始化与 UI 设计
  - TON Connect 集成
  - 代币发行与管理界面
- 20.4 测试、部署与上线

#### 第 21 章：实战项目二 —— NFT 市场place

- 21.1 项目概述与架构设计
- 21.2 智能合约开发
  - NFT 集合合约
  - NFT Item 合约
  - 市场place 合约（挂单、购买、版税）
- 21.3 前端开发
  - NFT 展示与浏览
  - 创建与上架 NFT
  - 购买与交易界面
- 21.4 测试、部署与上线

#### 第 22 章：实战项目三 —— DeFi 质押协议

- 22.1 项目概述与架构设计
- 22.2 智能合约开发
  - 质押池合约
  - 奖励分配合约
  - 治理合约
- 22.3 前端开发
  - 质押/解质押界面
  - 奖励查询与领取
  - 治理投票界面
- 22.4 测试、部署与上线

---

### 附录

- 附录 A：TON 术语表
- 附录 B：常用 TEP 标准速查
- 附录 C：Tolk / Tact 语法速查对照表
- 附录 D：常用开发命令速查
- 附录 E：测试网与水龙头资源
- 附录 F：学习资源与社区链接
- 附录 G：常见问题（FAQ）

---

> **说明**：
> - 本大纲基于 2025-2026 年 TON 生态最新状态编写
> - Tolk 为官方推荐首选合约语言，Tact 为社区推荐高级语言
> - Acton 为官方新一代开发工具，与 Blueprint 并列介绍
