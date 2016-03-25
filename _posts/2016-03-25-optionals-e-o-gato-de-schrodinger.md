---
layout:     post
title:      "Optionals e o Gato de Schrödinger"
subtitle:	"Optionals? Optionals!"
date:       2016-03-25 01:23:45
author:     "Francesco Perrotti-Garcia"
header-img: "img/fpg1503/mozzie.jpg"
category:   "optionals"
---


> Francesco Perrotti-Garcia ([@fpg1503](https://twitter.com/fpg1503){:target="_blank"}) é desenvolvedor iOS. Atualmente trabalha no [PlayKids](https://playkidsapp.com){:target="_blank"}) fazendo a melhor família de aplicativos para crianças do mundo. Programa desde os 12 anos e nos últimos 5 está cada vez mas envolvido com desenvolvimento iOS. Swift mudou sua maneira de ver o mundo e até de como programar em Objective-C. Adora gatos e nas horas vagas gosta de viajar, cozinhar e tirar fotos.


# O que são Optionals?

O [Swift Programming Guide](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/TheBasics.html#//apple_ref/doc/uid/TP40014097-CH5-ID309) define `Optional` como:

> [...] tipos opcionais lidam com a ausência de um valor. Eles dizem “há um valor e ele 
é x” ou “não há valor algum”. Usá-los é semelhante a usar nil com ponteiros em Objective-C mas eles funcionam para todos os tipos, não só para classes. Eles não só são mais seguros e expressivos do que ponteiros nulos, como estão no coração de muitas das funcionalidades mais poderosas de Swift.


Desenvolvedores de Objective-C nunca se preocuparam muito com nulabilidade, enviar uma mensagem para `nil` simplesmente retornava `nil` e isso era lindo (ou pelo menos pensávamos assim). Toda conversa de bar com nossos colegas desenvolvedores Java eram um bom motivo para trazer *null pointer exceptions* à tona. Foi com essa mentalidade que eu e muitas colegas começamos a desenvolver em Swift, mas este não é um bom caminho. Depois de muita reflexão e discussão considero que a raiz de todo mal esteja em tentar fazer encarar `Optional`s da mesma forma que encarávamos `nil`. Uma maneira que gosto de abordar esse tema é usando a metáfora do *Gato de Schrödinger*.


# *Gato de Schrödinger*

## O que é?

Uma experiência mental na qual um gato é imaginado em uma caixa com uma fonte de radiação e um veneno que será liberado assim que essa fonte (imprevisivelmente) emitir radiação. O gato é (de acordo com a mecânica quântica) considerado ao mesmo simultaneamente vivo e morto até que a caixa seja aberta e o gato observado.

## Modelando

Deixando `Optional`s de lado por um tempo vamos supor agora que quiséssemos modelar esse problema: de maneira bem simplista podemos dizer que há dois estados possíveis para o gato: **vivo** e **morto**. Uma maneira interessante de fazer isso seria utilizando um `enum`:

~~~swift
enum Cat {
    case Alive
    case Dead
}
~~~

![]({{ site.baseurl }}/img/fpg1503/alivedead.png)


Se houvesse uma instância de `Cat` chamada `meow` a única maneira de sabermos se ele está vivo ou morto é checando (o que é análogo a abrir a caixa no experimento).

~~~swift
switch meow {
case .Alive:
    print("The cat is alive!")
case .Dead:
    print("The cat is dead :(")
}
~~~

# Fazendo o nosso `Optional`


## Abordagem Inicial

Voltando para Optionals podemos dizer que, como o gato, é possível modelá-los com dois estados: **alguma coisa** ou **nada**.

~~~swift
enum MyOptional {
    case Some
    case None
}

let nothing = MyOptional.None
let something = MyOptional.Some
~~~


## Enumerações com valor associado

Uma das funcionalidades mais legais de Swift na minha opnião é poder criar [enums com valor associado](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Enumerations.html#//apple_ref/doc/uid/TP40014097-CH12-ID145) e com isso é possível fazer um modelo mais realista (e usável!) de um `Optional`:


~~~swift
enum MyOptional {
    case Some(Any)
    case None
}

let something = MyOptional.Some(3)
~~~

O problema dessa abordagem é que perdemos toda a **magia** dos tipos: se checamos `something.dynamicType` a resposta obtida é `MyOptional.Type`. Isso não nos diz muito coisa.


# Genéricos

Outra funcionalidade excepcional de Swift são os [genéricos](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Generics.html#//apple_ref/doc/uid/TP40014097-CH26-ID179): de forma (incrivelmente) resumida, genéricos permitem que você trabalhe com uma estrutura de forma genérica (faz sentido, não?) e reusável que funcionem em cima de qualquer tipo!


~~~swift
enum MyOptional<T> {
    case Some(T)
    case None
}

let something = MyOptional.Some(3)
~~~

Agora ao checar `something.dynamicType` a resposta obtida é `MyOptional<Int>.Type`! Bem mais interessante, não?

Se tentarmos criar uma instância de nada agora usando `let nothing = MyOptional.None` obtemos o seguinte erro:

>**Generic parameter '**`T`**' could not be inferred**

E isso **faz muito sentido**: na nossa abstração queremos expressar a ausência de um valor, mas de um valor de que tipo? Se esse tipo fosse `Int`, por exemplo, poderiamos fazer `let nothing = MyOptional<Int>.None`.

## Inicializadores de `Optional`

### Nada
Criar um `Optional` é tão simples quanto `Optional<Int>()` ou `Optional<MyNeatType>()`, porém se tentarmos fazer algo como `MyOptional<MyNeatType>()` obtemos um erro:

> *error:* '`MyOptional<MyNeatType>`' cannot be constructed because it has no accessible initializers

Podemos resolver isso facilmente criando um inicializador!

~~~swift
init() {
	self = .None
}
~~~

### Alguma coisa

Analogamente obteríamos um erro se tentassemos fazer:

~~~swift
let foo = MyNeatType()
let bar = MyOptional<MyNeatType>(foo)
~~~

> *error:* argument passed to call that takes no arguments

Escrever um incializador para esse caso é simples e usa nosso tipo genérico `T`:

~~~swift
init(_ some: T) {
    self = .Some(some)
}
~~~

# Optionals como caixas

![]({{ site.baseurl }}/img/fpg1503/box.png)

Definindo desse jeito, `Optional`s se mostram como excelentes caixas: pode haver um valor dentro, mas só saberemos ao abrir a caixa (ou desembrulhar o valor). Uma das excelentes belezas disso é que `MyOptional<Int>` é **intrinsecamente diferente** de `Int` e, enquanto somar dois `Int`s faz sentido, tentar somar dois Optionals mostrará que só devemos efetuar essa operação se ambos existirem. Mas primeiro **é necessário checar**.

Esse tipo de questionamento é exatamente o que não fazíamos em Objective-C (e se fazíamos, não tínhamos como expressar). Como saber se um método de Objective-C pode retornar ou receber `nil`? Olhe a documentação, se você tiver sorte estará lá. Isso mudou um pouco com as [anotações de nulabilidade](https://developer.apple.com/swift/blog/?id=25) mas ainda não é parte de nosso mindset.


## Desembrulhando

Desembrulhar nossos Optionals é bem simples! Eles são enumerações e trabalhar com enumerações em Swift é **incrivelmente prazeroso**:

### switch
~~~swift
let optionalNumber = MyOptional<Int>(3)
switch optionalNumber {
case .None:
    print("No number ¯\\_(ツ)_/¯")
case .Some(let number):
    print("\(number), Numberwang!")
}
~~~

### Pattern Matching

~~~swift
let optionalNumber = MyOptional<Int>(42)
if case .Some(let number) = optionalNumber {
    print("\(number), Numberwang!")
}
~~~

Essa sintaxe é muito parecida com nosso tão amado `if let`, não é mesmo? Isso acontece pois **Optionals são enums!** Sim, sua vida é uma mentira! Optionals não passam de um enum e açúcar sintático!


# Açúcar sintático e Optionals

Como dissemos lá em cima, Optionals estão incrivelmente enraizados em Swift e isso é possível graças a **muito** açúcar sintático (quase um canavial sintático 😝). Vamos ver agora o que conseguimos reproduzir e o que é açucar sintático:


## `if let`
O `if let` funciona se houver um caso `.Some` com um valor associado! Sim! Podemos usar `if let` com `MyOptional`, não é lindo? 😁

~~~swift
let optionalNumber = MyOptional<Int>(8001)
if let number = optionalNumber {
    print("\(number), Numberwang!")
}
~~~

## `var optionalNumber: MyOptional<Int> = nil`

Consigo criar Optionals que representam a ausência de um valor usando `nil`. Isso é facilmente implementado! Swift tem `LiteralConvertibles` que, de forma bem resumida, são coisas que podem ser criadas a partir de um [literal](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/zzSummaryOfTheGrammar.html#//apple_ref/doc/uid/TP40014097-CH38-ID458). 

Há um artigo fenomenal do [@mattt](https://twitter.com/mattt){:target="_blank"} no [NSHipster sobre Swift Literal Convertibles](http://nshipster.com/swift-literal-convertible/) então não entrarei em muitos detalhes aqui. Como brinde se você sempre se perguntou **o que ocorre quando uso um literal** recomendo fortemente dar uma olhada nos [docs do Swift](https://github.com/apple/swift/blob/master/docs/Literals.rst).

### `NilLiteralConvertible`
Como o próprio nome sugere `NilLiteralConvertible`s são coisas que podem ser criadas a partir de um `nil`. É um protocolo que define uma única função:

~~~swift
protocol NilLiteralConvertible {
    init(nilLiteral: ())
}
~~~

Nativamente este protocolo é implementado por `Optional` e `ImplicltlyUnwrappedOptional`. A título de curiosidade, ele também é usado por `UnsafePointer`, `UnsafeMutablePointer`, `AutoReleasingUnsafeMutablePointer` e `COpaquePointer` mas eles fogem do escopo deste artigo.


Como vimos, basta criar uma extensão para nosso `enum` que lida o caso. Notem que **a ausência é representada pela tupla vazia**:

~~~swift
extension MyOptional: NilLiteralConvertible {
    init(nilLiteral: ()) {
        self = .None
    }
}
~~~

## O que não conseguimos recriar

### Ponto de Interrogação

O ponto de interrogação como sufixo de um tipo é puro açúcar sintático e por isso não é possível usá-lo para nosso Optional 😕

### Inicializadores Falíveis

Agora que entendemos melhor o que são Optionals, vemos que inicializadores falíveis não fazem muito sentido! Se a única maneira de eu representar a ausência de um valor é com `Optional` então inicializadores falíveis seriam impossíveis. 

Considere que um inicializador sempre retorna o tipo dele: como retornaríamos outro tipo!? Tomemos como exemplo um dos mais famosos inicializadores falíveis: o de `NSURL`: 

~~~swift
class NSURL {
    public convenience init?(string URLString: String)
}
~~~

O código abaixo imprime `Optional<NSURL>`

~~~swift
let url = NSURL(string: "")
print(url.dynamicType)
~~~

Ou seja: inicializadores falíveis **não são inicializadores do seu tipo!** 😱. Eles são puro açúcar sintático: uma coisa que deixa isso bem evidente é o erro obtido ao tentar fazer algo como:



~~~swift
private init() {}

init?(password: String) {
    guard name == "tijolo22" else {
        return .None
    }
    self = SecretObject()
}
~~~

> `nil` is the only return value permitted in an initializer

O que mostra que o valor de retorno não está de fato sendo usado! Simplesmente é checado se ele é `nil`. Trocar `.None` para `nil` faz o código funcionar:

~~~swift
private init() {}

init?(password: String) {
    guard name == "tijolo22" else {
        return nil
    }
    self = SecretObject()
}
~~~


# Optionals são <s>mônadas</s> contêineres

Sim, Optionals são mônadas e há uma excelente talk sobre mônadas chamada [Monads are not Monsters](https://www.youtube.com/watch?v=vg7cOF30Svo) da UIKont de 2015 (obrigado [@talesp](https://twitter.com/talesp){:target="_blank"} pela recomendação!) mas para simplificar vamos só dizer que Optionals são contêineres.

Em contêineres podemos implementar `map` e `flatMap`, de maneira resumida:

- `map(f)` aplica uma função `f` a cada valor contido no contêiner e insere os resultados em um novo contêiner.
- `flatMap` faz a mesma coisa porém ao final "achata" o contêinter, ou seja, cria um contêiner com o conteúdo de seus sub-contêineres.

Pensando em listas, achatar `[[1, 2], 3, [4, [5]]]` produz `[1, 2, 3, 4, [5]]` (notem que apenas uma camada é achatada).

## Mas alguém usa isso?
Sim! Antigamente achava que nem, mas cada vez mais vejo colegas usando `map` e `flatMap` para reduzir mutabilidade dentro de funções e criar códigos mais expressivos. É estranho no começo, mas depois de pouquíssimo tempo você vai falar: *como eu vivi até hoje sem isso?*.

## `map`

Implementar `map` é simples: se há um valor, retornamos o valor da aplicação de `f` nele. Senão, retornamos `.None`:

~~~swift
func map<U>(f: (T -> U)) -> MyOptional<U> {
    guard let value = self else {
        return .None
    }
    let mappedValue = f(value)
    return .Some(mappedValue)
}
~~~

Porém, há um problema: isso não funcionaria se `f throws`. Isso é facilmente resolvido usando `rethrows`:

~~~swift
func map<U>(f: (T throws -> U)) rethrows -> MyOptional<U> {
    guard let value = self else {
        return .None
    }
    let mappedValue = try f(value)
    return .Some(mappedValue)
}
~~~

Por ser de um idioma funcional não queremos que nosso `map` seja usado por seu efeito colateral, ou seja, não queremos que seu resultado seja ignorado. Para garantir isso basta incluir `@warn_unused_result` antes da declaração da função e deixar o compilador fazer sua mágica! 😻

## `flatMap`

Poderíamos implementar o `flatMap` usando `map` e desembrulhando o valor. Porém, é mais fácil fazer uma implementação análoga à do `map` sem reembrulhar o retorno:

~~~swift
@warn_unused_result
func flatMap<U>(f: (T throws -> MyOptional<U>)) rethrows -> MyOptional<U> {
    guard let value = self else {
        return .None
    }
    let mappedValue = try f(value)
    return mappedValue
}
~~~

# `ImplicitlyUnwrappedOptionals`

`ImplicitlyUnwrappedOptional` é o **irmão malvado** do `Optional`. Ele é como um `Optional` mas, como o nome sugere, você consegue acessar seu valor sem precisar desembrulhá-lo. O problema disso é: se o valor não existe o app crasha. A principal razão de sua existência é para ponte com Objective-C.

A parte boa é: você consegue usar ele como um `Optional`, ou seja, é possível fazer o desembrulho condicional, *Optional chaining* e até mesmo usar *nil coalescing*. 

## IBOutlet

O Xcode gosta de nos atrapalhar. Um dos jeitos dele de fazer isso excepcionalmente bem é: quando criamos IBOutlets eles por padrão são `ImplicitlyUnwrappedOptional`s. Você poderia contra-argumentar que se funciona na sua máquina vai funcionar sempre e isso é o Xcode incentivando *fail-fast* para evitar que outlets sejam erroneamente desligados. Normalmente eu concordaria com você, mas depois de ver **diversos crashes** (em projetos diferentes) por IBOutlets que estavam `nil` eu preferiria parar de arriscar.


Como mencionei antes eu poderia simplesmente tratá-los como `Optional`s, mas como eu quero incentivar todos do meu time a fazerem isso criei um [pluginzinho](http://github.com/fpg1503/OptionalOutlets/) para deixá-los `Optional` automaticamente para mim! Se você preferir pode arrumar um por um, basta trocar o `!` por um `?`.


# Optionals e boas práticas 


## Evitar a *Piramyd of Doom*


Vamos supor o seguinte caso

~~~swift
var a: String? = "one"
var b: String? = "two"
var c: String? = "three"
~~~

Antigamente a única maneira de lidar com isso era:

~~~swift
if let aUnwrapped = a {
  if let bUnwrapped = b {
    if let cUnwrapped = c {
      println("\(aUnwrapped) - \(bUnwrapped) - \(cUnwrapped)")
    }
  }
}
~~~

Porém a partir do Swift 1.2 podemos simplesmente fazer todos os desembrulhamentos de uma só vez:

~~~swift
if let aUnwrapped = a, bUnwrapped = b, cUnwrapped = c {
  println("\(aUnwrapped) - \(bUnwrapped) - \(cUnwrapped)")
}
~~~

## Evitar o **force unwrap**

![]({{ site.baseurl }}/img/fpg1503/everytime.jpg)

O **force unwrap** (ou desembrulho forçado) é equivalente a dizer: **eu tenho certeza que tem uma coisa aqui!**. Se você estiver errado o app crasha.  

> Tenho um amigo que gosta de dizer que Optionals são caixas que podem ter bombas dentro.

Você abriria a caixa de uma vez ou faria um furinho primeiro para ver o que está lá? Imaginei...

Se seus crashes começarem a mostrar `EXC_BREAKPOINT` ou crashes na `linha 0` eu recomendaria começar procurando algum force unwrap. Fique atento pois ele pode estar acontecendo sem que você perceba através de um `ImplicitlyUnwrappedOptional`!

### Não confiar no Xcode

Como mencionamos antes, o Xcode gosta de nos atrapalhar: vive sugerindo que façamos o force unwrap, o que acaba gerando códigos como o abaixo:

`self?.collectionView?.indexPathsForSelectedItems()!`

Esse é o código que chamamos popularmente de **Swift Safadão**
![]({{ site.baseurl }}/img/fpg1503/99popt.jpg)

O problema dele é que se qualquer coisa não existir, haverá um crash. Não gostamos de crashes.

### Quando usar force unwrap
Idealmente? **Nunca**. Na prática? Depende do seu nível de desconfiança. Se você está criando uma `NSURL` a partir de uma `String` constante e funciona em dev provavelmente não haverá problemas em produção. Pessoalmente eu gosto de checar sempre, se der errado eu normalmente logo um erro **non-fatal** para entender o que está acontecendo.


## Nil coalescing

Muitas vezes a melhor maneira de lidar com um `Optional` é dando um valor para padrão para ele, para isso podemos usar o operador de nil coalescing:

~~~swift
let value = self?.someOptionalObject?.someOptionalValue ?? defaultValue
~~~

# Em suma

Opcionalidade é diferente de nulabilidade e tentar tratar os dois como a mesma coisa pode te levar a fazer muitos erros. `Optional` é um tipo e ele te dá *type-safety* do que pode não existir.

Optionals te trazem mais segurança e te livram de muita dor de cabeça. Vamos supor que haja uma sequência de `n` funções chamadas de maneira aninhada para processar um valor. Em linguagens sem Optionals, temos que tratar a ausência desse valor em todas as chamadas. Já nas com Optionals, basta tratar este caso nas mais externas. O fato de `Optional` ser um tipo distinto garante que nenhuma das outras chamadas sejam executadas com um valor que não existe.

O desembrulho forçado é incrivelmente perigoso e os opcionais desembrulhados implicitamente são **being evil for no reason**. Evite usá-los, seu *crash-free* (e seus usuários) agradecem.

Essa mudança veio por um motivo: devemos mudar nosso *mindset*. Quando começamos a pensar com Optionals o mundo fica mais lindo.

Optionals são caixas como a do experimento de Schrödinger: até abrir não temos certeza do estado de seu conteúdo.

# Imagens

> - As imagens dos Emojis foram fornecidas pelo [EmojiOne](http://emojione.com)
> - O gato da foto de capa do post é meu sobrinho [Mozzie](https://instagram.com/mozziethekitten){:target="_blank"})
