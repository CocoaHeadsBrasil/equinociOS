---
layout:     post
title:      "Auto Layout com Visual Format"
subtitle:   "Seu layout com uma linguagem visual e fácil"
date:       2016-03-21 00:00:00
author:     "Daniel Bonates"
header-img: "img/dbonates/header_bg.jpg"
category:   ios
---

## Uma linguagem visual e fácil que pode resolver problemas complexos

_Auto Layout_ ainda é um tópico delicado para alguns desenvolvedores. Seja no Interface Builder ou via código, definir o posicionamento e dimensões dos objetos satisfazendo todos os requerimentos para ele funcionar perfeitamente ainda pode ser um desafio. Acho justo comparar o _Auto Layout_ do iOS com o CSS, usado também para definições de layout, mas na platforma web. Eu mesmo já passei perrengues tentando ajustar um item sem quebrar o layout de outro; tipo o tal do _cobertor curto_.

<img src="{{ site.baseurl }}/img/dbonates/css.gif">


A primeira dica pra conseguir um resultado bom é: planejar. Digo sobre a disposição dos objetos, considerando as dimensões e distância entre eles. Pode parecer óbvio, mas o mais intuitivo é você observar a imagem, começar a inseri-los na tela e depois dar conta de _layoutar_ eles.

## Um linguagem simples, reduzida e... visual!
Os métodos tradicionais para aplicar constraints no seu layout via código nem são complicados, e ainda te dão a oportunidade de inserir constraints individuais para cada item, basta um pouco de paciência e seu esforço será recompensado. Ainda assim, existe uma alternativa pouco explorada que pode facilitar ainda mais nosso trabalho, é a _Visual Format Language_.

Trata-se de uma linguagem muito simples e resumida, mas com um poder de fogo considerável. Mas o mais legal é que para usá-la, basta uma string definindo os elementos na sequência em que eles aparecem na tela, considerando os eixos horizontal e/ou vertical `"H:[view1]-30-[view2]"` e pronto, funciona! 

É como **desenhar seu layout usando apenas texto**! Vamos a um exemplo simples.

## Mão na massa

Digamos que nosso objetivo é preencher a view principal com um retângulo. Esse código cria a view:

```swift
let v1 = UIView(frame: CGRectZero)
v1.backgroundColor = UIColor(red:0.5134,  green:0.8424,  blue:0.9366, alpha:0.6)
v1.translatesAutoresizingMaskIntoConstraints = false
self.view.addSubview(v1)
```

Observações importantes:
- Note que eu não defini o frame que desejamos para a _v1_. Vamos deixar o foco do nosso assunto tratar disso mais além!
- Outro detalhe fácil de passar despercebido, é a propriedade `translatesAutoresizingMaskIntoConstraints`. Ela precisa ser anulada uma vez que você vai definir as constraints manualmente. Sem isso, o iOS irá criá-las automaticamente usando suas dimensões iniciais como parâmetro. 

Uma vez tendo _v1_ sido adicionada à view principal, vamos alinhá-lo com nossas intenções de layout:

```swift
self.view.addConstraints(NSLayoutConstraint.constraintsWithVisualFormat("H:|[v1]|", options: [], metrics: nil, views: ["v1":v1]))
self.view.addConstraints(NSLayoutConstraint.constraintsWithVisualFormat("V:|[v1]|", options: [], metrics: nil, views: ["v1":v1]))
```

O resultado:

<img src="{{ site.baseurl }}/img/dbonates/full.png">

Eu sei, sem graça. Mas você conseguiu o que queria, não?! Lição brinde: programar é assim, você conquista o que você faz!

### Incrementando de leve
Vamos adicionar uma margem = 20 nesse layout. Mudaremos a string de `H:|[v1]|` para:

```swift
H:|-20-[v1]-20-|
```

e `V:|[v1]|` para:

```swift
V:|-20-[v1]-20-|
```

Aí teremos como resultado:

<img src="{{ site.baseurl }}/img/dbonates/m20.png">

Está começando a nascer :)

### Quebrando em partes

O método `constraintsWithVisualFormat` é o que usamos pra aplicar a linguagem visual do _Auto Layout_. Ela tem alguns parâmetros interessantes que aumentam seu poder de fogo, mas eu quero me concentrar aqui no principal, o que usamos e que é o coração desse recurso.

Usamos `"H:|-20-[v1]-20-|"`, mas a que se refere essa string? Vamos lá:

| _string_ | utilização |
|:-----------------|---------------|
|`H` | significa que estamos tratando os elementos no layout no eixo horizontal |
|`|` | esse _pipe_ faz referência à superview |
|`-20-` | especificamos um espaçamento igual a 20 entre os elementos à esquerda e à direira (no caso do eixo horizontal) |
`[v1]` | essa é a view sobre a qual estamos nos referindo na aplicação das constraints.|
| `["v1":v1]` | Dicionário contendo as views citadas na string de layout |


### Nota importante:
Não tente adicionar as constraints antes de adicioná-las à view onde vão estar contidas, caso contrário um crash irá ocorrer dado que `v1` não será encontrado em `self.view` ainda.

### Múltiplas views no layout

Começando a ficar mais sério o papo, vamos adicionar mais views. Mantenha sua observidade (eu inventei essa palavra agora, significa "fique observando, stay focused"), em como tratar o layout continua sendo simples e visual.

Esse é o código completo até agora com as duas views em nosso layout

```swift
let v1 = UIView(frame: CGRectZero)
v1.backgroundColor = UIColor(red:0.5134,  green:0.8424,  blue:0.9366, alpha:0.6)
v1.translatesAutoresizingMaskIntoConstraints = false
self.view.addSubview(v1)
        
let v2 = UIView(frame: CGRectZero)
v2.backgroundColor = UIColor(red:0.969,  green:0.443,  blue:0.455, alpha:0.8)
v2.translatesAutoresizingMaskIntoConstraints = false
self.view.addSubview(v2)

// constraints para a view 1
self.view.addConstraints(NSLayoutConstraint.constraintsWithVisualFormat("H:|-20-[v1]-20-|", options: [], metrics: nil, views: ["v1":v1]))
self.view.addConstraints(NSLayoutConstraint.constraintsWithVisualFormat("V:|-20-[v1(==300)]", options: [], metrics: nil, views: ["v1":v1]))

// constraints para a view 2        
self.view.addConstraints(NSLayoutConstraint.constraintsWithVisualFormat("H:|-20-[v2]-20-|", options: [], metrics: nil, views: ["v2":v2]))
self.view.addConstraints(NSLayoutConstraint.constraintsWithVisualFormat("V:[v1]-20-[v2]-20-|", options: [], metrics: nil, views: ["v1":v1, "v2":v2]))
```
Esse é o resultado:

<img src="{{ site.baseurl }}/img/dbonates/double.png">

Nesse trecho, além de adicionar a própria view:

1. temos um `==300` na definição de constraints para a _view 1_. É desse jeito que podemos definir um tamanho fixo ou variável para a view em questão. Por exemplo, ao invés de `==` poderíamos ter usado `>=`.

2. Outro detalhe fica por conta de termos adicionados as duas views no parâmetro que relaciona as views envolvidas num dicionário: `["v1":v1, "v2":v2]`.

3. E por último, a definição da relação entre as duas views definindo a posição da _view 2_ com relação à _view 1_, a uma distancia de 20 pixels: `V:[v1]-20-[v2]-20-|`

Ainda é um exemplo simples, mas você já consegue imaginar onde isso pode parar? Se estamos definindo uma string, por exemplo, que tal se ela embutir valores de variáveis? Exemplo:

```swift
let padding:CGFloat = 10.0
```
poderíamos alterar a string de visual format para:

```swift
"V:[v1]-\(padding)-[v2]-\(padding)-|"
```
Isso pode ir mais longe:

```swift
// constraints para a view 1
self.view.addConstraints(NSLayoutConstraint.constraintsWithVisualFormat("H:|-\(pad)-[v1]-\(pad)-|", options: [], metrics: nil, views: ["v1":self.v1]))
self.view.addConstraints(NSLayoutConstraint.constraintsWithVisualFormat("V:|-\(pad)-[v1(==v2)]-\(pad)-[v2]", options: [], metrics: nil, views: ["v1":self.v1,"v2":self.v2]))

// constraints para o label dentro de view 1
v1.addConstraints(NSLayoutConstraint.constraintsWithVisualFormat("H:|-[swipeUpInfo]-|", options: [], metrics: nil, views: ["swipeUpInfo" : swipeUpInfo]))
v1.addConstraints(NSLayoutConstraint.constraintsWithVisualFormat("V:|-[swipeUpInfo]-|", options: [], metrics: nil, views: ["swipeUpInfo" : swipeUpInfo]))

// constraints para a view 2
self.view.addConstraints(NSLayoutConstraint.constraintsWithVisualFormat("H:|-\(pad)-[v2]-\(pad)-|", options: [], metrics: nil, views: ["v2":self.v2]))
self.view.addConstraints(NSLayoutConstraint.constraintsWithVisualFormat("V:[v1]-\(pad)-[v2(==v3)]-\(pad)-[v3]", options: [], metrics: nil, views: ["v1":self.v1,"v2":self.v2,"v3":self.v3]))

// constraints para a view 3
self.view.addConstraints(NSLayoutConstraint.constraintsWithVisualFormat("H:|-\(pad)-[v3]-\(pad)-|", options: [], metrics: nil, views: ["v3":self.v3]))
self.view.addConstraints(NSLayoutConstraint.constraintsWithVisualFormat("V:[v2]-\(pad)-[v3(==v4)]-\(pad)-[v4]", options: [], metrics: nil, views: ["v2":self.v2,"v3":self.v3,"v4":self.v4]))

// constraints para a view 4
self.view.addConstraints(NSLayoutConstraint.constraintsWithVisualFormat("H:|-\(pad)-[v4]-\(pad)-|", options: [], metrics: nil, views: ["v4":self.v4]))
self.view.addConstraints(NSLayoutConstraint.constraintsWithVisualFormat("V:[v3]-\(pad)-[v4(==v3)]-\(pad)-|", options: [], metrics: nil, views: ["v3":self.v3,"v4":self.v4]))

// constraints para o label dentro de view 4
v4.addConstraints(NSLayoutConstraint.constraintsWithVisualFormat("H:|-[swipeDownInfo]-|", options: [], metrics: nil, views: ["swipeDownInfo" : swipeDownInfo]))
v4.addConstraints(NSLayoutConstraint.constraintsWithVisualFormat("V:|-[swipeDownInfo]-|", options: [], metrics: nil, views: ["swipeDownInfo" : swipeDownInfo]))     
```

vai produzir isso:
<img src="{{ site.baseurl }}/img/dbonates/fourbars.png">


## Vai um desafio?
A imagem a seguir, é uma célula dentro de uma CollectionView. Toda usando Visual Format Language:

<img src="{{ site.baseurl }}/img/dbonates/cell.png">

Acredite, esses elementos todos podem ser posicionados e dimensionados usando o que citamos por aqui. Ao todo, 22 linhas de código aplicando as strings de layout mais algumas definindo variáveis, como _ratio_, _largura_, etc.

## Links relacionados e código fonte desse artigo

A documentação da Apple sobre Visual Format Language é bem razoável e está nesse link, vale uma visita:
[https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/AutolayoutPG/VisualFormatLanguage.html](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/AutolayoutPG/VisualFormatLanguage.html)

### Sobre código fonte
O código fonte desse projeto vai um pouco além do foi postado aqui, mas usando tudo que falamos, além de alguns adornos como usar o gesto swipe para alterar o valor usado para padding. Fique à vontade para explorá-lo, o link é esse:

[https://github.com/dbonates/Auto-Layout-with-Visual-Format](https://github.com/dbonates/Auto-Layout-with-Visual-Format)


## Conclusão
Trabalhar com _Auto Layout_ pode ser muito bom e rápido se você usar toda versatilidade que a ferramenta oferece, e o que abordei aqui é apenas um dos recursos, porém muito interessante e pouco explorado. É possível fazer muita coisa usando a linguagem visual da Apple para _Auto Layout_. E eu espero que você tenha gostado e no mínimo tenha percebido que se bem compreendido, ele joga a seu favor, e sem muito esforço, ao contrário do que diz a lenda :P

Bem é isso, pessoal.

Críticas, comentários e sugestões? À vontade e sem neurose!

Um forte abraço a todos e até a próxima!

> Daniel Bonates<br>designer & developer -  <a href="http://bonates.com" target="_blank">bonates.com</a>
