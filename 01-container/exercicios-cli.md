# 🐳 Laboratório 01 - Gerenciamento de Containers com Docker

Neste laboratório, vamos praticar os comandos básicos para:

* executar um container;
* executar um container em segundo plano;
* entrar em um container;
* baixar imagens do Docker Hub;
* listar containers e imagens;
* parar e remover containers;
* remover imagens.

> 💡 **Importante:** execute os comandos no seu próprio terminal.

---

# 📁 Passo 0 - Preparar a pasta do laboratório

Antes de começar, vamos criar uma pasta para organizar nossos exercícios.

No terminal:

```bash
mkdir -p docker-labs/01-gerenciamento-containers
cd docker-labs/01-gerenciamento-containers
```

Para confirmar onde você está:

```bash
pwd
```

---

# 🐳 Exercício 1 - Executar o primeiro container

## Testando a instalação do Docker

Vamos executar a imagem oficial `hello-world`.

No terminal:

```bash
docker run hello-world
```

### 🔎 O que vai acontecer?

Se a imagem ainda não estiver na sua máquina, o Docker fará o download dela do **Docker Hub**.

Você provavelmente verá uma mensagem semelhante a:

```text
Unable to find image 'hello-world:latest' locally
```

Depois o Docker fará o download da imagem e executará o container.

No final, aparecerá:

```text
Hello from Docker!
```

O container será encerrado automaticamente depois de mostrar a mensagem.

### 🧠 O que aprendemos?

O comando:

```bash
docker run hello-world
```

faz algumas coisas:

1. procura a imagem `hello-world` localmente;
2. se não encontrar, baixa a imagem;
3. cria um container;
4. executa o container;
5. quando o processo termina, o container é encerrado.

---

# 🌐 Exercício 2 - Executar um container em segundo plano

## Detached mode `-d`

Agora vamos executar um servidor web **Nginx** em segundo plano.

No terminal:

```bash
docker run -d --name meu-nginx nginx
```

### 🔎 O que significa?

```text
-d
```

Executa o container em **detached mode**, ou seja, em segundo plano.

```text
--name meu-nginx
```

Define o nome do container como `meu-nginx`.

```text
nginx
```

É a imagem que será utilizada.

### 🔎 O que vai acontecer?

O Docker retornará um hash longo, que representa o ID do container:

```text
a1b2c3d4e5f6...
```

E você voltará imediatamente para o terminal.

Para verificar se o container está rodando:

```bash
docker ps
```

Você deverá encontrar o `meu-nginx` na lista.

---

# 🐧 Exercício 3 - Entrar em um container Ubuntu

## Modo interativo `-it`

Agora vamos criar um container Ubuntu e abrir um terminal dentro dele.

Execute:

```bash
docker run -it --name meu-ubuntu ubuntu bash
```

### 🔎 O que significa?

```text
-it
```

Combina duas opções:

* `-i` → mantém a entrada interativa aberta;
* `-t` → cria um terminal para interação.

```text
--name meu-ubuntu
```

Define o nome do container.

```text
ubuntu
```

É a imagem utilizada.

```text
bash
```

É o comando que queremos executar dentro do container.

### 🐚 Você está dentro do container!

Seu prompt poderá mudar para algo parecido com:

```text
root@a1b2c3d4e5f6:/#
```

Agora você está **dentro do container Ubuntu**.

Experimente:

```bash
ls
```

Depois:

```bash
whoami
```

E:

```bash
cat /etc/os-release
```

Observe as informações apresentadas.

### 🚪 Para sair

Digite:

```bash
exit
```

Isso encerrará o processo `bash` e, consequentemente, o container.

> 💡 **Atenção:** sair do container não significa necessariamente apagar o container. Ele continuará existindo como um container parado.

---

# 🐘 Exercício 4 - Baixar o PostgreSQL 15

## `docker pull`

Agora vamos baixar especificamente a versão **15** da imagem do PostgreSQL.

Execute:

```bash
docker pull postgres:15
```

### 🔎 O que significa?

```text
postgres
```

É o nome da imagem.

```text
:15
```

É a **tag** que especifica a versão que queremos baixar.

Portanto:

```text
postgres:15
```

significa:

> "Quero a imagem do PostgreSQL com a tag 15."

### ⚠️ Importante

O comando:

```bash
docker pull postgres:15
```

**baixa a imagem, mas não executa um container.**

Para visualizar as imagens disponíveis localmente:

```bash
docker images
```

Você deverá encontrar algo semelhante a:

```text
REPOSITORY   TAG
postgres     15
```

---

# 🧹 Exercício 5 - Limpeza do ambiente

Agora vamos praticar como listar e remover containers e imagens.

Isso é importante para evitar o acúmulo de recursos que não estamos mais utilizando.

---

## 5.1 - Listar todos os containers

Para visualizar containers **rodando e parados**:

```bash
docker ps -a
```

Compare com:

```bash
docker ps
```

### Diferença

```bash
docker ps
```

Mostra apenas containers em execução.

```bash
docker ps -a
```

Mostra containers em execução **e também os parados**.

---

## 5.2 - Parar o Nginx

Nosso Nginx está rodando em segundo plano.

Vamos pará-lo:

```bash
docker stop meu-nginx
```

Agora confira:

```bash
docker ps
```

O `meu-nginx` não deverá mais aparecer entre os containers em execução.

Mas ele ainda existe.

Para confirmar:

```bash
docker ps -a
```

---

## 5.3 - Remover os containers

Agora podemos remover os containers que criamos.

```bash
docker rm meu-nginx meu-ubuntu
```

> 💡 O `hello-world` também criou um container, mas seu nome pode variar. Por isso, antes de removê-lo, consulte `docker ps -a` e identifique o nome ou ID correspondente.

Se quiser remover um container específico:

```bash
docker rm NOME_OU_ID
```

---

## 5.4 - Remover as imagens

Para liberar espaço, podemos remover as imagens que baixamos:

```bash
docker rmi hello-world nginx postgres:15 ubuntu
```

### ⚠️ Se aparecer uma mensagem de erro

Se o Docker informar que uma imagem está sendo utilizada por algum container, primeiro remova o container que está utilizando essa imagem.

Você pode consultar:

```bash
docker ps -a
```

E depois remover o container:

```bash
docker rm NOME_OU_ID
```

Depois tente novamente:

```bash
docker rmi NOME_DA_IMAGEM
```

---

# 🧠 Resumo dos comandos

| Comando          | Para que serve               |
| ---------------- | ---------------------------- |
| `docker run`     | Cria e executa um container  |
| `docker run -d`  | Executa em segundo plano     |
| `docker run -it` | Executa de forma interativa  |
| `docker ps`      | Lista containers em execução |
| `docker ps -a`   | Lista todos os containers    |
| `docker stop`    | Para um container            |
| `docker rm`      | Remove um container          |
| `docker pull`    | Baixa uma imagem             |
| `docker images`  | Lista imagens locais         |
| `docker rmi`     | Remove uma imagem            |

---

# 🎯 Desafio final

Sem consultar os exemplos acima, tente responder:

### 1. Como você criaria um container chamado `meu-ubuntu` usando Ubuntu e abriria o Bash?

```bash
# sua resposta aqui
```

### 2. Como você executaria um Nginx em segundo plano com o nome `web`?

```bash
# sua resposta aqui
```

### 3. Como você listaria todos os containers, inclusive os parados?

```bash
# sua resposta aqui
```

### 4. Como você baixaria especificamente a imagem `postgres` na versão `15`?

```bash
# sua resposta aqui
```

### 5. Qual é a diferença entre:

```bash
docker ps
```

e:

```bash
docker ps -a
```

---

## 🚀 O que você praticou neste laboratório?

Ao terminar este exercício, você já deve conseguir:

* ✅ executar uma imagem;
* ✅ criar um container;
* ✅ nomear um container;
* ✅ executar containers em segundo plano;
* ✅ entrar em um container;
* ✅ trabalhar com tags de imagens;
* ✅ listar containers;
* ✅ parar containers;
* ✅ remover containers;
* ✅ baixar imagens;
* ✅ remover imagens.

> 🐳 **Próximo passo:** entender melhor a diferença entre **imagem, container e Dockerfile** e começar a criar suas próprias imagens.
