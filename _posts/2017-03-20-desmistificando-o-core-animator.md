---
layout:     post
title:      "Desmistificando o Core Animation"
subtitle:   "Uma introdução para perder o medo desse framework como o Core Animation pode ser útil ao elaborar suas animações"
date:       2017-03-20 00:13:00
author:     "Renato Sarro Matos"
header-img: "img/renatosarro/header-coreAnimation.png"
category:   animations
---

Fala galera! Bem vindos a mais uma rodada do EquinociOS! Espero que estejam aproveitando e absorvendo tudo =D

Bem, o artigo de hoje introduz um assunto que eu particularmente sempre corri dele e tentei ao máximo evitá-lo. Até que chegou o dia em que eu não tive como escapar e precisei encarar o "bixo": `CORE Animation`.
Vou falar que foi uma das batalhas mais satisfatórias que eu já enfrentei nesse mundo de desenvolvimento mobile, pois além de eu conseguir trabalhar minhas animações de uma forma muito melhor, abriu minha mente para detalhes que sempre passaram desapercebidos e que mudou muito a minha forma de encarar uma - até então - simples estrutura de tela/layout.

### LETS TALK!

--

# Core Animation

Infraestrutura de renderização gráfica e animação, disponível para iOS e OS X, que você utiliza para animar as views e outros elementos visuais de sua aplicação.
(Sim, copiei do guide da Apple).
A propósito, vou pular esta parte introdutória (chata) que todo mundo está cansado de ver e pular pra parte "cool" da bagaça hehehe.

> Sempre leiam o guide da Apple ;) (https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CoreAnimation_guide/Introduction/Introduction.html)

Certo, acredito que todos vocês em algum momento já se depararam com uns métodos de classe chamados:

~~~swift
animate(...)
animateKeyframes(...)
~~~

~~~objc
animateWith...
animateKeyframesWith...
~~~

Estes, são métodos, ou melhor, `big helpers` que encapsulam todo o processo de criação de um `CABasicAnimation` ou de um `CAKeyframeAnimation` por exemplo. Estes métodos são muito úteis, desde que usados com moderação e um certo bom senso, mas após nossa introdução você vai notar que pode ter um resultado mais apropriado utilizando o `Core Animation`, dependendo da complexidade de suas animações.

Antes de começar a ver como as animações acontecem na tela, precisamos entender um elemento bem importante: O `CALayer`, que é de fato, onde as animações acontecem.

Certo, vamos voltar mais um passo e entender um pouco do comportamento de uma `UIView` em relação à animação.
Uma `instância do UIView` modifica sua layer para delegar a renderização à estrutura de animação principal. Entretanto, as animações, quando adicionadas a uma layer, não modificam suas propriedades. A animação principal, mantém duas `hierarquias` de layers: `model` e `presentation layers`.
Por exemplo, considerando uma animação de fade out, se em qualquer momento da animação você verificar o valor da opacidade da camada (`model layer`), este valor será diferente do valor que está sendo apresentado na tela (`presentation layer`).

Como falado anteriormente, não é possível definir as propriedades na presentation layer, mas você pode utilizar os valores para criar novas animações, ou interagir com outras camadas enquanto a animação acontece.

É possível acessar qualquer uma dessas camadas utilizando os métodos `[CALayer presentationLayer]` e `[CALayer modelLayer]`.

> Certo, vamos ver como tudo isso acontece na prática.

## CABasicAnimation

Vamos pensar em uma animação bem básica. A primeira que vem à minha mente é deslocar um quadrado de um lado para outro. Logicamente, quando eu configurar meu `CABasicAnimation`, vou passar para ele a propriedade que eu quero alterar, o valor inicial, o valor final e a duração:

<script src="https://gist.github.com/renatosarro/4753b9eb2dd7e22aa1383000514c4c1a.js"></script>

`keyPath`: KVC (Key value code) "position.x", contém um membro CGPoint armazenada na propriedade position :)

`fromValue`: Posição Inicial

`toValue`: Posição Final

`duration`: Duração da animação

> Ao rodar a animação, observamos que ao terminar a animação, nosso quadrado pula da posição final para a posição inicial. Isto ocorre, pois como falado anteriormente, as animações não modificam as propriedades na presentation layer. Então, ao terminar a animação, a posição padrão continua sendo a inicial.
Podemos contornar isso, modificando a propriedade position, diretamente na layer:

~~~objc
view.layer.add(animation, forKey: "basicAnimation")

view.layer.position = CGPoint(x: 320, y: 20)
~~~

Desta forma, ao terminar a animação, a posição do elemento (`presentation layer`) será correspondente à posição final da animação (`model layer`).

Uma outra abordagem é configurar duas propriedades do `CABasicAnimation`.

Este objeto possui uma propriedade chamada `fillMode` que determina o que vai acontecer ao final da animação. Vamos configurar esta propriedade com a string `kCAFillModeForwards` que vai dizer para a animação permanecer em seu estado final.
Vamos também configurar a propriedade `isRemovedOnCompletion` com o boolean false. Isto vai garantir que o objeto não seja removido automaticamente. Desta forma, nosso código ficará assim:

<script src="https://gist.github.com/renatosarro/396251ef6307475353b2ce3f718a58ee.js"></script>

> Utilize estas duas propriedades com muita parcimônia e as modifique apenas se houver uma real necessidade, pois quando trabalhamos com animações, é de estrema importância que as layers model e presentation estejam sincronizadas.
É importante ressaltar também, que o objeto da animação criada é copiado assim que adicionamos a alguma layer. Isto é bem interessante, pois podemos reutilizar o objeto em outras layers como por exemplo:

<script src="https://gist.github.com/renatosarro/c5d8881d055ad76b546e26ece695bec8.js"></script>

Observe que, como a animação foi copiada para a layer da primeira view, ao configurarmos a propriedade beginTime para começar meio segundo depois, apenas a animação da layer da `view2` é afetada, pois como falado, ao adicionar a animação à layer, é gerado uma cópia dela.


PS: Podemos gastar mais um artigo inteiro falando só sobre como gerenciar o tempo de nossas animações. Acredite, estamos apenas começando e o `Core Animation` possui uma infinidade de possibilidades para trabalhar as animações. Com certeza mais posts virão =D

--

### Agora, como seria esta mesma animação sem o uso do Core Animation?

<script src="https://gist.github.com/renatosarro/dcc083ec91dfee76c46b5c48c56fd048.js"></script>

>
- Desta forma, a animação impacta diretamente na posição real do elemento (`presentation layer`);
- Vale ressaltar que se tratando de animações, o mais indicado é trabalhar sempre na model layer. Alterações na presentation layer precisam ser feitas com muito cuidado para evitar memory leak e uma animação não muito fluída.
- Estaríamos limitados a animações simples, como um fade out. Mas vamos pensar em um cenário onde nosso elemento passe por vários estados até chegar em sua posição final. Utilizar essa abstração oferecida pela `UIView` já não seria muito indicado.

Já que falamos sobre vários estados, vou aproveitar o gancho para puxar um outro elemento bem bacana presente no Core `Animation`:

# CAKeyframeAnimation

Não sei quantos vieram desta época, mas minha primeira experiência como desenvolvedor foi com Action Script. Era o boom do Flash. Fantástico, brilhante, inovador, perfeito para ter experiências mais interativas e claro, animadas.

Bem, o conceito básico para qualquer animação é pensar nela em forma de `Key frames`. Já ouviram falar de Folioscópio, ou Flipbook?

É exatamente assim que uma animação funciona. Cada quadro (frame) possui o elemento em um estado diferente, onde colocando todos os quadros para passar de forma contínua, forma um desenho animado.

Sim, posso `e devo` utilizar o mesmo conceito ao elaborar minha animação.

Imagine que nossa view em vez de apenas se deslocar para a direta, ela vá até o centro, volte para o início e depois vá para o fim da tela.

O primeiro passo, é fazer um array de valores.

~~~swift
let values = [20, 160, 20, 320]
~~~

Depois disso, vamos criar um array de `timming`, que vai ser relativo aos itens de estado. Ou seja, para cada posição, teremos um tempo que irá dizer quanto vai levar para a nossa view estar naquela posição.

~~~swift
let times = [0, 0.5, 1, 2]
~~~

Perceba que esta lista diz que minha view começa na posição 20. Após 0,5 segundo, ela estará na posição 160. Após 1 segundo, na posição 20 e após 2 segundos, na posição 320. Ou seja, minha animação terá a duração de 2 segundos.

Vamos então criar nosso objeto `CAKeyframeAnimation`.

<script src="https://gist.github.com/renatosarro/7dd14ca9892d894653f8f185d18f6409.js"></script>

Podemos notar uma propriedade nova. A `isAdditive`. Configurando esta propriedade como verdadeiro, significa que o valor especificado pela animação, será adicionado à árvore de renderização atual. Isso nos permite reutilizar a mesma animação para qualquer elemento que precise ter o mesmo comportamento.

--

Olhando assim, o Core Animation não parece tão monstruoso hehe... Na verdade ele pode e vai auxiliar muito quando precisar trabalhar mais com o lado animado da coisa =D

Claro, esta, é apenas uma introdução. Com isso podemos ir brincando com keyPahts, keyFrames e ver o que podemos fazer de mais complexo exercitando muito estas propriedades básicas, mas tão importantes se tratando de animação.

Há uma infinidade de possibilidades quando falamos de animações e com certeza falaremos mais sobre isso.

# Carry on!




