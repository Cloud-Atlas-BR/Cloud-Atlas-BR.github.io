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

Como dito anteriormente, o CDK quer ser visto como ***"Infra as Real Code"***, essa mensagem carrega uma carga de autonomia designada diretamente ao desenvolvedor que tem como objetivo entregar sua aplicação/projeto. Linguagens como TypeScript, JavaScript, NodeJs e Python. Aqui estamos lhe dando com um conceito chamado **Transpiler** que nada mais é que obter o código em uma linguagem específica como JavaScript e traduzir para um outro código correspondente.

Entao toda aquela verbosidade do Cloudformation é substituida por uma sintaxe familiar ao desenvolvedor ao passo que o seu desenvolvimento fica extramente mais direto e prazeroso. Como input, temos código puro em uma linguagem de preferencia do desenvolvedor que executa um transpiler cujo output sera um script de Cloudformation.

E claro, um dos beneficios é que nao precisamos dedicar tempo no aprendizado de uma DSL especifica como por exemplo o **Terraform**.

## Gerenciamento de Estados 

Como ja utilizado por outras ferramentas de IAC, o CDK também utiliza-se de um gerenciador de estados para a stack de Cloudformation, e tais estados sao armazenados em um bucket S3, bem como informações sobre os recursos de infra provisionados.

Bom, introdução e contextualização concluidas com sucesso !

## Proposta

Agora vamos pensar de forma conjunta, e se utilizassemos o nosso primeiro episódio de ML discovery e implementarmos o CDK como camada de abstração de nossa infraestrutura utilizando a linguagem Python como padrão.

## Talk is cheap, show me the code

Bom, entao vamos começar.

Primeiro, vamos instalar o CDK.

```console
rjekstein@instance~$ pip install aws-cdk.core, aws-cdk.aws-lambda
```
Como podemos ver, o CDK parte de uma instalação **core**, junto com essa instalação precisamos informar com quais módulos do CDK iremos trabalhar, que para esta caso utilizaremos o **lambda**.

Com os pacotes instalados precisamos inicializar o projeto com a seguinte diretiva.

```console
rjekstein@instance~$ cdk init mldiscovery-app --language python
```
com o comando executado acima o CDK irá criar uma estrutura de diretorios e arquivos contendo toda as peças necessárias para realizar o provisionamento da infraestrutura.
vamos entao conhecer o que temos dentro do diretorio da aplicação.

```console
(.env) $ tree
.
├── README.md
├── app.py
├── cdk.json
├── mldiscovrey_app
│   ├── __init__.py│   
│   └── mldiscovrey_app_stack.py
├── requirements.txt
├── setup.py
└── tests
    ├── __init__.py
    └── unit
        ├── __init__.py
        └── test_hello_construct.py
|_ source.bat
```

Agora vamos entender qual a função de cada elemento acima, os mesmos fazem parte do diretorio em que inicializamos o projeto **CDK**.

* README.md - arquivo README para o projeto
* app.py - script "main" para o projeto CDK.
* cdk.json - arquivo de condiguração que lista quais componentes devem ser executados durante o deploy da stack.
* mldiscovery_app - 









