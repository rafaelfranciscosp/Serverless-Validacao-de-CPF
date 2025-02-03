# Microsserviço Serverless - Validação de CPF

Este projeto é um microsserviço serverless para validação de CPF. Ele utiliza o [Serverless Framework](https://www.serverless.com/) para a implantação na Azure e a biblioteca [cpf-cnpj-validator](https://www.npmjs.com/package/cpf-cnpj-validator) para a validação do CPF.

## Tecnologias

- **Node.js** - Ambiente de execução JavaScript (versão 18.x)
- **Serverless Framework** - Framework para desenvolvimento de microsserviços serverless
- **Azure Functions** - Serviço serverless da Microsoft Azure
- **Jest** - Framework de testes para JavaScript
- **CPF-CNPJ-Validator** - Biblioteca JavaScript para validação de CPF e CNPJ

## Pré-requisitos

Antes de rodar o projeto, verifique se você possui os seguintes pré-requisitos:

- **Node.js 18.x ou superior**
- **Serverless Framework** instalado
- **Azure CLI** configurada
- **Conta na Azure**

## Instalação

1. Clone o repositório:

    ```bash
    git clone https://github.com/seu-usuario/cpf-validation-serverless.git
    cd cpf-validation-serverless
    ```

2. Instale as dependências:

    ```bash
    npm install
    ```

3. Configure suas credenciais da Azure:

    ```bash
    az login
    ```

## Executando localmente

Para executar a função localmente com o Serverless Framework, use o plugin **serverless-offline**:

```bash
serverless offline start
Acesse a API localmente em:

bash
Copiar
http://localhost:3000/validate-cpf?cpf=12345678909
Testes
Para rodar os testes, utilize o Jest:

bash
Copiar
npm test
Comandos para Inicializar
Instalar as dependências:

Para instalar as dependências do projeto, execute o seguinte comando:

bash
Copiar
npm install
Rodar o projeto localmente (com o serverless-offline):

Para rodar a função localmente utilizando o plugin serverless-offline, execute o seguinte comando:

bash
Copiar
serverless offline start
Após iniciar o servidor, acesse a API localmente em:

bash
Copiar
http://localhost:3000/validate-cpf?cpf=12345678909
Realizar o deploy para Azure:

Para realizar o deploy do microsserviço para a Azure, execute o seguinte comando:

bash

serverless deploy
Após o deploy, a URL do endpoint será fornecida no terminal. Você poderá acessar a função serverless pela URL gerada.

Testar a função utilizando o endpoint gerado após o deploy:

Após o deploy, você pode testar a função utilizando a URL gerada no terminal. Basta acessar a URL no formato:

https://<url-gerada>/validate-cpf?cpf=12345678909
Certifique-se de substituir <url-gerada> pela URL real fornecida após o deploy.

## Estrutura do Projeto

cpf-validation-serverless/
├── handler.js
├── package.json
├── serverless.yml
├── .gitignore
├── README.md
└── tests/
    └── cpfValidation.test.js

## Descrição dos Arquivos
1. package.json
Este arquivo contém as dependências e scripts do projeto. Ele inclui as bibliotecas necessárias para o funcionamento do microsserviço (como o cpf-cnpj-validator e azure-functions-core-tools) e configura o script para rodar os testes com o Jest.

json

{
  "name": "cpf-validation-serverless",
  "version": "1.0.0",
  "description": "Microsserviço serverless para validação de CPF",
  "main": "handler.js",
  "scripts": {
    "test": "jest"
  },
  "dependencies": {
    "cpf-cnpj-validator": "^2.0.1",
    "azure-functions-core-tools": "^3.0.3"
  },
  "devDependencies": {
    "jest": "^29.0.3",
    "serverless": "^3.25.0"
  },
  "author": "Seu Nome",
  "license": "MIT"
}

2. serverless.yml
Este arquivo configura o Serverless Framework para utilizar o Azure como provedor e define a função que será executada, além de configurar os eventos que acionam a função.

provider: Define o Azure como o provedor, com o ambiente de execução nodejs18.x.
functions: Define a função validateCpf que será acionada por requisições HTTP do tipo GET no caminho /validate-cpf.
plugins: Define o uso do plugin serverless-offline para permitir a execução local.
Exemplo de conteúdo:

yaml

service: cpf-validation-serverless

frameworkVersion: '3'

provider:
  name: azure
  runtime: nodejs18.x
  region: eastus
  stage: dev

functions:
  validateCpf:
    handler: handler.validateCpf
    events:
      - http:
          path: validate-cpf
          method: get
          cors: true

plugins:
  - serverless-offline

3. handler.js
Este arquivo contém a lógica de validação do CPF, usando a biblioteca cpf-cnpj-validator. A função validateCpf recebe um evento HTTP e um contexto da Azure, valida o CPF fornecido e retorna um status indicando se o CPF é válido ou não.

Validação de CPF: A função cpf.isValid() é usada para verificar se o CPF fornecido é válido.
Resposta HTTP: A função retorna um objeto com o status HTTP 200 se o CPF for fornecido corretamente ou 400 caso contrário.
Exemplo de conteúdo:

javascript

const { cpf } = require('cpf-cnpj-validator');

module.exports.validateCpf = async (context, req) => {
  const { cpf: cpfInput } = req.query;

  if (!cpfInput) {
    return context.res = {
      status: 400,
      body: { message: 'CPF não fornecido.' }
    };
  }

  const isValid = cpf.isValid(cpfInput);

  return context.res = {
    status: 200,
    body: { cpf: cpfInput, isValid: isValid }
  };
};

4. .gitignore
Este arquivo define os arquivos e diretórios a serem ignorados pelo Git. O principal objetivo é evitar que arquivos de dependências, configurações locais e pastas de build sejam enviados para o repositório.

Exemplo de conteúdo:

bash

node_modules/
.serverless/
.env

5. tests/cpfValidation.test.js
Este arquivo contém os testes automatizados usando o framework Jest. Os testes verificam se a função de validação do CPF está funcionando corretamente.

Testes de CPF válido: Testa um CPF válido e verifica se o retorno da função indica que ele é válido.
Testes de CPF inválido: Testa um CPF inválido e verifica se o retorno da função indica que ele é inválido.
Testes de ausência de CPF: Testa o caso em que nenhum CPF é fornecido e verifica se o retorno é um erro.
Exemplo de conteúdo:

javascript

const { validateCpf } = require('../handler');

describe('Validação de CPF', () => {
  it('deve retornar que o CPF é válido', async () => {
    const event = { query: { cpf: '123.456.789-09' } };
    const context = { res: {} };
    await validateCpf(context, event);
    expect(context.res.status).toBe(200);
    expect(context.res.body.isValid).toBe(true);
  });

  it('deve retornar que o CPF é inválido', async () => {
    const event = { query: { cpf: '123.456.789-00' } };
    const context = { res: {} };
    await validateCpf(context, event);
    expect(context.res.status).toBe(200);
    expect(context.res.body.isValid).toBe(false);
  });

  it('deve retornar erro se o CPF não for fornecido', async () => {
    const event = { query: {} };
    const context = { res: {} };
    await validateCpf(context, event);
    expect(context.res.status).toBe(400);
    expect(context.res.body.message).toBe('CPF não fornecido.');
  });
});



### Explicação Adicional

- **Função Serverless**: O código foi estruturado para ser facilmente executado na nuvem, sem a necessidade de servidores físicos, utilizando a Azure Functions. O Serverless Framework facilita o deploy e a configuração dos recursos necessários para a execução.
- **Testes**: A inclusão de testes com Jest garante que a validação do CPF funcione corretamente antes de ser implantada.

