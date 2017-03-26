---
layout:     post
title:      "Integração Contínua com Travis-CI"
subtitle:   "Chegou a hora de ter menos dores de cabeça durante o desenvolvimento do seu app."
date:       2017-03-26 12:00:00
author:     "Fabricio Serralvo"
header-img: "img/serralvo/cover.png"
category:   ios
---

# Integração Contínua com Travis CI
Integração Contínua é, em outras palavras, a prática que busca manter o ambiente de desenvolvimento em pleno funcionamento após iterações feitas pelos devs do projeto. Para que isso ocorra, é necessário que a cada alteração no repositório alguns itens do projeto sejam verificados, como a geração da _build_ e execução de testes, por exemplo.

Você pode ler um pouco mais sobre o assunto [aqui](https://martinfowler.com/articles/continuousIntegration.html).

# Travis CI

O Travis CI é um serviço dedicado exclusivamente à integração contínua, a principal limitação  do mesmo é a necessidade do projeto estar hospedado no GitHub. 

O uso do Travis para projetos _open source_ é gratuito 🎉 (isso sem dúvida ajudou a popularizá-lo). Para utilizá-lo em projetos privados você terá que desembolsar alguns dólares, o valor mensal (informações de março, 2017) vai de $69 no plano _Bootstrap_ até $489 💸 na opção _Premium_.

Para começar a utilizar basta acessar [travis-ci.org](https://travis-ci.org) para projetos open source ou [travis-ci.com](https://travis-ci.com) para projetos privados. O processo de _sign-up_ é bastante simples.

# Benefícios

Entre os benefícios do uso do Travis (ou qualquer outra opção para integração contínua) destaco a velocidade em que os problemas corriqueiros são encontrados, afinal, quanto mais rápido descobrimos alguma adversidade, menor a chance de uma dor de cabeça das grandes. 

Outro ponto interessante é a integração realizada pelo Travis com o GitHub, onde a cada _commit_ ou _pull request_, o Travis executa seu _job_ e marca visualmente se o ambiente foi comprometido ou não. Além também da geração dos tradicionais _badges_ informando o _status_ da projeto.

![]({{ site.baseurl }}/img/serralvo/commits-travis.png)

_Marcação visual do estado da build_

# Hora do Show

Para exemplificar o funcionamento do Travis criei um projeto para iOS, escrito em Swift 3, nele adicionei uma dependência para demonstrar o processo para obter tal.

Esse projeto gerou três itens para explorarmos: 

* Será necessário gerenciar as dependências do projeto —  sim, é isso que você está pensando: Cocoapods.
* Execução da _build_.
* Por fim, notificaremos o time sobre o estado do projeto.

# Arquivo .travis.yml

Para que essas tarefas listadas acima funcionem, nós precisaremos criar um arquivo que fornecerá instruções para o Travis. Aí é que entra o `.travis.yml`.

Antes de começarmos a escrever o arquivo de configuração, precisamos entender como funciona o ciclo de vida do processo de _build_ do Travis:

O Travis divide a execução do _job_ em duas etapas, a primeira delas é responsável pela atualização das dependências, seja do projeto ou do ambiente. A segunda etapa é voltada para a execução da _build_ e outras tarefas relacionadas, como _tests_, _reports_ e _deploy_. Nessas etapas podemos detectar eventos do ciclo de vida e incluir ações de acordo com a necessidade do projeto.

Para começarmos, o primeiro passo é realizar o cadastro no Travis, para repositórios públicos ou privados, o funcionamento será o mesmo.

![]({{ site.baseurl }}/img/serralvo/travis-ci-org.png)

O próximo passo é "ativar" o Travis para o repositório desejado, feito isso, no próximo _commit_ ou _pull request_ o _job_ será executado.

![]({{ site.baseurl }}/img/serralvo/start-travis.png)

Após o cadastro chegou a hora de criar o arquivo *.travis.yml* na raiz do repositório, feito isso, vamos incluir duas linhas: A primeira linha é usada para definir a linguagem do projeto (que é um valor obrigatório), no caso adicionaremos `objective-c`. A segunda representa a imagem que será usada para fazer a build do projeto, no caso, adicionamos `xcode8.1`. Veja abaixo:

~~~
language: objective-c
osx_image: xcode8.1
~~~

Agora você pode estar assim: 🤔

Ué, `objective-c`?! Mas o projeto não está escrito em Swift? Sim, isso mesmo, até o momento o Travis utiliza o valor `objective-c` para Swift e também para Objective-C.

## Baixando as dependências

Se você já usou CocoaPods ao menos uma vez, sabe que basta executar `pod install` para obter as dependências do projeto, no ambiente do Travis isso é um pouco diferente, se o `Podfile` estiver na raiz do projeto, o Travis fará o _fetch_ automaticamente, caso contrário, basta indicar no `.travis.yml` qual o local de tal arquivo:

~~~
podfile: path/to/Podfile
~~~

Se o seu projeto usa outro gerenciador de dependências, é possível fazer _override_ da etapa de `install` para realizar a instalação de acordo com sua necessidade:

~~~
install: sh dependencies.sh
~~~

## Build e Sucesso

Após obter as dependências do projeto, basta apenas configurar alguns parâmetros para a execução da _build_. Esses parâmetros envolvem alguns itens necessários para continuar o processo, são eles:

* _Path_ do `xcworkspace` ou `xcproject`.
* Algum _scheme_ com a opção _shared_ ativada ([saiba mais aqui](http://help.apple.com/xcode/mac/8.0/#/dev5426ddfcf)).
* SDK que será usado (no caso estamos usando `iphonesimulator`).

Conhecendo tais parâmetros é a hora de realizar a _build_ do projeto, para isso, vamos adicionar na etapa `script` a chamada do `xcodebuild`.

~~~
script:
  - xcodebuild -workspace DemoTravisCI.xcworkspace -scheme 'DemoTravisCI' -sdk iphonesimulator build
~~~

Sendo assim, no momento o arquivo `.travis.yml` possui o seguinte conteúdo:

~~~
language: objective-c
osx_image: xcode8.1

script:
  - xcodebuild -workspace DemoTravisCI.xcworkspace  -scheme DemoTravisCI -sdk iphonesimulator build
~~~

## Notificações

Para detectar possíveis problemas com o projeto, é interessante compartilhar o status da _build_ com o time a cada iteração. O Travis prove diversas opções para enviar notificações, você pode optar por receber tais alertas via e-mail e/ou utilizar alguma integração, no nosso caso, vamos ser avisados via Slack 📢.

Para isso, o primeiro passo é adicionar uma [nova integração ao Slack](https://my.slack.com/services/new/travis). Após selecionar o time e as demais opções, na própria página do Slack você encontrará o conteúdo que deve ser adicionado ao seu arquivo de configuração.

Concluido tal passo, vamos incluir a chave `notifications` no arquivo `.travis.yml`:

~~~
notifications:
  slack: yourteam:G1P621hDDwEH3pXeCcJpck8i
~~~

Importante: Caso seu projeto seja aberto, é recomendado criptografar a chave. Esse é um processo bastante simples e você pode saber mais [aqui](https://docs.travis-ci.com/user/encryption-keys/).


# Considerações Finais

Ferramentas de integração contínua são realidade no mercado e fazem parte da rotina de qualquer grande projeto de software. Algumas demandam maior esforço para que sejam configuradas (principalmente ferramentas _self-hosted_) outras não, como é o caso do Travis, onde basta um arquivo de marcação para que a mágica aconteça.

Se você se interessou pelo assunto e deseja implantar tal processo em seus projetos, e, por alguma limitação não irá conseguir utilizar o Travis, saiba que existem diversas opções, como Jenkins, Xcode Server, CircleCI, entre outras.

Para finalizar, Integração Contínua é um dos pilares do desenvolvimento ágil, e podemos considerar que também é um passo para a Entrega Contínua, mas isso é assunto para outro artigo 🙃.

### Referencias:

* [Imagem da Capa - Relax by Clem Onojeghuo](https://unsplash.com/photos/zlABb6Gke24)
* [Getting started](https://docs.travis-ci.com/user/getting-started/)
* [Building an Objective-C Project](https://docs.travis-ci.com/user/languages/objective-c/)
* [Notifications](https://docs.travis-ci.com/user/notifications/)

