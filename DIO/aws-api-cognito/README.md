# Segurança em APIs na AWS com Amazon Cognito
Este repositório contém o código fonte do Live Coding da DIO no dia 17/11/2021. Nesse projeto foi desenvolvido uma API com autenticação via Amazon Cognito.

## Serviços AWS Utilizados
  - Amazon Cognito;
  - Amazon DynamoDB;
  - Amazon API Gateway;
  - AWS Lambda.

## Etapas do Desenvolvimento
### Criando uma API REST no Amazon API Gateway
  - `API Gateway Dashboard` -> `Create API` -> `REST API` -> `Build`.
  - `Protocol - REST` -> `Create new API` -> `API name [dio_live_api]` -> `Endpoint Type - Regional` -> `Create API`.
  - `Resources` -> `Actions` -> `Create Resource` -> `Resource Name [Items]` -> `Create Resource`.

### Criando uma Tabela no Amazon DynamoDB
  - DynamoDB Dashboard -> `Tables` -> `Create table` -> `Table name [Items]` -> `Partition key [id]` -> `Create table`.

### Criando uma Função para Inserir Item no AWS Lambda
  - Lambda Dashboard -> `Create function` -> `Name [put_item_function]` -> `Create function`.
  - Inserir código da função `put_item_function.js` disponível na pasta `/src` -> `Deploy`.
  - `Configuration` -> `Execution role` -> Abrir a Role no console do IAM.
  - `IAM` -> `Roles` -> Role criada no passo anterior -> `Permissions` -> `Add inline policy`.
  - `Service - DynamoDB` -> `Manual actions` -> `Add actions` -> `putItem`.
  - `Resources` -> `Add ARN` -> Selecionar o ARN da tabela criada no DynamoDB -> `Add`.
  - Review policy -> `Name [lambda_dynamodb_putItem_policy]` -> `Create policy`.

### Integrando o API Gateway com o Lambda Backend
  - API Gateway Dashboard -> Selecionar a API criada -> `Resources` -> Selecionar o resource criado -> `Action` -> `Create method - POST`.
  - `Integration type` -> `Lambda function` -> `Use Lambda Proxy Integration` -> `Lambda function` -> Selecionar a função Lambda criada -> `Save`.
  - `Actions` -> `Deploy API` -> `Deployment Stage` -> `New Stage [dev]` -> `Deploy`.

### Realizando Testes de Inserção de Dados sem Autenticação utilizando o Postman
  - `Add Request` -> `Method POST` -> Copiar o endpoint gerado no API Gateway.
  - `Body` -> `Raw` -> `JSON` -> Adicionar o seguinte body.

  ```bash
  {
    "id": "003",
    "price": 600
  }
  ```
  - `Send`.

### Criando a Autenticação com o Amazon Cognito
  - Cognito Dashboard -> `Manage User Pools` -> `Create a User Pool` -> `Pool name [TestPool]`.
  - `How do you want your end users to sign in? - Email address or phone number` -> `Next Step`.
  - `What password strength do you want to require?`.
  - `Do you want to enable Multi-Factor Authentication (MFA)? Off` -> `Next Step`.
  - `Do you want to customize your email verification messages?` -> `Verification type`: `Link` -> `Next Step`.
  - `Which app clients will have access to this user pool?` -> `App client name [TestClient]` -> `Create App Client` -> `Next Step`.
  - `Create Pool`.

  - `App integration` -> `App client settings` -> `Enabled Identity Providers` - `Cognito User Pool`.
  - `Callback URL(s) [https://example.com/logout]`.
  - `OAuth 2.0` -> `Allowed OAuth Flows`, `Authorization code grant` e `Implicit grant`.
  - `Allowed OAuth Scopes`:	`email`	e `openid`.
  - `Save Changes`

  - `Domain name` -> `Domain prefix [diolive]` -> `Save`.

### Criando um Autorizador do Amazon Cognito para uma API REST no Amazon API Gateway
  - API Gateway Dashboard -> Selecionar a API criada -> `Authorizers` -> `Create New Authorizer`.
  - `Name [CognitoAuth]` -> `Type - Cognito` -> `Cognito User Pool [pool criada anteriormente]` -> `Token Source [Authorization]`.

  - `Resources` -> Selecionar o resource criado -> Selecionar o método criado -> `Method Request` -> `Authorization` - Selecionar o autorizador criado.

### Realizando Testes de Inserção de Dados com Autenticação utilizando o Postman
  - `Add request` -> `Authorization`.
  - `Type - OAuth 2.0`.
  - `Callback URL [https://example.com/logout]`.
  - `Auth URL [https://diolive.auth.sa-east-1.amazoncognito.com/login]`.
  - `Client ID` - Obter o Client ID do Cognito em App clients.
  - `Scope [email - openid]`.
  - `Client Authentication [Send client credentials in body]`.
  - `Get New Acces Token`.
  - Copiar o token gerado.

  - Selecionar a request para inserir item criada -> `Authorization` -> `Type - Bearer Token` -> Inserir o token copiado.
  - `Send`.