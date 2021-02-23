---
layout: post
title: Machine Learning Discovery - S01E02
subtitle: IAC (Infraestructure as a Code ), Não ! Infraestrutra é Código.
tags: [aws, mlops, machine, learning, lambda, container, docker, cdk]
comments: true
draft: true
---

Quando iniciamos a discussão referente ao provisionamento de infraestrutura, popularmente chamado de IAC (Infraestrcture as a Code) clichês começam a aparecer em nossa conversa. Na comunidade existe uma grande discussão, entre, prós e contras de se utilizar, ora Cloudformation, ora Terraform. Aqui nesse post o objetivo é darmos uma visão paralela e alternativa quando o assunto tange Infraestrutura como Código.

Inclusive, sera que este acrônimo **IAC - Infraestrutura como Código** ainda permanece imutável ? Ou, será que o provisionamento de nossa infraestrutura nao precisar ser ***as a code*** e sim escrita em forma código de ponta a ponta ?

Antes de falarmos mais sobre o objetivo deste post que é propriamente o **CDK - Cloud Development Kit**, vamos falar um pouco sobre a historia do nosso querido e guerreiro Cloudformation e como chegamos até o CDK.

Let's Bora !

## Um pouco de história e contextualização

o ano é 2011, e aqui nasce o Cloudformation e junto com ele o movimento de IAC começa a emergir e ganhar adeptos. A oferta deste serviço para a epoca ajudou muito os desenvolvedores a se preocuparem muito mais com a qualidade do serviço/projeto que estavam desenvolvendo, uma vez que toda a infraestrtura dessa aplicação estivesse abstraida por um padrao unico e utilizando sintaxes e padroes já conhecidos como Json e YAML.

É claro que com o padrão declarativo a visualização e o entendimento do que estamos provisionando facilita o entendimento do que realmente esta acontecendo e como. Porém, com o decorrer dos anos as milhares/centenas de linhas que são utilizadas nos extensos arquivos YAML's começaram a causar dificuldades e manutenções recorrentes apereciam como possiveis dores de cabeça interminaveis.

E entao, um cenario explicitamente de copy/paste começa a ocorrer. Os desenvolvedores tem a necessidade de recorrente reutilização de código bem como a sua generalização. Agora começamos a ter problemas com nosso fiel e guerreiro Cloudformation. 

Em paralelo, soluções de IAC começam a despontar, dentre elas, uma das mais fortes concorrentes( e conhecidas ) chamada Terraform, adotando sua própria **DSL**, tais soluções nao interagem com o Cloudformation, em vez disso, utilizam a API diretamente.

## CDK, Why Not ?




- Integração nativa com dezenas de serviços (S3, SQS, SNS, Kinesis, Api Gateway)
- [Gratuidade de até 1 milhão de requisições por mês](https://aws.amazon.com/lambda/pricing/)
- Precificação do uso em incrementos de milissegundo

Lambda é uma escolha ideal para pequenos *workloads* agendados ou orientados a eventos. 
Pequenos, pois as limitações das funções lambdas estão na execução da sua aplicação em até 6 CPUs, consumindo até 10 GB de memória em no máximo 15 minutos.

Existem, por outro lado, limitações menos comentadas.

O fato de trabalharmos apenas no nível da aplicação, nos impede de (1) instalar bibliotecas de sistema operacional e (2) garantir a reprodutibilidade dos resultados, pois não estaremos desenvolvendo no mesmo ambiente em que a função lambda é executada.

<p style="text-align: center"><img src="https://i.imgur.com/CZ6TqF5.jpg"></p>

Docker! [Assim como para metade de todos os problemas de desenvolvimento](https://i.redd.it/iv0oiaz7aqe41.jpg), o uso de *containers* docker é uma solução adequada para as limitações de reprodutibilidade: desenvolver no mesmo ambiente em que o código será implantado.

O projeto lambCI disponibiliza [imagens Docker](https://hub.docker.com/r/lambci/lambda/) para esse propósito. Inclusive sendo a [solução recomendada pela AWS para criação de Lambda Layers](https://aws.amazon.com/premiumsupport/knowledge-center/lambda-layer-simulated-docker/) - o equivalente de virtualenvs do python para as funções lambda.

Mas, covenhamos..., o uso de imagens que mimetizam o ambiente Lambda não resolve integralmente as questões levantadas. Ainda não conseguimos customizar o container docker no qual o código da função Lambda é executado, ou migrar este container para uma infraestrutura *on-premisse*. Não temos controle sobre esse container.

Ou melhor, não tínhamos!

## Lambda Containers

Em dezembro de 2020, a [AWS anuncia o suporte a containers às funções Lambda](https://aws.amazon.com/blogs/aws/new-for-aws-lambda-container-image-support/).

Isso significa que o serviço AWS Lambda passa a ser capaz de executar containers definidos pelo desenvolvedor. Passando de um [*function-as-a-service*](https://en.wikipedia.org/wiki/Function_as_a_service) para algo como um *entrypoint-as-a-service*.

As limitações de tempo (15 minutos) e memória (10 GB) se mantém, mas, diferente dos 250 MB de espaço disponíveis para a execução das funções, é possível executar imagens docker de até 10 GB.

## Implantando um Lambda Container

Nesse tutorial, vamos explorar o deploy de um modelo de **machine learning** em Python utilizando *lambda containers*.

Nosso modelo será uma *random forest* capaz de predizer uma classe binária proveniente de uma base de dados artificial. Como comentamos, para garantir a reprodutibilidade do modelo, iremos treiná-lo dentro da imagem docker que será executada pelo Lambda.

### Treinando o modelo

Nossa imagem personalizada será baseada na imagem [`python:3.8.8-slim-buster`](https://hub.docker.com/_/python). Treinaremos o modelo no *build* da imagem Docker com o [seguinte *script* Python](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.RandomForestClassifier.html).

```python
import joblib
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import make_classification

X, y = make_classification(n_samples=1000, n_features=4,
                           n_informative=2, n_redundant=0,   
                           random_state=0, shuffle=False)

clf = RandomForestClassifier(max_depth=2, random_state=0)

clf.fit(X, y)

joblib.dump(clf, "modelo.joblib")
```
<p style="text-align: center; margin-top: 0px">train.py</p>

As seguintes bibliotecas python devem ser instaladas para a correta execução do *script* de treinamento.

```python
joblib==1.0.1
numpy==1.20.1
scikit-learn==0.24.1
scipy==1.6.1
sklearn==0.0
threadpoolctl==2.1.0
```
<p style="text-align: center; margin-top: 0px">requirements.txt</p>

Em posse dos arquivos *train.py* e *requirements.txt* podemos começar a escrever nossa imagem Docker.


```Dockerfile
# Imagem Base
FROM python:3.8.8-slim-buster

# Criação do diretório padrão do lambda container
ARG FUNCTION_DIR="/function/"
WORKDIR ${FUNCTION_DIR}

# Treinamento do modelo

COPY requirements.txt ${FUNCTION_DIR}/requirements.txt
COPY train.py ${FUNCTION_DIR}/train.py

RUN pip3 install \
        --target ${FUNCTION_DIR} \
        -r requirements.txt

RUN python3 train.py
```

<p style="text-align: center; margin-top: 0px">Dockerfile para treinamento do modelo</p>

Em seguida, vamos executar e salvar a imagem Docker em nossa máquina com o comando.

```sh
docker build -t modelo .
```

### Criando o container lambda

Para que um container possa ser executado pelo AWS Lambda é necessário que o mesmo possua uma interface capaz de se comunicar com o serviço Lambda. No caso do Python, essa interface é provida pela biblioteca `awslambdaric`.

Ao executarmos, por exemplo, o comando `python3 -m awslambdaric app.handler` como *entrypoint* da nossa imagem, o serviço Lambda passa a entender que qualquer requisição deve ser processada pela função `handler` contida no arquivo `app.py`.

Para fins desse tutorial, nosso arquivo `app.py` terá a seguinte estrutura, na qual o modelo é carregado em tempo de execução e retorna para o usuário sua predição com base nos dados enviados.

```python
import joblib
import sklearn
import json

def handler(event, context):

    dados = list(event["data"])
    modelo = joblib.load("modelo.joblib")
    resposta = int(modelo.predict([dados]))

    return {

        'statusCode': 200,
        'body': resposta
    }
```
<p style="text-align: center; margin-top: 0px">app.py</p>

Neste ponto, já poderíamos escrever a imagem Docker para deploy na AWS. Mas, por quê não testar o funcionamento desse Lambda localmente?

Isso pode ser feito com a adição, na imagem, de um [emulador de interface](https://docs.aws.amazon.com/lambda/latest/dg/runtimes-images.html#runtimes-api-client) (`aws-lambda-rie`). Para isso, escreveremos um arquivo `entry.sh` que será executado como *entrypoint* de nossa imagem com o seguinte conteúdo.

```sh
#!/bin/sh
if [ -z "${AWS_LAMBDA_RUNTIME_API}" ]; then
  exec /usr/local/bin/aws-lambda-rie /usr/local/bin/python -m awslambdaric $1
else
  exec /usr/local/bin/python -m awslambdaric $1

fi
```

<p style="text-align: center; margin-top: 0px">entry.sh</p>

A variável de ambiente `AWS_LAMBDA_RUNTIME_API` permite identificar se estamos executando o container localmente, ou através do AWS Lambda. Em ambos os casos estaremos executando o comando comentado mais acima, mas no caso da execução ser local, encapsulamos a chamada dele através do `aws-lambda-rie`.

Com todas as cartas agora na mesa, chegamos na seguinte imagem Docker.

```Dockerfile
# Imagem Base
FROM modelo:latest

ARG FUNCTION_DIR="/function/"

# Instalação do awslambdaric
RUN apt-get update && \
  apt-get install -y \
  g++ \
  make \
  cmake \
  unzip \
  wget \
  libcurl4-openssl-dev

RUN pip3 install --target ${FUNCTION_DIR} awslambdaric

# Obtenção do aws-lambda-rie
RUN wget https://github.com/aws/aws-lambda-runtime-interface-emulator/releases/latest/download/aws-lambda-rie -P /usr/local/bin/
RUN chmod +x /usr/local/bin/aws-lambda-rie

# Cópia dos arquivos de execução
COPY app.py app.py
RUN chmod +x app.py

COPY entry.sh entry.sh
RUN chmod +x entry.sh

ENTRYPOINT [ "./entry.sh" ]
CMD [ "app.handler" ]
```

<p style="text-align: center; margin-top: 0px">Dockerfile para implantação do modelo</p>

Observe que na última instrução do Dockerfile, informamos para o arquivo `entry.sh` o valor do argumento `$1`, que deve corresponder à função a ser executada pelo lambda.

### Testando seu container lambda localmente

Em posse do Dockerfile anterior, execute os seguintes comando para montar e executar seu container:

```sh
docker build -t deploy .

docker run -p 9000:8080 deploy:latest
```

Você verá um *output* semelhante ao final da execução.

<p style="text-align: center"><img src="https://i.imgur.com/Iz7dTmG.png"></p>

Com isso, temos nosso container lambda esperando requisições no endereço: http://localhost:9000/2015-03-31/functions/function/invocations.

Experimente enviar uma requisição com o seguinte comando (em um outro terminal).

```sh
curl -X POST "http://localhost:9000/2015-03-31/functions/function/invocations" -d '{"data": [0,0,0,0]}'
```

A resposta da chamada terá output equivalente a este.

<p style="text-align: center"><img src="https://i.imgur.com/ydI6oj7.png"></p>

Já os *logs* do container retornarão uma saída semelhante a esta.

<p style="text-align: center"><img src="https://i.imgur.com/ViUGPGb.png"></p>

<p style="text-align: center"><img src="https://i.imgur.com/e1coPXt.jpg"></p>

### Implantando seu container na Cloud

Para fecharmos esse tutorial, vamos subir nosso container para o AWS Lambda. Inicialmente, criamos um *registry* na AWS e subimos nossa imagem para ele. No código abaixo, substitua a variável `<ACCOUNT ID>` pelos 12 dígitos do Id da sua conta AWS.

```sh
aws ecr create-repository --repository-name deploy

aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <ACCOUNT ID>.dkr.ecr.us-east-1.amazonaws.com

docker tag deploy:latest <ACCOUNT ID>.dkr.ecr.us-east-1.amazonaws.com/deploy:latest

docker push <ACCOUNT ID>.dkr.ecr.us-east-1.amazonaws.com/deploy:latest
```

Em seguida, abra sua conta AWS, e no serviço lambda, crie uma nova função. Selecione a opção `Container Image`, insira um nome para a função, e selecione a imagem Docker que você acabou de dar *push*.

<p style="text-align: center"><img src="https://i.imgur.com/vU99jGm.png"></p>

Uma vez criada, a função podemos fazer o teste da função lambda pelo console. Crie um novo teste e execute-o.

<p style="text-align: center"><img src="https://i.imgur.com/7jWtjYF.png"></p>

A primeira execução de um container lambda pode ser bastante lenta. No caso da nossa imagem, foram aproximadamente 9 segundos, o que gerou a necessidade de aumentar o *timeout* da função (*default* de 3 segundos). Contudo, a partir da segunda execução, o tempo de processamento reduziu para 0,5 segundos. Este é um padrão conhecido como [cold-start](https://aws.amazon.com/blogs/compute/new-for-aws-lambda-predictable-start-up-times-with-provisioned-concurrency/), típico das funções lambda.

<p style="text-align: center"><img src="https://i.imgur.com/xtiB8H9.png"></p>

## Pensamentos finais

Lambda *containers* surgem como uma interessante alternativa para trabalhar com aplicações conteinerizadas por conta do modelo de *pricing* das funções lambdas. A redução de custos em comparação com soluções como ECS ou EC2 é expressiva, mas o cold-start desses *containers* deve ser levado em conta para sistemas em que a baixa latência é um requisito.

Nos vemos no próximo episódio!

<p style="text-align: center"><img src="https://64.media.tumblr.com/7151274239517b2d595ea04b17da4b0b/tumblr_mmzgqw26UY1qafzsyo1_r1_500.gifv"></p>