---
layout:     post
title:      "O mundo é mais que seu umbigo"
date:       2016-03-01 00:00:00
author:     "Marcelo Fabri"
header-img: "img/fabri/pia17936_evening_star.jpg"
category:   "open-source"
---

> Marcelo Fabri ([@marcelofabri_](https://twitter.com/marcelofabri_){:target="_blank"}) é desenvolvedor iOS desde quando a contagem de referências era manual e Xcode e Interface Builder eram apps distintos.


A comunidade iOS está constantemente presente no nosso dia-a-dia. Aposto que você já se beneficiou dela de algum jeito, seja usando alguma biblioteca criada por outra pessoa, alguma ferramenta open source ou participando de algum encontro do [CocoaHeads](http://www.cocoaheads.com.br){:target="_blank"}. [Não??!](https://www.youtube.com/watch?v=8gEkCkhtUQU){:target="_blank"} Mas com certeza você já foi ajudado por alguma resposta do [Stack Overflow](http://stackoverflow.com){:target="_blank"}. 😅

A pergunta que eu faço agora é: o quanto você já retribuiu para essa comunidade?

### Retribuindo código em projetos existentes

Retribuir com código é a forma mais imediata de ajudar a comunidade. Afinal, somos (a maioria) desenvolvedores e é nisso que somos bons.

E é fácil começar. Você já deve ter se deparado com alguma limitação ou possível melhoria de alguma ferramenta ou biblioteca que usa. Por que não ajudar ao invés de reclamar?

Não precisa ser algo complexo ou revolucionário. Coisas simples também são importantes. Uma das minhas primeiras contribuições foi para uma biblioteca chamada [CGFloatType](https://github.com/kylef/CGFloatType){:target="_blank"}. Foi uma [pull request](https://github.com/kylef/CGFloatType/pull/4){:target="_blank"} de 21 linhas, adicionando uma função que eu precisava mas não existia no projeto ainda.

~~~objc
extern CGFloat roundCGFloat(CGFloat x) {
    #if CGFLOAT_IS_DOUBLE
      return round(x);
    #else
      return roundf(x);
    #endif	
}
~~~

O processo deve ter demorado uns 20 minutos, no máximo. Eu poderia ter gasto uns 5 minutos e deixado apenas no meu projeto, mas seria um desperdício não contribuir de volta. Esses 15 minutos não farão diferença alguma para o seu projeto (*e se fizer, tem algo muito errado acontecendo*), mas podem ser muito úteis para outras pessoas.

Algum tempo depois, estava trabalhando com Swift e usando o [Natalie](https://github.com/krzyzanowskim/Natalie){:target="_blank"} para gerar código com o objetivo de eliminar aquelas strings terríveis quando trabalhamos com Storyboards. Percebi um ponto que poderia ser melhorado e abri [uma pull request](https://github.com/krzyzanowskim/Natalie/pull/24){:target="_blank"}. O [Christian](https://twitter.com/chrisfsampaio){:target="_blank"}, que trabalhava comigo na época, também descobriu uma outra limitação e abriu [outra pull request](https://github.com/krzyzanowskim/Natalie/pull/15){:target="_blank"}.

Como vocês podem ver, não foram grandes alterações feitas, mas eram importantes para nós e imaginamos que seriam importantes para a comunidade. E para facilitar, foram alterações feitas em repositórios nas linguagens que trabalhamos diariamente: Objective-C e Swift.

Muitas ferramentas que usamos para trabalhar são feitas em Ruby, incluindo o [liftoff](https://github.com/thoughtbot/liftoff){:target="_blank"}, que serve para gerar projetos do Xcode conforme um template pré-determinado. Isso é algo muito bom para ajudar a espalhar boas práticas dentro de uma empresa e por isso começamos a usar. Porém, ainda tínhamos que fazer muitas configurações manualmente depois, já que sempre criávamos outras *build configurations* e *schemes*, usando algumas flags como `GCC_PREPROCESSOR_DEFINITIONS`.

Juntando o problema existente e a vontade de melhorar um pouco meu Ruby, resolvi tentar implementar isso no liftoff. Foi uma experiência bem desafiadora, já que tive que aprender melhor como *build configurations* e *schemes* funcionam no Xcode, além de aprender mais sobre Ruby e, principalmente, testes unitários. O resultado pode ser visto em duas pull requests: [#224](https://github.com/thoughtbot/liftoff/pull/224){:target="_blank"} e [#225](https://github.com/thoughtbot/liftoff/pull/225){:target="_blank"}. Olhem a quantidade de comentários feitos, principalmente na segunda, sobre como eu poderia melhorar o que eu fiz! Ganhei de graça uma aula sobre Ruby e testes. 😄

Além disso, é muito legal receber feedbacks positivos de pessoas que você considera como referências na sua área, como o pessoal da [thoughtbot](https://thoughtbot.com){:target="_blank"} nesse caso, ou do [Ash Furrow](https://twitter.com/ashfurrow){:target="_blank"} e do [Orta Therox](https://twitter.com/orta){:target="_blank"} ([orta/cocoapods-keys#125](https://github.com/orta/cocoapods-keys/pull/125){:target="_blank"}) ou ainda do [Felix Krause](https://twitter.com/KrauseFx){:target="_blank"} ([fastlane/fastlane#822](https://github.com/fastlane/fastlane/pull/822){:target="_blank"}), criador do [fastlane](https://github.com/fastlane/fastlane){:target="_blank"}.

Mesmo quando você faz cagadas, o pessoal geralmente é compreensivo. E eu já fiz algumas, como aumentar o tempo de processamento do [SwiftLint](https://github.com/realm/SwiftLint){:target="_blank"} em mais de 16x 😵.

![]({{ site.baseurl }}/img/fabri/swiftlint.png)
<span class="caption text-muted">Ooops! 😅 (<a href="https://github.com/realm/SwiftLint/pull/330" target="_blank">realm/SwiftLint#330</a>)</span>

Outro projeto que eu recentemente ajudei (e fiz cagada) é o [Danger](https://github.com/danger/danger/){:target="_blank"}, uma gem em Ruby para validar pull requests (se o título está dentro do esperado, se tem labels, etc). Em alguns dias trabalhando nisso a noite, consegui desenvolver duas features essenciais para adotarmos no nosso fluxo de trabalho: [disponibilizar labels](https://github.com/danger/danger/pull/52){:target="_blank"} e [ser compatível com o Jenkins](https://github.com/danger/danger/pull/61){:target="_blank"}.

Também deu tempo de fazer uma pequena cagada, [prontamente corrigida pelo Orta](https://github.com/danger/danger/pull/70){:target="_blank"}.

E tudo isso por que eu estava de saco cheio de ser o cara chato, comentando em pull requests no trabalho.

![]({{ site.baseurl }}/img/fabri/bot.png)
<span class="caption text-muted">Eu, sendo chato</span>

E não sou só eu que faço isso. Temos vários exemplos na própria comunidade brasileira: o Christian implementou boa parte do `NSNumberFormatter` no Foundation do Swift ([swift-corelibs-foundation#132](https://github.com/apple/swift-corelibs-foundation/pull/132){:target="_blank"}).

O [Koga](https://twitter.com/brunokoga){:target="_blank"} contribuiu com o [swift-sodium](https://github.com/jedisct1/swift-sodium){:target="_blank"} ([#20](https://github.com/jedisct1/swift-sodium/pull/20){:target="_blank"} e [#21](https://github.com/jedisct1/swift-sodium/pull/21){:target="_blank"}). 

O [Heberti](https://twitter.com/hebertialmeida){:target="_blank"} criou uma [biblioteca toda para fazer parse de ePub](https://github.com/FolioReader/FolioReaderKit){:target="_blank"}. 

O [Diogo](https://twitter.com/diogot){:target="_blank"} já fez [pull request](https://github.com/orta/ARAnalytics/pull/202){:target="_blank"} no [ARAnalytics](https://github.com/orta/ARAnalytics){:target="_blank"}.

### Criando projetos open source

Para os mais aventureiros, existe também a possibilidade de tornar um pedaço (ou tudo!) do seu app open source.

Isso geralmente é mais complicado, pois envolve [licenças](https://soundcloud.com/cocoaheadsbr/s01e08-libs){:target="_blank"}, políticas internas da empresa e até mesmo uma maior dedicação.

Um dos pontos mais complicados é convencer a liderança da empresa que isso é uma boa ideia. Por isso, aqui tem alguns pontos que acho interessantes serem mencionados para os chefes:

* **Visibilidade**: é muito difícil achar bons desenvolvedores, especialmente no caso de iOS. Com um (bom) projeto open source, fica mais fácil a empresa ser reconhecida/descoberta pela comunidade.
* **Contribuição da comunidade**: com um projeto open source, a comunidade pode ajudar e encontrar (e até mesmo arrumar!) bugs ou até mesmo desenvolver features que seriam úteis. Duvida que isso aconteça? Um exemplo é [essa pull request](https://github.com/artsy/eigen/pull/195){:target="_blank"} do Felix Krause, adicionando integração com o fastlane no [Eigen](https://github.com/artsy/eigen){:target="_blank"}.
* **Componentes e documentação**: na minha visão, o ideal é começar disponibilizando componentes reutilizáveis do projeto. É uma ótima oportunidade de descobrir eventuais acoplamentos ou dependências que não deveriam existir, além de documentar o componente, beneficiando até mesmo membros da equipe. Extrair um componente (e tornar open source) exige uma disciplina maior sobre barreiras do seu sistema, eliminando a tentação de "fazer assim mesmo agora, porque o prazo tá apertado". Outro benefício é o maior cuidado em seguir [Semantic Versioning](http://semver.org){:target="_blank"}, já que tendemos a ser um pouco desleixados com isso quando um componente é usado apenas internamente.

Entretanto, eu espero que esses pontos sejam usados apenas para convencer pessoas que não estão mais tão ligadas com desenvolvimento, já que, na minha visão, essas vantagens são apenas efeitos colaterais do principal motivo: retribuir a comunidade que tanto nos ajuda.

Eu recomendo a leitura do post [OSS Expectations](http://artsy.github.io/blog/2016/01/13/OSS-Expectations/){:target="_blank"}, que fala um pouco da experiência do pessoal da Artsy em tornar seus apps open source.

### Outras formas de retribuir

Nem só de código vive a comunidade. Existem várias outras formas de contribuir.

Dar sua opinião como usuário é importante, já que muitas vezes os colaboradores da ferramenta/biblioteca já estão muito envolvidos com ela. Talvez uma das minhas primeiras contribuições tenha sido assim: [uma issue](https://github.com/CocoaPods/CocoaPods/issues/853){:target="_blank"} no CocoaPods sugerindo que um sumário com as principais mudanças fosse impresso ao atualizar a versão.

Outra coisa importante é documentação. É algo chato, que todo mundo odeia fazer, mas importante, principalmente em projetos open source. Por menor que pareça a contribuição, ela é importante, mesmo que seja arrumando *typos*.

Algumas formas de contribuição não envolvem nem repositórios do GitHub.

O Diogo e o Koga escrevem no [Invariante](http://invariante.com){:target="_blank"}, levantando discussões importantes sobre iOS. Blogs são importantes para compartilhar conhecimento, mesmo que sejam posts curtos. Um bom exemplo disso é o blog da [Natasha the Robot](https://www.natashatherobot.com){:target="_blank"}, que sempre faz um post assim que ela aprende algo novo (olhe o ["NSStringFromClass in Swift is Here!"](https://www.natashatherobot.com/nsstringfromclass-in-swift/) para ter uma ideia do que estou falando). 

Uma galera contribui com os episódios do [Podcast do CocoaHeads](https://soundcloud.com/cocoaheadsbr){:target="_blank"}, em especial o [Douglas](https://twitter.com/DougDiskin){:target="_blank"}, que gasta umas 10 horas por semana apenas na edição de cada episódio.

Várias pessoas trabalham muito para os encontros do CocoaHeads acontecerem em [diversas cidades](http://www.cocoaheads.com.br/cidades){:target="_blank"} do Brasil todo mês, não desanimando mesmo quando uma galera confirma presença e não aparece no dia do evento 😔.

Muita gente palestra em eventos como o próprio CocoaHeads, compartilhando suas experiências.

<blockquote class="twitter-tweet" data-conversation="none" data-lang="pt"><p lang="en" dir="ltr">Speaker protip: If you&#39;ve ranted about something in tech, you&#39;re ready to speak. You need passion and perspective, not expert level skills.</p>&mdash; Jessica Rose (@jesslynnrose) <a href="https://twitter.com/jesslynnrose/status/702168652238426112">23 de fevereiro de 2016</a></blockquote> <script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

No [Slack do iOSDevBR](http://iosdevbr.herokuapp.com){:target="_blank"} sempre tem gente disposta a trocar ideia sobre qualquer assunto, desde dúvidas mais técnicas até discussões mais acaloradas.

Tem [um monte de gente](http://stackoverflow.com/research/developer-survey-2015#community-answer){:target="_blank"} que responde perguntas no Stack Overflow e uma das coisas mais legais de lá é o tanto de gente que você pode acabar ajudando.

![]({{ site.baseurl }}/img/fabri/so.png)

E por último, [mais de 20 pessoas](https://github.com/CocoaHeadsBrasil/equinociOS/issues){:target="_blank"} prontamente se comprometeram a escrever posts para o **equinociOS**, sem nenhuma remuneração ou retorno imediato, que não seja contribuir com a comunidade.


### E agora?

A comunidade de iOS no Brasil finalmente saiu das fraldas e está crescendo cada vez mais. Se antes os desenvolvedores iOS não sabiam quem eram seus pares, hoje temos eventos do CocoaHeads que lotam. 

Estamos num momento crucial de definição do que, como comunidade, queremos ser. Está na hora de cada um fazer a sua parte, se envolver, retribuir da maneira que mais se sente a vontade: sejam posts, palestras, organizando eventos, melhorando documentação, desenvolvendo código. Enfim, sair da sua zona de conforto.

Cada um tem algo a colaborar, por mais que ache que não. Assim como todo mundo, por mais experiente que seja, sempre tem o que aprender. E é nessas horas que pessoas diferentes, com backgrounds diferentes são importantes. Nos ajudam a ter contato com tecnologias, problemas e realidades diferentes das quais estamos acostumados (e em alguns casos, acomodados).

Espero que tenha gostado do post e foi uma honra ter sido convidado/forçado a abrir o **equinociOS**. Estou ansioso para ver como você vai contribuir para a comunidade!

Obrigado e até a próxima!

<center><img src="http://i.giphy.com/VyMljrLI1U9Ak.gif"></img></center>


### Referências

Alguns links que foram úteis no processo de escrita desse post:

* [Being a Good OSS Citizen](http://artsy.github.io/blog/2016/01/28/being-a-good-open-source-citizen/) - [@ashfurrow](https://twitter.com/ashfurrow)
* [Contributing to Open Source Doesn’t Require Changing the World](https://speakerdeck.com/orta/contributing-to-open-source-doesnt-require-changing-the-world) - [@orta](https://twitter.com/orta)
* [Building for Open-Source](https://speakerdeck.com/kylef/building-for-open-source) - [@kylef](https://twitter.com/orta)
* [Being Nice in Open Source](https://realm.io/news/altconf-orta-therox-being-nice-in-open-source/) - [@orta](https://twitter.com/orta)
* [Issue 22: iOS at Scale - Artsy](https://www.objc.io/issues/22-scale/artsy/) - [objc.io](https://www.objc.io)
* [Building Online Communities](https://ashfurrow.com/blog/building-online-communities/) - [@ashfurrow](https://twitter.com/ashfurrow)
* [Starting Open Source](https://blog.cocoapods.org/starting-open-source/) - [@K0nserv](https://twitter.com/K0nserv)


> A [foto](http://www.nasa.gov/jpl/msl/earth-view-from-mars-pia17936) de capa do post foi tirada em 31/01/2014 pela Curiosity Mars e mostra a Terra (sim, aquele pontinho brilhante é  a Terra!) e a nossa lua logo embaixo dela. 


