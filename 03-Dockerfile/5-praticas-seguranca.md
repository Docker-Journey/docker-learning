# 05 - Criando Imagens Seguras do Docker e Práticas Recomendadas

## 📘 Teoria: Segurança em Containers

Containers oferecem uma camada importante de isolamento em relação ao sistema operacional hospedeiro (*host*), mas **containers não são 100% seguros por padrão**. 

A principal ameaça de segurança em ecossistemas de containers é a chamada **Fuga do Container (*Container Escape*)**, onde um invasor ganha controle da aplicação dentro do container e consegue explorar vulnerabilidades para invadir o sistema operacional host.

Para mitigar esses riscos, aplicamos quatro pilares fundamentais de segurança:

---

### 1. Imagens de Fontes Confiáveis (*Trusted Content*)

Evite baixar imagens arbitrárias de repositórios desconhecidos no Docker Hub. Dê preferência aos três filtros de conteúdo confiável do Docker:
* **Docker Official Images:** Imagens mantidas por equipes dedicadas do Docker (ex: `python`, `node`, `ubuntu`).
* **Verified Publisher:** Imagens mantidas diretamente pelas empresas proprietárias dos softwares (ex: `redis`, `mongodb`).
* **Docker Sponsored Open Source:** Imagens de projetos open-source patrocinados pelo Docker.

---

### 2. Mantenha os Pacotes Atualizados

Mesmo imagens oficiais podem conter bibliotecas e pacotes desatualizados no momento do build.

Em imagens baseadas em Debian/Ubuntu, a boa prática é atualizar a lista de repositórios e instalar correções de segurança na construção da imagem:

```dockerfile
FROM ubuntu:22.04

# Atualiza e instala correções de segurança em uma única camada
RUN apt-get update && apt-get upgrade -y && \
    rm -rf /var/lib/apt/lists/*

```

### 3. Mantenha as Imagens Mínimas (Redução de Superfície de Ataque)

"Não existe software mais seguro do que aquele que não foi instalado."

Instale apenas o essencial: Não instale editores de texto (vim, nano),
ferramentas de rede (curl, ping, net-tools) ou bibliotecas desnecessárias
na imagem de produção.

Prefira imagens minimalistas: Utilize variantes enxutas da imagem base, 
como -slim ou -alpine (ex: python:3.10-slim ou node:18-alpine).

### 4. Princípio do Menor Privilégio: Nunca Roda como Root

Instalações e configurações do sistema precisam de privilégios root,
mas a execução do aplicativo não.

Padrão de Ordem Segura no Dockerfile:
Comece como root (padrão) para atualizar
o sistema e instalar dependências.

Crie um usuário comum sem privilégios administrativos.

Alterne para esse usuário com a instrução USER antes do comando de inicialização (CMD).
```dockerfile
FROM python:3.10-slim

WORKDIR /app

# 1. Passo executado como ROOT (necessário para baixar pacotes de sistema)
RUN apt-get update && apt-get install -y --no-install-recommends gcc && \
    rm -rf /var/lib/apt/lists/*

# 2. Criação do usuário não-root
RUN adduser --disabled-password --gecos "" appuser

COPY --chown=appuser:appuser requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY --chown=appuser:appuser . .

# 3. Mudança para o usuário não-root ANTES da execução
USER appuser

# 4. A aplicação rodará com permissões restritas
CMD ["python", "main.py"]

```
Exercício 1: Diagnóstico de Erro de Permissão
Analise o seguinte Dockerfile:

```dockerfile
FROM ubuntu
RUN useradd -m repl
USER repl
CMD apt-get install -y python3
```

Pergunta: O que acontecerá ao executar um container a partir desta imagem com docker run?

1. [ ] A. O Python 3 será instalado com sucesso.

2. [x] B. Ocorrerá um erro de Permission Denied (permissão negada), pois o usuário repl não é root e não pode instalar pacotes via apt-get.

3. [ ] C. O container vai rodar como root ignorando a linha USER repl.

Exercício 2: Atualizando Pacotes no Ubuntu de Forma Interativa

Para praticar o processo de atualização de pacotes dentro
de um container Ubuntu:

Inicie um container interativo com a imagem oficial do Ubuntu:

```bash
docker run -it ubuntu bash
```

Verifique se há atualizações disponíveis no gerenciador apt:

```
apt-get update
```
Execute a atualização geral dos pacotes:

```
apt-get upgrade
```
Observe o resumo impresso no terminal com os pacotes categorizados
em: upgraded, newly installed, to remove, e not upgraded.

## Exercícios Extras para Prática 


Exercício 3: Corrigindo a Ordem das Instruções

Reescreva o Dockerfile do Exercício 1 para que a instalação do python3
funcione corretamente sem que o container final rode como root.

```dockerfile
# Resposta Correta:
FROM ubuntu

# 1. Instalação efetuada como root durante o build
RUN apt-get update && apt-get install -y python3 && \
    rm -rf /var/lib/apt/lists/*

# 2. Criação do usuário comum
RUN useradd -m repl

# 3. Alterna para o usuário não-root
USER repl

# 4. Execução da aplicação sem privilégios de administrador
CMD ["python3", "-c", "print('Ambiente seguro iniciado sem privilégios root!')"]
```

Exercício 4: Checklist de Auditoria de Segurança no Dockerfile
Avalie o Dockerfile abaixo e liste 3 falhas de segurança encontradas:

```dockerfile
FROM ubuntu:latest
ENV DB_PASSWORD="senha_super_secreta_123"
COPY . /
RUN apt-get update
CMD ["python3", "/app.py"]
```

Gabarito das Falhas:

- Segredo exposto no ENV: A senha de banco de dados (`DB_PASSWORD`)
está exposta em texto limpo na imagem e pode ser lida com docker history.

- Execução como Root: Não há instrução `USER`, logo o container rodará
o script `app.py` com privilégios totais de root.

Uso da tag latest: Utilizar ubuntu:latest impede a reprodutibilidade
e garante que builds futuros possam quebrar sem aviso prévio
ou trazer pacotes incompatíveis.

## Respostas e Explicações dos Exercícios
1. Quiz: Práticas recomendadas de segurança

Alternativas Correta(s):

1. [x] Usar um contêiner é uma boa maneira de executar um executável
ou abrir um arquivo de uma fonte não confiável, pois você diminui
muito a chance de um agente mal-intencionado acessar o seu computador.

2. (Verdadeiro: O isolamento diminui drasticamente o risco,
embora o isolamento de containers não seja 100% infalível
contra exploits avançados).

3. [x] Não há aplicativo mais seguro do que aquele que não foi instalado.

4. (Verdadeiro: Conceito de superfície de ataque mínima: quanto menos pacotes na imagem, menos vulnerabilidades em potencial).

5. [x] Ao criarmos uma imagem nós mesmos, podemos aumentar a segurança alterando o usuário do Linux para algo diferente do usuário raiz.

6. (Verdadeiro: Alterar o usuário impede que códigos maliciosos executem operações administrativas dentro ou fora do container).

Por que as outras estão incorretas?

*"Se eu não seguir todas as precauções... não seguir nenhuma"*: Incorreto. 
Segurança é aplicada em camadas.

*"Nada no contêiner pode afetar o ambiente do host"*: Incorreto.
Falhas gravíssimas conhecidas como Container Escape podem ocorrer se
o container for executado como root ou com privilégios elevados.

2. Exercício: Manter os pacotes atualizados (apt-get upgrade)
   
Resposta Correta:

`upgraded, newly installed, to remove, e not upgraded`.

Explicação:

Ao executar `apt-get upgrade` em uma distribuição baseada em Debian/Ubuntu,
o terminal exibe o resumo de alterações categorizado exatamente como:

X upgraded (atualizados)

Y newly installed (novos instalados por dependência)

Z to remove (removidos por obsolescência)

W not upgraded (mantidos na versão atual devido a dependências presas)

3. Exercício: Seja seguro, não use o root (`repl_try_install`)

Resultado do Comando:

Quando você executa `docker run repl_try_install`, o
container falha com o seguinte erro no terminal:
```
E: Could not open lock file /var/lib/dpkg/lock-frontend - open (13: Permission denied)
E: Unable to acquire the dpkg frontend lock (/var/lib/dpkg/lock-frontend), are you root?
```

Motivo:

O Dockerfile altera o usuário para repl (USER repl) antes da instrução `CMD apt-get install python3`.
Como o usuário repl é um usuário comum (`não-root`), ele não tem privilégios de sistema para instalar
pacotes via apt-get.

> Lição Prática: Instalações de pacotes de sistema (RUN apt-get...)
> devem ser feitas como root durante o build da imagem. A mudança para
> um usuário comum (USER repl) deve ser feita no final do Dockerfile,
>  logo antes do CMD de inicialização da aplicação.
