# 01 - Comandos intermediários e Limpeza automática


Quando testamos containers no terminal, é comum deixá-los parados acumulando espaço em disco.

## Remoção Automática (`--rm`):

Adicionar a flag `--rm` ao `docker run` instrui o Docker a **deletar** o container **automaticamente** assim que ele parar de rodar.

```bash
docker run --rm ubuntu echo "Executa e limpa sozinho"
```

- Inspecionar Detalhes (docker inspect):

Retorna um JSON completo com todas as configurações do container (IP, montagens, variáveis de ambiente, portas).

- Ver Logs de Containers Parados ou Rodando (docker logs):

Permite ver a saída do terminal de um container sem precisar estar conectado a ele.

- Limpeza Geral (docker system prune):

Remove containers parados, redes não utilizadas e imagens sem tag que estão ocupando espaço.

---

**Exercício 1:** Limpeza Automática de Container
Qual flag deve ser adicionada ao comando docker run para garantir que o container seja excluído automaticamente assim que sua execução terminar?

A) -d

B) --rm

C) --clean

D) -i

**Exercício 2:** Passando um arquivo para um container em execução
Imagine que você tem um arquivo chamado `dados.csv` na pasta atual da sua máquina e
deseja executá-lo dentro de um container de análise Python na pasta `/app/dados.csv`.

Escreva o comando usando a flag `-v` e a variável de ambiente $(pwd) para montar esse arquivo dentro do container interativo da imagem python:3.10.

```Bash
docker run -v $(pwd)/dados.csv:/app/dados.csv -it python:3.10
```
| Parte              | Significado                                                  |
| ------------------ | ------------------------------------------------------------ |
| `docker run`       | Cria e inicia um contêiner.                                  |
| `-v`               | Monta um arquivo ou diretório.                               |
| `$(pwd)/dados.csv` | Caminho completo do arquivo na máquina host.                 |
| `/app/dados.csv`   | Caminho onde o arquivo ficará acessível dentro do contêiner. |
| `-it`              | Permite interação com o terminal.                            |
| `python:3.10`      | Imagem Python na versão 3.10.                                |

**Exercício 3: Persistência de Banco de dados**
Você está configurando um container PostgreSQL. O que acontece com os dados gravados no banco se você não utilizar
um Volume ou Bind Mount e o container for excluído com docker rm?

Os dados gravados no PostgreSQL serão perdidos quando o contêiner for excluído, caso estejam armazenados apenas na camada gravável do próprio contêiner.

Sem Volume ou Bind Mount, os dados não ficam protegidos contra a remoção do contêiner.

Para preservar os dados, você pode usar um volume Docker:

O gerenciamento intermediário de containers exige ferramentas para monitorar o ciclo de vida dos containers, 
inspecionar alterações e compartilhar dados de maneira persistente entre a máquina hospedeira (*host*) e o
ambiente isolado do container.

```Bash
docker run -d \
  -v postgres_data:/var/lib/postgresql/data \
  postgres
```

---

### 1. Comandos Intermediários Úteis

* **`docker ps` / `docker container ls`:** Lista os containers em execução.
  * `docker ps -a`: Lista todos os containers no sistema (em execução e parados).
* **`docker diff <container_id>`:** Inspeciona as alterações feitas no sistema de arquivos de um container em relação à imagem original a partir da qual ele foi criado.
  * **`A` (Added):** Arquivo ou diretório adicionado.
  * **`C` (Changed):** Arquivo ou diretório alterado.
  * **`D` (Deleted):** Arquivo ou diretório excluído.
* **`docker network`:** Gerencia as redes internas e drivers de comunicação do Docker.


---

### 2. Limpeza Automática de Containers (`--rm`)

Por padrão, quando um container termina sua execução, ele permanece no sistema com o status `Exited` (parado),
ocupando espaço em disco e acumulando histórico desnecessário durante testes locais.

Para garantir que o container seja **removido automaticamente do sistema assim que seu processo principal finalizar**, utilizamos a flag `--rm`:

```bash
docker run -it --rm alpine:3.19.2
```
> ⚠️ Atenção: A remoção automática via `--rm` ocorre no comando docker run (que cria e executa um novo container),
> e não no comando docker start (que apenas reinicia um container já existente).

---

O que estudar a seguir:
➡️ 03 - Montando o Sistema de Arquivos do Host (Bind Mounts)
