---
layout: post
title: Criando um pipeline de MLOps na AWS - Parte 0
subtitle: Desenho de arquitetura  
tags: [aws, mlops, machine, learning]
comments: true
---

Conforme a área de *machine learning* evoluía, a dificuldade de obter bons modelos passou a caminhar junto com o esforço de implantar esses mesmos modelos. 

De um ponto de vista prático, implantar um modelo de *machine learning* pode envolver o provisionamento de infraestrutura (servidores), a containerização da aplicação (docker ou podman), escrita de testes unitários e de integração, e até o refatoramento do código em busca do aumento de performance. Idealmente, todos esses elementos fazem parte de uma esteira de integração e entrega contínua (CI/CD pipeline).

Nenhum dos pontos mencionados acima é novo, inclusive, fazem parte da cultura DevOps. O que torna essa disciplina um novo desafio para a área de *machine learning* são os **dados**.

Dados são entidades vivas. Ao longo do tempo, variam de volume, podem sofrer mudanças em sua distribuição estatística e disponibilidade, e, inevitavelmente, são manipulados por humanos, podendo sofrer de enviesamentos, erros, e da qualidade do código e análise desenvolvida. Todas essas questões, se não consideradas, resultam na perda da qualidade do modelo, lentidão de processamento, erros de processamento



Apesar de plural e de trabalhar diretamente com linguagens de programação, no seu dia-a-dia, o cientista de dados não (necessariamente) atua em ferramentas associadas à cultura DevOps: 



## 

## Arquitetura proposta


