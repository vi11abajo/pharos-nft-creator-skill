# Pharos NFT Creator — Guia do Usuário

## Visão Geral

Uma Pharos Agent Center Skill que permite a agentes de IA criar coleções NFT de ciclo completo na blockchain Pharos. Suporta os modos generativo (baseado em traits), imagens individuais e imagem única. Você interage com o agente em linguagem natural, e ele cuida da geração de imagens, uploads no IPFS, implantação de contratos inteligentes, minting e distribuição.

## Funcionalidades

- Verificação automática de dependências a cada invocação (Python, Forge, Cast, Node.js, etc.)
- Criação automática do arquivo `.env` com variáveis de ambiente
- Normalização automática da chave privada (cole com ou sem o prefixo `0x`)
- Três modos de arte: generativo (traits), imagens individuais, imagem única
- Criação de NFTs generativos com camadas de traits e pesos de raridade
- Composição de imagens off-chain a partir de camadas PNG usando Python Pillow
- Upload de imagens + pasta de metadados para o IPFS via Pinata (SDK para arquivos, REST API para pastas)
- Padrão BaseURI — o contrato retorna `ipfs://CID/tokenId` para cada NFT
- Dados da rede Pharos integrados — escolha testnet/mainnet, RPC configurado automaticamente
- Implantação de contrato inteligente ERC-721 com 6 modos de mint (público gratuito/pago, somente owner, whitelist gratuito/pago, whitelist gratuito + público pago)
- Landing page de minting opcional (HTML + ethers.js, configurada automaticamente)
- Minting em lote e airdrop para qualquer quantidade de carteiras (inline ou via arquivo)
- Suporte a whitelist via Merkle tree (gerada automaticamente com provas corretas)
- Análise de holders com exportação em JSON/CSV
- O agente se comunica no idioma do usuário

## Instalação

Copie a pasta **inteira** `pharos-nft-creator-skill` para o diretório de skills do seu agente:

| Agente | Caminho |
|--------|---------|
| Claude Code | `~/.claude/skills/` |
| OpenClaw | `~/.openclaw/skills/` |
| Codex | `~/.codex/skills/` |

Exemplo para Claude Code:
```bash
cp -r pharos-nft-creator-skill ~/.claude/skills/
```

Estrutura de pastas da skill:
```
pharos-nft-creator-skill/
├── pharos-nft-creator-skill.md         — arquivo principal da skill
├── dependencies.json                  — lista de dependências para verificação automática
├── .env.example                       — modelo de variáveis de ambiente
├── README.md                          — instruções (EN)
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

Reinicie a sessão do seu agente após a instalação.

## Primeira Execução — Verificações Automáticas

Ao invocar a skill pela primeira vez, o agente executa automaticamente a **Fase 0: Verificação de Dependências**:

1. **Verifica as ferramentas** — Python, Pillow, Forge, Cast, jq, Node.js, Pinata SDK. Oferece-se para instalar as que estiverem faltando.
2. **Cria o arquivo `.env`** — se não existir no projeto, copia a partir do `.env.example` com espaços reservados.
3. **Verifica as variáveis** — checa se `PRIVATE_KEY` e `PINATA_JWT` estão preenchidas.
4. **Orienta você até o `.env`** — se os segredos estiverem vazios, mostra o **caminho do arquivo** e indica quais linhas preencher.

As URLs de RPC e os dados da rede Pharos estão **embutidos na Skill** — nenhuma configuração manual é necessária. Basta escolher a rede durante a configuração.

Exemplo de saída:
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

## Variáveis de Ambiente

Todos os segredos são armazenados no arquivo `.env` na raiz do projeto. **Nunca cole chaves diretamente no chat.**

O arquivo `.env` (criado automaticamente):
```bash
# Sua chave privada da carteira.
# Cole com ou sem o prefixo 0x — a skill normaliza automaticamente.
# SEGURANÇA: Preencha o arquivo diretamente. NÃO cole no chat.
PRIVATE_KEY=

# Token JWT do Pinata — como obter:
#   1. Acesse https://app.pinata.cloud → cadastre-se / faça login
#   2. Barra lateral esquerda → "API Keys" → "New Key"
#   3. Dê um nome (ex.: "nft-builder"), mantenha as permissões padrão
#   4. Clique em "Create Key" → copie o JWT (texto longo começando com "eyJ...")
#   5. Cole abaixo
PINATA_JWT=

# A rede é selecionada durante a configuração — não é necessário definir a URL do RPC manualmente.
```

### Onde obter os valores

| Variável | Origem |
|----------|--------|
| `PRIVATE_KEY` | MetaMask → três pontinhos → Detalhes da conta → Mostrar chave privada. Cole com ou sem `0x` — normalização automática |
| `PINATA_JWT` | [app.pinata.cloud](https://app.pinata.cloud) → API Keys → New Key → copie o JWT |

## Preparando a Arte

A skill suporta **três modos de arte**:

### Modo A: Generativo (Traits)

Abordagem generativa clássica — múltiplas camadas de traits são combinadas aleatoriamente. Todos os arquivos devem ser PNG, RGBA, com as mesmas dimensões.

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
│   ├── none.png          (PNG transparente vazio)
│   ├── crown.png
│   ├── cap.png
│   └── helmet.png
└── Weapon/
    ├── none.png
    ├── sword.png
    └── axe.png
```

### Modo B: Imagens Individuais

Uma pasta de PNGs prontos — cada arquivo = um NFT único. Você atribui um peso de raridade a cada um. Forneça uma configuração JSON/CSV ou insira manualmente.

```
my-nfts/
├── 001_samurai.png     (peso: 10 — lendário)
├── 002_ninja.png       (peso: 30 — incomum)
├── 003_wizard.png      (peso: 5 — lendário)
├── 004_knight.png      (peso: 55 — comum)
└── ...
```

Configuração (opcional, JSON):
```json
[
  {"file": "001_samurai.png", "weight": 10, "name": "Samurai"},
  {"file": "002_ninja.png", "weight": 30, "name": "Ninja"}
]
```

### Modo C: Imagem Única

Um único PNG para todos os NFTs. Cada token fica idêntico. Ideal para passes de associação, ingressos, NFTs utilitários.

```
collection-art.png    (imagem única para toda a coleção)
```

### Pesos de Raridade (para os Modos A e B)

Os pesos determinam a frequência com que cada variante de trait aparece. Peso maior = mais comum:

| Categoria | Peso | Frequência |
|-----------|------|------------|
| Lendário | 1–4 | Muito raro |
| Raro | 5–19 | Incomum |
| Incomum | 20–49 | Moderado |
| Comum | 50–100 | Frequente |

Exemplo: se Headwear tiver Crown(5), Cap(30), Helmet(40), None(25), Crown aparecerá em aproximadamente 5% dos NFTs gerados.

## Uso

### 1. Iniciar a Criação da Coleção

Diga ao agente:
> "Deploy a generative NFT collection on Pharos"

O agente verificará as dependências, criará o `.env` se necessário e iniciará a configuração interativa.

### 2. Responder às Perguntas de Configuração

O agente perguntará sobre:
- **Qual rede usar** — Pharos Atlantic Testnet (gratuita, para testes) ou Pharos Mainnet (tokens PROS reais)
- Nome e símbolo da coleção
- Descrição
- Fornecimento total (número máximo de NFTs)
- Tipo de mint (veja abaixo)
- **Modo de arte** — generativo (traits), imagens individuais ou imagem única
- Detalhes da arte: camadas de traits, ou pasta de PNGs + pesos, ou caminho para um PNG único

### 3. Tipos de Mint

O agente explicará cada opção e ajudará você a escolher. Seis tipos estão disponíveis:

| Tipo | Descrição | Caso de uso |
|------|-----------|-------------|
| **Public Free** | Qualquer pessoa pode fazer mint gratuitamente | Coleções comunitárias abertas |
| **Public Paid** | Qualquer pessoa paga um preço definido | Coleções premium |
| **Owner-Only** | Apenas o dono do contrato pode fazer mint | Airdrops, distribuições privadas |
| **Whitelist Free** | Apenas endereços da whitelist podem fazer mint gratuitamente | Drops exclusivos para a comunidade |
| **Whitelist Paid** | Apenas endereços da whitelist pagam para fazer mint | Drops premium para a comunidade |
| **WL Free + Public Paid** | Whitelist faz mint grátis, os demais pagam | Recompensas para a comunidade + venda pública |

Para os tipos com Whitelist, você precisa fornecer uma lista de endereços (arquivo ou inline). O agente gera automaticamente uma Merkle tree e a envia ao contrato.

### 4. Após a Implantação

**Fazer mint e distribuir NFTs:**
> "Mint 50 NFTs from my collection and send them to the addresses in airdrop_list.txt"

Formato do arquivo de endereços — um endereço por linha:
```
0xAAA...BBB...CCC...
# Linhas que começam com # são comentários
```

Ou inline:
> "Mint 3 NFTs and send to: 0xAAA..., 0xBBB..., 0xCCC..."

**Visualizar holders:**
> "Show me all holders of my NFT collection"

**Exportar para arquivo:**
> "Export holder data to CSV"

**Configurar whitelist:**
> "Set up a whitelist for my collection using these addresses: 0xAAA, 0xBBB"

**Atualizar parâmetros de mint:**
> "Enable public minting at 0.05 PROS, max 5 per wallet"

## Solução de Problemas

| Problema | Solução |
|----------|---------|
| Dimensões dos PNGs incompatíveis | Verifique se todos os arquivos têm o mesmo tamanho, exporte novamente se necessário |
| Falha no upload para IPFS | Verifique o `PINATA_JWT` no `.env` e a cota da conta Pinata |
| **Pinata: limite de 500 arquivos** | Plano gratuito = 500 arquivos, 1 GB. Modo A: ~250 NFTs (2N arquivos). Modo B: até 500-M NFTs. Modo C: até ~500 NFTs |
| Forge não encontrado | Instale: `curl -L https://foundry.paradigm.xyz \| bash && foundryup` |
| Pillow não encontrado | Instale: `pip install Pillow` |
| Erro **Stack too deep** | Adicione `via_ir = true` ao `foundry.toml` — obrigatório para este contrato |
| Erro de compilação do Forge | Certifique-se de que o OpenZeppelin v5 está instalado: `forge install https://github.com/OpenZeppelin/openzeppelin-contracts@v5.0.2 --no-git` |
| Transação falha na Pharos | Sempre use a flag `--legacy` com `forge create` e `cast send` |
| Gas insuficiente | Verifique o saldo da carteira na Pharos |
| Mint não funciona | Verifique se você é o dono do contrato e se publicMintEnabled está true para coleções públicas |
| `.env` não está carregando | Cada comando bash deve começar com `set -a && source .env && set +a` |
| Imagem do NFT não aparece no explorer | Certifique-se de que os metadados usam `ipfs://CID` (não URLs de gateway). Verifique se o CID da pasta está correto. |

## Segurança

- **Nunca cole** chaves privadas, tokens JWT ou quaisquer segredos no chat
- Todos os segredos são armazenados **apenas no arquivo `.env`** — o agente o lê automaticamente
- `.env` é adicionado automaticamente ao `.gitignore`
- `state.json` também é adicionado ao `.gitignore`
- Certifique-se de ter controle sobre a chave de owner do contrato
- Se você acidentalmente colar um segredo no chat — rotacione essa chave imediatamente

## Estrutura de Arquivos de Saída

Após a execução da skill, seu projeto conterá:

```
project/
├── .env                         # Seus segredos (NUNCA faça commit!)
├── .gitignore                   # .env e state.json excluídos
└── nft-collection/
    ├── traits/                  # Arquivos PNG originais de traits (Modo A)
    ├── composed/                # Imagens NFT finais compostas (Pillow)
    ├── metadata/                # Metadados JSON para cada token (nomeados por ID)
    ├── src/                     # Contrato Solidity (Foundry)
    ├── landing/                 # Página web de minting (opcional, Fase 5)
    ├── state.json               # Banco de dados local da coleção
    └── holders.json             # Dados de holders (após análise)
```
