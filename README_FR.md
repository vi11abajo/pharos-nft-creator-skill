# Pharos NFT Creator — Guide utilisateur

## Apercu

Un Skill Pharos Agent Center qui permet aux agents IA de creer des collections NFT completes sur la blockchain Pharos. Prend en charge les modes generatif (base sur les traits), images individuelles et image unique. Vous interagissez avec l'agent en langage naturel, et il gere la generation d'images, les telechargements IPFS, le deploiement de contrats intelligents, le minting et la distribution.

## Fonctionnalites

- Verification automatique des dependances a chaque appel (Python, Forge, Cast, Node.js, etc.)
- Creation automatique du fichier `.env` avec les variables d'environnement
- Normalisation automatique de la cle privee (coller avec ou sans le prefixe `0x`)
- Trois modes de creation artistique : generatif (traits), images individuelles, image unique
- Creation de NFT generatifs avec des couches de traits et des poids de rarete
- Composition d'images hors chaine a partir de couches PNG via Python Pillow
- Telechargement IPFS des images + dossier de metadonnees via Pinata (SDK pour les fichiers, API REST pour les dossiers)
- Modele BaseURI — le contrat renvoie `ipfs://CID/tokenId` pour chaque NFT
- Donnees du reseau Pharos integrees — choisissez testnet/mainnet, RPC configure automatiquement
- Deploiement de contrat intelligent ERC-721 avec 6 modes de minting (public gratuit/payant, proprietaire uniquement, whitelist gratuit/payant, whitelist gratuit + public payant)
- Page de minting optionnelle (HTML + ethers.js, configuree automatiquement)
- Minting par lots et airdrop vers un nombre quelconque de portefeuilles (en ligne ou depuis un fichier)
- Prise en charge de la whitelist via arbre de Merkle (genere automatiquement avec les preuves correctes)
- Analytique des detenteurs avec export JSON/CSV
- L'agent communique dans la langue de l'utilisateur

## Installation

Copiez le dossier **complet** `pharos-nft-creator-skill` dans le repertoire des skills de votre agent :

| Agent | Chemin |
|-------|--------|
| Claude Code | `~/.claude/skills/` |
| OpenClaw | `~/.openclaw/skills/` |
| Codex | `~/.codex/skills/` |

Exemple pour Claude Code :
```bash
cp -r pharos-nft-creator-skill ~/.claude/skills/
```

Structure du dossier du skill :
```
pharos-nft-creator-skill/
├── pharos-nft-creator-skill.md         — fichier principal du skill
├── dependencies.json                  — liste des dependances pour verification automatique
├── .env.example                       — modele de variables d'environnement
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

Redemarrez votre session d'agent apres l'installation.

## Premier lancement — Verifications automatiques

Lorsque vous appelez le skill pour la premiere fois, l'agent execute automatiquement la **Phase 0 : Verification des dependances** :

1. **Verification des outils** — Python, Pillow, Forge, Cast, jq, Node.js, Pinata SDK. Propose d'installer ceux qui manquent.
2. **Creation du fichier `.env`** — s'il n'existe pas dans le projet, le copie depuis `.env.example` avec des espaces reserves.
3. **Verification des variables** — verifie si `PRIVATE_KEY`, `PINATA_JWT` sont renseignes.
4. **Guide vers `.env`** — si les secrets sont vides, affiche le **chemin du fichier** et indique les lignes a remplir.

Les URL RPC Pharos et les donnees du reseau sont **integrees au Skill** — aucune configuration manuelle necessaire. Il vous suffit de choisir le reseau lors de la configuration.

Exemple de resultat :
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

## Variables d'environnement

Tous les secrets sont stockes dans le fichier `.env` a la racine du projet. **Ne collez jamais de cles directement dans le chat.**

Le fichier `.env` (cree automatiquement) :
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

### Ou obtenir les valeurs

| Variable | Source |
|----------|--------|
| `PRIVATE_KEY` | MetaMask → trois points → Details du compte → Afficher la cle privee. Coller avec ou sans `0x` — normalisee automatiquement |
| `PINATA_JWT` | [app.pinata.cloud](https://app.pinata.cloud) → API Keys → New Key → copier le JWT |

## Preparation des oeuvres

Le skill prend en charge **trois modes de creation artistique** :

### Mode A : Generatif (Traits)

Approche generative classique — plusieurs couches de traits sont combinees aleatoirement. Tous les fichiers doivent etre au format PNG, RGBA, de memes dimensions.

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
│   ├── none.png          (PNG transparent vide)
│   ├── crown.png
│   ├── cap.png
│   └── helmet.png
└── Weapon/
    ├── none.png
    ├── sword.png
    └── axe.png
```

### Mode B : Images individuelles

Un dossier de PNG pre-faits — chaque fichier = un NFT unique. Vous attribuez un poids de rarete a chacun. Fournissez une configuration JSON/CSV ou saisissez-la manuellement.

```
my-nfts/
├── 001_samurai.png     (weight: 10 — legendaire)
├── 002_ninja.png       (weight: 30 — peu commun)
├── 003_wizard.png      (weight: 5 — legendaire)
├── 004_knight.png      (weight: 55 — commun)
└── ...
```

Configuration (optionnelle, JSON) :
```json
[
  {"file": "001_samurai.png", "weight": 10, "name": "Samurai"},
  {"file": "002_ninja.png", "weight": 30, "name": "Ninja"}
]
```

### Mode C : Image unique

Un seul PNG pour tous les NFT. Chaque jeton a la meme apparence. Ideal pour les passes de membre, les billets, les NFT utilitaires.

```
collection-art.png    (image unique pour toute la collection)
```

### Poids de rarete (pour les Modes A et B)

Les poids determinent la frequence d'apparition de chaque variante de trait. Un poids plus eleve = plus courant :

| Categorie | Poids | Frequence |
|-----------|-------|-----------|
| Legendaire | 1–4 | Tres rare |
| Rare | 5–19 | Peu commun |
| Peu commun | 20–49 | Modere |
| Commun | 50–100 | Frequent |

Exemple : si Headwear a Crown(5), Cap(30), Helmet(40), None(25), Crown apparaitra dans environ 5 % des NFT generes.

## Utilisation

### 1. Demarrer la creation de collection

Dites a l'agent :
> "Deploy a generative NFT collection on Pharos"

L'agent verifiera les dependances, creera le fichier `.env` si necessaire, et commencera la configuration interactive.

### 2. Repondre aux questions de configuration

L'agent vous demandera :
- **Quel reseau utiliser** — Pharos Atlantic Testnet (gratuit, pour les tests) ou Pharos Mainnet (vrais jetons PROS)
- Le nom et le symbole de la collection
- La description
- L'offre totale (nombre maximum de NFT)
- Le type de minting (voir ci-dessous)
- **Le mode de creation artistique** — generatif (traits), images individuelles ou image unique
- Les details des oeuvres : couches de traits, ou dossier de PNG + poids, ou chemin vers un PNG unique

### 3. Types de minting

L'agent vous expliquera chaque option et vous aidera a choisir. Six types sont disponibles :

| Type | Description | Cas d'usage |
|------|-------------|-------------|
| **Public Free** | Tout le monde peut miner gratuitement | Collections communautaires ouvertes |
| **Public Paid** | Tout le monde paie un prix defini | Collections premium |
| **Owner-Only** | Seul le proprietaire du contrat peut miner | Airdrops, distributions privees |
| **Whitelist Free** | Seules les adresses en whitelist peuvent miner gratuitement | Drops communautaires exclusifs |
| **Whitelist Paid** | Seules les adresses en whitelist paient pour miner | Drops communautaires premium |
| **WL Free + Public Paid** | La whitelist mine gratuitement, les autres paient | Recompenses communautaires + vente publique |

Pour les types Whitelist, vous devez fournir une liste d'adresses (fichier ou en ligne). L'agent genere automatiquement un arbre de Merkle et le telecharge dans le contrat.

### 4. Apres le deploiement

**Miner et distribuer des NFT :**
> "Mint 50 NFTs from my collection and send them to the addresses in airdrop_list.txt"

Format du fichier d'adresses — une adresse par ligne :
```
0xAAA...BBB...CCC...
# Lines starting with # are comments
```

Ou en ligne :
> "Mint 3 NFTs and send to: 0xAAA..., 0xBBB..., 0xCCC..."

**Voir les detenteurs :**
> "Show me all holders of my NFT collection"

**Exporter vers un fichier :**
> "Export holder data to CSV"

**Configurer une whitelist :**
> "Set up a whitelist for my collection using these addresses: 0xAAA, 0xBBB"

**Mettre a jour les parametres de minting :**
> "Enable public minting at 0.05 PROS, max 5 per wallet"

## Depannage

| Probleme | Solution |
|----------|----------|
| Dimensions PNG non concordantes | Verifiez que tous les fichiers ont la meme taille, re-exportez si necessaire |
| Echec du telechargement IPFS | Verifiez `PINATA_JWT` dans `.env` et le quota de votre compte Pinata |
| **Pinata : limite de 500 fichiers** | Plan gratuit = 500 fichiers, 1 Go. Mode A : ~250 NFT (2N fichiers). Mode B : jusqu'a 500-M NFT. Mode C : jusqu'a ~500 NFT |
| Forge introuvable | Installer : `curl -L https://foundry.paradigm.xyz \| bash && foundryup` |
| Pillow introuvable | Installer : `pip install Pillow` |
| Erreur **Stack too deep** | Ajoutez `via_ir = true` dans `foundry.toml` — requis pour ce contrat |
| Erreur de compilation Forge | Assurez-vous qu'OpenZeppelin v5 est installe : `forge install https://github.com/OpenZeppelin/openzeppelin-contracts@v5.0.2 --no-git` |
| Echec de la transaction sur Pharos | Utilisez toujours le flag `--legacy` avec `forge create` et `cast send` |
| Gas insuffisant | Verifiez le solde de votre portefeuille sur Pharos |
| Le minting ne fonctionne pas | Verifiez que vous etes le proprietaire du contrat et que publicMintEnabled est true pour les collections publiques |
| `.env` ne se charge pas | Chaque commande bash doit commencer par `set -a && source .env && set +a` |
| L'image NFT ne s'affiche pas dans l'explorateur | Assurez-vous que les metadonnees utilisent `ipfs://CID` (pas d'URL de passerelle). Verifiez que le CID du dossier est correct. |

## Securite

- **Ne collez jamais** de cles privees, de jetons JWT ou de secrets dans le chat
- Tous les secrets sont stockes **uniquement dans le fichier `.env`** — l'agent le lit automatiquement
- `.env` est automatiquement ajoute a `.gitignore`
- `state.json` est egalement ajoute a `.gitignore`
- Assurez-vous de controler la cle proprietaire du contrat
- Si vous collez accidentellement un secret dans le chat — renouvelez cette cle immediatement

## Structure des fichiers de sortie

Apres l'execution du skill, votre projet contiendra :

```
project/
├── .env                         # Vos secrets (NE JAMAIS commiter !)
├── .gitignore                   # .env et state.json exclus
└── nft-collection/
    ├── traits/                  # Fichiers PNG de traits originaux (Mode A)
    ├── composed/                # Images NFT finales composees (Pillow)
    ├── metadata/                # Metadonnees JSON pour chaque jeton (nommes par ID)
    ├── src/                     # Contrat Solidity (Foundry)
    ├── landing/                 # Page web de minting (optionnel, Phase 5)
    ├── state.json               # Base de donnees locale de la collection
    └── holders.json             # Donnees des detenteurs (apres analytique)
```
