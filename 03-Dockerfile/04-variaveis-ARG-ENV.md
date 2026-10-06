# 04 - Variáveis no Dockerfile (`ARG` vs `ENV`) e Segurança

## 📘 Teoria: Parametrizando Imagens Docker

Usar variáveis nos Dockerfiles torna o código mais limpo,
fácil de manter e flexível para reuso em diferentes ambientes
(Desenvolvimento, Homologação e Produção).

O Docker oferece duas instruções para trabalhar com variáveis: **`ARG`** e **`ENV`**.

---

### 1. Variáveis de Build: `ARG` (Build-Time)

A instrução `ARG` (*Build Arguments*) define variáveis que existem
**apenas durante o processo de construção da imagem** (`docker build`).
Assim que o build é finalizado, a variável deixa de existir e **não** fica
acessível dentro do container em execução.

#### Sintaxe no Dockerfile:
```dockerfile
ARG PYTHON_VERSION=3.10
FROM python:${PYTHON_VERSION}
```
Casos de Uso do ARG:

- Centralizar versões: Definir versões de softwares ou linguagens no topo do Dockerfile.

- Personalização no Build: Mudar o comportamento de compilação sem alterar o código do Dockerfile.

Sobrescrevendo ARG no Terminal:

Você pode alterar o valor padrão de um ARG durante a compilação usando a flag --build-arg:

```Bash
docker build --build-arg PYTHON_VERSION=3.11 -t minha-imagem .
```

## 2. Variáveis de Ambiente: ENV (Runtime)
A instrução ENV (Environment Variables) define variáveis
que persistem na imagem e continuam disponíveis dentro
do container durante sua execução (docker run).

Sintaxe no Dockerfile:
```dockerfile
Dockerfile
ENV PORT=8080
ENV APP_ENV=production
```

Casos de Uso do ENV:

- Configurar variáveis de execução da aplicação
(ex: porta do servidor, modo de execução production/development, timezone).

- Configurações de conexão (ex: DB_HOST, DB_PORT).

Sobrescrevendo ENV no Terminal:

Diferente do ARG, o ENV não pode ser alterado no docker build,
mas pode ser alterado na inicialização do container com a flag `--env` (ou -e):

```Bash
docker run -e PORT=3000 --name meu-app minha-imagem
```

## 3. Tabela Comparativa: ARG vs ENV

| **Característica** | **ARG** | **ENV** |
|---|---|---|
| **Momento de Acesso** | Apenas no **Build** (`docker build`) | Durante a **Execução** (`docker run`) |
| **Disponível no Container?** | ❌ Não | ✅ Sim |
| **Como Sobrescrever?** | `docker build --build-arg VAR=val .` | `docker run --env VAR=val imagem` |
| **Seguro para Senhas?** | ❌ **NÃO** | ❌ **NÃO** |

## 4. Risco Crítico de Segurança: Segredos em Variáveis ⚠️

NUNCA insira senhas, chaves de API, tokens ou certificados em instruções ARG ou ENV!

Qualquer pessoa que receber a imagem pode rodar 
o comando docker history minha-imagem e visualizar 
em texto puro todos os valores definidos via ARG e ENV.

Além disso, os valores passados via terminal podem 
ficar salvos no histórico de comandos do sistema (bash history).

Boas Práticas para Segredos: Utilize ferramentas adequadas
como Docker Secrets, volumes montados em tempo de execução, 
ou gerenciadores de segredos (AWS Secrets Manager, Vault, HashiCorp).

---

## Exercícios Práticos

Exercício 1: Compreensão Teórica de ARG e ENV

Marque as afirmações verdadeiras sobre ARG e ENV:

1. [ ] A. Variáveis ARG não ficam acessíveis no container, logo é seguro usá-las para armazenar senhas.

2. [x] B. Variáveis ENV ficam disponíveis dentro dos containers, sendo ideais para configurações de runtime.

3. [x] C. É possível substituir variáveis ARG durante a compilação via parâmetro --build-arg.

4. [x] D. Cada usuário que inicia um container pode definir valores diferentes para variáveis ENV via parâmetro --env.

Exercício 2: Substituindo ARG no Build

Cenário: Dado o seguinte Dockerfile:
```dockerfile
Dockerfile
FROM ubuntu
ARG WELCOME_TEXT=Hello!
RUN echo $WELCOME_TEXT
CMD echo $WELCOME_TEXT
```

Tarefa: Digite o comando no terminal para construir a imagem substituindo o texto padrão para "Welcome!".

Comando:
```Bash
docker build --build-arg WELCOME_TEXT="Welcome!" -t minha-imagem .
```

Exercício 3: Modificando o Comportamento com ENV no Runtime

Cenário: Dado o seguinte Dockerfile:

```Dockerfile
FROM ubuntu:22.04
ENV NAME=Tim
CMD echo "Hello, my name is $NAME"
```
Passo 1: Crie a imagem com o nome hello_image:

```Bash
docker build -t hello_image .
```

Passo 2: Inicie um container a partir da imagem hello_image, alterando a variável NAME para o seu nome (ex: Carlos):

```Bash
docker run --env NAME=Carlos hello_image
```

Saída esperada no terminal: Hello, my name is Carlos

## Exercícios Extras para Prática

Exercício 4: Multi-Stage/Versões Dinâmicas com ARG

Crie um Dockerfile para Node.js onde a versão da imagem base
alpine possa ser alterada na compilação, mantendo 18 como versão padrão.
```Dockerfile
# Resposta:
ARG NODE_VERSION=18
FROM node:${NODE_VERSION}-alpine

WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .

CMD ["npm", "start"]
```

Para testar a alteração no build: `docker build --build-arg NODE_VERSION=20 -t node-app-v20 . `

Exercício 5: Configurando Ambiente de Produção com ENV

Crie um Dockerfile para Python que defina a variável
de ambiente FLASK_ENV como production por padrão e a porta 5000.

```Dockerfile
# Resposta:
FROM python:3.10-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .

# Variáveis de ambiente de execução
ENV FLASK_ENV=production
ENV PORT=5000

EXPOSE 5000
CMD ["python", "app.py"]
```

Para rodar em modo desenvolvimento: `docker run -p 5000:5000 -e FLASK_ENV=development minha-app-flask`


---


### 1. "Qual o sentido do ARG se não vamos usar depois de criar a imagem?"

O ARG serve exclusivamente para a fase de construção (Build) da imagem. 
Ele funciona como uma "variável de compilação".

- Caso de uso 1: Atualização centralizada de versões no Dockerfile

Se você instala 5 pacotes que usam a mesma versão do Python ou Node, define ARG PYTHON_VERSION=3.10 no topo. Se precisar atualizar a versão no futuro, altera apenas uma linha em vez de buscar no arquivo inteiro.

- Caso de uso 2: Reaproveitar o mesmo Dockerfile para ambientes diferentes (Dev vs Prod)

Em Dev, você pode querer compilar o projeto com ferramentas de depuração ativadas (ARG BUILD_TYPE=debug). Em Produção, você usa o mesmo Dockerfile, mas altera o parâmetro para ARG BUILD_TYPE=release.

## 2. "Como funciona o build-arg no terminal?"

Imagine que você tem o seguinte Dockerfile:
```dockerfile
Dockerfile
FROM node:18-alpine
ARG NODE_ENV=production
RUN echo "Compilando para o ambiente: $NODE_ENV"
```
Se você apenas rodar `docker build -t minha-app .`, a variável `$NODE_ENV` terá o valor padrão "production".

Porém, sem alterar o arquivo, você pode sobrescrever o
valor diretamente pelo terminal na hora de criar a imagem usando 
a flag `--build-arg`:

```Bash
docker build --build-arg NODE_ENV=development -t minha-app-dev .
```

O Docker criará a imagem injetando "development" dentro do `$NODE_ENV` durante esse build.

## 3. Explicação Detalhada do Quiz (Verdadeiro ou Falso)
   
"As variáveis definidas em um Dockerfile usando a instrução ARG
não ficam acessíveis depois que a imagem é criada. Isso significa 
que é seguro usar ARG para armazenar segredos em um Dockerfile."

INCORRETA ❌: Embora o ARG não vire variável dentro do container rodando, 
qualquer pessoa com acesso à imagem pode rodar docker history minha-imagem
e ver em texto puro o valor passado no --build-arg (como senhas ou tokens).

"As variáveis definidas usando ENV podem ser usadas em contêineres a
partir da sua imagem, o que faz com que seja uma boa maneira de
definir a configuração usando um tempo de execução."

CORRETA ✅: O ENV persiste dentro da imagem e fica disponível como
variável de ambiente do sistema operacional do container enquanto ele estiver rodando.

"É possível substituir as variáveis definidas com ARG durante
a compilação, o que nos permite configurar as imagens no momento da compilação."

CORRETA ✅: Através da flag docker build --build-arg NOME_VAR=valor.

"Cada usuário que inicia um contêiner a partir da nossa imagem
pode selecionar um valor diferente para qualquer variável ENV
que definimos na nossa imagem."

CORRETA ✅: Quem executa o container pode sobrescrever o valor padrão
do ENV na inicialização usando docker run --env NOME_VAR=novo_valor.







