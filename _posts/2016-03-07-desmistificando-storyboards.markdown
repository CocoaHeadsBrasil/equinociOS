---
layout:     post
title:      "Desmistificando Storyboards"
subtitle:   "Ganhe produtividade sem perder sua sanidade"
date:       2016-03-07 00:00:00
author:     "Rafael Nobre"
header-img: "img/home-bg.jpg"
category:   storyboards
---

> Rafael Nobre é desenvolvedor iOS desde 2009, tendo trabalhado com UIs programáticas, XIBs, Storyboards, tanto com frames calculados e autosizing masks ou AutoLayout.  
> Durante este mês do equinociOS, aproveita para abrir sua mente com os temas propostos e estudar muita coisa nova.

# Interface Builder

Desde o período de Beta da primeira SDK do então iPhoneOS, em 2008, a Apple já havia disponibilizado e recomendado o uso de sua aplicação de construção de interfaces gráficas - o *Interface Builder* ou apenas *IB*.  
Sua função é representar uma hierarquia de *views* em um formato *XML* interno, permitindo visualmente manipular suas características e comportamento sem precisar utilizar código fonte *Objective-C* ou *Swift*.  

## A vantagem
A principal vantagem a ser citada é o ganho em produtividade. Não que escrever *UI* via código seja difícil, mas pode ser um processo tedioso e de muitos ciclos de *tweak*, *build*, *run*. Já que, na maior parte das vezes, código de *UI* é *trivial* e *repetitivo*, congelá-lo em um *resource* pode ser visto como uma vantagem.

## A desvantagem
É evidente que, por ser uma abstração construída sobre as classes de *UIKit*, pode se dizer que toda *UI* construída via *IB* pode ser construída via código, mas o inverso nem sempre é verdadeiro. É preciso ser criterioso e conhecer as ferramentas que estão a disposição do desenvolvedor, escolhendo a forma que melhor se adeque à situação apresentada.

# XIBs vs Storyboards
Em sua concepção inicial, o *IB*, que até o *Xcode 3* não era integrado à IDE, atuava em arquivos chamados *NIBs* (versão binária do formato, que posteriormente foi modificada para os atuais *XIBs*, de versionamento mais conveniente). Sua unidade de trabalho mínima é a *UIView*.  
Já as Storyboards chegaram com o iOS 5, e o diferencial é poder lidar com a construção da hierarquia de diversas *views* e a navegação entre elas, sendo o *view controller* sua unidade de trabalho mínima.  

## O problema das Storyboards
Como toda novidade da Apple, muitos desenvolvedores rapidamente adotaram as *Storyboards* e - infelizmente - com o suporte e recomendação da própria *Apple*¹, passaram a utilizá-la indiscriminadamente em seus novos projetos. A terrível *Main.storyboard* agora era promovida como detentora de *toda* a *UI*, independente do tamanho do *app*, seus diferentes fluxos de interação e casos de uso. Este é o resultado:  

![]({{ site.baseurl }}/img/nobre84/sb-nightmare.jpg)
<span class="caption text-muted">Storyboard nightmare</span>

Além da clara questão organizacional, outro ponto culminante para criar a má fama das Storyboards na comunidade iOS é o fato de que, ao trabalhar em apenas um arquivo monolítico compartilhado em um sistema de versionamento de código, os conflitos se tornam constantes e difíceis de resolver. Pra completar a revolta, o trabalho torna-se difícil em monitores menores, além de degradar a performance do Mac.

## Tem jeito?
A boa notícia é que sim, tem jeito. No primeiro projeto que iniciei após a chegada das *Storyboards*, todos esses problemas já haviam chegado à tona. Foi quando, pesquisando sobre boas práticas e se valeria a pena utilizar, li o artigo [UIStoryboard Best Practices](http://robsprogramknowledge.blogspot.com.br/2012/01/uistoryboard-best-practices.html), que ajudou a moldar minha forma de pensar sobre como modularizar os fluxos e casos de uso de um aplicativo em diversas *Storyboards*. Isto era 2012 minha gente, e a *Apple* até 2014 ainda recomendava fortemente usar uma única *Storyboard*, sendo que sua solução de modularização só foi ser lançada no iOS 9. Para dar suporte a modularização no iOS 7+, recomendo [RNExternStoryboard](https://github.com/nobre84/RNExternStoryboard) ou [RBStoryboardLink](https://github.com/rob-brown/RBStoryboardLink).  
Ao separar seus fluxos de trabalho em unidades coesas, mitigamos esses impedimentos, permitindo-nos aproveitar de alguns benefícios: prototipagem extremamente rápida, maior facilidade no entendimento do funcionamento e fluxo do app pela equipe, a possibilidade de utilizar *table views* estáticas etc.

# Strings, strings, strings
Um dos problemas que afligem o desenvolvedor iOS desde os primórdios são as **hard-coded** strings. No caso das *Storyboards*, como o desenvolvedor precisa referenciar não apenas o *view controller*, como também a representação de sua *view*, que está no *XML* da *Storyboard*, para instanciá-los programaticamente é preciso referenciar a *Storyboard* e o *Storyboard ID* do *controller*.  
Pra lidar com esse problema e outros de mesma natureza, vêm surgindo diversas ferramentas que facilitam fortemente a abolição das *hard-coded* strings. A proposta é nunca mais ter que referenciar imagens, *Storyboard IDs*, *segue identifiers*, *cell identifiers*, ou mesmo chaves de localização pelo seu identificador textual.  
Por exemplo, o [R.swift](https://github.com/mac-cain13/R.swift) propõe:

![]({{ site.baseurl }}/img/nobre84/rswiftdemo.gif)
<span class="caption text-muted">Autocomplete! <3</span>  

Já o [SwiftGen](https://github.com/AliSoftware/SwiftGen) aborda também Localização, criando um *enum* que faz parse de todas suas chaves localizáveis e seus possíveis parâmetros, tornando a internacionalização do *app* mais segura ao gerar um erro de compilação ao invés de falhar em *runtime*.  
No lado do *Objective-C*, ainda não vi um grande nome. O único que achei que se propôs a oferecer suporte foi o [referee](https://github.com/Dynamit/referee). Por não restringir as referências geradas ao seu próprio contexto, ainda estou à procura de uma solução adequada, ajude-nos nos comentários!

# Finalizando
O intuito é despertar no desenvolvedor o senso crítico: nos são propostas **ferramentas**. Como as utilizamos depende inteiramente de nós, e o *feedback* que trazemos à comunidade é justamente o que as farão evoluir para atender nossas crescentes necessidades.  
Se prender a uma ou outra tecnologia nos torna menos efetivos para desempenhar nosso papel de solucionadores de problemas. Meu posicionamento a respeito de quando usar cada ferramenta, no cenário atual, é: *views* programáticas quando estritamente necessário, por exemplo *views* extremamente dinâmicas; *XIBs* para abrigar *views* com potencial de reutilização, por exemplo células em *table views* e *collection views*, *storyboards* modulares para modelar o fluxo de navegação do *app*, cada tela sendo uma composição elementar das *views* reutilizáveis e as contextuais e/ou estáticas que se mostrem necessárias desenhadas diretamente na *storyboard*.  
Recomendo a leitura de alguns artigos que falam de como referenciar *views* desenhadas em *XIBs* na *Storyboard*, uma poderosa ferramenta para ganhar praticidade e reusabilidade nos *layouts* com *Storyboard*:  

* [Reusing views in storyboards with Auto Layout](http://cocoanuts.mobi/2014/03/26/reusable/)  
* [Xcode 6 Live Rendering from nib](http://justabeech.com/2014/07/27/xcode-6-live-rendering-from-nib/)

Um abraço, e até a próxima!
