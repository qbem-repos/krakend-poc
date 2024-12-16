# KrakenD POC - Integração com Keycloak

Este projeto demonstra como configurar o KrakenD para autenticação e autorização usando Keycloak. Ele integra o Keycloak como Identity Provider para proteger endpoints expostos pelo KrakenD.

---

## Visão Geral

A integração utiliza o Keycloak para autenticar e autorizar requisições feitas ao KrakenD. Nesta PoC:

- **Keycloak** gerencia usuários, clientes e papéis.
- **KrakenD** valida tokens JWT emitidos pelo Keycloak para controlar o acesso aos endpoints.

---

## Estrutura do Projeto

- **compose.yml**: Configuração do Docker Compose para subir o ambiente com Keycloak e KrakenD.
- **realm-export.json**: Configuração do Realm do Keycloak com usuários, clientes e papéis.
- **krakend.json**: Configuração do KrakenD com integração ao Keycloak.
- **requests.http**: Exemplos de requisições HTTP para testar os endpoints protegidos.

---

## Configurando o Ambiente

### Pré-requisitos

- Docker e Docker Compose instalados.

### Instalação

1. Clone o repositório:

    ```bash
    git clone https://github.com/seu-usuario/krakend-keycloak-poc.git
    ```

2. Navegue para o diretório do projeto:

    ```bash
    cd krakend-keycloak-poc
    ```

3. Suba os serviços com Docker Compose:

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

Configura os serviços Docker para Keycloak e KrakenD:

```yaml
services:
  keycloak:
    image: quay.io/keycloak/keycloak:16.1.1
    environment:
      KEYCLOAK_USER: admin
      KEYCLOAK_PASSWORD: admin
      DB_VENDOR: h2
      KEYCLOAK_IMPORT: /opt/jboss/keycloak/realms/realm-export.json
    ports:
      - "9090:8080"
    volumes:
      - ./realm-export.json:/opt/jboss/keycloak/realms/realm-export.json

  krakend:
    image: devopsfaith/krakend
    ports:
      - "8080:8080"
    volumes:
      - ./krakend.json:/etc/krakend/krakend.json
    depends_on:
      - keycloak
```

- **Keycloak**: Fornece autenticação e autorização.
- **KrakenD**: API Gateway configurado para validar tokens JWT emitidos pelo Keycloak.

---

### realm-export.json

Arquivo de configuração do Realm no Keycloak:

```json
{
  "realm": "krakend",
  "clients": [
    {
      "clientId": "krakend-client",
      "secret": "krakend"
    }
  ],
  "roles": {
    "realm": [
      {
        "name": "user"
      }
    ]
  }
}
```

- **realm**: Nome do Realm configurado.
- **clients**: Configura o cliente usado pelo KrakenD.
- **roles**: Define papéis para controle de acesso.

---

### krakend.json

Configuração do KrakenD com integração ao Keycloak:

```json
{
  "extra_config": {
    "auth/authorization": {
      "name": "keycloak",
      "jwk_url": "http://localhost:9090/auth/realms/krakend/protocol/openid-connect/certs",
      "roles": ["user"],
      "audience": ["krakend-client"]
    }
  }
}
```

- **jwk_url**: URL para os certificados do Keycloak.
- **roles**: Papéis exigidos para acessar os endpoints.

---

### requests.http

Arquivo para testar os endpoints:

```http
### Token de autenticação
POST http://localhost:9090/auth/realms/krakend/protocol/openid-connect/token
Content-Type: application/x-www-form-urlencoded

client_id=krakend-client
&client_secret=krakend
&grant_type=password
&username=duck
&password=quaquaqua

### Endpoint protegido
GET http://localhost:8080/users/1
Authorization: Bearer <seu_token_aqui>
```

---

## Testando o Ambiente

1. Obtenha um token de autenticação via endpoint do Keycloak.
2. Use o token para acessar os endpoints protegidos no KrakenD.
3. Valide o comportamento com diferentes papéis ou usuários.

---

## Próximos Passos

- Adicione novos papéis e restrições nos endpoints.
- Explore outras integrações avançadas, como autenticação por client credentials.
- Substitua o Keycloak configurado com H2 para um banco de dados persistente, como PostgreSQL.