---
layout: post
title: Infraestrutura como Codigo? Não! Infraestrutura é Código.
subtitle: Machine Learning Discovery - S01E02
tags: [aws, mlops, machine, learning, lambda, container, docker, cdk]
comments: true
draft: true
---

Quando iniciamos a discussão referente ao provisionamento de infraestrutura, popularmente chamado de [IAC (Infraestructure as a Code)](https://pt.wikipedia.org/wiki/Infraestrutura_como_C%C3%B3digo), clichês começam na aparecer em nossa conversa. 

Na comunidade, existe uma grande discussão sobre os prós e contras de se utilizar, ora Cloudformation, ora Terraform. Nesse post, nosso objetivo é dar uma visão alternativa quando o assunto tange Infraestrutura como Código.

Inclusive, será que este acrônimo **IAC - Infraestrutura como Código** ainda permanece imutável? Ou será que o provisionamento de nossa infraestrutura não precisa ser ***as a code***, e sim ser código de ponta a ponta ?

Antes de falarmos mais sobre o objetivo deste post, que é propriamente o [CDK - Cloud Development Kit](https://aws.amazon.com/cdk/), vamos falar um pouco sobre a historia do nosso querido e guerreiro [Cloudformation](https://aws.amazon.com/cloudformation/) e como chegamos dele até o CDK.

Let's Bora!

## Um pouco de história e contextualização

O ano é 2011, e aqui nasce o [AWS Cloudformation](https://aws.amazon.com/cloudformation/), e junto com ele o movimento de IAC começa a emergir e ganhar adeptos. 

A oferta deste serviço, para a época, ajudou muito os desenvolvedores a se preocuparem muito mais com a qualidade do serviço/projeto que estavam desenvolvendo, uma vez que toda a infraestrutura dessa aplicação estaria abstraída por um padrão único e utilizando sintaxes já conhecidas como Json e Yaml.

Com o padrão declarativo do Clouformation, a visualização do que estamos provisionando facilita o entendimento do que realmente está acontecendo. Porém, com o decorrer dos anos, as centenas de linhas que são utilizadas nos seus extensos arquivos começaram a causar dificuldades, e manutenções recorrentes apareciam como possíveis dores de cabeça intermináveis.

Um cenário explicitamente de *copy/paste* começa a ocorrer. Os desenvolvedores tem a necessidade de recorrente reutilização de código bem como a sua generalização. Agora, começamos a ter problemas com nosso fiel e guerreiro Cloudformation. 

Em paralelo, soluções de IAC começam a despontar, dentre elas, uma das mais fortes concorrentes (e conhecidas) chamada [Terraform](https://www.terraform.io/). Adotando sua própria [**DSL**](https://en.wikipedia.org/wiki/Domain-specific_language), tais soluções não interagem com o Cloudformation, em vez disso, utilizam a API de outros serviços diretamente.

## CDK, Why Not ?

Como dito anteriormente, o CDK quer ser visto como ***"Infra as Real Code"***, essa mensagem carrega uma carga de autonomia designada diretamente ao desenvolvedor que tem como objetivo entregar sua aplicação/projeto em linguagens como TypeScript, JavaScript, Node.Js e Python. 

Aqui, estamos lidando com um conceito chamado **Transpiler**, que nada mais é do que obter o código em uma linguagem específica como JavaScript e traduzir para um outro código correspondente.

Então, toda aquela verbosidade do Cloudformation é substituída por uma sintaxe familiar ao desenvolvedor, ao passo que o seu desenvolvimento fica extremamente mais direto e prazeroso. Como input, temos código puro em uma linguagem de preferência do desenvolvedor que ao executar um transpiler terá como output seu script de Cloudformation.

Fica claro que um dos benefícios dessa abordagem é que não precisamos dedicar tempo no aprendizado de uma DSL especifica como, por exemplo, o **Terraform**.

## Gerenciamento de Estados 

Como já utilizado por outras ferramentas de IAC, o CDK também utiliza-se de um gerenciador de estados para sa stacks de Cloudformation, e tais estados, bem como informações sobre os recursos de infra provisionados, são armazenados em um bucket S3.

Bom, introdução e contextualização concluídas com sucesso!

## Proposta

Agora, vamos pensar de forma conjunta. 

E se utilizássemos o nosso [primeiro episódio de ML Discovery](https://cloud-atlas-br.github.io/2021-02-20-ml-discovery-s1e1/) para implementar o CDK como camada de abstração de nossa infraestrutura utilizando a linguagem Python?

## Talk is cheap, Show me the code

Primeiro, vamos instalar o CDK.

```console
$ pip install aws-cdk.core aws-cdk.aws-lambda
```
Como podemos ver, o CDK parte de uma instalação **core**, junto com essa instalação precisamos informar com quais módulos do CDK iremos trabalhar, que para esta caso será o **AWS Lambda**.

Com os pacotes instalados, precisamos inicializar o projeto com a seguinte diretiva.

```console
$ cdk init mldiscovery-app --language python
```
Com o comando executado acima, o CDK irá criar uma estrutura de diretórios e arquivos contendo todas as peças necessárias para realizar o provisionamento da infraestrutura.

Vamos então conhecer o que temos dentro do diretório da aplicação.

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

Agora vamos entender qual é a função de cada arquivo acima, os mesmos fazem parte do diretório em que inicializamos o projeto **CDK**.

* `README.md` - arquivo README para o projeto
* `app.py` - script "main" para o projeto CDK
* `cdk.json` - arquivo de configuração que lista quais componentes devem ser executados durante o deploy da stack
* `mldiscovery_app` - Diretório com o código Python
  * `mldiscovery_app_stack.py` - script Python com o código responsável pelo provisionamento da infraestrutura
* `requirements.txt` - Arquivo com as dependências Python a serem utilizadas no projeto
* `tests e unit` - Código referente ao conjunto de testes da aplicação
* `setup.py` -  Define como será realizado o build do pacote Python, sua descrição e quais dependências são necessárias.
* `source.bat` - arquivo gerado para estações Windows, contendo a ativação da máquina virtual.

Agora que sabemos exatamente o uso de cada arquivo/diretório, vamos começar a desenvolver o provisionamento do nosso modelo de machine learning, o mesmo tem como arquitetura base a utilização de uma função Lambda junto com containers Docker.

Criaremos, a partir daqui, um diretório chamado `model`, com nosso **Dockerfile**, script **app.py** e também nosso arquivo **requirements.txt** 

Dentro desse diretório, também incluiremos o arquivo **entry.sh**, que é responsável por expor o container através de uma porta especifica.

Nosso foco será no deploy de nossa stack utilizando o tão esperado **CDK**. Todos os arquivos de nosso projeto **lambda container** estão descritos detalhadamente em [nosso primeiro episódio da série](https://cloud-atlas-br.github.io/2021-02-20-ml-discovery-s1e1/).


## Deploy CDK - Infra as Real Code

Bom pessoal, chegamos na fase em que necessitamos fazer com que nosso código Python (responsável pelo provisionamento do modelo em um [Lambda Container](https://aws.amazon.com/blogs/aws/new-for-aws-lambda-container-image-support/)) ganhe corpo e nos mostre a que veio.

Primeiro, vamos analisar o arquivo `mldiscovery_app_stack.py` responsável por realizar o provisionamento da infraestrutura da nossa função lambda através do código Python 

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

Dentro deste método **__init__** escrevemos nosso código de provisionamento. Iniciamos com a atribuição do diretório `model` à variável `model_folder`, onde estão os arquivos referente ao modelo, Dockerfile e dependências.

Após essa atribuição, efetivamente criamos um recurso `lambda` indicando que a mesma terá como base uma imagem `Docker`, através da classe [Construct](https://docs.aws.amazon.com/cdk/latest/guide/constructs.html) `DockerImageFunction`, tal classe encarrega-se de criar um função lambda e define que o handler da mesma trata-se de uma imagem Docker.

Agora, precisamos informar a esta função que acabara de ser criada, qual é o caminho dos arquivos/scripts responsáveis pelo *handler* Docker, que são os arquivos localizados em nosso diretório `model`.

Utilizamos, então, a classe `DockerImageCode`. Esta classe pode ser utilizada passando a localização da imagem em um Registry [ECR](https://aws.amazon.com/pt/ecr/), ou, então, diretamente de um diretório contendo, obrigatoriamente, um Dockerfile. Dessa forma, utilizamos `from_image_asset` indicando o supracitado diretório `model`

Além destas informações, também dimensionamos a quantidade de memoria RAM utilizada pela função Lambda, bem como o seu timeout de execução.

Sobram os dois primeiros parâmetros, que são `self` (o próprio `Construct`) e o Id de nossa função Lambda, que nomeamos de  `Mldiscovery`.

Após o provisionamento de nosso Lambda Container, criamos também um [Api Gateway](https://aws.amazon.com/pt/api-gateway/) que fará o papel de camada para consumo do nosso modelo presente na função Lambda.

Novamente passamos o parâmetro `self`, o nome do nosso [API Gateway](https://aws.amazon.com/pt/api-gateway/) e por ultimo qual integração será realizada, que para o nosso caso o tipo de integração que utilizaremos será [Lambda Proxy Integration](https://docs.aws.amazon.com/apigateway/latest/developerguide/set-up-lambda-proxy-integrations.html) passando a função lambda que criamos anteriormente como target de nossa integração.

E por último, porém não menos importante, devolvemos o output da URL do API Gateway para facilitar o nosso consumo no fim do nosso provisionamento.

O nosso diretório `model` ficou com a seguinte estrutura.

```bash
$ tree model/
.
├── Dockerfile
├── app.py
├── entry.sh
├── requirements.txt
└── train.py
```

## CDK Bootstrap

Pessoal, antes de iniciarmos o deploy de nossa aplicação via CDK CLI precisamos executar o comando.

``` console
$ cdk bootstrap
```

A iteração acima se faz necessária, pois as *Stacks* provisionadas pelo CDK precisam que seu estado seja guardado em algum lugar, este lugar que o CDK julga como mantenedor destes dados é o S3. Então, ao rodarmos o comando acima uma Stack de CloudFormation será criada na `Region` que o CDK efetuará o deploy.

A Stack que será criada chama-se `CDKToolkit`, conforme imagem abaixo.

<p style="text-align: center"><img src="https://i.imgur.com/0Ex4zj2.jpg"></p>

Podemos observar também que uma Bucket Policy e um Bucket de *staging* foi criado para guardar o estado das Stacks provisionadas pelo CDK.

Realizado o `Bootstrap`, podemos partir para o tão aguardado deploy. Comecemos com um comando para entender qual será o Cloudformation de saída, dessa forma, podemos ver o `transpiler` na prática.

```console
rjekste@desk$ cdk synth
```
Agora, vamos efetivamente realizar o deploy de nossa `stack`.

```console
rjekste@desk$ cdk deploy
```

Antes de efetivamente realizarmos o deploy, nos será mostrado todos recursos e componentes que serão criados e também o que não pedimos para criar, mas que são **obrigatórios** para termos sucesso em nossa stack CDK.

<p style="text-align: center"><img src="https://i.imgur.com/5KveoKb.jpg"></p>

A partir do momento que aceitamos o provisionamento destes recursos, imediatamente será criado um stack de Cloudformation com o nome do nosso projeto CDK.

<p style="text-align: center"><img src="https://i.imgur.com/J0m0XHi.jpg"></p>

Após a `stack` obter o status de **COMPLETED** podemos conferir quais recursos foram criados.

<p style="text-align: center"><img src="https://i.imgur.com/3ZeJpnT.jpg"></p>

Por último, porém (novamente), não menos importante temos o **Output** com o a URL do nosso API Gateway.

<p style="text-align: center"><img src="https://i.imgur.com/JuYB98H.jpg"></p>
<p style="text-align: center; margin-top:0">Output</p>

<p style="text-align: center"><img src="https://i.imgur.com/9FDAHFA.jpg"></p>
<p style="text-align: center; margin-top:0">API Gateway com a integração junto ao Lambda</p>

Bom pessoal, o trabalho do CDK acaba por aqui, e o mesmo entrega como prometido a ideia de que **a infraestrutura é codigo**.

Podemos chamar nossa API através de um *curl* como apresentado abaixo.

```console
$ curl -X POST "https://rcd9kh404f.execute-api.sa-east-1.amazonaws.com/" -d '{"data": [0,0,0,0]}'

[1]
```

## Pensamentos finais

Então, chegamos ao fim deste episódio.

Espero que tenhamos conseguido transmitir uma opção alternativa em meio à tantas discussões binárias entre Terraform e Cloudformation.

Aqui, preservamos o desenvolvedor sempre em contato com seu habitat natural, o ` código`, e é a partir dele (e somente dele) que provisionamos a nossa infraestrutura, reduzindo a verbosidade e a necessidade de aprender uma DSL específica.

Nos vemos no próximo episódio!

<p style="text-align: center"><img src="https://64.media.tumblr.com/7151274239517b2d595ea04b17da4b0b/tumblr_mmzgqw26UY1qafzsyo1_r1_500.gifv"></p>

## Repositório Github

[Repositório Github](https://github.com/Cloud-Atlas-BR/CDK-ML-Discovery-S01E02) com source code deste artigo

## Referências

* [CDK Developer Guide](https://docs.aws.amazon.com/cdk/latest/guide/home.html)
* [CDK API Documentation](https://docs.aws.amazon.com/cdk/api/latest)
* [CDK Python Documentation](https://docs.aws.amazon.com/cdk/api/latest/python/index.html)
* [CDK Document Gateway](https://docs.aws.amazon.com/pt_br/cdk/?id=docs_gateway)