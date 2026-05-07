# Sistema de Certificados Mave - Documentacao

## Visao Geral

Sistema web para geracao de **Certificados de Conformidade** e **Fichas Tecnicas** de produtos da empresa **Mave Comercio de Acessorios Ltda** (cintas de amarracao, catracas, cintas de elevacao, etc.).

E uma aplicacao **100% frontend** (sem backend), hospedada no **Firebase Hosting**. Toda a logica roda no navegador do usuario.

---

## Stack Tecnologica

| Camada       | Tecnologia                                  |
|--------------|---------------------------------------------|
| Frontend     | HTML5 + CSS3 + JavaScript puro (Vanilla JS) |
| Hospedagem   | Google Firebase Hosting                     |
| Gerar PDF    | html2canvas (v1.4.1) + jsPDF (v2.5.2)      |
| Banco dados  | Arquivo JSON estatico (`produtos.json`)     |

**Nao existe backend, banco de dados relacional, nem autenticacao.**

---

## Estrutura de Arquivos

```
sistema-certificados/
├── .firebaserc              # ID do projeto Firebase
├── firebase.json            # Config do Firebase Hosting
├── package.json             # Dependencias Node (sharp, xlsx)
└── public/                  # Pasta servida pelo Firebase
    ├── index.html           # APLICACAO INTEIRA (HTML + CSS + JS)
    ├── produtos.json        # Base de dados dos produtos (~855KB)
    └── imagem/
        ├── logo.png         # Logo da Mave (usado no certificado)
        ├── assinatura.png   # Assinatura digital (usada no certificado)
        ├── assinatura.jpeg  # Assinatura backup
        ├── cinta-amarracao.png  # Ilustracao cinta
        ├── catraca.png          # Ilustracao catraca
        └── elevacao.png         # Diagrama cinta de elevacao
```

> **IMPORTANTE:** Toda a aplicacao (HTML, CSS e JavaScript) esta em um unico arquivo: `public/index.html` (~1650 linhas).

---

## Funcionalidades Principais

### 1. Busca e Selecao de Produtos
- Campo de busca com **autocomplete/dropdown**
- Filtra por codigo OU descricao do produto
- Carrega dados do `produtos.json` ao iniciar

### 2. Formulario de Dados (3 etapas com barra de progresso)

| Etapa     | Campos                                                      |
|-----------|-------------------------------------------------------------|
| Produto   | Busca e selecao do produto                                  |
| Cliente   | CNPJ (formatacao automatica), Razao Social, Nota Fiscal     |
| Ensaio    | Codigo de Rastreabilidade, Data da Realizacao                |

> Todos os campos exceto o produto sao **opcionais**. Campos nao preenchidos ficam em branco no documento.

### 3. Geracao de Documentos (2 tipos)

**Certificado de Conformidade:**
- Dados do produto (nome, materia-prima, dimensional, capacidade, cor)
- Dados do cliente (CNPJ, razao social)
- Dados do ensaio (numero, data)
- Nota fiscal
- Logo e assinatura da empresa
- Tabela de capacidades (para produtos de elevacao)

**Ficha Tecnica:**
- Especificacoes do produto
- Ilustracoes tecnicas (cinta, catraca ou elevacao)
- Propriedades fisicas e dimensionais
- Tabela de capacidades por aplicacao (para elevacao)

### 4. Exportacao para PDF
- Converte o documento HTML em imagem via `html2canvas`
- Gera PDF via `jsPDF`
- Download automatico no navegador

---

## Logica de Negocios (funcao `processarProdutos`)

Ao carregar o `produtos.json`, cada produto e processado assim:

### Classificacao por Tipo
- Codigo comeca com **"MSL"** → Produto de **Elevacao**
- Outros codigos → Produto de **Amarracao**

### Classificacao por Categoria
Com base no nome/descricao:
- `BARRIGUEIRA` → Barrigueira
- `CONJUNTO` + `CATRACA` → Conjunto Cinta e Catraca
- `CONJUNTO` + `ESTICADOR` → Conjunto Esticador
- `CONJUNTO` → Conjunto de Amarracao
- `CATRACA` → Catraca de Amarracao
- `MSL*` → Cinta de Elevacao
- Padrao → Cinta de Amarracao

### Calculo de Capacidades
- **Carga**: Extraida do nome do produto (ex: "800KG", "2TON")
- **Fator de Seguranca**: 
  - Amarracao: **2:1** (ruptura = carga x 2)
  - Elevacao: **7:1** (ruptura = carga x 4, com tabela especifica)
- **Materia-prima**: Determinada automaticamente pelos componentes
  - Tem fio/poliester + metal → "Poliester de alta tenacidade e acos laminados zincados"
  - So poliester → "100% Poliester de alta tenacidade"

### Tabela de Capacidades de Elevacao (`CAPACIDADE_ELEVACAO`)
Tabela fixa no codigo para cargas de 1000 a 10000 KG:

| Carga (kg) | Largura (mm) | Forca (kg) | Cesto | Cesto 45 | Cesto 60 |
|-------------|--------------|------------|-------|----------|----------|
| 1000        | 30           | 800        | 2000  | 1400     | 1000     |
| 2000        | 60           | 1600       | 4000  | 2800     | 2000     |
| 3000        | 90           | 2400       | 6000  | 4200     | 3000     |
| ...         | ...          | ...        | ...   | ...      | ...      |
| 10000       | 300          | 8000       | 20000 | 14000    | 10000    |

---

## Formato do `produtos.json`

Cada produto no JSON tem esta estrutura:

```json
{
  "codigo": "MCJ001",
  "descricao": "CONJUNTO CINTA E CATRACA 800KG X 50MM X 10M GANCHO J",
  "cor": "PRETO",
  "dimensional": "50MM",
  "componentes": [
    { "codigo": "COMP001", "nome": "FIO POLIESTER PRETO" },
    { "codigo": "COMP002", "nome": "CATRACA 50MM" }
  ]
}
```

> Se precisar adicionar/alterar produtos, edite diretamente o `produtos.json`.

---

## Como Rodar Localmente

### Pre-requisitos
- Node.js instalado
- Firebase CLI (`npm install -g firebase-tools`)

### Opcao 1: Servidor local simples
```bash
cd sistema-certificados
npx http-server public -p 8080
# Acessar http://localhost:8080
```

### Opcao 2: Firebase emulador
```bash
cd sistema-certificados
firebase serve
# Acessar http://localhost:5000
```

---

## Como Fazer Deploy (Producao)

```bash
# Login no Firebase (se necessario)
firebase login

# Deploy para producao
firebase deploy

# Ou so hosting
firebase deploy --only hosting
```

O projeto Firebase e: **`sistema-certificados-77f22`** (configurado em `.firebaserc`).

---

## Fluxo de Uso da Aplicacao

```
1. Usuario abre o sistema
2. Sistema carrega produtos.json
3. Usuario busca e seleciona um produto
4. (Opcional) Preenche dados do cliente (CNPJ, Razao Social, NF)
5. (Opcional) Preenche dados do ensaio (Codigo, Data)
6. Clica em "Visualizar Certificado"
7. Escolhe entre "Certificado" ou "Ficha Tecnica"
8. Visualiza o documento na tela
9. Clica em "Gerar PDF" para baixar
```

---

## O Que Voce Precisa Saber Para Mexer Neste Projeto

### Onde fica o codigo?
**Tudo esta em `public/index.html`**. Nao existem arquivos JS ou CSS separados.

### Como alterar o layout do certificado?
Procure as funcoes:
- `generateCertificado()` - gera o HTML do Certificado de Conformidade
- `generateFichaTecnica()` - gera o HTML da Ficha Tecnica (procure por esta funcao no index.html)

### Como adicionar novos produtos?
Edite o arquivo `public/produtos.json` seguindo o formato descrito acima.

### Como alterar imagens (logo, assinatura)?
Substitua os arquivos na pasta `public/imagem/`. Mantenha os mesmos nomes de arquivo.

### Como alterar a tabela de capacidades de elevacao?
Edite o objeto `CAPACIDADE_ELEVACAO` no JavaScript dentro do `index.html` (aproximadamente linha 866).

### Como funciona a geracao de PDF?
1. O documento e renderizado como HTML na tela
2. `html2canvas` captura o HTML como imagem (canvas)
3. `jsPDF` converte a imagem em PDF
4. O PDF e baixado automaticamente

### Bibliotecas externas (carregadas via CDN)
- **html2canvas** v1.4.1 - Captura HTML para canvas
- **jsPDF** v2.5.2 - Gera arquivos PDF

### Dependencias do package.json
- `sharp` e `xlsx` estao listados mas **nao sao usados no frontend**. Podem ter sido usados em scripts auxiliares de processamento.

---

## Pontos de Atencao

1. **Arquivo unico**: Todo o codigo esta em `index.html`. Se o projeto crescer, considere separar em arquivos JS/CSS.
2. **Sem versionamento de dados**: Os produtos estao em JSON estatico. Alteracoes exigem re-deploy.
3. **Sem autenticacao**: Qualquer pessoa com a URL pode gerar certificados.
4. **Tabela de capacidades hardcoded**: Os valores de `CAPACIDADE_ELEVACAO` estao fixos no codigo.
5. **Imagens no certificado**: O logo e a assinatura sao carregados da pasta `imagem/`. Se os caminhos mudarem, o certificado quebra.
6. **CNPJ**: Tem formatacao automatica mas **nao valida** se o CNPJ e real (nao consulta Receita Federal).

---

## Contato / Empresa

**Mave Comercio de Acessorios Ltda**  
Sistema de Certificados v1.0 - 2026
