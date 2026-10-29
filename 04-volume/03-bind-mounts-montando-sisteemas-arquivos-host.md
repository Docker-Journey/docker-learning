# 3. Montando o Sistema de Arquivos do Host (Bind Mounts)

Os dados dentro de um container são temporários (efêmeros). Se o container for destruído, todas as alterações no sistema de arquivos interno são perdidas.

Os **Bind Mounts** permitem espelhar um arquivo ou diretório da máquina host para dentro do container.

**Principais Benefícios:**

- Desenvolvimento Dinâmico: Alterações no código-fonte feitas no editor (VS Code, por exemplo) na máquina host refletem
imediatamente no container sem necessidade de reconstruir a imagem (docker build).

- Compartilhamento Bidirecional: Arquivos gerados pelo container são salvos diretamente no sistema de arquivos do host.

- Sintaxe do Bind Mount (`-v` ou `--mount`):

Usando a flag `-v` (formato curto):

```Bash
docker run -d -v $(pwd)/meu_codigo:/app/codigo python:3.10
```

Usando a sintaxe `--mount` (formato explícito e recomendado):

```Bash
docker run -d --mount type=bind,source="$(pwd)/meu_codigo",target=/app/codigo python:3.10
```

--- 

### 1. Entendendo o Sistema de Arquivos dos Containers (Isolamento)

Cada container iniciado a partir de uma imagem ganha seu próprio **sistema de arquivos isolado e temporário (efêmero)**.

#### Exemplo Prático de Isolamento:

Imagine uma receita de bolo (Imagem Docker). Você usa a receita para fazer dois bolos separados:

* **Container 1 (Bolo 1):** Você coloca granulado de chocolate por cima.

* **Container 2 (Bolo 2):** Continua normal, sem granulado.

O granulado no Bolo 1 **não aparece** no Bolo 2, nem muda a receita original no papel. 

Se você jogar o Bolo 1 no lixo (deletar o container com `docker rm`), o granulado **some para sempre**.

```text
[ Imagem Docker ] (Receita Original - Leitura)
       │
       ├───► [ Container 1 ] ──► Altera /home/NewFile (Existe apenas AQUI)
       │
       └───► [ Container 2 ] ──► Diretorio /home continua limpo

```
### 2. O que é Bind-Mount?
O Bind Mount é uma ponte/espelhamento direto entre uma pasta do seu computador real (Host) e uma pasta de dentro do container.

Em vez de guardar arquivos no disco temporário do container, o container passa a ler e escrever direto no disco da sua máquina hospedeira.

Diagrama de um Bind Mount:
```text

┌─────────────────────────────────────────────────────────┐
│                      MÁQUINA HOST                       │
│  Sua pasta real: /home/usuario/meu-site                 │
│  └── index.html                                         │
└───────────────────────────┬─────────────────────────────┘
                            │
              PONTE / ESPELHAMENTO (Bind Mount)
                            │
┌───────────────────────────▼─────────────────────────────┐
│                      CONTAINER                          │
│  Pasta interna: /usr/share/nginx/html                   │
│  └── index.html  <── (Lê em tempo real do Host)         │
└─────────────────────────────────────────────────────────┘

```
**Vantagens do Bind Mount:**
- Persistência: Se você apagar o container, os arquivos continuam intactos no seu computador.

- Desenvolvimento Live: Ao alterar o código na sua IDE (VS Code) no Host, o container atualiza na hora sem precisar de um novo docker build.

- Atualização do Container: Você pode atualizar a versão da imagem do seu aplicativo sem perder os dados armazenados.

## 3. Como Identificar: Host vs. Container?
Uma dúvida comum ao olhar os comandos do Docker é diferenciar o que pertence ao Host e o que pertence ao Container.

A regra fundamental da sintaxe é:

```
ORIGEM (Sua Máquina / Host) : DESTINO (Dentro do Container)
```

EXEMPLO:
```
-v /caminho/no/host:/caminho/no/container
```

**Regras Visuais para Não Errar:**

- O lado ESQUERDO dos dois pontos (:) é sempre a sua máquina (Host).

- O lado DIREITO dos dois pontos (:) é sempre a pasta interna do Container.

<img width="1536" height="1024" alt="Image" src="https://github.com/user-attachments/assets/8b5ce7f2-ec8f-4fe2-9347-68eeac2eae41" />

```
docker run -d -v /pgdata:/opt/data -v /pg.conf:/etc/pg.conf postgres
```
* `/pgdata` (Esquerda = Host): Pasta no seu computador onde os dados das tabelas ficam salvos.

* `/opt/data` (Direita = Container): Pasta onde o PostgreSQL lê e grava suas tabelas dentro do container.

* `/pg.conf` (Esquerda = Host): Arquivo de configuração personalizado criado por você.

* `/etc/pg.conf` (Direita = Container): Local onde o PostgreSQL busca sua configuração ao iniciar.

> ⚠️ Atenção: O Bind Mount oculta arquivos existentes na pasta do container!
> Se você montar uma pasta vazia do Host em cima de uma pasta do container que já tinha arquivos,
> a pasta dentro do container parecerá vazia enquanto o mount estiver ativo.

**Comparativo: -v vs. --mount**

Ambas as opções criam Bind Mounts, mas possuem sintaxes e usos diferentes.

**Sintaxe A: -v (ou --volume) — A Forma Curta**

É a forma mais tradicional e rápida de digitar no terminal.

```
docker run -d -v $(pwd)/html:/usr/share/nginx/html nginx
```
Vantagem: 

- Curta e prática para comandos no terminal do dia a dia.

- Comportamento Automático: Se a pasta da esquerda (Host) não existir no seu computador,
o Docker cria essa pasta automaticamente como diretório (o que pode gerar confusões se você errar a digitação do nome da pasta).


**Sintaxe B: --mount — A Forma Explícita / Moderna**

É a sintaxe mais verbosa, recomendada para scripts, pipelines e arquivos de produção.

```
docker run -d --mount type=bind,source="$(pwd)/html",target=/usr/share/nginx/html nginx
```
Vantagem: 
- Explícita, clara e flexível. Facilita a leitura e evita ambiguidades.

- Comportamento Seguro: Se a pasta definida na source (Host) não existir, o Docker retorna um erro em vez de criá-la silenciosamente.


## 3. Mapeamento de Arquivo de Configuração Somente Leitura

Crie o comando para executar uma aplicação em Python, mapeando apenas o arquivo de configuração config.json da pasta atual do computador
($(pwd)/config.json) para a pasta /app/config.json do container, garantindo permissão apenas de leitura (ro).

```
docker run -d -v $(pwd)/config.json:/app/config.json:ro python:3.10
```
A adição da tag :ro no final do -v garante que o container não consiga modificar o arquivo original no Hos


**Exercício 01:** Mapeando Diretório Atual com Bind Mount

Escreva o comando para rodar um container da imagem Nginx em segundo plano (-d),
mapeando a pasta atual da máquina host ($(pwd)) para o diretório de arquivos estáticos do Nginx no container (/usr/share/nginx/html).

```Bash

# Resposta:
docker run -d -p 8080:80 -v $(pwd):/usr/share/nginx/html nginx
```

