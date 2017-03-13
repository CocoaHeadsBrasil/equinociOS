---
layout:     post
title:      "Introdução a Arquitetura Evolutiva"
subtitle:   "O básico para construir uma arquitetura que evolua com a sua aplicação"
date:       2017-03-03 00:00:00
author:     "Bruno Mazzo"
header-img: "img/brunomazzo/building.jpg"
category:   arquitetura
---

Já é bem aceito dentro do mundo de desenvolvimento de software que novos requisitos e funcionalidades de um produto podem e vão aparecer durante o processo de desenvolvimento. Por causa disso metodologias ágeis nasceram e são muito bem aceitas dentro da indústria. Porém nem tudo dentro da indústria segue um modelo incremental.

Imagine que você irá começar um novo projeto, qual a primeira coisa que você precisa fazer? Muitos vão responder “definir a arquitetura/framework que eu vou usar”. Existem muitos pontos errados nesse pensamento, mas o mais errado é a mentalidade de que a arquitetura é algo estático, imutável no projeto. Muitos pensam que se não escolhermos uma arquitetura no começo do projeto o código ficará bagunçado e ruim.

Como não sabemos todas funcionalidades a sua aplicação irá ter, não faz sentido tentar escolher uma arquitetura que possa atender a tudo. Infelizmente não existe um padrão "MValguma-coisa" que possa resolver todos os problemas. Ainda se pode argumentar que se sabe alguns requisitos antes de se iniciar o projeto e por isso devemos já começar com o modelo A. Porém, ainda não sabemos se os requisitos mais importantes já foram descobertos ou se a funcionalidade mais usada já foi pensada, então se comprometer com uma arquitetura é provavelmente otimizar para a coisa errada. Assim a resposta mais certa será: vamos começar simples e conforme formos adicionando funcionalidades, vamos também evoluindo e adaptando a arquitetura. A arquitetura da aplicação deve mudar e crescer junto com a aplicação. 

# Arquitetura evolutiva

Arquitetura evolutiva são princípios que podemos seguir para conseguir obter esse comportamento incremental de maneira um pouco menos trabalhosa. Vou explicar os que eu considero os mais importantes, mas caso concorde com eles procure os outros porque eles também acrescentam aos projetos.

## Complexidade

Em todo projeto temos dois tipos de complexidade: 
- a inerente ao problema
- a acidental

A inerente ao problema é basicamente a complexidade que nosso software precisa resolver. Infelizmente existem problemas complexos, geralmente é neles que trabalhamos e não temos muito o que fazer quanto a isso. Já a acidental vem da nossa tentativa de criar alguma abstração ou generalização desnecessária.

### KISS
O primeiro princípio é uma boa prática bem conhecida que tenta eliminar a complexidade acidental. Essa complexidade nasce porque tentamos fazer coisas mais “sofisticadas” do que o necessário. Para evitar isso devemos sempre que possível manter as coisas simples (KISS - Keep it simple). Uma boa arquitetura faz o problema parecer simples lidando com a complexidade inerente e não possuindo complexidade acidental.

OK, mas como fazer isso? Experimentando, testando soluções e experimentando mais, refatorando, aprendendo e evoluindo a cada tentativa. Assim chegamos ao segundo princípio: refatoração.


### Refatorar
Não importa quem você é, provavelmente não vai conseguir acertar de primeira. Isso até é bom, indica que podemos aprender mais e melhorar na próxima vez. Porém isso exige que você tenha capacidade e paciência para refatorar seu código constantemente. Constantemente mesmo, toda semana, todo dia! Só assim você vai poder encontrar uma solução que seja boa e evoluir o produto.

Teste novos padrões, implemente idéias que você teve durante uma discussão técnica, experimente um jeito novo que você viu em uma palestra, discuta os resultados e caso não ache que ficou mais simples de lidar com o problema apague e tente de novo. No pior caso você aprendeu que essa implementação não é o caminho certo e não devemos seguir com ela.

Agora para conseguirmos refatorar precisamos do terceiro princípio: reversibilidade.

### Reversibilidade
Devemos sempre tentar seguir por um caminho que caso não seja o certo, possamos voltar. Basicamente é, “não tenho certeza se é exatamente isso que eu preciso, então não vou comprar o pacote Premium Enterprise do vendor X pra testar”, vou experimentar alguma coisa que se não me atender, eu possa trocar sem prejudicar o projeto.

Isso vale para tudo, desde código até serviços e infraestrutura. Não crie uma estrutura complexa que caso não seja o que você precisa te obrigue a mudar tudo no projeto, a não ser que você saiba que é isso que você precisa.

Se o código é impossível de ser revertido, é impossível refatorar. Se não podemos refatorar, não conseguimos evoluir. Se não evoluímos, não iremos adaptar o software aos novos requisitos e passa a ser uma questão de tempo até a morte dele.

Para ajudar nesse caminho de reversibilidade temos o que chamamos de último momento de responsabilidade.

### Último momento de responsabilidade
O que é isso? Simples, adiei suas decisões até o último momento sem que isso te prejudique. O jeito mais fácil de explicar esse princípio é através de um exemplo: 

> Estou construindo um app e sei que eu vou precisar persistir dados nele. Vou começar a desenvolver hoje, será que é necessário decidir qual persistência de dados eu preciso agora? Provavelmente não. Posso trabalhar com a tela mocando a persistência e aprendendo como ela deve se comportar quando for implementada. 

>OK, terminei o layout da tela e agora vou recuperar os dados do servidor e salvar no app. Preciso decidir agora entre core data, ou realm, ou SQLite puro? Ainda não, posso implementar a chamada ao servidor e continuar mocando a persistência por enquanto. Assim eu aprendo como ela deve se comportar quando terminar uma requisição. 

> Agora com a requisição pronta, tela pronta falta apenas a persistência. Bem, agora eu preciso decidir. 

Adiando até o último momento eu aprendo o que a tela e a camada de rede esperam da persistência. Eu tenho mais conhecimento sobre o que eu realmente preciso e por isso tenho mais chances de acertar quando escolher.

# TL;DR
Durante o desenvolvimento da aplicação, aprendemos cada vez mais sobre o domínio do problema e qual a melhor maneira de resolver ele. A arquitetura evolutiva prega que devemos usar esse conhecimento adquirido diariamente e adicionar ao seu projeto. Para isso ela propoem práticas que ajudem você a alterar e adaptar seu projeto a essas mudanças.

Se você achou interessante e gostaria de aprender mais, recomendo fortemente ver a palestra de Venkat Subramaniam chamada [Towards an Evolutionary Architecture](https://www.youtube.com/watch?v=VEPwR4Hpi7M&t=21s)
