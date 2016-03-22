---
layout:     post
title:      "App Extension - Today Widget"
subtitle:   "Introdução ao Today Extension"
date:       2016-03-22 00:00:00
author:     "Renato Matos"
header-img: "img/renatosarro/header-renatosarro.jpg"
category:   ios
---

# App Extensions

## Overview

**App extensions** é uma feature presente no Xcode desde a versão 6 da IDE.
Essa feature, tem como objetivo melhorar a **experiência do usuário** na forma em que ele se relaciona com seu app.

Uma das partes mais interessantes, é que você pode expandir uma ou mais funcionalidades do seu app para outros app's, fazendo com os usuários possam interagir com seu app, mesmo se estiverem utilizando outro aplicativo.

---

### Podemos criar 6 tipos de extensions:

- **Share:** 
Compartilhar conteúdo com alguém, ou outros aplicativos

- **Action:** 
Exibir um conteúdo ou executar uma ação dentro de outro contexto

- **Photo Editing:**
Editar vídeo ou foto

- **Document Provider:**
Acessar e/ou gerenciar um repositório de arquivos

- **Custom Keyboard:**
Substituir o teclado padrão iOS por outro personalizado

- **Today Widget:**
Obter informações relevantes ou executar ações rápidas no Centro de Notificações (Today Notification)


---
##<a name="markdown-pane"></a> Today Notification - Widget

Neste post, vamos abordar o **Today Notification**. A idéia aqui, é dar uma introdução ao assunto e estimular você a explorar mais esta feature que tanto agrega à experiência que o usuário tem ao utilizar seu app.

>Um **App Extension** dentro do **Centro de Notificações**, é chamado **Widget**.

**Widgets** tem o objetivo de trazer insformações que são relevantes ao usuário naquele momento como por exemplo, informações sobre o Clima, compromissos, ou cotação da bolsa (widgets que já vem habilitados por default).

**Today Widgets** precisam necessariamente garantir que as informações exibidas estejam **sempre atualizadas**, responder de forma adequada às interações do usuário e ter uma **boa performance** (sempre importante lembrar deste ponto, pois se não cuidar do consumo de memória com carinho, seu widget pode ser finalizado pelo sistema).

--

>Chega de **~blá blá blá~** e vamos à parte prática da parada \o/

--

Neste post, abordaremos 4 pontos principais sobre a implementação do **Today Widget**:

- Setup Inicial - Adicionando Today Widget
- Manipular elementos visuais em runtime
- Atualização do conteúdo
- Abrir um app que esteja no mesmo contexto do widget

--

###Setup Inicial

Abra o **Xcode** e clique em **Create a new Xcode project**. Para este exemplo, podemos criar um Single View Application. Defina um nome para o seu projeto e let's play a game =)

<img src="{{ site.baseurl }}/img/renatosarro/img1.png">

Com o projeto aberto, clique em **File > New > Target**.

Em Application Extension, selecione **Today Extension**.

<img src="{{ site.baseurl }}/img/renatosarro/img2.png">

Defina um nome para o projeto.

<img src="{{ site.baseurl }}/img/renatosarro/img3.png">

Após isto, você será questionado se deseja ativar um scheme para o seu novo Target. 

######Particularmente, prefiro já ativar, pois desta forma você poderá rodar e debugar o seu target de forma independente.
<img src="{{ site.baseurl }}/img/renatosarro/img4.png">

>Se preferir, não precisa ativar agora, pode adicionar depois se achar necessário

Ao rodar o projeto você pode notar que ele já se encontra disponível no **Today Notification**.

<img src="{{ site.baseurl }}/img/renatosarro/img5.png">

###Manipulando Elementos Visuais

Outra coisa bacana do **Today Extension** é a liberdade que você tem para manipular os elementos visuais, como se estivesse trabalhando em uma tela do seu app.

Ao adicionar um novo target para o **Today Extension**, você pode notar - no **Project Navigator** - que foi criada uma pasta com o nome do seu target e dentro dela foi adicionado alguns arquivos como, um Class para View Controller (.h e .m se o projeto for Objective-C e .swift se o projeto for Swift) , um Storyboard e um Info.plist.

---

Abrindo a implementação da View Controller, você pode notar que ela implementa o protocolo **NCWidgetProviding**.

<script src="https://gist.github.com/renatosarro/012a80e28805403e00b4.js"></script>

Este protocolo, possui alguns métodos que permitem você customizar o layout e alguns comportamentos do seu widget.

>Widget é uma extensão no ***Today Notification***.

Agora abra o Storyboard. Note que temos um object **View Controller** adicionado com um layout padrão, com um label "Hello World". É nesta view que vamos trabalhar nosso layout.

>Para o post não ficar muito longo e cansativo, dividi a implementação em 3 passos que podemos acompanhar nos vídeos abaixo. Enjoy!

/**** VIDEO SOBRE COMPONENTES VISUAIS E IMPLEMENTACAO ****/
lista estática - mock
/**** VIDEO SOBRE COMPONENTES VISUAIS E IMPLEMENTACAO ****/

/**** VIDEO SOBRE ATUALIZAR O CONTEUDO ****/
lista dinâmica - api
/**** VIDEO SOBRE ATUALIZAR O CONTEUDO ****/

/**** VIDEO SOBRE ABRIR UM APP ****/
custom url scheme
/**** VIDEO SOBRE ABRIR UM APP ****/



