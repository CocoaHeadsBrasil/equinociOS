---
layout:     post
title:      "Vamos falar de Xcode"
date:       2016-03-10 00:00:00
author:     "Tales Pinheiro"
category:   "xcode"
---

> Tales Pinheiro ([@talesp](https://twitter.com/talesp){:target="_blank"}) é mestre em computação pelo IME-USP, trabalhou por 8 anos com backend bancário programando principalmente em C, quando em 2007 (antes do anuncio do primeiro iPhone!) resolveu aprender Objective-C, e tem escrito Swift ha pouco mais de 6 meses.

# Vamos falar de Xcode

Algum tempo atrás apresentei no CocoaHeads SP a palestra "Design de Arcabouços: definindo uma arquitetura de fluxo de trabalho para o desenvolvimento ágil de múltiplos projetos". Nela falei um pouco sobre como usar o Xcode (e algumas técnicas/ferramentas adicionais) de forma eficiente para projetos complexos, mais ou menos o que o Hector Zarate, engenheiro do Spotify, apresentou na palestra [iOS at Spotify: From Plan to Done](https://www.youtube.com/watch?v=DWw1ankfqO0&list=PLdr22uU_wISpW6XI1J0S7Lp-X8Km-HaQW&index=14) que assisti quando fui na UIKonf - apesar da palestra dele ser menos técnica.

Mas o assunto é extenso, e o Xcode é uma ferramenta complexa e complicada, então resolvi estender e destrinchar um pouco mais o assunto. Vou falar então de forma um pouco mais detalhada sobre os tópicos a seguir:

* Projetos
* _Workspaces_
* _Build configurations_
* _Targets_
* _Build Phases_
* _Schemes_


## Projetos
Em geral, a primeira coisa que fazemos quando vamos iniciar um novo aplicativo é criar um novo projeto. É ele que contém os arquivos com código fonte, esquemas de construção dos _targets_, configurações de configuração, entre outras coisas.

![]({{ site.baseurl }}/img/talesp/project.png)

Ao criar um novo projeto no Xcode, é gerado um "arquivo _bundle_" no diretório raiz do projeto. Esse "arquivo", que na verdade é um diretório especial que o Finder exibe como arquivo, tem extensão _xcodeproj_. Ao navegarmos dentro desse diretório especial, vemos a seguinte estrutura de diretórios (extraído [daqui](https://gist.githubusercontent.com/adamgit/3786883/raw/81819edb27057d54239d96710620369bc7f7378a/.gitignore)):

~~~
 /(root)/
   /(project-name).xcodeproj/
     project.pbxproj
     /project.xcworkspace/
       contents.xcworkspacedata
       /xcuserdata/
         /(your name)/xcuserdatad/
           UserInterfaceState.xcuserstate
     /xcshareddata/
       /xcschemes/
         (shared scheme name).xcscheme
     /xcuserdata/
       /(your name)/xcuserdatad/
         (private scheme).xcscheme
         xcschememanagement.plist
~~~

Vemos então que na raiz desse diretório temos o arquivo *project.pbxproj*, e os diretórios *project.xcworkspace*, *xcshareddata* e *xcuserdata*.

O arquivo *project.pbxproj* nada mais é que um arquivo JSON. Nele temos o nome/caminho dos arquivos de código fonte (_headers_ e arquivos de implementação), _assets_ do projeto, frameworks usados pelo aplicativo, estrutura hierárquica de grupos de arquivos dentro do projeto (a forma como você organiza os arquivos dentro do projeto), as configurações das _build phases_, configurações de _targets_, entre outras coisas. Basicamente tudo que você vê ao selecionar o projeto dentro do _Project Navigator_ (exibido rapidamente com o atalho "cmd+1". Apesar de um pouco estranho, é relativamente fácil de entender esse arquivo, pois o próprio Xcode adiciona comentário descrevendo cada parte, como por exemplo:

~~~json
// !$*UTF8*$!
{
  archiveVersion = 1;
  classes = {
  };
  objectVersion = 46;
  objects = {

/* Begin PBXBuildFile section */
    02CEB9D31B6E8EDB00488E8F /* AppDelegate.swift in Sources */ = {isa = PBXBuildFile; fileRef = 02CEB9D21B6E8EDB00488E8F /* AppDelegate.swift */; };
    02CEB9D51B6E8EDB00488E8F /* ViewController.swift in Sources */ = {isa = PBXBuildFile; fileRef = 02CEB9D41B6E8EDB00488E8F /* ViewController.swift */; };
    02CEB9D81B6E8EDB00488E8F /* Main.storyboard in Resources */ = {isa = PBXBuildFile; fileRef = 02CEB9D61B6E8EDB00488E8F /* Main.storyboard */; };
    02CEB9DA1B6E8EDB00488E8F /* Assets.xcassets in Resources */ = {isa = PBXBuildFile; fileRef = 02CEB9D91B6E8EDB00488E8F /* Assets.xcassets */; };
    02CEB9DD1B6E8EDB00488E8F /* LaunchScreen.storyboard in Resources */ = {isa = PBXBuildFile; fileRef = 02CEB9DB1B6E8EDB00488E8F /* LaunchScreen.storyboard */; };
    02CEB9E81B6E8EDB00488E8F /* CatsTests.swift in Sources */ = {isa = PBXBuildFile; fileRef = 02CEB9E71B6E8EDB00488E8F /* CatsTests.swift */; };
    02CEB9F31B6E8EDB00488E8F /* CatsUITests.swift in Sources */ = {isa = PBXBuildFile; fileRef = 02CEB9F21B6E8EDB00488E8F /* CatsUITests.swift */; };
/* End PBXBuildFile section */

/* Begin PBXContainerItemProxy section */
    02CEB9E41B6E8EDB00488E8F /* PBXContainerItemProxy */ = {
      isa = PBXContainerItemProxy;
      containerPortal = 02CEB9C71B6E8EDB00488E8F /* Project object */;
      proxyType = 1;
      remoteGlobalIDString = 02CEB9CE1B6E8EDB00488E8F;
      remoteInfo = Cats;
    };
    02CEB9EF1B6E8EDB00488E8F /* PBXContainerItemProxy */ = {
      isa = PBXContainerItemProxy;
      containerPortal = 02CEB9C71B6E8EDB00488E8F /* Project object */;
      proxyType = 1;
      remoteGlobalIDString = 02CEB9CE1B6E8EDB00488E8F;
      remoteInfo = Cats;
    };
/* End PBXContainerItemProxy section */

~~~

Como podemos ver, a maior parte do arquivo é tomada pela chave _objects_, que nada mais é do que um dicionário, com uma chave hexadecimal - usada para referenciar os objetos do projeto de outras partes - e um valor, que em minhas pesquisas indicam ser sempre um outro dicionário. Para facilitar o entendimento, o Xcode adiciona ainda comentário internos, como por exemplo, `/* AppDelegate.swift in Sources */`. Em geral não vamos editar esse arquivo na mão, mas entende-lo pode facilitar bastante na hora de resolver um conflito de _merge_ com o uso de sistemas de versionamento de arquivos como _git_.

Vamos voltar ao Xcode e a estrutura de diretórios. Podemos ver que mesmo ao criar um projeto simples (por exemplo, via *File->New Project*), temos dentro da pasta do projeto uma subpasta chamada *project.xcworkspace*. A seguir descrevo os *workspaces* um pouco melhor.

## _Workspaces_

_Workspaces_ mantém a referência para projetos/subprojetos, _save states_ (janelas e abas abertas, breakpints do usuário, etc), localização do produto (_app_, biblioteca estática ou dinámica, binários temporários) e o indice de simbolos (classes, métodos, funções, testes, etc), agrupando isso de forma a facilitar a organização de projetos interrelacionados.

E como é o _workspace_ que registra o índice de símbolos, indexando todos os projetos dentro dele, _code completion_, _Jump to definition_ e outras funcionalidades sensíveis ao contexto/conteúdo funcionam de forma transparente entre os projetos. Então se você adiciona sub-projetos, navegar entre eles é bastante facilitado. Con isso, até mesmo refatorar código é feito entre vários projetos de um mesmo _workspace_.

Por padrão, todos os projetos dentro de um mesmo _workspace_ são construídos no mesmo diretório, chamado _workspace build directory_, e cada _worskpace_tem seu proprio diretório. E como todos os projetos são construídos nesse diretório, esses arquivos construídos são visíveis pelos projetos incluídos no _workspace_. O Xcode indexa esse diretório e tenta descobrir dependências implícitas. Assim, se um projeto de um aplicativo constrói também uma biblioteca como dependência direta, e esta tem um link

Mesmo que você não crie - explicitamente ou através do *cocoapods*, por exemplo - um _workspace_, é gerado um automaticamente e implicitamente para você dentro do diretório do projeto. O diretório *project.xcworkspace* é outro arquivo _bundle_, e ao analisar seu conteúdo, vemos o arquivo XML "contents.xcworkspacedata" e o arquivo "/(your name)/xcuserdatad/UserInterfaceState.xcuserstate", que efetivamente contém os breakpoints, arquivos abertos, etc. O conteúdo do arquivo contents.xcworkspace é bastante simples, conforme podemos ver abaixo:

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<Workspace
   version = "1.0">
   <FileRef
      location = "group:MyApp/MyApp.xcodeproj">
   </FileRef>
   <FileRef
      location = "group:Pods/Pods.xcodeproj">
   </FileRef>
</Workspace>
~~~

Quando usamos o CocoaPods para gerenciamento de dependências, um _workspace_ é criado, e nele é adicionado o seu projeto original, além de um novo projeto exclusivo para gerenciar as dependências. 

## _Targets_

![]({{ site.baseurl }}/img/talesp/project.png)

*Targets* especificam produtos a serem construídos (bibliotecas, apps, etc), e suas divisões mais "tradicionais" são _General_, _Info_, _Build Settings_, _Build Phases_ (além das mais recentes _Capabilities_, _Resource Tags_ e da pouco usada _Build Rules_). Um _target_ especifica um produto, organizando um conjunto de arquivos de entrada, um conjunto de regras e ações a serem tomadas sobre esses arquivos a fim de gerar um produto - um aplicativo ou biblioteca, por exemplo - como saída. Um projeto pode conter mais de um target, podendo gerar (tomando devidas precauções, como não usar APIs indisponíveis) por exemplo, uma biblioteca compilada para iOS e tvOS, compartilhando código do mesmo projeto em diferentes _targets_.

As configurações gerais de como gerar o produto estão disponíveis na aba _Build Settings_, e é organizada de forma hierarquica, podendo inclusive ser compartilhada entre projetos não relacionados (nem mesmo estando no mesmo _workspace_, através do uso de arquivos de _build configurations_. A figura abaixo mostra a visão combinada das _Build Settings_.

![Combined Build Settings]({{ site.baseurl }}/img/talesp/combined.png)

Mas recentemente tive um [problema com o CocoaPods](https://github.com/CocoaPods/CocoaPods/issues/4928), e para facilitar a identificação de onde especificamente estava o problema, utilizei a visão categorizada, disponível no botão _Levels_, que pode ser exibida abaixo.

![]({{ site.baseurl }}/img/talesp/levels.png)

Nesse exemplo podemos ver, da direita (nível hierárquico mais básico) para a esquerda (configuração final aceita como _Build Setting_ a ser utilizada) os níveis _iOS Default_, _CocoaHeads_ (primeiro as configurações do projeto, sendo aqui o nome do seu projeto), _CocoaHeads_ (aqui as configurações do _target_) e _Resolved_ (opção exibida quando _Combined_ está selecionado). Quando o projeto usa CocoaPods (ou quando você adiciona manualmente), são adicionados arquivos de configuração, que adicionam um nível adicional, chamado _Config File_. Foi nesse item que encontrei o causador do meu problema, qual era o arquivo que o estava causando, me permitindo abrir a _issue_ no repositório do CocoaPods.

### _Build Settings_
Como vimos acima, uma das abas to _Target Editor_ é a _Build Settings_, que lista todas essas configurações disponíveis, mas estudando um pouco mais, podemos ver que cada _build setting_ é na verdade uma variável que contém uma informação ou configuração específica sobre o processo de construção do produto, como para quais arquiteturas o binário será compilado. Como vimos acima, podemos especificar a configuração no projeto ou sobrescrever para um _target_ específico.

Para descobrir qual o nome da variável, basta selecionar a _build setting_ desejada e abrir a aba _Quick Help_ do _Attribute Inspector_ (por exemplo, pressionando Opt+⌘+2), como pode ser visto abaixo (onde a opção `SUPPORTED_PLATFORMS` está selecionada).

![]({{ site.baseurl }}/img/talesp/quickhelp.png)

Para uma lista das variáveis, valores disponíveis e valores padrão, consulte a pagina [Build Setting Reference](https://developer.apple.com/library/mac/documentation/DeveloperTools/Reference/XcodeBuildSettingRef/1-Build_Setting_Reference/build_setting_ref.html). Além disso, é possível criar configurações definidas por você, que podem ser usadas por sistemas de integração continua, compilação via linha de comando e, em alguns casos, como macros de preprocessamento dentro do código (ou dentro de um arquivo de _build configuration_). Também é útil utilizar configurações condicionais, permitindo por exemplo linkar com versões diferentes de uma biblioteca caso esteja sendo construído para o dispositivo ou para o simulador: já precisei usar isso quando me forneceram duas biliotecas, uma compilada apenas para arquitetura `i386` (usada pelo simulador em processadores 32 bits. Macs mais novos usam arquitetura `x86_64`) e outro compilada para `armv6` e `armv7`.

Uma referência legal de como utilizar é a pagina [Xcode Build Settings Part 1: Preprocessing](https://robots.thoughtbot.com/xcode-build-settings-part-1-preprocessing)

### _Build Phases_

As _build phases_ são um pequeno conjunto de fases necessárias para a geração do seu produto. As mais comuns são:

- _Target Dependencies_: Dependencias explicitas de outras bibliotecas. Caso seu projeto dependa de uma biblioteca não compilada por um projeto dentro da sua _workspace_ (explicita ou a implícita, incluída dentro do _bundle_ `xcodeproj`), você adiciona aqui a dependência
- _Compile Sources_: A lista de todos os arquivos de código fonte compilados para gerar seu produto. Podemos passar _flags_ de compilação (a lista depende do compilador, mas para o clang a lista completa está [aqui](http://clang.llvm.org/docs/UsersManual.html#command-line-options)). Durante a migração de projetos que não usavam _Automatic Reference Couting_ era comum ir migrando o projeto aos poucos, e marcar arquivo por arquivo quais usavam ARC ou não. Além de ser possível tratar individualmente por arquivo, é possível tratar de forma global via _Build Setting_. Quando trabalho por mais tempo em um produto, gosto de ligar por exemplo as opções `-Weverything` e para a geração do binário final, `-Werror`. Assim tenho uma analise minuciosa do compilador, e todos os _warnings_ são tratados como erro. Infelizmente em projetos mais curtos não é possível ter tanta garantia, pois a validação disso as vezes exige um tempo maior.
- _Link binary with Libraries_: Caso alguém (ou alguma empresa) tenho lhe fornecido uma bilioteca pré-compilada (por exemplo, o SDK da Hockey ou Fabric), ela estará listada aqui. Normalmente a integração é feita de forma automática por gerenciadores de dependência, mas pode ser necessário adicionar manualmente aqui.
- _Copy bundle resources_: arquivos de storyboard, assets como imagens, vídeos, e arquivos de áudio, por exemplo, são listados aqui.

Além dessas mais comuns e incluídas por padrão, pode ser útil - ou mesmo necessário - incluir uma (ou mais) através da opção _New run script phase_. Por exemplo, ferramentas de geração de código como [Natalie](https://github.com/krzyzanowskim/Natalie) ou [mogenerator](https://github.com/rentzsch/mogenerator) podem ter scripts para analisar, respectivamente, os arquivos de storyboard ou core Data, e ferramentas de _lint_, como [OCLint](http://oclint.org) ou [SwiftLint](https://github.com/realm/SwiftLint) podem fazer analises do código e identificar coisas fora do padrão do time ou _code smells_, sendo nesse caso adicionar scripts no começo do processo, e ferramentas como Kockeyapp ou Fabric podem pedir para adicionar um script no final, habilitando a nova versão no serviço deles. A imagem abaixo mostra como adicionar uma nova _run script phase_

![]({{ site.baseurl }}/img/talesp/runscript.png)

### _Build configurations_

_Build Configurations_ são conjuntos de _build settings_, a serem usadas por uma determinada compilação, especificada por um _scheme_. Por padrão, temos sempre duas configurações dessas: _Debug_ e _Release_. É através dela que podemos, por exemplo configurar para que sob certas condições, um _build setting_ específico seja usado. Por exemplo, podemos duplicar a _build configuration_ _Release_ como exibido na imagem abaixo, para ter configurações de construção do produto diferente para, digamos, um ambiente corporativo. (a imagem abaixo foi tirada do projeto [ribotTeamiOS-tvOS](https://github.com/manuelmarcos/ribotTeamiOS-tvOS), que demonstra o compartilhamento de código de forma simples entre iOS e tvOS)

![]({{ site.baseurl }}/img/talesp/ribotTeam.png)

Podemos, com a ajuda da técnica esposta no artigo da Thoughtbot, gerar binários diferentes com _Bindle ID_ diferentes para o ambiente empresarial usando distribuição _Enterprise_, por exemplo. As facilidades que isso nos dá são muitas!

## _Schemes_

Por ultimo gostaria de falar um pouco sobre _Schemes_, que são um conjunto de ações que definem quais _targets_ devem ser construídos, qual conjunto de configurações devem ser usadas para cada tipo de compilação (execução no simulador/dispositivo, teste, _profile_, para analise estática e arquivamento para distribuição.

Você pode ter multiplos esquemas, mas apenas um está ativo de cada vez. Por padrão os _schemes_ são armazenados no projeto, mas caso você deseje por exemplo usar o Xcode Server como servidor de integração continua através do uso de Xcode Bots, o _scheme deve ser compartilhado via workspace. A figura abaixo exibe as opções normalmente disponíveis na aba _Build_, indicando qual _target_ deve ser construído para cada tipo de execução. Como podemos ver, não faz sentido compilar o _bundle_ de testes para a compilação para distribuição - _archive_. Podemos ver também que esse esquema não foi compartilhado, o que impede seu uso pelos Bots do Xcode.

![]({{ site.baseurl }}/img/talesp/schemes.png)

Na aba _Run_ temos a chance de configurar a forma como o app é executado - no simulador ou no device - e podemos passar "parâmentro de linha de comando" para a execução do app. Já usei isso para habilitar/desabilitar mensagens no console, em um app onde o requisito era não logar nada, mas para desenvolvimento ligavamos as mensagens de _debug_ - inclusíve controlando níveis de mensagens de erro (info/ _warning_, erro, etc) de acordo com o parâmentro. Assim não corriamos a chance de acidentalmente enviar o app para produção exibindo os logs. Essa não é a unica forma que isso pode ser feito, mas foi a forma que usamos, e isso ter outras utilidades para você :D

Outra funcionalidade da aba _Run_ que já foi muito mais usada no passado - na era pre-ARC - foi habilitar a opção _Enable Zombie Objects_ na aba _Diagnostics_, o que permitia detectar e corrigir vários bugs de gerenciamento de memória. Você pode ver um pouco mais [aqui](http://cocoadev.com/DebuggingAutorelease), mas não vejo mais tanta utilidade no dia a dia. A imagem abaixo exibe essa aba e mais algumas opções.

![]({{ site.baseurl }}/img/talesp/zombies.png)


# Conclusão

Não quis de forma nenhuma descrever todas essas funcionalidades e conceitos de forma estensiva, até porque ainda teria MUITO a escrever. Mas acho importante conhecer esses conceitos, nos ajudam na hora de configurar como o app é construído/arquitetado, como é compilado, nos permitem pensar em novas formas de 

Espero que tenham gostado, e que no futuro eu consiga escrever um outro artigo que daria uma espécie de sequencia para esse artigo. Bom proveito e bons _hacks_ na forma de construír seus apps :)