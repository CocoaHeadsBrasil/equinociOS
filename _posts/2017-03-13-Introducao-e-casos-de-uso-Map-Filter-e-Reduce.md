---
layout:     post
title:      "Introdução e casos de uso: Map, Filter e Reduce."
date:       2017-03-13 02:19:00
author:     "Ezequiel França"
header-img: "img/ezefranca/header.png"
category:   "swift"
---

> Ezequiel França ([@ezefranca](https://twitter.com/ezefranca){:target="_blank"}) é desenvolvedor iOS, postador de piadas no Slack e também entusiasta de Internet das Coisas e mundo Maker.

O mundo iOS é gigantesco, e cada dia mais temos opções e alternativas, que vão desde a [programação reativa](https://www.raywenderlich.com/138547/getting-started-with-rxswift-and-rxcocoa), até a [programação funcional](https://www.raywenderlich.com/114456/introduction-functional-programming-swift) propriamente dita.

Para chegar lá não existe mágica, e alias, não que estes paradigmas vão resolver tudo, mas devemos pensar em evoluir a cada dia.

Bem, eu geralmente gosto de escrever sobre o que me fez falta. Quando falamos em Map, Filter e Reduce, principalmente para um desenvolvedor junior é como se estivéssemos falando em uma bicho de sete cabeças. Acredito que isso naturalmente seja resultado de como programção é ensinada, tópicos como programação funcional são típicos de cursos de Ciência ou Engenharia da Computação (ou Matemática), por terem um rigor matemático maior. Entretanto para entender e utilizar muitos destes recursos você não precisa ter estudado Cálculo IV ou ter muitos e muitos anos de programação para entender. 

A ideia deste artigo é justamente essa, mostrar alguns casos onde podemos utilizar estes conceitos funcionais, para facilitar ou deixar mais legível o tratamento de alguns tipos de dados.

### Teoria: Função de ordem superior

Uma função de ordem superior é uma função que:

• tem outra função como argumento, ou
• produz uma função como resultado.

![]({{  }}/img/ezefranca/hof.JPG)

Por enquanto é isso. Vamos entender melhor com exemplos.

### Map

![]({{  }}/img/ezefranca/map.png)

O Map faz um loop sobre uma coleção (um array por exemplo) e aplica a mesma operação a cada elemento da coleção. Vamos exemplificar. 
Digamos que temos um array com notas de alunos e queremos conceder um ponto extra para cada aluno. Poderiamos facilmente fazer um for e acrescentar cada elemento:

```swift

var alunosNotas = [4, 5, 7, 9, 6, 10, 3]

// Adicionar 1 a todos valores de um array de notas de alunos

var novaNotas:[Int] = []

/* Percorre todo array de alunosNotas, adiciona 1 ao valor em seguida adiciona
    no array novasNotas este elemento.
 */
for nota in alunosNotas {
    let novaNota = nota + 1
    novaNotas.append(novaNota)
}

```

Com isso teriamos no array ```novaNotas``` as notas acrescidas de um ponto. A ideia central do map é essa, fazer um loop sobre uma coleção, então vejamos como ficaria utilizando o ```map``` para fazer a mesma operação:

```swift

// Uma das váriações de sintaxes do map, utilizando closures

var alunosNotas = [4, 5, 7, 9, 6, 10, 3]

alunosNotas = alunosNotas.map({(nota:Int) -> Int in
    return nota + 1
})


```

Além da forma mostrada acima, podemos utilizar o ```map``` com a *syntax sugar* com $ (minha favorita), onde $0 é o elemento atual, $1 próximo elemento e assim sucessivamente:

```swift

var alunosNotas = [4, 5, 7, 9, 6, 10, 3]
alunosNotas = alunosNotas.map {$0 + 1}


```
Existem ainda outras formas de escrever o map, mas em todas elas a ideia central será percorrer uma coleção e retornar uma função em cada elemento.

#### Quando não utilizar map ?

Existem algumas situações onde utilizar ```map``` não seja uma boa escolha, como por exemplo, quando você não precisa percorrer a coleção inteira ou precisa interroper o loop em algum momento. Além disso não é recomendado utilizar em coleções muito, MUITO grandes.


### Filter

![]({{  }}/img/ezefranca/filter.png)

Faz um loop sobre uma coleção e retorna uma coleção que contém elementos que atendem a *uma condição*. Vamos exemplificar. 
Digamos que temos o nosso mesmo array com as notas dos alunos e queremos *filtar* apenas os alunos aprovados. Para ser aprovado é necessario que o aluno tenha uma nota 5.0 ou maior. Poderiamos facilmente fazer um for e comparar cada elemento:


```swift

var alunosNotas = [4, 5, 7, 9, 6, 10, 3]

// Adicionar 1 a todos valores de um array de notas de alunos

var aprovadosNotas:[Int] = []

/* Percorre todo array de alunosNotas, adiciona 1 ao valor em seguida adiciona
    no array novasNotas este elemento.
 */
for nota in alunosNotas {
    if nota >= 5 {
    	aprovadosNotas.append(novaNota)
    }
}

```

Com isso teriamos no array ```aprovadosNotas``` as notas dos alunos aprovados. A ideia central do ```filter``` é essa, fazer um loop sobre uma coleção e aplicar un "filtro" nela, então vejamos como ficaria utilizando o ```filter``` para fazer a mesma operação. Podemos utilizar o ```filter``` com a sintaxe de $ vista anteriormente:

```swift
// Verifica se a nota é >= a 5

var alunosNotas = [4, 5, 7, 9, 6, 10, 3]
alunosNotas = alunosNotas.filter { $0 >= 5 }

```
Assim como ```map```, existem ainda outras formas de escrever o ```filter```, mas em todas elas a ideia central será a mesma, filtrar os elementos de uma coleção, baseado em uma condição.

### Reduce

![]({{  }}/img/ezefranca/reduce.png)

Combina todos os itens de uma coleção para criar um único valor.
Vamos supor que precisamos obter a soma de todas as notas dos alunos, uma solução seria implementar outro loop For-In:


```swift

var alunosNotasReduce = [4, 5, 7, 9, 6, 10, 3]

var soma = 0

/* Percorre todo array de alunosNotasReduce, e a cada elemento soma ele mesmo com a variavel soma */

for nota in alunosNotasFilter {
    soma += nota
}

```

Com isso teriamos no valor ```soma``` a soma das notas dos alunos, essa é a ideia central do ```reduce```.

```swift

// Uma das sintaxes do reduce, utilizando $, onde ele recebe a closure com a operação e o valor inicial (nesse caso 0).

var alunosNotas = [4, 5, 7, 9, 6, 10, 3]
soma = 0
soma = alunosNotas.reduce (0, {$0 + $1})

```

Além da forma mostrada acima, podemos utilizar o ```reduce``` com uma sintaxe mais simples, passando o valor inicial e a operação que vai ocorrer entre cada elemento:

```swift

soma = 0
soma = alunosNotas.reduce (0, +)

```

Este artigo tem a intenção de ser um primeiro passo na utilização das funções de ordem maior. Além das mencionadas aqui, ainda temos o ```flatMap``` (a qual não será abordada, entretanto com [vasta fonte na internet](https://developer.apple.com/reference/swift/dictionary/1687661-flatmap)) e também temos as operações em cadeia, ou seja, a possibilidade de combinar todas essas funções.

### Revisão

![](http://www.monolitonimbus.com.br/wp-content/uploads/2015/01/revisao_telecurso.jpg)

**Map** : Retorna uma coleção contendo resultados de se aplicar uma transformação para cada item, **map** vai **mapear** sua coleção inteira.

**Filter** : Retorna uma coleção contendo apenas os itens que correspondem a uma condição de **filtro**.

**Reduce** : Retorna um único valor calculado através da sua coleção. **Reduz** sua coleção a um unico valor.

Brincadeiras a parte, espero que tenham gostado e que encontrem situações onde estas funções possam facilitar seu dia-a-dia, ou até mesmo ajudar você entender o código de outro desenvolvedor.

Todos os exemplos estão [neste playground](https://github.com/ezefranca/map-filter-reduce-equinocios).

Obrigado e até a próxima 😉!

![](https://media.giphy.com/media/l41YflLBmVOHbWCVq/giphy.gif)

### Referências

* [Sequence](https://developer.apple.com/reference/swift/sequence)
* [Closures](https://developer.apple.com/library/prerelease/content/documentation/Swift/Conceptual/Swift_Programming_Language/Closures.html)
* [Swift Guide to Map Filter Reduce](https://useyourloaf.com/blog/swift-guide-to-map-filter-reduce/)
* [Simple Higher Order Functions in Swift 3.0 — Map, filter,flatMap reduce](https://medium.com/@mimicatcodes/simple-higher-order-functions-in-swift-3-0-map-filter-reduce-and-flatmap-984fa00b2532#.4od07v215)
* [Esse vídeo: (principalmente os 2 primeiros segundos)](https://www.youtube.com/watch?v=v6wImnaYW1I)



