# 🗳️ VotoPovo

> **Fork do Freedom Tool (Rarimo) adaptado para o Brasil — pesquisa de opinião pública descentralizada, anônima e antifraude usando título de eleitor.**

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](https://opensource.org/licenses/MIT)
[![Base: Freedom Tool](https://img.shields.io/badge/Base-Freedom%20Tool%20(Rarimo)-blueviolet)](https://github.com/rarimo/FreedomTool)
[![Auditado: Halborn](https://img.shields.io/badge/Contratos-Auditados%20Halborn-orange)](https://www.halborn.com/audits/rarimo/rarimo-voting-contracts)
[![Status: RFC](https://img.shields.io/badge/Status-RFC%20Aberto-blue)]()
[![Blockchain: Polygon](https://img.shields.io/badge/Blockchain-Polygon-purple)]()

---

## O Problema

As pesquisas de opinião publicadas no Brasil não podem ser auditadas. Quem financiou? Qual a metodologia real? Os dados existem para checagem? A resposta é sempre a mesma: não.

**O VotoPovo resolve isso com código, não com promessas.**

---

## Por que um Fork do Freedom Tool?

O **Freedom Tool** da Rarimo é open source (MIT) e já resolveu os problemas mais difíceis:

- ✅ Arquitetura ZK para votação anônima verificável — **pronta e testada**
- ✅ Smart contracts auditados pela **Halborn Security**
- ✅ Prova de conceito real em 3 países: Russia2024 (15k downloads dia 1), Iranians Vote, United Space (Geórgia)
- ✅ App iOS funcional, Web UI React funcional
- ✅ Circuitos Circom com geração de prova no device

**O único problema:** usa passaporte biométrico com chip NFC. O Brasil usa título de eleitor, sem chip NFC.

**A solução:** fork do Freedom Tool trocando o documento de entrada. Toda a arquitetura ZK, os smart contracts, e o sistema de nullifiers permanecem intactos.

---

## A Diferença Técnica

```
FREEDOM TOOL ORIGINAL     →    VOTOPOVO (fork)
─────────────────────────────────────────────────
Passaporte + chip NFC     →    Título de eleitor
Leitura NFC               →    OCR local (Tesseract.js)
Verificação PKI/CSCA      →    Layout validation + face match
Merkle Tree: Rarimo state →    Merkle Tree: base pública TSE
Qualquer país             →    Apenas eleitores brasileiros
```

**Tudo o mais é idêntico.**

---

## Como Funciona

### Para não-técnicos

```
1. Você fotografa seu título de eleitor
   → Processado no SEU celular, não vai para nenhum servidor

2. Você faz uma selfie ao vivo
   → Prova que é humano real, comparada com o título localmente

3. Seu celular gera uma prova matemática invisível
   → Prova que você é eleitor válido sem revelar quem você é

4. A prova + seu voto anônimo vão para a blockchain
   → Imutável. Nem nós conseguimos alterar.

5. O resultado aparece em tempo real
   → Qualquer pessoa no mundo pode verificar
```

### Para técnicos

**Fase de registro (uma vez por usuário):**
```
Título de eleitor → OCR (Tesseract.js) → extrai número, zona, seção
                 → Liveness check (FaceTec 3D) → face match vs. título
                 → identity commitment gerado localmente
                 → commitment → Registration.sol → Poseidon Sparse Merkle Tree
```

**Fase de votação:**
```
Opção escolhida → Circom circuit (client-side proving via ultragroth)
               → ZK Proof: título válido + nullifier não usado
               → Proof + nullifier + opção → Voting.sol
               → Contrato verifica → nullifier registrado → contador++
               → Resultado auditável em tempo real (The Graph)
```

**O Nullifier (anti-duplo voto):**
```javascript
// Gerado 100% no device, nunca enviado para servidor
// Muda por pesquisa → impossível cruzar quem votou em pesquisas diferentes
nullifier = poseidon(numero_titulo + zona + secao + id_pesquisa)
```

---

## Repositórios do Projeto

| Repositório | Origem | Status |
|---|---|---|
| `votopovo/voting-contracts` | Fork de `rarimo/voting-contracts` | 🔄 Adaptar RegisterVerifier |
| `votopovo/web-ui` | Fork de `rarimo/freedomtool-web-ui` | 🔄 Substituir passport scan |
| `votopovo/ios-app` | Fork de `rarimo/FreedomTool` | 🔄 Substituir NFC por OCR |
| `votopovo/merkle-tree-br` | Novo | 🔄 Gerar árvore da base TSE |
| `votopovo/docs` | Novo | ✅ PRD + arquitetura |

---

## Stack Tecnológico

| Camada | Tecnologia | Origem |
|---|---|---|
| Smart Contracts | Solidity (Rarimo) | Fork |
| ZK Circuits | Circom + SnarkJS (Rarimo) | Fork adaptado |
| Prover | ultragroth (C++ client-side) | Rarimo direto |
| OCR | Tesseract.js 5.x | Novo |
| Liveness | FaceTec SDK | Novo |
| Frontend | React + TypeScript | Fork adaptado |
| App iOS | Swift (Xcode) | Fork adaptado |
| Blockchain | Polygon | Mantido |
| Merkle Tree | Poseidon SMT | Mantido |
| Indexação | The Graph | Mantido |
| Hospedagem | IPFS + ENS | Mantido |

---

## Segurança

Os smart contracts base foram **auditados pela Halborn Security**. O relatório completo está disponível publicamente.

As modificações do VotoPovo (RegisterVerifier adaptado para títulos brasileiros) passarão por **auditoria independente** antes do deploy em mainnet.

### Garantias do sistema

- Nenhum dado pessoal sai do device — OCR, liveness e geração de prova são 100% locais
- Anonimato matemático — ZK Proofs provam elegibilidade sem revelar identidade
- Unicidade verificável — Nullifiers únicos por pesquisa, verificados on-chain
- Resultados imutáveis — Smart contract sem função de alteração pós-deploy
- Código auditável — 100% open source

---

## Como Rodar Localmente

```bash
# Clone
git clone https://github.com/votopovo/votopovo.git
cd votopovo

# Contratos
cd contracts && npm install
npx hardhat node                          # rede local
npx hardhat run scripts/deploy.ts --network localhost
npx hardhat test

# Frontend
cd ../web-ui && npm install
cp .env.example .env.local               # configurar endereços dos contratos
npm run dev                              # http://localhost:3000
```

---

## Estrutura do Repositório

```
votopovo/
├── contracts/                 # Fork rarimo/voting-contracts
│   ├── src/
│   │   ├── Registration.sol   # Registro de eleitores (Rarimo)
│   │   ├── Voting.sol         # Lógica de votação (Rarimo)
│   │   ├── VotingRegistry.sol # Factory de pesquisas (Rarimo)
│   │   └── TituloVerifier.sol # NOVO: verificador para título BR
│   └── test/
│
├── circuits/                  # Fork rarimo/passport-zk-circuits
│   ├── titulo_register.circom # ADAPTADO: sem lógica NFC
│   └── titulo_vote.circom     # ADAPTADO: Merkle proof TSE
│
├── web-ui/                    # Fork rarimo/freedomtool-web-ui
│   ├── src/
│   │   ├── components/
│   │   │   ├── TituloScan/    # NOVO: fluxo de OCR do título
│   │   │   ├── LivenessCheck/ # NOVO: face match
│   │   │   └── Pesquisas/     # ADAPTADO: lista de pesquisas BR
│   │   └── lib/
│   │       ├── ocr.ts         # NOVO: Tesseract.js integration
│   │       └── merkle-br.ts   # NOVO: Merkle Tree TSE
│
├── merkle-tree-br/            # NOVO: geração da árvore de eleitores
│   └── build-tree.ts          # A partir da base pública TSE
│
└── docs/
    ├── PRD.md
    └── ARQUITETURA.md
```

---

## Como Contribuir

**Este projeto é da comunidade brasileira. Não tem empresa. Não tem partido. Não tem servidor.**

### O que precisa ser feito agora

As tarefas mais críticas para o MVP:

**1. Adaptar o RegisterVerifier (Solidity)**  
Substituir a verificação PKI de passaportes por verificação de Merkle proof para títulos brasileiros. Esta é a mudança mais crítica.

**2. Adaptar o circuito Circom**  
Remover a lógica de verificação NFC/CSCA, manter a lógica de Merkle proof e nullifier. Pode ser um trabalho menor que o esperado se a estrutura do Freedom Tool for modular o suficiente.

**3. OCR do título de eleitor (JavaScript)**  
Integração do Tesseract.js com lógica de validação do layout específico do título de eleitor brasileiro.

**4. Script da Merkle Tree (Node.js)**  
Gerar Poseidon Sparse Merkle Tree compatível com os contratos Rarimo a partir da base pública de eleitores do TSE.

**5. Fork do Web UI (React/TypeScript)**  
Substituir o fluxo de scan do passaporte NFC pelo fluxo de fotografia + OCR do título.

### Processo de contribuição

```bash
# 1. Fork o repositório
# 2. Crie uma branch
git checkout -b feature/minha-feature

# 3. Desenvolva e teste
# 4. Pull Request com descrição clara
# 5. Review mínimo de 2 aprovações
# 6. Merge
```

---

## Financiamento

Projeto open source sem fins lucrativos. Custo ~30% menor que uma implementação do zero porque os contratos principais já foram auditados pela Halborn.

| Item | Estimativa (USD) |
|---|---|
| Auditoria das modificações nos contratos | $10k – $20k |
| Auditoria do circuito ZK adaptado | $8k – $15k |
| Infraestrutura (6 meses) | $2k – $5k |
| Bug bounty program | $10k – $30k |
| **Total recomendado** | **~$70.000** |

Contribuições financeiras → Gnosis Safe multisig público, transparência total.  
Nenhuma contribuição financeira dá poder sobre o protocolo.

---

## Roadmap

- [x] 📋 Documentação de arquitetura (PRD + README)
- [ ] 🔱 Fork dos repositórios base da Rarimo
- [ ] 📄 OCR do título de eleitor no browser
- [ ] 🌳 Script Merkle Tree a partir de dados TSE
- [ ] 🔐 Adaptação do RegisterVerifier
- [ ] ⚡ Adaptação do circuito Circom
- [ ] 🧪 Deploy + testes na testnet
- [ ] 🔍 Auditoria de segurança
- [ ] 🚀 Deploy Polygon Mainnet
- [ ] 📱 App mobile (iOS + Android)
- [ ] 🌐 Frontend no IPFS (votopovo.eth)
- [ ] 🏛️ Formação da DAO

---

## Perguntas Frequentes

**Por que não usar o Freedom Tool direto?**  
Porque o Freedom Tool usa passaporte biométrico com chip NFC para verificação. A maioria dos brasileiros não usa passaporte como documento principal — o título de eleitor é o documento eleitoral do país.

**E se passarmos a usar passaporte no Brasil?**  
A migração seria trivial: apenas reverter para o Freedom Tool original, mantendo toda a arquitetura. O VotoPovo é estruturalmente compatível.

**O sistema é mais fraco que o Freedom Tool por não ter NFC?**  
Sim, criptograficamente. Mas suficiente para o objetivo: o custo de fraude por voto é alto o suficiente para tornar manipulação em escala inviável. E é infinitamente mais auditável que qualquer pesquisa tradicional.

**Meus dados do título ficam salvos em algum lugar?**  
Não. O processamento ocorre 100% no seu device. Nenhuma imagem, nenhum número de título, nenhuma foto sai do seu celular.

**Quem controla as pesquisas publicadas?**  
No MVP: um comitê multisig público. Na fase DAO: a comunidade por votação.

---

## Referências e Créditos

- **Freedom Tool (Rarimo):** https://github.com/rarimo/FreedomTool — base do projeto, licença MIT
- **voting-contracts:** https://github.com/rarimo/voting-contracts
- **Halborn Audit:** https://www.halborn.com/audits/rarimo/rarimo-voting-contracts
- **Russia2024:** https://russia2024.world/ — caso de uso real do Freedom Tool

---

## Licença

MIT License — o mesmo do Freedom Tool original.

Você pode usar, modificar e distribuir livremente, inclusive comercialmente. A única condição é manter a atribuição.

---

<div align="center">

**Feito no Brasil 🇧🇴 pela comunidade brasileira**

*Fork do Freedom Tool — nenhuma empresa, nenhum partido, nenhum servidor*

*"A cypherpunk promise to defend privacy with cryptography"*

</div>
