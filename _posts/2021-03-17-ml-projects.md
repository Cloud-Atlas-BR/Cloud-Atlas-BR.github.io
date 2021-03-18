---
layout: post
title: Montando um bot serverless para Twitter
subtitle:  Machine Learning Projects - 1
tags: [aws, mlops, project, bot, twitter]
comments: true
draft: true
---

{: .box-note}
**Este é um texto em desenvolvimento**: Ainda estamos escrevendo e/ou revisando seu conteúdo. Até o dia de sua publicação, ele não estará listado na página inicial do blog.

Diversas plataformas sociais permitem que seus usuários interajam através de APIs. Bastando que você tenha as credenciais da conta, Twitter, Telegram, ou mesmo Discord, permitem que você publique textos e curta comentários a partir de uma linguagem de programação (e.g. Python, R, Go).

Os chamados ***bots*** são utilizados com diferentes propósitos: empresas utilizam *bots* para melhor direcionar seus clientes num atendimento de dúvidas ou problemas; orgãos públicos e instituções desenvolvem *bots* para facilitar a comunicação com os cidadãos e MUITOS cientistas de dados dados utilizam *bots* para coleta de dados.

Então pensamos... Por que não criar um *bot* que nos ajude a informar a comunidade sobre os últimos temas de [#MLOPs]() no Twitter?

Assim, nasceu o [@MLOpsBot](https://twitter.com/MLOpsBot)!

<p style="text-align: center"><img src="https://i.imgur.com/3a0WjfB.png"></p>

Continuamente, o [@MLOpsBot](https://twitter.com/MLOpsBot) busca por novos tweets usando a hashtag [#MLOps](), e os retweeta.

## Claro que isso não é só uma propaganda!

Neste *post*, vamos entender a arquitetura implantada para o [@MLOpsBot](https://twitter.com/MLOpsBot) na AWS e as oportunidades que temos para incorporar *Machine Learning* em nosso *bot*.

## MLOps Bot

Dois requisitos guiaram o desenvolvimento deste *bot*: Possuir uma arquitetura (1) orientada a eventos, e (2) *serverless*. Para além das boas práticas que este desenho nos garante, também estamos pensando em **custos**

Utilizar buckets S3 e funções Lambda implica em um *billing* associado ao uso desses recursos, enquanto que ao utilizar um banco de dados (RDS) e máquinas virtuais (EC2), seriamos cobrados por hora do serviço existindo em nossas contas.

Desta forma, chegamos no seguinte desenho.

<p style="text-align: center"><img src="https://i.imgur.com/G2qjoCl.png"></p>

Abaixo, detalhamos cada etapa do funcionamento do MLOps Bot:

**(1)** A cada 10 minutos, o serviço [EventBridge](https://aws.amazon.com/eventbridge/) envia um evento para uma função Lambda.

**(2)** A partir do evento, a função Lambda busca as credenciais da API do Twitter no [Secrets Manager](https://aws.amazon.com/secrets-manager/).

**(3)** Com as credenciais, a função Lambda faz uma chamada à API do Twitter buscando os últimos 50 tweets com a hashtag #MLOps.

**(4)** A função Lambda salva os metadados de cada tweet individualmente em um bucket S3.

**(5)** Quando um novo tweet pousa no bucket, é acionando um evento do S3 que adiciona em uma fila do [SQS](https://aws.amazon.com/sqs/) a informação de um novo tweet para ser retweetado.

**(6)** O serviço Lambda faz o *polling* das mensagens na fila SQS e as envia para a função Lambda de retweet.

**(7)** De posse da informação da localização dos dados do tweet no bucket S3, a função lambda carrega os metadados dele em memória e valida se irá realizar o retweet.

**(8)** Validado, a função lambda obtém as credenciais da API e faz o retweet.

Abaixo, você pode verificar a anatomia de um tweet extraído obtido pela API do Twitter. Temos informações do usuário, estatísticas do tweet e o tweet em si.

```json
{
    "created_at": "Tue Mar 09 20:30:57 +0000 2021",
    "id": 1369385237197766664,
    "id_str": "1369385237197766664",
    "text": "Webinar: #MLOps with #Microsoft Azure and #Kainos - the path to building a competitive edge - March 26th, 2021 10:0\u2026 https://t.co/x7Q3zoqm8N",
    "truncated": true,
    "entities": {
        "hashtags": [
            {
                "text": "MLOps",
                "indices": [
                    9,
                    15
                ]
            },
            {
                "text": "Microsoft",
                "indices": [
                    21,
                    31
                ]
            },
            {
                "text": "Kainos",
                "indices": [
                    42,
                    49
                ]
            }
        ],
        "symbols": [],
        "user_mentions": [],
        "urls": [
            {
                "url": "https://t.co/x7Q3zoqm8N",
                "expanded_url": "https://twitter.com/i/web/status/1369385237197766664",
                "display_url": "twitter.com/i/web/status/1\u2026",
                "indices": [
                    117,
                    140
                ]
            }
        ]
    },
    "metadata": {
        "iso_language_code": "en",
        "result_type": "recent"
    },
    "source": "<a href=\"https://prod1.sprinklr.com\" rel=\"nofollow\">Sprinklr Publishing</a>",
    "in_reply_to_status_id": null,
    "in_reply_to_status_id_str": null,
    "in_reply_to_user_id": null,
    "in_reply_to_user_id_str": null,
    "in_reply_to_screen_name": null,
    "user": {
        "id": 26706077,
        "id_str": "26706077",
        "name": "Microsoft Developer Ireland",
        "screen_name": "MSdevIRL",
        "location": "Ireland",
        "description": "Follow for Dev tips, resources, code samples & events. Everything you need to develop great apps! \n\nSupport: @MicrosoftHelps",
        "url": "https://t.co/wFjYCZDIwx",
        "entities": {
            "url": {
                "urls": [
                    {
                        "url": "https://t.co/wFjYCZDIwx",
                        "expanded_url": "https://developer.microsoft.com/en-ie/",
                        "display_url": "developer.microsoft.com/en-ie/",
                        "indices": [
                            0,
                            23
                        ]
                    }
                ]
            },
            "description": {
                "urls": []
            }
        },
        "protected": false,
        "followers_count": 2922,
        "friends_count": 1748,
        "listed_count": 107,
        "created_at": "Thu Mar 26 07:16:21 +0000 2009",
        "favourites_count": 3206,
        "utc_offset": null,
        "time_zone": null,
        "geo_enabled": true,
        "verified": false,
        "statuses_count": 8763,
        "lang": null,
        "contributors_enabled": false,
        "is_translator": false,
        "is_translation_enabled": true,
        "profile_background_color": "000000",
        "profile_link_color": "4A913C",
        "profile_sidebar_border_color": "000000",
        "profile_sidebar_fill_color": "000000",
        "profile_text_color": "000000",
        "profile_use_background_image": false,
        "has_extended_profile": false,
        "default_profile": false,
        "default_profile_image": false,
        "following": false,
        "follow_request_sent": false,
        "notifications": false,
        "translator_type": "none"
    },
    "geo": null,
    "coordinates": null,
    "place": null,
    "contributors": null,
    "is_quote_status": false,
    "retweet_count": 4,
    "favorite_count": 2,
    "favorited": false,
    "retweeted": false,
    "possibly_sensitive": false,
    "lang": "en"
}
```

## Como criamos e atualizamos o MLOps Bot?

Para garantir o provisionamento de todos os recursos de forma automatizada, implementamos uma esteira de CI/CD utilizando o [Github Actions](https://github.com/features/actions). Inclusive, temos um episódio inteiro sobre esse tema aqui no blog.

<center>
<blockquote class="twitter-tweet"><p lang="pt" dir="ltr">A cultura <a href="https://twitter.com/hashtag/MLOps?src=hash&amp;ref_src=twsrc%5Etfw">#MLOps</a> tem como um dos seus pilares a automatização dos processos de <a href="https://twitter.com/hashtag/ML?src=hash&amp;ref_src=twsrc%5Etfw">#ML</a>.<br><br>Entregamos sempre, para entregar bem. Assim, processos podem evoluir a partir da experiência de erros e acertos.<br><br>Mais um post em parceira com <a href="https://twitter.com/Rjeks2?ref_src=twsrc%5Etfw">@Rjeks2</a>.<a href="https://twitter.com/hashtag/AWS?src=hash&amp;ref_src=twsrc%5Etfw">#AWS</a> <a href="https://twitter.com/hashtag/DevOps?src=hash&amp;ref_src=twsrc%5Etfw">#DevOps</a> <a href="https://t.co/NcD5hTc9kd">https://t.co/NcD5hTc9kd</a></p>&mdash; Son of Cydonia (@AdelmoFilho42) <a href="https://twitter.com/AdelmoFilho42/status/1368883619666747392?ref_src=twsrc%5Etfw">March 8, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script> 
</center>

A esteira que construimos tem a estrutura apresentada abaixo.

<p style="text-align: center"><img src="https://i.imgur.com/42ttrJe.png"></p>

Uma vez realizado um *push* ao repositório, a esteira inicia criando e subindo as imagens Docker dos Lambdas para um repositório ECR. 

É interessante comentar que não estamos utilizando as funções Lambda clássicas, mas funções Lambda com suporte a containers Docker. Também temos um *post* sobre esse serviço aqui no Blog.

<center>
<blockquote class="twitter-tweet"><p lang="pt" dir="ltr">Quer saber mais sobre deploy de machine learning na <a href="https://twitter.com/awscloud?ref_src=twsrc%5Etfw">@awscloud</a>?<br><br>O <a href="https://twitter.com/AdelmoFilho42?ref_src=twsrc%5Etfw">@AdelmoFilho42</a> e <a href="https://twitter.com/Rjeks2?ref_src=twsrc%5Etfw">@rjeks2</a> acabaram de lançar o primeiro post da serie. <br><br>AWS Lambda Containers + Machine Learning<a href="https://t.co/r9VW23YERk">https://t.co/r9VW23YERk</a><a href="https://twitter.com/hashtag/MLOps?src=hash&amp;ref_src=twsrc%5Etfw">#MLOps</a> <a href="https://twitter.com/hashtag/MachineLearning?src=hash&amp;ref_src=twsrc%5Etfw">#MachineLearning</a> <a href="https://twitter.com/hashtag/AWS?src=hash&amp;ref_src=twsrc%5Etfw">#AWS</a> <a href="https://twitter.com/hashtag/lambda?src=hash&amp;ref_src=twsrc%5Etfw">#lambda</a> <a href="https://twitter.com/hashtag/containers?src=hash&amp;ref_src=twsrc%5Etfw">#containers</a></p>&mdash; AWS User Group São Paulo (@awsusergroupsp) <a href="https://twitter.com/awsusergroupsp/status/1364187566086492161?ref_src=twsrc%5Etfw">February 23, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script> 
</center>

Em seguida, executamos um arquivo cloudformation contendo todos os recursos e conexões entre eles 

<center>
<a href="https://github.com/Cloud-Atlas-BR/MLOpsBot"><img align="center" src="https://github-readme-stats.vercel.app/api/pin?username=Cloud-Atlas-BR&repo=MLOpsBot&show_owner=true"></a>
</center>