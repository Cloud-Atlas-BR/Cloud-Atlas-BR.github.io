---
layout: post
title: Criando um pipeline de MLOps na AWS - S01E01
subtitle: Desenho de arquitetura  
tags: [aws, mlops, machine, learning]
comments: true
---

Conforme a área de *machine learning* evoluía, a dificuldade de obter bons modelos passou a caminhar junto com o esforço de implantar esses mesmos modelos. 

De um ponto de vista prático, implantar um modelo de *machine learning* pode envolver o provisionamento de infraestrutura (servidores), a conteinerização da aplicação (docker ou podman), escrita de testes unitários e de integração, e até o refatoramento do código em busca do aumento de performance. Idealmente, todos esses elementos fazem parte de uma esteira de integração e entrega contínua (CI/CD pipeline).

Nenhum dos pontos mencionados acima é novo, inclusive, fazem parte da cultura DevOps. O que torna essa disciplina um novo desafio para a área de *machine learning* são os **dados**.

Dados são entidades vivas. Ao longo do tempo, variam de volume, podem sofrer mudanças em sua distribuição estatística e disponibilidade, e, inevitavelmente, são manipulados por humanos, podendo sofrer de enviesamentos, erros, e da qualidade do código e análise desenvolvida. Todas essas questões, se não consideradas, resultam na perda da qualidade do modelo, lentidão de respostas, erros de processamento e **custos**.

Apesar de plural e de trabalhar diretamente com linguagens de programação, no seu dia-a-dia, o cientista de dados não (necessariamente) atua ou tem conhecimento nas ferramentas necessárias para implantar, observar e dar manutenção nos seus modelos. Nesse momento, entra a figura do **engenheiro de machine learning**.

Combinando *skills* de ciência de dados com engenharia de software, o engenheiro de machine learning não só atua no *deploy* do modelo, mas também na criação dos componentes que permitirão que o modelo possa ser implantado e que virão a compor a esteira de implantação desses modelos.

As particularidades impostas pela dinâmica entre os dados, e o modelo a ser implantado, resultou na criação de uma disciplina filha da DevOps, conhecida como **MLOps** (*machine learning operations*).

É claro que **Devops** e **MLOps** são da mesma familia como mencionado acima, porém, é importante entender que é possível aplicar conceitos pré-existentes no cenário **Devops**, entretanto, não é Ctrl+C Ctrl+v. Abaixo explicamos as principais diferenças.

## O que pode compor um pipeline de MLOps?

Do ponto de vista do cientista de dados, um pipeline de MLOps integra alterações de código realizadas em uma ferramenta de controle de versão (git), a chamada integração contínua (CI), etapa típica dos pipelines de DevOps. Os pipelines de MLOps, ainda podem se beneficiar de outras etapas tradicionais das esteiras de CI/CD de DevOps, tais como:

- Testes unitários de código
- Testes de integração
- Validações de segurança
- Provisionamento de infraestrutura do modelo

Por outro lado, as esteiras de MLOps, em razão da natureza única dos modelos de *machine learning*, necessitam da criação de etapas customizadas, a citar:

### Treinamento do modelo

Os artefatos de modelagem (.pkl, .RData, .joblib) podem ingressar ou serem criados em tempo de execução da esteira. No primeiro caso, os artefatos foram enviados pelo cientista de dados para algum *object storage* (S3, Google Drive, MinIO), e a esteira realiza a cópia deles para o container, o cluster, ou a VM a ser provisionada. Apesar de prática, está abordagem pode sofrer de problemas de reprodutibilidade, uma vez que o treinamento do modelo não está associada a nenhum processo sistêmico e parametrizável.

No caso do modelo ser treinado durante a execução da esteira, temos um maior controle do artefato sendo criado, pois todas as configurações de como treinar o modelo devem estar expostas nos arquivos do repositório git do cientista de dados. Desta forma, é possível desenhar diferentes controles para decidir se o modelo pode ser implantado dada sua qualidade e performance; realizar o versionamento de modelos treinados anteriores; e garantir que as bases de dados existam e tem a qualidade necessária para executar o treinamento.




### Provisionamento dos componentes de *observability* do modelo

    - Monitoração e Alarmes

Durante esta etapa o principal objetivo é entender o comportamento do nosso Modelo de Machine Learning nao apenas nos atentamos a metricas transacionais, mas também levamos em consideração metricas especificas dos modelos, e tais metricas podem ser utilizadas nas etapas subsequentes e assim servindo de *trigger , como por exemplo, a recalibração do modelo, atualização de coleta de metricas, disponibilização de novos testes A/B.

    - Coleta de métricas

Aqui ja começamos adentrar em questoes relacionadas a personalização da etapa **Monitoração e Alarmes**. Independente de qual metrica o modelo esta utilizando(gini,mse,rmse,acurácia), cabe ao pipeline o provisionamento e execução desta etapa.

Algumas estrategias podem ser utilizadas, como por exemplo, coletarmos metricas de forma assincrona a fim de realizar uma comparação de resultados com uma baseline previamente criado. Outra estrategia que também poderiamos utilizar seria medir o **Desvio Padrão**, e caso o mesmo sofresse um aumento e X %, um alarme/monitoração poderia ser ativado.

Aqui a principal mensagem é que esta etapa do pipeline não é efemera, a mesma mantem-se de forma simbiótica com o Modelo de ***Machine Learning** entregue pela pipeline.


    - Testes A/B
    - Recalibração do modelo


## Criando um pipeline de MLOps na AWS

Bom, a partir daqui, já podemos apresentar uma arquitetura base com todos os componentes que entendemos como essencias para um pipeline **MLOps**.

Para essa jornada que estamos iniciando, utilizaremos como provider a Amazon, nossa querida AWS. O motivo é a familiaridade, vivência e facilidade de implantação da arquitetura base, bem como de todos os componentes e serviços envolvidos neste pipeline de MLOps.

### Arquitetura proposta


