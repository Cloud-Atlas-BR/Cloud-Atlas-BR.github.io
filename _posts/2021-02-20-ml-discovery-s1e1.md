---
layout: post
title: Machine Learning Discovery - S01E01
subtitle: AWS Lambda Containers
tags: [aws, mlops, machine, learning, lambda, container, docker]
comments: true
draft: true
---

Em um levantamento da [2nd Watch](https://www.2ndwatch.com/), o AWS Lambda desponta como o [segundo serviço de computação mais popular](https://www.techrepublic.com/article/the-top-30-amazon-products-and-services-tech-pros-used-this-year/), e o 13° entre todos os serviços. Através das AWS Lambda *functions*, é possível executar códigos de [diferentes linguagens de programação](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html) (python, go, node, java) com toda infraestrutura sendo gerenciada pela AWS.

<p style="text-align: center"><img src="https://i.imgur.com/d9Bnnqq.png"></p>

Essa poderosa simplicidade permite que a atenção do desenvolvedor esteja no código da aplicação, ao mesmo tempo que entrega features como:

- Integração nativa com dezenas de serviços (S3, SQS, SNS, Kinesis, Api Gateway)
- [Gratuidade de até 1 milhão de requisições por mês](https://aws.amazon.com/lambda/pricing/)
- Precificação do uso em incrementos de milissegundo

Lambda é uma escolha ideal para pequenos *workloads* agendados ou orientados a eventos. 
Pequenos, pois as limitações das funções lambdas estão na execução da sua aplicação em CPUs, consumindo até 10 GB de memória em no máximo 15 minutos.

Existem, por outro lado, limitações menos comentadas.

O fato de trabalharmos apenas no nível da aplicação, nos impede (gambiarra free) de (1) instalar bibliotecas de sistema operacional e (2) garantir a reprodutibilidade dos resultados, pois não estaremos desenvolvendo no mesmo ambiente em que a função lambda é executada.

<p style="text-align: center"><img src="https://i.imgur.com/CZ6TqF5.jpg"></p>

Docker! [Assim como para metade de todos os problemas de desenvolvimento](https://i.redd.it/iv0oiaz7aqe41.jpg), o uso de *containers* docker é uma solução adequada para as limitações de reprodutibilidade: desenvolver no mesmo ambiente que o código será executado.

O projeto lambCI disponibiliza [imagens Docker](https://hub.docker.com/r/lambci/lambda/) para esse proposito. Inclusive sendo a [solução recomendada pela AWS para criação de Lambda Layers](https://aws.amazon.com/premiumsupport/knowledge-center/lambda-layer-simulated-docker/) - o equivalente de virtualenvs do python para as funções lambda.

Mas, covenhamos... o uso de imagens que mimetizam o ambiente Lambda não resolve integralmente as questões levantadas. Ainda não conseguimos customizar o container docker no qual o código da função Lambda é executado, ou migrar este container para uma infraestrutura *on-premisse*. Não temos controle sobre esse container.

Ou melhor, não tínhamos!

## Lambda Containers

Em dezembro de 2020, a [AWS anuncia o suporte a containers às funções Lambda](https://aws.amazon.com/blogs/aws/new-for-aws-lambda-container-image-support/).

Isso significa que o serviço AWS Lambda passa a ser capaz de executar containers definidos pelo desenvolvedor. Passando de um [*function-as-a-service*](https://en.wikipedia.org/wiki/Function_as_a_service) para algo como um *entrypoint-as-a-service*.

As limitações de tempo (15 minutos) e memória (10 GB) se mantém, mas, diferente dos 250 MB de espaço disponíveis para a execução das funções, é possível executar imagens docker de até 10 GB.

## Criando uma imagem docker para Lambda



```Dockerfile

```



{: .box-note}
**Note:** This is a notification box.
asdad