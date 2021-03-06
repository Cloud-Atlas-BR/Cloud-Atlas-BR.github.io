---
layout: post
title: Infraestrutura como Código? Não! Infraestrutura é Código
subtitle: Machine Learning Discovery - S01E02
tags: [aws, mlops, machine, learning, lambda, container, docker, cdk]
comments: true
draft: false
---

Quando iniciamos a discussão referente ao provisionamento de infraestrutura, popularmente chamado de [IAC (Infraestructure as a Code)](https://pt.wikipedia.org/wiki/Infraestrutura_como_C%C3%B3digo), clichês começam na aparecer em nossa conversa. 

Na comunidade de tecnologia, existe uma grande discussão sobre os prós e contras de se utilizar, ora Cloudformation, ora Terraform. Nesse post, nosso objetivo é dar uma visão alternativa quando o assunto tange Infraestrutura como Código.

Inclusive, será que este acrônimo **IAC - Infraestrutura como Código** ainda permanece imutável? Ou será que o provisionamento de nossa infraestrutura não precisa ser ***as a code***, e sim **ser** código de ponta a ponta?

A AWS oferece uma solução para essa questão: o [CDK - Cloud Development Kit](https://aws.amazon.com/cdk/). Um *framework* destinado ao provisionamento de recursos na nuvem a partir de linguagens de programação tradicionais.

Mas, antes de falarmos mais sobre o CDK, vamos entender um pouco da historia do nosso querido [Cloudformation](https://aws.amazon.com/cloudformation/) e como chegamos dele até o CDK.

Let's Bora!

## Um pouco de história e contextualização

O ano é 2011, e aqui nasce o [AWS Cloudformation](https://aws.amazon.com/cloudformation/), e junto com ele o movimento de IAC começa a emergir e ganhar adeptos. 

A oferta deste serviço, para a época, ajudou muito os desenvolvedores a se preocuparem mais com a qualidade do serviço/projeto que estavam desenvolvendo, uma vez que toda a infraestrutura dessa aplicação estaria abstraída por um padrão único e utilizando sintaxes já conhecidas como Json e Yaml.

Com o padrão declarativo do Clouformation, a visualização do que estamos provisionando facilita o entendimento do que realmente está acontecendo. 

```yaml
AWSTemplateFormatVersion: 2010-09-09
Parameters:
  BucketName:
    Type: String
    Description: 'The name of the S3 Bucket'
Resources:
  S3BUCKET:
    Type: 'AWS::S3::Bucket'
    Properties:
      BucketName: !Ref BucketName
```
<p style="text-align: center; margin-top: 0">Exemplo de script Cloudformation para criação de um bucket S3.</p>

Em paralelo, diferente soluções de IAC começam a despontar, como a [Terraform](https://www.terraform.io/), uma das mais fortes concorrentes. Adotando sua própria [DSL](https://en.wikipedia.org/wiki/Domain-specific_language) (Domain-Specific Language), tais soluções não interagem com o Cloudformation, em vez disso, utilizam a API de outros serviços diretamente.

Porém, com o decorrer dos anos, a operacionalização das centenas de linhas que são utilizadas nos arquivos de IAC começa a causar dificuldades, e manutenções recorrentes são inevitáveis. Além de ser observar um cenário explícito de *copy/paste* pela necessidade de recorrente reutilização de templates de Cloudformation e Terraform.

Fica evidente a necessidade de um *approach* diferente.

## CDK, Why Not?

Como dito anteriormente, o CDK quer ser visto como ***"Infra as Real Code"***, essa mensagem carrega uma carga de autonomia designada diretamente ao desenvolvedor que tem como objetivo entregar sua aplicação/projeto em linguagens como TypeScript, JavaScript, Node.Js e Python. 

Aqui, estamos lidando com um conceito chamado [Transpiler](https://en.wikipedia.org/wiki/Source-to-source_compiler), que nada mais é do que obter o código em uma linguagem específica como JavaScript e traduzir para um outro código correspondente.

Desta forma, toda aquela verbosidade do Cloudformation é substituída por uma sintaxe familiar ao desenvolvedor, ao passo que o seu desenvolvimento fica extremamente mais direto e prazeroso. 

Como input, temos código puro em uma linguagem de preferência do desenvolvedor, o qual **ao executar um transpiler, terá como output seu script de Cloudformation**.

Fica claro que um dos benefícios dessa abordagem é que não precisamos dedicar tempo no aprendizado de uma DSL especifica como, por exemplo, o **Terraform**.

## Gerenciamento de Estados 

Como utilizado por outras ferramentas de IAC, o CDK também utiliza de um gerenciador de estados para as stacks de Cloudformation, e tais estados, bem como informações sobre os recursos de infraestrutura provisionados, são armazenados em um bucket S3.

## Proposta

Bom, introdução e contextualização concluídas com sucesso!

Agora, vamos pensar de forma conjunta. 

E se utilizássemos o nosso [primeiro episódio de ML Discovery](https://cloud-atlas-br.github.io/2021-02-20-ml-discovery-s1e1/) para implementar o CDK como camada de abstração de nossa infraestrutura utilizando a linguagem Python?

Para isso, montaremos, via CDK, uma arquitetura conectando nossa função lambda (container) com um API Gateway.

## Talk is cheap, Show me the code

Primeiro, vamos instalar o CDK toolkit.

```sh
npm install -g aws-cdk
```

Como iremos trabalhar com Python, se faz necessária uma instalação específica do módulo **core** e dos módulos **AWS Lambda** e **AWS API Gateway**.

```console
$ pip install aws-cdk.core \
              aws-cdk.aws-lambda \
              aws-cdk.aws-apigatewayv2 \
              aws-cdk.aws-apigatewayv2-integrations
```

Com os pacotes instalados, precisamos inicializar o projeto com a seguinte diretiva.

```console
$ cdk init mldiscovery-app --language python
```

Com o comando executado acima, o CDK irá criar uma estrutura de diretórios e arquivos contendo todas as peças necessárias para realizar o provisionamento da infraestrutura.

```console
(.env) $ tree
.
├── README.md
├── app.py
├── cdk.json
├── mldiscovrey_app
│   ├── __init__.py│   
│   └── mldiscovrey_app_stack.py
├── requirements.txt
├── setup.py
└── tests
    ├── __init__.py
    └── unit
        ├── __init__.py
        └── test_hello_construct.py
|_ source.bat
```

Abaixo, descrevemos a função de cada arquivo criado.

* `README.md` - Arquivo README para o projeto.
* `app.py` - Script "main" para o projeto CDK.
* `cdk.json` - Arquivo de configuração que lista quais componentes devem ser executados durante o deploy da stack.
* `mldiscovery_app` - Diretório com o código Python.
  * `mldiscovery_app_stack.py` - Script Python com o código responsável pelo provisionamento da infraestrutura.
* `requirements.txt` - Arquivo com as dependências Python a serem utilizadas no projeto.
* `tests e unit` - Código referente ao conjunto de testes da aplicação.
* `setup.py` -  Define como será realizado o build do pacote Python, sua descrição e quais dependências são necessárias.
* `source.bat` - Arquivo gerado para estações Windows, contendo a ativação da máquina virtual.

Agora que sabemos exatamente o uso de cada arquivo/diretório, vamos começar a desenvolver o provisionamento do nosso modelo de machine learning, o mesmo tem como arquitetura base a utilização de uma função Lambda junto com containers Docker.

Criaremos, a partir daqui, um diretório chamado `model`, com nosso `Dockerfile`, script `app.py` e o arquivo `requirements.txt` com as dependências do modelo.

Dentro desse diretório, também incluiremos o arquivo `entry.sh`, que é responsável por expor o container através de uma porta especifica.

Assim, o diretório `model` toma a seguinte estrutura.

```bash
$ tree model/
.
├── Dockerfile
├── app.py
├── entry.sh
├── requirements.txt
└── train.py
```

O conteúdo de todos esses arquivos estão descritos detalhadamente em [nosso primeiro episódio da série ML Discovery](https://cloud-atlas-br.github.io/2021-02-20-ml-discovery-s1e1/).


## Deploy CDK - Infra as Real Code

Bom pessoal, chegamos na fase em que necessitamos fazer com que nosso código Python (responsável pelo provisionamento do modelo em um [Lambda Container](https://aws.amazon.com/blogs/aws/new-for-aws-lambda-container-image-support/)) ganhe corpo e nos mostre a que veio.

Primeiro, vamos analisar o arquivo `mldiscovery_app_stack.py`, responsável por realizar o provisionamento da infraestrutura da nossa função lambda.

```python
from aws_cdk import (
    aws_lambda as _lambda,
    aws_apigatewayv2 as api_gw,
    aws_apigatewayv2_integrations as integrations,
    core
)
import os

class MldiscoveryAppStack(core.Stack):

    def __init__(self, scope: core.Construct, construct_id: str, **kwargs) -> None:
        super().__init__(scope, construct_id, **kwargs)

        # Definindo uma função Lambda com Container
        model_folder = os.path.dirname(os.path.realpath(__file__)) + "/../model"
        predictive_lambda = _lambda.DockerImageFunction(self, 'Mldiscovery',
                                                        code=_lambda.DockerImageCode.from_image_asset(model_folder),
                                                        memory_size=4096,
                                                        timeout=core.Duration.seconds(15))
        # Configurando API Gateway que recebera os requests e direcionara para a lambda
        api = api_gw.HttpApi(self, 'MldiscoveryEndpoint',
                             default_integration=integrations.LambdaProxyIntegration(handler=predictive_lambda));

        core.CfnOutput(self, 'HTTP API Url', value=api.url);

```

O código acima inicia com o `import` de todos os [Constructs](https://docs.aws.amazon.com/cdk/latest/guide/constructs.html) necessários para o provisionamento. Para esta aplicação estamos utilizando:

* Lambda
* Api Gateway
* Integrações com Api Gateway

Dentro do método **__init__** escrevemos nosso código de provisionamento. Iniciamos com a atribuição do diretório `model` à variável `model_folder`, onde estão os arquivos referente ao modelo, Dockerfile e dependências.

```python
model_folder = os.path.dirname(os.path.realpath(__file__)) + "/../model"
```

Após essa atribuição, efetivamente criamos um recurso `lambda`. Neste, indicamos que a função Lambda terá como *handler* uma imagem `Docker` através da classe [Construct](https://docs.aws.amazon.com/cdk/latest/guide/constructs.html) `DockerImageFunction`.

Os dois primeiros parâmetro desta classe são: o `self` (o próprio `Construct`) e o nome de nossa função Lambda, que nomeamos de `Mldiscovery`.

```python
predictive_lambda = \
    _lambda.DockerImageFunction(self, 'Mldiscovery', ...)
```

Em seguida, precisamos informar a esta função, qual é o caminho dos arquivos/scripts responsáveis pelo *handler* Docker, que são os arquivos localizados em nosso diretório `model`.

Utilizamos, então, a classe `DockerImageCode`. Nela, passamos a localização da imagem em um Registry [ECR](https://aws.amazon.com/pt/ecr/), ou, então, o caminho de um diretório contendo, obrigatoriamente, um Dockerfile. Dessa forma, utilizamos `from_image_asset` indicando o supracitado diretório `model`

```python
predictive_lambda = \
    _lambda.DockerImageFunction(self, 'Mldiscovery',
        code=_lambda.DockerImageCode.from_image_asset(model_folder), ...)
```

Além destas informações, também dimensionamos a quantidade de memoria RAM utilizada pela função Lambda, bem como o seu timeout de execução.

```python
predictive_lambda = \
    _lambda.DockerImageFunction(self, 'Mldiscovery',
        code=_lambda.DockerImageCode.from_image_asset(model_folder),
        memory_size=4096,
        timeout=core.Duration.seconds(15))
```

Após o provisionamento de nosso Lambda Container, criamos também um [Api Gateway](https://aws.amazon.com/pt/api-gateway/) que fará o papel de camada para consumo do nosso modelo presente na função Lambda.

Passamos o parâmetro `self` novamente, o nome do nosso [API Gateway](https://aws.amazon.com/pt/api-gateway/) e o tipo de integração, que para o nosso caso será a [Lambda Proxy Integration](https://docs.aws.amazon.com/apigateway/latest/developerguide/set-up-lambda-proxy-integrations.html). Para esta integração, informamos a função lambda definida como target anteriormente.

```python
api = \
    api_gw.HttpApi(self, 
        'MldiscoveryEndpoint',
        default_integration=integrations.LambdaProxyIntegration(handler=predictive_lambda))
```

Por último, porém, não menos importante, retornamos a URL do API Gateway.

```python
core.CfnOutput(self, 'HTTP API Url', value=api.url)
```

## CDK Bootstrap

Pessoal, antes de iniciarmos o deploy de nossa aplicação, precisamos executar o comando:

``` console
$ cdk bootstrap
```

A iteração acima se faz necessária, pois as *Stacks* provisionadas pelo CDK precisam que seu estado seja guardado em algum lugar. Este lugar, como comentamos, é o S3. Então, ao rodarmos o comando acima, será iniciada uma Stack de CloudFormation para criação do bucket S3 na região em que o CDK efetuará o deploy.

A Stack que será criada chama-se `CDKToolkit`, conforme imagem abaixo.

<p style="text-align: center"><img src="https://i.imgur.com/0Ex4zj2.jpg"></p>

Realizado o `Bootstrap`, podemos partir para o tão aguardado deploy. Comecemos com um comando para entender qual será o Cloudformation produzido, dessa forma, podemos ver o `transpiler` na prática.

```sh
$ cdk synth
```

Acesse [Aqui](https://gist.github.com/adelmofilho/69793338980a2372f5c75c58e8883505) para visualizar o Clouformation criado.

Agora, vamos efetivamente realizar o deploy de nossa `stack` com o comando:

```console
$ cdk deploy
```

Nesse momento, serão mostrados todos os recursos e componentes que serão criados e também o que não pedimos para criar, mas que são **obrigatórios** para termos sucesso em nossa stack CDK.

<p style="text-align: center"><img src="https://i.imgur.com/5KveoKb.jpg"></p>

A partir do momento que aceitamos o provisionamento destes recursos, será criado uma stack de Cloudformation com o nome do nosso projeto CDK.

<p style="text-align: center"><img src="https://i.imgur.com/J0m0XHi.jpg"></p>

Após a stack obter o status de **COMPLETED**, podemos conferir quais recursos foram criados.

<p style="text-align: center"><img src="https://i.imgur.com/3ZeJpnT.jpg"></p>

Por último, porém (novamente), não menos importante, temos a URL do nosso API Gateway na seção **Output** no console do Cloudformation.

<p style="text-align: center;margin-bottom:0"><img src="https://i.imgur.com/JuYB98H.jpg"></p>
<p style="text-align: center; margin-top:0">Output</p>

<p style="text-align: center;margin-bottom:0"><img src="https://i.imgur.com/9FDAHFA.jpg"></p>
<p style="text-align: center; margin-top:0">Integração do API Gateway com a função Lambda</p>

Assim, podemos chamar nossa API através de um *curl* como apresentado abaixo.

```console
$ curl -X POST "https://rcd9kh404f.execute-api.sa-east-1.amazonaws.com/" -d '{"data": [0,0,0,0]}'

[1]
```

Bom pessoal, o trabalho do CDK acaba por aqui, e o mesmo entrega como prometido a ideia de que **a infraestrutura é codigo**.

## Pensamentos finais

Com esse artigo, procuramos transmitir uma opção alternativa sobre IAC em meio à tantas discussões binárias entre Terraform e Cloudformation.

Aqui, preservamos o desenvolvedor, mantendo-o em contato com sua linguagem de programação habitual, pela qual (e somente por ela) é provisionada sua infraestrutura, reduzindo a verbosidade típica do IAC e a necessidade de aprender uma DSL específica.

Nos vemos no próximo episódio!

<p style="text-align: center"><img src="https://64.media.tumblr.com/7151274239517b2d595ea04b17da4b0b/tumblr_mmzgqw26UY1qafzsyo1_r1_500.gifv"></p>

## Repositório Github

[Repositório Github](https://github.com/Cloud-Atlas-BR/CDK-ML-Discovery-S01E02) com source code deste artigo.

## Referências

* [CDK Developer Guide](https://docs.aws.amazon.com/cdk/latest/guide/home.html)
* [CDK API Documentation](https://docs.aws.amazon.com/cdk/api/latest)
* [CDK Python Documentation](https://docs.aws.amazon.com/cdk/api/latest/python/index.html)
* [CDK Document Gateway](https://docs.aws.amazon.com/pt_br/cdk/?id=docs_gateway)
