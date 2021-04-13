---
layout: post
title: Projeto Hardin
subtitle: Cloud Atlas Projects - S02E01
tags: [aws, mlops, project]
comments: true
draft: false
---

Chegamos na segunda temporada do Cloud Atlas, e com ela queremos continuar informando e democratizando o uso da cloud para projetos de machine learning, mas, de uma maneira diferente nesta nova etapa.

Durante esta temporada, exploraremos novas ferramentas e serviços de forma orientada ao nosso projeto **Hardin - Aprendizado de máquina com dados da política brasileira**.

Neste primeiro episódio, vamos dedicar um tempo explicando nossos objetivos, arquitetura alvo, bem como a utilidade de cada componente representado em nosso desenho de solução.

## Qual é o Desafio?

Queremos construir um fluxo de ponta a ponta para a execução de modelos de machine learning. Nessa jornada, seguiremos desde a captura e catalogação dos dados até a construção e deploy de modelos.

**Que modelos são esses?** Não sabemos! (ainda)

Iremos explorar esses dados com serviços da AWS, e identificar oportunidades de desenvolvimento de modelos que colaborem com a sociedade no entendimento da esfera política e em nossas decisões como eleitores.

Aprenderemos muito nesse processo, e queremos que vocês sejam parte disso!

## Que dados são esses?

No Brasil, as diretrizes de transparência e acesso à informação permitiram a criação de plataformas de dados abertos. Nelas, é possível acessar informações sobre políticos e candidatos, gastos e projetos votados, e até mesmo o texto dos discursos realizados por um senador ou deputado em plenário.

<p style="text-align: center; margin-bottom: 0px"><a href="https://dadosabertos.camara.leg.br/"><img src="https://i.imgur.com/pmPl00h.png"></a></p>
<p style="text-align: center; margin-top:0"><b>Portal Dados Abertos da Câmara dos Deputados</b></p>

Partiremos dos portais de dados abertos da Câmara Federal e Senado. Em ambos, os dados são disponibilizados por meio de APIs ou arquivos *.csv*.

Conforme a necessidade, novas fontes de dados serão exploradas e sua captura e processamento implantados em nosso fluxo.

Inclusive, a engenharia de dados será um dos grande tema dessa temporada. É a partir de uma sólida fundação em dados que a engenharia de machine learning obtém seu potencial máximo.

## Qual a nossa arquitetura?

O desenho abaixo apresenta a nossa arquitetura de referência para este projeto. 

Para os que não conhecem os ícones utilizados, fiquem tranquilos, cada componente será explicado posteriormente. E para aqueles que conhecem, fiquem frios, detalharemos muito mais a arquitetura proposta.

<p style="text-align: center"><img src="https://i.imgur.com/LqQFSg6.png"></p>

## Quais serão nossos próximos passos?

Nos próximos posts, iremos trabalhar mais sobre cada componente em nossa arquitetura e dividir com vocês nossos aprendizados. Para isso, pensamos nas seguintes séries:

- **Cloud Atlas Essentials**: Vamos conhecer sobre a AWS e seus serviços de forma introdutória

- **Cloud Atlas Drops**: Tópicos rápidos de AWS e Machine Learning de forma prática e aplicada

- **Cloud Atlas Projects**: Divulgação do resultados e evolução do projeto Hardin

Nos vemos no próximo episódio!