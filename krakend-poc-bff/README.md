# KrakenD POC - Back-for-Front

Este é um projeto de demonstração para explorar o conceito de Back-for-Front (BFF) usando o KrakenD como API Gateway. A PoC foca na agregação e filtragem de dados provenientes de múltiplos serviços backend para simplificar o consumo pelo frontend. Este projeto está organizado como uma subpasta do projeto principal **KrakenD POC**.

---

## Visão Geral

O **KrakenD POC - Back-for-Front** implementa um gateway que expõe um único endpoint para consumir dados combinados de múltiplos serviços backend, utilizando como base a API de exemplo [JSONPlaceholder](https://jsonplaceholder.typicode.com).

---

## Estrutura do Projeto

- **compose.yml**: Configuração do Docker Compose para gerenciar o ambiente do KrakenD.
- **krakend.json**: Configuração principal do KrakenD para o endpoint de agregação.
- **requests.http**: Exemplos de requisições HTTP para testar o endpoint BFF.

---

## Configurando o Ambiente

### Pré-requisitos

- Docker e Docker Compose instalados.

### Instalação

1. Clone o repositório principal do projeto:

    ```bash
    git clone https://github.com/seu-usuario/krakend-poc.git
    ```

2. Navegue para a subpasta **krakend-poc-bff**:

    ```bash
    cd krakend-poc/krakend-poc-bff
    ```

3. Suba o ambiente com o Docker Compose:

    ```bash
    docker-compose up -d
    ```

4. Verifique os serviços:

    ```bash
    docker-compose ps
    ```

---

## O Que Cada Arquivo Faz

### compose.yml

Configura o ambiente Docker para o KrakenD:

```yaml
version: '3.8'

services:
  krakend:
    image: devopsfaith/krakend:latest
    container_name: krakend
    ports:
      - "8080:8080"
    volumes:
      - ./krakend.json:/etc/krakend/krakend.json
    command: ["run", "-c", "/etc/krakend/krakend.json", "-d"]
```

- **krakend**: Serviço principal com a imagem oficial do KrakenD.
- **ports**: Mapeia a porta `8080` para o host local.
- **volumes**: Monta o arquivo de configuração no container.
- **command**: Comando para iniciar o KrakenD em modo debug.

---

### krakend.json

Configura o endpoint BFF que agrega dados de usuários e seus posts:

```json
{
    "endpoint": "/blog/{user}",
    "method": "GET",
    "output_encoding": "json",
    "backend": [
        {
            "host": ["https://jsonplaceholder.typicode.com"],
            "encoding": "json",
            "url_pattern": "/users/{user}",
            "group": "user",
            "deny": ["id"]
        },
        {
            "host": ["https://jsonplaceholder.typicode.com"],
            "encoding": "json",
            "url_pattern": "/posts?userId={user}",
            "deny": ["id", "userId"],
            "group": "last_post"
        }
    ]
}
```

- **/blog/{user}**: Endpoint que combina dados do usuário e de seus posts.
- **Agregação**: Dados de `/users/{user}` e `/posts?userId={user}`.
- **Filtragem**: Remove campos desnecessários, como `id` e `userId`.

---

### requests.http

Arquivo com exemplos de requisições HTTP para testar o endpoint BFF:

```http
### Testar o endpoint BFF
GET http://localhost:8080/blog/1
Content-Type: application/json
```

---

## Testando o Endpoint

1. Certifique-se de que o serviço está rodando na porta `8080`.

2. Use ferramentas como:
    - [VSCode REST Client](https://marketplace.visualstudio.com/items?itemName=humao.rest-client)
    - Postman
    - cURL

3. Faça a requisição para o endpoint `/blog/{user}`:

    ```bash
    curl -X GET http://localhost:8080/blog/1
    ```

4. Valide a resposta, que deve combinar os dados do usuário e seus posts:

```json
{
  "user": {
    "name": "Leanne Graham",
    "username": "Bret",
    "email": "Sincere@april.biz"
  },
  "last_post": [
    {
      "title": "Post Title",
      "body": "Post Content"
    }
  ]
}
```

---

## Próximos Passos

- Adicione novos serviços ao endpoint BFF para expandir a agregação.
- Explore funcionalidades avançadas do KrakenD, como autenticação e cache distribuído.
- Substitua o JSONPlaceholder por APIs reais em seus testes e implemente filtros mais complexos.