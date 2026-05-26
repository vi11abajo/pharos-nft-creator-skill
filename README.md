# Pharos NFT Creator — User Guide

## Overview

A Pharos Agent Center Skill that enables AI agents to build full-lifecycle NFT collections on the Pharos blockchain. Supports generative (trait-based), individual image, and single-image modes. You interact with the agent in natural language, and it handles image generation, IPFS uploads, smart contract deployment, minting, and distribution.

## Features

- Automatic dependency check on every invocation (Python, Forge, Cast, Node.js, etc.)
- Auto-creation of `.env` file with environment variables
- Auto-normalization of private key (paste with or without `0x` prefix)
- Three artwork modes: generative (traits), individual images, single image
- Generative NFT creation with trait layers and rarity weights
- Off-chain image composition from PNG layers using Python Pillow
- IPFS upload of images + metadata folder via Pinata (SDK for files, REST API for folders)
- BaseURI pattern — contract returns `ipfs://CID/tokenId` for each NFT
- Pharos network data built-in — choose testnet/mainnet, RPC auto-configured
- ERC-721 smart contract deployment with 6 mint modes (public free/paid, owner-only, whitelist free/paid, whitelist free + public paid)
- Optional minting landing page (HTML + ethers.js, auto-configured)
- Batch minting and airdrop to any number of wallets (inline or from file)
- Whitelist support via Merkle tree (auto-generated with correct proofs)
- Holder analytics with JSON/CSV export
- Agent communicates in the user's language

## Installation

Copy the **entire** `pharos-nft-creator-skill` folder to your agent's skills directory:

| Agent | Path |
|-------|------|
| Claude Code | `~/.claude/skills/` |
| OpenClaw | `~/.openclaw/skills/` |
| Codex | `~/.codex/skills/` |

Example for Claude Code:
```bash
cp -r pharos-nft-creator-skill ~/.claude/skills/
```

Skill folder structure:
```
pharos-nft-creator-skill/
├── pharos-nft-creator-skill.md         — main skill file
├── dependencies.json                  — dependency list for auto-check
├── .env.example                       — environment variables template
├── README.md                          — instructions (EN)
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

Restart your agent session after installing.

## First Launch — Automatic Checks

When you invoke the skill for the first time, the agent automatically runs **Phase 0: Dependency Check**:

1. **Checks tools** — Python, Pillow, Forge, Cast, jq, Node.js, Pinata SDK. Offers to install any missing ones.
2. **Creates `.env` file** — if it doesn't exist in the project, copies from `.env.example` with placeholders.
3. **Verifies variables** — checks if `PRIVATE_KEY`, `PINATA_JWT` are filled in.
4. **Guides you to `.env`** — if secrets are empty, shows the **file path** and tells you which lines to fill.

Pharos RPC URLs and network data are **built into the Skill** — no manual configuration needed. You simply choose the network during setup.

Example output:
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

## Environment Variables

All secrets are stored in the `.env` file in the project root. **Never paste keys directly into the chat.**

The `.env` file (created automatically):
```bash
# Your wallet private key.
# Paste with or without 0x prefix — the skill auto-normalizes it.
# SECURITY: Fill in the file directly. Do NOT paste in chat.
PRIVATE_KEY=

# Pinata JWT token — how to get it:
#   1. Go to https://app.pinata.cloud → sign up / log in
#   2. Left sidebar → "API Keys" → "New Key"
#   3. Name it (e.g. "nft-builder"), leave default permissions
#   4. Click "Create Key" → copy the JWT (long string starting with "eyJ...")
#   5. Paste below
PINATA_JWT=

# Network is selected during configuration — no need to set RPC URL manually.
```

### Where to get values

| Variable | Source |
|----------|--------|
| `PRIVATE_KEY` | MetaMask → three dots → Account details → Show private key. Paste with or without `0x` — auto-normalized |
| `PINATA_JWT` | [app.pinata.cloud](https://app.pinata.cloud) → API Keys → New Key → copy the JWT |

## Preparing Artwork

The skill supports **three artwork modes**:

### Mode A: Generative (Traits)

Classic generative approach — multiple trait layers are randomly combined. All files must be PNG, RGBA, same dimensions.

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
│   ├── none.png          (empty transparent PNG)
│   ├── crown.png
│   ├── cap.png
│   └── helmet.png
└── Weapon/
    ├── none.png
    ├── sword.png
    └── axe.png
```

### Mode B: Individual Images

A folder of pre-made PNGs — each file = one unique NFT. You assign a rarity weight to each. Provide a JSON/CSV config or enter manually.

```
my-nfts/
├── 001_samurai.png     (weight: 10 — legendary)
├── 002_ninja.png       (weight: 30 — uncommon)
├── 003_wizard.png      (weight: 5 — legendary)
├── 004_knight.png      (weight: 55 — common)
└── ...
```

Config (optional, JSON):
```json
[
  {"file": "001_samurai.png", "weight": 10, "name": "Samurai"},
  {"file": "002_ninja.png", "weight": 30, "name": "Ninja"}
]
```

### Mode C: Single Image

One PNG for all NFTs. Every token looks the same. Good for membership passes, tickets, utility NFTs.

```
collection-art.png    (single image for the entire collection)
```

### Rarity Weights (for Modes A and B)

Weights determine how frequently each trait variant appears. Higher weight = more common:

| Category | Weight | Frequency |
|----------|--------|-----------|
| Legendary | 1–4 | Very rare |
| Rare | 5–19 | Uncommon |
| Uncommon | 20–49 | Moderate |
| Common | 50–100 | Frequent |

Example: if Headwear has Crown(5), Cap(30), Helmet(40), None(25), Crown will appear in roughly 5% of generated NFTs.

## Usage

### 1. Start Collection Creation

Tell the agent:
> "Deploy a generative NFT collection on Pharos"

The agent will check dependencies, create `.env` if needed, and begin interactive configuration.

### 2. Answer Configuration Questions

The agent will ask for:
- **Which network to use** — Pharos Atlantic Testnet (free, for testing) or Pharos Mainnet (real PROS tokens)
- Collection name and symbol
- Description
- Total supply (max number of NFTs)
- Mint type (see below)
- **Artwork mode** — generative (traits), individual images, or single image
- Artwork details: trait layers, or folder of PNGs + weights, or path to a single PNG

### 3. Mint Types

The agent will explain each option and help you choose. Six types are available:

| Type | Description | Use case |
|------|-------------|----------|
| **Public Free** | Anyone can mint for free | Open community collections |
| **Public Paid** | Anyone pays a set price | Premium collections |
| **Owner-Only** | Only contract owner can mint | Airdrops, private distributions |
| **Whitelist Free** | Only whitelisted addresses can mint for free | Exclusive community drops |
| **Whitelist Paid** | Only whitelisted addresses pay to mint | Premium community drops |
| **WL Free + Public Paid** | Whitelisted mint free, everyone else pays | Community rewards + public sale |

For Whitelist types, you need to provide a list of addresses (file or inline). The agent automatically generates a Merkle tree and uploads it to the contract.

### 4. After Deployment

**Mint and distribute NFTs:**
> "Mint 50 NFTs from my collection and send them to the addresses in airdrop_list.txt"

Address file format — one address per line:
```
0xAAA...BBB...CCC...
# Lines starting with # are comments
```

Or inline:
> "Mint 3 NFTs and send to: 0xAAA..., 0xBBB..., 0xCCC..."

**View holders:**
> "Show me all holders of my NFT collection"

**Export to file:**
> "Export holder data to CSV"

**Set up whitelist:**
> "Set up a whitelist for my collection using these addresses: 0xAAA, 0xBBB"

**Update mint parameters:**
> "Enable public minting at 0.05 PROS, max 5 per wallet"

## Troubleshooting

| Issue | Solution |
|-------|----------|
| PNG dimensions mismatch | Verify all files are the same size, re-export if needed |
| IPFS upload failure | Check `PINATA_JWT` in `.env` and Pinata account quota |
| **Pinata: 500 file limit** | Free plan = 500 files, 1 GB. Mode A: ~250 NFT (2N files). Mode B: up to 500-M NFTs. Mode C: up to ~500 NFTs |
| Forge not found | Install: `curl -L https://foundry.paradigm.xyz \| bash && foundryup` |
| Pillow not found | Install: `pip install Pillow` |
| **Stack too deep** error | Add `via_ir = true` to `foundry.toml` — required for this contract |
| Forge compilation error | Ensure OpenZeppelin v5 is installed: `forge install https://github.com/OpenZeppelin/openzeppelin-contracts@v5.0.2 --no-git` |
| Transaction fails on Pharos | Always use `--legacy` flag with `forge create` and `cast send` |
| Insufficient gas | Check wallet balance on Pharos |
| Mint not working | Verify you are the contract owner, and publicMintEnabled is true for public collections |
| `.env` not loading | Each bash command must start with `set -a && source .env && set +a` |
| NFT image not showing in explorer | Ensure metadata uses `ipfs://CID` (not gateway URLs). Verify folder CID is correct. |

## Security

- **Never paste** private keys, JWT tokens, or any secrets into the chat
- All secrets are stored **only in the `.env` file** — the agent reads it automatically
- `.env` is automatically added to `.gitignore`
- `state.json` is also added to `.gitignore`
- Ensure you control the contract's owner key
- If you accidentally paste a secret in chat — rotate that key immediately

## Output File Structure

After the skill runs, your project will contain:

```
project/
├── .env                         # Your secrets (NEVER commit!)
├── .gitignore                   # .env and state.json excluded
└── nft-collection/
    ├── traits/                  # Original trait PNG files (Mode A)
    ├── composed/                # Composited final NFT images (Pillow)
    ├── metadata/                # JSON metadata for each token (named by ID)
    ├── src/                     # Solidity contract (Foundry)
    ├── landing/                 # Minting web page (optional, Phase 5)
    ├── state.json               # Local collection database
    └── holders.json             # Holder data (after analytics)
```
