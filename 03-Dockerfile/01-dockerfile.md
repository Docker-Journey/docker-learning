# 03 - Dockerfile (Construindo Imagens Personalizadas)

> **Resumo:** O `Dockerfile` é a "receita" de texto usada para automatizar a criação de imagens personalizadas. Ele define a imagem base, instala dependências, copia arquivos do projeto e configura o processo principal de inicialização.

---

Na analogia da Bicicleta, que você confere aqui [`Container vs Imagens`](https://github.com/Docker-Journey/docker-learning/blob/main/01-container/02-container.md), Dockerfile é o manual de instruções de montagem (ou a receita do projeto) que ensina a fábrica a criar o molde.

📝 Dockerfile: O Manual / Manual de Instruções (O papel onde você escreveu passo a passo: "Passo 1: Pegue a estrutura do modelo base 'Ubuntu'. Passo 2: Pinte de azul. Passo 3: Coloque pneus de corrida. Passo 4: Defina que, ao subir na bicicleta, ela deve começar a pedalar sozinha (CMD)").

🛠️ docker build: O Processo de Fabricação do Molde (Você entrega o manual na mão da fábrica e ela constrói o molde oficial com base no que você escreveu).

📐 Imagem Docker: O Molde Físico de Impressão (O molde estático já pronto, aprovado e guardado na prateleira).

🏭 Docker Engine: A Fábrica / Linha de Montagem (A estrutura que lê o manual com build e fabrica/executa a bicicleta com run).

🚲 Container: A Bicicleta Pronta em Movimento (A bicicleta de verdade rodando na rua).


Sendo assim:

O Dockerfile é texto puro: É um arquivo leve que você commita no Git.

A Imagem é o resultado compilado: Criada a partir do Dockerfile com o comando docker build.

O Container é a execução viva: Criado a partir da Imagem com o comando docker run.

---

## 🗂️ Estrutura do Lab
- `Dockerfile`: Instruções de build da imagem.
- `app.py`: Script Python simples executado pela imagem.
- `exercicios-dockerfile.md`: Documentação deste experimento.

---

## 🤔 O que é cada instrução?

| Instrução | Para que serve? | Exemplo no Lab |
| :--- | :--- | :--- |
| **`FROM`** | Define a imagem base de onde o build vai começar | `FROM python:3.10-slim` |
| **`WORKDIR`** | Cria e define a pasta de trabalho padrão dentro do contêiner | `WORKDIR /app` |
| **`COPY`** | Copia arquivos da sua máquina física para dentro da imagem | `COPY app.py /app/app.py` |
| **`CMD`** | Define o comando padrão que roda ao iniciar o contêiner | `CMD ["python", "app.py"]` |

---

## 🔨 Hands-on (Passo a Passo)

### 1. Script da Aplicação (`app.py`)
```python
print("🚀 Docker Journey: Meu primeiro script Python rodando dentro do contêiner!")
```

### 2. Configuração da Receita / Manual (Dockerfile)

```python
FROM python:3.10-slim
WORKDIR /app
COPY app.py /app/app.py
CMD ["python", "app.py"]
```
### 3. Comandos Executados no Terminal

```bash
# Criar a imagem com a tag 'meu-app-python' no diretório atual (.)
docker build -t meu-app-python .

# Rodar o contêiner usando o CMD padrão e removendo-o ao finalizar (--rm)
docker run --rm meu-app-python

# Sobrescrever o CMD padrão para depurar e inspecionar os arquivos via bash
docker run -it --rm meu-app-python bash
```

💥 Pontos de Atenção
- Esquecer o ponto . no build: O comando docker build exige o caminho do contexto de build
(onde está o Dockerfile). Sem o ., o Docker retorna um erro pedindo o caminho.

- Ordem das instruções: Se colocar o COPY antes do WORKDIR, os arquivos vão para a raiz / em vez da pasta /app.

- Retenção de contêineres mortos: Rodar muitos testes gera contêineres parados acumulando espaço em disco.

- O CMD não roda durante o docker build, apenas durante o docker run.

- A instrução WORKDIR evita ter que passar caminhos absolutos gigantes em todos os comandos subsequentes.

- Passar bash ao final do docker run ignora a instrução CMD do Dockerfile, permitindo explorar o contêiner por dentro em modo interativo.