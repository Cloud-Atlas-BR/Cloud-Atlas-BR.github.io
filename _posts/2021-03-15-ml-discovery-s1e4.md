---
layout: post
title: MLflow e Sagemaker, juntos somos mais fortes.
subtitle: ML Drops v3
tags: [aws, mlops, cloudformation, iaac, iac]
comments: true
draft: true
---

{: .box-note}
**Este é um texto em desenvolvimento**: Ainda estamos escrevendo e/ou revisando seu conteúdo. Até o dia de sua publicação, ele não estará listado na página inicial do blog.


Fala Galera!

No Drops de hoje, vamos falar sobre um assunto interessantíssimo, aprenderemos como utilizar em conjunto o [MLflow](https://mlflow.org/) e o [AWS Sagemaker](https://aws.amazon.com/pt/sagemaker/) para catalogação e ***Deploy*** dos nossos Modelos de Machine Learning.

Aqui, o protagonismo fica a cargo do [MLflow](https://mlflow.org/) cuja a responsabilidade é servir de registry central para nossos modelos, enquanto que o [AWS Sagemaker](https://aws.amazon.com/pt/sagemaker/) ficará responsável por expor um Endpoint de consumo para o modelo. Este serviço é conhecido como [AWS Sagemaker Endpoint](https://docs.aws.amazon.com/sagemaker/latest/dg/deploy-model.html).


O objetivo aqui é apresentar de forma didática e simples a parceria entre  [MLflow](https://mlflow.org/) e [AWS Sagemaker](https://aws.amazon.com/pt/sagemaker/). Mostraremos também os frutos dessa parceria através de uma implementação real de um modelo de Machine Learning utilizando esses dois serviços.


Let's Bora !

## Conhecendo o MLFlow

O [MLflow](https://mlflow.org/) é definido como uma plataforma open source para gerenciamento do ciclo de vida de um Modelo de Machine Learning.

<p style="text-align: center"><img src="https://i.imgur.com/aosrKdj.png?1"></p>


Partindo da questão Ciclo de Vida de um Modelo, o [MLflow](https://mlflow.org/) concentra suas funcionalidades em quatro principais componentes

* [MLflow Tracking](https://mlflow.org/docs/latest/tracking.html) - Monitoração de Modelos
* [MLflow Projects](https://mlflow.org/docs/latest/projects.html) - Garantia de reprodutibilidade e idempotencia do Modelo
* [MLflow Models](https://mlflow.org/docs/latest/models.html) - Produtização e Deploy
* [MLflow Model Registry](https://mlflow.org/docs/latest/model-registry.html)  - Repositório/Catalogo centralizado dos Modelos

Para o Drops de hoje iremos demonstrar a funcionalidade do quarto componente, o [MLflow Model Registry](https://mlflow.org/docs/latest/model-registry.html) 

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

Para termos um registro visual do que explicamos ate agora, apresento-lhes o desenho da arquitetura deste episódio.


<p style="text-align: center"><img src="https://i.imgur.com/l3iMrUJ.jpg"></p>

O objetivo desta arquitetura é justamente implementar de forma rápida e simples o que foi apresentado até aqui.

Para atingirmos este objetivo, utilizaremos os seguintes recurso:

 * `AWS ECR` - Registry que ficará a nossa imagem Docker
 * `AWS Fargate` - Serviço AWS que disponibiliza o AWS ECS de forma serverless.
 * `AWS Sagemaker Endpoint` - serviço AWS responsável por expor o endpoint com nosso modelo de Machine Learning.

Bom, agora chega de blablablá e vamos para a ação.

## Let's Get Started

Iniciaremos configurando nosso servidor serverless do MLflow.

Como explicado anteriormente utilizaremos o [AWS Fargate](https://aws.amazon.com/pt/fargate), o mesmo tem como objetivo fornecer nosso [MLflow](https://https://mlflow.org/)  instanciado através de um container [Docker](https://www.docker.com/) em uma infraestrutura totalmente serverless.


 Abaixo temos o nosso Dockerfile referente ao [MLflow](https://https://mlflow.org/) :

```Dockerfile
ROM python:3.8.0

RUN pip install \
    mlflow \
    pymysql \
    boto3 & \
    mkdir /mlflow/

EXPOSE 5000

## As variaveis de ambiente serão preenchidas em nossa task definition
CMD mlflow server \
    --host 0.0.0.0 \
    --port 5000 \
    --default-artifact-root ${BUCKET} \
    --backend-store-uri mysql+pymysql://${USERNAME}:${PASSWORD}@${HOST}:${PORT}/${DATABASE}
```

Por padrão, o [MLflow](https://https://mlflow.org/) utiliza a porta `5000` para comunicação. Instalamos também algumas dependências como `pymysql` e `boto3` com o objetivo de fornecer conectividade entre o [MLflow](https://https://mlflow.org/), [AWS RDS MySQL](https://aws.amazon.com/pt/rds/mysql/) e [AWS S3](https://aws.amazon.com/pt/s3/). 

Para este exemplo também configuramos um bucket S3 e uma string de conexão para o [AWS RDS MySQL](https://aws.amazon.com/pt/rds/mysql/) em nosso Dockerfile.

O objetivo do bucket S3 é guardar possiveis artefatos gerados pelo modelo de Machine Learning, o MLflow entende isso como *file-store* e de forma nativa já suporta o [AWS S3](https://aws.amazon.com/pt/s3/). 

A segunda forma de armazenamento de informações do [MLflow](https://https://mlflow.org/) chama-se *database-backed* e é utilizado para guardar informações de metadados, parametros gerais, métricas, tag's e experimentos.

Para esta segunda forma de armazenamento estamos utilizando em nosso Dockerfile o [AWS RDS MySQL](https://aws.amazon.com/pt/rds/mysql/)

## Provisionando com AWS CDK

Para ganharmos produtividade e fluidez nesse artigo, utilizaremos  o AWS CDK como serviço de provisionamento de nossa infraestrutura.

Aqui no blog temos um [episódio](https://cloudatlas.tech/2021-03-01-ml-discovery-s1e2/) inteiro dedicado ao AWS CDK. Através desse serviço nossa infraestrutura será provisionada de ponta a ponta como código.

Agora, vamos listar todos os componentes necessários para nosso projeto :

* `AWS RDS MySQL` - Servir a camada de *Backend Store* do MLflow.
* `AWS ECS FARGATE` - Container serverless com nosso MLflow server.
* `AWS Elastic Load Balancer` - Balanceador responsavel por receber as requisções e direcionar ao MLflow server.
* `AWS S3` - Bcuker que armazenará a camada .
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

Agora, em nossa classe principal antes de iniciarmos a criação dos recursos, precisamos realizar algumas parametrizações que inclusive, serão utilizadas em nossa imagem Docker, como por exemplo os parâmetros de nosso banco de dados.

```python
class MLflowStack(core.Stack):
    def __init__(self, scope: core.Construct, id: str, **kwargs) -> None:
        super().__init__(scope, id, **kwargs)
        ##
        ##Parametros gerais utilizados para provisioamento de infra
        ##
        project_name_param = core.CfnParameter(scope=self, id='ProjectName', type='String')
        db_name = 'mlflowdb'
        port = 3306
        username = 'master'
        bucket_name = f'{project_name_param.value_as_string}-artifacts-{core.Aws.ACCOUNT_ID}'
        container_repo_name = 'mlflow-containers'
        cluster_name = 'mlflow'
        service_name = 'mlflow'
```

Iniciaremos então efetivamente o provisionamento de nossa infraestrutura começando pelas `IAM Roles` e Secrets Manager:
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

        #S ecurity group para ingress
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

 Apenas para enriquecer nossa stack, vamos adicionar um `output` contendo o DNS Name do balanceador provisionado no código acima :

 ``` python
  core.CfnOutput(scope=self, id='LoadBalancerDNS', value=fargate_service.load_balancer.load_balancer_dns_name)
  ```

Finalizamos o código da nossa infraestrutura, vamos acecssar a camada de front-end do [MLflow](https://https://mlflow.org/).

Antes, o deploy de nossa stack [AWS CDK](https://aws.amazon.com/pt/cdk/) :

```bash
$ cdk deploy
```

## Apresentando o MLflow

Em nossa stack do [AWS CDK](https://aws.amazon.com/pt/cdk/) colocamos como parâmetro de output o endereço DNS de nosso balanceador.  

<p style="text-align: center"><img src="https://i.imgur.com/co9HjjV.jpg"></p>



Com este endereço, podemos acessar o [MLflow](https://https://mlflow.org/) via browser :
<p style="text-align: center"><img src="https://i.imgur.com/ovFX54h.jpg"></p>

Agora, partimos para o upload do nosso container em nosso repositório do [MLflow](https://https://mlflow.org/).

Durante os proximos passos utilizaremos dois registries diferentes :

* [`AWS ECR`](https://aws.amazon.com/pt/ecr/)
* `MLflow Registry`

O primeiro será responsável por armazenar a imagem do nosso modelo, enquanto que o segundo armazenará o código do nosso modelo de Machine Learning.

Por debaixo dos panos, no momento em que realizamos o deploy do nosso modelo, a imagem em nosso repositório `AWS ECR` junto com o código fonte do nosso modelo transformam-se em uma nova imagem.

Tal imagem será entregue em um `AWS Sagemaker Endpoint` expondo as rotas `/invocations` e `/ping`, conforme explicado no início do artigo.

Chega de falar, vamos para o código.

## Talk is Cheap, show me the code !

