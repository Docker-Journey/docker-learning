<p align="center">
  <img
    width="400"
    height="200"
    alt="Banner"
    src="https://github.com/user-attachments/assets/884165b8-b2e7-45f3-b7ab-2a1f80156d65" 
  />
</p>

# 🔷 Docker: Overview

Este documento explica **O que é Docker** e **por que aprender Docker**, **o que é este
projeto**, e **cada arquivo** que o compõe - incluindo a ordem em
que eles dependem uns dos outros e o motivo dessa ordem.

---

<div align="center">
  <img width="150" alt="Microsoft Azure" src="https://github.com/user-attachments/assets/4f51bfbd-7e0e-4ce3-b216-d57ffaf18989" />
</div>

## 1. Por que aprender Docker

Antes de containers, rodar uma aplicação em outra máquina exigia
replicar manualmente todo um ambiente: a versão certa da linguagem,
as bibliotecas certas, variáveis de ambiente, configurações do
sistema operacional. Isso gera o problema mais clássico do
desenvolvimento de software:

> "Na minha máquina funciona."

O Docker resolve isso empacotando a aplicação **junto com tudo que
ela precisa pra rodar** - sistema, linguagem, bibliotecas, código e
comando de start - em uma unidade só, chamada **imagem**. Essa
imagem roda de forma idêntica em qualquer lugar que tenha Docker
instalado: seu notebook, o servidor da empresa, a nuvem (AWS, GCP,
Azure), o notebook de outro desenvolvedor.

Por isso Docker é considerado uma habilidade básica em DevOps hoje:

- **Portabilidade** - o mesmo pacote roda igual em qualquer ambiente
- **Isolamento** - cada aplicação roda separada das outras, sem
  conflito de versões de bibliotecas entre projetos diferentes
- **Padronização de times** - todo mundo do time sobe o projeto com
  o mesmo comando, sem precisar seguir um manual de instalação
  manual, cheio de passos que variam por sistema operacional
- **Base para orquestração** - ferramentas como Kubernetes,
  usadas em produção em larga escala, funcionam gerenciando
  containers. Entender Docker é o primeiro degrau pra chegar lá
- **Ambientes de CI/CD** - pipelines de teste e deploy automatizado
  quase sempre rodam dentro de containers, justamente pela
  consistência que eles garantem

## 2. Os objetos do Docker

### 📜 `Dockerfile`
A "receita" que diz ao Docker como construir a imagem.
É lido de cima para baixo, e as instruções que alteram arquivos (`RUN`, `COPY`, `ADD`) viram camadas (layers).
Não roda nada sozinho: ele só serve de entrada para o `docker build`.

### 🖼️ `Image`
O "pacote de instalação": sistema, linguagem, bibliotecas, código e comando de start.
É imutável, como uma foto congelada, e é formada por camadas empilhadas.
Uma mesma imagem pode gerar quantos containers você quiser.

### 📦 `Container`
Uma imagem em execução: um processo isolado na sua máquina, não uma máquina virtual.
Como os apps do celular, cada container roda separado dos outros e do host.
Pode ser criado, parado e removido sem deixar nada instalado na máquina.

### 📮 `Registry`
O lugar onde as imagens ficam guardadas e são compartilhadas, como uma "loja de apps".
O mais conhecido é o Docker Hub, mas existem outros, como AWS ECR e JFrog Artifactory.
Você envia imagens com `docker push` e baixa com `docker pull`.

### 💾 `Volume`
Um espaço para guardar dados fora do container, que continua existindo mesmo se ele for removido.
Sem volume, tudo que o container grava some junto com ele.
É essencial para bancos de dados e qualquer dado que precise persistir.

### 🌐 `Network`
A rede que permite que containers conversem entre si pelo nome, mantendo o isolamento.
O port mapping (`-p 8080:5000`) é o que conecta uma porta do container a uma porta do seu PC.
Sem ele, a aplicação roda, mas você não consegue acessá-la de fora.

### 🧩 `Docker Compose`
Um arquivo YAML que descreve vários containers juntos: imagens, portas, variáveis e volumes.
É como escrever vários `docker run` de uma vez, em um só lugar.
Com `docker compose up`, o ambiente inteiro sobe com um único comando.

## 3. Fluxo visual

```mermaid
%%{init: {'theme':'base', 'themeVariables': {'fontSize':'15px'}}}%%
flowchart TD
    A["📜 <b>Dockerfile</b><br/><i>a receita</i>"]
    B["🖼️ <b>Image</b><br/><i>o pacote congelado</i>"]
    R["📮 <b>Registry</b><br/><i>a loja de imagens</i><br/>Docker Hub, ECR..."]

    subgraph CP["🧩 Docker Compose  ·  docker compose up"]
        direction TB
        subgraph N["🌐 Network"]
            direction LR
            C1["📦 <b>Container</b><br/>sua aplicação"]
            C2["📦 <b>Container</b><br/>ex.: banco de dados"]
            C1 <-- "conversam pelo nome" --> C2
        end
        V["💾 <b>Volume</b><br/><i>dados que persistem</i>"]
        C2 --- V
    end

    H["💻 <b>Seu PC</b><br/>localhost:porta"]

    A -- "docker build" --> B
    B -- "docker push" --> R
    B -- "docker run" --> C1
    R -- "docker pull<br/>(imagem pronta)" --> C2
    C1 -- "-p host:container" --> H

    classDef receita fill:#fce7f3,stroke:#ec4899,stroke-width:2px,color:#831843
    classDef imagem fill:#f3e8ff,stroke:#c084fc,stroke-width:2px,color:#581c87
    classDef registry fill:#fdf4ff,stroke:#e879f9,stroke-width:2px,color:#701a75
    classDef container fill:#ede9fe,stroke:#a855f7,stroke-width:2px,color:#4c1d95
    classDef volume fill:#fce7f3,stroke:#f472b6,stroke-width:2px,color:#831843
    classDef final fill:#a855f7,stroke:#7e22ce,stroke-width:2px,color:#ffffff

    class A receita
    class B imagem
    class R registry
    class C1,C2 container
    class V volume
    class H final

    style CP fill:#fdf2f8,stroke:#ec4899,stroke-width:2px,stroke-dasharray:6 4,color:#9d174d
    style N fill:#faf5ff,stroke:#a855f7,stroke-width:2px,stroke-dasharray:4 4,color:#6b21a8
    linkStyle default stroke:#c084fc,stroke-width:2px
```


---
<div align="center">

Se gostou, deixa uma ⭐

<img width="200" alt="Image" src="https://github.com/user-attachments/assets/aca57b06-3ea1-49e4-96fb-b2a00b8f8918" />

</div>

<div align="center">
Feito com 💙 por RegiMaria
</div>

<div align="center">
