# Documentação do Projeto: Controle de Rotas

## Visão Geral

O projeto **Controle de Rotas** tem como objetivo criar dois microserviços baseados em AWS Lambda: `pathwriter` e `pathreader`, que estão armazenados em repositórios Git diferentes. Cada Lambda terá uma função específica e será publicada na AWS usando **Terraform**.

A infraestrutura do projeto inclui o provisionamento de **API Gateway** e **CloudFront** para que os usuários possam acessar as funções Lambda de maneira pública e segura. A comunicação entre os serviços será feita por meio de requisições HTTP.

## Estrutura do Projeto

O projeto está dividido em dois repositórios principais:

1. **Repositório Pathwriter**: Contém a Lambda `pathwriter`.
2. **Repositório Pathreader**: Contém a Lambda `pathreader`.

Ambos os repositórios seguem uma estrutura similar, contendo os diretórios para o código da Lambda e para a configuração de infraestrutura com Terraform.

### Estrutura de Diretórios

Abaixo está a estrutura de diretórios de cada repositório (Pathwriter e Pathreader):

```
controle-de-rotas/
├── app/                     # Diretório raiz para o código das Lambdas
│   ├── src/                 # Código principal da Lambda
│   │   └── utils/           # Funções utilitárias usadas pela Lambda
│   ├── tests/               # Testes unitários para a Lambda
│   └── lambda_function.py   # Função Lambda principal (handler)
├── infra/                   # Diretório de infraestrutura (Terraform)
│   ├── main.tf              # Definição principal da infraestrutura com Terraform
│   ├── variables.tf         # Definição de variáveis usadas pelo Terraform
│   ├── outputs.tf           # Saídas (outputs) do Terraform
│   ├── lambda.tf            # Configuração de AWS Lambda com Terraform
│   ├── apigateway.tf        # Configuração do API Gateway
│   └── cloudfront.tf        # Configuração do CloudFront
└── README.md                # Documentação do projeto
```

### Explicação de Componentes

#### **app/**

Este diretório contém o código-fonte das funções Lambda:

- **src/**: Contém o código principal da Lambda, incluindo funções auxiliares ou de apoio na pasta `utils`.
- **tests/**: Contém os testes unitários para garantir que as Lambdas funcionem conforme o esperado.
- **lambda_function.py**: Arquivo principal onde está a função Lambda (`handler`) que será executada quando a Lambda for invocada.

#### **infra/**

Este diretório contém a infraestrutura que será provisionada com **Terraform**. Ele inclui a definição de recursos da AWS, como Lambda, API Gateway e CloudFront.

- **main.tf**: Arquivo principal do Terraform que chama os demais módulos e recursos.
- **lambda.tf**: Define a função Lambda, incluindo o código-fonte, configurações de timeout, políticas de IAM, etc.
- **apigateway.tf**: Define o API Gateway que irá expor a Lambda via HTTP para que o usuário final possa acessá-la.
- **cloudfront.tf**: Configura o CloudFront para servir o conteúdo e adicionar caching, controle de acesso e segurança.
- **variables.tf**: Define as variáveis de entrada necessárias para configurar o ambiente de infraestrutura.
- **outputs.tf**: Define as saídas da execução do Terraform, como o endpoint do API Gateway e a URL do CloudFront.

## Descrição dos Serviços

### 1. **Lambda Pathwriter**

- **Descrição**: A função Lambda `pathwriter` é responsável por escrever informações de rotas (paths) em um banco de dados ou serviço de armazenamento. Ela recebe dados via requisição HTTP e persiste essas informações.
- **Código**: Está contido no diretório `app/src` do repositório `pathwriter`.
- **Infraestrutura**: A infraestrutura para essa Lambda será provisionada com Terraform, e o código será implantado na AWS Lambda. O API Gateway será configurado para permitir que os usuários enviem requisições HTTP para esta Lambda.

### 2. **Lambda Pathreader**

- **Descrição**: A função Lambda `pathreader` é responsável por ler as informações de rotas previamente armazenadas e retornar essas informações para o usuário. Ela é acessada via requisição HTTP através de um endpoint exposto no API Gateway.
- **Código**: Está contido no diretório `app/src` do repositório `pathreader`.
- **Infraestrutura**: Semelhante ao `pathwriter`, a função `pathreader` será publicada na AWS usando Terraform e terá um endpoint HTTP configurado no API Gateway.

## Infraestrutura AWS com Terraform

### 1. **Lambda**

Cada repositório terá sua própria função Lambda gerenciada pelo Terraform. O código será publicado diretamente nos serviços Lambda da AWS a partir dos respectivos repositórios.

- **Definição de Lambda**: O código da Lambda será implantado diretamente no serviço AWS Lambda, que será configurado com as permissões necessárias para execução.
- **Gerenciamento de versão**: As Lambdas serão versionadas para garantir que atualizações de código sejam rastreadas e fáceis de reverter.

### 2. **API Gateway**

O **API Gateway** será configurado para permitir que usuários externos façam chamadas HTTP para invocar as funções Lambda. Cada Lambda terá seu próprio endpoint no API Gateway:

- **Pathwriter Endpoint**: Um endpoint público para enviar dados para o `pathwriter`.
- **Pathreader Endpoint**: Um endpoint público para obter dados do `pathreader`.

### 3. **CloudFront**

O **CloudFront** será configurado como uma camada de cache e proteção adicional, permitindo a distribuição de conteúdo de maneira segura e eficiente. Ele atuará como um proxy entre os usuários e o API Gateway:

- **Cache**: O CloudFront fornecerá caching para melhorar o desempenho e reduzir latência.
- **Segurança**: Com o uso do CloudFront, é possível adicionar camadas de proteção como HTTPS e controle de acesso.

## Procedimento de Deploy

### 1. **Configuração Inicial**

1. Clone os repositórios `pathwriter` e `pathreader`:
   ```bash
   git clone https://github.com/.../controle-de-rotas-pathwriter.git
   git clone https://github.com/.../controle-de-rotas-pathreader.git
   ```

2. Instale as dependências necessárias para o Terraform e configure as credenciais AWS no ambiente local:
   ```bash
   terraform init
   ```

### 2. **Deploy da Infraestrutura com Terraform**

1. Navegue até o diretório `infra/` de cada repositório e aplique o Terraform para provisionar a infraestrutura na AWS:

   ```bash
   terraform apply
   ```

   Isso provisionará as Lambdas, API Gateway e CloudFront, e exibirá as URLs para acesso ao final do processo.

2. **Variáveis de Configuração**:
   - As variáveis do Terraform podem ser definidas no arquivo `variables.tf` ou passadas como parâmetros durante a execução.
   - Exemplo de execução com variáveis:
     ```bash
     terraform apply -var="environment=production"
     ```

### 3. **Deploy do Código da Lambda**

O código das Lambdas será automaticamente publicado na AWS Lambda durante o processo de Terraform. Para atualizar o código, basta realizar as seguintes etapas:

1. Faça as modificações no código.
2. Realize o build e upload do código via Terraform ou AWS CLI.
3. Aplique o Terraform novamente para atualizar a função Lambda:
   ```bash
   terraform apply
   ```

### 4. **Testes e Validação**

Após o deploy, você pode realizar testes usando as URLs geradas pelo API Gateway e CloudFront.

- Acesse o endpoint de escrita (`pathwriter`) para enviar dados.
- Acesse o endpoint de leitura (`pathreader`) para verificar se os dados foram persistidos corretamente.

### Ferramentas Utilizadas

- **AWS Lambda**: Para hospedagem das funções serverless.
- **API Gateway**: Para expor os endpoints HTTP.
- **CloudFront**: Para caching e proteção adicional.
- **Terraform**: Para provisionamento da infraestrutura na AWS.
