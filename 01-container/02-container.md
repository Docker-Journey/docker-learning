# 🐳 Docker: Imagem x Container

## 🚲 A analogia da bicicleta

Se você está começando a estudar Docker, uma das primeiras coisas que precisa entender é a diferença entre **imagem** e **container**.

Uma forma simples de visualizar isso é pensar na **IMAGEM** como uma **fábrica de bicicletas**.


<p align="center">
  <img
    width="800"
    height="400"
    alt="Banner"
    src="https://github.com/user-attachments/assets/f9274074-f10b-4c5d-b917-281b2ca1b657" 
  />
</p>

---

## Imagem Docker = É fábrica de bicicletas. Projeta o projeto/molde da bicicleta

Imagine que temos o **projeto de um modelo de bicicleta**.

Nesse projeto está definido que a bicicleta precisa ter:

* 🚲 2 rodas
* 🔲 quadro
* 🪑 selim
* 🛞 pneus
* 🕹️ guidão
* ⚙️ pedais
* 📋 instruções de funcionamento

Esse projeto define a **estrutura essencial** da bicicleta.

No Docker, esse papel é desempenhado pela **imagem**.

> 💡 **Imagem Docker = projeto/molde que define como o container será criado.**

Uma imagem contém tudo o que é necessário para criar e executar um determinado ambiente: arquivos, dependências, configurações e instruções.

---

## Container = bicicleta produzida

Agora imagine que você pega esse projeto e produz uma bicicleta.

Essa bicicleta é uma **instância** daquele projeto.

No Docker, essa instância é o **container**.

Quando você executa:

```bash
docker run minha-imagem
```

é como dizer:

> 🏭 "Produza uma bicicleta a partir desse projeto."

O resultado é um container.

```text
               IMAGEM DOCKER
           "Projeto da bicicleta"
                    │
             docker run
                    │
                    ↓
              🐳 CONTAINER
              🚲 Bicicleta
```

---

## Vários containers podem usar a mesma imagem

Agora vem uma parte muito importante:

**uma mesma imagem pode criar vários containers.**

Imagine que temos uma única imagem:

```text
       IMAGEM
"Projeto da bicicleta"
```

A partir dela podemos criar:

```text
                      IMAGEM DOCKER
                 "Projeto da bicicleta"
                           │
              ┌────────────┼────────────┐
              ↓            ↓            ↓
           🚲 Azul      🚲 Vermelha   🚲 Verde
          Container      Container     Container
```

Todas as bicicletas foram produzidas a partir do **mesmo projeto**.

Por isso, elas possuem a mesma estrutura essencial.

Mas cada uma é uma **instância independente**.

---

## Cada container pode ter características próprias

Imagine agora:

```text
🚲 Bicicleta Azul
   └── 🧺 cesta

🚲 Bicicleta Vermelha
   └── 🔴 rodinhas

🚲 Bicicleta Verde
   └── 🪑 banco diferente
```

A ideia é semelhante no Docker.

Os containers podem ter:

* dados diferentes;
* configurações diferentes;
* processos diferentes;
* estados diferentes;
* portas diferentes;
* volumes diferentes.

Mesmo assim, todos podem ter sido criados a partir da **mesma imagem**.

---

## 🛑 Se eu parar um container, os outros continuam

Imagine que a bicicleta vermelha pare de funcionar:

```bash
docker stop container-vermelha
```

Isso não significa que as outras bicicletas vão parar.

```text
🚲 Azul       → funcionando ✅

🚲 Vermelha   → parada 🛑

🚲 Verde      → funcionando ✅
```

Isso acontece porque cada container é uma **instância independente**.

---

# E onde entra o Docker Engine?

Aqui fazemos uma pequena correção na analogia.

A **imagem não é exatamente a fábrica**.

Uma maneira mais precisa de pensar é:

> **Docker Engine = fábrica que sabe produzir e executar**
>
> **Imagem = projeto/molde da bicicleta**
>
> **Container = bicicleta produzida a partir do projeto**

Visualmente:

```text
               IMAGEM
        "Projeto da bicicleta"
                  │
                  │ docker run
                  ↓
          🐳 DOCKER ENGINE
                  │
          ┌───────┼────────┐
          ↓       ↓        ↓
         🚲       🚲       🚲
        Azul    Vermelha   Verde
      Container Container Container
```

---

# E onde entra o `docker build`?

Agora podemos conectar essa analogia com o que fazemos no terminal.

Quando executamos:

```bash
docker build -t pipeline_debug .
```

estamos dizendo ao Docker:

> "Construa uma imagem chamada `pipeline_debug` usando o projeto que está neste diretório."

Podemos pensar:

```text
Dockerfile
    │
    │ docker build
    ↓
  IMAGEM
pipeline_debug
```

O **Dockerfile** é como as instruções usadas para construir o nosso projeto/molde.

---

# E o `docker run`?

Depois que temos a imagem:

```text
📐 pipeline_debug
```

podemos criar um container:

```bash
docker run pipeline_debug
```

Visualmente:

```text
📐 pipeline_debug
       │
       │ docker run
       ↓
🚲 Container
```

E podemos fazer novamente:

```bash
docker run pipeline_debug
```

E novamente:

```bash
docker run pipeline_debug
```

Teremos:

```text
             📐 pipeline_debug
                    │
       ┌────────────┼────────────┐
       ↓            ↓            ↓
    🚲 #1         🚲 #2         🚲 #3
  Container     Container     Container
```

**Uma imagem pode gerar vários containers.**

---

# 🧠 Resumo

| Docker           | Analogia da bicicleta           |
| ---------------- | ------------------------------- |
| 📝 `Dockerfile`  | Instruções do projeto           |
| 📐 Imagem        | Projeto/molde da bicicleta      |
| 🐳 Docker Engine | Fábrica que cria/executa        |
| 🚲 Container     | Bicicleta produzida             |
| `docker build`   | Construir o projeto/molde       |
| `docker run`     | Produzir/executar uma bicicleta |

---

## ⭐ Para guardar

Se você lembrar de apenas duas coisas, lembre destas:

> **Imagem é o modelo. Container é uma instância desse modelo.**

E:

> **Uma imagem pode ser usada para criar vários containers.** 🐳🚲

Essa diferença é uma das bases para entender Docker.
