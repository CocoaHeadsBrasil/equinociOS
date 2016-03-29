---
layout:     post
title:      "ReactiveCocoa + MVVM"
subtitle:   "Utilizando ReactiveCocoa e MVVM"
date:       2016-03-29 00:00:00
author:     "Alessandro Santos"
header-img: "img/home-bg.jpg"
category:   reactive
---

> Alessandro Santos ([@andydelarge](https://twitter.com/andydelarge){:target="_blank"}) Carioca, desenvolvedor iOS desde 2011, atualmente faz parte (orgulhosamente) do time iOS da 99 Táxis, e está prestes a realizar o sonho em Trabalhar fora do país.

## Introdução
A utilização dos paradigmas de linguagens funcional e/ou reativas parecem estar "na moda" já há algum tempo, em linguagens como Scala, ou até mesmo com o lançamento do Swift. O que a maioria parece não saber, ou ignorar, é que "qualquer" linguagem permite o uso destes conceitos, sendo que algumas dão um "pouco mais de trabalho" para utilizar estas técnicas. É importante ressaltar que o uso do ReactiveCocoa muitas vezes pode parecer como conhecemos "uma bazuca para matar formiga" e embora possamos nos beneficiar destas técnicas escrevendo menos código, o seu uso deve ser bem avaliado. Para começarmos de forma simples, vamos ver um pouco do seu uso como bindings em um padrão de arquitetura conhecido como MVVM.


## ReactiveCocoa
ReactiveCocoa (RAC) nada mais é que um framework, inspirado pelo paradigma funcional e reativo de programação. Existem várias matérias explicando os conceitos de programação reativa e funcional (inclusive aqui no blog), portanto, vamos nos ater ao seu uso apenas para o uso de "bindings" na arquiterura MVVM.

## Bindings
Utilizar ReactiveCocoa para efetuar bindings significa obter apenas algumas adições, no mecanismo já existente de KVO em Objective-C. Existe algo novo, que o ReactiveCocoa traz para o KVO? Uma interface mais amigável com certeza, adicionando ainda, as habilidades de se descrever regras, de um estado de um modelo de dados para o estado da interface de forma declarativa.

## MVVM
![]({{ site.baseurl }}/img/delarge/MVVMPattern.png)

Model-View-ViewModel, ou MVVM como é conhecido, trata-se de um design pattern que consiste em separar lógicas de UI, da lógica de negócios,
tornando as aplicações mais fáceis de serem desenvolvidas e testadas. Reúne todos os dados brutos que podem vir de banco de dados, webservices e etc, aplica lógicas e formata estes dados para apresentação em uma ViewController. Expõe via properties somente a informação que a ViewController precisa saber para efetuar o trabalho de mostrar os dados em uma view.

## MVVM e binding de dados

![]({{ site.baseurl }}/img/delarge/MVVMReactiveCocoa.png)

No contexto deste post, de utilizar ReactiveCocoa somente para binding de dados, usando a arquitetura MVVM, o ReactiveCocoa executa uma função muito específica. Ele fornece uma espécie de "ponte" da ViewModel para a view, monitorando todas as mudanças realizadas no modelo de dados e mapeando estas mudanças para as properties da ViewModel efetuando qualquer lógica de negócios necessária.
Para dar um exemplo, imagine que nosso modelo de dados contenha uma property do tipo NSDate chamada de dateAdded. Nós iremos monitorar suas mudanças para atualizar uma property do nosso ViewModel, também chamada de dateAdded, mas esta, do nosso ViewModel. A property dateAdded do nosso modelo, como dito anteriormente, será uma NSDate enquanto a do nosso ViewModel será uma NSString. O binding irá parecer com algo descrito abaixo:

~~~objc
@property (nonatomic) NSString *dateAdded;
@property (nonatomic) Model *model;

RAC(self,dateAdded)=[RACObserve(self.model,dateAdded)
	map:^(NSDate *date){
	 return [[ViewModel dateFormatter] stringFromDate:date];
 }];
~~~

dateFormatter é um método de classe em nosso ViewModel, que faz uso de cache de uma instância NSDateFormatter que estará pronta para reuso (NSDateFormatter são muito custosas para se criar a todo tempo). Então nossa ViewController poderá monitorar todas as mudanças da property dateAdded da ViewModel, exibindo em uma label, por exemplo.


~~~objc
RAC(self.label,text) = RACObserve(self.ViewModel,dateAdded);
~~~

Aqui nós estamos abstraindo toda a lógica de se transformar um NSDate para uma NSString para a nossa ViewModel, possibilitando escrever testes unitários para garantir seu funcionamento. Pode parecer um pouco confuso neste exemplo e um tanto quanto trivial como podemos ver, mas como vimos isso ajuda a reduzir significamente a quantidade de código de lógica de UI de nossas ViewControllers.
Desta forma toda vez que a property dateAdded do nosso modelo, for modificada a property dateAdded do modelo será populada com o valor correspondente. E nós criamos uma conexão "invisível" entre elas. Essa é a razão pela qual chamamos reativa: duas properties estão agora relacionadas, então as mudanças que ocorrerem em uma irão afetar reativamente a outra.    

## Mas o que seria um view-model?

Resumindo, é a parte que trata da lógica de como se representam os dados. Os dados raramente são representados em sua aplicação da mesma forma em que são persistidos, quase sempre são transformados antes. Assim como citamos no exemplo acima, precisamos transformar um NSDate para uma NSString para melhor compreensão do usuário. E é para isto que uma ViewModel serve: Ela possui o modelo de dados, transforma-o e normaliza-o para ser exibido posteriormente pela UI. A ViewModel não é responsável apenas por estas transformações, mas também por ações associadas ao modelo.

## Desacoplando e compondo ViewModels.
Esta é uma parte importante, em qualquer parte no desenvolvimento de software - poder ser facilmente desacoplado. Na maioria das vezes, uma particular ViewModel serve toda uma tela do aplicativo, fazendo mais de uma coisa dinamicamente. Agora imagine outra ViewModel servindo outra view com alguma coisa em comum com esta outra. Neste cenário ViewModels podem ser refatoradas, de forma com que funcionalidades em comum sejam movidas para uma terceira ViewModel. Isso nos permite ter um código mais granular e menos dependente. Como reusar uma ViewModel em comum? Simplesmente crie uma property de outra ViewModel conectando entradas e saídas desta forma:

~~~objc
@property (nonatomic) SubViewModel *subViewModel;
@property (nonatomic) Model *inputModel;
@property (nonatomic) NSString *outputValue;

- (instancetype)init {
    self = [super init];
    if (self) {
        //tune subViewModel
        RAC(self, subViewModel.model) = RACObserve(self, inputModel);

        //tune output
        RAC(self, outputValue) = RACObserve(self, subViewModel.itsOutputValue);
			}
		return self;
}
~~~

## Testabilidade

Aparentemente ViewModels possuem um padrão muito conveniente para se cobrir com testes, por possuir entrada e saída de dados. Testes possuem um padrão muito bem definido: Configurar valores arbitrários de entrada e verificar os valores esperados em sua saída..
ViewModels que fazem uso de FRP, particularmente ReactiveCocoa, fazem de maneira mais limpa e direta como implementar a funcionalidade desejada. Isto não significa necessariamente que você escreverá menos código, mas com certeza o fará de maneira mais limpa. Quanto mais claramente você entender o que precisa fazer, menos erros irá cometer. E esta é a vantagem principal do ReactiveCocoa e porque MVVM é tão bom: permitem com que você cometa cada vez menos erros.

## E agora? Como proceder?
As utilizações do ReactiveCocoa vão MUITO além de simples bindings. Seu conteúdo é bastante vasto e sua curva de aprendizado bastante alta. Desde simples bindings até camadas de conexão, vale a pena o seu estudo, por tornar sua aplicação reativa com todas as vantagens que isto pode oferecer.   

## Algumas (valiosas) Fontes de informação
* [Introduction to MVVM by Ash Furrow ](https://www.objc.io/issues/13-architecture/mvvm/)
* [A sample app by Ash Furrow](https://github.com/AshFurrow/C-41)
* [MVC, MVVM, FRP, And Building Bridges by Jonathan Penn](http://cocoamanifest.net/articles/2013/10/mvc-mvvm-frp-and-building-bridges.html)
* [MVVM Tutorial with ReactiveCocoa](https://www.raywenderlich.com/74106/mvvm-tutorial-with-ReactiveCocoa-part-1)
* [Basic MVVM with ReactiveCocoa by Colin Wheeler](http://cocoasamurai.blogspot.com.br/2013/03/basic-mvvm-with-ReactiveCocoa.html)
* [On MVVM, and Architecture Questions by Chris Trott](http://twocentstudios.com/2014/06/08/on-mvvm-and-architecture-questions/)
* [JaviLorbada/FRP iOS Learning resources](https://gist.github.com/JaviLorbada/4a7bd6129275ebefd5a6)
