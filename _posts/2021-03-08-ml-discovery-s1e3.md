---
layout: post
title: Sua esteira de deploy de Machine Learning no Github
subtitle: Machine Learning Discovery - S01E03
tags: [aws, mlops, machine, learning, lambda, container, github-actions, github]
comments: true
draft: true
---

No [primeiro episódio da série ML Discovery](https://cloud-atlas-br.github.io/2021-02-20-ml-discovery-s1e1/), descobrimos como criar um [Lambda container](https://aws.amazon.com/blogs/aws/new-for-aws-lambda-container-image-support/) de forma manual. **Hoje, vamos dar o próximo passo** e desenvolver uma esteira automatizada para implantar nosso container Lambda.

Na cultura DevOps, essas esteiras são conhecidas como *CI/CD Pipelines*. Na prática, elas são um conjunto de instruções que conecta seu código em um repositório git (Gitlab, Github) até a infraestrutura (servidor), onde seu código será executado. No caminho dessas esteiras, podemos ter etapas de testes, compilação, provisionamento de infraestrutura etc.

Tradicionalmente, montar essas esteiras eram por si só um desafio. Imagine configurar máquinas, segurança, corrigir *bugs* e monitorar performance, mas, o que você queria era apenas realizar o *deploy* de sua aplicação.

**O bom é que isso mudou!**

Hoje, muitas empresas fornecem esteiras de CI/CD como um serviço, bastando a escrita do código que será executado nos estágios da esteira.

<p style="text-align: center"><img src="https://i.imgur.com/Z06j7F3.png"></p>

Neste episódio, vamos criar uma esteira de CI/CD utilizando [Github Actions](https://docs.github.com/en/actions) para realizar o deploy de nosso modelo de Machine Learning.

## Action!

Primeiro, criamos um [repositório no Github](https://github.com/Cloud-Atlas-BR/ML-Discovery-S01E01) com os arquivos e códigos da implementação do Lambda Container para Machine Learning do primeiro episódio. A criação de uma `Action` pode ser feita via interface do Github, ou escrevendo os arquivos da esteira diretamente no repositório.

Seguindo a última opção, vamos clonar o repositório e criar o diretório `.github/workflows` para adicionar nosso script de esteira. Qualquer arquivo da extensão `.yml` configurado corretamente será executado pelo Github Actions quando um *push* for realizado.

```sh
git clone git@github.com:Cloud-Atlas-BR/ML-Discovery-S01E01.git

cd ML-Discovery-S01E01

mkdir -p .github/workflows

touch .github/workflows/deploy.yml

```

No arquivo `deploy.yml` escreveremos o código de nossa esteira de CI/CD. O Github Action espera que este arquivo possua uma estrutura similar à seguinte.

```yaml
name: <Nome da sua esteira CI/CD>

on: <Condicional que deve ser verdadeira para executar a esteira>

jobs: <Conjunto de scripts e divisão lógica da esteira>
```

Abstrato né! Mas, você verá como é simples. 

Em seguida, temos a configuração real da nossa esteira. Dê uma lida e me encontre lá embaixo para detalharmos passo a passo.

```yaml
name: ML Lambda Container Deploy

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-20.04
    steps:      
      - name: Checkout
        uses: actions/checkout@v2

      - name: Configure AWS credentials from your account
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Build and Push image
        run: |
          aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${{ secrets.ACCOUNT_ID }}.dkr.ecr.us-east-1.amazonaws.com
          docker build -t discovery .
          docker tag discovery:latest ${{ secrets.ACCOUNT_ID }}.dkr.ecr.us-east-1.amazonaws.com/discovery:latest
          docker push ${{ secrets.ACCOUNT_ID }}.dkr.ecr.us-east-1.amazonaws.com/discovery:latest

      - name: Create or update lambda function
        run: | 
          aws cloudformation deploy \
            --stack-name discovery-lambda-stack \
            --template-file lambda.yml \
            --capabilities CAPABILITY_NAMED_IAM
```

A chave `name` define o nome da nossa esteira. Fácil!

Em seguida, temos o condicional `on`. Neste configuramos para que a esteira seja executada apenas quando houver um *push* para a *branch* **main**.

É na chave `jobs` que definimos as etapas e códigos da nossa esteira.

Criamos um único estágio chamado `build` (esse nome não é obrigatório) e informamos que todas etapas desse estágio serão executadas em container Ubuntu 20.04.

```yaml
jobs:
  build:
    runs-on: ubuntu-20.04
```

A partir daí definimos os passos desse estágio. 

No passo `Checkout`, importamos a action que [permite que sua esteira acesse os arquivos em seu repositório](https://github.com/actions/checkout).

```yaml
- name: Checkout
  uses: actions/checkout@v2
```

Como usaremos essa esteira para realizar o deploy de nosso Lambda Container, no próximo passo usamos a action `aws-actions/configure-aws-credentials@v1` para obtenção de credenciais da conta AWS.

```yaml
- name: Configure AWS credentials from your account
  uses: aws-actions/configure-aws-credentials@v1
  with:
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    aws-region: us-east-1
```

Observe que aqui estamos utilizando uma variável especial: `{{secrets.}}`. O Github permite a [criação de variáveis de ambiente encriptografadas](https://docs.github.com/en/actions/reference/encrypted-secrets) para evitar a exposição de informações sensíveis. Siga este [tutorial](https://docs.github.com/en/actions/reference/encrypted-secrets#creating-encrypted-secrets-for-a-repository) para criar os segredos `AWS_ACCESS_KEY_ID` e `AWS_SECRET_ACCESS_KEY` a partir das [credenciais de usuário da sua conta AWS](https://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html).

No passo `Build and Push image` é que começa a ação! Primeiramente fazemos o login no serviço de registro de containers (ECR) para posterior *push* da imagem que será criada.

```yaml

- name: Build and Push image
  run: |
    aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${{ secrets.ACCOUNT_ID }}.dkr.ecr.us-east-1.amazonaws.com
    docker build -t discovery .
    docker tag discovery:latest ${{ secrets.ACCOUNT_ID }}.dkr.ecr.us-east-1.amazonaws.com/discovery:latest
    docker push ${{ secrets.ACCOUNT_ID }}.dkr.ecr.us-east-1.amazonaws.com/discovery:latest
```

Se criada com sucesso, a imagem Docker é *taggeada* seguindo o padrão do ECR e enviada ao Registry. Observe que não criamos o repositório da imagem nesse script, isso foi feito manualmente através do comando.

```sh
aws ecr create-repository --repository-name discovery
```

No último estágio da nossa esteira, executamos, emfim, o *script* que criará o lambda container na sua conta da AWS. Para isso, utilizamos o serviço [cloudformation](https://aws.amazon.com/cloudformation/) que permite o provisionamento da [infraestrutura como código](https://en.wikipedia.org/wiki/Infrastructure_as_code).

```yaml
- name: Create or update lambda function
  run: |
    aws cloudformation deploy \
     --stack-name discovery-lambda-stack \
     --template-file lambda.yml
     --capabilities CAPABILITY_NAMED_IAM
```

Observe que aqui, apresentamos um novo arquivo: `lambda.yml`. Neste, escrevemos e definimos todos os recursos que devem ser instanciados (Função Lambda, IAM Role) na AWS.

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Lambda Container
Resources:
  Function:
    Type: AWS::Lambda::Function
    Properties:
      FunctionName: discovery
      Role: !GetAtt Role.Arn
      PackageType: Image
      Code:
        ImageUri: 977053370764.dkr.ecr.us-east-1.amazonaws.com/discovery:latest
      Timeout: 5
  Role:
    Type: AWS::IAM::Role
    Properties:
      RoleName: discovery-lambda-role
      AssumeRolePolicyDocument:
        Version: 2012-10-17
        Statement:
          - Effect: Allow
            Principal:
              Service:
                - lambda.amazonaws.com
            Action:
              - 'sts:AssumeRole'
      Path: /
      Policies:
        - PolicyName: discovery-lambda-policy
          PolicyDocument:
            Version: 2012-10-17
            Statement:
              - Effect: Allow
                Action: 
                  - "cloudwatch:*"
                  - "ec2:DescribeSecurityGroups"
                  - "ec2:DescribeSubnets"
                  - "ec2:DescribeVpcs"
                  - "events:*"
                  - "iam:GetPolicy"
                  - "iam:GetPolicyVersion"
                  - "iam:GetRole"
                  - "iam:GetRolePolicy"
                  - "iam:ListAttachedRolePolicies"
                  - "iam:ListRolePolicies"
                  - "iam:ListRoles"
                  - "iam:PassRole"
                  - "kms:ListAliases"
                  - "lambda:*"
                  - "logs:*"
                  - "tag:GetResources"
                Resource: '*'
      MaxSessionDuration: 3600 
```
<p style="text-align: center; margin-top:0">lambda.yml</p>

Com todos os arquivos prontos, podemos *commitar* e fazer o *push* das nossas alterações. Lembre-se de fazê-las na branch *main* para que a esteira seja executada.

```sh
git add -A
git commit -m "Update deploy.yml"
git push origin main
```

## Acompanhando a esteira



Nos vemos no próximo episódio!