---
layout:     post
title:      "Vamos falar de Xcode"
date:       2016-03-06 00:00:00
author:     "Tales Pinheiro"
header-img: "img/fabri/pia17936_evening_star.jpg"
category:   "open-source"
---

> Tales Pinheiro ([@talesp](https://twitter.com/talesp){:target="_blank"}) é mestre em computação pelo IME-USP, trabalhou por 8 anos com backend bancário programando principalmente em C, quando em 2007 (antes do anuncio do primeiro iPhone!) resolveu aprender Objective-C, e tem escrito Swift ha pouco mais de 6 meses.

# Vamos falar de Xcode

Algum tempo atrás apresentei no CocoaHeads SP a palestra "Design de Arcabouços: definindo uma arquitetura de fluxo de trabalho para o desenvolvimento ágil de múltiplos projetos". Nela falei um pouco sobre como usar o Xcode (e algumas técnicas/ferramentas adicionais) de forma eficiente para projetos complexos, mais ou menos o que o Hector Zarate, engenheiro do Spotify, apresentou na palestra [iOS at Spotify: From Plan to Done](https://www.youtube.com/watch?v=DWw1ankfqO0&list=PLdr22uU_wISpW6XI1J0S7Lp-X8Km-HaQW&index=14) que assisti quando fui na UIKonf - apesar da palestra dele ser menos técnica.

Mas o assunto é extenso, e o Xcode é uma ferramenta complexa e complicada, então resolvi extender e destrinchar um pouco mais o assunto. Vou falar então de forma um pouco mais detalhada sobre os tópicos a seguir:

* Projetos
* Workspaces
* Build configurations
* Targets
* Build Phases
* Schemes


## Projetos
Em geral, a primeira coisa que fazemos quando vamos iniciar um novo aplicativo é criar um novo projeto. É ele que contém os arquivos com código fonte, esquemas de construção dos _targets_, configurações de configuração, entre outras coisas.

![]({{ site.baseurl }}/img/talesp/project.png)

Ao criar um novo projeto no Xcode, é gerado um "arquivo _bundle_" no diretório raiz do projeto. Esse "arquivo", que na verdade é um diretório especial que o Finder exibe como arquivo, tem extensão _xcodeproj_. Ao navegarmos dentro desse diretório especial, vemos a seguinte estrututura de diretórios (extraído [daqui](https://gist.githubusercontent.com/adamgit/3786883/raw/81819edb27057d54239d96710620369bc7f7378a/.gitignore)):

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

Vemos então que na raíz desse diretório temos o arquivo *project.pbxproj*, e os diretórios *project.xcworkspace*, *xcshareddata* e *xcuserdata*.

O arquivo *project.pbxproj* nada mais é que um arquivo JSON. Nele temos o nome/caminho dos arquivos de código fonte (_headers_ e arquivos de implementação), _assets_ do projeto, frameworks usados pelo aplicativo, estrutura hierarquica de grupos de arquivos dentro do projeto (a forma como você organiza os arquivos dentro do projeto), as configurações das _build phases_, configurações de _targets_, entre outras coisas. Basicamente tudo que você vê ao selecionar o projeto dentro do _Project Navigator_ (exibido rapidamente com o atalho "cmd+1". Apesar de um pouco estranho, é relativamente fácil de entender esse arquivo, pois o próprio Xcode adiciona comentário descrevendo cada parte, como por exemplo:

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

E como é o _workspace_ que registra o indice de simbolos, indexando todos os projetos dentro dele, _code completion_, _Jump to definition_ e outras funcionalidades sensiveis ao contexto/conteúdo funcionam de forma transparente entre os projetos. Então se você adiciona sub-projetos, navegar entre eles é bastante facilitado. Con isso, até mesmo refatorar código é feito entre vários projetos de um mesmo _workspace_.

Por padrão, todos os projetos dentro de um mesmo _workspace_ são construídos no mesmo diretório, chamado _workspace build directory_, e cada _worskpace_tem seu proprio diretório. E como todos os projetos são construídos nesse diretório, esses arquivos construídos são visiveis pelos projetos incluídos no _workspace_. O Xcode indexa esse diretório e tenta descobrir dependencias implicitas. Assim, se um projeto de um aplicativo constrói também uma biblioteca como dependencia direta, e esta tem um link

Mesmo que você não crie - explicitamente ou através do *cocoapods*, por exemplo - um _workspace_, é gerado um automáticamente e implicitamente para você dentro do diretório do projeto. O diretório *project.xcworkspace* é outro arquivo _bundle_, e ao analisar seu conteúdo, vemos o arquivo XML "contents.xcworkspacedata" e o arquivo "/(your name)/xcuserdatad/UserInterfaceState.xcuserstate", que efetivamente contém os breakpoints, arquivos abertos, etc. O conteúdo do arquivo contents.xcworkspace é bastante simples, conforme podemos ver abaixo:

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

## _Build configurations_

## _Targets_

*Targets* especificam produtos a serem construídos (bibliotecas, apps, etc), e contém instruções de como

## _Build Phases_

## _Schemes_
