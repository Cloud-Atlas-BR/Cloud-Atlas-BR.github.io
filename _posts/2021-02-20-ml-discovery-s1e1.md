---
layout: post
title: Machine Learning Discovery - S01E01
subtitle: AWS Lambda Containers
tags: [aws, mlops, machine, learning, lambda, container, docker]
comments: true
draft: true
---

Em um levantamento da [2nd Watch](https://www.2ndwatch.com/), o AWS Lambda desponta como o [segundo serviço de computação mais popular](https://www.techrepublic.com/article/the-top-30-amazon-products-and-services-tech-pros-used-this-year/), e o 13° entre todos os serviços. Através das AWS Lambda *functions*, é possível executar códigos de [diferentes linguagens de programação](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html) (python, go, node, java) com toda infraestrutura sendo gerenciada pela AWS.

Essa poderosa simplicidade permite que a atenção do desenvolvedor esteja no código da aplicação, ao mesmo tempo que entrega features como:

- Integração nativa com dezenas de serviços (S3, SQS, SNS, Kinesis, Api Gateway)
- [Gratuidade de até 1 milhão de requisições por mês](https://aws.amazon.com/lambda/pricing/)
- Precificação do uso em incrementos de milissegundo

Lambda é uma escolha ideal para pequenos *workloads* agendados ou orientados a eventos. 
Pequenos, pois as limitações das funções lambdas estão na execução da sua aplicação em CPUs, consumindo até 10 GB de memória em no máximo 15 minutos.


Em dezembro de 2020, a [AWS anuncia o suporte a containers às funções Lambda](https://aws.amazon.com/blogs/aws/new-for-aws-lambda-container-image-support/). A popularidade do AWS Lambda pode ser explicada pela 


{: .box-note}
**Note:** This is a notification box.
asdad