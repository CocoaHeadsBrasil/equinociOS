---
layout:     post
title:      "RxSwift: como eu vim parar aqui?"
subtitle:   "Minha experiência aprendendo programação reativa e funcional"
date:       2016-03-14 00:00:00
author:     "Bruno Koga"
header-img: "img/brunokoga/stream.jpg"
---

>☝️
>Esse artigo não visa ensinar nenhum conceito ou técnica diretamente. Ele é muito mais um relato pessoal da minha experiência no aprendizado de alguns conceitos e paradigmas. Se você não tem ideia do que é programação reativa, eu recomendo fortemente [esse guia](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754). 

>✌️
>A maioria dos exemplos contidos nesse artigo foram retirados de projetos reais e de documentações ou exemplos providos pelos autores das bibliotecas citadas. Todos os respectivos links estam disponíveis no final do artigo.

###Passado
Programo para iOS desde 2009. Até 8 meses atrás, Objective-C (e o básico de C) eram as duas únicas linguagens que eu sabia programar. 

E por muitos anos a minha abordagem para desenvolver software sempre envolveu: Objective-C, [programação 100% orientada a objetos](https://pt.wikipedia.org/wiki/Orientação_a_objetos) e [programação imperativa](https://pt.wikipedia.org/wiki/Programação_imperativa). Foi assim que aprendi a programar e foi assim que sempre programei.
 
###Presente
Em junho de 2015, logo após o lançamento do Swift 2.0, entrei num projeto 100% escrito em Swift (1.2). Era meu primeiro contato real com Swift. 

![]({{ site.baseurl }}/img/brunokoga/aleera.png)
<span class="caption text-muted">Logo eu me vingarei!


> **Opinião:** aprender Swift é como aprender qualquer outra linguagem de programação. Sua sintaxe é relativamente simples de aprender e já há muita documentação e exemplos na internet. Não acho que Swift deveria ser barreira para ninguém: é algo que se aprende (o básico) em poucos dias. E é questão de dias para você ser produtivo usando Swift.

Porém, a camada de comunicação com o servidor desse aplicativo era construída em cima de uma biblioteca escrita chamada [BrightFutures](https://github.com/Thomvis/BrightFutures). E com isso me deparei com dois outros conceitos que eu nunca havia estudado (muito menos usado): [programação funcional](https://pt.wikipedia.org/wiki/Programação_funcional) e [programação reativa](https://en.wikipedia.org/wiki/Reactive_programming). 

> **Opinião:** o resumo de programação funcional é, **para mim**, algo do tipo: uma mudança de pensamento que, ao invés de você pensar em classes e objetos e na interação entre essas entidades, você pensa muito mais em funções e composição de funções. É um tema bastante extenso. Pela minha experiência, isso é algo que leva um certo tempo para aprender e requer disciplina para ler teorias e documentações. 

E por fim: programação reativa. Até então, eu nunca tinha conseguido entender exatamete o que era programação reativa. Todas a vezes que tentei usar [ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa) desisti no meio por perder o controle do que estava acontecendo.

> **Opinião:** ![]({{ site.baseurl }}/img/brunokoga/rac.png)
<span class="caption text-muted">meu snippet para `;rac` no TextExpander

###Futuro
Voltando ao `BrightFutures`: na época, mesmo sem saber nada de programação funcional e programação reativa, foi fácil para eu entender a teoria e a filosofia por trás do _framework_: prover uma forma de tratar com código assíncrono e tratamento de erros através de [futuros e promessas](https://en.wikipedia.org/wiki/Futures_and_promises).

E foi aí que comecei a entender _um pouco_ o que significava programação funcional e porque o Swift era uma linguagem que permitia a implementação e o uso de conceitos funcionais.

> Vamos começar a viajar um pouco. Essa é a hora ideal para uma dose de cafeína ☕️.

Imagine o caso em que você tem uma série de eventos futuros que você quer que sejam encadeados, respeitando (ou não) uma ordem e que no final desse encadeamento de eventos (ou seja, no futuro) você quer ser notificado.

Por exemplo: um aplicativo em que o usário pode logar com a sua conta e baixar todos os seus posts. Temos então um evento de `logIn` que, em caso de sucesso, dispara um evento de `fetchPosts` para aquele usuário. Tanto o evento de `logIn` como o de `fetchPosts` seriam funções que retornam um "Futuro" (no caso do _BrightFutures_, um `Future`). O resultado da execução do `logIn`, por exemplo, é um `future` que representa um erro (em caso de erro) ou um `userId` (em caso de sucesso). No caso do `fetchPosts`, o retorno seria um  `future` representando um erro (falha) ou um array de posts (sucesso). 

Antes mesmo de ler sequer uma linha de código, a abstração acima faz sentido. O problema agora é como programar usando esse "arquitetura". No exemplo acima, o código seria algo assim (código retirado do [repositório do Bright Futures](https://github.com/Thomvis/BrightFutures)):

~~~swift
User.logIn(username,password).flatMap { user in
    Posts.fetchPosts(user)
}.onSuccess { posts in
    // do something with the user's posts
}.onFailure { error in
    // either logging in or fetching posts failed
}
~~~

A assinatura das funções `User.logIn` e `Posts.fetchPosts` seria algo assim:

~~~swift
func logIn(username: String, password: String) -> Future<User, ErrorType>
func fetchPosts(user: User) -> Future<[Posts], ErrorType>
~~~

Se você não estiver acostumado com a sintaxe do Swift ou o código acima parecer muito confuso para você, essa é a explicação desse código (traduzido do [repositório do Bright Futures](https://github.com/Thomvis/BrightFutures)):
 
Quando o futuro retornado por `User.logIn` falha (por exemplo, se o `username` e `password` não estiverem corretos), tanto o `flatMap` como o `onSuccess` são pulados, e o closure `onFailure` é chamado com o `error` que ocorreu na tentativa de `logIn`. Se a tentativa de realizar o `logIn` for bem sucedida, o resultado da operação (que é um objeto `user`) é passado para o `flatMap`, que "transforma" o usuário em um _array_ com seus posts. Se os posts não puderem ser baixados (por causa de um erro), `onSuccess` é pulado, e `onFailure` é chamado com o `error` que ocorreu na tentativa de baixar os posts. Se os posts puderem ser baixados com sucesso, `onSuccess` é chamado com os posts do usuário.

> **Opinião:** o código pode parecer estranho a princípio, mas novamente a teoria faz sentido. E por isso que acho que tanto programação funcional como programação reativa são dois conceitos que requerem estudo da teoria antes da prática. É um pouco diferente de aprender uma linguagem de programação nova (como Swift), que algo 100% _hands on_ pode ser efetivo.

Falando nisso, o `flatMap` aí em cima é um dos caras que fazem parte dos conceitos funcionais. Caso você não o entenda (ainda!), a idéia é que ele "transforma" (ou "mapeia") o **resultado** de um `Future` no **valor** de um novo `Future`. Ou seja, ele transforma o resultado do `logIn` (que é um `user`) no valor de entrada para o `fetchPosts` (que por sua vez retorna outro `Future`).

E é aí que entra a mágica do Swift ser tão restrito em relação a tipos (ou seja, ser uma linguagem [_type safe_](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/TheBasics.html), totalmente diferente do Objective-C): o compilador consegue te garantir que os tipos de dados dos valores retornados por uma função e passados para outra função estão sempre corretos. E a partir daí, o atalho preferido do Xcode passa a ser `⌥ + click`:

![]({{ site.baseurl }}/img/brunokoga/typesafe1.png)
<span class="caption text-muted"> O compilador está do nosso lado! Após concatenar várias funções, você (quase) sempre pode confiar nele para te dizer o tipo do retorno das suas funções. Essa técnica é muito útil para você checar se o tipo retornado é mesmo o que você espera.

![]({{ site.baseurl }}/img/brunokoga/typesafe2.png)
<span class="caption text-muted">a mesma estratégia funciona também para os parâmetros das funções concatenadas. Mesmo que você não entenda o código acima, a idéia é que você tem a segurança de saber que está trabalhando com o tipo de dado correto (diferentemente do Objective-C, onde não há essa garantia).


###Resumindo

`Futures` são ações futuras (assíncronas, com tratamento de erros). O paradigma da programação funcional permite transformar e encadear essas ações de uma forma clara e segura. Além disso, ao trabalhar com `Futures`, estamos trabalhando de uma forma "reativa". Em outras palavras, estamos programando baseado no "retorno" dos `futures`. Esses `futures` se responsabilizam de fazer o trabalho deles assincronamente. E ao encadear esses `futures`, estamos trabalhando com os conceitos funcionais. Por isso que, apesar de diferentes, esses dois termos são vistos comumente juntos: **Programação Reativa Funcional**.

###RxSwift

[RxSwift](https://github.com/ReactiveX/RxSwift) é a versão escrita em Swift do [ReactiveX](http://reactivex.io). O [ReactiveX](http://reactivex.io) por sua vez é uma biblioteca que possibilita o uso de programação baseada em eventos (reativa), de forma assíncrona e que pode ser composta e encadeada (funcional) através de [**operadores**](http://reactivex.io/documentation/operators.html).

> **Opinião:** novamente, a melhor forma - na minha opinião - de aprender os conceitos básicos de programação reativa é esse link: [The introduction to Reactive Programming you've been missing](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754).

A idéia básica da programação reativa é que você trabalha em cima de _streams_ (fluxos, correntes, sequências) de dados. Cada _stream_ é uma sequência de eventos que acontecem em uma linha de tempo e podem emitir três tipos de "coisas": um valor, um erro ou um _sinal de finalizado_.

No nosso exemplo anterior, usando [BrightFutures]((https://github.com/Thomvis/BrightFutures)), a nossa entidade `Future` nada mais é do que um _stream_, que pode produzir um valor (e um sinal de "finalizado" logo em seguida) ou um erro.

Com o `RxSwift`, porém, as coisas são mais amplas que apenas `Futures`. Você pode definir outros tipos de _streams_. Um bom exemplo é um `UITextField`: ao invés de programar pensando nos conceitos de _delegates_, o `RxSwift` te permite **observar** os valores do _stream_ `rx_text` do `UITextField` e **reagir** de acordo com a emissão desses valores. Veja um exemplo:

~~~swift
let lengthOfBytes =
        textView.rx_text
            .map { $0.lengthOfBytesUsingEncoding(NSUTF8StringEncoding) }
            .filter { $0 > 5 }
~~~

Com o código acima, estamos definindo que a cada vez que `rx_text` emitir um valor, nós iremos _mapear_ (ou seja, transformar) esse valor em um `Int`, e então iremos filtrar esse valor, seguindo adiante apenas de ele for maior que 5.

Fazendo novamente a comparação com `Futures`, o `RxSwift` encapsula seus _streams_  em `Observables`. Ou seja, o tipo de dado de `rx_text` é um `Observable<String>`, enquanto, no exemplo acima, o tipo de dados de `lengthOfBytes` é um `Observable<Int>` (ou seja, é um _stream_ também):

![]({{ site.baseurl }}/img/brunokoga/rxswift1.png)
<span class="caption text-muted">Lembre-se: `⌥ + click` é seu melhor amigo!

A última peça que falta nesse quebra-cabeças do `RxSwift` é que **nada acontece** até você der um _subscribe_ no seu _stream_ (ou na sua sequência de _streams). Antes disso, você está apenas definindo as ações que você irá tomar, baseada nos eventos que podem acontecer. O _subscribe_ é o sinal verde para que o `Observable` comece a emitir itens:

~~~swift
self.messagesHandler.messages
            .window(timeSpan: 0.5, count: 10000, scheduler: MainScheduler.instance)
            .flatMap { observable -> Observable<ServerMessage> in
                observable.take(1)
            }
            .subscribeNext { [weak self] message in
                    self?.playMessageArrivedSound()
                }
            }.addDisposableTo(disposeBag)
~~~

> O `disposeBag`, a grosso modo, é a forma do `RxSwift` liberar os recursos alocados, quando você acabar de observar a sequência (seja por opção ou porque ela terminou). Não vou entrar em detalhes sobre como usar o `dispose` do `RxSwift`. Não é algo muito complicado e você pode ler mais sobre o assunto [aqui](https://github.com/ReactiveX/RxSwift/blob/master/Documentation/GettingStarted.md#disposing).
 
O `subscribeNext` é a forma que o seu `Observer` fala "a cada valor emitido, faça isso". O código acima é um exemplo de como observar mensagens recebidas em um aplicativo de mensagens instantâneas o disparar um som cada vez que uma mensagem chega. Os operadores `window` e `flatMap` garantem que haja um intervalo mínimo de 0.5 segundos entre cada som disparado (para saber mais, veja [essa pergunta no StackOverflow](http://stackoverflow.com/questions/35438268/rxswift-debounce-throttle-inverse)). No exemplo acima, não estamos reagindo em caso de erro, nem para os sinais de `completed`, já que, em teoria, esse _stream_ de mensagens recebidas nunca encerra.

Uma forma comum de se usar esse paradigma de observar `streams` é com **binding**. [Data binding](https://en.wikipedia.org/wiki/Data_binding) é o processo de "conectar" a informação apresentada na sua UI com o seu modelo ou lógica de negócios. 

Usando o `RxSwift` por exemplo, a implementação de uma tela de Settings poderia ser a seguinte:

~~~swift
        //bind, onde nameLabel = firstName + " " + lastName
        Observable
         .combineLatest(firstNameTextField.rx_text, lastNameTextField.rx_text) {
            $0 + " " + $1
        }.bindTo(nameLabel.rx_text)
         .addDisposableTo(disposeBag)
        
        //bind, onde mapeamos o valor do Switch em um emoji:
        settingsSwitch.rx_value
         .map {
            $0 ? "👍" : "👎"
        }.bindTo(settingsLabel.rx_text)
         .addDisposableTo(disposeBag)
~~~
> Se a sintaxe de`$0` e `$1` for meio etranha pra você, eles são recursos do próprio Swift. Recomendo a leitura do capítulo de _Closures_ do [The Swift Programming Language](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Closures.html).

E esse é o resultado:

<center><img src="/img/brunokoga/bind.gif" alt=""/></center><span class="caption text-muted">Suas telas de Settings nunca mais serão as mesmas! ✨


###E esse é só o começo

Acredite: esses são apenas os primeiros passos dentro do mundo de programação reativa e/ou funcional. Ainda existe muito a ser explorado e muito, muito a ser aprendido.
Algo que tem me ajudado bastante nesse processo de aprendizado é ter a consciência de que o tema é extenso e não se aprende da noite pro dia (diferente de aprender uma nova linguagem de programação, por exemplo). A verdade é que Programação Funcional e Reativa são conceitos longos e complexos e que levam tempo para serem assimilados. Mas acredito que uma vez que você aprenda a teoria, a escolha de bibliotecas, seja um "mero" detalhe.

Espero que compartilhar a minha experiência possa ser útil para você começar a entender um pouco mais desses paradigmas de programação. Caso queira se aprofundar mais, coloquei alguns links nas **Referências** abaixo.

###Agradecimentos
* [Lars](https://twitter.com/larslockefeer) por me ensinar Swift e os conceitos do [BrightFutures](https://github.com/Thomvis/BrightFutures);
* [Thomas](https://twitter.com/thomvis88) por ter escrito a [BrightFutures](https://github.com/Thomvis/BrightFutures);
* [Mentos](https://twitter.com/gsampaio) por ter sido a inspiração para eu criar meu snippet `;rac` no TextExpander.

###Referências
Fontes utilizadas para a escrita deste artigo:

* [Bright Futures](https://github.com/Thomvis/BrightFutures)
* [RxSwift](https://github.com/ReactiveX/RxSwift/blob/master/Documentation/Examples.md)
* [Slack do RxSwift](http://slack.rxswift.org) 
* [The introduction to Reactive Programming you've been missing](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754)
* [ReactiveX](http://reactivex.io)
* [RxMarbles](http://rxmarbles.com)

Obrigado pela leitura!

> Bruno Koga
<br>[@brunokoga](https://twitter.com/brunokoga)
<br>http://www.brunokoga.com 
