# 02 - Armazenamento em Cache no Docker (`CACHED`)

## 📘 Teoria: Como Funciona o Cache do Docker

Construir imagens Docker pode levar tempo, mas ao rodar o `docker build` pela segunda vez, o processo costuma ser bem mais rápido. Isso acontece por conta do **mecanismo de cache por camadas**.

---

### 1. O Conceito de Camadas (*Layers*)

O Docker constrói imagens executando as instruções do `Dockerfile` de forma sequencial. Cada instrução que altera o sistema de arquivos gera uma nova **camada de modificações**.

* **O que é uma camada:** É o registro exato das alterações feitas no sistema de arquivos durante aquela instrução (como baixar pacotes, criar pastas ou copiar arquivos).
* **Composição:** A imagem final é a união de todas essas camadas empilhadas em ordem, juntas com metadados (como o comando `CMD`).
* **Output do Build:** Durante o comando `docker build`, o terminal indica em qual etapa/camada o Docker está trabalhando (ex: `Step 3/8`).

---

### 2. O Mecanismo de Cache (`CACHED`)

Quando você reexecuta o `docker build`, o Docker tenta reaproveitar as camadas criadas em builds anteriores em vez de reexecutar as instruções do zero.

* **Identificação:** Se uma instrução não sofreu alterações, o Docker reutiliza a camada correspondente e exibe a palavra **`CACHED`** no terminal.
* **Regras de Invalidação do Cache:** O Docker só reutiliza o cache de uma etapa se **duas condições** forem atendidas:
  1. A instrução do `Dockerfile` é exatamente idêntica à do build anterior.
  2. **Todas as instruções anteriores** também usaram o cache e não sofreram alterações.
* **Efeito Dominó:** Se o cache for quebrado em uma determinada instrução, **todas as instruções subsequentes serão reconstruídas do zero**, mesmo que não tenham mudado.

---

### 3. O Comportamento "Cego" do Cache (Armadilha do `RUN`)

O Docker avalia apenas o **texto da instrução** presente no `Dockerfile`, e não o resultado retornado do ambiente externo.

* **Exemplo:** Com a instrução `RUN apt-get update && apt-get install -y python3`, o Docker verifica apenas se essa linha de texto mudou.
* **Consequência:** Se uma nova versão do Python 3 for lançada nos repositórios remotos, o `docker build` **não** baixará a atualização se a camada estiver em cache. O Docker assume que o resultado seria idêntico e reusa a camada guardada.

---

### 4. Boa Prática: Ordenação de Instruções

Para otimizar o tempo de build, as instruções do `Dockerfile` devem ser organizadas **da instrução que Menos Muda para a que Mais Muda**.

#### Estrutura Ineficiente ❌
```dockerfile
FROM node:18-alpine
WORKDIR /app

# Copia TODOS os arquivos (muda com frequência)
COPY . .

# Instala dependências (muda raramente)
RUN npm install
```
- Problema: Qualquer alteração no código-fonte invalida o cache no COPY . ..
Isso força o Docker a reexecutar o `RUN npm install` do zero em todo build,
baixando dependências novamente.

#### Estrutura eficiente ✅
```
FROM node:18-alpine
WORKDIR /app

# Copia apenas os manifestos de dependência (muda raramente)
COPY package*.json ./

# Instala dependências (ficará em cache se o package.json não mudar)
RUN npm install

# Copia o restante do código (muda frequentemente)
COPY . .
```
- Vantagem: O `npm install` só rodará novamente se o arquivo `package.json`
  for alterado. Se você mudar apenas o código-fonte da aplicação, o Docker
  reusará o cache até a etapa do `npm install` e executará apenas o `COPY . .`
  final em poucos milissegundos.

# Exercícios Práticos de Fixação

Exercício 1: Identificando a Invalidação de Cache
Considere o seguinte Dockerfile:
```
FROM python:3.10
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
CMD ["python", "main.py"]
```

Pergunta: Se você alterar apenas uma linha de código no arquivo `main.py` e rodar o docker build novamente:

1. A instrução `COPY . .` usará cache?

2. A instrução `RUN pip install -r requirements.txt` usará cache? Por quê?















