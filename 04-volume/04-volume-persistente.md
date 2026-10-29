# 1. Volume Persistente no Docker

## 1. O que é um Volume Persistente no Docker?

**Por que os Volumes existem?**

Por padrão, os contêineres Docker são efêmeros (descartáveis). Se você cria um contêiner,
insere registros em um banco de dados e depois apaga esse contêiner (docker rm), todos os dados salvos desaparecem para sempre.

Os Volumes Persistentes surgiram para resolver esse problema: eles funcionam como um "disco rígido externo virtuais" 
gerenciado diretamente pelo Docker. O volume existe independentemente do contêiner: você pode criar, 
parar ou destruir contêineres quantas vezes quiser, e os seus dados continuarão intactos dentro do volume.

## 2. Como Funciona um Volume

Imagine o contêiner como um computador de uma lan house:

Tudo o que você salva na área de trabalho dele é apagado quando você desliga a máquina.

O Docker Volume é como um Pendrive: você conecta o pendrive no computador da lan house,
salva seus arquivos nele e, quando o computador é desligado, você desconecta o pendrive com todos os seus dados salvos com segurança.

Mais tarde, você pode conectar esse mesmo pendrive em um outro computador totalmente novo (outro contêiner) e
todos os seus arquivos estarão lá exatamente como você deixou.

```
┌──────────────────────────────┐          ┌──────────────────────────────┐
│         CONTAINER A          │          │         CONTAINER B          │
│   (Pode ser destruído)       │          │   (Pode ser um novo banco)   │
└──────────────┬───────────────┘          └──────────────┬───────────────┘
               │                                         │
               │        ┌───────────────────────┐        │
               └───────►│    DOCKER VOLUME      │◄───────┘
                        │  (sqldata / persistent)│
                        │                       │
                        │  Dados do Banco de    │
                        │  Dados / Uploads      │
                        └───────────────────────┘
```
## 3. Onde os Volumes ficam armazenados?

Diferente do Bind Mount (em que você escolhe manualmente uma pasta no seu computador, como C:\Projetos), 
o Docker salva os volumes em uma área privada e isolada do próprio sistema do Docker:

- No Linux (Nativo): Fica em /var/lib/docker/volumes/

- No Windows / macOS: Fica armazenado dentro da máquina virtual leve gerenciada pelo Docker Desktop.

Você raramente precisa acessar essa pasta diretamente pelo seu sistema operacional; o gerenciamento é feito sempre pelos comandos da linha de comando (docker volume).


## 4. Gerenciando Volumes na prática (Linha de Comando)

Para criar e gerenciar seus volumes, o Docker disponibiliza a família de comandos docker volume:

**A. Criar um Volume (docker volume create)**

Cria um novo espaço de armazenamento isolado no Docker.
```
docker volume create sqldata
```
**B. Listar Volumes Existing (docker volume ls)**

Exibe a lista com todos os volumes criados no seu ambiente Docker.

```Bash
docker volume ls
```
Saída esperada:
```
DRIVER    VOLUME NAME
local     sqldata
```

**C. Inspecionar Detalhes do Volume (docker volume inspect)**

Retorna um JSON detalhado contendo metadados do volume: data de criação, driver utilizado e o caminho exato onde ele está montado (Mountpoint).

```Bash
docker volume inspect sqldata
```
D. Remover um Volume (docker volume rm)
Exclui definitivamente um volume e todos os dados armazenados nele. Atenção: O volume só pode ser removido se não estiver conectado a nenhum contêiner ativo ou parado.

```Bash
docker volume rm sqldata
```

**E. Limpeza Geral de Volumes Órfãos (docker volume prune)**

Remove de uma só vez todos os volumes que não estão associados a nenhum contêiner existente no sistema.
```
docker volume prune
```

## 5. Como Anexar um Volume a um Contêiner

Para conectar o volume a um contêiner durante a inicialização, usamos a flag -v no comando docker run.

Sintaxe do Comando:

<img width="879" height="129" alt="Image" src="https://github.com/user-attachments/assets/785ca024-3f12-41b9-92e1-ec53b2c35669" />

---

Exemplo Prático (Banco de Dados PostgreSQL):

```
# 1. Criamos o volume gerenciado pelo Docker
docker volume create dados_postgres

# 2. Subimos o container do PostgreSQL conectando o volume na pasta interna do banco
docker run -d \
  --name meu_banco \
  -v dados_postgres:/var/lib/postgresql/data \
  postgres:15
```

**O que aconteceu aqui?**

1. O Docker conectou o volume `dados_postgres` ao diretório `/var/lib/postgresql/data` dentro do contêiner.

2. Todas as tabelas, índices e dados criados pelo PostgreSQL serão salvos diretamente no volume `dados_postgres`.

3. Se você rodar `docker rm -f meu_banco`, o contêiner morre, mas o volume dados_postgres continua existindo intacto no seu computador.


