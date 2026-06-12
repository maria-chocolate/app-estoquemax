# 📦 EstoqueMax — Aplicativo de Controle de Inventário

> Aplicativo mobile desenvolvido para realizar coleta e controle de estoque com leitura de código de barras pela câmera, suporte a múltiplas datas de validade por produto e exportação de arquivos para integração com sistemas de gestão (ERP/WMS).

[![HTML](https://img.shields.io/badge/HTML5-E34F26?style=flat&logo=html5&logoColor=white)](https://developer.mozilla.org/en-US/docs/Web/HTML)
[![CSS](https://img.shields.io/badge/CSS3-1572B6?style=flat&logo=css3&logoColor=white)](https://developer.mozilla.org/en-US/docs/Web/CSS)
[![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=flat&logo=javascript&logoColor=black)](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
[![PWA](https://img.shields.io/badge/PWA-5A0FC8?style=flat&logo=pwa&logoColor=white)](https://web.dev/progressive-web-apps/)
[![IndexedDB](https://img.shields.io/badge/IndexedDB-003B57?style=flat&logo=sqlite&logoColor=white)](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)

---

## 🌐 Demo ao Vivo

| Portal | Link |
|---|---|
| 📦 EstoqueMax App | [heberthaugusto.github.io/portfolio-app-estoquemax](https://heberthaugusto.github.io/portfolio-app-estoquemax/) |

> ℹ️ Para a experiência completa, acesse pelo **Chrome no Android** e instale via "Instalar app" nos 3 pontinhos. A leitura de código de barras pela câmera requer HTTPS.

---

## 🎯 Sobre o Projeto

O **EstoqueMax** nasceu de uma necessidade real identificada durante o processo de recebimento e controle de mercadorias: os aplicativos de inventário disponíveis não permitem registrar o **mesmo produto com múltiplas datas de validade distintas** em uma única coleta — ao lançar uma segunda validade, a anterior era substituída.

### Casos de uso

**Recebimento de mercadoria** — ao receber um lote, o operador escaneia cada produto com a câmera, informa a quantidade recebida e a data de validade estampada na embalagem. Se o mesmo produto chegar com lotes de datas diferentes (comum em produtos alimentícios e de higiene), cada validade é registrada separadamente. Ao final, o app exporta os arquivos prontos para importação no sistema de gestão.

**Controle de validades** — o arquivo `_validade.txt` alimenta diretamente planilhas ou sistemas de controle de vencimento, com uma linha por lote de validade, permitindo rastrear exatamente quais unidades vencem em cada data e planejar a rotatividade do estoque com precisão.

**Inventário periódico** — o operador percorre as prateleiras escaneando os produtos. O app identifica automaticamente cada item pelo catálogo importado, registra a quantidade e gera dois arquivos ao final: um com detalhamento por validade e outro no formato de coletor para importação no ERP.

---

## ✨ Funcionalidades Implementadas

### 📷 Coleta de Estoque
- Leitura automática de código de barras pela câmera traseira (EAN-13, EAN-8, Code 128)
- Identificação instantânea do produto pelo catálogo importado via IndexedDB
- **Detecção inteligente de códigos fracionados** — produtos cujo código impresso segue o padrão `2[SKU]0000XX` são identificados automaticamente pelo fragmento de SKU, sem necessidade de inserção manual
- Inserção manual de código com busca por **SKU ou nome** — prioriza correspondência exata de SKU
- **Múltiplas validades** por produto — cada validade gera uma linha separada no arquivo exportado
- Data de validade **opcional** — suporta produtos sem validade
- 💡 **Lanterna integrada** — botão para acender/apagar a lanterna do celular diretamente na tela da câmera; desliga automaticamente ao fechar a câmera
- Edição e exclusão de lançamentos após registro
- Card com **totais em tempo real**: quantidade de produtos distintos e total de unidades contadas
- Botão flutuante de acesso rápido ao scanner

### 📂 Catálogo de Produtos
- Importação via arquivo `.txt` com separador `|` — testado com **63.000+ produtos**
- Importação assíncrona em lotes com barra de progresso — sem travamento da UI
- **Importação incremental** — produtos já cadastrados são ignorados automaticamente
- Cadastro manual com código de barras, nome, SKU, categoria e unidade
- Busca em tempo real por nome, código de barras ou SKU (IndexedDB com índices)

### 📄 Exportação
Ao exportar, **dois arquivos `.txt` são gerados simultaneamente**:

**`NOME_validade.txt`** — detalhamento completo, uma linha por lote de validade:

| Coluna | Conteúdo | Exemplo |
|---|---|---|
| 1 | SKU | `572` |
| 2 | Código de barras | `7896051115090` |
| 3 | Nome do produto | `LEITE CONDENSADO ITAMBE 1KG` |
| 4 | Data de validade (DD-MM-AA) | `09-01-27` |
| 5 | Quantidade | `48` |

```
572|7896051115090|LEITE CONDENSADO ITAMBE 1KG|09-01-27|48
572|7896051115090|LEITE CONDENSADO ITAMBE 1KG|04-04-27|60
```

**`NOME.txt`** — formato coletor para importação no ERP, quantidades totais por produto:

| Coluna | Conteúdo | Regra de formatação |
|---|---|---|
| 1 | Código de barras | 14 caracteres — espaços à direita se menor |
| 2 | Quantidade total | 5 dígitos — zeros à esquerda · soma todas as validades do produto |

```
7896051115090 ;00108
```

Integração com **Google Drive** via Web Share API nativa do Android — um toque para enviar direto para a pasta desejada.

### 🕐 Histórico de Contagens
- Registro automático das últimas **20 coletas exportadas** com nome, data/hora e quantidade de lançamentos
- Recarregamento de qualquer coleta anterior diretamente na tela de trabalho
- Confirmação obrigatória antes de substituir uma contagem em andamento — evita perda acidental de dados

---

## 🏗️ Arquitetura e Decisões Técnicas

| Decisão | Justificativa |
|---|---|
| **PWA sem framework** | Instalável como app nativo sem loja de aplicativos. Single-file HTML facilita distribuição, versionamento e deploy via GitHub Pages. |
| **IndexedDB para catálogo** | localStorage tem limite de ~5 MB — insuficiente para catálogos com 60k+ produtos (~9,5 MB). IndexedDB suporta centenas de MB sem limitação prática. |
| **localStorage para contagens** | Lançamentos ativos têm poucos registros. localStorage é síncrono e mais direto para dados pequenos e frequentemente atualizados. |
| **Índices IDB para busca** | Com 60k+ produtos, cursor scan seria lento. Índices por `barcode`, `sku` e `name` permitem lookup O(log n) — busca instantânea independente do tamanho do catálogo. |
| **Importação assíncrona em lotes** | Processar 60k registros de uma vez trava a UI. Lotes de 2.000 registros com `await new Promise(r => setTimeout(r, 0))` mantêm o browser responsivo e exibem progresso em tempo real. |
| **Detecção de códigos fracionados** | Produtos com código impresso no padrão `2[SKU]0000XX` são identificados pelo fragmento do SKU sem necessidade de recadastro, com validação por múltiplos critérios para evitar falsos positivos. |
| **html5-qrcode** | Biblioteca madura para leitura de códigos via câmera no browser. Suporta os principais formatos de código de barras usados no varejo. |
| **Web Share API** | Permite compartilhar arquivos gerados diretamente para o Google Drive sem intermediários — menos cliques para o usuário final. |
| **Service Worker** | Cache offline para funcionamento sem internet após instalação. Atualização automática de versão sem necessidade de reinstalar. |

---

## 🛠️ Stack Tecnológica

```
Frontend:         HTML5 · CSS3 · JavaScript (ES2022)
Armazenamento:    IndexedDB · localStorage
Scanner:          html5-qrcode (CDN)
Tipografia:       Space Grotesk · JetBrains Mono (Google Fonts)
Hospedagem:       GitHub Pages (HTTPS automático)
PWA:              Web App Manifest · Service Worker
Compartilhamento: Web Share API
```

---

## 📱 Como Instalar no Android

1. Acesse o link pelo **Chrome**
2. Toque nos **3 pontinhos** → **"Instalar app"**
3. O app é adicionado à tela inicial e abre em tela cheia, sem barra do navegador
4. Na primeira abertura, permita o acesso à câmera

> Funciona offline após a instalação. Atualizações são aplicadas automaticamente na próxima abertura com internet.

---

## 📁 Estrutura do Projeto

```
portfolio-app-estoquemax/
├── index.html       Aplicativo completo (single-file PWA)
├── manifest.json    Configuração PWA (nome, ícone, start_url)
├── sw.js            Service Worker (cache offline)
├── icon-192.png     Ícone para tela inicial (192×192 px)
├── icon-512.png     Ícone para splash screen (512×512 px)
└── README.md        Este arquivo
```

---

## 👤 Autor

Desenvolvido por **Heberth Augusto**

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=flat&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/heberth-dornela-ti/)
[![GitHub](https://img.shields.io/badge/GitHub-100000?style=flat&logo=github&logoColor=white)](https://github.com/heberthaugusto)

---

*Projeto desenvolvido utilizando a **Claude IA** como assistente de desenvolvimento.*
