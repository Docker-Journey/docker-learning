# 03 - Alteração de Usuários e do Diretório de Trabalho (`WORKDIR` e `USER`)

## Organização e Segurança no Dockerfile

Por padrão, as imagens Docker são construídas e executadas utilizando o diretório raiz (`/`)
e a conta de superusuário (**`root`**). 

Usar essa configuração padrão traz dois grandes problemas:
1. **Falta de Organização:** Arquivos jogados no diretório raiz do sistema operacional.

2. **Risco Crítico de Segurança:** Se uma aplicação rodando como `root` for invadida por uma vulnerabilidade,
o atacante ganha privilégios máximos dentro do container e pode tentar ataques de escape para o sistema hospedeiro (*host*).

Para resolver isso, utilizamos duas instruções fundamentais: **`WORKDIR`** e **`USER`**.

---

### 1. Definindo o Diretório de Trabalho: `WORKDIR`

Em vez de usar comandos de shell temporários como `RUN cd /app` (cujo efeito é perdido entre as camadas do Docker),
utilizamos a instrução `WORKDIR`.

* **O que faz:** Define o diretório de trabalho padrão dentro da imagem.
Todas as instruções subsequentes (`RUN`, `COPY`, `ADD`, `CMD`, `ENTRYPOINT`) serão executadas a partir dessa pasta.

* **Criação Automática:** Se o caminho especificado não existir na imagem, o Docker o cria automaticamente.

```dockerfile
FROM python:3.10

# Define /app como pasta de trabalho (cria se não existir)
WORKDIR /app

# Copia os arquivos diretamente para /app
COPY requirements.txt .

```
> 💡 Dica de terminal: Você pode testar qual é o diretório de trabalho padrão de uma imagem executando:
`docker run --rm minha-imagem pwd`

2. Restringindo Privilégios: USER
   
A instrução `USER` altera o usuário ativo (e opcionalmente o grupo) responsável por executar 
as instruções seguintes no Dockerfile e por rodar o container final.

- Princípio do Menor Privilégio: A aplicação deve rodar apenas com as permissões necessárias
para a sua execução, nunca como administrador (root).

- Como utilizar: Você pode indicar um usuário existente na imagem base (ex: node em imagens Node.js)
ou criar um usuário dedicado através da instrução `RUN adduser`.

```dockerfile
# Criando um usuário não-root chamado 'repl'
RUN adduser -D repl

# Alternando a execução para o usuário 'repl'
USER repl
```
> Dica de terminal: Você pode verificar qual usuário está executando o container com o comando:
`docker run --rm minha-imagem whoami`

3. A Importância das Permissões com `COPY --chown`
Quando trocamos para um usuário não-root (`USER repl`),
qualquer arquivo copiado depois via `COPY` por padrão ainda pertencerá ao `root`.
Se a sua aplicação precisar escrever nesses arquivos
(como gerar logs ou salvar arquivos estáticos), ocorrerá um erro de
permissão negada (Permission Denied).

Para resolver isso, usamos a flag --chown:
```dockerfile
COPY --chown=repl:repl . .
```

4. Estrutura Comparativa: Inseguro vs. Seguro
   
Inseguro / Desorganizado ❌

```dockerfile
FROM python:3.10
# Roda na raiz (/) e como usuário ROOT
COPY . .
RUN pip install -r requirements.txt
CMD ["python", "main.py"]

```

Seguro / Organizado ✅

```dockerfile

FROM python:3.10

# 1. Cria usuário não-root
RUN adduser -D repl

# 2. Alterna para o novo usuário
USER repl

# 3. Define a pasta de trabalho (ex: pasta home do usuário)
WORKDIR /home/repl

# 4. Copia dependências ajustando a propriedade para o usuário 'repl'
COPY --chown=repl:repl requirements.txt .

# 5. Instala pacotes
RUN pip install -r requirements.txt

# 6. Copia o código-fonte com a propriedade correta
COPY --chown=repl:repl . .

CMD ["python", "main.py"]
```
















