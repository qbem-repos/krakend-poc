Beleza, Camilo! Vamos criar um README mais descontraído para o seu projeto "KrakenD with Keycloak".

---

# PoC de KrakenD com Keycloak

## Visão Geral

Essa é uma Prova de Conceito (PoC) para uma integração entre KrakenD e Keycloak. A ideia é mostrar como usar o KrakenD como API Gateway com autenticação e autorização via Keycloak.

## Estrutura do Projeto

- **compose.yml**: Configuração do Docker Compose pra subir os serviços.
- **krakend.json**: Configuração do KrakenD API Gateway.
- **realm-export.json**: Configuração do Keycloak para autenticação.
- **requests.http**: Requisições HTTP pra testar a API Gateway.

## Tecnologias Usadas

- KrakenD
- Keycloak
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

Configura os serviços Docker pra rodar o KrakenD e Keycloak:

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

  keycloak:
    image: jboss/keycloak:latest
    environment:
      DB_VENDOR: H2
      KEYCLOAK_USER: admin
      KEYCLOAK_PASSWORD: admin
    ports:
      - "9090:8080"
    volumes:
      - ./realm-export.json:/tmp/realm-export.json
    command: ["-Dkeycloak.import=/tmp/realm-export.json"]
```

- **krakend**: Serviço usando a imagem do KrakenD.
- **keycloak**: Serviço usando a imagem do Keycloak.
- **ports**: Mapeia a porta 8080 do host pra porta 8080 do KrakenD e a porta 9090 do host pra porta 8080 do Keycloak.
- **volumes**: Monta os arquivos `krakend.json` e `realm-export.json` dentro dos containers.
- **command**: Comandos pra iniciar os serviços com suas respectivas configurações.

### krakend.json

Configura a API Gateway com autenticação via Keycloak:

```json
{
  "version": 3,
  "name": "KrakenD with Keycloak",
  "port": 8080,
  "cache_ttl": "3600s",
  "timeout": "3000ms",
  "extra_config": {
    "github.com/devopsfaith/krakend-cors": {
      "allow_origins": ["*"],
      "allow_methods": ["GET", "POST", "PUT", "DELETE"],
      "expose_headers": ["Content-Length", "Content-Type"],
      "max_age": "12h",
      "allow_credentials": true
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
      ],
      "extra_config": {
        "auth/authorization": {
          "name": "keycloak",
          "alg": "RS256",
          "jwk_url": "http://localhost:9090/auth/realms/krakend/protocol/openid-connect/certs",
          "roles_key": "roles",
          "roles": ["user"],
          "audience": ["krakend-client"],
          "cache": true,
          "cache_duration": "300s"
        }
      }
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
      ],
      "extra_config": {
        "auth/authorization": {
          "name": "keycloak",
          "alg": "RS256",
          "jwk_url": "http://localhost:9090/auth/realms/krakend/protocol/openid-connect/certs",
          "roles_key": "roles",
          "roles": ["user"],
          "audience": ["krakend-client"],
          "cache": true,
          "cache_duration": "300s"
        }
      }
    }
  ]
}
```

- **endpoints**: Define os endpoints da API Gateway:
  - **/users/{user}**: Roteia pra `https://jsonplaceholder.typicode.com/users/{user}`, excluindo `id`, e usa autenticação via Keycloak.
  - **/posts/{user}**: Roteia pra `https://jsonplaceholder.typicode.com/posts/{user}`, excluindo `id` e `userId`, e usa autenticação via Keycloak.

### realm-export.json

Configuração do Keycloak para autenticação:

```json
{
  "realm": "krakend",
  "enabled": true,
  "users": [
    {
      "username": "duck",
      "enabled": true,
      "credentials": [
        {
          "type": "password",
          "value": "quaquaqua"
        }
      ],
      "realmRoles": ["user"]
    }
  ],
  "clients": [
    {
      "clientId": "krakend-client",
      "enabled": true,
      "protocol": "openid-connect",
      "redirectUris": [
        "*"
      ],
      "publicClient": false,
      "secret": "krakend",
      "directAccessGrantsEnabled": true,
      "bearerOnly": false,
      "standardFlowEnabled": true,
      "implicitFlowEnabled": false,
      "serviceAccountsEnabled": true,
      "defaultRoles": ["user"]
    }
  ],
  "roles": {
    "realm": [
      {
        "name": "user",
        "description": "User"
      },
      {
        "name": "post",
        "description": "Post"
      }
    ]
  }
}
```

- **realm**: Configura o realm "krakend".
- **users**: Define usuários com suas credenciais e roles.
- **clients**: Define clientes com suas configurações de acesso.
- **roles**: Define as roles disponíveis no realm.

### requests.http

Requisições pra testar a API Gateway:

```http
### Requisição para obter dados de um usuário (autenticado)
GET http://localhost:8080/users/1
Authorization: Bearer <token>

### Requisição para obter posts de um usuário (autenticado)
GET http://localhost:8080/posts/1
Authorization: Bearer <token>
```

- **GET /users/{user}**: Testa o endpoint `/users/{user}` autenticado.
- **GET /posts/{user}**: Testa o endpoint `/posts/{user}` autenticado.

## Usando o Projeto

### Rodando o Projeto

1. Certifique-se de que os serviços estão rodando:

    ```bash
    docker-compose ps
    ```

2. A API Gateway estará disponível na porta 8080 e o Keycloak na porta 9090.

3. Use as requisições no `requests.http` pra testar a API Gateway. Pode usar VSCode com a extensão "REST Client" ou Postman.

    - Obtenha um token de acesso do Keycloak e substitua `<token>` nas requisições.

    - Testar `/users/{user}`:

        ```http
        GET http://localhost:8080/users/1
        Authorization: Bearer <token>
        ```

    - Testar `/posts/{user}`:

        ```http
        GET http://localhost:8080/posts/1
        Authorization: Bearer <token>
        ```