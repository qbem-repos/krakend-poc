Claro, Camilo! Aqui está a versão atualizada do README com as informações corretas:

---

# krakend-poc

Este repositório contém provas de conceito (PoCs) que utilizam o KrakenD como API Gateway e Back-end For Front-end (BFF), integrando com o Keycloak para autenticação e autorização.

## Objetivo

Demonstrar a capacidade do KrakenD de atuar como um API Gateway e BFF, utilizando o Keycloak para gerenciar autenticação e autorização dos usuários.

## Estrutura do Projeto

O repositório está organizado em três pastas principais, cada uma focada em um aspecto específico da PoC:

1. **back-for-front**: Contém a configuração da PoC para o Back-end For Front-end (BFF) utilizando KrakenD.
2. **api-gateway**: Contém a configuração da PoC para o API Gateway utilizando KrakenD.
3. **keycloak-integration**: Contém a configuração da PoC para a integração com Keycloak.

### Arquivos Importantes

- **compose.yml**: Configurações para orquestração dos serviços utilizando Docker Compose.
- **krakend.json**: Configurações do KrakenD, incluindo os endpoints e integração com backend.
- **realm-export.json**: Configurações do Keycloak para autenticação e autorização.
- **requests.http**: Conjunto de requisições HTTP para testar os endpoints configurados no KrakenD.
- **README.md**: Este arquivo, contendo a descrição do projeto e instruções de uso.

## Configuração e Execução

### Back-for-Front

Configurado com os seguintes endpoints:

- `/blog/{user}`: Obtém informações do usuário e suas postagens.

### API Gateway

Configurado com os seguintes endpoints:

- `/users/{user}`: Obtém informações do usuário.
- `/posts/{user}`: Obtém as postagens do usuário.

### Keycloak Integration

Utilizado para gerenciar autenticação e autorização dos usuários que acessam os endpoints configurados no KrakenD.

### Passos para Configuração

1. **Clone o repositório**:

    ```bash
    git clone <URL do repositório>
    cd krakend-poc
    ```

2. **Suba os serviços com Docker Compose**:

    ```bash
    docker-compose up -d
    ```

3. **Configurar Keycloak**:
    - O Keycloak estará disponível na porta 9090.
    - Acesse o Keycloak em `http://localhost:9090/auth` e faça o login com as credenciais de administrador configuradas no `compose.yml` (usuário: admin, senha: admin).
    - Verifique se o realm "krakend" foi importado corretamente a partir do arquivo `realm-export.json`.

4. **Testar os Endpoints**:
    - Utilize as requisições no arquivo `requests.http` para testar os endpoints configurados no KrakenD.
    - Obtenha um token de acesso do Keycloak e substitua `<token>` nas requisições.

## Exemplos de Requisições

### Requisição para obter dados de um usuário (autenticado)

```http
GET http://localhost:8080/users/1
Authorization: Bearer <token>
```

### Requisição para obter posts de um usuário (autenticado)

```http
GET http://localhost:8080/posts/1
Authorization: Bearer <token>
```

### Requisição para obter dados combinados de um usuário

```http
GET http://localhost:8080/blog/1
```

## Notas Finais

Esta PoC é um ponto de partida para entender como integrar o KrakenD com o Keycloak e outros serviços backend. Modificações e extensões podem ser feitas conforme necessário para atender às necessidades específicas do seu projeto.