---
name: pharos-nft-creator-skill
description: Full-lifecycle NFT collection builder for Pharos — generative (trait layers composed off-chain into images), individual images, or single image modes. Supports 6 mint types (public free/paid, owner-only, whitelist free/paid, whitelist free + public paid), IPFS upload via Pinata (images + metadata folder), baseURI pattern, minting landing page, batch minting, airdrop, Merkle whitelist, holder analytics, and auto dependency checking.
version: 2.0.0
frameworks:
  - openclaw
  - claude-code
  - codex
tags:
  - nft
  - erc721
  - generative-art
  - individual-images
  - traits
  - rarity
  - off-chain-composition
  - base-uri
  - ipfs
  - pinata
  - deploy
  - mint
  - batch-transfer
  - airdrop
  - whitelist
  - merkle-tree
  - holder-analytics
  - landing-page
  - pharos
  - phrs
  - pros
networks:
  - pharos-testnet
  - pharos-mainnet
---

# NFT Creator

Build and manage full-lifecycle NFT collections on Pharos — from trait system design through off-chain image composition, IPFS upload, contract deployment, minting, batch distribution, and holder analytics.

## Built-in Network Configuration

The skill knows Pharos network details. The agent does NOT need the user to provide RPC URLs or chain IDs — they are embedded below. At Phase 1 the agent only asks which network to use.

### Pharos Atlantic Testnet

| Parameter | Value |
|-----------|-------|
| Network name | Pharos Atlantic Testnet |
| Chain ID | 688689 (0xa8231) |
| Currency | PHRS |
| RPC URL | `https://atlantic.dplabs-internal.com` |
| Block Explorer | `https://atlantic.pharosscan.com` (verify if available) |

### Pharos Mainnet

| Parameter | Value |
|-----------|-------|
| Network name | Pharos Mainnet |
| Chain ID | (check current — may vary) |
| Currency | PROS |
| RPC URLs (fallback order) | 1. `https://rpc.pharos.xyz` 2. `https://infra.originstake.com/pharos/evm` |
| Block Explorer | `https://www.pharosscan.xyz` |

When the user selects a network, the agent MUST:
1. Set `PHAROS_RPC_URL` to the corresponding RPC URL
2. Set `PHAROS_NETWORK` to the network name
3. Set `PHAROS_CHAIN_ID` to the chain ID
4. Use the correct currency symbol based on network: `PHRS` for testnet, `PROS` for mainnet
5. For mainnet, prefer `https://rpc.pharos.xyz` and fall back to `https://infra.originstake.com/pharos/evm` if it fails

## Capabilities

- **Trait System Design** — Define layers (background, body, headwear, weapon, etc.) with named variants and rarity weights. The skill guides the user through creating a complete trait hierarchy before any code is written.
- **Off-Chain Image Composition** — For generative mode, trait layers are composed into final PNG images off-chain using Python Pillow. Individual and single image modes use user-provided images directly.
- **IPFS Upload** — Upload composite images + metadata JSON folder to IPFS via Pinata. Uses `pinata` npm package for individual files and REST API for folder uploads.
- **Three Artwork Modes** — Generative (trait layers composed off-chain), Individual Images (user images), or Single Image (one image for all NFTs).
- **Smart Contract Deployment** — Generate and deploy a configurable ERC-721 contract with baseURI pattern, supporting 6 mint types: public free, public paid, owner-only, whitelist free, whitelist paid, and whitelist free + public paid.
- **Contract Verification** — Verify deployed contract source code on the Pharos block explorer for transparency.
- **Batch Minting & Distribution** — Mint NFTs to any number of recipient wallets. Accept addresses inline or from a text file (one address per line).
- **Holder Analytics** — Query on-chain data to list all unique token holders, compute distribution stats, and export results to file.
- **Local State Persistence** — Save all collection configuration, deployment details, IPFS CIDs, and holder snapshots to a local JSON database for future reference.

## Phase 0: Dependency Check (Automatic)

**This phase runs automatically every time the skill is invoked, before any other work begins.**

Read the `dependencies.json` file located alongside this skill file (same directory). For each entry in the `checks` array, run the corresponding `test` command and verify the result.

### Step 0.1: Environment File Setup

Before checking env vars, ensure the `.env` file exists:

1. Check if `.env` exists in the current project root directory.
2. If `.env` does **not** exist:
   - Copy `.env.example` from this skill's directory to `.env` in the project root.
   - If `.env.example` is also missing, create `.env` with the following content:

```
# ============================================
# NFT Creator — Environment Variables
# ============================================
# Fill in your values below. NEVER share this file or commit it to git.
# ============================================

# Your wallet private key (without 0x prefix)
# SECURITY: Edit this file directly. Do NOT paste your key in chat.
PRIVATE_KEY=

# Pharos RPC endpoint URL
PHAROS_RPC_URL=

# Pinata JWT token (get one at https://pinata.cloud)
PINATA_JWT=

# Pinata Gateway URL (for accessing IPFS content via HTTP)
PINATA_GATEWAY=https://gateway.pinata.cloud/ipfs/

# Pharos block explorer API URL (optional, for contract verification)
PHAROS_EXPLORER_API=

# Network name (pharos-testnet / pharos-mainnet)
PHAROS_NETWORK=pharos-testnet
```

3. Ensure `.env` is listed in `.gitignore`. If `.gitignore` exists, check for `.env` entry. If missing, append it. If `.gitignore` doesn't exist, create it with `.env` as the first line.
4. Load variables from `.env` into the current shell session by running: `set -a && source .env && set +a` (bash) or equivalent for the current OS.

**CRITICAL — Shell state does NOT persist between Bash tool calls.** Each Bash command runs in a separate shell process. This means:
- `export` and `source` from one command are NOT available in the next command
- EVERY command that uses env vars (PRIVATE_KEY, PINATA_JWT, PHAROS_RPC_URL, CONTRACT) MUST start with: `set -a && source /absolute/path/to/.env && set +a && ...rest of command...`
- **Use absolute path to .env** — relative `source .env` fails if the agent's CWD changed (e.g. after `cd nft-collection`). Always use the full path, e.g. `source /root/.env` or `source $HOME/.env`.
- Example: `set -a && source $HOME/.env && set +a && forge create ... --private-key $PRIVATE_KEY ...`
- Node.js scripts that read `process.env` also need the source prefix: `set -a && source .env && set +a && node upload-traits.mjs`

### Step 0.2: Check Dependencies

For each entry in `dependencies.json` `checks` array:

1. Run the `test` command via Bash
2. If `expectContains` is set — verify the output contains that string
3. If `expectNotEmpty` is set — verify the output is not empty
4. If `expectEmpty` is set — verify the command produces no output (stderr only, or exit code 0). This is used for checks like `require('package')` where a successful import produces no stdout.
5. If the check passes → mark as `OK`
6. If the check fails:
   - If `required: true` → mark as `MISSING` (blocking)
   - If `required: false` → mark as `OPTIONAL` (non-blocking, note what it's needed for using `neededFor`)

### Required Checks (must pass to proceed)

| ID | What | Test |
|----|------|------|
| `python` | Python 3.10+ | `python3 --version` or `python --version` |
| `pillow` | Pillow library | `python3 -c "from PIL import Image; print('ok')"` |
| `forge` | Foundry Forge | `forge --version` — if fails, try `$HOME/.foundry/bin/forge --version` |
| `cast` | Foundry Cast | `cast --version` — if fails, try `$HOME/.foundry/bin/cast --version` |
| `jq` | JSON processor | `jq --version` |
| `node` | Node.js | `node --version` |
| `pinata_sdk` | Pinata SDK (for public IPFS uploads) | `node -e "require('pinata')"` |

**Foundry PATH note:** If forge/cast are found in `~/.foundry/bin/` but not in PATH, the agent MUST remember to prepend `export PATH="$PATH:$HOME/.foundry/bin"` to every subsequent command that uses `forge` or `cast`. Do NOT ask the user to reinstall — just fix the PATH.

**Install Pinata SDK:** `npm install pinata`

**Why Pinata SDK, not curl/API directly:**
- Pinata v1 API (`/pinning/pinFileToIPFS`) no longer supports folder uploads — returns error
- Pinata v3 API (`/uploads.pinata.cloud/v3/files`) uploads to **private IPFS** by default — content is NOT publicly accessible
- The `pinata` npm package provides `pinata.upload.public.file()` which uploads to **public IPFS** correctly
- The old `@pinata/sdk` package does NOT have the `.upload.public` API — it is a different, outdated package

### Optional Checks (warn but allow to proceed)

**Before checking env vars, source the .env file IN THE SAME command:**
```bash
set -a && source .env && set +a && echo "PRIVATE_KEY=$PRIVATE_KEY" && echo "PINATA_JWT=$PINATA_JWT"
```
Do NOT run `echo $PRIVATE_KEY` without sourcing first — shell state does NOT persist between Bash tool calls.

| ID | What | Needed for |
|----|------|------------|
| `private_key` | PRIVATE_KEY env var | deployment, minting, contract interaction |
| `pinata_jwt` | PINATA_JWT env var | IPFS upload (images + metadata) |
| `git` | Git | Foundry dependency management |

Note: `PHAROS_RPC_URL` is NOT an env var — the skill knows all Pharos RPC URLs internally. The user selects the network at Phase 1 Step 1.1.

### Step 0.3: Status Report

Display a clear status table to the user:

```
=== NFT Creator — Dependency Check ===

Required:
  [OK]   Python 3.10+         — Python 3.12.4
  [OK]   Pillow library       — ok
  [OK]   Foundry Forge        — forge 1.7.1
  [OK]   Foundry Cast         — cast 1.7.1
  [OK]   jq                   — jq-1.7
  [OK]   Node.js              — v20.20.0
  [OK]   Pinata SDK           — ok

Optional:
  [OK]   PRIVATE_KEY          — set (0x...abc)
  [MISS] PINATA_JWT           — not set (needed for: IPFS upload)

Network: will be selected at configuration (Pharos Testnet / Pharos Mainnet)

Result: 7/7 required OK. 1/2 optional set.
Action: Can proceed with configuration. IPFS upload will require PINATA_JWT.
```

### Step 0.4: Auto-Fix Missing Tools

If any required dependency is missing, the agent MUST present **numbered options** to the user. Do NOT ask open-ended text questions. Format:

```
Missing dependencies:
  - Foundry (forge + cast)

How would you like to proceed?
1. Install Foundry automatically (curl -L https://foundry.paradigm.xyz | bash && foundryup)
2. I already have Foundry installed — it's just not in PATH
3. I'll install it myself — continue without it for now
4. Cancel — I'll set up dependencies manually and come back
```

If the user picks option 2, check `~/.foundry/bin/forge --version` and remember to add PATH in future commands.

After fixing dependencies, re-run the check to confirm everything passes before proceeding.

### Step 0.5: Guide User to Fill Missing Secrets

If env vars (PRIVATE_KEY, PINATA_JWT) are empty, the agent MUST:

1. Show the exact path to the `.env` file: `The env file is located at: {absolute_path_to_.env}`
2. Tell the user which variables need to be filled, with detailed instructions for each:

   ```
   Some environment variables are not set. Please open the file and fill them in:

     File: /home/user/project/.env

   ─── PRIVATE_KEY ───
   Your wallet private key. You can paste it with or without the 0x prefix —
   the skill will normalize it automatically.
   How to export from MetaMask:
     1. Open MetaMask → click your account name at the top
     2. Click the three dots menu → "Account details"
     3. Click "Show private key"
     4. Enter your MetaMask password
     5. Copy the key and paste it into the file:
        PRIVATE_KEY=your_key_here

   ─── PINATA_JWT ───
   Your Pinata API JWT token for uploading files to IPFS.
   How to get it:
     1. Go to https://app.pinata.cloud and sign up / log in
     2. Open the left sidebar → click "API Keys"
     3. Click "New Key" button at the top right
     4. Key name: enter any name (e.g. "nft-builder")
     5. Permissions: leave defaults (all scopes enabled) or select only:
        - pins: pinFileToIPFS, pinJSONToIPFS
     6. Click "Create Key"
     7. You will see three values: API Key, API Secret, JWT
        → Copy ONLY the JWT (the long string) and paste it:
        PINATA_JWT=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

   Edit the file, save it, then tell me "done" to continue.

   Note: RPC URL is not needed here — you will choose the network (testnet/mainnet) during configuration.
   ```
3. **ABSOLUTELY NEVER** ask the user to paste their private key, JWT, or any secret directly into the chat. The agent must never read, display, or log any secret value.
4. **Wait** for the user to confirm they have filled in the file before proceeding.
5. After confirmation, run the following normalization steps before loading:

   **PRIVATE_KEY normalization** — the user may paste the key with or without `0x` prefix. The agent MUST:
   1. Read the `.env` file using `sed` or similar (without printing contents to chat)
   2. Check if the PRIVATE_KEY value starts with `0x` or `0X`
   3. If it does, automatically strip the prefix:
      ```bash
      sed -i 's/^PRIVATE_KEY=0[xX]\(.*\)/PRIVATE_KEY=\1/' .env
      ```
   4. Verify the key is now a valid hex string (64 characters after stripping)
   5. Do NOT display the key value at any point — only confirm "PRIVATE_KEY normalized"

   **PINATA_JWT normalization** — trim any leading/trailing whitespace or quotes:
   ```bash
   sed -i 's/^PINATA_JWT=["'\'']\(.*\)["'\'']/PINATA_JWT=\1/' .env
   ```

6. Re-load the `.env` file: `set -a && source .env && set +a`

If the user says a variable is not needed right now (e.g., PINATA_JWT for a configuration-only session), note it and continue. Remind them when they reach the phase that requires it.

### Security Rules (CRITICAL — enforce at all times)

1. **NEVER** ask the user to paste private keys, JWT tokens, or any secret in chat
2. **NEVER** display or log the value of PRIVATE_KEY, PINATA_JWT, or any secret
3. **ALWAYS** direct the user to edit the `.env` file directly on their machine
4. **ALWAYS** show the full absolute path to the `.env` file so the user can find it
5. When loading `.env`, use `source` — do not `cat` or print the file contents
6. If the user accidentally pastes a secret in chat, warn them immediately and suggest rotating that key

## Language Rule (CRITICAL — enforce at all times)

The agent MUST communicate with the user in the **same language the user uses**. This applies to:
- All questions, explanations, and instructions
- Status reports and summaries
- Error messages and troubleshooting guidance
- The dependency check report (Phase 0)

Detection: use the language of the user's most recent message. If the user switches language mid-conversation, switch accordingly. Technical terms (variable names, commands, Solidity code) remain in English regardless.

## Phase 1: Interactive Configuration

The agent MUST explain each step to the user in the user's language before asking for input. Do not just list questions — briefly explain what this step does and why it matters, then ask.

The agent MUST ask the user for each piece of information below before proceeding. Do not assume defaults.

### Question Format Rules (CRITICAL — enforce at all times)

**Selection questions** (choosing from options) MUST use **numbered lists** with the last option being "Other / Custom" for free-form input. Do NOT ask open-ended text questions when the user is choosing from a fixed set.

Correct format:
```
Choose the network:
1. Pharos Atlantic Testnet (free, for testing)
2. Pharos Mainnet (real PROS tokens)
3. Other / Custom
```

**Free-text questions** (names, descriptions, prices, addresses, weights) stay as open questions — the user needs to type a specific value.

**Confirmation questions** (summary tables, trait verification) MUST use numbered options:
```
1. Confirm — proceed with these settings
2. Change something
3. Cancel and start over
```

This applies to ALL phases — network selection, artwork mode, mint type, total supply, trait input method, layer order, summary confirmations, landing page offer, etc.

### Step 1.1: Network Selection

Explain briefly: "First, we need to choose which Pharos network to deploy on." Then present numbered options:

```
1. Pharos Atlantic Testnet (free PHRS, for testing)
2. Pharos Mainnet (real PROS tokens)
3. Other / Custom
```

Network details (agent uses internally, does NOT show in the question):

| # | Network | Chain ID | RPC URL | Currency |
|---|---------|----------|---------|----------|
| 1 | **Pharos Atlantic Testnet** | 688689 (0xa8231) | `https://atlantic.dplabs-internal.com` | PHRS |
| 2 | **Pharos Mainnet** | (auto-detected) | `https://rpc.pharos.xyz` (fallback: `https://infra.originstake.com/pharos/evm`) | PROS |

After the user chooses, the agent MUST:
1. Set `PHAROS_RPC_URL` to the selected RPC URL in the current session
2. Set `PHAROS_NETWORK` to the selected network name
3. Set `PHAROS_CHAIN_ID` to the chain ID
4. Warn the user if mainnet is selected: "You selected Mainnet — real PROS tokens will be spent on gas and minting. Are you sure?"
5. For mainnet, verify the RPC is reachable before proceeding: `cast chain-id --rpc-url $PHAROS_RPC_URL`

### Step 1.2: Collection Metadata

Explain to the user: "Now we need to set up the basic info for your collection — its name, symbol, and how many NFTs will exist."

**MANDATORY RULES — the agent MUST follow these exactly:**
1. Ask each question ONE AT A TIME. Do NOT batch multiple questions into one message.
2. Do NOT provide example values, suggestions, or prefilled options.
3. Do NOT guess, infer, or auto-generate any value from previous answers.
4. Do NOT skip a field — every single field below must be asked.
5. Do NOT show a confirmation summary with guessed values — only confirm what the user actually typed.
6. The user must type their own answer for every single field.

Questions (ask each separately, wait for answer, then ask next):

1. **Collection name** — Ask: "Enter a name for your collection:" → wait for user to type it
2. **Symbol** — Ask: "Enter a short symbol (no spaces):" → wait for user to type it. Do NOT derive it from the name.
3. **Description** — Ask: "Enter a short description for your collection:" → wait for user to type it. Do NOT generate it from the name.
4. **Base token name pattern** — How individual tokens are named. Default: `"{CollectionName} #{tokenId}"`. Ask only if the user wants to change this.

**CRITICAL:** The agent must NEVER assume or auto-fill symbol, description, or any other field based on the collection name or any other input. Every value must come from the user's keyboard.

### Step 1.3: Mint Configuration

Explain to the user: "Now we need to choose how minting will work — who can mint, whether it's free or paid, and any limits. This determines how your NFTs get distributed."

**IMPORTANT: The agent MUST present ALL SIX options below. Never skip or hide any of them.** The agent must show them as a numbered/lettered list that the user can choose from:

Present numbered options:
```
1. Public Free Mint — anyone can mint for free
2. Public Paid Mint — anyone pays a set price per NFT
3. Owner-Only Mint — only you (contract owner) can mint
4. Whitelist Free Mint — only whitelisted addresses mint for free
5. Whitelist Paid Mint — only whitelisted addresses pay to mint
6. Whitelist Free + Public Paid — whitelisted mint free, everyone else pays
7. Other / Custom
```

Then ask follow-up questions based on the chosen type. **Ask each question directly. Do NOT suggest or prefilled values:**
- **Mint price** (only for types 2, 5, 6) — Ask: "Enter the mint price per NFT:" (in native token — PHRS for testnet, PROS for mainnet). Explain what this means if needed.
- **Max per wallet** (for types 1, 2, 3, 4, 5) — Ask: "Enter max NFTs per wallet:" Explain this prevents one user from taking too many.
- **Public mint start** (for types 1, 2, 5) — Ask: "Enable minting immediately after deploy, or start paused?" Explain that starting paused gives time to verify the contract first.
- **Whitelist addresses** (only for types 4, 5, 6) — Ask: "Provide whitelist addresses — paste them inline or give a file path:" Explain that a Merkle tree will be generated from these addresses.

### Step 1.4: Artwork Mode Selection

Explain briefly: "Now we need to decide how your NFT artwork works." Present numbered options:

```
1. Generative (Traits) — multiple trait layers combined randomly, each NFT unique
2. Individual Images — folder of pre-made PNGs, you assign rarity to each
3. Single Image — one image for all NFTs (membership passes, tickets)
4. Other / Custom
```

### Step 1.4.1: Total Supply (after artwork mode is chosen)

Explain briefly: "Now we need to decide how many NFTs to create." Present numbered options:

```
1. 100 NFTs
2. 1,000 NFTs
3. 10,000 NFTs
4. Custom amount (max 10,000)
```

**Pinata Free Plan limits:**
- **Mode A:** 2N files (N composite images + N metadata). Max ~250 NFTs on free plan.
- **Mode B:** M + N files (M unique images + N metadata). M images can repeat across NFTs. Max NFTs = 500 - M (e.g. 5 images → max 495 NFTs, 250 images → max 250 NFTs).
- **Mode C:** N+1 files (1 image + N metadata). Max ~500 NFTs on free plan.

**If the chosen amount exceeds the Pinata free plan limit:** warn the user with the exact calculation and offer numbered options:
```
With {M} images and {N} NFTs you need {M+N} files on Pinata. Free plan limit is 500.
1. Reduce collection to {500-M} NFTs (fits free plan)
2. Reduce unique images to fit (needs {500-N} or fewer)
3. Proceed anyway (I'll upgrade my Pinata plan or use a different account)
4. Cancel and rethink
```

**If Mode A and unique combinations < total supply:** warn: "You have {combos} unique trait combinations but {total} NFTs — {duplicates} will have identical traits. This is normal for large collections."

### Step 1.4A: Trait System Design (only if user chose Option A)

Explain to the user: "Your collection is built from layers (like background, body, headwear). Each layer has variants with different rarity — some are common, others rare. Images are composed off-chain from your trait layers."

First, ask: **"Enter image dimensions (width × height in pixels):"** — e.g., "500x500" or "1000x1000". All trait layer PNGs must match these dimensions.

Then explain the trait system briefly:
- A collection is made of **layers** (e.g., Background, Body, Eyes, Headwear, Weapon)
- Each layer has **variants** (e.g., Headwear layer has Crown, Cap, Helmet, None)
- Each variant has a **weight** that determines rarity. Higher weight = more common. Weights are relative (Crown=10, Cap=30 → Cap appears 3x more often).
- **Layer order matters** — layers are stacked bottom-to-top. Background first, then body, then accessories on top.

#### Trait input method

Ask the user with numbered options:
```
How would you like to provide trait data?
1. Folder structure (recommended) — PNGs organized in layer folders
2. JSON config file — provide a config with layers, variants, weights
3. Enter manually in chat
4. Other / Custom
```
Organize trait PNGs in a folder hierarchy — each subfolder is a layer, each file is a variant:
```
traits/
├── Background/
│   ├── blue.png
│   ├── red.png
│   └── green.png
├── Body/
│   ├── dark.png
│   ├── light.png
│   └── alien.png
├── Headwear/
│   ├── crown.png
│   ├── cap.png
│   └── none.png    ← transparent PNG for optional layers
```
Ask: **"Enter the path to your traits folder:"**

Scan the folder. For each subfolder: layer name = folder name, variants = filenames (without `.png` extension). Report what was found.

**Option 2 — JSON config file:**
Provide this template — user fills it and points to the file:
```json
{
  "dimensions": "512x512",
  "layers": [
    {
      "name": "Background",
      "required": true,
      "variants": [
        {"name": "blue", "file": "traits/Background/blue.png", "weight": 30},
        {"name": "red", "file": "traits/Background/red.png", "weight": 10},
        {"name": "green", "file": "traits/Background/green.png", "weight": 50}
      ]
    },
    {
      "name": "Headwear",
      "required": false,
      "variants": [
        {"name": "crown", "file": "traits/Headwear/crown.png", "weight": 5},
        {"name": "none", "file": "traits/Headwear/none.png", "weight": 60}
      ]
    }
  ]
}
```
Ask: **"Enter the path to your traits-config.json:"**

Read and parse the file. Validate structure (required fields: name, variants with name + file + weight).

#### After collecting trait data (either method):

**Step 1 — Confirm layer order.** Layers stack bottom-to-top in the composite image. Display the detected layers and ask with numbered options:
```
Layer stacking order (bottom to top):
  1. Background → 2. Body → 3. Headwear → 4. Weapon

Is this order correct?
1. Yes, proceed
2. No — let me specify the correct order
```
If the user picks 2, ask them to enter layer names in correct order, comma-separated.

**Step 2 — Configure layers.** For each layer (in the confirmed order), ask with numbered options:
```
Is [LayerName] required (every NFT must have it)?
1. Required — every NFT gets a variant from this layer
2. Optional — some NFTs may skip this layer (needs a transparent "none" variant)
```

For weights, ask: **"Enter weights for [Layer] variants (comma-separated, same order as listed above):"**
Guide: common=50-100, uncommon=20-49, rare=5-19, legendary=1-4.

If JSON config was used and weights are already provided, just show them and confirm.

**Step 3 — Summary.** Display a table showing:
- Each layer (in stacking order) with its variants and weights
- Whether each layer is required or optional
- Theoretical max unique combinations (product of all variant counts)
- Pinata limit warning if 2 × totalSupply > 500
- Confirmation with numbered options:
```
1. Confirm — proceed with these traits
2. Change something
3. Cancel and start over
```

**Important:** All trait images MUST be the same dimensions (matching the value entered earlier). Verify this before proceeding. Reject mismatched dimensions with a clear error.

### Step 1.4B: Individual Images Setup (only if user chose Option B)

Explain to the user: "You have a set of pre-made images. Each image can be assigned to multiple NFTs based on rarity weights. Higher weight = higher chance of being assigned to a token."

Ask the user:
1. **Folder path** — path to the folder containing PNG files. Expected: one PNG per NFT, named however they want (e.g., `001_samurai.png`, `002_ninja.png`, ...).
2. For each image (or ask the user to provide a config file):
   - **Rarity weight** (integer, 1-100). Same scale as traits: common=50-100, uncommon=20-49, rare=5-19, legendary=1-4.
   - **Display name** (optional) — if not provided, derive from filename (remove extension, replace underscores with spaces).

Alternatively, accept a JSON or CSV config file mapping filenames to weights:
```json
[
  {"file": "001_samurai.png", "weight": 10, "name": "Samurai"},
  {"file": "002_ninja.png", "weight": 30, "name": "Ninja"},
  {"file": "003_wizard.png", "weight": 5, "name": "Wizard"}
]
```

Or CSV:
```
file,weight,name
001_samurai.png,10,Samurai
002_ninja.png,30,Ninja
003_wizard.png,5,Wizard
```

**Weight interpretation for individual images:** When minting, each NFT gets a random image assigned based on weights. If there are 10 images and total weights sum to 200, an image with weight 10 has a 10/200 = 5% chance per mint.

After collecting all data, display a summary:
- Total number of unique images
- Rarity breakdown (legendary, rare, uncommon, common)
- How many NFTs will be minted (totalSupply)
- Confirmation prompt

**Important:** All images MUST be the same dimensions and PNG format. Verify this before proceeding.

### Step 1.4C: Single Image Setup (only if user chose Option C)

Explain to the user: "All NFTs in this collection will share the same image. This is great for membership passes, event tickets, or utility tokens where artwork doesn't need to vary."

Ask the user:
1. **Image file path** — path to a single PNG file. Must be PNG format.
2. Confirm: "All {totalSupply} NFTs will use this image. Continue?"

No rarity system needed — every token is identical visually.

### Step 1.5: IPFS Configuration

Explain to the user: "Your trait layer images (or individual images) need to be uploaded to IPFS so the smart contract can reference them. We use Pinata SDK for public IPFS uploads."

Verify that:
1. **PINATA_JWT** is set in `.env` — if missing, guide the user to pinata.cloud to sign up and create an API key
2. **Pinata SDK** is installed — if missing, run `npm install pinata`

The JWT is already loaded from `.env` during Phase 0.

## Phase 2: Asset Preparation

### Step 2.1: Create Working Directory Structure

Create the following directory structure relative to the current project:

```
nft-collection/
├── traits/          # User places trait PNGs here (layer-name/variant-name.png)
├── images/          # User places individual images here (Mode B), or single image (Mode C)
├── composed/        # Generated: composite PNG images (Mode A) or copies (Mode B/C)
├── metadata/        # Generated: JSON metadata files (named by tokenId, no extension)
├── src/             # Generated: Solidity contract (Foundry)
├── landing/         # Generated: minting web page (Phase 5)
│   ├── index.html
│   ├── app.js
│   ├── style.css
│   ├── ethers.umd.min.js  # Downloaded locally (no CDN)
│   └── config.js          # Auto-generated from state.json
├── trait-assignments.json   # Generated: trait index assignments per token (Mode A)
├── image-assignments.json   # Generated: image index assignments per token (Mode B)
├── image-cids.json          # Generated: IPFS CIDs for uploaded images
├── merkle-data.json         # Generated: Merkle root + proofs (whitelist mint types)
└── state.json               # Local database: config, deployment info, CIDs
```

Save the initial configuration to `nft-collection/state.json`:

```json
{
  "collection": {
    "name": "",
    "symbol": "",
    "description": "",
    "totalSupply": 0,
    "baseNamePattern": ""
  },
  "artworkMode": "generative",
  "imageDimensions": {
    "width": 0,
    "height": 0
  },
  "mint": {
    "type": "",
    "price": "0",
    "maxPerWallet": 1,
    "publicMintEnabled": false,
    "whitelistAddresses": []
  },
  "traits": {
    "layers": []
  },
  "individualImages": {
    "images": []
  },
  "singleImage": {
    "path": ""
  },
  "ipfs": {
    "imageCids": {},
    "folderCid": ""
  },
  "deployment": {
    "contractAddress": "",
    "txHash": "",
    "network": "",
    "blockNumber": 0,
    "verified": false
  },
  "holders": [],
  "status": "configuring"
}
```

Set `artworkMode` to `"generative"`, `"individual"`, or `"single"` based on the user's choice in Step 1.4.

### Step 2.2: Validate Images

Validation depends on the artwork mode:

**Mode A (Generative):** For each layer and variant, verify:
1. The image file exists at the specified path
2. All images are the same dimensions (width x height)
3. All images are PNG format
4. All images use RGBA color mode (transparent backgrounds for layering)

**Mode B (Individual):** For each image in the folder, verify:
1. The image file exists
2. All images are the same dimensions
3. All images are PNG format
4. Weight is provided for each image

**Mode C (Single):** Verify:
1. The single image file exists
2. It is PNG format

If any check fails, report the exact file and issue to the user and stop.

### Step 2.3A: Compose Images + Generate Assignments (Mode A — Generative)

Compose trait layers into final PNG images off-chain, then generate trait assignments.

**Why off-chain:** On-chain SVG with `<image href="ipfs://...">` tags does NOT render in blockchain explorers — they block external resource loading for security. Composite PNG images uploaded to IPFS are the working approach.

Step 1 — Generate trait assignments and compose images:

```python
#!/usr/bin/env python3
"""Generate trait assignments and compose images for generative NFTs."""

import json, random
from pathlib import Path
from PIL import Image

with open("nft-collection/state.json") as f:
    config = json.load(f)

TOTAL_SUPPLY = config["collection"]["totalSupply"]
LAYERS = config["traits"]["layers"]

COMPOSED_DIR = Path("nft-collection/composed")
COMPOSED_DIR.mkdir(parents=True, exist_ok=True)

layer_choices = []
for layer in LAYERS:
    names = [v["name"] for v in layer["variants"]]
    weights = [v["weight"] for v in layer["variants"]]
    paths = [v["imagePath"] for v in layer["variants"]]
    layer_choices.append((names, weights, paths))

assignments = {}
used_combos = set()

for token_id in range(1, TOTAL_SUPPLY + 1):
    selected_indices = []
    selected_names = []
    selected_paths = []

    for names, weights, paths in layer_choices:
        idx = random.choices(range(len(names)), weights=weights, k=1)[0]
        selected_indices.append(idx)
        selected_names.append(names[idx])
        selected_paths.append(paths[idx])

    combo = tuple(selected_names)
    used_combos.add(combo)

    # Compose image: stack layers bottom-to-top using Pillow
    result = Image.open(selected_paths[0]).convert("RGBA")
    for path in selected_paths[1:]:
        layer = Image.open(path).convert("RGBA")
        result = Image.alpha_composite(result, layer)
    result.save(COMPOSED_DIR / f"{token_id}.png")

    assignments[str(token_id)] = {
        "traitIndices": selected_indices,
        "traitNames": selected_names
    }

    if token_id % 50 == 0:
        print(f"Composed {token_id}/{TOTAL_SUPPLY} images...")

with open("nft-collection/trait-assignments.json", "w") as f:
    json.dump(assignments, f, indent=2)

print(f"\nDone! Composed {TOTAL_SUPPLY} images in composed/")
print(f"Unique combinations: {len(used_combos)}/{TOTAL_SUPPLY}")
```

Execute this script. Output: `composed/1.png`, `composed/2.png`, ... and `trait-assignments.json`.

Update `state.json` status to `"images_composed"`.

### Step 2.3B: Assign Images to Tokens (Mode B — Individual)

Copy individual images to `composed/` directory (renamed by tokenId), then generate assignments.

```python
#!/usr/bin/env python3
"""Assigns individual images to tokens based on rarity weights."""

import json, random, shutil
from pathlib import Path

with open("nft-collection/state.json") as f:
    config = json.load(f)

TOTAL_SUPPLY = config["collection"]["totalSupply"]
IMAGES = config["individualImages"]["images"]

COMPOSED_DIR = Path("nft-collection/composed")
COMPOSED_DIR.mkdir(parents=True, exist_ok=True)

names = [img["name"] for img in IMAGES]
weights = [img["weight"] for img in IMAGES]
paths = [img["path"] for img in IMAGES]

assignments = {}
for token_id in range(1, TOTAL_SUPPLY + 1):
    idx = random.choices(range(len(names)), weights=weights, k=1)[0]
    shutil.copy2(paths[idx], COMPOSED_DIR / f"{token_id}.png")
    assignments[str(token_id)] = {"imageIndex": idx, "imageName": names[idx]}

with open("nft-collection/image-assignments.json", "w") as f:
    json.dump(assignments, f, indent=2)

from collections import Counter
counts = Counter(v["imageName"] for v in assignments.values())
print(f"Done! Copied {TOTAL_SUPPLY} images to composed/.")
for name, count in counts.most_common():
    print(f"  {name}: {count} ({count*100//TOTAL_SUPPLY}%)")
```

Update `state.json` status to `"images_composed"`.

### Step 2.3C: Prepare Single Image (Mode C — Single Image)

Copy the single image. All tokens will reference it.

```python
#!/usr/bin/env python3
import shutil
from pathlib import Path
import json

with open("nft-collection/state.json") as f:
    config = json.load(f)

COMPOSED_DIR = Path("nft-collection/composed")
COMPOSED_DIR.mkdir(parents=True, exist_ok=True)
shutil.copy2(config["singleImage"]["path"], COMPOSED_DIR / "1.png")
print("Done! Single image copied to composed/1.png.")
```

Update `state.json` status to `"image_ready"`.

### Step 2.4: Rarity Analysis

Applies to Mode A (Generative) and Mode B (Individual). Skip for Mode C (Single Image).

After generation, compute and display rarity statistics:
- **Mode A:** For each layer, the percentage occurrence of each variant. The rarest and most common tokens based on combined trait probability. A "uniqueness score" for each token.
- **Mode B:** Count of how many tokens received each image. Percentage distribution. Rarity labels (Legendary/Rare/Uncommon/Common).

Present the top 10 rarest tokens to the user.

## Phase 3: IPFS Upload

Explain to the user: "Now we upload the images to IPFS via Pinata, generate metadata JSON files, and upload the metadata folder. The contract will use a baseURI pointing to the metadata folder."

### Pinata Upload Method

**CRITICAL — Use the correct tools:**
- **Individual file uploads:** Use `pinata` npm package (NOT `@pinata/sdk`) with `pinata.upload.public.file()`
- **Folder uploads:** Use Pinata v1 REST API (`https://api.pinata.cloud/pinning/pinFileToIPFS`) with FormData — the SDK has no folder method
- **Pinata v3 API** uploads to **private IPFS** — do NOT use it
- **The old `@pinata/sdk`** package is a different, outdated package without `.upload.public` — do NOT use it

**Pinata Free Plan Limits:**
- **500 pinned files** total
- **1 GB storage** total
- **Pinata deduplicates by CID** — if multiple files have identical content, they share one CID and count as 1 pin. This is beneficial: Mode A with many duplicate composites will use fewer pins than expected.

Files to upload per mode (worst case — all unique):
- **Mode A:** N composite PNGs + N metadata JSONs = 2N files. Max ~250 NFTs on free plan. With duplicates, actual pins may be much less.
- **Mode B:** M unique images + N metadata JSONs = M+N files (M images can repeat across tokens). Max NFTs = 500-M.
- **Mode C:** 1 image + N metadata JSONs = N+1 files. Max ~500 NFTs on free plan.

**Metadata folder upload:** REST API `POST /pinning/pinFileToIPFS` with FormData works on free plan. Each metadata file is unique (different tokenId, attributes), so no deduplication benefit — all N files count as N pins.

### Step 3.1: Upload Images to IPFS

Upload each image from `nft-collection/composed/` to IPFS via Pinata SDK:

```javascript
// upload-images.mjs
import { PinataSDK } from "pinata";
import fs from "fs";

const pinata = new PinataSDK({
    pinataJwt: process.env.PINATA_JWT,
    pinataGateway: "gateway.pinata.cloud"
});

const config = JSON.parse(fs.readFileSync("nft-collection/state.json"));
const totalSupply = config.collection.totalSupply;
const mode = config.artworkMode;
const imageCids = {};

// Mode C: only 1 image for all tokens
const count = mode === "single" ? 1 : totalSupply;

for (let i = 1; i <= count; i++) {
    const buffer = fs.readFileSync(`nft-collection/composed/${i}.png`);
    const file = new File([buffer], `${i}.png`, { type: "image/png" });
    const result = await pinata.upload.public.file(file);
    imageCids[i] = result.cid;
    console.log(`[${i}/${count}] Image ${i}: ${result.cid}`);
}

fs.writeFileSync("nft-collection/image-cids.json", JSON.stringify(imageCids, null, 2));
console.log(`\nDone! Uploaded ${count} images.`);
```

Run: `set -a && source .env && set +a && node upload-images.mjs`

### Step 3.2: Generate Metadata JSON Files

Generate ERC-721 metadata JSON files for each token. Files are named by tokenId (no extension) and placed in `nft-collection/metadata/`.

```python
#!/usr/bin/env python3
"""Generate metadata JSON files for NFT collection."""

import json
from pathlib import Path

with open("nft-collection/state.json") as f:
    config = json.load(f)
with open("nft-collection/image-cids.json") as f:
    image_cids = json.load(f)

TOTAL = config["collection"]["totalSupply"]
NAME = config["collection"]["name"]
DESC = config["collection"]["description"]
PATTERN = config["collection"].get("baseNamePattern", f"{NAME} #{{}}")
MODE = config["artworkMode"]

META_DIR = Path("nft-collection/metadata")
META_DIR.mkdir(parents=True, exist_ok=True)

for token_id in range(1, TOTAL + 1):
    # Mode C: all tokens use image 1
    img_cid = image_cids["1"] if MODE == "single" else image_cids[str(token_id)]

    token_name = PATTERN.replace("{}", str(token_id)) if "{}" in PATTERN else PATTERN

    metadata = {
        "name": token_name,
        "description": DESC,
        "image": f"ipfs://{img_cid}",
        "attributes": []
    }

    # Mode A: add trait attributes from trait-assignments.json
    if MODE == "generative":
        assignments_file = Path("nft-collection/trait-assignments.json")
        if assignments_file.exists():
            assignments = json.loads(assignments_file.read_text())
            if str(token_id) in assignments:
                layers = config["traits"]["layers"]
                trait_names = assignments[str(token_id)]["traitNames"]
                for layer, trait in zip(layers, trait_names):
                    metadata["attributes"].append({
                        "trait_type": layer["name"],
                        "value": trait
                    })

    # Mode B: add image name as attribute
    elif MODE == "individual":
        assignments_file = Path("nft-collection/image-assignments.json")
        if assignments_file.exists():
            assignments = json.loads(assignments_file.read_text())
            if str(token_id) in assignments:
                metadata["attributes"].append({
                    "trait_type": "Image",
                    "value": assignments[str(token_id)]["imageName"]
                })

    # Write metadata file (no extension, name = tokenId)
    (META_DIR / str(token_id)).write_text(json.dumps(metadata, indent=2))

print(f"Done! Generated {TOTAL} metadata files in metadata/")
```

Run: `python3 generate-metadata.py`

### Step 3.3: Upload Metadata Folder to IPFS

Upload the `metadata/` folder as a virtual directory to IPFS via Pinata REST API. This creates a folder CID where files are accessible as `ipfs://CID/1`, `ipfs://CID/2`, etc.

**CRITICAL:** File names in FormData MUST include the `metadata/` prefix to create a proper folder structure in IPFS.

```javascript
// upload-metadata-folder.mjs
import fs from "fs";
import path from "path";

const PINATA_JWT = process.env.PINATA_JWT;
const config = JSON.parse(fs.readFileSync("nft-collection/state.json"));
const totalSupply = config.collection.totalSupply;

const formData = new FormData();
for (let i = 1; i <= totalSupply; i++) {
    const content = fs.readFileSync(`nft-collection/metadata/${i}`);
    formData.append("file", new Blob([content]), `metadata/${i}`);
}

console.log(`Uploading metadata folder (${totalSupply} files)...`);
const resp = await fetch("https://api.pinata.cloud/pinning/pinFileToIPFS", {
    method: "POST",
    headers: { Authorization: `Bearer ${PINATA_JWT}` },
    body: formData,
});

if (!resp.ok) {
    const err = await resp.text();
    console.error(`Upload failed: ${resp.status} ${err}`);
    process.exit(1);
}

const { IpfsHash } = await resp.json();
console.log(`\nFolder CID: ${IpfsHash}`);
console.log(`Verify: https://gateway.pinata.cloud/ipfs/${IpfsHash}/1`);

// Update state.json
config.ipfs.folderCid = IpfsHash;
config.status = "ipfs_uploaded";
fs.writeFileSync("nft-collection/state.json", JSON.stringify(config, null, 2));
```

Run: `set -a && source .env && set +a && node upload-metadata-folder.mjs`

After upload, verify the metadata is accessible:
```bash
curl -s https://gateway.pinata.cloud/ipfs/FOLDER_CID/1 | jq .
```

Record the folder CID in `state.json`. This CID becomes the `baseURI` for the smart contract: `ipfs://FOLDER_CID/`.

## Phase 4: Smart Contract

### Step 4.1: Generate Solidity Contract

Create a Foundry project and generate the NFT contract at `nft-collection/src/PharosNFTCollection.sol`:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/utils/Strings.sol";
import "@openzeppelin/contracts/utils/cryptography/MerkleProof.sol";

contract PharosNFTCollection is ERC721, Ownable {
    using Strings for uint256;

    uint256 public maxSupply;
    uint256 public mintPrice;
    uint256 public maxPerWallet;
    uint8 public mintType; // 0=owner, 1=public_free, 2=public_paid, 3=wl_free, 4=wl_paid, 5=wl_free+public_paid
    bool public publicMintEnabled;
    bytes32 public merkleRoot;
    string private _baseTokenURI;

    uint256 private _nextTokenId = 1;
    mapping(address => uint256) private _mintedCount;

    constructor(
        string memory name_,
        string memory symbol_,
        uint256 maxSupply_,
        uint256 mintPrice_,
        uint256 maxPerWallet_,
        uint8 mintType_,
        bytes32 merkleRoot_
    ) ERC721(name_, symbol_) Ownable(msg.sender) {
        maxSupply = maxSupply_;
        mintPrice = mintPrice_;
        maxPerWallet = maxPerWallet_;
        mintType = mintType_;
        merkleRoot = merkleRoot_;
        publicMintEnabled = (mintType_ == 1 || mintType_ == 2);
    }

    // ─── tokenURI ────────────────────────────────────────────────────

    function tokenURI(uint256 tokenId) public view override returns (string memory) {
        require(_ownerOf(tokenId) != address(0), "Token does not exist");
        return string(abi.encodePacked(_baseTokenURI, tokenId.toString()));
    }

    // ─── Public Mint ─────────────────────────────────────────────────
    // Types 1, 2: public only. Type 5: public paid (whitelist uses whitelistMint for free)

    function mint(uint256 quantity) external payable {
        require(mintType == 1 || mintType == 2 || mintType == 5, "Not a public mint collection");
        require(publicMintEnabled, "Mint is paused");
        if (mintType == 2 || mintType == 5) {
            require(msg.value >= mintPrice * quantity, "Insufficient payment");
        }
        _validateAndMint(quantity);
    }

    // ─── Whitelist Mint ──────────────────────────────────────────────
    // Leaf = keccak256(abi.encodePacked(msg.sender)) — 20 raw bytes, NO padding
    // Proof = sibling nodes in Merkle tree (NOT the leaf itself)
    // Type 5: free for whitelisted (public pays via mint())

    function whitelistMint(uint256 quantity, bytes32[] calldata proof) external payable {
        require(mintType == 3 || mintType == 4 || mintType == 5, "Not a whitelist mint collection");
        require(publicMintEnabled, "Mint is paused");
        require(
            MerkleProof.verify(proof, merkleRoot, keccak256(abi.encodePacked(msg.sender))),
            "Not whitelisted"
        );
        if (mintType == 4) {
            require(msg.value >= mintPrice * quantity, "Insufficient payment");
        }
        _validateAndMint(quantity);
    }

    // ─── Owner Functions ─────────────────────────────────────────────

    function ownerMint(address to, uint256 quantity) external onlyOwner {
        require(quantity > 0, "Quantity must be > 0");
        require(_nextTokenId + quantity - 1 <= maxSupply, "Exceeds max supply");
        for (uint256 i = 0; i < quantity; i++) {
            _safeMint(to, _nextTokenId++);
        }
    }

    function batchMintToAddresses(address[] calldata recipients) external onlyOwner {
        require(recipients.length > 0, "No recipients");
        require(_nextTokenId + recipients.length - 1 <= maxSupply, "Exceeds max supply");
        for (uint256 i = 0; i < recipients.length; i++) {
            _safeMint(recipients[i], _nextTokenId++);
        }
    }

    function airdrop(address[] calldata recipients, uint256[] calldata amounts) external onlyOwner {
        require(recipients.length == amounts.length, "Array length mismatch");
        uint256 total = 0;
        for (uint256 i = 0; i < amounts.length; i++) {
            total += amounts[i];
        }
        require(_nextTokenId + total - 1 <= maxSupply, "Exceeds max supply");
        for (uint256 i = 0; i < recipients.length; i++) {
            for (uint256 j = 0; j < amounts[i]; j++) {
                _safeMint(recipients[i], _nextTokenId++);
            }
        }
    }

    // ─── Internal ────────────────────────────────────────────────────

    function _validateAndMint(uint256 quantity) internal {
        require(quantity > 0, "Quantity must be > 0");
        require(_nextTokenId + quantity - 1 <= maxSupply, "Exceeds max supply");
        require(_mintedCount[msg.sender] + quantity <= maxPerWallet, "Exceeds max per wallet");
        _mintedCount[msg.sender] += quantity;
        for (uint256 i = 0; i < quantity; i++) {
            _safeMint(msg.sender, _nextTokenId++);
        }
    }

    // ─── Admin ───────────────────────────────────────────────────────

    function setBaseURI(string memory uri) external onlyOwner {
        _baseTokenURI = uri;
    }

    function togglePublicMint() external onlyOwner {
        publicMintEnabled = !publicMintEnabled;
    }

    function setMintPrice(uint256 price) external onlyOwner {
        mintPrice = price;
    }

    function setMaxPerWallet(uint256 max) external onlyOwner {
        maxPerWallet = max;
    }

    function setMerkleRoot(bytes32 root) external onlyOwner {
        merkleRoot = root;
    }

    function withdraw() external onlyOwner {
        (bool success,) = payable(owner()).call{value: address(this).balance}("");
        require(success, "Withdraw failed");
    }

    // ─── View Functions ──────────────────────────────────────────────

    function totalMinted() external view returns (uint256) {
        return _nextTokenId - 1;
    }

    function mintedBy(address account) external view returns (uint256) {
        return _mintedCount[account];
    }

    function remainingSupply() external view returns (uint256) {
        return maxSupply - (_nextTokenId - 1);
    }
}
```

### Step 4.2: Initialize Foundry Project

```bash
cd nft-collection
forge init --no-commit --force
forge install https://github.com/OpenZeppelin/openzeppelin-contracts@v5.0.2 --no-git
```

Place the contract at `src/PharosNFTCollection.sol`.

Create `foundry.toml`:

```toml
[profile.default]
src = "src"
out = "out"
libs = ["lib"]
solc_version = "0.8.28"
optimizer = true
optimizer_runs = 200
via_ir = true

remappings = [
    "@openzeppelin/=lib/openzeppelin-contracts/"
]
```

**Note:** `via_ir = true` is **required** — the contract is large enough to trigger "Stack too deep" without it.

### Step 4.3: Compile

```bash
forge build
```

If compilation fails, debug and fix. Common issues:
- OpenZeppelin version mismatch — ensure v5.x is installed
- Import paths — check remappings in `foundry.toml`

### Step 4.4: Deploy

Determine the Pharos RPC URL from the user's context (testnet or mainnet). Deploy using `forge create` (always include `--broadcast` — without it, forge does a dry run and does NOT actually deploy). **Pharos requires `--legacy` flag** for correct transaction type.

Constructor args: `name_, symbol_, maxSupply_, mintPrice_, maxPerWallet_, mintType_, merkleRoot_`

```bash
# Example: public paid mint (mintType=2), no whitelist (bytes32 zero)
set -a && source .env && set +a && forge create src/PharosNFTCollection.sol:PharosNFTCollection \
  --rpc-url $PHAROS_RPC_URL \
  --private-key $PRIVATE_KEY \
  --broadcast \
  --legacy \
  --constructor-args \
    "CollectionName" "SYMBOL" 1000 "10000000000000000" 5 2 \
    0x0000000000000000000000000000000000000000000000000000000000000000

# Example: whitelist paid mint (mintType=4), with Merkle root
set -a && source .env && set +a && forge create src/PharosNFTCollection.sol:PharosNFTCollection \
  --rpc-url $PHAROS_RPC_URL \
  --private-key $PRIVATE_KEY \
  --broadcast \
  --legacy \
  --constructor-args \
    "CollectionName" "SYMBOL" 1000 "10000000000000000" 2 4 \
    0xYOUR_MERKLE_ROOT

# Example: owner-only mint (mintType=0), free
set -a && source .env && set +a && forge create src/PharosNFTCollection.sol:PharosNFTCollection \
  --rpc-url $PHAROS_RPC_URL \
  --private-key $PRIVATE_KEY \
  --broadcast \
  --legacy \
  --constructor-args \
    "CollectionName" "SYMBOL" 1000 0 1 0 \
    0x0000000000000000000000000000000000000000000000000000000000000000
```

**Constructor args mapping:**
1. `name_` — Collection name
2. `symbol_` — Collection symbol
3. `maxSupply_` — Total supply
4. `mintPrice_` — Mint price in wei (0 for free)
5. `maxPerWallet_` — Max mints per wallet
6. `mintType_` — 0 (owner), 1 (public free), 2 (public paid), 3 (whitelist free), 4 (whitelist paid), 5 (whitelist free + public paid)
7. `merkleRoot_` — Merkle root hash (bytes32 zero for non-whitelist types)

**Mint type mapping (UI → contract → behavior):**

| mintType | Name | `mint()` | `whitelistMint()` | Merkle Root |
|----------|------|----------|--------------------|----|
| 0 | Owner-Only | blocked | blocked | ignored |
| 1 | Public Free | free | blocked | ignored |
| 2 | Public Paid | paid | blocked | ignored |
| 3 | Whitelist Free | blocked | free (proof) | required |
| 4 | Whitelist Paid | blocked | paid (proof) | required |
| 5 | WL Free + Public Paid | paid | free (proof) | required |

Record the deployed contract address and transaction hash in `state.json`. Update status to `"deployed"`.

### Step 4.5: Set Base URI and Enable Minting

After deployment, set the base URI to point to the IPFS metadata folder, then enable minting.

```bash
# Set base URI — must end with /
set -a && source .env && set +a && cast send \
  --rpc-url $PHAROS_RPC_URL \
  --private-key $PRIVATE_KEY \
  --legacy \
  $CONTRACT \
  "setBaseURI(string)" \
  "ipfs://FOLDER_CID/"

# Enable minting
set -a && source .env && set +a && cast send \
  --rpc-url $PHAROS_RPC_URL \
  --private-key $PRIVATE_KEY \
  --legacy \
  $CONTRACT \
  "togglePublicMint()"
```

**Verify:**
```bash
# Check tokenURI returns correct path
set -a && source .env && set +a && cast call \
  --rpc-url $PHAROS_RPC_URL \
  $CONTRACT \
  "tokenURI(uint256)(string)" 1
# Expected: "ipfs://FOLDER_CID/1"

# Verify metadata is accessible
curl -s https://gateway.pinata.cloud/ipfs/FOLDER_CID/1 | jq .
```

Update `state.json` status to `"setup_complete"`.

### Step 4.6: Verify Contract

```bash
set -a && source .env && set +a && forge verify-contract \
  --rpc-url $PHAROS_RPC_URL \
  --contract src/PharosNFTCollection.sol:PharosNFTCollection \
  --constructor-args $(cast abi-encode "constructor(string,string,uint256,uint256,uint256,uint8,bytes32)" "CollectionName" "SYMBOL" 1000 0 5 2 0x0000000000000000000000000000000000000000000000000000000000000000) \
  --verifier-url <PHAROS_BLOCK_EXPLORER_API> \
  $CONTRACT
```

Update `state.json` verified field to `true`. Update status to `"verified"`.

## Phase 5: Landing Page (Optional)

Ask the user with numbered options:
```
Would you like a minting web page for your collection?
1. Yes — create a landing page
2. No — skip to minting
```

If yes, proceed. If no, skip to Phase 6.

### Step 5.1: Create Landing Directory

```bash
mkdir -p nft-collection/landing
```

### Step 5.2: Download ethers.js Locally

**Never use CDN links** — `cdn.ethers.io` is often unreachable, causing `ethers is not defined` in browser console. Download locally.

Try these sources in order:

```bash
# Option 1: cdnjs
curl -sL "https://cdnjs.cloudflare.com/ajax/libs/ethers/5.7.2/ethers.umd.min.js" \
  -o nft-collection/landing/ethers.umd.min.js

# If cdnjs fails, try unpkg:
curl -sL "https://unpkg.com/ethers@5.7.2/dist/ethers.umd.min.js" \
  -o nft-collection/landing/ethers.umd.min.js

# If unpkg fails, try jsdelivr:
curl -sL "https://cdn.jsdelivr.net/npm/ethers@5.7.2/dist/ethers.umd.min.js" \
  -o nft-collection/landing/ethers.umd.min.js

# If all CDNs fail, use npm:
npm install ethers@5.7.2 && cp node_modules/ethers/dist/ethers.umd.min.js nft-collection/landing/
```

Verify the file is non-empty: `ls -la nft-collection/landing/ethers.umd.min.js`

### Step 5.3: Generate config.js

Read `state.json` and `nft-collection/merkle-data.json` (if exists). Generate `config.js` automatically — the user should NOT edit this file manually.

```javascript
// ============================================================
// NFT MINTING PAGE — CONFIGURATION
// Auto-generated from state.json. Do not edit manually.
// ============================================================

const CONFIG = {
  contractAddress: "0xYOUR_CONTRACT_ADDRESS",

  network: {
    chainIdHex: "0xA8231",       // Pharos testnet
    chainId: 688689,
    chainName: "Pharos Atlantic Testnet",
    rpcUrl: "https://atlantic.dplabs-internal.com",
    currencyName: "PHRS",
    currencySymbol: "PHRS",
    currencyDecimals: 18,
    blockExplorer: "https://atlantic.pharosscan.com",
  },

  // For whitelist mint (mintType 3, 4): fill from merkle-data.json
  // For public mint (mintType 1, 2): set to empty: whitelistProofs: {}
  merkleRoot: "0xROOT_OR_EMPTY_STRING",
  whitelistProofs: {
    "0xaddress_lowercase": ["0x_sibling_leaf_proof"],
  },
};
```

**Substitution table:**

| Placeholder | Source |
|---|---|
| contractAddress | `state.json` → `deployment.contractAddress` |
| network.* | From Phase 1 network selection |
| merkleRoot | `merkle-data.json` → `root` (or empty string for public mint) |
| whitelistProofs | `merkle-data.json` → `proofs` (keys must be lowercase, or `{}` for public mint) |

**Network defaults:**

Pharos Atlantic Testnet:
```
chainIdHex: "0xA8231", chainId: 688689, chainName: "Pharos Atlantic Testnet",
rpcUrl: "https://atlantic.dplabs-internal.com", currencyName: "PHRS", currencySymbol: "PHRS",
blockExplorer: "https://atlantic.pharosscan.com"
```

Pharos Mainnet:
```
chainIdHex: "0x...", chainId: ..., chainName: "Pharos Mainnet",
rpcUrl: "https://rpc.pharos.xyz", currencyName: "PROS", currencySymbol: "PROS",
blockExplorer: "https://www.pharosscan.xyz"
```

**chainIdHex calculation — CRITICAL:** Do NOT calculate hex manually — it's error-prone. Use the skill's known values for testnet (`0xA8231`) and mainnet. If the chain ID is unknown, compute it programmatically: `"0x" + chainId.toString(16)`. Always verify: the hex decoded back must equal the decimal chain ID.

### Step 5.4: Write Template Files

Write the following three files to `nft-collection/landing/`. These are static templates — no changes needed.

**index.html:**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>NFT Mint</title>
  <link rel="stylesheet" href="style.css" />
</head>
<body>
  <div id="app">
    <header>
      <h1 id="collection-name">Loading...</h1>
      <p id="collection-symbol" class="symbol"></p>
    </header>

    <main>
      <section class="stats">
        <div class="stat-box">
          <span class="stat-label">Price</span>
          <span id="mint-price" class="stat-value">—</span>
        </div>
        <div class="stat-box">
          <span class="stat-label">Minted</span>
          <span id="minted-count" class="stat-value">—</span>
        </div>
        <div class="stat-box">
          <span class="stat-label">Remaining</span>
          <span id="remaining" class="stat-value">—</span>
        </div>
        <div class="stat-box">
          <span class="stat-label">Max / Wallet</span>
          <span id="max-per-wallet" class="stat-value">—</span>
        </div>
      </section>

      <section class="mint-section">
        <div class="mint-type-badge" id="mint-type-badge">—</div>

        <div class="wallet-row">
          <button id="btn-connect" onclick="connectWallet()">Connect Wallet</button>
          <span id="wallet-address" class="mono"></span>
        </div>

        <div class="mint-row">
          <button id="btn-minus" onclick="changeQty(-1)">−</button>
          <span id="qty">1</span>
          <button id="btn-plus" onclick="changeQty(1)">+</button>
          <button id="btn-mint" onclick="mint()" disabled>Mint</button>
        </div>

        <p id="total-cost" class="mono"></p>
        <p id="status" class="status-msg"></p>
      </section>

      <section class="network-row">
        <span id="network-name">—</span>
        <button id="btn-switch-network" onclick="switchNetwork()" style="display:none">
          Switch Network
        </button>
      </section>
    </main>

    <footer>
      <p>
        Contract:
        <a id="contract-link" href="#" target="_blank" class="mono">—</a>
      </p>
    </footer>
  </div>

  <!-- ethers.js loaded locally — do NOT use CDN -->
  <script src="ethers.umd.min.js"></script>
  <script src="config.js"></script>
  <script src="app.js"></script>
</body>
</html>
```

**style.css:**

```css
:root {
  --bg: #0b0b0f;
  --surface: #14141a;
  --border: #2a2a35;
  --text: #e8e8ec;
  --dim: #7a7a8a;
  --accent: #6c5ce7;
  --accent-hover: #7f70ee;
  --success: #00cec9;
  --error: #e74c3c;
}

* { margin: 0; padding: 0; box-sizing: border-box; }

body {
  font-family: -apple-system, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
  background: var(--bg);
  color: var(--text);
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
}

#app {
  width: 100%;
  max-width: 520px;
  padding: 24px;
}

header {
  text-align: center;
  margin-bottom: 32px;
}

header h1 {
  font-size: 2rem;
  font-weight: 700;
  letter-spacing: -0.5px;
}

.symbol {
  color: var(--accent);
  font-size: 0.9rem;
  margin-top: 4px;
}

.stats {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
  margin-bottom: 28px;
}

.stat-box {
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: 10px;
  padding: 14px;
  text-align: center;
}

.stat-label {
  display: block;
  font-size: 0.75rem;
  text-transform: uppercase;
  letter-spacing: 1px;
  color: var(--dim);
  margin-bottom: 6px;
}

.stat-value {
  font-size: 1.25rem;
  font-weight: 600;
}

.mint-section {
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: 12px;
  padding: 24px;
  text-align: center;
}

.mint-type-badge {
  display: inline-block;
  background: var(--accent);
  color: #fff;
  font-size: 0.7rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 1px;
  padding: 4px 12px;
  border-radius: 20px;
  margin-bottom: 18px;
}

.wallet-row {
  margin-bottom: 18px;
}

#btn-connect {
  background: var(--accent);
  color: #fff;
  border: none;
  padding: 10px 24px;
  border-radius: 8px;
  font-size: 0.9rem;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.2s;
}

#btn-connect:hover { background: var(--accent-hover); }

#wallet-address {
  display: block;
  margin-top: 8px;
  font-size: 0.8rem;
  color: var(--dim);
}

.mint-row {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 12px;
  margin-bottom: 12px;
}

#btn-minus, #btn-plus {
  width: 40px;
  height: 40px;
  border-radius: 8px;
  border: 1px solid var(--border);
  background: var(--bg);
  color: var(--text);
  font-size: 1.2rem;
  cursor: pointer;
  transition: border-color 0.2s;
}

#btn-minus:hover, #btn-plus:hover { border-color: var(--accent); }

#qty {
  font-size: 1.4rem;
  font-weight: 700;
  min-width: 32px;
}

#btn-mint {
  background: var(--accent);
  color: #fff;
  border: none;
  padding: 12px 32px;
  border-radius: 8px;
  font-size: 1rem;
  font-weight: 700;
  cursor: pointer;
  transition: background 0.2s;
}

#btn-mint:hover:not(:disabled) { background: var(--accent-hover); }
#btn-mint:disabled { opacity: 0.4; cursor: not-allowed; }

#total-cost {
  font-size: 0.85rem;
  color: var(--dim);
  margin-bottom: 8px;
}

.status-msg {
  font-size: 0.85rem;
  min-height: 20px;
  margin-top: 8px;
}

.status-msg.error { color: var(--error); }
.status-msg.success { color: var(--success); }
.status-msg.info { color: var(--dim); }

.network-row {
  text-align: center;
  margin-top: 20px;
  font-size: 0.8rem;
  color: var(--dim);
}

#btn-switch-network {
  background: transparent;
  color: var(--accent);
  border: 1px solid var(--accent);
  padding: 6px 16px;
  border-radius: 6px;
  font-size: 0.8rem;
  cursor: pointer;
  margin-left: 8px;
}

footer {
  text-align: center;
  margin-top: 32px;
  font-size: 0.75rem;
  color: var(--dim);
}

footer a {
  color: var(--accent);
  text-decoration: none;
}

.mono { font-family: "SF Mono", "Fira Code", monospace; }
```

**app.js:**

```javascript
// ============================================================
// NFT Minting Page — App Logic
// Do not edit — all config is in config.js
// ============================================================

const ABI = [
  "function name() view returns (string)",
  "function symbol() view returns (string)",
  "function maxSupply() view returns (uint256)",
  "function mintPrice() view returns (uint256)",
  "function maxPerWallet() view returns (uint256)",
  "function publicMintEnabled() view returns (bool)",
  "function mintType() view returns (uint8)",
  "function totalMinted() view returns (uint256)",
  "function remainingSupply() view returns (uint256)",
  "function mintedBy(address) view returns (uint256)",
  "function mint(uint256) payable",
  "function whitelistMint(uint256, bytes32[]) payable",
  "function ownerMint(address, uint256)",
];

const MINT_TYPES = {
  0: "Owner Only",
  1: "Public Free",
  2: "Public Paid",
  3: "Whitelist Free",
  4: "Whitelist Paid",
  5: "WL Free + Public Paid",
};

let provider, signer, contract, userAddress;
let contractData = {};
let qty = 1;

// ─── Init ─────────────────────────────────────────────────

async function init() {
  try {
    provider = new ethers.providers.JsonRpcProvider(CONFIG.network.rpcUrl);
    contract = new ethers.Contract(CONFIG.contractAddress, ABI, provider);
    await loadContractData();
    renderStatic();
    startPolling();
    if (window.ethereum && window.ethereum.selectedAddress) {
      await connectWallet();
    }
  } catch (e) {
    setStatus("Failed to load contract: " + e.message, "error");
  }
}

// ─── Contract reads ───────────────────────────────────────

async function loadContractData() {
  const [name, symbol, maxSupply, mintPrice, maxPerWallet, mintType, totalMinted, remaining] =
    await Promise.all([
      contract.name(), contract.symbol(), contract.maxSupply(),
      contract.mintPrice(), contract.maxPerWallet(), contract.mintType(),
      contract.totalMinted(), contract.remainingSupply(),
    ]);
  contractData = {
    name, symbol,
    maxSupply: maxSupply.toNumber(),
    mintPrice: mintPrice,
    maxPerWallet: maxPerWallet.toNumber(),
    mintType: mintType,
    totalMinted: totalMinted.toNumber(),
    remaining: remaining.toNumber(),
  };
}

// ─── Render ───────────────────────────────────────────────

function renderStatic() {
  const d = contractData;
  document.getElementById("collection-name").textContent = d.name;
  document.getElementById("collection-symbol").textContent = d.symbol;
  document.getElementById("mint-type-badge").textContent = MINT_TYPES[d.mintType] || "Unknown";

  const explorer = CONFIG.network.blockExplorer;
  const link = document.getElementById("contract-link");
  link.textContent = shortAddr(CONFIG.contractAddress);
  link.href = explorer ? `${explorer}/address/${CONFIG.contractAddress}` : "#";

  document.getElementById("network-name").textContent = CONFIG.network.chainName;
  renderDynamic();
}

function renderDynamic() {
  const d = contractData;
  const priceEth = ethers.utils.formatEther(d.mintPrice);
  document.getElementById("mint-price").textContent =
    d.mintPrice.isZero() ? "FREE" : `${priceEth} ${CONFIG.network.currencySymbol}`;
  document.getElementById("minted-count").textContent = `${d.totalMinted} / ${d.maxSupply}`;
  document.getElementById("remaining").textContent = d.remaining;
  document.getElementById("max-per-wallet").textContent = d.maxPerWallet;
  updateCost();
}

function updateCost() {
  const priceWei = contractData.mintPrice.mul(qty);
  const costEth = ethers.utils.formatEther(priceWei);
  const el = document.getElementById("total-cost");
  if (contractData.mintPrice.isZero()) {
    el.textContent = qty > 1 ? `${qty} x Free` : "";
  } else {
    el.textContent = `Total: ${costEth} ${CONFIG.network.currencySymbol}`;
  }
}

// ─── Wallet ───────────────────────────────────────────────

async function connectWallet() {
  if (!window.ethereum) {
    setStatus("No wallet detected. Install Rabby or MetaMask.", "error");
    return;
  }
  try {
    setStatus("Connecting...", "info");
    provider = new ethers.providers.Web3Provider(window.ethereum);
    await provider.send("eth_requestAccounts", []);
    signer = provider.getSigner();
    userAddress = await signer.getAddress();
    contract = new ethers.Contract(CONFIG.contractAddress, ABI, signer);

    document.getElementById("wallet-address").textContent = shortAddr(userAddress);
    document.getElementById("btn-connect").textContent = "Connected";
    document.getElementById("btn-connect").disabled = true;

    const ok = await checkNetwork();
    if (ok) {
      const wlCheck = checkWhitelistStatus();
      document.getElementById("btn-mint").disabled = !wlCheck;
      await refreshMinted();
    }
    if (!document.getElementById("btn-mint").disabled) {
      setStatus("Ready to mint!", "success");
    }
  } catch (e) {
    setStatus("Connection failed: " + e.message, "error");
  }
}

async function checkNetwork() {
  const { chainId } = await provider.getNetwork();
  if (chainId === CONFIG.network.chainId) return true;
  document.getElementById("btn-switch-network").style.display = "inline-block";
  setStatus("Wrong network. Click 'Switch Network'.", "error");
  return false;
}

async function switchNetwork() {
  if (!window.ethereum) return;
  const n = CONFIG.network;
  const chainIdHex = n.chainIdHex || "0x" + n.chainId.toString(16);
  setStatus("Switching network...", "info");
  try {
    await window.ethereum.request({
      method: "wallet_switchEthereumChain",
      params: [{ chainId: chainIdHex }],
    });
  } catch (switchErr) {
    // Always try to add the chain — some wallets don't return code 4902
    try {
      await window.ethereum.request({
        method: "wallet_addEthereumChain",
        params: [{
          chainId: chainIdHex,
          chainName: n.chainName,
          rpcUrls: [n.rpcUrl],
          nativeCurrency: {
            name: n.currencyName, symbol: n.currencySymbol, decimals: n.currencyDecimals,
          },
          blockExplorerUrls: n.blockExplorer ? [n.blockExplorer] : undefined,
        }],
      });
    } catch (addErr) {
      setStatus("Failed to add network: " + (addErr.message || addErr), "error");
      return;
    }
  }
  provider = new ethers.providers.Web3Provider(window.ethereum);
  signer = provider.getSigner();
  contract = new ethers.Contract(CONFIG.contractAddress, ABI, signer);
  document.getElementById("btn-switch-network").style.display = "none";
  document.getElementById("btn-mint").disabled = false;
  setStatus("Network switched!", "success");
  await refreshMinted();
}

// ─── Mint ─────────────────────────────────────────────────

function changeQty(delta) {
  qty = Math.max(1, Math.min(qty + delta, contractData.maxPerWallet, contractData.remaining));
  document.getElementById("qty").textContent = qty;
  updateCost();
}

async function mint() {
  if (!signer) { setStatus("Connect wallet first", "error"); return; }
  const mt = contractData.mintType;
  try {
    setStatus("Confirm in wallet...", "info");
    let tx;
    const value = contractData.mintPrice.mul(qty);

    if (mt === 1 || mt === 2) {
      // Public mint (free or paid)
      tx = await contract.mint(qty, { value });
    } else if (mt === 5) {
      // Type 5: whitelisted mint free via whitelistMint(), public pays via mint()
      const proof = getWhitelistProof();
      if (proof) {
        tx = await contract.whitelistMint(qty, proof, { value: 0 });
      } else {
        tx = await contract.mint(qty, { value });
      }
    } else if (mt === 3 || mt === 4) {
      const proof = getWhitelistProof();
      if (!proof) { setStatus("Your wallet is not whitelisted.", "error"); return; }
      tx = await contract.whitelistMint(qty, proof, { value });
    } else {
      setStatus("Minting is owner-only. Use cast/CLI.", "error");
      return;
    }

    setStatus("Tx sent! Waiting for confirmation...", "info");
    await tx.wait();
    setStatus(`Minted ${qty} NFT(s)! TX: ${shortAddr(tx.hash)}`, "success");
    await refreshMinted();
  } catch (e) {
    const msg = e.reason || e.message || "Unknown error";
    setStatus("Mint failed: " + msg.slice(0, 120), "error");
  }
}

function getWhitelistProof() {
  if (!userAddress) return null;
  const proofs = CONFIG.whitelistProofs || {};
  return proofs[userAddress.toLowerCase()] || null;
}

function checkWhitelistStatus() {
  const mt = contractData.mintType;
  if (mt === 1 || mt === 2) return true;
  if (mt === 5) {
    // Type 5: anyone can mint (paid). Whitelisted get free via whitelistMint button
    // Show both options — whitelisted user gets free mint, non-whitelisted pays
    const proof = getWhitelistProof();
    if (proof) {
      setStatus("You are whitelisted — you can mint for free!", "success");
    } else {
      setStatus("Not whitelisted — you can mint at the public price.", "info");
    }
    return true;
  }
  if (mt === 0) { setStatus("Owner-only mint. Use CLI to mint.", "info"); return false; }
  const proof = getWhitelistProof();
  if (!proof) { setStatus("Your wallet is not whitelisted for this collection.", "error"); return false; }
  return true;
}

// ─── Polling ──────────────────────────────────────────────

async function refreshMinted() {
  try {
    const [minted, remaining] = await Promise.all([
      contract.totalMinted(), contract.remainingSupply(),
    ]);
    contractData.totalMinted = minted.toNumber();
    contractData.remaining = remaining.toNumber();
    renderDynamic();
  } catch (_) {}
}

function startPolling() { setInterval(refreshMinted, 15000); }

// ─── Utils ────────────────────────────────────────────────

function shortAddr(addr) {
  return addr ? addr.slice(0, 6) + "..." + addr.slice(-4) : "";
}

function setStatus(msg, type) {
  const el = document.getElementById("status");
  el.textContent = msg;
  el.className = "status-msg" + (type ? " " + type : "");
}

// ─── Start ────────────────────────────────────────────────

window.addEventListener("DOMContentLoaded", init);
```

### Step 5.5: Start HTTP Server

Ask the user for a port number (default: `33333`).

**Bind to `0.0.0.0`** — without it the server listens on `127.0.0.1` only and external connections get `ERR_CONNECTION_TIMED_OUT`.

```bash
# Python (already installed). Use nohup to survive session disconnect:
nohup python3 -m http.server 33333 --bind 0.0.0.0 --directory nft-collection/landing > /dev/null 2>&1 &

# Or Node.js alternative:
nohup npx http-server nft-collection/landing -p 33333 -a 0.0.0.0 > /dev/null 2>&1 &
```

### Step 5.6: Open Firewall Port (if needed)

Check if the firewall blocks incoming connections:

```bash
iptables -L INPUT -n | head -5
```

If the output shows `Chain INPUT (policy DROP)`, open the port:

```bash
iptables -I INPUT 1 -p tcp --dport 33333 -j ACCEPT
```

If using `ufw`:

```bash
ufw allow 33333/tcp
```

If policy is `ACCEPT`, no action needed.

### Step 5.7: Get Public IP and Display URL

```bash
curl -s ifconfig.me
```

Display to the user: **"Your minting page is live at: http://<PUBLIC_IP>:33333"**

Update `state.json` with the landing URL and set status to `"landing_deployed"`.

## Phase 6: Minting & Distribution

### Step 6.1: Batch Mint to Addresses

The user can request minting NFTs and distributing them to specific wallets. Accept addresses in two formats:

**Format A — Inline list:**
> "Mint 10 NFTs and send to: 0xAAA..., 0xBBB..., 0xCCC..."

**Format B — From file:**
> "Mint NFTs and send to addresses in wallet_list.txt"

For file input, read the file and parse one address per line. Skip empty lines and lines starting with `#`.

Execute batch minting via `cast`:

```bash
# For batch minting (one NFT per address):
cast send \
  --rpc-url $PHAROS_RPC_URL \
  --private-key $PRIVATE_KEY \
  --legacy \
  <CONTRACT_ADDRESS> \
  "batchMintToAddresses(address[])" \
  "[0xAAA...,0xBBB...,0xCCC...]"
```

For large lists (>50 addresses), split into batches of 50 to avoid gas limit issues. Show progress after each batch.

**Important:** For owner-only collections, only the owner can mint. For public collections, the owner can still use `batchMintToAddresses` or `airdrop`.

### Step 6.2: Airdrop with Custom Amounts

If the user wants to send different amounts to different addresses:

```bash
cast send \
  --rpc-url $PHAROS_RPC_URL \
  --private-key $PRIVATE_KEY \
  --legacy \
  <CONTRACT_ADDRESS> \
  "airdrop(address[],uint256[])" \
  "[0xAAA...,0xBBB...]" \
  "[3,1]"
```

### Step 6.3: Whitelist Setup (Merkle Tree)

For mint types 3, 4, or 5 (whitelist), generate a Merkle tree from the whitelist addresses.

**CRITICAL — How the contract verifies whitelist:**
```solidity
MerkleProof.verify(proof, merkleRoot, keccak256(abi.encodePacked(msg.sender)))
```

This means:
1. **Leaf = `keccak256(abi.encodePacked(address))`** — hashes the raw 20 bytes of the address, **NO padding**. `abi.encodePacked(address)` ≠ `abi.encode(address)` (the latter pads to 32 bytes).
2. **Proof = sibling nodes** at each level of the tree, **NOT the leaf itself**. Passing the leaf as its own proof produces `hash(leaf || leaf)` which will never match the root.
3. **Pairs are sorted** before hashing (OpenZeppelin convention): `parent = keccak256(sorted_concat(left, right))`.

**Quick verification with cast (2 addresses):**
```bash
ADDR1="0x049e88506b69324604D076c3E31be0e70Cb059f5"
ADDR2="0xe62db3731e2FdAaFfD653533744136589677d3d9"

# Leaves — cast keccak hashes the 20 raw bytes (abi.encodePacked, no padding)
LEAF1=$(cast keccak $ADDR1)
LEAF2=$(cast keccak $ADDR2)

# Sort ascending, then hash the pair
FIRST=$LEAF2   # 580e... < fa2d...
SECOND=$LEAF1
ROOT=$(cast keccak 0x${FIRST#0x}${SECOND#0x})

echo "Root: $ROOT"

# Proofs = sibling leaf (NOT the leaf itself!)
PROOF_FOR_ADDR1=$LEAF2   # sibling
PROOF_FOR_ADDR2=$LEAF1   # sibling
```

**For any number of addresses, use this Python script** (uses `cast keccak` — no extra Python packages needed):

```python
#!/usr/bin/env python3
"""Generate Merkle tree and proofs for whitelist minting.

Uses 'cast keccak' for keccak256 — no extra Python packages needed.

CRITICAL:
- Leaf = keccak256(abi.encodePacked(address)) = keccak256 of 20 raw bytes, NO padding
- Proof = sibling nodes from leaf to root (NOT the leaf itself)
- Pairs are sorted before hashing (OpenZeppelin MerkleProof convention)
"""

import json, subprocess, sys, os


def cast_keccak(hex_input):
    """Compute keccak256 via cast. hex_input must have 0x prefix."""
    r = subprocess.run(["cast", "keccak", hex_input],
                       capture_output=True, text=True, check=True)
    return r.stdout.strip()


def sort_pair(a, b):
    """Return (smaller, larger) by hex value."""
    return (a, b) if a.lower() <= b.lower() else (b, a)


def build_merkle(leaves):
    """Build sorted-pair Merkle tree (OpenZeppelin convention).
    Returns (root_hex, {original_index: [proof_element_hex, ...]}).
    """
    n = len(leaves)
    if n == 0:
        return "0x" + "00" * 32, {}
    if n == 1:
        return leaves[0], {0: []}

    # Pad to power of 2 by duplicating last leaf
    size = 1
    while size < n:
        size *= 2
    padded = leaves + [leaves[-1]] * (size - n)

    proofs = {i: [] for i in range(n)}
    current_level = list(padded)
    # Track which original leaf indices are under each node
    index_groups = [[i] for i in range(size)]

    while len(current_level) > 1:
        next_level = []
        next_groups = []

        for i in range(0, len(current_level), 2):
            left = current_level[i]
            right = current_level[i + 1]
            lo, hi = sort_pair(left, right)

            parent = cast_keccak("0x" + lo[2:] + hi[2:])
            next_level.append(parent)

            left_indices = index_groups[i]
            right_indices = index_groups[i + 1]
            next_groups.append(left_indices + right_indices)

            # Left subtree gets right as sibling, right subtree gets left
            for idx in left_indices:
                if idx < n:
                    proofs[idx].append(right)
            for idx in right_indices:
                if idx < n:
                    proofs[idx].append(left)

        current_level = next_level
        index_groups = next_groups

    return current_level[0], proofs


# ─── Main ───

# Source .env manually (shell state does NOT persist between calls)
env_path = os.path.join(os.path.dirname(os.path.abspath(__file__)), ".env")
if os.path.exists(env_path):
    with open(env_path) as f:
        for line in f:
            line = line.strip()
            if line and not line.startswith("#") and "=" in line:
                k, v = line.split("=", 1)
                os.environ[k] = v.strip()

# Load whitelist
if len(sys.argv) > 1:
    whitelist = [a.strip() for a in sys.argv[1:] if a.strip()]
else:
    state = json.load(open("nft-collection/state.json"))
    whitelist = state.get("mintConfig", {}).get("whitelistAddresses", [])

if not whitelist:
    print("ERROR: No whitelist addresses found.")
    sys.exit(1)

print(f"Generating Merkle tree for {len(whitelist)} addresses...")

# Step 1: Compute leaves
leaves = []
for addr in whitelist:
    leaf = cast_keccak(addr)
    leaves.append(leaf)
    print(f"  {addr} -> leaf: {leaf}")

# Step 2: Build tree + proofs
root, proofs = build_merkle(leaves)
print(f"\nMerkle root: {root}")

# Step 3: Save output
output = {
    "root": root,
    "leaves": {whitelist[i]: leaves[i] for i in range(len(whitelist))},
    "proofs": {whitelist[i]: proofs[i] for i in range(len(whitelist))}
}

out_path = "nft-collection/merkle-data.json"
with open(out_path, "w") as f:
    json.dump(output, f, indent=2)

print(f"\nSaved to {out_path}")
print(f"\nTo set root on contract:")
print(f'  cast send --rpc-url $PHAROS_RPC_URL --private-key $PRIVATE_KEY --legacy $CONTRACT "setMerkleRoot(bytes32)" {root}')
print(f"\nTo mint (whitelist address):")
print(f'  cast send --rpc-url $PHAROS_RPC_URL --private-key $PRIVATE_KEY --legacy $CONTRACT "whitelistMint(uint256,bytes32[])" 1 "[PROOF]" --value <PRICE_IN_WEI>')
```

Run: `python3 generate-merkle.py` (or pass addresses as args: `python3 generate-merkle.py 0xAAA... 0xBBB...`)

After generating, set the Merkle root on the contract:

```bash
set -a && source .env && set +a && cast send \
  --rpc-url $PHAROS_RPC_URL \
  --private-key $PRIVATE_KEY \
  --legacy \
  $CONTRACT \
  "setMerkleRoot(bytes32)" \
  <ROOT_FROM_OUTPUT>
```

To whitelist-mint for a specific address, read the proof from `merkle-data.json`:

```bash
set -a && source .env && set +a && cast send \
  --rpc-url $PHAROS_RPC_URL \
  --private-key $WHITELIST_PRIVATE_KEY \
  --legacy \
  $CONTRACT \
  "whitelistMint(uint256,bytes32[])" \
  1 \
  '["0xSIBLING_HASH_1","0xSIBLING_HASH_2",...]'
```

## Phase 7: Holder Analytics

### Step 7.1: Query Unique Holders

Scan all minted tokens to build a list of unique holders:

```python
#!/usr/bin/env python3
"""Query NFT holders from on-chain data."""
import subprocess
import json

CONTRACT = "0x..."  # From state.json
RPC_URL = "..."     # From environment

# Get total minted
result = subprocess.run(
    ["cast", "call", "--rpc-url", RPC_URL, CONTRACT, "totalMinted()"],
    capture_output=True, text=True
)
total_minted = int(result.stdout.strip(), 16)

# Query owner of each token
holders = {}
for token_id in range(1, total_minted + 1):
    result = subprocess.run(
        ["cast", "call", "--rpc-url", RPC_URL, CONTRACT,
         f"ownerOf(uint256)", str(token_id)],
        capture_output=True, text=True
    )
    owner = result.stdout.strip()
    if owner not in holders:
        holders[owner] = []
    holders[owner].append(token_id)

    if token_id % 50 == 0:
        print(f"Scanned {token_id}/{total_minted} tokens...")

# Summary
print(f"\nTotal tokens: {total_minted}")
print(f"Unique holders: {len(holders)}")
print(f"\nTop holders:")
for addr, tokens in sorted(holders.items(), key=lambda x: -len(x[1]))[:10]:
    print(f"  {addr}: {len(tokens)} tokens")
```

### Step 7.2: Export Holders

Save holder data to `nft-collection/holders.json`:

```json
{
  "generatedAt": "2026-05-25T12:00:00Z",
  "totalTokens": 1000,
  "uniqueHolders": 847,
  "holders": {
    "0xAAA...": {"count": 3, "tokens": [1, 42, 789]},
    "0xBBB...": {"count": 1, "tokens": [2]}
  }
}
```

Optionally export to CSV:

```csv
address,token_count,token_ids
0xAAA...,3,"1,42,789"
0xBBB...,1,"2"
```

Ask the user if they want the export as JSON, CSV, or both.

### Step 7.3: Distribution Analysis

Compute and display:
- Average tokens per holder
- Median tokens per holder
- Gini coefficient (distribution inequality)
- Top 10 holders and their percentages
- Percentage of supply held by top 10

## State Management

### Local Database (`state.json`)

All collection state is tracked in `nft-collection/state.json`. The agent reads from and writes to this file at every phase. This enables:
- Resuming a collection setup after interruption
- Querying deployment info without re-deploying
- Tracking which phases are complete

### State Transitions

```
configuring → images_composed/image_ready → ipfs_uploaded → deployed → setup_complete → verified → landing_deployed → minted → distributed → analyzed
```

### Reading State

Before starting any phase, check `state.json` to determine the current status. Skip already-completed phases unless the user explicitly requests redoing them.

### Updating State

After completing each phase, update `state.json` with:
- New status value
- Any output data (CIDs, addresses, hashes)
- Timestamp of completion

## Error Handling

- **Image dimension mismatch:** Stop and report the offending file with its actual dimensions vs. expected.
- **IPFS upload failure:** Retry once with exponential backoff. If it fails again, suggest the user check their Pinata JWT and quota.
- **Forge compilation error:** Display the compiler output and suggest fixes (usually import path or version issues).
- **Deployment failure (revert):** Check if the constructor args are correct, the wallet has enough gas, and the RPC is reachable.
- **Mint failure:** Verify the caller is the owner (for owner functions), the supply isn't exceeded, and the contract is deployed.
- **Holder query timeout:** For large collections, batch the queries and save intermediate results.

## Security Considerations

- **Never hardcode private keys** in any file. Always use environment variables (`PRIVATE_KEY`).
- **Pinata JWT** must also come from env vars, not be committed to git.
- **Add `state.json` to `.gitignore`** if it contains sensitive info.
- The generated Solidity contract uses OpenZeppelin's audited ERC-721 and Ownable implementations.
- `withdraw()` uses the `call` pattern instead of `transfer` to prevent gas-related reverts.
- Contract ownership is set to the deployer address — ensure the user controls this key.

## Example Usage Prompts

After installing this skill, the user can trigger it with prompts like:

1. **"Deploy a generative NFT collection on Pharos"** — Starts the full interactive configuration flow
2. **"Create a minting page for my NFT collection"** — Triggers landing page generation (Phase 5)
3. **"Mint 50 NFTs from my collection and send them to the addresses in airdrop_list.txt"** — Triggers batch minting phase
4. **"Show me all holders of my NFT collection"** — Triggers holder analytics
5. **"Export my collection's holder data to a CSV file"** — Triggers export phase
6. **"Set up a whitelist for my NFT collection using these addresses: 0xAAA, 0xBBB, 0xCCC"** — Triggers Merkle tree generation and contract update
7. **"How rare is token #42 in my collection?"** — Looks up trait combination and computes rarity score
8. **"Enable public minting for my collection at 0.05 PROS per mint, max 3 per wallet"** — Updates contract parameters
