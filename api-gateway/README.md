Beleza, Camilo! Vamos deixar o README mais completo incluindo como rodar o projeto.

---

# PoC de API Gateway

## Visão Geral

Essa é uma Prova de Conceito (PoC) para uma API Gateway usando KrakenD. A ideia é mostrar como a API Gateway funciona, roteando e manipulando requisições para diferentes endpoints backend.

## Estrutura do Projeto

- **compose.yml**: Configuração do Docker Compose pra subir os serviços.
- **krakend.json**: Configuração do KrakenD API Gateway.
- **requests.http**: Requisições HTTP pra testar a API Gateway.

## Tecnologias Usadas

- KrakenD
- Docker
- [jsonplaceholder.typicode.com](https://jsonplaceholder.typicode.com) (API de exemplo)

## Configurando o Ambiente

### Pré-requisitos

- Docker e Docker Compose instalados.

### Instalação

1. Clone o repositório:

    ```bash
    git clone https://github.com/seu-usuario/nome-do-repositorio.git
    ```

2. Entre no diretório do projeto:

    ```bash
    cd nome-do-repositorio
    ```

3. Suba os serviços com Docker Compose:

    ```bash
    docker-compose up -d
    ```

## O Que Cada Arquivo Faz

### compose.yml

Configura os serviços Docker pra rodar o KrakenD:

```yaml
version: '3.8'

services:
  krakend:
    image: devopsfaith/krakend:latest
    ports:
      - "8080:8080"
    volumes:
      - ./krakend.json:/etc/krakend/krakend.json
    command: [
      "-c",
      "/etc/krakend/krakend.json",
      "-d",
      "DEBUG"
    ]
```

- **krakend**: Serviço usando a imagem do KrakenD.
- **ports**: Mapeia a porta 8080 do host pra porta 8080 do container.
- **volumes**: Monta o arquivo `krakend.json` dentro do container.
- **command**: Comando pra iniciar o KrakenD com debug.

### krakend.json

Configura a API Gateway:

```json
{
    "version": 3,
    "name": "API Gateway",
    "port": 8080,
    "timeout": "3000ms",
    "extra_config": {
        "router": {
            "disable_gzip": false
        }
    },
    "endpoints": [
        {
            "endpoint": "/users/{user}",
            "method": "GET",
            "output_encoding": "json",
            "backend": [
                {
                    "host": [
                        "https://jsonplaceholder.typicode.com"
                    ],
                    "encoding": "json",
                    "url_pattern": "/users/{user}",
                    "group": "user",
                    "deny": [
                        "id"
                    ]
                }
            ]
        },
        {
            "endpoint": "/posts/{user}",
            "method": "GET",
            "output_encoding": "json",
            "backend": [
                {
                    "host": [
                        "https://jsonplaceholder.typicode.com"
                    ],
                    "encoding": "json",
                    "url_pattern": "/posts/{user}",
                    "deny": [
                        "id",
                        "userId"
                    ],
                    "group": "last_post"
                }
            ]
        }
    ]
}
```

- **endpoints**: Define os endpoints da API Gateway:
  - **/users/{user}**: Roteia pra `https://jsonplaceholder.typicode.com/users/{user}`, excluindo `id`.
  - **/posts/{user}**: Roteia pra `https://jsonplaceholder.typicode.com/posts/{user}`, excluindo `id` e `userId`.

### requests.http

Requisições pra testar a API Gateway:

```http
### Requisição para obter dados de um usuário
GET http://localhost:8080/users/1

### Requisição para obter posts de um usuário
GET http://localhost:8080/posts/1
```

- **GET /users/{user}**: Testa o endpoint `/users/{user}`.
- **GET /posts/{user}**: Testa o endpoint `/posts/{user}`.

## Usando o Projeto

### Rodando o Projeto

1. Certifique-se de que os serviços estão rodando:

    ```bash
    docker-compose ps
    ```

2. A API Gateway estará disponível na porta 8080.

3. Use as requisições no `requests.http` pra testar a API Gateway. Pode usar VSCode com a extensão "REST Client" ou Postman.

    - Testar `/users/{user}`:

        ```http
        GET http://localhost:8080/users/1
        ```

    - Testar `/posts/{user}`:

        ```http
        GET http://localhost:8080/posts/1
        ```