# EstoqueMax — Aplicativo de Controle de Inventário

Aplicativo mobile para coleta e controle de estoque, desenvolvido como PWA (Progressive Web App). Funciona diretamente no navegador do dispositivo, sem necessidade de instalação via loja de aplicativos.

---

## Visão Geral

O EstoqueMax resolve um problema específico do processo de inventário: registrar o mesmo produto com **múltiplas datas de validade distintas** em uma única coleta — funcionalidade ausente nos aplicativos comerciais avaliados.

O aplicativo é instalável na tela inicial do celular Android, funciona offline após o primeiro acesso e exporta os arquivos de coleta nos formatos exigidos pelo sistema de gestão.

---

## Funcionalidades

### Coleta de Estoque
- Leitura de código de barras pela câmera traseira do dispositivo
- Identificação automática do produto pelo catálogo importado
- Inserção manual de código de barras com busca por SKU ou nome do produto
- Registro de múltiplas validades para o mesmo produto
- Data de validade opcional (produtos sem validade são suportados)
- Edição e exclusão de lançamentos após registro
- Botão flutuante de acesso rápido ao scanner

### Catálogo de Produtos
- Importação via arquivo `.txt` com separador `|` (pipeline)
- Suporte a catálogos grandes (testado com até 63.000 produtos)
- Importação incremental — produtos já cadastrados são ignorados automaticamente
- Cadastro manual de produtos (código de barras, nome, SKU, categoria, unidade)
- Busca em tempo real por nome, código de barras ou SKU

### Exportação
Ao exportar, dois arquivos `.txt` são gerados simultaneamente:

**Arquivo de validades** (`NOME_validade.txt`) — 5 colunas separadas por `|`:
```
SKU|CÓDIGO DE BARRAS|NOME DO PRODUTO|VALIDADE|QUANTIDADE
572|7896051115090|LEITE CONDENSADO ITAMBE 1KG|09-01-27|48
```

**Arquivo coletor** (`NOME.txt`) — 2 colunas separadas por `;`:
```
7896051115090 ;00048
```
> Código de barras: 14 caracteres com padding de espaços à direita
> Quantidade: 5 dígitos com padding de zeros à esquerda

A exportação pode ser feita diretamente para o **Google Drive** via compartilhamento nativo do Android.

### Histórico
- Registro automático das últimas 20 coletas exportadas
- Possibilidade de recarregar uma coleta anterior para consulta ou reexportação

---

## Formato do Arquivo de Catálogo

```
CÓDIGO_DE_BARRAS|NOME DO PRODUTO|SKU
7896051115090|LEITE CONDENSADO ITAMBE 1KG|572
7891008405019|BALA TOFEE GAROTO 200G|1234
2686|FITA CETIM LISA 10M|2686
```

- Separador: `|` (pipeline)
- Coluna 1: código de barras
- Coluna 2: nome do produto
- Coluna 3: SKU (opcional)
- Linhas inválidas são ignoradas automaticamente na importação

---

## Instalação no Dispositivo (Android)

1. Acesse o link do aplicativo pelo **Chrome**
2. Toque nos **3 pontinhos** → **"Instalar app"**
3. Confirme a instalação
4. O app é adicionado à tela inicial e abre em tela cheia

Na primeira abertura, o Service Worker faz cache dos arquivos para funcionamento offline.

---

## Estrutura dos Arquivos

```
/
├── index.html       Aplicativo completo (single-file PWA)
├── manifest.json    Configuração PWA (nome, ícone, tema, URL de início)
├── sw.js            Service Worker (cache offline)
├── icon-192.png     Ícone para tela inicial (192×192 px)
├── icon-512.png     Ícone para splash screen (512×512 px)
└── README.md        Este arquivo
```

---

## Configuração do GitHub Pages

O aplicativo é hospedado via GitHub Pages. Para configurar:

1. Acesse **Settings → Pages** no repositório
2. Em *Source*: selecione **Deploy from a branch**
3. Branch: **main** → pasta: **/ (root)**
4. Clique em **Save**

Após 1–2 minutos o aplicativo estará disponível na URL do repositório.

> **Atenção:** o arquivo `sw.js` contém o caminho `/appestoque/` hardcoded. Ao trocar de repositório, atualizar esse caminho para corresponder ao nome do novo repositório. O mesmo vale para o campo `"start_url"` no `manifest.json`.

---

## Dados e Armazenamento

Todos os dados são armazenados exclusivamente no dispositivo do usuário:

| Dado | Armazenamento | Limite |
|---|---|---|
| Catálogo de produtos | IndexedDB | Sem limite prático (~centenas de MB) |
| Lançamentos de contagem | localStorage | Pequeno (poucos KB) |
| Histórico de coletas | localStorage | Últimas 20 coletas |
| Nome da contagem | localStorage | — |

Nenhuma informação é enviada para servidores externos. O aplicativo não requer autenticação, conta de usuário ou conexão com a internet após o primeiro carregamento.

---

## Observações Técnicas

- O catálogo utiliza **IndexedDB** com índices por `barcode`, `sku` e `name` para busca rápida
- A leitura de código de barras requer acesso via **HTTPS** (obrigatório pelo Chrome)
- GitHub Pages fornece HTTPS automaticamente — não é necessária nenhuma configuração adicional
- Atualizar o catálogo via nova importação não apaga lançamentos de estoque existentes
- O campo "Limpar Catálogo" e o campo "Limpar Toda a Contagem" possuem confirmação obrigatória antes de executar
