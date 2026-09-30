# Exercício

Neste módulo, vamos criar uma aplicação Python simples, 
empacotá-la numa imagem personalizada e aprender como o `WORKDIR`,
o `COPY` e o `CMD` trabalham juntos.

## Passo 1: Preparar o diretório no terminal

Navegue até a pasta do lab no seu terminal VS Code e crie os dois arquivos necessários:

```bash
touch Dockerfile app.py
```

## asso 2: Escrever o código do script (app.py)

Abra o arquivo app.py no VS Code e adicione a seguinte linha de código:
```python
print("🚀 Docker Journey: Meu primeiro script Python rodando dentro do contêiner!")
```

## Passo 3: Escrever o Dockerfile

Abra o Dockerfile e insira as seguintes instruções:

```python
# 1. Imagem base oficial do Python (tag leve)
FROM python:3.10-slim

# 2. Define o diretório de trabalho dentro do contêiner
WORKDIR /app

# 3. Copia o script local para dentro do diretório /app do contêiner
COPY app.py /app/app.py

# 4. Define o comando padrão que executará o script assim que o contêiner nascer
CMD ["python", "app.py"]

```

## 1.Passo 4: Gerar a imagem personalizada:

docker build.

No terminal, execute o comando de build para compilar a imagem com a tag meu-app-python:
```bash
docker build -t meu-app-python .
```
> Atenção: Não esqueça do ponto . no final! Ele indica ao Docker que o Dockerfile está no diretório atual.

## 2.Passo 5: Testar o contêiner com o CMD padrão:

docker run.

Agora rode o contêiner criado a partir da sua nova imagem:
```bash
docker run --rm meu-app-python
```

a rode o contêiner criado a partir da sua nova imagem:

O que vai acontecer: O Docker criará o contêiner, executará o CMD ["python", "app.py"]
imprimindo a mensagem do script no terminal e, graças à flag --rm, 
destruirá o contêiner automaticamente após a finalização.

## Passo 6: Entrar no contêiner para depuração (Debug):
Sobrescrevendo o CMD.

Imagine que o script deu erro e você quer entrar no contêiner
para inspecionar a pasta /app. Sobrescreva o CMD abrindo o bash:
```bash
docker run -it --rm meu-app-python bash
```

Dentro do contêiner:

1. Digite pwd (você estará em /app por conta da instrução WORKDIR).

2. Digite ls (verá o seu app.py lá dentro).

3. Digite python app.py para rodar o script manualmente.

4. Digite exit para sair.