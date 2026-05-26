# Pharos NFT Creator — صارف کی رہنمائی

## جائزہ

Pharos Agent Center کا ایک Skill جو AI ایجنٹوں کو Pharos بلاک چین پر مکمل لائف سائیکل NFT کلیکشن بنانے کے قابل بناتا ہے۔ یہ جنریٹو (trait پر مبنی)، انفرادی تصویر، اور سنگل امیج موڈز کو سپورٹ کرتا ہے۔ آپ ایجنٹ سے قدرتی زبان میں بات چیت کرتے ہیں، اور یہ تصویر کی تخلیق، IPFS اپلوڈز، سمارٹ کنٹریکٹ ڈپلائمنٹ، مِنٹنگ، اور تقسیم کا کام انجام دیتا ہے۔

## خصوصیات

- ہر استعمال پر خودکار انحصار کی جانچ (Python, Forge, Cast, Node.js, وغیرہ)
- ماحولیاتی متغیرات کے ساتھ `.env` فائل کی خود بخود تخلیق
- پرائیویٹ کلید کی خودکار نارملائزیشن (`0x` سابقے کے ساتھ یا بغیر چسپاں کریں)
- تین آرٹ ورک موڈز: جنریٹو (traits)، انفرادی تصاویر، سنگل تصویر
- trait لیئرز اور ندرت کے وزن کے ساتھ جنریٹو NFT کی تخلیق
- Python Pillow کا استعمال کرتے ہوئے PNG لیئرز سے آف چین امیج کمپوزیشن
- Pinata کے ذریعے تصاویر + میٹا ڈیٹا فولڈر کا IPFS اپلوڈ (فائلز کے لیے SDK، فولڈرز کے لیے REST API)
- BaseURI پیٹرن — کنٹریکٹ ہر NFT کے لیے `ipfs://CID/tokenId` لوٹاتا ہے
- Pharos نیٹ ورک کا ڈیٹا بلٹ ان — ٹیسٹ نیٹ/مین نیٹ کا انتخاب کریں، RPC خودکار طور پر ترتیب دیا جاتا ہے
- 6 منٹ موڈز کے ساتھ ERC-721 سمارٹ کنٹریکٹ کی ڈپلائمنٹ (پبلک مفت/ادائیگی، صرف مالک، وائٹ لسٹ مفت/ادائیگی، وائٹ لسٹ مفت + پبلک ادائیگی)
- اختیاری منٹنگ لینڈنگ پیج (HTML + ethers.js، خودکار ترتیب)
- کسی بھی تعداد میں والیٹس تک بیچ منٹنگ اور ایئر ڈراپ (ان لائن یا فائل سے)
- Merkle tree کے ذریعے وائٹ لسٹ سپورٹ (درست ثبوت کے ساتھ خود بخود تیار)
- JSON/CSV ایکسپورٹ کے ساتھ ہولڈر تجزیات
- ایجنٹ صارف کی زبان میں بات چیت کرتا ہے

## تنصیب

`pharos-nft-creator-skill` فولڈر کو **مکمل طور پر** اپنے ایجنٹ کی skills ڈائرکٹری میں کاپی کریں:

| ایجنٹ | راستہ |
|-------|-------|
| Claude Code | `~/.claude/skills/` |
| OpenClaw | `~/.openclaw/skills/` |
| Codex | `~/.codex/skills/` |

Claude Code کے لیے مثال:
```bash
cp -r pharos-nft-creator-skill ~/.claude/skills/
```

Skill فولڈر کی ساخت:
```
pharos-nft-creator-skill/
├── pharos-nft-creator-skill.md         — مین skill فائل
├── dependencies.json                  — خودکار جانچ کے لیے انحصار کی فہرست
├── .env.example                       — ماحولیاتی متغیرات کا ٹیمپلیٹ
├── README.md                          — ہدایات (EN)
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

تنصیب کے بعد اپنے ایجنٹ کا سیشن دوبارہ شروع کریں۔

## پہلی لاہ — خودکار جانچ

جب آپ skill کو پہلی بار استعمال کرتے ہیں، تو ایجنٹ خود بخود **Phase 0: Dependency Check** چلاتا ہے:

1. **ٹولز کی جانچ** — Python, Pillow, Forge, Cast, jq, Node.js, Pinata SDK۔ کسی بھی غائب ٹول کو انسٹال کرنے کی پیشکش کرتا ہے۔
2. **`.env` فائل کی تخلیق** — اگر پروجیکٹ میں موجود نہیں ہے، تو `.env.example` سے پلیس ہولڈرز کے ساتھ کاپی کرتا ہے۔
3. **متغیرات کی تصدیق** — چیک کرتا ہے کہ `PRIVATE_KEY`، `PINATA_JWT` بھرے ہوئے ہیں یا نہیں۔
4. **`.env` تک رہنمائی** — اگر راز خالی ہیں، تو **فائل کا راستہ** دکھاتا ہے اور بتاتا ہے کہ کون سی لائنیں بھرنی ہیں۔

Pharos RPC URLs اور نیٹ ورک ڈیٹا **Skill میں بلٹ ان** ہے — دستی ترتیب کی ضرورت نہیں۔ آپ صرف سیٹ اپ کے دوران نیٹ ورک کا انتخاب کرتے ہیں۔

مثال آؤٹ پٹ:
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

## ماحولیاتی متغیرات

تمام راز پروجیکٹ کی جڑ میں `.env` فائل میں محفوظ ہوتے ہیں۔ **کبھی بھی کلیدیں براہ راست چیٹ میں چسپاں نہ کریں۔**

`.env` فائل (خود بخود بنائی جاتی ہے):
```bash
# آپ کا والیٹ پرائیویٹ کلید۔
# 0x سابقے کے ساتھ یا بغیر چسپاں کریں — skill خود اسے نارملائز کرے گا۔
# سیکیورٹی: فائل میں براہ راست بھریں۔ چیٹ میں چسپاں نہ کریں۔
PRIVATE_KEY=

# Pinata JWT ٹوکن — حاصل کرنے کا طریقہ:
#   1. https://app.pinata.cloud پر جائیں → سائن اپ / لاگ ان کریں
#   2. بائیں سائڈ بار → "API Keys" → "New Key"
#   3. نام دیں (جیسے "nft-builder")، ڈیفالٹ اجازتیں چھوڑ دیں
#   4. "Create Key" پر کلک کریں → JWT کاپی کریں (لمبی اسٹرنگ جو "eyJ..." سے شروع ہوتی ہے)
#   5. نیچے چسپاں کریں
PINATA_JWT=

# نیٹ ورک کنفیگریشن کے دوران منتخب کیا جاتا ہے — RPC URL دستی طور پر سیٹ کرنے کی ضرورت نہیں۔
```

### اقدار کہاں سے حاصل کریں

| متغیر | ذریعہ |
|-------|-------|
| `PRIVATE_KEY` | MetaMask → تین نقطے → Account details → Show private key۔ `0x` کے ساتھ یا بغیر چسپاں کریں — خود بخود نارملائز ہوگا |
| `PINATA_JWT` | [app.pinata.cloud](https://app.pinata.cloud) → API Keys → New Key → JWT کاپی کریں |

## آرٹ ورک کی تیاری

Skill **تین آرٹ ورک موڈز** سپورٹ کرتی ہے:

### موڈ A: جنریٹو (Traits)

کلاسک جنریٹو طریقہ — متعدد trait لیئرز بے ترتیب طور پر ملائی جاتی ہیں۔ تمام فائلز PNG, RGBA، ایک ہی ابعاد کی ہونی چاہئیں۔

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
│   ├── none.png          (خالی شفاف PNG)
│   ├── crown.png
│   ├── cap.png
│   └── helmet.png
└── Weapon/
    ├── none.png
    ├── sword.png
    └── axe.png
```

### موڈ B: انفرادی تصاویر

پہلے سے تیار PNGs کا فولڈر — ہر فائل = ایک منفرد NFT۔ آپ ہر ایک کو ندرت کا وزن تفویض کرتے ہیں۔ JSON/CSV کنفگ فراہم کریں یا دستی طور پر داخل کریں۔

```
my-nfts/
├── 001_samurai.png     (weight: 10 — legendary)
├── 002_ninja.png       (weight: 30 — uncommon)
├── 003_wizard.png      (weight: 5 — legendary)
├── 004_knight.png      (weight: 55 — common)
└── ...
```

کنفگ (اختیاری، JSON):
```json
[
  {"file": "001_samurai.png", "weight": 10, "name": "Samurai"},
  {"file": "002_ninja.png", "weight": 30, "name": "Ninja"}
]
```

### موڈ C: سنگل امیج

تمام NFTs کے لیے ایک PNG۔ ہر ٹوکن ایک جیسا نظر آئے گا۔ ممبرشپ پاسز، ٹکٹس، یوٹیلیٹی NFTs کے لیے موزوں۔

```
collection-art.png    (پوری کلیکشن کے لیے ایک ہی تصویر)
```

### ندرت کے وزن (موڈ A اور B کے لیے)

وزن طے کرتے ہیں کہ ہر trait ویرینٹ کتنی بار ظاہر ہوتا ہے۔ زیادہ وزن = زیادہ عام:

| زمرہ | وزن | تعدد |
|------|-----|------|
| Legendary | 1–4 | بہت نایاب |
| Rare | 5–19 | کم عام |
| Uncommon | 20–49 | درمیانہ |
| Common | 50–100 | زیادہ عام |

مثال: اگر Headwear میں Crown(5)، Cap(30)، Helmet(40)، None(25) ہے، تو Crown تقریباً 5% جنریٹ ہونے والی NFTs میں نظر آئے گا۔

## استعمال

### 1. کلیکشن کی تخلیق شروع کریں

ایجنٹ کو بتائیں:
> "Deploy a generative NFT collection on Pharos"

ایجنٹ انحصار کی جانچ کرے گا، اگر ضروری ہو تو `.env` بنائے گا، اور انٹرایکٹو کنفیگریشن شروع کرے گا۔

### 2. کنفیگریشن کے سوالات کے جوابات دیں

ایجنٹ آپ سے پوچھے گا:
- **کون سا نیٹ ورک استعمال کرنا ہے** — Pharos Atlantic Testnet (مفت، ٹیسٹنگ کے لیے) یا Pharos Mainnet (اصل PROS ٹوکنز)
- کلیکشن کا نام اور علامت (symbol)
- تفصیل
- کل سپلائی (زیادہ سے زیادہ NFTs کی تعداد)
- منٹ کی قسم (نیچے دیکھیں)
- **آرٹ ورک موڈ** — جنریٹو (traits)، انفرادی تصاویر، یا سنگل تصویر
- آرٹ ورک کی تفصیلات: trait لیئرز، یا PNGs کا فولڈر + وزن، یا سنگل PNG کا راستہ

### 3. منٹ کی اقسام

ایجنٹ ہر آپشن کی وضاحت کرے گا اور آپ کو منتخب کرنے میں مدد دے گا۔ چھ اقسام دستیاب ہیں:

| قسم | تفصیل | استعمال کا موقع |
|------|--------|----------------|
| **Public Free** | کوئی بھی مفت میں منٹ کر سکتا ہے | کھلی کمیونٹی کلیکشنز |
| **Public Paid** | کوئی بھی مقررہ قیمت ادا کر کے منٹ کرتا ہے | پریمیم کلیکشنز |
| **Owner-Only** | صرف کنٹریکٹ مالک منٹ کر سکتا ہے | ایئر ڈراپس، نجی تقسیم |
| **Whitelist Free** | صرف وائٹ لسٹ شدہ ایڈریسز مفت میں منٹ کر سکتے ہیں | خصوصی کمیونٹی ڈراپس |
| **Whitelist Paid** | صرف وائٹ لسٹ شدہ ایڈریسز ادائیگی کر کے منٹ کرتے ہیں | پریمیم کمیونٹی ڈراپس |
| **WL Free + Public Paid** | وائٹ لسٹ شدہ مفت منٹ کرتے ہیں، باقی سب ادائیگی کرتے ہیں | کمیونٹی انعامات + پبلک سیل |

وائٹ لسٹ اقسام کے لیے، آپ کو ایڈریسز کی فہرست فراہم کرنی ہوگی (فائل یا ان لائن)۔ ایجنٹ خود بخود Merkle tree بنتا ہے اور اسے کنٹریکٹ پر اپلوڈ کرتا ہے۔

### 4. ڈپلائمنٹ کے بعد

**NFTs کی منٹنگ اور تقسیم:**
> "Mint 50 NFTs from my collection and send them to the addresses in airdrop_list.txt"

ایڈریس فائل کی شکل — ہر لائن پر ایک ایڈریس:
```
0xAAA...BBB...CCC...
# # سے شروع ہونے والی لائنیں تبصرے ہیں
```

یا ان لائن:
> "Mint 3 NFTs and send to: 0xAAA..., 0xBBB..., 0xCCC..."

**ہولڈرز دیکھیں:**
> "Show me all holders of my NFT collection"

**فائل میں ایکسپورٹ کریں:**
> "Export holder data to CSV"

**وائٹ لسٹ سیٹ اپ کریں:**
> "Set up a whitelist for my collection using these addresses: 0xAAA, 0xBBB"

**منٹ پیرامیٹرز اپ ڈیٹ کریں:**
> "Enable public minting at 0.05 PROS, max 5 per wallet"

## مسئلہ حل

| مسئلہ | حل |
|-------|-----|
| PNG ابعاد میں فرق | تصدیق کریں کہ تمام فائلز ایک ہی سائز کی ہیں، اگر ضروری ہو تو دوبارہ ایکسپورٹ کریں |
| IPFS اپلوڈ ناکام | `.env` میں `PINATA_JWT` اور Pinata اکاؤنٹ کے کوٹے کی جانچ کریں |
| **Pinata: 500 فائل کی حد** | مفت پلان = 500 فائلز، 1 GB۔ موڈ A: ~250 NFT (2N فائلز)۔ موڈ B: 500-M تک NFTs۔ موڈ C: ~500 تک NFTs |
| Forge نہیں ملا | انسٹال کریں: `curl -L https://foundry.paradigm.xyz \| bash && foundryup` |
| Pillow نہیں ملی | انسٹال کریں: `pip install Pillow` |
| **Stack too deep** خرابی | `foundry.toml` میں `via_ir = true` شامل کریں — اس کنٹریکٹ کے لیے ضروری ہے |
| Forge کمپائلیشن خرابی | یقینی بنائیں کہ OpenZeppelin v5 انسٹال ہے: `forge install https://github.com/OpenZeppelin/openzeppelin-contracts@v5.0.2 --no-git` |
| Pharos پر ٹرانزیکشن ناکام | `forge create` اور `cast send` کے ساتھ ہمیشہ `--legacy` فلیگ استعمال کریں |
| گیس ناکافی | Pharos پر والیٹ بیلنس چیک کریں |
| منٹ کام نہیں کر رہا | تصدیق کریں کہ آپ کنٹریکٹ کے مالک ہیں، اور پبلک کلیکشنز کے لیے publicMintEnabled درست ہے |
| `.env` لوڈ نہیں ہو رہا | ہر bash کمانڈ کا آغاز `set -a && source .env && set +a` سے ہونا چاہیے |
| NFT کی تصویر ایکسپلورر میں نظر نہیں آ رہی | یقینی بنائیں کہ میٹا ڈیٹا `ipfs://CID` استعمال کر رہا ہے (گیٹ وے URLs نہیں)۔ فولڈر CID درست ہے تصدیق کریں۔ |

## سیکیورٹی

- پرائیویٹ کلیدیں، JWT ٹوکنز، یا کوئی بھی راز **کبھی بھی** چیٹ میں چسپاں نہ کریں
- تمام راز **صرف `.env` فائل میں** محفوظ ہوتے ہیں — ایجنٹ خود بخود اسے پڑھتا ہے
- `.env` خود بخود `.gitignore` میں شامل ہو جاتی ہے
- `state.json` بھی `.gitignore` میں شامل ہو جاتی ہے
- یقینی بنائیں کہ آپ کنٹریکٹ کی مالکانہ کلید کنٹرول کرتے ہیں
- اگر آپ غلطی سے کوئی راز چیٹ میں چسپاں کر دیتے ہیں — فوراً اس کلید کو تبدیل کریں

## آؤٹ پٹ فائل کی ساخت

Skill چلنے کے بعد، آپ کے پروجیکٹ میں یہ شامل ہوں گے:

```
project/
├── .env                         # آپ کے راز (کبھی commit نہ کریں!)
├── .gitignore                   # .env اور state.json مستثنیٰ
└── nft-collection/
    ├── traits/                  # اصل trait PNG فائلز (موڈ A)
    ├── composed/                # کمپوزڈ حتمی NFT تصاویر (Pillow)
    ├── metadata/                # ہر ٹوکن کے لیے JSON میٹا ڈیٹا (ID کے نام سے)
    ├── src/                     # Solidity کنٹریکٹ (Foundry)
    ├── landing/                 # منٹنگ ویب پیج (اختیاری، Phase 5)
    ├── state.json               # مقامی کلیکشن ڈیٹا بیس
    └── holders.json             # ہولڈر ڈیٹا (تجزیات کے بعد)
```
