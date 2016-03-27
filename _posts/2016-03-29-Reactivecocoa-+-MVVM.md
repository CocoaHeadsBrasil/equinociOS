---
layout:     post
title:      "Reactivecocoa + MVVM"
subtitle:   "Utilizando Reactivecocoa e MVVM"
date:       2016-03-12 00:00:00
author:     "Alessandro Santos"
header-img: "img/home-bg.jpg"
category:   Regex
---

> Alessandro Santos ([@andydelarge](https://twitter.com/andydelarge){:target="_blank"}) Carioca , desenvolvedor iOS desde 2011 , atualmente faz parte (orgulhosamente) do time iOS da 99 Taxis, e esta prestes a realizar o sonho em Trabalhar fora do pais .

## Introdução
A utilização dos paradigmas de linguagens funcional e/ou reativas parecem estar "na moda" já a algum tempo, em linguagens como scala, ou até mesmo com o  lançamento do swift . O que a maioria parece não saber, ou ignorar, é que "qualquer" linguagem permite o uso destes conceitos , sendo que algumas dão um "pouco mais de trabalho" para utilizar estas técnicas . O seu uso muitas vezes pode parecer como conhecmos "uma bazuca para matar formiga" e embora possamos nos beneficiar destas tecnicas escrevendo menos código , o seu uso deve ser bem avaliado.  Para começarmos de forma simples , vamos ver um pouco do seu uso como bindings utilizando o framework Reactivecocoa em um padrão de arquitetura conhecido como MVVM.


## Reactivecocoa
Reactivecocoa (RAC) nada mais é que um framework , inspirado pelo paradigma funcional e reativo de programação. Existem várias matérias explicando os conceitos de programação reativa e funcional (inclusive aqui no blog) , portanto vamos nos ater ao seu uso apenas para o uso de "bindings" na arquiterura MVVM .

## Bindings
Utilizar Reactivecocoa para efetuar bindings , significa obter apenas algumas adições , no mecanismo já existente de KVO em Objective-C. Existe algo novo, que o Reactivecocoa traz para o KVO ? Uma interface mais amigavel com certeza , adionando ainda , as habilidades de se descrever regras , de um estado de um modelo de dados  para o estado da interface de forma declarativa .

## MVVM
![]({{ site.baseurl }}/img/Delarge/MVVMPattern.png)

Model-View-ViewModel , ou MVVM como é conhecido , trata-se de um design pattern , que consiste em separar lógicas de UI, da lógica de negócios ,
tornando as aplicações mais faceis de serem desenvolvidas e testadas. Reúne todos os dados brutos que podem vir de banco de dados , webservices e etc , aplica lógicas , e formata estes dados para apresentação em uma viewcontroller. Expõe via properties somente a informação que a viewcontroller precisa saber para efetuar o trabalho de mostrar os dados em uma view .

## MVVM e binding de dados

![]({{ site.baseurl }}/img/Delarge/MVVMReactiveCocoa.png)

No contexto deste post , de utilizar ReactiveCocoa somente para binding de dados , usando a arquitetura MVVM , o ReactiveCocoa executa uma função muito especifica. Ele fornece uma espécie de "ponte" da viewmodel para a view ,  monitorando todas as mudanças realizadas no modelo de dados, e mapeando estas mudanças para as properties da viewmodel efetuando qualquer lógica de negócios necessária .
Para dar um exemplo , imagine que nosso modelo de dados , contenha uma property NSDate chamada de dateAdded, e nós iremos monitorar suas mudanças , para  atualizar outra property também chamada de dateAdded, mas esta, do nosso viewmodel. A property dateAdded do nosso modelo , como dito anteriormente , será uma NSDate enquanto a do nosso viewmodel será uma NSString. O binding irá parecer com algo descrito abaixo (Dentro do metodo init em nossa viewmodel) :

~~~objc
RAC(self,dateAdded)=[RACObserve(self.model,dateAdded)
	map:^(NSDate *date){
	 return [[ViewModel dateFormatter] stringFromDate:date];
 }];
~~~

dateFormatter é um metodo de classe em nosso viewmodel , que faz uso de cache de uma instância NSDateFormatter que estará pronta para reuso (NSDateFormatter são muito custosas para se criar a todo tempo). Então nossa viewcontroller poderá monitorar todas as mudanças da nossa property dateAdded da nossa viewModel, podendo exibi-la em uma label,  por exemplo.


~~~objc
RAC(self.label,text) = RACObserve(self.viewModel,dateAdded);
~~~

Aqui nós estamos abstraindo toda a lógica de se transformar um NSDate para uma NSString para a nossa viewmodel , onde nós podemos escrever testes unitários para ela . Pode parecer um pouco confuso neste exemplo e um tanto quanto trivial como podemos ver , mas como vimos isso ajuda a reduzir significamente a quantidade de código de lógica de UI de nossas viewcontrollers.

## E agora ? Como proceder ?
As utilizações do Reactivecocoa vão MUITO além de simples bindings . Seu conteúdo é bastante vasto e sua curva de aprendizado bastante alta . Desde simples bindings até camadas de conexão , vale a pena o seu estudo , por tornar sua aplicação reativa com todas as vantagens que isto pode oferecer .   

## Algumas (valiosas) Fontes de informação
* [Introduction to MVVM by Ash Furrow ](https://www.objc.io/issues/13-architecture/mvvm/)
* [A sample app by Ash Furrow](https://github.com/AshFurrow/C-41)
* [MVC, MVVM, FRP, And Building Bridges by Jonathan Penn](http://cocoamanifest.net/articles/2013/10/mvc-mvvm-frp-and-building-bridges.html)
* [MVVM Tutorial with ReactiveCocoa](https://www.raywenderlich.com/74106/mvvm-tutorial-with-reactivecocoa-part-1)
* [Basic MVVM with ReactiveCocoa by Colin Wheeler](http://cocoasamurai.blogspot.com.br/2013/03/basic-mvvm-with-reactivecocoa.html)
* [On MVVM, and Architecture Questions by Chris Trott](http://twocentstudios.com/2014/06/08/on-mvvm-and-architecture-questions/)
* [JaviLorbada/FRP iOS Learning resources](https://gist.github.com/JaviLorbada/4a7bd6129275ebefd5a6)
