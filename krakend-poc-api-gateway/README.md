# KrakenD POC - API Gateway

Este é um projeto de demonstração para explorar as capacidades do KrakenD como API Gateway, com foco em agregação, filtragem e configuração simplificada. Ele está organizado como uma subpasta do projeto maior **KrakenD POC**.

---

## Visão Geral

O **KrakenD POC - API Gateway** implementa um gateway para expor endpoints que agregam e manipulam dados de serviços backend. Nesta PoC, utilizamos a API de exemplo [JSONPlaceholder](https://jsonplaceholder.typicode.com).

---

## Estrutura do Projeto

- **compose.yml**: Configuração do Docker Compose para gerenciar o ambiente do KrakenD.
- **krakend.json**: Arquivo principal de configuração do KrakenD.
- **requests.http**: Exemplos de requisições para testar os endpoints.

---

## Configurando o Ambiente

### Pré-requisitos

- Docker e Docker Compose instalados.

### Instalação

1. Clone o repositório principal do projeto:

    ```bash
    git clone https://github.com/qbem-repos/krakend-poc.git
    ```

2. Navegue para a subpasta **krakend-poc-api-gateway**:

    ```bash
    cd krakend-poc/krakend-poc-api-gateway
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

Define os endpoints e a lógica de agregação da API Gateway:

- **/v1/posts**: Gerencia dados de posts (GET e POST).
- **/v1/posts/{id}**: Manipula posts específicos (PUT e DELETE).

Exemplo de configuração de um endpoint:

```json
{
  "endpoint": "/v1/posts",
  "method": "GET",
  "output_encoding": "negotiate",
  "backend": [
    {
      "url_pattern": "/posts",
      "encoding": "json",
      "host": ["https://jsonplaceholder.typicode.com"]
    }
  ]
}
```

---

### requests.http

Arquivo com exemplos de requisições HTTP para testar os endpoints:

```http
### Testar listagem de posts
GET http://localhost:8080/v1/posts

### Testar criação de post
POST http://localhost:8080/v1/posts
Content-Type: application/json

{
  "title": "Meu Post",
  "body": "Conteúdo do post",
  "userId": 1
}
```

---

## Testando os Endpoints

1. Certifique-se de que o serviço está rodando na porta `8080`.
2. Use ferramentas como:
    - [VSCode REST Client](https://marketplace.visualstudio.com/items?itemName=humao.rest-client)
    - Postman
    - cURL

3. Faça requisições para os endpoints definidos. Por exemplo:

    ```bash
    curl -X GET http://localhost:8080/v1/posts
    ```

4. Valide as respostas retornadas pela API Gateway.

---

## Próximos Passos

- Experimente modificar a configuração do KrakenD para incluir novos endpoints ou aplicar filtros adicionais.
- Explore integrações com serviços reais além do JSONPlaceholder.
- Teste os recursos de cache, CORS e compressão já habilitados na configuração.
