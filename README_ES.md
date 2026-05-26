# Pharos NFT Creator — Guía de usuario

## Descripción general

Una Skill de Pharos Agent Center que permite a los agentes de IA crear colecciones NFT de ciclo completo en la blockchain de Pharos. Soporta los modos generativo (basado en rasgos), imágenes individuales e imagen única. Usted interactúa con el agente en lenguaje natural, y este se encarga de la generación de imágenes, las subidas a IPFS, el despliegue de contratos inteligentes, el minting y la distribución.

## Funcionalidades

- Verificación automática de dependencias en cada invocación (Python, Forge, Cast, Node.js, etc.)
- Creación automática del archivo `.env` con las variables de entorno
- Normalización automática de la clave privada (pegar con o sin prefijo `0x`)
- Tres modos de artwork: generativo (rasgos), imágenes individuales, imagen única
- Creación de NFTs generativos con capas de rasgos y pesos de rareza
- Composición de imágenes off-chain a partir de capas PNG usando Python Pillow
- Subida a IPFS de imágenes y carpeta de metadatos vía Pinata (SDK para archivos, REST API para carpetas)
- Patrón BaseURI — el contrato devuelve `ipfs://CID/tokenId` para cada NFT
- Datos de red Pharos integrados — elija testnet/mainnet, RPC se configura automáticamente
- Despliegue de contrato inteligente ERC-721 con 6 modos de minting (público gratuito/con costo, solo propietario, whitelist gratuita/con costo, whitelist gratuita + público con costo)
- Página de aterrizaje de minting opcional (HTML + ethers.js, configurada automáticamente)
- Minting por lotes y airdrop a cualquier cantidad de billeteras (en línea o desde archivo)
- Soporte de whitelist mediante árbol Merkle (generado automáticamente con pruebas correctas)
- Analítica de holders con exportación a JSON/CSV
- El agente se comunica en el idioma del usuario

## Instalación

Copie la carpeta **completa** `pharos-nft-creator-skill` al directorio de skills de su agente:

| Agente | Ruta |
|--------|------|
| Claude Code | `~/.claude/skills/` |
| OpenClaw | `~/.openclaw/skills/` |
| Codex | `~/.codex/skills/` |

Ejemplo para Claude Code:
```bash
cp -r pharos-nft-creator-skill ~/.claude/skills/
```

Estructura de la carpeta del skill:
```
pharos-nft-creator-skill/
├── pharos-nft-creator-skill.md         — archivo principal del skill
├── dependencies.json                  — lista de dependencias para verificación automática
├── .env.example                       — plantilla de variables de entorno
├── README.md                          — instrucciones (EN)
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

Reinicie la sesión de su agente después de la instalación.

## Primer inicio — Verificaciones automáticas

Cuando invoque el skill por primera vez, el agente ejecuta automáticamente la **Phase 0: Dependency Check**:

1. **Verifica las herramientas** — Python, Pillow, Forge, Cast, jq, Node.js, Pinata SDK. Ofrece instalar las que falten.
2. **Crea el archivo `.env`** — si no existe en el proyecto, lo copia desde `.env.example` con marcadores de posición.
3. **Verifica las variables** — comprueba si `PRIVATE_KEY` y `PINATA_JWT` están completados.
4. **Lo guía hasta `.env`** — si hay secretos vacíos, muestra la **ruta del archivo** e indica qué líneas debe completar.

Las URLs RPC de Pharos y los datos de red están **integrados en el Skill** — no se necesita configuración manual. Simplemente elija la red durante la configuración.

Ejemplo de salida:
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

## Variables de entorno

Todos los secretos se almacenan en el archivo `.env` en la raíz del proyecto. **Nunca pegue claves directamente en el chat.**

El archivo `.env` (se crea automáticamente):
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

### Dónde obtener los valores

| Variable | Origen |
|----------|--------|
| `PRIVATE_KEY` | MetaMask → tres puntos → Detalles de la cuenta → Mostrar clave privada. Pegar con o sin `0x` — se normaliza automáticamente |
| `PINATA_JWT` | [app.pinata.cloud](https://app.pinata.cloud) → API Keys → New Key → copiar el JWT |

## Preparación del artwork

El skill soporta **tres modos de artwork**:

### Modo A: Generativo (Rasgos)

Enfoque generativo clásico — múltiples capas de rasgos se combinan aleatoriamente. Todos los archivos deben ser PNG, RGBA, con las mismas dimensiones.

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

### Modo B: Imágenes individuales

Una carpeta de PNGs prehechos — cada archivo = un NFT único. Usted asigna un peso de rareza a cada uno. Proporcione una configuración JSON/CSV o ingrésela manualmente.

```
my-nfts/
├── 001_samurai.png     (weight: 10 — legendary)
├── 002_ninja.png       (weight: 30 — uncommon)
├── 003_wizard.png      (weight: 5 — legendary)
├── 004_knight.png      (weight: 55 — common)
└── ...
```

Configuración (opcional, JSON):
```json
[
  {"file": "001_samurai.png", "weight": 10, "name": "Samurai"},
  {"file": "002_ninja.png", "weight": 30, "name": "Ninja"}
]
```

### Modo C: Imagen única

Un PNG para todos los NFTs. Cada token se ve igual. Ideal para pases de membresía, entradas, NFTs de utilidad.

```
collection-art.png    (single image for the entire collection)
```

### Pesos de rareza (para los Modos A y B)

Los pesos determinan la frecuencia con la que aparece cada variante de rasgo. Mayor peso = más común:

| Categoría | Peso | Frecuencia |
|-----------|------|------------|
| Legendario | 1–4 | Muy raro |
| Raro | 5–19 | Poco común |
| Poco común | 20–49 | Moderado |
| Común | 50–100 | Frecuente |

Ejemplo: si Headwear tiene Crown(5), Cap(30), Helmet(40), None(25), Crown aparecerá en aproximadamente el 5% de los NFTs generados.

## Uso

### 1. Iniciar la creación de la colección

Dígale al agente:
> "Deploy a generative NFT collection on Pharos"

El agente verificará las dependencias, creará el `.env` si es necesario e iniciará la configuración interactiva.

### 2. Responder las preguntas de configuración

El agente le preguntará sobre:
- **Qué red utilizar** — Pharos Atlantic Testnet (gratuita, para pruebas) o Pharos Mainnet (tokens PROS reales)
- Nombre y símbolo de la colección
- Descripción
- Suministro total (número máximo de NFTs)
- Tipo de minting (ver más abajo)
- **Modo de artwork** — generativo (rasgos), imágenes individuales o imagen única
- Detalles del artwork: capas de rasgos, o carpeta de PNGs con pesos, o ruta a un PNG único

### 3. Tipos de minting

El agente le explicará cada opción y le ayudará a elegir. Hay seis tipos disponibles:

| Tipo | Descripción | Caso de uso |
|------|-------------|-------------|
| **Public Free** | Cualquiera puede hacer mint de forma gratuita | Colecciones comunitarias abiertas |
| **Public Paid** | Cualquiera paga un precio establecido | Colecciones premium |
| **Owner-Only** | Solo el propietario del contrato puede hacer mint | Airdrops, distribuciones privadas |
| **Whitelist Free** | Solo las direcciones en whitelist pueden hacer mint gratis | Drops exclusivos para la comunidad |
| **Whitelist Paid** | Solo las direcciones en whitelist pagan para hacer mint | Drops premium para la comunidad |
| **WL Free + Public Paid** | Whitelist hace mint gratis, todos los demás pagan | Recompensas comunitarias + venta pública |

Para los tipos con Whitelist, debe proporcionar una lista de direcciones (archivo o en línea). El agente genera automáticamente un árbol Merkle y lo sube al contrato.

### 4. Después del despliegue

**Hacer mint y distribuir NFTs:**
> "Mint 50 NFTs from my collection and send them to the addresses in airdrop_list.txt"

Formato del archivo de direcciones — una dirección por línea:
```
0xAAA...BBB...CCC...
# Lines starting with # are comments
```

O en línea:
> "Mint 3 NFTs and send to: 0xAAA..., 0xBBB..., 0xCCC..."

**Ver holders:**
> "Show me all holders of my NFT collection"

**Exportar a archivo:**
> "Export holder data to CSV"

**Configurar whitelist:**
> "Set up a whitelist for my collection using these addresses: 0xAAA, 0xBBB"

**Actualizar parámetros de minting:**
> "Enable public minting at 0.05 PROS, max 5 per wallet"

## Solución de problemas

| Problema | Solución |
|----------|----------|
| Dimensiones PNG incompatibles | Verifique que todos los archivos tengan el mismo tamaño, reexporte si es necesario |
| Fallo en la subida a IPFS | Compruebe `PINATA_JWT` en `.env` y la cuota de su cuenta de Pinata |
| **Pinata: límite de 500 archivos** | Plan gratuito = 500 archivos, 1 GB. Modo A: ~250 NFT (2N archivos). Modo B: hasta 500-M NFTs. Modo C: hasta ~500 NFTs |
| Forge no encontrado | Instalar: `curl -L https://foundry.paradigm.xyz \| bash && foundryup` |
| Pillow no encontrado | Instalar: `pip install Pillow` |
| Error **Stack too deep** | Agregue `via_ir = true` a `foundry.toml` — necesario para este contrato |
| Error de compilación de Forge | Asegúrese de que OpenZeppelin v5 esté instalado: `forge install https://github.com/OpenZeppelin/openzeppelin-contracts@v5.0.2 --no-git` |
| La transacción falla en Pharos | Use siempre la bandera `--legacy` con `forge create` y `cast send` |
| Gas insuficiente | Verifique el saldo de su billetera en Pharos |
| El minting no funciona | Verifique que usted sea el propietario del contrato y que publicMintEnabled sea true para colecciones públicas |
| `.env` no se carga | Cada comando bash debe comenzar con `set -a && source .env && set +a` |
| La imagen del NFT no se muestra en el explorador | Asegúrese de que los metadatos usen `ipfs://CID` (no URLs de gateway). Verifique que el CID de la carpeta sea correcto. |

## Seguridad

- **Nunca pegue** claves privadas, tokens JWT ni ningún secreto en el chat
- Todos los secretos se almacenan **únicamente en el archivo `.env`** — el agente lo lee automáticamente
- `.env` se agrega automáticamente a `.gitignore`
- `state.json` también se agrega a `.gitignore`
- Asegúrese de controlar la clave propietaria del contrato
- Si accidentalmente pega un secreto en el chat — rote esa clave inmediatamente

## Estructura de archivos de salida

Después de ejecutar el skill, su proyecto contendrá:

```
project/
├── .env                         # Tus secretos (¡NUNCA haga commit!)
├── .gitignore                   # .env y state.json excluidos
└── nft-collection/
    ├── traits/                  # Archivos PNG de rasgos originales (Modo A)
    ├── composed/                # Imágenes NFT finales compuestas (Pillow)
    ├── metadata/                # Metadatos JSON para cada token (nombrados por ID)
    ├── src/                     # Contrato Solidity (Foundry)
    ├── landing/                 # Página web de minting (opcional, Phase 5)
    ├── state.json               # Base de datos local de la colección
    └── holders.json             # Datos de holders (tras la analítica)
```
