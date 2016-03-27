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

## Reactivecocoa
Reactivecocoa (RAC) nada mais é que um framework , inspirado pelo paradigma funcional e reativo de programação. Existem várias matérias explicando os conceitos de programação reativa e funcional (inclusive aqui no blog) , portanto vamos nos ater ao seu uso apenas para o uso de "bindings" na arquiterura MVVM .

## Bindings

Utilizar Reactivecocoa para efetuar bindings , significa obter apenas algumas adições , no mecanismo já existente de KVO em Objective-C. Existe algo novo, que o Reactivecocoa traz para o KVO ? Uma interface mais amigavel com certeza , adionando ainda , as habilidades de se descrever regras , de um estado de um modelo de dados  para o estado da interface de forma declarativa .

## MVVM
![]({{ site.baseurl }}/img/Delarge/MVVMPattern.png)

Model-View-ViewModel , ou MVVM como é conhecido , trata-se de um design pattern , que consiste em separar lógicas de UI, da lógica de negócios ,
tornando as aplicações mais faceis de serem desenvolvidas e testadas. Reúne todos os dados brutos que podem vir de banco de dados , webservices e etc , aplica lógicas , e formata estes dados para apresentação em uma viewcontroller. Expoe via properties somente a informação que a viewcontroller precisa saber para efetuar o trabalho de mostrar os dados em uma view .

## MVVM e binding de dados

![]({{ site.baseurl }}/img/Delarge/MVVMReactiveCocoa.png)

Within the context of MVVM, ReactiveCocoa performs a very specific role. It provides the ‘glue’ that binds the ViewModel to the View.
We have “updates” in our MVVM diagram as somewhat ambiguous. There’s nothing about MVVM that forces you to use specific mechanisms to update the view model, or the view. However, in the scope of this book, we’ll be using ReactiveCocoa.
ReactiveCocoa will monitor for changes in the model and map those changes to properties on the view model, performing any necessary business logic.
As a concrete example, imagine our model contains a date called dateAdded that we want to monitor for changes, and update our view model’s dateAdded property. The model’s property is an NSDate instance, while the view model’s is a transformed NSString. The binding would look something like the following (inside the view model’s init method).

~~~objc
RAC(self,dateAdded)=[RACObserve(self.model,dateAdded)
	map:^(NSDate *date){
	 return [[ViewModel dateFormatter] stringFromDate:date];
 }];
~~~
dateFormatter is a class method on ViewModel that caches an NSDateFormatter instance so it can be reused (they’re expensive to create). Then, the view controller can monitor the view model’s dateAdded property, binding it to a label.

~~~objc
RAC(self.label,text) = RACObserve(self.viewModel,dateAdded);
~~~

We’ve now abstracted the logic of transforming the date to a string into our view model, where we might write unit tests for it. It seems contrived in this trivial example, but as we’ll see, it helps reduce the amount of logic in your view controllers significantly.
