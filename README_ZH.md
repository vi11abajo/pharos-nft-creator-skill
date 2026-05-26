# Pharos NFT Creator — 用户指南

## 概述

这是一个 Pharos Agent Center 技能插件，使 AI 代理能够在 Pharos 区块链上构建全生命周期的 NFT 合集。支持生成式（基于属性）、独立图片和单图三种模式。你可以使用自然语言与代理交互，代理将自动处理图片生成、IPFS 上传、智能合约部署、铸造和分发。

## 功能特性

- 每次调用时自动检查依赖项（Python、Forge、Cast、Node.js 等）
- 自动创建包含环境变量的 `.env` 文件
- 自动规范化私钥（粘贴时可带或不带 `0x` 前缀）
- 三种艺术作品模式：生成式（属性层）、独立图片、单图
- 通过属性层和稀有度权重创建生成式 NFT
- 使用 Python Pillow 从 PNG 图层进行链下图片合成
- 通过 Pinata 上传图片 + 元数据文件夹（文件使用 SDK，文件夹使用 REST API）
- BaseURI 模式 — 合约为每个 NFT 返回 `ipfs://CID/tokenId`
- 内置 Pharos 网络数据 — 选择测试网/主网，RPC 自动配置
- ERC-721 智能合约部署，支持 6 种铸造模式（公开免费/付费、仅限所有者、白名单免费/付费、白名单免费 + 公开付费）
- 可选的铸造落地页（HTML + ethers.js，自动配置）
- 批量铸造和空投至任意数量的钱包（内联或从文件导入）
- 通过 Merkle 树支持白名单（自动生成正确的证明）
- 持有者分析，支持 JSON/CSV 导出
- 代理使用用户的语言进行交流

## 安装

将整个 `pharos-nft-creator-skill` 文件夹复制到你代理的技能目录中：

| 代理 | 路径 |
|-------|------|
| Claude Code | `~/.claude/skills/` |
| OpenClaw | `~/.openclaw/skills/` |
| Codex | `~/.codex/skills/` |

以 Claude Code 为例：
```bash
cp -r pharos-nft-creator-skill ~/.claude/skills/
```

技能文件夹结构：
```
pharos-nft-creator-skill/
├── pharos-nft-creator-skill.md         — 主技能文件
├── dependencies.json                  — 自动检查的依赖列表
├── .env.example                       — 环境变量模板
├── README.md                          — 说明文档 (EN)
├── README_ZH.md                       — 中文说明
├── README_HI.md                       — हिन्दी निर्देश
├── README_ES.md                       — instrucciones (ES)
├── README_FR.md                       — instructions (FR)
├── README_AR.md                       — تعليمات (AR)
├── README_BN.md                       — নির্দেশনা (BN)
├── README_UK.md                       — інструкції (UK)
├── README_PT.md                       — instruções (PT)
└── README_UR.md                       — ہدایات (UR)
```

安装后请重启你的代理会话。

## 首次启动 — 自动检查

当你首次调用该技能时，代理会自动运行 **阶段 0：依赖检查**：

1. **检查工具** — Python、Pillow、Forge、Cast、jq、Node.js、Pinata SDK。如果缺少任何工具，会提示你安装。
2. **创建 `.env` 文件** — 如果项目根目录中不存在该文件，则从 `.env.example` 复制并填充占位符。
3. **验证变量** — 检查 `PRIVATE_KEY`、`PINATA_JWT` 是否已填写。
4. **引导你到 `.env`** — 如果密钥为空，会显示**文件路径**并告诉你需要填写哪些行。

Pharos RPC URL 和网络数据**内置在技能中** — 无需手动配置。你只需在设置过程中选择网络即可。

示例输出：
```
=== Pharos NFT Creator — Dependency Check ===

Required:
  [OK]   Python 3.10+         — Python 3.12.4
  [OK]   Pillow library       — ok
  [OK]   Foundry Forge        — forge 0.2.0
  [OK]   Foundry Cast         — cast 0.2.0
  [OK]   jq                   — jq-1.7
  [OK]   Node.js              — v20.20.0
  [OK]   Pinata SDK           — ok

Optional:
  [MISS] PRIVATE_KEY          — not set (needed for: deployment, minting)
  [MISS] PINATA_JWT           — not set (needed for: IPFS upload)

Network: will be selected at configuration (Pharos Testnet / Pharos Mainnet)

Result: 7/7 required OK. 0/2 optional set.
Action: Can proceed with configuration. Deployment and IPFS will require env vars.
```

## 环境变量

所有密钥存储在项目根目录的 `.env` 文件中。**切勿将密钥直接粘贴到聊天中。**

`.env` 文件（自动创建）：
```bash
# 你的钱包私钥。
# 粘贴时可带或不带 0x 前缀 — 技能会自动规范化。
# 安全提示：直接在文件中填写。不要在聊天中粘贴。
PRIVATE_KEY=

# Pinata JWT 令牌 — 获取方式：
#   1. 访问 https://app.pinata.cloud → 注册/登录
#   2. 左侧边栏 → "API Keys" → "New Key"
#   3. 命名（如 "nft-builder"），保留默认权限
#   4. 点击 "Create Key" → 复制 JWT（以 "eyJ..." 开头的长字符串）
#   5. 粘贴到下方
PINATA_JWT=

# 网络在配置过程中选择 — 无需手动设置 RPC URL。
```

### 如何获取各变量的值

| 变量 | 来源 |
|----------|--------|
| `PRIVATE_KEY` | MetaMask → 三个点 → 账户详情 → 显示私钥。粘贴时可带或不带 `0x` — 自动规范化 |
| `PINATA_JWT` | [app.pinata.cloud](https://app.pinata.cloud) → API Keys → New Key → 复制 JWT |

## 准备艺术作品

该技能支持 **三种艺术作品模式**：

### 模式 A：生成式（属性层）

经典生成式方法 — 多个属性层随机组合。所有文件必须是 PNG 格式、RGBA 颜色模式、相同尺寸。

```
traits/
├── Background/
│   ├── blue.png
│   ├── red.png
│   └── green.png
├── Body/
│   ├── normal.png
│   └── muscular.png
├── Headwear/
│   ├── none.png          （空白透明 PNG）
│   ├── crown.png
│   ├── cap.png
│   └── helmet.png
└── Weapon/
    ├── none.png
    ├── sword.png
    └── axe.png
```

### 模式 B：独立图片

一个包含预制 PNG 的文件夹 — 每个文件 = 一个独特的 NFT。你可以为每个文件分配稀有度权重。可以提供 JSON/CSV 配置文件，也可以手动输入。

```
my-nfts/
├── 001_samurai.png     (权重: 10 — 传说级)
├── 002_ninja.png       (权重: 30 — 非凡级)
├── 003_wizard.png      (权重: 5 — 传说级)
├── 004_knight.png      (权重: 55 — 普通级)
└── ...
```

配置文件（可选，JSON 格式）：
```json
[
  {"file": "001_samurai.png", "weight": 10, "name": "Samurai"},
  {"file": "002_ninja.png", "weight": 30, "name": "Ninja"}
]
```

### 模式 C：单图

一张 PNG 用于所有 NFT。每个代币的外观相同。适用于会员通行证、门票、功能型 NFT。

```
collection-art.png    （整个合集使用的单张图片）
```

### 稀有度权重（适用于模式 A 和 B）

权重决定了每个属性变体出现的频率。权重越高 = 越常见：

| 类别 | 权重 | 频率 |
|----------|--------|-----------|
| 传说级 | 1–4 | 极稀有 |
| 稀有 | 5–19 | 较少见 |
| 非凡 | 20–49 | 中等 |
| 普通 | 50–100 | 频繁出现 |

示例：如果头饰属性有 Crown(5)、Cap(30)、Helmet(40)、None(25)，那么 Crown 大约会出现在 5% 的生成 NFT 中。

## 使用方法

### 1. 开始创建合集

告诉代理：
> "在 Pharos 上部署一个生成式 NFT 合集"

代理将检查依赖项、按需创建 `.env`，并开始交互式配置。

### 2. 回答配置问题

代理会询问以下信息：
- **使用哪个网络** — Pharos Atlantic 测试网（免费，用于测试）或 Pharos 主网（真实 PROS 代币）
- 合集名称和符号
- 描述
- 总供应量（NFT 的最大数量）
- 铸造类型（见下文）
- **艺术作品模式** — 生成式（属性层）、独立图片或单图
- 艺术作品详情：属性层，或 PNG 文件夹 + 权重，或单张 PNG 的路径

### 3. 铸造类型

代理会解释每个选项并帮助你选择。共有六种类型：

| 类型 | 描述 | 适用场景 |
|------|-------------|----------|
| **公开免费** | 任何人都可以免费铸造 | 开放社区合集 |
| **公开付费** | 任何人支付设定价格即可铸造 | 高级合集 |
| **仅限所有者** | 只有合约所有者可以铸造 | 空投、私人分发 |
| **白名单免费** | 只有白名单地址可以免费铸造 | 专属社区发行 |
| **白名单付费** | 只有白名单地址付费铸造 | 高级社区发行 |
| **白名单免费 + 公开付费** | 白名单用户免费铸造，其他人付费铸造 | 社区奖励 + 公开销售 |

对于白名单类型，你需要提供一个地址列表（文件或内联）。代理会自动生成 Merkle 树并将其上传到合约。

### 4. 部署后

**铸造并分发 NFT：**
> "铸造 50 个 NFT 并发送到 airdrop_list.txt 中的地址"

地址文件格式 — 每行一个地址：
```
0xAAA...BBB...CCC...
# 以 # 开头的行为注释
```

或者内联方式：
> "铸造 3 个 NFT 并发送到：0xAAA..., 0xBBB..., 0xCCC..."

**查看持有者：**
> "显示我的 NFT 合集的所有持有者"

**导出到文件：**
> "将持有者数据导出为 CSV"

**设置白名单：**
> "使用以下地址为我的合集设置白名单：0xAAA, 0xBBB"

**更新铸造参数：**
> "启用公开铸造，价格为 0.05 PROS，每个钱包最多 5 个"

## 常见问题排查

| 问题 | 解决方案 |
|-------|----------|
| PNG 尺寸不匹配 | 确认所有文件尺寸一致，必要时重新导出 |
| IPFS 上传失败 | 检查 `.env` 中的 `PINATA_JWT` 和 Pinata 账户配额 |
| **Pinata：500 文件限制** | 免费计划 = 500 个文件，1 GB。模式 A：约 250 个 NFT（2N 个文件）。模式 B：最多 500-M 个 NFT。模式 C：最多约 500 个 NFT |
| 找不到 Forge | 安装：`curl -L https://foundry.paradigm.xyz | bash && foundryup` |
| 找不到 Pillow | 安装：`pip install Pillow` |
| **Stack too deep** 错误 | 在 `foundry.toml` 中添加 `via_ir = true` — 此合约必需 |
| Forge 编译错误 | 确保已安装 OpenZeppelin v5：`forge install https://github.com/OpenZeppelin/openzeppelin-contracts@v5.0.2 --no-git` |
| Pharos 上交易失败 | 始终在 `forge create` 和 `cast send` 中使用 `--legacy` 标志 |
| Gas 不足 | 检查 Pharos 上的钱包余额 |
| 铸造不生效 | 确认你是合约所有者，并且对于公开合集，publicMintEnabled 为 true |
| `.env` 未加载 | 每个 bash 命令必须以 `set -a && source .env && set +a` 开头 |
| NFT 图片在浏览器中不显示 | 确保元数据使用 `ipfs://CID`（而非网关 URL）。确认文件夹 CID 正确。 |

## 安全须知

- **切勿将**私钥、JWT 令牌或任何密钥粘贴到聊天中
- 所有密钥**仅存储在 `.env` 文件中** — 代理会自动读取
- `.env` 会自动添加到 `.gitignore`
- `state.json` 也会添加到 `.gitignore`
- 确保你控制合约的所有者密钥
- 如果你不小心在聊天中粘贴了密钥 — 请立即轮换该密钥

## 输出文件结构

技能运行后，你的项目将包含：

```
project/
├── .env                         # 你的密钥（切勿提交！）
├── .gitignore                   # 排除 .env 和 state.json
└── nft-collection/
    ├── traits/                  # 原始属性 PNG 文件（模式 A）
    ├── composed/                # 合成后的最终 NFT 图片（Pillow）
    ├── metadata/                # 每个代币的 JSON 元数据（按 ID 命名）
    ├── src/                     # Solidity 合约（Foundry）
    ├── landing/                 # 铸造网页（可选，阶段 5）
    ├── state.json               # 本地合集数据库
    └── holders.json             # 持有者数据（分析后生成）
```
