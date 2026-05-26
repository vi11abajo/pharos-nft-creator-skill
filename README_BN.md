# Pharos NFT Creator — ব্যবহারকারী নির্দেশিকা

## সংক্ষিপ্ত বিবরণ

একটি Pharos Agent Center Skill যা AI এজেন্টদের Pharos ব্লকচেইনে সম্পূর্ণ জীবনচক্রের NFT কালেকশন তৈরি করতে সক্ষম করে। জেনারেটিভ (ট্রেইট-ভিত্তিক), স্বতন্ত্র ইমেজ এবং একক-ইমেজ মোড সমর্থিত। আপনি প্রাকৃতিক ভাষায় এজেন্টের সাথে কথা বলেন, এবং এটি ইমেজ তৈরি, IPFS আপলোড, স্মার্ট কন্ট্রাক্ট ডিপ্লয়মেন্ট, মিন্টিং এবং বিতরণ পরিচালনা করে।

## বৈশিষ্ট্যসমূহ

- প্রতিটি কলে স্বয়ংক্রিয় নির্ভরতা পরীক্ষা (Python, Forge, Cast, Node.js ইত্যাদি)
- এনভায়রনমেন্ট ভেরিয়েবল সহ `.env` ফাইলের স্বয়ংক্রিয় তৈরি
- প্রাইভেট কি-এর স্বয়ংক্রিয় নরমালাইজেশন (`0x` প্রিফিক্স সহ বা ছাড়া পেস্ট করুন)
- তিনটি আর্টওয়ার্ক মোড: জেনারেটিভ (traits), স্বতন্ত্র ইমেজ, একক ইমেজ
- ট্রেইট লেয়ার এবং র‍্যারিটি ওয়েট সহ জেনারেটিভ NFT তৈরি
- Python Pillow ব্যবহার করে PNG লেয়ার থেকে অফ-চেইন ইমেজ কম্পোজিশন
- Pinata-র মাধ্যমে ইমেজ + মেটাডেটা ফোল্ডারের IPFS আপলোড (ফাইলের জন্য SDK, ফোল্ডারের জন্য REST API)
- BaseURI প্যাটার্ন — কন্ট্রাক্ট প্রতিটি NFT-র জন্য `ipfs://CID/tokenId` রিটার্ন করে
- অন্তর্নির্মিত Pharos নেটওয়ার্ক ডেটা — testnet/mainnet নির্বাচন করুন, RPC স্বয়ংক্রিয়ভাবে কনফিগার হয়
- ৬টি মিন্ট মোড সহ ERC-721 স্মার্ট কন্ট্রাক্ট ডিপ্লয়মেন্ট (public free/paid, owner-only, whitelist free/paid, whitelist free + public paid)
- ঐচ্ছিক মিন্টিং ল্যান্ডিং পেজ (HTML + ethers.js, স্বয়ংক্রিয় কনফিগার্ড)
- যেকোনো সংখ্যক ওয়ালেটে ব্যাচ মিন্টিং এবং এয়ারড্রপ (ইনলাইন বা ফাইল থেকে)
- Merkle tree-র মাধ্যমে হোয়াইটলিস্ট সমর্থন (সঠিক প্রুফ সহ স্বয়ংক্রিয়ভাবে তৈরি)
- JSON/CSV এক্সপোর্ট সহ হোল্ডার অ্যানালিটিক্স
- এজেন্ট ব্যবহারকারীর ভাষায় যোগাযোগ করে

## ইনস্টলেশন

সম্পূর্ণ `pharos-nft-creator-skill` ফোল্ডারটি আপনার এজেন্টের skills ডিরেক্টরিতে কপি করুন:

| এজেন্ট | পাথ |
|-------|------|
| Claude Code | `~/.claude/skills/` |
| OpenClaw | `~/.openclaw/skills/` |
| Codex | `~/.codex/skills/` |

Claude Code-এর উদাহরণ:
```bash
cp -r pharos-nft-creator-skill ~/.claude/skills/
```

Skill ফোল্ডার কাঠামো:
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

ইনস্টল করার পর আপনার এজেন্ট সেশন রিস্টার্ট করুন।

## প্রথম লঞ্চ — স্বয়ংক্রিয় পরীক্ষা

আপনি যখন প্রথমবার Skill টি কল করেন, এজেন্ট স্বয়ংক্রিয়ভাবে **Phase 0: Dependency Check** চালায়:

1. **টুল পরীক্ষা** — Python, Pillow, Forge, Cast, jq, Node.js, Pinata SDK। অনুপস্থিত থাকলে ইনস্টল করার প্রস্তাব দেয়।
2. **`.env` ফাইল তৈরি** — প্রজেক্টে এটি না থাকলে, `.env.example` থেকে প্লেসহোল্ডার সহ কপি করে।
3. **ভেরিয়েবল যাচাই** — `PRIVATE_KEY`, `PINATA_JWT` পূরণ আছে কিনা পরীক্ষা করে।
4. **`.env`-এ গাইড করে** — সিক্রেট খালি থাকলে, **ফাইল পাথ** দেখায় এবং কোন লাইন পূরণ করতে হবে তা জানায়।

Pharos RPC URL এবং নেটওয়ার্ক ডেটা **Skill-এ অন্তর্নির্মিত** — কোনো ম্যানুয়াল কনফিগারেশন প্রয়োজন নেই। আপনি শুধু সেটআপের সময় নেটওয়ার্ক নির্বাচন করুন।

উদাহরণ আউটপুট:
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

## এনভায়রনমেন্ট ভেরিয়েবল

সমস্ত সিক্রেট প্রজেক্ট রুটের `.env` ফাইলে সংরক্ষিত থাকে। **কখনোই সরাসরি চ্যাটে কি পেস্ট করবেন না।**

`.env` ফাইল (স্বয়ংক্রিয়ভাবে তৈরি হয়):
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

### মান কোথায় পাবেন

| ভেরিয়েবল | উৎস |
|----------|--------|
| `PRIVATE_KEY` | MetaMask → তিনটি ডট → Account details → Show private key। `0x` সহ বা ছাড়া পেস্ট করুন — স্বয়ংক্রিয়ভাবে নরমালাইজ হবে |
| `PINATA_JWT` | [app.pinata.cloud](https://app.pinata.cloud) → API Keys → New Key → JWT কপি করুন |

## আর্টওয়ার্ক প্রস্তুতি

Skill টি **তিনটি আর্টওয়ার্ক মোড** সমর্থন করে:

### মোড A: জেনারেটিভ (Traits)

ক্লাসিক জেনারেটিভ পদ্ধতি — একাধিক ট্রেইট লেয়ার এলোমেলোভাবে সম্মিলিত হয়। সমস্ত ফাইল অবশ্যই PNG, RGBA, একই মাত্রার হতে হবে।

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

### মোড B: স্বতন্ত্র ইমেজ

পূর্ব-প্রস্তুত PNG-এর একটি ফোল্ডার — প্রতিটি ফাইল = একটি অনন্য NFT। প্রতিটিতে আপনি একটি র‍্যারিটি ওয়েট নির্ধারণ করেন। একটি JSON/CSV কনফিগ প্রদান করুন অথবা ম্যানুয়ালি প্রবেশ করুন।

```
my-nfts/
├── 001_samurai.png     (weight: 10 — legendary)
├── 002_ninja.png       (weight: 30 — uncommon)
├── 003_wizard.png      (weight: 5 — legendary)
├── 004_knight.png      (weight: 55 — common)
└── ...
```

কনফিগ (ঐচ্ছিক, JSON):
```json
[
  {"file": "001_samurai.png", "weight": 10, "name": "Samurai"},
  {"file": "002_ninja.png", "weight": 30, "name": "Ninja"}
]
```

### মোড C: একক ইমেজ

সমস্ত NFT-র জন্য একটি PNG। প্রতিটি টোকেন একই দেখাবে। সদস্যপদ পাস, টিকিট, ইউটিলিটি NFT-র জন্য উপযুক্ত।

```
collection-art.png    (single image for the entire collection)
```

### র‍্যারিটি ওয়েট (মোড A এবং B-এর জন্য)

ওয়েট নির্ধারণ করে প্রতিটি ট্রেইট ভেরিয়েন্ট কতটা ঘনঘন প্রদর্শিত হবে। বেশি ওয়েট = বেশি সাধারণ:

| বিভাগ | ওয়েট | ফ্রিকোয়েন্সি |
|----------|--------|-----------|
| Legendary | ১–৪ | অত্যন্ত বিরল |
| Rare | ৫–১৯ | অসাধারণ |
| Uncommon | ২০–৪৯ | মাঝারি |
| Common | ৫০–১০০ | ঘনঘন |

উদাহরণ: যদি Headwear-এ Crown(5), Cap(30), Helmet(40), None(25) থাকে, তাহলে Crown প্রায় ৫% জেনারেটেড NFT-তে প্রদর্শিত হবে।

## ব্যবহার

### ১. কালেকশন তৈরি শুরু করুন

এজেন্টকে বলুন:
> "Deploy a generative NFT collection on Pharos"

এজেন্ট নির্ভরতা পরীক্ষা করবে, প্রয়োজনে `.env` তৈরি করবে এবং ইন্টারেক্টিভ কনফিগারেশন শুরু করবে।

### ২. কনফিগারেশন প্রশ্নের উত্তর দিন

এজেন্ট জিজ্ঞাসা করবে:
- **কোন নেটওয়ার্ক ব্যবহার করবেন** — Pharos Atlantic Testnet (বিনামূল্যে, পরীক্ষার জন্য) অথবা Pharos Mainnet (প্রকৃত PROS টোকেন)
- কালেকশনের নাম এবং প্রতীক
- বিবরণ
- মোট সাপ্লাই (সর্বোচ্চ NFT সংখ্যা)
- মিন্ট টাইপ (নিচে দেখুন)
- **আর্টওয়ার্ক মোড** — জেনারেটিভ (traits), স্বতন্ত্র ইমেজ, অথবা একক ইমেজ
- আর্টওয়ার্ক বিবরণ: ট্রেইট লেয়ার, অথবা PNG ফোল্ডার + ওয়েট, অথবা একটি একক PNG-এর পাথ

### ৩. মিন্ট টাইপ

এজেন্ট প্রতিটি বিকল্প ব্যাখ্যা করবে এবং আপনাকে বেছে নিতে সাহায্য করবে। ছয়টি টাইপ উপলব্ধ:

| টাইপ | বিবরণ | ব্যবহারের ক্ষেত্র |
|------|-------------|----------|
| **Public Free** | যে কেউ বিনামূল্যে মিন্ট করতে পারে | উন্মুক্ত কমিউনিটি কালেকশন |
| **Public Paid** | যে কেউ নির্দিষ্ট মূল্য প্রদান করে | প্রিমিয়াম কালেকশন |
| **Owner-Only** | শুধুমাত্র কন্ট্রাক্ট মালিক মিন্ট করতে পারে | এয়ারড্রপ, ব্যক্তিগত বিতরণ |
| **Whitelist Free** | শুধুমাত্র হোয়াইটলিস্টেড ঠিকানাগুলো বিনামূল্যে মিন্ট করতে পারে | এক্সক্লুসিভ কমিউনিটি ড্রপ |
| **Whitelist Paid** | শুধুমাত্র হোয়াইটলিস্টেড ঠিকানাগুলো অর্থ প্রদান করে মিন্ট করে | প্রিমিয়াম কমিউনিটি ড্রপ |
| **WL Free + Public Paid** | হোয়াইটলিস্টেডরা বিনামূল্যে মিন্ট করে, বাকিরা অর্থ প্রদান করে | কমিউনিটি পুরস্কার + পাবলিক সেল |

হোয়াইটলিস্ট টাইপের জন্য, আপনাকে ঠিকানার একটি তালিকা প্রদান করতে হবে (ফাইল বা ইনলাইন)। এজেন্ট স্বয়ংক্রিয়ভাবে একটি Merkle tree তৈরি করে এবং কন্ট্রাক্টে আপলোড করে।

### ৪. ডিপ্লয়মেন্টের পর

**NFT মিন্ট এবং বিতরণ করুন:**
> "Mint 50 NFTs from my collection and send them to the addresses in airdrop_list.txt"

ঠিকানা ফাইলের ফরম্যাট — প্রতি লাইনে একটি ঠিকানা:
```
0xAAA...BBB...CCC...
# Lines starting with # are comments
```

অথবা ইনলাইন:
> "Mint 3 NFTs and send to: 0xAAA..., 0xBBB..., 0xCCC..."

**হোল্ডার দেখুন:**
> "Show me all holders of my NFT collection"

**ফাইলে এক্সপোর্ট করুন:**
> "Export holder data to CSV"

**হোয়াইটলিস্ট সেটআপ করুন:**
> "Set up a whitelist for my collection using these addresses: 0xAAA, 0xBBB"

**মিন্ট প্যারামিটার আপডেট করুন:**
> "Enable public minting at 0.05 PROS, max 5 per wallet"

## সমস্যা সমাধান

| সমস্যা | সমাধান |
|-------|----------|
| PNG মাত্রা অমিল | সমস্ত ফাইল একই আকারের কিনা যাচাই করুন, প্রয়োজনে পুনরায় এক্সপোর্ট করুন |
| IPFS আপলোড ব্যর্থতা | `.env`-এ `PINATA_JWT` এবং Pinata অ্যাকাউন্ট কোটা পরীক্ষা করুন |
| **Pinata: ৫০০ ফাইল সীমা** | ফ্রি প্ল্যান = ৫০০ ফাইল, ১ GB। মোড A: ~২৫০ NFT (2N ফাইল)। মোড B: ৫০০-M পর্যন্ত NFT। মোড C: ~৫০০ পর্যন্ত NFT |
| Forge পাওয়া যায়নি | ইনস্টল: `curl -L https://foundry.paradigm.xyz \| bash && foundryup` |
| Pillow পাওয়া যায়নি | ইনস্টল: `pip install Pillow` |
| **Stack too deep** ত্রুটি | `foundry.toml`-এ `via_ir = true` যোগ করুন — এই কন্ট্রাক্টের জন্য প্রয়োজনীয় |
| Forge কম্পাইলেশন ত্রুটি | OpenZeppelin v5 ইনস্টল আছে নিশ্চিত করুন: `forge install https://github.com/OpenZeppelin/openzeppelin-contracts@v5.0.2 --no-git` |
| Pharos-এ ট্রানজেকশন ব্যর্থ | `forge create` এবং `cast send`-এর সাথে সবসময় `--legacy` ফ্ল্যাগ ব্যবহার করুন |
| অপর্যাপ্ত গ্যাস | Pharos-এ ওয়ালেট ব্যালেন্স পরীক্ষা করুন |
| মিন্ট কাজ করছে না | আপনি কন্ট্রাক্ট মালিক কিনা যাচাই করুন এবং public কালেকশনে publicMintEnabled সত্য কিনা দেখুন |
| `.env` লোড হচ্ছে না | প্রতিটি bash কমান্ড `set -a && source .env && set +a` দিয়ে শুরু করতে হবে |
| NFT ইমেজ এক্সপ্লোরারে দেখাচ্ছে না | মেটাডেটায় `ipfs://CID` ব্যবহৃত হয়েছে কিনা নিশ্চিত করুন (গেটওয়ে URL নয়)। ফোল্ডার CID সঠিক কিনা যাচাই করুন। |

## নিরাপত্তা

- প্রাইভেট কি, JWT টোকেন বা যেকোনো সিক্রেট **কখনোই চ্যাটে পেস্ট করবেন না**
- সমস্ত সিক্রেট **শুধুমাত্র `.env` ফাইলে** সংরক্ষিত থাকে — এজেন্ট এটি স্বয়ংক্রিয়ভাবে পড়ে
- `.env` স্বয়ংক্রিয়ভাবে `.gitignore`-এ যোগ হয়
- `state.json`-ও `.gitignore`-এ যোগ হয়
- নিশ্চিত করুন যে আপনি কন্ট্রাক্টের মালিকানা কি নিয়ন্ত্রণ করেন
- আপনি অনিচ্ছাকৃতভাবে চ্যাটে কোনো সিক্রেট পেস্ট করে ফেললে — অবিলম্বে সেই কি রিপ্লেস করুন

## আউটপুট ফাইল কাঠামো

Skill চলার পর, আপনার প্রজেক্টে থাকবে:

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
