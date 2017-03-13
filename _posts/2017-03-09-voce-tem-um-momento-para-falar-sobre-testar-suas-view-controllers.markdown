---
layout:     post
title:      "Você tem um momento para falar sobre testar suas View Controllers?"
subtitle:   "Um resumo de como comecei a escrever Snapshot Tests"
date:       2017-03-09 12:00:00
author:     “Christian Sampaio“
header-img: "img/chrisfsampaio/mona-lisa-snapshot-comparison.png"
category:   testes
---

## O impasse

Nós, conhecidos como desenvolvedores _front-end_, muitas vezes nos encontramos com o impasse dos testes unitários. As perguntas são sempre as mesmas. O que testar? Como escrever código que seja fácil de testar? Como praticar _TDD_? Vale a pena escrever e manter testes?   
Alguns vão mais longe na discussão e buscam indicadores que garantirão a qualidade e robustez de um projeto, como cobertura de teste mínima.

Acho muito relevante a empolgação sobre o assunto, desde que não esqueçamos do objetivo dos testes unitários: validar o funcionamento de um sistema. Todas as outras consequências não são nada mais do que apenas consequências. Por isso, devemos ser muito cautelosos ao afirmar que seguir todos os princípios _SOLID_, ou qualquer outro regulamento, [significa qualidade garantida](https://twitter.com/jduv2683/status/834608248569417728). Novamente, é muito válida a intenção de encontrar teorias para que possamos oficializar metodologias eficientes para o nosso ofício. Entretanto, é essencial validarmos que essas regras vão realmente atingir o objetivo almejado. Caso a resposta for negativa, é nosso papel encontrar uma solução equivalente.

E é na nossa área, desenvolvimento _front-end_, que _Snapshot Tests_ se apresenta como uma solução justa para um problema impertinente – testar código de _UI_. É fato que grande parte de um projeto _front-end_ é código dedicado à parte visual do programa, como o próprio nome sugere. Ao passo que a maioria dos guias de como seguir _TDD_ não incluem uma linha desse tipo de código. O que faz sentido, porque não desejo a ninguém escrever testes para verificar se todos os elementos visuais, de uma determinada interface, estejam nas posições, cores, proporções, fontes, ou opacidade corretas.

## Um caso de teste

Abandonando a discussão filosófica (ufa!), tomemos como exemplo um aplicativo iOS. Que, por sua vez, implica o uso de `UIViewController`s na maioria dos casos – uma área negligenciada em termos de testes, por motivos de praticidade geralmente.

Aqui cabe a leitura de dois textos do caro [Diogo](https://twitter.com/diogot), doutor [^1]. [O primeiro](http://invariante.com/2015/10/20/todo-view-controller-deveria-ter-delegate/) elabora sobre delegar as ações originadas de uma `UIViewController`. [O segundo](http://invariante.com/2016/04/18/minimizando-acoplamento-view-e-viewcontrollers/) sugere o uso de _view models_, que, em suma, fornecem somente as informações estritamente necessárias para que `UIViewController`s configurem suas _views_.  Tendo essas ideias estabelecidas, `UIViewController`s tendem a ficar simples e fáceis de serem inicializadas em diferentes contextos isolados, como em um caso de teste. Meio caminho andado. Se o nosso objetivo é testar, precisamos verificar se o resultado está de acordo com o esperado. É nessa tarefa que _Snapshot Tests_ facilita nossa vida.

Quando li sobre a ideia pela primeira vez, achei criativa. Mas descartei involuntariamente porque assumi que seria difícil de configurar, manter ou funcionar na prática. Eu sei, atitude errada. Mas você já deve ter notado que mudei de opinião em algum momento. Pois bem, numa bela manhã, meu ex-colega - porém ainda amigo - [Lars](https://twitter.com/larslockefeer), submeteu uma _Pull Request_ com umas _screenshots_. A narrativa naquele _diff_ me convenceu por si só. Veja um exemplo análogo, retirado do [projeto open source Eigen, da Artsy](https://github.com/artsy/eigen/pull/2169/files).

![]({{ site.baseurl }}/img/chrisfsampaio/snapshots-pr-example.png)

De maneira consolidada, podemos analisar e revisar as mudanças feitas em termos de código e quais são as consequências visuais dessas mudanças. O que além de facilitar muito o processo de revisão, é também muito conveniente para documentação e referência futura.

Mas não nos desviemos do objetivo: validar o funcionamento. E para entender como _Snapshot Tests_ funcionam, vamos usar o _framework_ [FBSnapshotTestCase](https://github.com/facebook/ios-snapshot-test-case), do Facebook. Depois de [integrar o _framework_](https://github.com/facebook/ios-snapshot-test-case#installation-with-cocoapods) ao seu projeto, o processo para escrever testes é composto pelas seguintes partes:

1. Começamos escrevendo um caso de teste como qualquer outro, exceto pela parte que herdaremos de `FBSnapshotTestCase` ao invés de `XCTestCase`.

2.  Escrevemos uma função para testar um estado da `UIViewController` sob teste.

3.  Atribuímos o valor `true` à propriedade `recordMode` do caso de teste. Esse é o passo que irá fazer com que imagens de referência sejam geradas.

4. Utilizamos a função `FBSnapshotVerifyView`, passando como parâmetro a _view_ associada à `UIViewController` em questão. Essa função irá capturar o resultado visual da _view_ recebida, e uma imagem, usada como referência, é salva em disco. O caminho do arquivo é escrito no _console_ para fácil acesso.

5. Verificamos se o arquivo de referência corresponde ao resultado esperado. Em caso negativo, ajustamos o código sob teste e repetimos o passo anterior até obtermos uma imagem satisfatória.

6. Atribuímos o valor `false` à propriedade `recordMode` do caso de teste. Este último passo usará as chamadas à `FBSnapshotVerifyView` para comparar, ponto a ponto, o resultado visual da _view_ recebida com a imagem de referência já existente. O teste somente irá passar se a comparação não vir a falhar.

![]({{ site.baseurl }}/img/chrisfsampaio/butterfly-old-photo.jpg)

Um teste simples ficaria semelhante às linhas do código a seguir.

~~~swift
import XCTest
import FBSnapshotTestCase
@testable import MyHumbleApp

class LoginViewControllerTests: FBSnapshotTestCase {

    override func setUp() {
        super.setUp()
        recordMode = true // Mudar para false quando o teste gerar um resultado satisfatório.
    }

    func testEmptyState() {
        let viewController = LoginViewController()
        viewController.frame.size = UIScreen.main.bounds.size
        FBSnapshotVerifyView(viewController.view)
    }
}
~~~

Para evitar a repetição de alguns caracteres, costumo usar uma classe base que encapsula as particularidades dos frameworks envolvidos:

~~~swift
import XCTest
import FBSnapshotTestCase

class BaseSnapshotTests: FBSnapshotTestCase {

    func shouldRecord() -> Bool {
        fatalError("Should be overriden")
    }

    override func setUp() {
        super.setUp()
        self.isDeviceAgnostic = false
        self.recordMode = shouldRecord()
    }

    func validateView(of viewController: UIViewController, windowLevel: UIWindowLevel = UIWindowLevelStatusBar, containerSize: CGSize = UIScreen.main.bounds.size, file: StaticString = #file, line: UInt = #line) {
        let frame = CGRect(origin: .zero, size: containerSize)
        let window = UIWindow(frame: frame)
        window.backgroundColor = .clear
        window.windowLevel = windowLevel
        window.rootViewController = viewController
        window.isHidden = false
        viewController.view.frame = frame
        viewController.view.layoutIfNeeded()
        FBSnapshotVerifyView(viewController.view, file: file, line: line)
    }

    func validate(view: UIView, withSize size: CGSize, file: StaticString = #file, line: UInt = #line) {
        view.frame = CGRect(origin: CGPoint.zero, size: size)
        view.layoutIfNeeded()
        FBSnapshotVerifyView(view, file: file, line: line)
    }
}
~~~

Simplificando, assim, o nosso caso de teste de exemplo:

~~~swift
@testable import MyHumbleApp

class LoginViewControllerTests: FBSnapshotTestCase {

    func shouldRecord() -> Bool {
        return true // Mudar para `false` quando o teste gerar um resultado satisfatório.
    }

    func testEmptyState() {
        let viewController = LoginViewController()
        validateView(of: viewController)
    }
}
~~~

## Em síntese

É importante ressaltar que, ao escrever este tipo de teste – ou qualquer outro teste unitário -, é necessário mantê-los. Isto é, ajustá-los conforme os requisitos mudarem, ou excluí-los se os requisitos não existirem mais. É também necessário que estes testes sejam executados com certa frequência, utilizando integração contínua de preferência. _Snapshot Tests_ são rápidos e podem ser executados juntamente com os outros testes unitários existentes no projeto.

Naturalmente, os testes também incluirão o código necessário para configurar as `UIViewController`s. Esse código foi omitido no exemplo por motivo de clareza. Mas devemos nos atentar em minimizar e isolar esse código o máximo possível, utilizando _view models_  ou não. Se um teste unitário tem muitas dependências, há uma grande chance dele não ser mantido apropriadamente ou facilmente mal interpretado, por outros ou pelo próprio autor, no futuro. 

Uma outra consequência, é a mudança no processo utilizado para escrever código de _UI_. Para construir a última tela de um fluxo, por exemplo, muitas vezes são necessários vários _taps_, _swipes_ e _bytes_ trafegados na internet. Consumindo muito tempo acumulado pelas infinitas iterações para atingir aquele resultado _pixel perfect_, que deixa seu amigo _designer_ de olhos cheios. Este tempo não é necessário quando se escreve um _Snapshot Test_ para sua _UIViewController_. Você pode até tentar convencer seu amigo _designer_ a participar do processo de code review, verificando se as imagens de referência estão de acordo, e mitigando assim, a necessidade daquele ticket recorrente e entitulado “Design Review”.

## Epílogo

De modo geral, como foi sugerido, tenho experiências positivas com _Snapshot Testing_. Veja, não estou dizendo que foi a solução de todos os meus problemas, mas sim uma boa ferramenta que ajudou manter minhas `UIViewController`s mais estáveis. 

Agradeço pela leitura e espero que essa ferramenta possa te ajudar também. 

Caso queira entrar em contato, estou no [Twitter (@chrisfsampaio)](https://twitter.com/chrisfsampaio) e no Slack do [iOSDevBR](http://iosdevbr.herokuapp.com/) (@christian). 

---- 

## Referências 

* Novamente agradeço ao estimado [Lars](https://twitter.com/larslockefeer). Que depois de ter me apresentado o assunto, [compartilhou sua experiência](https://twitter.com/larslockefeer/status/753496126393880576) numa [apresentação](https://twitter.com/larslockefeer/status/753495593268510720) do [CocoaHeads dos Países Baixos](http://cocoaheads.nl/).  
* Vale também mencionar [um relato](https://www.objc.io/issues/15-testing/snapshot-testing), do [Orta Therox](http://orta.io), deveras parecido com a experiência que tive.

---- 


### Errata
1. Meu amigo [Fabri](https://twitter.com/marcelofabri_) fez uma boa ressalva – executar o teste antes de gerar a primeira imagem e observamos que o mesmo irá falhar. Este passo é importante para validarmos que o teste falha quando não há arquivo para ser usado como referência.

2.  O caro [Fabri](https://twitter.com/marcelofabri_) também lembrou que vale mencionar o [_matcher_ para `Expecta`](https://github.com/dblock/ios-snapshot-test-case-expecta), que permite uma sintáxe mais natural nas linhas de:  

~~~swift  
    expect(view).to.recordSnapshot()
    expect(view).to.haveValidSnapshot()
    expect(view).to.haveValidSnapshotWithTolerance(0.01)
    expect(view).to.haveValidSnapshotNamedWithTolerance(@"unique snapshot name", 0.01)
~~~  

<br/>

#### Notas
[^1]: Na verdade, recomendo a leitura de todos os textos que estão lá no [invariante](http://invariante.com/).
