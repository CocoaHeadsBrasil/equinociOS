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

Eu sempre trabalhei com o desenvolvimento de apps para o usuário final do produto, os famosos *ipas* que vão para a loja. Mas, recentemente mudei meu trabalho para uma empresa que desenvolve bibliotecas para outras empresas.

<br/>

<center>
<img alt="I'm going on an adventure" src="https://45.media.tumblr.com/3ad2585f368bd07136af9d01aa285906/tumblr_n7y9otYV1i1s9t7xfo1_500.gif"/>
</center>

<br/>

Mesmo tendo conhecimento sobre as definições e paradigmas de *libraries* e *frameworks*, eu enfrentei alguns problemas que nunca tinha enfrentado e precisei rever alguns pontos e detalhes da plataforma Apple. Especialmente o processo de distribuição de binários.

Minha intenção, aqui, é trazer alguns conceitos que permeiam as bibliotecas, mesmo que superficialmente!

<br/>

### Library?

Mesmo que você nunca tenha desenvolvido esse tipo de distribuição, bibliotecas fazem parte do seu dia-a-dia de desenvolvimento. Seja com código de terceiros como *AFNetworking* (ou *AlamoFire*), *OCMock*, *Realm*, entre muitos outros, ou com o *Foundation*, *UIKit* e os mais diversos módulos da SDK do iOS e do OS X.

São formas de reutilizar seu código em múltiplos projetos! Você poderia copiar e colar os arquivos para cada novo app, mas, além disso não ser prático, dificulta, e muito, a manutenção e integração.

Além disso, são formas de produção e distribuição de código que não são atrelados a um produto específico. Como, por exemplo, um set de lógica para tratar de um protocolo de serialização, ou reconhecimento de forma em visão computacional.

<br/>

### Library x Framework

<center><img alt="Static library e framework" src="{{ site.baseurl }}/img/igorcferreira/library_types.jpg"/></center>

Aí está a primeira coisa que gostaria de trazer a tona: A diferença entre **Library** e **Framework**! Só que para explicar melhor a diferença temos que, primeiro, declarar a diferença entre **static link** e **dynamic link**.

**Static & Dynamic**

A diferença entre uma *static linked library* e *dynamic linked library* é a forma como o código será usado pela aplicação.

O código **static linked** em sua aplicação é adicionado ao binário da aplicação. Assim, faz parte do produto final, portanto é carregado em memória junto com o resto da aplicação e caso haja uma atualização na biblioteca, o produto precisa ser recompilado. Até recentemente, por conta do modelo de assinatura de binários e a estrutura do sistema operacional, essa era a única forma possível de biblioteca para iOS.

Por sua vez, o código **dynamic linked** não é associado diretamente ao binário da aplicação. É algo separado do produto final e seu carregamento de memória acontece antes que seus *symbols* sejam resolvidos. Além disso, já que é algo separado, seu código pode ser alterado sem precisar recompilar a aplicação. Este modelo já está presente nos aplicativos para OS X [há bastante tempo (arquivos *dylib*)](https://developer.apple.com/library/mac/documentation/DeveloperTools/Conceptual/DynamicLibraries/000-Introduction/Introduction.html).

Até o iOS 8, o único modelo de projeto disponível para bibliotecas era a *[Static Library](https://developer.apple.com/library/ios/technotes/iOSStaticLibraries/Articles/creating.html)*, que resulta na compilação de um arquivo *.a*. Um arquivo com a compactação dos múltiplos pré compilados (*.o*) das classes da biblioteca.

<center><img alt="Tá me zoando?" src="{{ site.baseurl }}/img/igorcferreira/are_you_kidding_me.jpg"/></center>

Mas, relaxe, agora já é possível criar *[Frameworks](https://developer.apple.com/library/mac/documentation/MacOSX/Conceptual/BPFrameworks/Concepts/WhatAreFrameworks.html)*!

Para entender um pouco:

**Library** 

Como eu mencionei logo acima, uma *static library* é um arquivo compactado de todos os pré compilados de suas classes. Incluindo todas as arquiteturas de seu produto (armv7 e arm64, por exemplo). Depois, quem for usar basta importar este arquivo .a e avisar o Xcode que a biblioteca precisa ser linkada ao seu produto.

O desenvolvimento é bem simples: Basta criar as classes e compilar tudo! Sem segredo e sem dificuldade.

<center><img alt="Arquivos de uma static lib" src="{{ site.baseurl }}/img/igorcferreira/library_structure.jpg"/></center>

Algo muito lindo, muito simpático e funcional!

Mas, digamos que sua biblioteca tenha alguns arquivos *xib* para a construção de uma view. E, uma vez que tenho uma view, tenho arquivos *string* de localização. Esses arquivos, mesmo que associados à sua biblioteca não são compactados no .a final.

Assim, junto com o .a, você precisa enviar todo e qualquer *resource* que sua biblioteca usar. O mesmo vale para os *headers* públicos. 

Se você usa o [CocoaPods](http://cocoapods.org/), já deve ter visto o *Build Phase* *Copy Pods Resources* e o script *Pods-resources.sh*. Eles servem para, justamente, garantir que o seu app carregará todos os arquivos necessários, sem que você precise lidar com isso.

<center>🎉</center>

Aí temos um segundo problema, já que a biblioteca é agregada ao binário do app, toda biblioteca é carregada em memória por todo aplicativo que a usar.

Isso significa que todos os aplicativos que usam o *AFNetworking* como biblioteca estática carregam o código em memória?

<center><img alt="Sim :(" src="{{ site.baseurl }}/img/igorcferreira/yes.jpg"></center>

Agora imagine se todos os aplicativos carregassem o *UIKit* ou o *Foundation*!? E qual a solução para esse problema?

**Framework**

Em uma tradução livre da documentação da Apple:

> Uma *framework* é um diretório estruturado que encapsula *resources* compartilhados, como bibliotecas dinâmicas, arquivos *nib*, imagens, *localized strings*, *headers* e documentação em uma distribuição única. Mais de um aplicativo pode utilizar do mesmo recurso simultaneamente. O sistema carrega os itens em memória conforme o necessário e compartilha uma cópia entre todas as aplicações.  

E, olhando para a estrutura de uma framework, percebe-se que é possível encapsular diferentes versões da mesma biblioteca no mesmo arquivo de distribuição:

~~~
MyFramework.framework/
    MyFramework  -> Versions/Current/MyFramework
    Resources    -> Versions/Current/Resources
    Versions/
        A/
            MyFramework
            Resources/
                English.lproj/
                    InfoPlist.strings
                Info.plist
        Current  -> A
~~~
<span class="caption text-muted">O current é um link para a versão A</span>

Assim, caso diferentes apps estejam usando diferentes versões da *framework*, isso não se torna um problema.

E a chave é a *dynamic library*!

O código que você utiliza da SDK, é um conjunto de *frameworks* que sabem lidar com o sistema operacional. Assim, o *UIKit*, por exemplo, é carregado em memória uma única vez e usado por todos os aplicativos. Além disso, a biblioteca pode ser atualizada sem precisar recompilar todos os aplicativos que estão na loja.

Inclusive, você pode ver as frameworks que são carregadas nos simuladores no path

`/Library/Developer/CoreSimulator/Profiles/Runtimes/iOS 9.0.simruntime/Contents/Resources/RuntimeRoot/System/Library/Frameworks/`

***

Até a oitava versão do sistema operacional *mobile*, não havia uma forma simples de criar frameworks. Isso era uma exclusividade do OS X. 

Alguns, para criar essa estrutura criaram um script para compilar a biblioteca e criar o diretório da forma correta. Mesmo assim, só era possível criar uma **static framework**, já que não é possível criar uma *dynamic library* para iOS.

Agora, o Xcode dispõe de um modelo de projeto do tipo *Cocoa Touch Framework* onde a estruturação da pasta é controlada por scripts do sistema e o desenvolvimento é semelhante ao desenvolvimento de uma biblioteca estática ou de um aplicativo:

<center><img src="{{ site.baseurl }}/img/igorcferreira/framework_structure.jpg" alt="Estrutura de uma framework"/></center>

Olhando para o código (em um projeto começado do zero), a framework tem 2 arquivos a mais do que uma static library:

- *Info.plist*: O arquivo .framework precisa de um arquivo de informações sobre o binário. Nele contém nome, língua, versão e outras coisas básicas.

- *[Umbrella header](https://developer.apple.com/library/mac/documentation/MacOSX/Conceptual/BPFrameworks/Tasks/IncludingFrameworks.html)*: O umbrella header é um arquivo .h que importa, ou incluí, todos os demais headers da biblioteca. Assim, é possível apenas usar `#import <TestFramework/TestFramework.h>` em vez de importar cada arquivo individualmente.

No seu umbrella header, você encontrará a exportação de duas constantes:

~~~objc
//! Project version number for TestFramework.
FOUNDATION_EXPORT double TestFrameworkVersionNumber;

//! Project version string for TestFramework.
FOUNDATION_EXPORT const unsigned char TestFrameworkVersionString[];
~~~

Eles são responsáveis por exteriorizar a versão  e o nome da biblioteca, muito úteis para controle de código e link correto do binário pelo sistema.

Além disso, a IDE vai te permitir, através do `Build Phases`, separar seus headers em *Public*, *Private* e *Project*.

<center><img src="{{ site.baseurl }}/img/igorcferreira/framework_header.jpg" alt="Estrutura dos headers"></center>

Nessa divisão tem se que:

- Public headers estarão visíveis para quem importar a framework
- Private headers serão incluídos no .framework dentro de uma pasta /Private, assim poderão ser usados, apenas de não serem mostrados pelo autocomplete da IDE, por exemplo.
- Project headers são visíveis durante o desenvolvimento da biblioteca mas não são exportados para o arquivo .framework.

<br/>

### Formas de distribuição

O mais diferente no desenvolvimento de uma biblioteca para um aplicativo (e o mais chato) é a forma de distribuição.

É possível utilizar ferramentas como CocoaPods ou [Carthage](https://github.com/Carthage/Carthage) para isso, especialmente para projetos open source ou bibliotecas internas. Estas, são conjuntos de scripts para lidar com o processo de compilação e referência da biblioteca. Mas, nem sempre você pode fazer a distribuição dessa forma.

O CocoaPods pode ser usado tanto para *static libraries* (sua origem) quanto para frameworks e ele, usa seu *Podspec* para especificar a forma de compilação. Assim, não é nem necessário um .xcodeproj, apenas os arquivos fonte e *resources*. 

Já o Carthage, utiliza os *shared schemes* de um projeto para compilar a framework e, depois, se preocupa em referenciá-las no seu aplicativo.

Os dois abordam a compilação de formas bem diferentes, mas, depois, o processo de referência da framework tem pontos comuns, que servem para lidar com as chatices disso.

Caso você esteja simplesmente mandando os arquivos compilados para um cliente ou outro dev, as coisas ficam um pouco mais complicadas.

***

No caso de uma *static library*, basta copiar o arquivo .a e os *resources* para seu projeto, adicionar a referência do binário em `Link Binary With Libraries`, o path dos headers em `HEADER_SEARCH_PATHS` e, caso seja necessário, as demais flags.

Caso você esqueça de algum desses passos seu código, simplesmente, não compila e você consegue identificar qual dos pontos esqueceu. 😄

***

Para frameworks, arrasta os arquivos para `Linked Frameworks and Libraries`, configura as demais flags e é isso, certo? Não. Isso funciona apenas para frameworks do sistema!

Frameworks podem ser um pouco mais complicadas, especialmente quando se tratar de uma *universal framework* (trato disso mais abaixo). Sendo muito comum acabar com os error:

**Library not loaded**

~~~
dyld: Library not loaded: @TestFramework
  Referenced from: /var/mobile/Applications/App.app/TestFramework
  Reason: image not found
~~~ 
<span class="caption text-muted">Xcode: Não achei essa biblioteca, #tryagain</span>

O primeiro problema mais comum na importação de uma biblioteca, que não é nativo do sistema, é que os arquivos precisam ser incluídos em `Embedded Binaries`. Assim, eles serão adicionados ao seu `.ipa`, na pasta `Frameworks`. Quando o aplicativo for instalado, o iOS irá usar o diretório `Frameworks` para procurar por algo que não está no sistema (tanto por nome, quanto por versão).

<center><img alt="Frameworks usadas pelo WWDC" src="{{ site.baseurl }}/img/igorcferreira/imported_framework.jpg"/></center>

> Aqui, o seu ambiente de desenvolvimento pode ser seu inimigo! Digamos que você tenha a framework em um app A. Começou a desenvolver o app B, mas esqueceu de embedar os binários. No seu device, irá funcionar lindamente, porque ele já tem uma versão válida instalada por conta do app A!

**Could not load library**

~~~
dyld: could not load inserted library '/TestFramework.framework/TestFramework' because no suitable image found.  
  Did find: /private/var/mobile/Containers/Data/Application/TestFramework.framework/TestFramework
~~~ 
<span class="caption text-muted">Xcode: Achei!! Mas não posso ussar essa, #sorrynotsorry</span>

O segundo erro muito comum é não ter a versão correta da framework no sistema. Isso pode acontecer por X motivos, mas, para citar alguns:

- *`Framework` não foi assinada*: Assim como seu .ipa, seu arquivo .framework também tem que ser assinado antes de ser enviado para a loja
- *Versão da framework*: A versão da framework foi atualizada, mas a compilação não está dentro da pasta .framework
- *Arquitetura errada*: Você está embarcando uma framework compilada para uma arquitetura diferente do que se quer executar. Por exemplo, uma framework de simulador em um device e vice-versa.

***

Para evitar esses problemas que o Carthage tem o [CopyFrameworks.swift](https://github.com/Carthage/Carthage/blob/master/Source/carthage/CopyFrameworks.swift) e o CocoaPods tem o [Pods-frameworks.sh](https://github.com/CocoaPods/CocoaPods/blob/5d0c4c7a47925a15a7bd80fd46b0233a8f670f98/lib/cocoapods/generator/embed_frameworks_script.rb#L46)! São scripts que, em uma explicação superficial, são responsáveis por:

- *Remover arquivos indevidos*: A framework gerada pelo Xcode mantém os headers privados em uma pasta `Private`, além de alguns outros arquivos que não precisam, necessariamente, ser incluídos no produto final.

- *Copiar os `debug symbols` para a framework*: Como a biblioteca é compilada antes de ser incluída no diretório, pode-se perder os símbolos de *debug*, o que pode ser bem ruim, caso se use uma ferramenta como `Crashlytics` ou `HockeyApp` (além de alguns bugs no App Store). Esse step é mais válido em bibliotecas open-source, onde se tem acesso ao código para *breakpoint* e demais.
 
- *Strip a framework para que seja mantido apenas as arquiteturas válidas*: Em uma *universal framework*, ou em uma *fat* framework, pode conter a mesma biblioteca compilada para as diversas arquiteturas. É importante remover todas as arquiteturas que não são relevantes ao aplicativo.

- *Copiar as bibliotecas Swift*: Para bibliotecas feitas em Swift e compiladas em uma versão do Xcode anterior ao 7, é necessário copiar algumas libs do Swift para o seu aplicativo
 
- *Re-assinar as frameworks*: Por fim, mas não menos importante, reassinar os arquivos .framework com o mesmo certificado que assina a aplicação. Assim, podendo ser enviado para a loja.

***

Então, em um resumo, se lembrar de seguir os processos acima e associar seu arquivo .framework no `Embedded Binaries`, sua framework estará funcionando lindamente!

<br/>

### Universal Framework

Mas, um conceito importante para a distribuição de uma framework é: *universa framework*.

Imagine uma biblioteca como o [Alamofire](https://github.com/Alamofire/Alamofire) ou [AFNetorking](https://github.com/AFNetworking/AFNetworking), onde o mesmo código pode ser usado em iOS, OS X, watchOS e tvOS. No `xcodeproj` dessas bibliotecas há um target para cada uma dessas plataformas. Assim, você acaba compilando 8 frameworks diferentes (1 para cada sistema e 1 para os respectivos simuladores).

Seria um empecilho muito grande ter de lidar com 8 binários. Principalmente no switch entre desenvolvimento e archive!

Para solucionar esse problema, é feita uma *universal framework*. Onde, usando `lipo`, agrega-se os múltiplos binários das bibliotecas em um único arquivo.

~~~
$lipo -create -output MyFramework.framework/MyFramework MyFramework-iphonesimulator.framework/MyFramework MyFramework-iphoneos.framework/MyFramework MyFramework-watchos.framework/MyFramework
$lipo MyFramework.framework/MyFramework
Architectures in the fat file: MyFramework.framework/MyFramework are: i386 x86_64 armv7 armv7s arm64 armv7k
~~~

Depois, refaz a estrutura do diretório .framework para conter os headers e assets de cada versão da biblioteca separados em subdiretórios.

~~~
MyFramework.framework/
	MyFramework
	Resources/
		English.lproj/
			InfoPlist.strings
		Info.plist
	Headers/
		MyFramework.h
		iphoneos/
			OtherHeader.h
       watchos/
	       OtherHeader.h
~~~

E, como já expliquei no item anterior, se faz necessário remover os arquivos inválidos no momento da instalação. Por exemplo, remover a pasta `watchos` quando gerar um app para `iphoneos`.

O mesmo vale para o arquivo pré compilado. Sendo necessário executar um `lipo` para remover as arquiteturas não suportadas:

~~~
$lipo remove armv7k -output MyFramework.framework/MyFramework MyFramework.framework/MyFramework
$lipo remove i386 -output MyFramework.framework/MyFramework MyFramework.framework/MyFramework
$lipo remove x86_64 -output MyFramework.framework/MyFramework MyFramework.framework/MyFramework
~~~ 

<br/>

### Swift

Estamos em um período de transição entre o Objective-C e o Swift (pelo menos para muitos). E, igual à linguagem, são dois mundos diferentes quando se fala de bibliotecas.

1 - O Swift não permite o uso de *static library*! Dado todos os detalhes envolvendo o runtime do Swift, compatibilidade de versões e otimizações de compilação, não faz sentido fazer um link estático.

2 - Mesmo que Swift não tenha headers, a framework ainda tem um umbrella header para exteriorizar versão e nome da biblioteca. Além de permitir a importação da mesma no Objective-C.

3 - Caso você esteja usando uma biblioteca feita em Swift em um projeto Objective-C (independente se app ou outra biblioteca), é preciso setar `YES` para a flag `EMBEDDED_CONTENT_CONTAINS_SWIFT`. Isso faz com que o Xcode saiba lidar com as bibliotecas de runtime do Swift.

4 - Caso você esteja criando uma biblioteca em Objective-C, é importante configurar um  *module map* e habilitar `DEFINES_MODULE`. Assim, você poderá usar `import MyFramework`, não precisará de um bridge header e evita conflito de nomenclaturas.

<center><img src="{{ site.baseulr }}/img/igorcferreira/module_map.jpg" alt="Module map"></center>

<br/>

### Arquitetura

Agora que o conceito e detalhamento de frameworks e static library foi feito, é importante saber que a arquitetura de uma boa biblioteca incluí, também, a simplicidade de uso, proteção de entrada (`NSAssert`) e design da camada pública da sua API. É isso que o cliente irá ver e usar!

Não vou entrar em detalhes sobre isso aqui porque esse post já está gigante. Talvez outro dia.

### Testes

O último ponto técnico que gostaria de levantar aqui é em relação ao testes.

Além dos testes unitários, de integração e de UI, é importante incluir um *sample app*. Ele servirá como um teste de que a framework pode ser importada em um projeto sem problemas e que a interface pública faz sentido!

<br/>

### Um produto maior que o código

Se você aguentou até aqui, lembre-se: **Bibliotecas não são apenas código!**

É importante criar uma boa documentação, guideline de implementação, taggear e versionar corretamente seus releases, códigos de exemplo, pensar em manutenção e evolução...

Seu cliente é o desenvolvedor que vai usar o seu código. Assim como uma boa UI/UX é vital na retenção de usuários, a camada pública e documentação da sua API pode atrair ou afugentar usuários!

<br/>

> Por fim, como diria um bom Youtuber, deixe seus comentários e compartilhe esse post com sua família, amigos e pets. Ah, não esqueça de nos seguir nas redes sociais! Tchau! 👋
