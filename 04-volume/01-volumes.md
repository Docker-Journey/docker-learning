<div align="center">

# 04 · Volumes

<img src="https://img.shields.io/badge/Tema-01%20Containers-f472b6?style=for-the-badge&logo=docker&logoColor=white"/>
<img src="https://img.shields.io/badge/Status-Em%20andamento-c084fc?style=for-the-badge"/>
<img src="https://img.shields.io/badge/Nível-Iniciante-a855f7?style=for-the-badge"/>

*Aprender • Construir • Quebrar • Consertar • Compartilhar*

</div>

## 1. O que é Volume

Um volume no Docker é um mecanismo criado para **salvar e persistir dados fora** do ciclo de vida dos containers.

**Um container é descartável**
Por padrão, um container é descartável: se você o remove (`docker rm`), todos os arquivos gerados dentro dele **desaparecem para sempre**.

**Volume é um disco rígido**

O volume funciona como um "disco rígido externo" criado e gerenciado pelo Docker.
Você conecta esse disco ao container e, mesmo que o container seja destruído, atualizado ou recriado, os dados salvos nesse disco permanecem intactos.

**O que os volumes guardam?**

Eles guardam qualquer tipo de arquivo ou dado que sua aplicação precise preservar entre reinicializações ou atualizações, como:

- Bancos de dados: Tabelas e arquivos do PostgreSQL, MySQL, MongoDB, Redis.

- Uploads de usuários: Imagens, PDFs e arquivos enviados para a aplicação.

- Logs da aplicação: Histórico de execuções para análise de erros.

- Arquivos de configuração e certificados: Chaves SSL, arquivos .env ou configurações de servidores Web.

### Quais tipos de armazenamento de dados existem no Docker?
O Docker oferece 3 formas principais de lidar com dados:

**1. Docker Volumes (Volumes Gerenciados) - O mais recomendado**
- O que é: É uma área de armazenamento criada e gerenciada 100% pela CLI do Docker (`docker volume create`).

- Onde fica armazenado: Fica em uma pasta restrita do próprio sistema operacional do Docker, geralmente em `/var/lib/docker/volumes/` (no Linux).

- Para que serve: Ideal para bancos de dados, uploads em produção e qualquer dado que precise de alto desempenho de leitura/escrita e isolamento do usuário da máquina host.

**2. Bind Mounts (Montagem de Pastas do Host)**
- O que é: É o espelhamento direto de uma pasta ou arquivo específico do seu `computador/servidor (Host) para dentro do container`.

- Onde fica armazenado: Em qualquer caminho do seu sistema de arquivos que você escolher (ex: `~/meu-projeto/codigo` ou C:\Projetos\app).

- Para que serve: Excelente para ambiente de desenvolvimento local. Quando você altera uma linha de código no seu VS Code na máquina host, a alteração reflete
instantaneamente dentro do container em execução sem precisar rodar um novo docker build.

**3. tmpfs Mounts (Montagem em Memória RAM)**
- O que é: É um armazenamento temporário criado diretamente na memória RAM do host (e não no disco rígido/SSD). Gerenciado pelo Kernel do linux.

- Onde fica armazenado: Na memória RAM do servidor.

- Para que serve: Dados ultrassensíveis (como senhas temporárias/chaves de criptografia) ou dados temporários de altíssima
velocidade que devem sumir assim que o container parar.

### Exempo prático

```Bash
# 1. Usando um DOCKER VOLUME (Passa o NOME do volume)
docker run -d -v meu_postgres_data:/var/lib/postgresql/data postgres

# 2. Usando um BIND MOUNT (Passa o CAMINHO da sua máquina)
docker run -d -v $(pwd)/meu-codigo:/app node

```

---

<img width="1312" height="1199" alt="Image" src="https://github.com/user-attachments/assets/ac7b3f22-1859-4b9e-a3a0-5f4e353a2c2d" />

---

O volume fica sob gerenciamento do Docker e pode ser montado em vários contêineres ao mesmo tempo.
Assim, os dados não dependem do ciclo de vida de um único contêiner.

No diagrama, `shared_data_volume` é o volume. Os dois contêineres montam esse mesmo volume em `/data`.

**Como interpretar**

- Host: é a máquina ou servidor onde o Docker está executando.
- Volume: é a área de armazenamento persistente gerenciada pelo Docker.
- Containers: acessam os dados do volume pelo caminho `/data`.
- Persistência: remover um contêiner não remove automaticamente o volume nem os dados armazenados nele.

---

## O que vamos estudar?
Vamos focar nesta primeira parte: Comandos Úteis, `--rm`, Bind Mounts e Volumes Persistentes.

1. Você está em[ 01 - Volumes](https://github.com/Docker-Journey/docker-learning/blob/main/04-volume/01-volumes.md)

2. ➡️ [02 - Comandos intermediários do Docker](https://github.com/Docker-Journey/docker-learning/blob/main/04-volume/02-comandos-intermediarios-limpeza-automatica.md)

3. ➡️ [03- Montando o sistema de arquivos do host](https://github.com/Docker-Journey/docker-learning/blob/main/04-volume/03-bind-mounts-montando-sisteemas-arquivos-host.md)

4. ➡️ [04- Volumes persistentes](https://github.com/Docker-Journey/docker-learning/blob/main/04-volume/04-volume-persistente.md)


---

Boa sorte nos estudos!
