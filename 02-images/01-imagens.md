# 02 - Images (Imagens Docker)

> **Resumo:** Se o contêiner é a casa construída, a **Imagem** é a planta arquitetônica e a fundação. Ela é um pacote somente leitura (*read-only*) que contém tudo o que a sua aplicação precisa para rodar: sistema operacional simplificado, código, bibliotecas e variáveis de ambiente.

### Para entender mais sobre imagens no Docker leia: [`Container vs Imagens`](https://github.com/Docker-Journey/docker-learning/blob/main/01-container/02-container.md)
---

## Tipos de Imagens no Docker Hub

O **Docker Hub** é o registro público oficial. Lá você encontrará dois tipos principais de imagens:

1. **Softwares Prontos para Uso:**
   - Imagens prontas para rodar serviços sem que você precise configurar o ambiente do zero.
   - *Exemplos:* `postgres`, `nginx`, `redis`, `mongodb`.

2. **Imagens Base para Desenvolvimento:**
   - Imagens que fornecem a linguagem e suas ferramentas (`pip`, `npm`, etc.) para você usar como ponto de partida (instrução `FROM`) no seu `Dockerfile`.
   - *Exemplos:* `python:3.10`, `node:18-alpine`, `ubuntu`, `golang`.

---

## O que são Tags nas Imagens?

As imagens são versionadas por meio de **Tags** (no formato `imagem:tag`).

- `postgres:15` $\rightarrow$ Baixa a versão específica 15 do Postgres.
- `python:3.10-slim` $\rightarrow$ Baixa a versão 3.10 do Python em uma variante leve (*slim*).
- `ubuntu:latest` $\rightarrow$ Baixa a tag padrão `latest` (a versão estável mais recente).

> ⚠️ **Boa prática:** Evite usar a tag `:latest` em ambientes de produção. Sempre especifique uma versão fixa para garantir que seu código rode sempre no mesmo ambiente.

---

## Comandos Essenciais da CLI para Imagens

| Comando | O que faz na prática |
| :--- | :--- |
| `docker pull <imagem>:<tag>` | Baixa a imagem do Docker Hub sem executar nenhum contêiner. |
| `docker images` | Lista todas as imagens salvas no seu computador. |
| `docker rmi <imagem>` | Remova uma imagem local (necessário remover ou parar o contêiner antes). |
| `docker history <imagem>` | Mostra as camadas de construção da imagem. |

---

## Prática no Terminal

Execute os comandos no seu terminal VS Code para testar o gerenciamento de imagens:

```bash
# 1. Baixar a imagem base do Python na versão 3.10-slim
docker pull python:3.10-slim

# 2. Listar suas imagens locais
docker images

# 3. Testar a execução rápida dessa imagem abrindo o Python interativo
docker run -it python:3.10-slim python

# (Dentro do Python, digite exit() para sair)

# 4. Limpar a imagem baixada
docker rmi python:3.10-slim

# 5. Apenas os contêineres ativos agora
docker ps

# 6. Todos os contêineres (ativos + parados)
docker ps -a

# 6. Listar TODOS os contêineres para VER QUAL IMAGEM PERTENCE A QUAL CONTÊINER
# A coluna "IMAGE" mostra de qual imagem o contêiner foi criado
docker ps -a --format "table {{.ID}}\t{{.Names}}\t{{.Image}}\t{{.Status}}"

# 7. Remover um CONTÊINER ESPECÍFICO pelo ID ou NOME
docker rm <CONTAINER_ID_OU_NOME>

# 8. DICA: Remover TODOS os contêineres parados gerados por uma imagem específica (ex: python:3.10-slim)
docker rm $(docker ps -a -q --filter ancestor=python:3.10-slim)

# 9. Remover a IMAGEM local
# (Nota: o Docker só deixa remover a imagem se os contêineres dela já foram removidos)
docker rmi python:3.10-slim
