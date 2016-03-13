---
layout:     post
title:      "Bibliotecas 📚"
date:       2016-03-17 00:00:00
author:     "Igor Ferreira"
header-img: "img/igorcferreira/library.jpg"
category:   "library"
---
> Eu ([@icastanheda](https://twitter.com/icastanheda)) sou desenvolvedor mobile desde de 2012, quando ARC, storyboard e o Ice Cream Sandwich eram recentes no mundo. Hoje em dia, trabalho com desenvolvimento de bibliotecas para os ambientes Apple (OSX, iOS, WatchOS e tvOS).

### Prefácio
Eu sempre trabalhei com o desenvolvimento de apps para o usuário final do produto (primeiro para desktop e depois para mobile). Mas, recentemente mudei meu trabalho para uma empresa que desenvolve bibliotecas para outras empresas.

<center>
<img alt="I'm going on an adventure" src="https://45.media.tumblr.com/3ad2585f368bd07136af9d01aa285906/tumblr_n7y9otYV1i1s9t7xfo1_500.gif"/>
<span class="caption text-muted">O prazer de fazer algo novo</span>
</center>

Mesmo tendo um conhecimento sobre as definições e paradigmas de libraries e frameworks, eu enfrentei alguns problemas que nunca tinha enfrentado e precisei estudar mais a fundo alguns conceitos e detalhes da plataforma Apple. 

Minha intenção, aqui, é trazer alguns desses pontos, mesmo que superficialmente.

### <abbr title="1">1️⃣</abbr> Library?

```
/Library/Developer/CoreSimulator/Profiles/Runtimes/iOS 9.0.simruntime/Contents/Resources/RuntimeRoot/System/Library/Frameworks
```

### <abbr title="2">2️⃣</abbr> Library x Framework

### <abbr title="3">3️⃣</abbr> Formas de distribuição

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

### <abbr title="4">4️⃣</abbr> Swift

### <abbr title="5">5️⃣</abbr> Universal Framework

### <abbr title="6">6️⃣</abbr> Um produto maior que o código

### <abbr title="7">7️⃣</abbr> Testes


> Por fim, como diria um bom Youtuber, deixe seus comentários e compartilhe esse post com sua família, amigos e pets. Ah, não esqueça de nos seguir nas redes sociais! Tchau! 👋