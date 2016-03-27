---
layout:     post
title:      "Swift In The Sky With Types"
subtitle:   "Ou apenas mais uma conversa sobre linguagens, compiladores, tipos e... Swift!"
date:       2016-03-28 00:15:00
author:     "Matheus Brasil"
header-img: "img/mabrasil/cover.png"
category:   swift
---

> Antes de tudo, algumas coisas a se levar em consideração:

> 1. Esse post é baseado experiências - e eu não sou nenhum tipo de *dono da
verdade*: se você tem alguma opinião sobre isso - concordando, discordando,
complementando -, sinta-se livre para compartilhá-la.

> 2. Nem tudo nesse post é sobre Swift: boa parte dele está relacionado a
experiências bem gerais relacionadas a projeto de linguagens de programação.

> 3. Há boatos que a experiência de ler o post é enriquecida ouvindo-se o álbum
[Sgt. Pepper's Lonely Hearts Club Band](https://play.spotify.com/album/6QaVfG1pHYl1z15ZxkvVDW) ☺️.

## Prólogo

> Algumas coisas que me levaram a pensar sobre *Sistemas de Tipos* - e a escrever
esse post...

Eu, Matheus, sou originalmente desenvolvedor JavaScript. E, como em outras
linguagens, tenho de lidar com alguns *probleminhas*. Por exemplo, sua tipagem
dinâmica pode ser problemática: JavaScript não sabe que tipo uma variável é até
que esta seja realmente instanciada em execução - o que significa que pode ser
tarde demais, uma vez que você não sabe se algum erro de tipo estava em seu
código e quebrou antes de executá-lo.

Além disso, nós não podemos contar com caras como o `instanceof` e o` typeof`:
eles não funcionam de forma consistente - o que acaba nos mostrando que
verificação de tipos em JavaScript é um problema - e você provavelmente tem que
fazer alguns *workarounds* para ajudar você a superar tal inconsistência - ou
usar caras como o [TypeScript](http://www.typescriptlang.org/) ou o
[Flow](http://flowtype.org/).

O que tudo isso tem a ver com Swift? Bem, o contato com algumas inconsistências
relacionadas a tipos presenciadas em algumas linguagens - bem como sistemas de
tipos exemplares que pude experimentar em linguagems como
[Haskell](https://www.haskell.org/), [OCaml](https://ocaml.org/) e
[F#](http://fsharp.org/)- me fez ter sempre tal tópico em mente ao começar em
uma linguagem - e, há alguns meses atrás, quando comecei a estudar a linguagem
da Apple, não foi diferente.

## Mas Antes...

... de começarmos a ver coisas específicas de Swift relacionadas a seu
sistema de tipos, penso que seja interessante fazermos um *apanhado* de
alguns conceitos mais "gerais".

### Dados?

Esse é um conceito bem primitivo - e que muitas vezes não é discutido e só
aceito. Na *Filosofia*, temos uma definição parecida com isso:

> Coisas conhecidas ou assumidas como *fatos*, tornando-se a *base do raciocínio
ou cálculo*.

Ao adentrarmos na *Computação*, começamos a pensar de uma forma ou pouco
deferente sobre *dados* - ou melhor dizendo: um pouco mais *específica* para o
contexto em que estamos - e encontramos definições como:

> *Quantidades*, *caracteres* ou *símbolos* dos quais operações são executadas
por um computador, sendo armazenados e transmitidos na forma de sinais
eléctricos e gravados no suporte de gravação magnética, óptica ou mecânica.

E assim, começamos a pensar mais sobre dados como sendo um determinado pedaço de
informação - *textos*, *números*, *listas*, *figuras* etc.: são todos pedaços de
informação.

E, como podemos ler no famigerado
[*Livro do Mago*](https://mitpress.mit.edu/sicp/full-text/book/book.html):

> [...] Computational processes are abstract beings that inhabit computers. As
they evolve, processes manipulate other abstract things called data. The
evolution of a process is directed by a pattern of rules called a program.
People create programs to direct processes. In effect, we conjure the spirits
of the computer with our spells.

Dados são uma das partes mais importantes da arte de programar.

E, particularmente, eu diria que a conversa começa a ficar mais legal quando
pensamos sobre como essas informações são passadas a um computador - afinal,
eles não sabem o que um *número* é, ou o que o *infográfico de seu documento*
é, ou o que a *URL* da sua página é - e assim por diante.

E assim chegamos a - já esperada - conclusão de que, se queremos passar a nosso
computador qualquer uma dessas *pedaços de informação*, precisamos saber
representá-los de alguma maneira que o computador entenda.

Guarda bem essa palavra: **representação** - pois ela nos leva ao próximo
tópico.

### Tipos de Dados?

> **tl;dr**: Uma representação específica de algum(ns) dado(s).

Basicamente, essa galera aqui te diz como interagir com uma determinado
*pedaço de informação* e ainda pode dizer como essa informação (dado) é
representada por meio de outras informações mais primitivas - ou *vice-versa*:
como interpretar dados mais primitivos a partir de um determinado tipo de dados.

Se o ato de programar consiste em gerenciar processos de tal forma que eles
possam manipular dados, **é de importância trivial que tais processos manipulem
dados de forma eficiente e correta**.

Seguindo o raciocínio, modelar dados de forma que estes possam ser *manipulados
de forma eficiente e correta* é uma parte essencial de todas as tarefas de um
programador - e os tipos de dados são essenciais para tal modelagem - assim,
temos neles a **importantíssima** tarefa de garantir que as interações
com um determinado *pedaço de informação* estejam sempre corretas.

A propósito, guarde bem, também, essa palavra: **correto** - pois ela ainda será bem
discutida mais a frente.

### Sistemas de Tipos?

> **tl;dr**: Sistemas do tipo são, em sua essência, estruturas de análise de
programas.

Soa confuso? É, realmente pode ser *bem* confuso.

Que tal pedirmos ajuda aos universitários?

Bem, [Benjamin Pierce](http://www.cis.upenn.edu/~bcpierce/), em seu livro
[Types and Programming Languages](https://www.cis.upenn.edu/~bcpierce/tapl/),
nos conta que:

> A type system is a tractable syntactic method for proving the absence of
certain program behaviors by classifying phrases according to the kinds of
values they compute.

Eu diria que uma das chaves para entendermos o tópico é o trecho:

> [...] for proving the absence of certain program behaviors [...]

a partir do qual podemos ver que, para todo sistema de tipos específico,
haverá uma lista de coisas que este visa a provar - do que exatamente consiste
tal prova é deixado em aberto.

Uma forma prática de tentar enxergar isso é pensar em simples expressões. Por
exemplo, uma vez que o verificador de tipos pega expressão `1 + 2`, este pode
"olhar" para o `1` e inferir que se trata de um inteiro, olhar para o `2` - e
também inferir que este é um inteiro - e, em seguida, olhar para o operador
`+`, e saber que, quando `+` é aplicada a dois inteiros, o resultado é um
número inteiro - e aí temos nossa prova.

### *Correctness-by-Design*

Esse conceito aqui também é sempre legal de se pensar sobre, garanto.

Um dos principais objetivos de uma linguagem de programação deve ser a
capacidade de orientar programador a uma abordagem de *Correctness-by-Design*.
Isto é, um programa que é válido nesta linguagem, também deve ser um programa
que funciona como o esperado - ou seja, o sistema simplesmente não poderá chegar
a um estado inválido; um estado que não cumpre os requisitos de tal programa.

Para tanto, a linguagem deve fazer um esforço de rejeitar programas que possam
estar errados - ou, ao menos, torná-los mais difíceis de se escrever. Assim,
você literalmente não pode escrever código incorreto pelo simples fato de o
compilador não deixar.

### *Correctness-by-Design* & Sistemas de Tipos

Agora que já temos um ideia geral em torno de sistemas de tipos e de como
linguagens devem ser projetadas de modo a naturalmente evitar que programas
feitos nestas atinjam estados inválidos, você deve estar se perguntando: qual é
a exata relação entre ambos?

Sinceramente, eu gostaria de ter uma relação própria já estabelecida tão
expressiva quanto a feita pelo ilustre pesquisador
[Luca Cardelli](http://lucacardelli.name/), da
[Microsoft Research](http://research.microsoft.com/en-us/default.aspx):

> The fundamental purpose of a type system is to prevent the occurrence of
execution errors during the running of aprogram.

Ainda subjetivo? Ele vai além:

> Type systems provide conceptual tools with which to judge the adequacy of
important aspects of language definitions. Informal language descriptions
often fail to specify the type structure of a language in sufficient detail
to allow unambiguous implementation. It often happens that different
compilers for the same language implement slightly different type systems.
Moreover, many language definitions have been found to be type unsound,
allowing a program to crash even though it is judged acceptable by a
typechecker.

Assim, vemos no nosso sistema de tipos a figura responsável por *"julgar a
adequação de aspectos importantes das definições de uma linguagem"*, de forma a
a *"evitar a escrita de código incorreto pelo simples fato de o compilador não
deixar"*.

