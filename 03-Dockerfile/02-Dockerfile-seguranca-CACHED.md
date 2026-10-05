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


**Gabarito dos Exercícios**

Resposta do Exercício 1

1. Não. O arquivo `main.py` foi alterado, portanto o conteúdo do diretório mudou,
invalidando o cache da etapa `COPY . ..`

2. Não. Devido ao efeito dominó, uma vez que o cache foi quebrado na etapa `COPY . .` (que veio antes), todas as etapas seguintes (RUN pip install...) são forçadas a executar do zero.


**Resposta do Exercício 2**
```
FROM python:3.10
WORKDIR /app

# 1. Copia apenas o manifesto de dependências
COPY requirements.txt ./

# 2. Instala os pacotes
RUN pip install -r requirements.txt

# 3. Copia o restante dos arquivos do projeto
COPY . .

CMD ["python", "main.py"]

```

## Ordem das Instruções

Essa conexão entre Ordem das Instruções e Reaproveitamento de Cache
é um dos pilares mais importantes para quem trabalha com Docker no dia a dia.

Quando alinhamos a ordem das instruções à frequência de mudança dos arquivos,
evitamos perder minutos preciosos a cada `build`.

### A Lógica por Trás da Ordem
O Docker constrói imagens de cima para baixo.
Se qualquer camada sofrer alteração, o cache quebra daquela linha em diante.

Portanto, a regra de ouro é ordenar o Dockerfile do Menos Frequente para o Mais Frequente:

```
[Imagem Base]        --> Muda raramente (ex: Ubuntu, Python, Node)
      ↓
[Instalação de Pacotes] --> Muda quando adicionamos dependências novas
      ↓
[Código-Fonte]         --> Muda constantemente a cada alteração/bugfix

```

**Resolução do Exercício**

Cenário: Temos um projeto em Python com os seguintes arquivos:

1. `requirements.txt` (pacotes e bibliotecas — muda com pouca frequência)

2. `pipeline.py` (código-fonte com as regras de negócio — muda o tempo todo)

Ordem Correta das Instruções (Do maior reuso para o menor)

```Dockerfile
# 1. Definição da Imagem Base
FROM python:3.10

# 2. Definição do Diretório de Trabalho
WORKDIR /app

# 3. Copia APENAS o arquivo de dependências (Muda com pouca frequência)
COPY requirements.txt .

# 4. Instala os pacotes do pipeline (Muda com pouca frequência)
RUN pip install -r requirements.txt

# 5. Copia o código do pipeline (Muda com MUITA frequência)
COPY pipeline.py .

# 6. Comando de inicialização
CMD ["python", "pipeline.py"]
```
**Explicação Detalhada do Processo**

Cenário A: Você edita uma função no `pipeline.py` e executa o build

1. `FROM python:3.10` --> CACHED (Não mudou)

2. `WORKDIR /app` ---> CACHED (Não mudou)

3. COPY requirements.txt . --> CACHED (O arquivo não sofreu alterações)

4. `RUN pip install -r requirements.txt` ---> CACHED (A camada anterior não mudou, então os pacotes não são baixados novamente)

5. `COPY pipeline.py .` --> Recompilado em milissegundos (Apenas o arquivo `pipeline.py` novo é copiado)

> Resultado: O build é concluído em 1 ou 2 segundos.





