---
layout: post
title: MLflow e Sagemaker - Juntos somos mais fortes.
subtitle:  Machine Learning Discovery - S01E04
tags: [aws, mlops, cloudformation, iaac, iac]
comments: true
draft: true
---

{: .box-note}
**Este é um texto em desenvolvimento**: Ainda estamos escrevendo e/ou revisando seu conteúdo. Até o dia de sua publicação, ele não estará listado na página inicial do blog.

Fala Galera!

No episódio de hoje, vamos falar sobre um assunto interessantíssimo, aprenderemos como utilizar em conjunto o [MLflow](https://mlflow.org/) e o [AWS Sagemaker](https://aws.amazon.com/pt/sagemaker/) para catalogação e ***Deploy*** dos nossos Modelos de Machine Learning.

Aqui, o protagonismo fica a cargo do [MLflow](https://mlflow.org/) cuja a responsabilidade é servir de registry central para nossos modelos, enquanto que o [AWS Sagemaker](https://aws.amazon.com/pt/sagemaker/) ficará responsável por expor um Endpoint de consumo para o modelo. Este serviço é conhecido como [AWS Sagemaker Endpoint](https://docs.aws.amazon.com/sagemaker/latest/dg/deploy-model.html).


O objetivo aqui é apresentar de forma didática e simples a parceria entre  [MLflow](https://mlflow.org/) e [AWS Sagemaker](https://aws.amazon.com/pt/sagemaker/). Mostraremos também os frutos dessa parceria através de uma implementação real de um modelo de Machine Learning utilizando esses dois serviços.

Let's Bora !

## Conhecendo o MLFlow

O **MLflow** é definido como uma plataforma **open source** para gerenciamento do ciclo de vida de um Modelo de Machine Learning.

<p style="text-align: center"><img src="https://i.imgur.com/uzFL2bN.png"></p>

O **MLflow** concentra suas funcionalidades em quatro principais componentes para englobar as etapas deste ciclo.

* [MLflow Tracking](https://mlflow.org/docs/latest/tracking.html) - Monitoração de Modelos
* [MLflow Projects](https://mlflow.org/docs/latest/projects.html) - Garantia de reprodutibilidade e idempotência do Modelo
* [MLflow Models](https://mlflow.org/docs/latest/models.html) - Produtização e Deploy
* [MLflow Model Registry](https://mlflow.org/docs/latest/model-registry.html) - Repositório e Catálogo centralizado dos Modelos

Para o Episódio de hoje iremos demonstrar a funcionalidade do quarto componente, o [MLflow Model Registry](https://mlflow.org/docs/latest/model-registry.html) 

A partir deste componente será possível centralizarmos nosso registry/catalogo de modelos, bem como gerencia-los através de API's, UI, etc.

Com nosso catalog/registry disponibilizado pelo [MLflow](https://mlflow.org/), partimos agora para a produtização dos nossos modelos previamente catalogados, para esta tarefa utilizaremos o [AWS Sagemaker](https://aws.amazon.com/pt/sagemaker/)

## Conhecendo o AWS Sagemaker Endpoint

Com o nosso modelo devidamente catalogado e armazenado em nosso registry, iniciamos a caminhada de disponibilizar o mesmo para consumo.

Para isso, utilizaremos o [AWS Sagemaker Endpoint](https://docs.aws.amazon.com/sagemaker/latest/dg/deploy-model.html). 

Antes de continuarmos pessoal, vale ressaltarmos que o [AWS Sagemaker](https://aws.amazon.com/pt/sagemaker/) possui uma stack de serviços muito extensa, que por si só teria conteúdo para uma série inteira, todinha pra ele.

De uma forma simples e direta o [AWS Sagemaker Endpoint](https://aws.amazon.com/pt/sagemaker/) nos disponibiliza uma lista de instâncias genreciadas cujo o propósito será expor um endpoint com rotas de API especificas para consumo.

* `/invocation` - Rota que recebe o payload e transmite o mesmo ao modelo.
* `/ping` - Rota de health check.

Tais rotas terão como destino um container Docker, sendo que dentro deste container Docker o código do nosso modelo estará presente.

## Arquitetura 

Para termos um registro visual do que construiremos nesse artigo, apresento-lhes o desenho de arquitetura proposto.

<p style="text-align: center"><img src="https://i.imgur.com/l3iMrUJ.jpg"></p>

## Let's Get Started

Como explicado anteriormente utilizaremos o [AWS Fargate](https://aws.amazon.com/pt/fargate), o mesmo tem como objetivo fornecer nosso [MLflow](https://https://mlflow.org/)  instanciado através de um container [Docker](https://www.docker.com/) em uma infraestrutura totalmente serverless.

Abaixo, temos o Dockerfile referente ao **MLflow server**:

```Dockerfile
FROM python:3.8.0

RUN pip install \
    mlflow \
    pymysql \
    boto3

WORKDIR /mlflow

EXPOSE 5000

## As variaveis de ambiente serão preenchidas pelo fargate
CMD mlflow server \
    --host 0.0.0.0 \
    --port 5000 \
    --default-artifact-root ${BUCKET} \
    --backend-store-uri mysql+pymysql://${USERNAME}:${PASSWORD}@${HOST}:${PORT}/${DATABASE}
```

Por padrão, o MLflow utiliza a porta `5000` para comunicação. Instalamos e configuramos a dependências `pymysql` e `boto3` com o objetivo de fornecer conectividade entre o MLflow e o [AWS RDS MySQL](https://aws.amazon.com/pt/rds/mysql/) e [AWS S3](https://aws.amazon.com/pt/s3/). 

O objetivo do bucket S3 é guardar os artefatos gerados pelo modelo de Machine Learning, o MLflow entende isso como *file-store*, e de forma nativa já suporta o [AWS S3](https://aws.amazon.com/pt/s3/). 

A segunda forma de armazenamento de informações do MLflow chama-se *database-backed*, e é utilizado para guardar metadados, parâmetros dos modelos, métricas, tags e experimentos. 

Para esta segunda forma de armazenamento, que adicionamos uma conexão com o [AWS RDS MySQL](https://aws.amazon.com/pt/rds/mysql/) em nosso Dockerfile.

## Provisionando com AWS CDK

Para ganharmos produtividade e fluidez nesse artigo, utilizaremos  o AWS CDK como serviço de provisionamento de nossa infraestrutura.

Aqui no blog temos um [episódio](https://cloudatlas.tech/2021-03-01-ml-discovery-s1e2/) inteiro dedicado ao AWS CDK. Através desse serviço nossa infraestrutura será provisionada de ponta a ponta como código.

Agora, vamos listar todos os componentes necessários para nosso projeto :

* `AWS RDS MySQL` - Servir a camada de *Backend Store* do MLflow.
* `AWS ECS FARGATE` - Container serverless com nosso MLflow server.
* `AWS Elastic Load Balancer` - Balanceador responsavel por receber as requisções e direcionar ao MLflow server.
* `AWS S3` - Bucket que armazenará a camada de  .
* `AWS Secrets Manager` - Armazenamento de nossa senha do banco de dadosRDS MySQL.
* `Roles e Parametros` - Associação de Roles para a Task de nosso ECS Fargate e paranmetrização de variáveis de ambiente.

Agora, vamos ao código CDK. Explicarei parte por parte do código utilizado para o provisionamento.

Como dito anteriormente, para as etapas de inicialização do projeto e download de dependências possuímos um [episódio](https://cloudatlas.tech/2021-03-01-ml-discovery-s1e2/) aqui no blog que explica detalhadamente estes passos.

Iniciamos declarando todas as dependencias/constructs que serão utilizados em nossa infra estrutura.

```python
from aws_cdk import (
    aws_ec2 as ec2,
    aws_s3 as s3,
    aws_ecs as ecs,
    aws_rds as rds,
    aws_iam as iam,
    aws_secretsmanager as sm,
    aws_ecs_patterns as ecs_patterns,
    core
)
```

Em nossa classe principal, antes de iniciarmos a criação dos recursos, precisamos realizar algumas parametrizações que serão utilizadas na imagem Docker, como, por exemplo, os parâmetros do banco de dados.

```python
class MLflowStack(core.Stack):
    def __init__(self, scope: core.Construct, id: str, **kwargs) -> None:
        super().__init__(scope, id, **kwargs)
        # Parâmetros utilizados para provisionamento da infra
        project_name_param = core.CfnParameter(scope=self, id='ProjectName', type='String')
        db_name = 'mlflowdb'
        port = 3306
        username = 'master'
        bucket_name = f'{project_name_param.value_as_string}-artifacts-{core.Aws.ACCOUNT_ID}'
        container_repo_name = 'mlflow-containers'
        cluster_name = 'mlflow'
        service_name = 'mlflow'
```

Iniciaremos efetivamente o provisionamento de nossa infraestrutura, definindo `IAM Roles` e segredos no AWS Secrets Manager.

```python
#Associação das policys gerenciadas a role que sera atribuida a task ECS.
        role_mlflow = iam.Role(scope=self, id='TASKROLE', assumed_by=iam.       ServicePrincipal(service='ecs-tasks.amazonaws.com'))
        role_mlflow.add_managed_policy(iam.ManagedPolicy.from_aws_managed_policy_name('AmazonS3FullAccess'))
        role_mlflow.add_managed_policy(iam.ManagedPolicy.from_aws_managed_policy_name('AmazonECS_FullAccess'))

#Secrets Manager responsavel pelo armazenamento do password do nosso RDS MySQL
db_password_secret = sm.Secret(
            scope=self,
            id='DBSECRET',
            secret_name='dbPassword',
            generate_secret_string=sm.SecretStringGenerator(password_length=20, exclude_punctuation=True)
        )
```

Agora provisionaremos as camadas de *artifact store* e *backend store* representados pelo nosso bucket S3 e RDS MySQL respectivamente.
```python
        #Criação do Bucket S3
        artifact_bucket = s3.Bucket(
            scope=self,
            id='ARTIFACTBUCKET',
            bucket_name=bucket_name,
            public_read_access=False,
            block_public_access=s3.BlockPublicAccess.BLOCK_ALL,
            removal_policy=core.RemovalPolicy.DESTROY
        )

        #Obtenção de VPC para atribuição ao RDS
        dev_vpc = ec2.Vpc.from_vpc_attributes(
            self, '<VPC_NAME>',
            vpc_id = "<VPC_ID>",
            availability_zones = core.Fn.get_azs(),
            private_subnet_ids = ["PRIVATE_SUBNET_ID_1","PRIVATE_SUBNET_ID_2","PRIVATE_SUBNET_ID_3"]
       )

        
        # Criação de Security Group para acesso ao DB
        sg_rds = ec2.SecurityGroup(scope=self, id='SGRDS', vpc=vpc_dev, security_group_name='sg_rds')
        
        # Adicionamos aqui efeito de testes 0.0.0.0/0
        sg_rds.add_ingress_rule(peer=ec2.Peer.ipv4('0.0.0.0/0'), connection=ec2.Port.tcp(port))
        # Criação da instancia RDS
        database = rds.DatabaseInstance(
            scope=self,
            id='MYSQL',
            database_name=db_name,
            port=port,
            credentials=rds.Credentials.from_username(username=username, password=db_password_secret.secret_value),
            engine=rds.DatabaseInstanceEngine.mysql(version=rds.MysqlEngineVersion.VER_8_0_19),
            instance_type=ec2.InstanceType.of(ec2.InstanceClass.BURSTABLE2, ec2.InstanceSize.SMALL),            
            security_groups=[sg_rds],           
            # multi_az=True,
            vpc=vpc_dev,
            removal_policy=core.RemovalPolicy.DESTROY,
            deletion_protection=False
        )
```

Agora, como ultimo componente restante de nossa arquitetura, vamos provisionar nosso servidor `MLflow` tendo como base a imagem `Docker` apresentada no inicio deste artigo

```python
#Criação do Cluster ECS
cluster = ecs.Cluster(scope=self, id='CLUSTER', cluster_name=cluster_name)
        #Task Definition para Fargate
        task_definition = ecs.FargateTaskDefinition(
            scope=self,
            id='MLflow',
            task_role=role,
        )
        #Criando nosso container com base no Dockerfile do MLflow
        container = task_definition.add_container(
            id='Container',
            image=ecs.ContainerImage.from_asset(
                directory='../../container',
                repository_name=container_repo_name
            ),
            #Atribuição Variaves ambiente
            environment={
                'BUCKET': f's3://{artifact_bucket.bucket_name}',
                'HOST': database.db_instance_endpoint_address,
                'PORT': str(port),
                'DATABASE': db_name,
                'USERNAME': username
            },
            #Secres contendo o password do RDS MySQL
            secrets={
                'PASSWORD': ecs.Secret.from_secrets_manager(db_password_secret)
            }
        )
        #Port Mapping para exposição do Container MLflow
        port_mapping = ecs.PortMapping(container_port=5000, host_port=5000, protocol=ecs.Protocol.TCP)
        container.add_port_mappings(port_mapping)

        #Atribuição de Load Balancer
        fargate_service = ecs_patterns.NetworkLoadBalancedFargateService(
            scope=self,
            id='MLFLOW',
            service_name=service_name,
            cluster=cluster,
            task_definition=task_definition
        )

        # Security group para ingress
        fargate_service.service.connections.security_groups[0].add_ingress_rule(
            peer=ec2.Peer.ipv4('0.0.0.0/0'),
            connection=ec2.Port.tcp(5000),
            description='Allow inbound from VPC for mlflow'
        )

        # Auto Scaling Policy para nosso balanceador
        scaling = fargate_service.service.auto_scale_task_count(max_capacity=2)
        scaling.scale_on_cpu_utilization(
            id='AUTOSCALING',
            target_utilization_percent=70,
            scale_in_cooldown=core.Duration.seconds(60),
            scale_out_cooldown=core.Duration.seconds(60)
        )
 ```

 Apenas para enriquecer nossa *stack*, vamos adicionar um `output` contendo o *DNS Name* do balanceador provisionado no código acima.

 ``` python
  core.CfnOutput(scope=self, id='LoadBalancerDNS', value=fargate_service.load_balancer.load_balancer_dns_name)
```

Enfim, realizamos o deploy de nossa stack.

```bash
$ cdk deploy
```

Finalizado o provisionamento de nossa infraestrutura, podemos acecssar o *front-end* do **MLflow server**.

## Apresentando o MLflow

Em nossa stack do AWS CDK, colocamos como parâmetro de *output* o endereço DNS de nosso balanceador.  

<p style="text-align: center"><img src="https://i.imgur.com/co9HjjV.jpg"></p>

Com este endereço, podemos acessar o **MLflow server** via navegador:

<p style="text-align: center"><img src="https://i.imgur.com/ovFX54h.jpg"></p>

Agora, partimos para o upload do nosso container em nosso repositório do [MLflow](https://https://mlflow.org/).

Durante os proximos passos utilizaremos dois *registries* diferentes:

* [AWS ECR](https://aws.amazon.com/pt/ecr/)
* MLflow Registry

O primeiro será responsável por armazenar a imagem do nosso modelo, enquanto que o segundo armazenará o código do nosso modelo de Machine Learning.

Por debaixo dos panos, no momento em que realizamos o deploy do nosso modelo, a imagem em nosso repositório ECR junto com o código fonte do nosso modelo transformam-se em uma nova imagem.

Tal imagem será entregue em um `AWS Sagemaker Endpoint` expondo as rotas `/invocations` e `/ping`, conforme explicado no início do artigo.

Chega de falar, vamos para o código.

## Talk is Cheap, show me the code!

Para nosso artigo vamos utilizar o comando abaixo:
``` bash
$ mlflow sagemaker build-and-push-container
```
Este, é responsável por criar um container padrão do MLflow para o [AWS Sagemaker Endpoint](https://docs.aws.amazon.com/sagemaker/latest/dg/deploy-model.html).

Tal container será transmitido ao `AWS ECR` para ser utilizado posteriormente em nosso deploy.

Agora, vamos criar um modelo no [MLflow](https://https://mlflow.org/)

```python
import mlflow
import boto3
import mlflow.sagemaker
import mlflow.sklearn
from sklearn.datasets import load_iris
from sklearn import tree

#Iniciamos atribuindo a URL do MLflow server, a mesma do output de nossa stack CDK.
mlflow.set_tracking_uri('<Loadbalancer DNS Name>')

# Load do dataset Iris
dataset_iris = load_iris()

# Region que nosso modelo será implantado
region = sagemaker.Session().boto_region_name

# Obtenção da role de execução das API do Sagemaker
role = sagemaker.get_execution_role() 

# Ajuste de um modelo de árvore
iris_model = tree.DecisionTreeClassifier()
iris_model = iris_model.fit(dataset_iris.data, dataset_iris.target)

#Chamando o metodo log_model para salvar um MLflow Model
mlflow.sklearn.log_model(iris_model,"sk_models")  
```

Com o nosso modelo já presente no *registry* do MLflow, podemos realizar o deploy do mesmo.

```python
role = sagemaker.get_execution_role() 
image_uri = '<Endereço Imagem ECR mlflow-pyfunc>'
endpoint_name = 'iris-endpoint'
mlflow_model_uri = 'models:/<NOME_MODELO>/<VERSAOMODELO>'

mlflow.sagemaker.deploy(
    mode='create',
    app_name=endpoint_name,
    model_uri=mlflow_model_uri,
    image_url=image_uri,
    execution_role_arn=role,
    instance_type='ml.m5.xlarge',
    instance_count=1,
    region_name=region
)
```
Como o nosso Sagemaker Endpoint entregue pela api do `MLflow server`, bora então para o nosso amado predict.

## Predict

```python
#Runtime do Sagemaker
runtime= boto3.client('runtime.sagemaker')

#Obtendo dados para o predico
raw_data =  "data:[[0,0,0,0]]"
payload  = json.dumps(raw_data)

#Enviando Requisição ao nosso Endpoint
runtime_response = runtime.invoke_endpoint(EndpointName=endpoint_name, ContentType='application/json', Body=payload)

#Convertendo resultado para json
result = json.loads(runtime_response['Body'].read().decode())

#Obtendo Predição e Payload enviado no request
print(f'Payload: {payload}')
print(f'Predicao: {result}')
```

Exbindo o resultado: 

```bash
Payload: "data:[[0,0,0,0]]"
Predicao: "[Iris-setosa]



## Não esqueça do *Cleanup*

Por ultimo, porém nao menos importante, vamos deletar o Endpoint que criamos e logo em seguida a stack do CDK, respectivamente.

```
mlflow.sagemaker.delete(app_name=endpoint_name, region_name=region)
```

Em seguida, deletamos os demais recursos criados via CDK.

```bash
cdk destroy
```

## Pensamentos Finais

Bom pessoal, foi uma longa jornada até aqui, porém, muito enriquecedora.

Conseguimos montar uma solução de catalogação e deploy de nossos modelos de Machine Learning, utilizando o MLflow e o Sagemaker Endpoint.

Por utilizarmos serviços também disponíveis em outras *clouds* (Azure, GCP), esta mesma arquitetura pode ser replicada em abordagens *multi-cloud*.

Até o próximo episódio!

<p style="text-align: center"><img src="https://64.media.tumblr.com/7151274239517b2d595ea04b17da4b0b/tumblr_mmzgqw26UY1qafzsyo1_r1_500.gifv"></p>

## Códigos deste post

- [MLflow-ML-Discovery-S01E03](https://github.com/Cloud-Atlas-BR/MLflow-ML-Discovery-S01E03)

## Referências
* [CDK Developer Guide](https://docs.aws.amazon.com/cdk/latest/guide/home.html)
* [MLflow](https://https://mlflow.org/)
* [AWS Sagemaker Endpoint](https://docs.aws.amazon.com/sagemaker/latest/dg/deploy-model.html)
* [AWS Fargate](https://aws.amazon.com/pt/fargate)