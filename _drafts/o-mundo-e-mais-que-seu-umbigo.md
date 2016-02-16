---
layout:     post
title:      "O mundo é mais que seu umbigo"
date:       2016-01-01 15:58:00
author:     "Marcelo Fabri"
header-img: "img/post-exemplo.jpg"
category:   Categoria
---

> Marcelo Fabri ([@marcelofabri_](https://twitter.com/marcelofabri_)) é desenvolvedor iOS desde quando a contagem de referências era manual e Xcode e Interface Builder eram apps distintos.


A comunidade iOS está constantemente presente no nosso dia-a-dia. Aposto que você já se beneficiou dela de algum jeito, seja usando alguma biblioteca criada por outra pessoa, alguma ferramenta open source ou participando de algum encontro do [CocoaHeads](http://www.cocoaheads.com.br). [Não??!](https://www.youtube.com/watch?v=8gEkCkhtUQU) Mas com certeza você já se beneficiou de alguma resposta do [Stack Overflow](http://stackoverflow.com). 😅

A pergunta que eu faço agora é: o quanto você já retribuiu para esse comunidade?

### Por que retribuir?

### Retribuindo código em projetos existentes

Retribuir com código é a forma mais imediata de ajudar a comunidade. Afinal, somos (a maioria) desenvolvedores e é nisso que somos bons.

E é fácil começar. Você já deve ter se deparado com alguma limitação ou possível melhoria de alguma ferramenta ou biblioteca que usa. Por que não ajudar ao invés de reclamar?

Não precisa ser algo complexo ou revolucionário. Coisas simples também são importantes. Uma das minhas primeiras contribuições foi para uma biblioteca chamada [CGFloatType](https://github.com/kylef/CGFloatType). Foi uma [pull request](https://github.com/kylef/CGFloatType/pull/4) de 21 linhas, adicionando uma função que eu precisava mas não existia no projeto ainda. O processo deve ter demorado uns 20 minutos, no máximo. Eu poderia ter gasto uns 5 minutos e deixado apenas no meu projeto, mas seria um desperdício não contribuir de volta. Esses 15 minutos não farão diferença alguma para o seu projeto (e se fizer, tem algo muito errado acontecendo), mas podem ser muito úteis para outras pessoas.

Algum tempo depois, estava trabalhando com Swift e usando o [Natalie](https://github.com/krzyzanowskim/Natalie) para gerar código com o objetivo de eliminar aquelas strings terríveis quando trabalhamos com Storyboards. Percebi um ponto que poderia ser melhorado e abri [uma pull request](https://github.com/krzyzanowskim/Natalie/pull/24). O [Christian](https://twitter.com/chrisfsampaio), que trabalhava comigo na época, também descobriu uma outra limitação e abriu [outra pull request](https://github.com/krzyzanowskim/Natalie/pull/15).

Como vocês podem ver, não foram grandes alterações feitas, mas eram importantes para nós e imaginamos que seriam importantes para a comunidade. E para facilitar, foram alterações feitas em repositórios nas linguagens que trabalhamos diariamente: Objective-C e Swift.

Muitas ferramentas que usamos diariamente são feitas em Ruby, incluindo o [liftoff](https://github.com/thoughtbot/liftoff), que serve para gerar projetos do Xcode conforme um template pré-determinado. Isso é algo muito bom para ajudar a espalhar boas práticas dentro de uma empresa e começamos a usar. Porém, ainda tínhamos que fazer muitas configurações manualmente depois, pois  sempre criávamos outras *build configurations* e *schemes*, configurando algumas flags como `GCC_PREPROCESSOR_DEFINITIONS`.

Juntando o problema existente e a vontade de melhorar um pouco meu Ruby, resolvi tentar implementar isso no liftoff. Foi uma experiência bem desafiadora, já que tive que aprender melhor como *build configurations* e *schemes* funcionam no Xcode, além de aprender mais sobre Ruby e, principalmente, testes unitários. O resultado pode ser visto em duas pull requests: [#224](https://github.com/thoughtbot/liftoff/pull/224) e [#225](https://github.com/thoughtbot/liftoff/pull/225). Olhem a quantidade de comentários feitos, principalmente na segunda, sobre como eu poderia melhorar o que eu fiz! Ganhei de graça uma aula sobre Ruby e testes. 😄

Além disso, é muito legal receber feedbacks positivos de pessoas que você considera como referências na sua área, como o pessoal da [thoughtbot](https://thoughtbot.com) nesse caso, ou [do Ash Furrow e do Orta Therox](https://github.com/orta/cocoapods-keys/pull/125) ou ainda do [Felix Krause](https://github.com/fastlane/fastlane/pull/822), criador do [fastlane](https://github.com/fastlane/fastlane).

Mesmo quando você faz cagadas, o pessoal geralmente é compreensível. E eu já fiz algumas, como aumentar o tempo de processamento do [SwiftLint](https://github.com/realm/SwiftLint) em mais de 16x 😵.

![]({{ site.baseurl }}/img/fabri/swiftlint.png)
Ooops! 😅 ([https://github.com/realm/SwiftLint/pull/330](https://github.com/realm/SwiftLint/pull/330))

Outro projeto que eu recentemente ajudei (e fiz cagada) é o [Danger](https://github.com/danger/danger/), uma gem em Ruby para validar pull requests (se o título está dentro do esperado, se tem labels, etc). Em alguns dias trabalhando nisso a noite, consegui fazer duas features críticas para adotarmos no nosso fluxo de trabalho: [disponibilizar labels](https://github.com/danger/danger/pull/52) e [ser compatível com o Jenkins](https://github.com/danger/danger/pull/61).

Também deu tempo de fazer uma pequena cagada, [prontamente corrigida pelo Orta](https://github.com/danger/danger/pull/70).

E tudo isso por que eu estava de saco cheio de ser o cara chato, comentando em pull requests no trabalho.

![]({{ site.baseurl }}/img/fabri/bot.png)

E não sou só eu que faço isso. Temos vários exemplos na nossa própria comunidade brasileira: o Christian implementou boa parte do `NSNumberFormatter` no Foundation do Swift ([swift-corelibs-foundation/pull/132](https://github.com/apple/swift-corelibs-foundation/pull/132)). 

O [Koga](https://twitter.com/brunokoga) contribuiu com o [swift-sodium](https://github.com/jedisct1/swift-sodium) ([#20](https://github.com/jedisct1/swift-sodium/pull/20) e [#21](https://github.com/jedisct1/swift-sodium/pull/21)). 

O [Heberti](https://twitter.com/hebertialmeida) criou uma [biblioteca toda para fazer parse de ePub](https://github.com/FolioReader/FolioReaderKit). 

O [Diogo](https://twitter.com/diogot) já fez [pull request](https://github.com/orta/ARAnalytics/pull/202) no [ARAnalytics](https://github.com/orta/ARAnalytics). 

### Criando projetos open source


### Outras formas de retribuir

Nem só de código vive a comunidade. Existem várias outras formas de contribuir.

Dar sua opinião como usuário é importante, já que muitas vezes os colaboradores da ferramenta/biblioteca já estão muito envolvidos com ela. Talvez uma das minhas primeiras contribuições tenha sido assim: [uma issue](https://github.com/CocoaPods/CocoaPods/issues/853) no CocoaPods sugerindo que um sumário com as principais mudanças fosse impresso ao atualizar a versão.

Outra coisa importante é documentação. É algo chato, que todo mundo odeia fazer, mas importante, principalmente em projetos open source. Por menor que pareça a contribuição, ela é importante, mesmo que seja arrumando *typos*.

Algumas formas de contribuição não envolvem nem repositórios do GitHub.

O Diogo e o Koga escrevem no [Invariante](http://invariante.com), levantando discussões importantes sobre iOS.

Uma galera contribui com os [Podcasts do CocoaHeads](http://www.cocoaheads.com.br/podcasts/).

Várias pessoas trabalham muito para os encontros do CocoaHeads acontecerem em [diversas cidades](http://www.cocoaheads.com.br/cidades) do Brasil todo mês.

Muita gente palestra em eventos como o próprio CocoaHeads, compartilhando suas experiências.

No [Slack do iOSDevBR]() sempre tem gente disposta a trocar ideia sobre qualquer assunto, desde dúvidas mais técnicas até discussões mais acaloradas.

Tem [um monte de gente](http://stackoverflow.com/research/developer-survey-2015#community-answer) que responde perguntas no Stack Overflow e uma das coisas mais legais de lá é o tanto de gente que você pode acabar ajudando.

![]({{ site.baseurl }}/img/fabri/so.png)



### E agora?
