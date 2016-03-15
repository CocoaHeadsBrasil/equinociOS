---
layout:     post
title:      "Bibliotecas 📚"
date:       2016-03-17 00:00:00
author:     "Igor Ferreira"
header-img: "img/igorcferreira/library.jpg"
category:   "library"
---
> Eu ([@icastanheda](https://twitter.com/icastanheda)) sou desenvolvedor mobile desde de 2012, quando ARC, storyboard e o Ice Cream Sandwich eram recentes no mundo.
> Alguns de vocês já deve ter ouvido minha voz em um dos episódios do [Podcast do CocoaHeads Brasil](http://soundcloud.com/cocoaheadsbr).

<br/>

### Prefácio

Eu sempre trabalhei com o desenvolvimento de apps para o usuário final do produto, os famosos ipas que vão para a loja. Mas, recentemente mudei meu trabalho para uma empresa que desenvolve bibliotecas para outras empresas.

<br/>

<center>
<img alt="I'm going on an adventure" src="https://45.media.tumblr.com/3ad2585f368bd07136af9d01aa285906/tumblr_n7y9otYV1i1s9t7xfo1_500.gif"/>
</center>

<br/>

Mesmo tendo conhecimento sobre as definições e paradigmas de libraries e frameworks, eu enfrentei alguns problemas que nunca tinha enfrentado e precisei rever alguns pontos e detalhes da plataforma Apple. Especialmente o processo de distribuição de binários.

Minha intenção, aqui, é trazer alguns conceitos que permeiam as bibliotecas, mesmo que superficialmente!

<br/>

### Library?

Mesmo que você nunca tenha desenvolvido esse tipo de distribuição, libraries fazem parte do seu dia-a-dia de desenvolvimento. Seja com 3rd parties como *AFNetworking* (ou *AlamoFire*), *OCMock*, *Realm*, entre muitos outros, ou com o *Foundation*, *UIKit* e os mais diversos módulos da SDK do iOS e do OS X.

Bibliotecas são formas de reutilizar seu código em múltiplos projetos. Você poderia copiar e colar os arquivos para cada novo app, mas, além disso não ser prático, dificulta, e muito, a manutenção e integração.

Alám disso, são formas de produção e distribuição de código que não são atrelados a um produto específico. Como, por exemplo, um set de lógica para tratar de um protocolo de serialização.

<br/>

### Library x Framework

<center><img alt="Static library e framework" src="{{ site.baseurl }}/img/igorcferreira/library_types.jpg"/></center>

Aí está a primeira coisa que gostaria de trazer a tona: A diferença entre **Library** e **Framework**! E, esta não é a unica diferenciação. Temos, ainda, a diferença entre **static** e **dinamyc**.

**Static & Dynamic**

A diferença entre static e dynamic é o momento em que a library é linkada ao código.

**Library & Framework**

**iOS**


E, sim, o código que você utiliza da SDK, nada mais é do que um conjunto de bibliotecas que sabem lidar com o sistema operacional. Inclusive, você pode ver os pacotes que são inseridos em cada versão do iOS nos simuladores:

```
/Library/Developer/CoreSimulator/Profiles/Runtimes/iOS 9.0.simruntime/Contents/Resources/RuntimeRoot/System/Library/Frameworks
```

<br/>

### Formas de distribuição

~~~
dyld: Library not loaded: @TestFramework
  Referenced from: /var/mobile/Applications/App.app/TestFramework
  Reason: image not found
~~~ 
<span class="caption text-muted">Xcode: Não achei essa biblioteca, #tryagain</span>


~~~
dyld: could not load inserted library '/TestFramework.framework/TestFramework' because no suitable image found.  
  Did find: /private/var/mobile/Containers/Data/Application/TestFramework.framework/TestFramework
~~~ 
<span class="caption text-muted">Xcode: Achei!! Mas não posso ussar essa, #sorrynotsorry</span>

<br/>

### Swift

<br/>

### Universal Framework

<br/>

### Um produto maior que o código

<br/>

### Testes

<br/>

> Por fim, como diria um bom Youtuber, deixe seus comentários e compartilhe esse post com sua família, amigos e pets. Ah, não esqueça de nos seguir nas redes sociais! Tchau! 👋
