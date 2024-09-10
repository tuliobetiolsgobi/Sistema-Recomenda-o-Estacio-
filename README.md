
# Sistema de Recomendação em Tempo Real com Apache Spark e MongoDB

## 🚀 1 - Introdução

Este projeto tem como objetivo criar um sistema de recomendação utilizando o modelo ALS (Alternating Least Squares) da biblioteca MLlib do Apache Spark. As recomendações serão armazenadas localmente no MongoDB. Também será desenvolvida uma API com o FastAPI. Além disso, uma aplicação de Spark Streaming se conectará a um tópico do Kafka para fornecer recomendações em tempo real.

## ⚙️ 2 - Hardware e Aplicativos Utilizados

- **Apache Spark**: Plataforma de processamento de dados em larga escala.
- **MLlib**: Biblioteca de aprendizado de máquina do Spark.
- **MongoDB**: Banco de dados NoSQL.
- **FastAPI**: Framework para construção de APIs em Python.
- **Docker**: Ferramenta para criação e gerenciamento de containers.

## 📖 3 - Requisitos do Projeto

1. Implementar um sistema de recomendação utilizando o modelo ALS do Spark MLlib.
2. Armazenar as recomendações no MongoDB.
3. Criar uma API usando FastAPI para fornecer recomendações baseadas nas preferências dos usuários.

## 📝 4 - Procedimentos e Resultados

### Configuração do Ambiente

Utilizaremos Docker com as imagens do MongoDB e do Jupyter Notebook.

![Docker](./assets/docker.png)

Como já tenho alguns containers do MongoDB instalados, alterei a porta para 28017 para evitar conflitos.

Criaremos uma rede para isso. Execute o seguinte comando no prompt de comando:

```
docker network create app-tier --driver bridge
```

![Rede Docker](./assets/rede.png)

Em seguida, instale o Jupyter Notebook:

```
docker pull jupyter/pyspark-notebook
```

![Instalação Jupyter](./assets/instalacao_jupyter.png)

Depois, instale o MongoDB:

```
docker pull mongo
```

![Instalação Mongo](./assets/instalacao_mongo.png)

Uma vez que os containers do Jupyter e do Mongo estiverem em execução, acesse o Jupyter e execute o script `ExemploALS.ipynb`.

![Jupyter](./assets/jupyter.png)

O script executa as seguintes etapas:

#### 1 - Importação de Bibliotecas
Nesta célula, são importadas as bibliotecas necessárias para a criação do sistema de recomendação e verificada a versão do Python para garantir compatibilidade.

#### 2 - Configuração da Sessão Spark
Cria uma sessão do Spark e configura a conexão com um banco de dados MongoDB.

#### 3 - Leitura e Processamento dos Dados
Lê um arquivo de texto com avaliações de filmes e transforma essas avaliações em um DataFrame do Spark.

#### 4 - Divisão dos Dados
Divide os dados em conjuntos de treinamento e teste, sendo 80% para treinamento e 20% para teste.

#### 5 - Treinamento do Modelo
Configura e treina um modelo de recomendação ALS usando o conjunto de treinamento.

#### 6 - Avaliação do Modelo
Transforma os dados de teste com o modelo treinado e avalia a precisão do modelo usando o RMSE (Root Mean Square Error).

#### 7 - Recomendação para Usuários
Gera recomendações de filmes para todos os usuários e exibe as primeiras 10 recomendações.

#### 8 - Recomendação para Itens (Filmes)
Gera recomendações de usuários para todos os filmes e exibe as primeiras 20 recomendações.

#### 9 - Seleção de Recomendação por IDs de Filmes
Seleciona apenas os IDs dos filmes recomendados para os usuários.

#### 10 - Salvando as Recomendações no MongoDB
Salva as recomendações geradas no MongoDB.

Depois de executar o script de treinamento do modelo e inserir as recomendações no MongoDB, utilizamos o MongoDB Compass para verificar os itens inseridos:

![MongoDB Compass](./assets/Mongo.png)

![Recomendações](./assets/Recomendacoes.png)

### 10 - Criação da API utilizando FastAPI

Primeiro, criaremos um script Python para realizar consultas em nosso MongoDB. Para isso, precisamos instalar os requisitos executando o arquivo `requirements`:

```
fastapi==0.68.0
uvicorn==0.15.0
pymongo==3.12.0
```

O script será o seguinte:

```python
from pymongo import MongoClient

def inicia_conexao():
    client = MongoClient("localhost", 28017)
    db = client['puc']
    col = db.recomendacoes
    return col

def consulta_recomendacoes(usuario, conexao):
    recomendacoes = list(conexao.find({"userId": usuario}))
    list_rec = []
    for rec in recomendacoes:
        list_rec.append({'id': rec['movieId'], 'rating': rec['rating']})
    return list_rec

conn = inicia_conexao()
print(consulta_recomendacoes(28, conn))
```

Este script Python conecta-se a um banco de dados MongoDB, especificamente à coleção "recomendacoes" dentro do banco "puc" em um servidor. Ele contém uma função para iniciar essa conexão e outra para consultar as recomendações de filmes de um usuário específico, identificado pelo campo `userId`. A consulta retorna uma lista de dicionários contendo `movieId` e `rating` das recomendações do usuário.

Agora estamos prontos para criar nossa API utilizando FastAPI. Para isso, vamos executar o seguinte script:

```python
from fastapi import FastAPI
import uvicorn
from bd import mongo
from typing import List

app = FastAPI()
conexao = mongo.inicia_conexao()

@app.get("/rec/v1")
def rota_padrao():
    return {"Rota padrão": "Você acessou a rota default"}

@app.get("/rec/v2/{usuario}")
def consulta_rec(usuario: int):
    return {"usuario": usuario, "resultado_recs": mongo.consulta_recomendacoes(usuario, conexao)}

@app.post("/rec/v3/")
def consulta_multi_recs(usuarios: List[int]):
    results = {}
    for usuario in usuarios:
        recs = mongo.consulta_recomendacoes(usuario, conexao)
        movie_ids = [rec['id'] for rec in recs]
        results[usuario] = movie_ids
    return results

if __name__ == "__main__":
    uvicorn.run(app, host='localhost', port=8000)
```

Este script define uma aplicação web usando FastAPI que se conecta a um banco de dados MongoDB para fornecer recomendações de filmes. Ele possui três rotas: a primeira retorna uma mensagem padrão, a segunda consulta recomendações para um único usuário especificado, e a terceira recebe uma lista de usuários e retorna apenas os IDs dos filmes recomendados para cada um. A aplicação é executada localmente usando o servidor `uvicorn`.

![API FastAPI](./assets/api.png)

Ao executarmos a terceira rota, passamos uma lista de usuários e os IDs dos filmes recomendados são retornados:

![API FastAPI Resultados](./assets/api2.png)

## 📋 5 - Conclusão

A execução deste projeto permitiu a implementação de um sistema de recomendação robusto utilizando diversas tecnologias de big data e aprendizado de máquina. O uso do modelo ALS do Spark MLlib proporcionou recomendações eficazes, e o armazenamento no MongoDB garantiu uma recuperação rápida dos dados. A API desenvolvida com FastAPI facilitou o acesso às recomendações, enquanto a integração com Kafka e Spark Streaming permitiu o fornecimento de recomendações em tempo real.

Integrantes do projeto de extensão:
Jean Oliveira Fraga - Matricula:202106009922
Luciana Alburquerque - Matricula: 202205183599
Tulio Betiol Sgobi – Matricula: 202205025403

