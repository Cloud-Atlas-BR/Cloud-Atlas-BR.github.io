---
layout: post
title: Montando um twitter bot serverless
subtitle:  Machine Learning Projects - 1
tags: [aws, mlops, project, bot, twitter]
comments: true
draft: true
---

{: .box-note}
**Este é um texto em desenvolvimento**: Ainda estamos escrevendo e/ou revisando seu conteúdo. Até o dia de sua publicação, ele não estará listado na página inicial do blog.

Diversas plataformas sociais permitem que seu usuário possa ser manipulado através de chamadas de API. Bastando que você tenha as credenciais da conta, Twitter, Telegram, ou mesmo Discord, permitem que você publique textos e favorite comentários a partir de uma linguagem de programação (e.g. Python, R, Go).

Os chamados ***bots*** são utilizados com diferentes propósitos: empresas utilizam *bots* para melhor direcionar seus clientes num atendimento de dúvidas ou problemas; orgãos públicos e instituções desenvolvem *bots* para facilitar a comunicação com os cidadãos e MUITOS cientistas de dados dados utilizam *bots* para coleta de dados.

Então pensamos... Por que não criar um *bot* que nos ajude a informar a comunidade sobre os últimos temas de #MLOPs no Twitter?

Assim, nasceu o [@MLOpsBot](https://twitter.com/MLOpsBot)!

<p style="text-align: center"><img src="https://i.imgur.com/3a0WjfB.png"></p>

Continuamente, o [@MLOpsBot](https://twitter.com/MLOpsBot) busca por novos tweets usando a hashtag [#MLOps](), e os retweeta para que seus seguidores.

## Claro que isso não é só uma propaganda!



<p style="text-align: center"><img src="https://i.imgur.com/RW7rCWk.png"></p>

