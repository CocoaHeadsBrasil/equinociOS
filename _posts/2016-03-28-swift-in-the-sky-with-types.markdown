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
