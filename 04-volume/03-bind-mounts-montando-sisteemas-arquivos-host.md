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

---

<div align="center">

# Exercícios

</div>

**Exercício 01:** Mapeando Diretório Atual com Bind Mount

Escreva o comando para rodar um container da imagem Nginx em segundo plano (-d),
mapeando a pasta atual da máquina host ($(pwd)) para o diretório de arquivos estáticos do Nginx no container (/usr/share/nginx/html).

```Bash

# Resposta:
docker run -d -p 8080:80 -v $(pwd):/usr/share/nginx/html nginx
```

**Exercício 02:** Dessectando o Comando Peça por Peça

*Passando um arquivo para um contêiner em execução*

Sua gerente ficou satisfeita com seu novo conhecimento sobre compartilhamento de dados no Docker e
veio com um cenário para você investigar. O aplicativo principal da sua empresa se chama `crane` e 
foi recentemente conteinerizado pelo seu departamento. Esse aplicativo é personalizado para cada
cliente da empresa e, até agora, exigia criar e manter uma imagem de contêiner separada para cada cliente.
Sua gerente quer que você encontre uma maneira de simplificar esse processo.

Depois de revisar a configuração do contêiner, você vê que cada imagem de contêiner procura um arquivo
chamado `customerdata.json` dentro da imagem. Esse arquivo contém as informações específicas do cliente
e precisa ser incluído em cada imagem no momento da build.

Instruções

1. Execute a imagem do contêiner `crane` usando o comando apropriado do Docker. Você deve ver uma mensagem fazendo referência às informações padrão do usuário.

2. Modifique o arquivo `~/workspace/customerdata.json` e atualize o campo `Company Name` para `The File Store`.

3. Execute novamente a imagem do contêiner crane, desta vez passando o arquivo `~/workspace/customerdata.json` para o caminho `/customerdata.json` no contêiner.
Certifique-se de usar exatamente os caminhos mostrados.


```
docker run --rm -v ~/workspace/customerdata.json:/customerdata.json crane
```

| **Pedaço do Comando**           | **O que é?**                  | **Explicação Detalhada**                                                                                             |
|---------------------------------|-------------------------------|----------------------------------------------------------------------------------------------------------------------|
| `docker run`                    | **Ação**                      | Cria e inicia um novo container.                                                                                     |
| `--rm`                          | **Instrução de Limpeza**      | Deleta o container automaticamente assim que ele terminar de rodar (para não acumular lixo no terminal).             |
| `-v`                            | **Flag de Volume/Bind Mount** | Indica que vamos fazer uma "ponte" entre o seu computador e o container.                                             |
| `~/workspace/customerdata.json` | **Caminho no HOST (Seu PC)**  | O arquivo real que está salvo no seu computador/terminal. *(Fica SEMPRE à esquerda dos dois pontos `:`)*.           |
| `/customerdata.json`            | **Caminho no CONTAINER**      | Onde o aplicativo de dentro do container espera encontrar o arquivo. *(Fica SEMPRE à direita dos dois pontos `:`)*. |
| `crane`                         | **Nome da Imagem**            | A imagem base a partir da qual o container será criado. *(Vem SEMPRE no final do comando)*.                          |


## Solução

Por que executamos dessa forma e O QUE FIZEMOS AQUI?

- O Problema Antigo da Empresa:

Antes, a empresa precisava criar uma imagem diferente para cada cliente:

- crane-cliente-a (com os dados do Cliente A)

- crane-cliente-b (com os dados do Cliente B)

Isso exigia compilar dezenas de imagens com docker build o tempo todo, consumindo espaço e tempo imensos.

## A Solução Inteligente:
Em vez de recompilar a imagem toda vez que o cliente muda, nós usamos uma única imagem genérica (crane)
e injetamos o arquivo de dados do cliente em tempo de execução via Bind Mount (`-v`)!

- O app `crane` foi programado para procurar o arquivo na raiz do container (/customerdata.json).

- a gente pegou o arquivo atualizado no nosso computador (~/workspace/customerdata.json) e "sobrepôs" aquele caminho dentro do container.

- Quando o crane iniciou, ele leu o seu arquivo da máquina host como se ele estivesse nativamente instalado dentro da imagem!

**Por que no passo 1 tinha --rm e no passo 3 não tinha?**

Dúvida sobre a diferença entre as duas execuções:

**No Passo 1 (Com --rm):**

```Bash
docker run --rm -v ~/workspace/customerdata.json:/customerdata.json crane
```

A flag `--rm` foi usada para um teste rápido. O container roda, mostra o resultado no terminal e
é destruído na hora. É a melhor prática para testar se a injeção do arquivo funcionou sem deixar rastros no sistema.

**No Passo 3 (Sem --rm):**

```Bash
docker run -v ~/workspace/customerdata.json:/customerdata.json crane
```

Ao tirar o --rm, o container executa, mas, ao terminar, ele fica guardado no histórico de containers do seu sistema 
(visível ao rodar docker ps -a). Em plataformas de exercícios (como o DataCamp), alguns testes automatizados verificam
o histórico do Docker para saber se o container realmente existiu após o comando, por isso solicitam a execução sem a remoção automática.

**Exercício 04**
O contêiner do aplicativo crane procura um arquivo chamado `/customerdata.json` quando é executado. 
Se você não passar um arquivo, o programa ainda funciona e imprime "DEFAULT NAME". 
Isso significa que ele está lendo um arquivo dentro da própria imagem do contêiner.

*O que você acha que acontece com esse arquivo quando você passa um arquivo do host usando docker run -v?*

Resposta: O arquivo incluído na imagem fica temporariamente oculto e o seu é referenciado no lugar dele

**EXPLICAÇÃO:**

Essa é uma das características mais importantes do funcionamento dos Bind Mounts no Docker

A resposta correta é: "O arquivo incluído na imagem fica temporariamente oculto e o seu é referenciado no lugar dele".

**Por que essa é a resposta certa?**

Para entender o porquê, pense no sistema de arquivos do Docker como uma pilha de folhas de papel transparentes (camadas):

1. A Imagem Base (O arquivo padrão)
Quando a imagem crane foi criada, o desenvolvedor colocou o arquivo `customerdata.json` padrão dentro dela.
Esse arquivo está gravado de forma fixa na camada de leitura da imagem.

3. O que o -v (Bind Mount) faz na prática?

Quando você executa o comando com o -v:

```Bash
docker run -v ~/workspace/customerdata.json:/customerdata.json crane
```
*Você não está apagando nem modificando a imagem original. O Docker faz uma espécie de "sobreposição" (ou mascaramento):*

- O Docker pega o seu arquivo do host (~/workspace/customerdata.json).

- Ele coloca esse arquivo exatamente por cima do caminho /customerdata.json dentro do container em execução.

- Como o seu arquivo fica na frente, o programa crane enxerga apenas o seu arquivo e lê os novos dados.

- O arquivo original que veio dentro da imagem continua lá, mas fica "atrás" do seu mount, ou seja, temporariamente oculto.

**O que acontece se você rodar o container SEM o -v na próxima vez?**

Se você rodar novamente apenas:

```
docker run crane
```
O Docker não vai usar o seu arquivo do host. Como não há nenhuma sobreposição (mount) ativa, o arquivo original da imagem deixa de ficar oculto e volta a aparecer normalmente! É por isso que o container volta a imprimir `"DEFAULT NAME"`

**Resumo do Comportamento**

- Ele NÃO apaga o arquivo da imagem: A imagem Docker é imutável (não pode ser alterada depois de construída).

- Ele NÃO renomeia o arquivo: O Docker apenas redireciona a leitura daquele caminho específico para o seu computador.

- Ele apenas OCULTA enquanto o container roda: Enquanto a montagem estiver ativa, o caminho aponta para o seu arquivo no host.
Quando o container para e outro é iniciado sem o -v, o arquivo original da imagem reaparece.

**Como injetar e persistir arquivos ou configurações dentro de um container Docker sem precisar alterar a imagem original**


**Exercicio 06:**
Depois de conversar com sua gerente sobre a capacidade de passar configurações para o contêiner do aplicativo da sua empresa
(imagem `crane:latest`), ela pediu que você fornecesse uma lista de outras opções que se comportariam de maneira semelhante.
O cenário principal é garantir que um arquivo chamado `/data/customerinfo.json` esteja presente no contêiner.

**Instruções**
Avalie se cada opção permitiria passar informações para um contêiner e coloque-a no grupo apropriado. 

Classificação dos Grupos

- Permite passar o arquivo `/data/customerinfo.json` para o container ✅

* `docker run -v /host/path/customerinfo.json:/data/customerinfo.json crane:latest`

Explicação: Usa a sintaxe de Bind Mount (-v). O arquivo presente no Host é espelhado exatamente no caminho /data/customerinfo.json dentro do container.

* `docker run --mount type=bind,source=/host/path/customerinfo.json,target=/data/customerinfo.json crane:latest`

Explicação: Usa a sintaxe explícita --mount com type=bind. Ela tem exatamente o mesmo resultado do -v, sobrepondo o arquivo local no destino esperado.

- Criar um Docker Volume e montar em `/data` (ex: docker run -v meu_volume:/data crane:latest contendo o arquivo)

Explicação: Permite que um volume gerenciado pelo Docker injete os dados dentro do diretório /data do container.


**❌NÃO permite passar o arquivo /data/customerinfo.json para o container ❌**

* `docker run -p 8080:80 crane:latest`

Explicação: A flag -p serve para mapeamento de portas de rede entre o Host e o Container, não para compartilhamento de arquivos do sistema.

* `docker run -e DATA=/data/customerinfo.json crane:latest`

Explicação: A flag -e define variáveis de ambiente (ENV). Ela apenas cria uma variável com o texto do caminho, mas não passa nem injeta o conteúdo do arquivo físico.

* `docker run -w /data/customerinfo.json crane:latest`

Explicação: A flag `-w` define o diretório de trabalho (WORKDIR) onde o container iniciará a execução, e não injeta arquivos externos.

**Resumo: Métodos diferentes, mesmo resultado**

Se o objetivo é injetar o arquivo /data/customerinfo.json em tempo de execução sem alterar a imagem crane:latest, as duas opções padrão do Docker são:

1. Sintaxe Curta (-v):
```
docker run -v /caminho/no/host/customerinfo.json:/data/customerinfo.json crane:latest
```

2. Sintaxe Explícita (--mount):

```
docker run --mount type=bind,source=/caminho/no/host/customerinfo.json,target=/data/customerinfo.json crane:latest
```
Ambas as abordagens alcançam exatamente o mesmo resultado: ocultar temporariamente qualquer arquivo padrão no container e substituí-lo pelo arquivo fornecido pelo Host.

--- 

O que estudar a seguir:
⬅️ Volte para 01 - Volumes
⬅️ Volte para 02 - Comandos intermediários
