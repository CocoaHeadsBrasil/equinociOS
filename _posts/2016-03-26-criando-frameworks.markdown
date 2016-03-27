---
layout:     post
title:      "Criando frameworks para várias plataformas"
date:       2016-03-26 00:00:00
author:     "Guilherme Martinez Sampaio"
header-img: "img/gsampaio/brick.jpg"
category:   "frameworks"
---
> Guilherme Sampaio [@gsampaio](https://twitter.com/gsampaio) trabalha desenvolvendo apps para crianças no PlayKids. Gosta de boas práticas de engenharia de software e arquitetura de software.

Durante o último ano tivemos o lançamento de duas novas plataformas de desenvolvimento da Apple: o tvOS e o watchOS. Com isso cada vez mais é importante compartilharmos código entre diversos projetos.

Assim, neste post iremos tratar de como criar e distribuir frameworks para várias plataformas, além de mostrar dicas de como melhorar a qualidade de seus frameworks. Mas antes disso, caso não tenha lido ainda o [post do Igor](http://equinocios.com/library/2016/03/17/bibliotecas/) sobre bibliotecas, esta é uma ótima oportunidade!

## Zen

Para efeitos de demonstração, vamos criar um framework que simplesmente fará wrap da API Zen do Github. Toda vez que fizermos `GET` para esta API iremos receber de volta uma frase inspiradora. Para testar o seu funcionamento basta abrir o seu terminal e digitar `curl -X GET https://api.github.com/zen`.

![]({{ site.baseurl }}/img/gsampaio/zen.png)
<span class="caption text-muted">Exemplo de uso da API</span>

### Criando um framework para iOS

Começamos criando um novo workspace dentro do Xcode. Depois iremos criar dois projetos dentro deste workspace, um para o nosso framework iOS e outro para os projetos de exemplo. 

![]({{ site.baseurl }}/img/gsampaio/workspace.png)
<span class="caption text-muted">Workspace com os dois projetos</span>


Após criar os dois projetos, vamos __linkar__ o nosso framework dentro do nosso target de exemplo. Para isso basta clicar no projeto de exemplo e adicionar na sessão __Linked Frameworks and Libraries__ o nosso framework. Note que como não iremos distribuir a app de exemplo, não teremos problema por ela não estar nos _Embedded Binaries_.

Agora que temos tudo pronto para compilar o Exemplo, vamos começar a implementar o nosso framework. Começamos criando dois arquivos, o primeiro `Zen.swift`, e o segundo `ZenResponse.swift`. 

~~~swift 
//  Zen.swift

import Foundation

/// Type for request zen strings to github
public struct Zen {
    
    /// Public initilizer
    public init() {}
    
    /// Fetches a Zen String from the Github API
    ///
    /// - parameter session: NSURLSession to perform the request.
    /// Fallback to the sharedSession in case it is not provided
    ///
    /// - parameter completion: Closure that receives a ZenResponse with the response of the network request
    public func retrieveZen(session: NSURLSession = NSURLSession.sharedSession(), completion: ZenResponse -> Void) {

        let url = NSURL(string: "https://api.github.com/zen")!
        let request = NSURLRequest(URL: url)
        let dataTask = session.dataTaskWithRequest(request) { data, response, error in
            guard let data = data else {

                NSOperationQueue.mainQueue().addOperationWithBlock {
                    completion(.Failure(error))
                }
                return
            }

            if let zenString = String(data: data, encoding: NSUTF8StringEncoding) {
                NSOperationQueue.mainQueue().addOperationWithBlock {
                    completion(.Success(zen: zenString))
                }
            } else {
                NSOperationQueue.mainQueue().addOperationWithBlock {
                    let error = NSError(domain: "com.gsampaio.zen.error", code: 0, userInfo: nil)
                    completion(.Failure(error))
                }
            }
        }

        dataTask.resume()
    }
}
~~~

Note que por se tratar de um framework, temos que nos preocupar com a visibilidade dos nossos tipos. Swift tem como padrão a visibilidade _internal_, portanto como queremos que `Zen` seja exposto precisamos marcá-lo como _public_, bem como toda função que queremos expor para a aplicação que usará o nosso código. 

Por conta disso, tivemos que criar um `public init()` mesmo que o Swift, por padrão, crie initializers para structs. 

Como a nossa função `retrieveZen` retorna de forma assíncrona, criamos um enum `ZenResponse` que encapsula a lógica de sucesso e falha, como podemos ver no próximo snippet. 

~~~swift 
//  ZenResponse.swift

import Foundation

public enum ZenResponse {
    case Success(zen: String)
    case Failure(NSError?)
}

extension ZenResponse : Equatable {}

public func ==(lhs: ZenResponse, rhs: ZenResponse) -> Bool {
    switch (lhs, rhs) {
    case (.Success(zen: let leftZen), .Success(zen: let rightZen)):
        return leftZen == rightZen
    case (.Failure(let leftError), .Failure(let rightError)):
        return leftError == rightError
    default:
        return false
    }
}
~~~

Este enumerador representa um sucesso com o tipo associado `String`, ou uma falha com um tipo associado `NSError?`. Estendemos o nosso tipo para conformar com Equatable, o que é uma [boa prática para value types em swift](https://developer.apple.com/videos/play/wwdc2015/414/?time=1115)

### Projeto de Exemplo

Com isso acabamos a implementação do nosso framework. Agora vamos implementar no view controller do exemplo uma maneira fácil de mostrar para os usuários do nosso framework como ele funciona. Para fins de demonstração, vamos colocar tudo dentro do próprio view controller. Na vida real considere separar a lógica de apresentação da lógica de network. [O Diogo escreveu um post excelente](http://equinocios.com/arquitetura/2016/03/08/minimizando-acoplamento-view-e-viewcontrollers/) falando como arquitetar um app, recomendo dar uma olhada nele. 


~~~swift
//  ViewController.swift

import UIKit
import Zen

class ViewController: UIViewController {

    let zen = Zen()
    @IBOutlet weak var label: UILabel?

    override func viewDidAppear(animated: Bool) {
        super.viewDidAppear(animated)
        self.label?.text = ""
        zen.retrieveZen { [weak self] zenResponse in
            switch zenResponse {
            case .Success(zen: let zenString):
                print("Zen: \(zenString)")
                self?.label?.text = zenString
            case .Failure(let error):
                print("error: \(error)")
            }
        }
    }
}
~~~

Fazendo algumas alterações no Storyboard conseguimos chegar a um exemplo parecido com a imagem abaixo. 
![]({{site.baseurl}}/img/gsampaio/exemplo-ios.png){:style="display: block; margin-left: auto; margin-right: auto;" }

### Criando um framework para TV

Uma vez que temos o nosso framework para iOS, vamos criar um framework para tvOS. Crie um target _TV Framework_ no projeto Zen. Após criar o framework vai no *Build Settings* e altere o *Product Name* para *Zen*. 

![]({{site.baseurl}}/img/gsampaio/tvos-framework.png){:style="display: block; margin-left: auto; margin-right: auto;" }

Como não estamos usando nenhuma API que não está disponível em ambas as plataformas, podemos adicionar diretamente ao framework tvOS os arquivos `Zen.swift` e `ZenResponse.swift`.

![]({{site.baseurl}}/img/gsampaio/target-membership.png){:style="display: block; margin-left: auto; margin-right: auto;" }

Caso fosse necessário no futuro diversificar o que esta disponível para cada plataforma, poderíamos usar as marcações de OS ou availability como mostro abaixo. 

~~~swift

#if os(tvOS)

// Código que será disponível apenas para tvOS

#endif

if #available(tvOS 9.1, *) {
    // Código disponível apenas para tvOS 9.1 para frente
}

~~~

Pronto, com isso temos o nosso framework tvOS. Agora podemos criar outro projeto de exemplo para tvOS e colocar o mesmo código do ViewController de exemplo do iOS, e chegamos ao resultado abaixo. 

![]({{site.baseurl}}/img/gsampaio/exemplo-tvos.png){:style="display: block; margin-left: auto; margin-right: auto;" }

## Distribuindo seu framework

Agora que temos os nossos frameworks prontos, vamos pensar em como distribuí-los. Para isso daremos suporte tanto ao Carthage quanto ao CocoaPods, que são os dois gerenciadores de dependência mais utilizados pela comunidade. 

### Carthage

O [Carthage](https://github.com/Carthage/Carthage) é um gerenciador de dependência para iOS/OSX/watchOS/tvOS que não tem um repositório central de dependências. Para utilizá-lo você precisa criar um `Cartfile` no seu projeto e adicionar as referências ao repositórios que deseja utilizar. Ao rodar `carthage update` no seu projeto, o Carthage irá fazer checkout de todas as dependências e procurar por shared build schemes dentro dos projetos das dependências, buildando um a um. 

Para adicionarmos o suporte ao Carthage, a única alteração que vamos ter que fazer no nosso workspace é fazer com que nossos frameworks estejam com seu build scheme marcados com _Shared_. Para isso vá em _Product_ > _Scheme_ > _Manage Schemes_ e marque ambos os frameworks como shared. 

![]({{site.baseurl}}/img/gsampaio/shared-schemes.png){:style="display: block; margin-left: auto; margin-right: auto;" }

Pronto. Para testar basta criar um novo projeto do Xcode, criar um `Cartfile`, adicionar a referência para o repositório que contém o framework e rodar `carthage update`. Depois disso, teremos uma nova pasta Carthage no seu diretório com duas pastas: Build e Checkouts. Na pasta Build você encontrará ambos os framework buildados. 

### CocoaPods

CocoaPods é o gerenciador de dependências mais famoso para desenvolvimento para as plataformas da Apple. Diferente do Carthage, o CocoaPods contém um repositório central de `pods`, fazendo a integração automática e criando um workspace onde todas as dependências são compiladas junto com o seu projeto. 

Para permitir o uso do CocoaPods precisamos criar um `podspec`, que indica quais arquivos serão usados para buildar os frameworks: 

~~~ruby
Pod::Spec.new do |s|
  s.name         = "Zen"
  s.version      = "0.0.1"
  s.summary      = "Swift implementation of the Zen API from Github"
  s.description  = <<-DESC
    Zen provides an easy way to access the Zen API provided by folks at Github
  DESC

  s.homepage = "http://github.com/gsampaio/Zen"
  s.license = "MIT"
  s.author = { "Guilherme Sampaio" => "guilhermesampaio@gmail.com" }
  s.social_media_url = "http://twitter.com/gsampaio"
  
  s.ios.deployment_target = "8.0"
  s.tvos.deployment_target = "9.0"

  s.source       = { :git => "https://github.com/gsampaio/Zen.git", :tag => "0.0.1" }
  s.source_files  = "Zen/Zen/*.{swift}"

  s.requires_arc = true
end

~~~

Agora, basta criar um novo projeto e um `Podfile`, adicionando a linha `pod 'Zen', :path=> "path do nosso podspec"`.

## Utilitários

Agora que sabemos como criamos e distribuímos frameworks, iremos plugar alguma ferramentas que irão ajudar na qualidade. Nesta sessão vamos abordar __Testes Unitários e Cobertura__, __Documentação__ e __Integração Contínua__. 

### Testes unitários e cobertura de código. 

Para termos certeza - ou ao menos segurança - que o nosso código funciona, vamos adicionar casos de testes aos nossos frameworks e depois vamos gerar relatórios de cobertura para entender melhor quais partes do nosso código estamos testando. 

#### Testes Unitários

Vamos começar implementando testes no arquivo `ZenTests.swift`, dentro do target `ZenTests`. Para executar os testes, vamos criar uma fake classe de `NSURLSession` e `NSURLSessionDataTask` para mockar seu funcionamento e injetarmos o resultado que queremos.

~~~swift
//  ZenTests.swift

import XCTest
@testable import Zen

class ZenTests: XCTestCase {
    func testSuccess() {
        let string = "Practicality beats purity." as NSString
        let data = string.dataUsingEncoding(NSUTF8StringEncoding)!
        
        let zen = Zen()
        zen.retrieveZen(FakeURLSession(data: data)) { response in
            switch response {
            case .Success(zen: let zenString):
                XCTAssertEqual(zenString, string)
            default:
                XCTFail()
            }
        }
    }
}

private final class FakeURLSession : NSURLSession {
    let data: NSData
    
    init(data: NSData) {
        self.data = data
    }
    
    private override func dataTaskWithRequest(request: NSURLRequest, completionHandler: (NSData?, NSURLResponse?, NSError?) -> Void) -> NSURLSessionDataTask {
        let task = FakeURLSessionDataTask(data: data, completionHandler: completionHandler)
        return task
    }
}

private final class FakeURLSessionDataTask : NSURLSessionDataTask {
    
    let data: NSData?
    let completion: (NSData?, NSURLResponse?, NSError?) -> Void
    init(data: NSData, completionHandler: (NSData?, NSURLResponse?, NSError?) -> Void) {
        self.data = data
        self.completion = completionHandler
        super.init()
    }
    
    private override func resume() {
        completion(data, nil, nil)
    }
}

~~~

Não vou entrar em mais detalhes sobre a implementação de testes de network, mas caso você tenha mais interesse, recomendo dar uma olhada no [DVR](http://github.com/venmo/DVR) e no [OHHTTPStubs](https://github.com/AliSoftware/OHHTTPStubs).

#### Cobertura de testes

Uma das mudanças do Xcode 7 foi a integração de cobertura de testes dentro da IDE. Para isso, precisamos editar o scheme Zen, marcar `Gather coverage data` na área de `Test` do scheme Zen, como ilustra a imagem abaixo:

![]({{site.baseurl}}/img/gsampaio/cobertura-xcode.png){:style="display: block; margin-left: auto; margin-right: auto;" }

Rodando nossos testes unitários, agora é possível olhar a cobertura dentro do próprio Xcode.

![]({{site.baseurl}}/img/gsampaio/xcode-cobertura.png){:style="display: block; margin-left: auto; margin-right: auto;" }


#### Slather

[Slather](https://github.com/SlatherOrg/slather) é uma ferramenta para gerar relatórios de cobertura de código. Com ele, é possível gerar relatórios HTML para visualizar cobertura de testes, assim como relatórios XML para uso em ambientes de integração contínua. 

Para fins de exemplo, iremos gerar um HTML com o relatório de cobertura. Para isso, basta instalar o Slather e rodar o comando abaixo:

`slather coverage --html --show Zen/Zen.xcodeproj`

![]({{site.baseurl}}/img/gsampaio/slather.png){:style="display: block; margin-left: auto; margin-right: auto;" }

Caso exista interesse, é possível plugar o slather dentro do seu ambiente de integração continua e enviar relatórios de cobertura para serviços como o [Coveralls](https://coveralls.io), e assim acompanhar a evolução da cobertura do seu projeto. Um exemplo é o [VENCore](https://github.com/venmo/VENCore), que tem [seus relatórios de cobertura](https://coveralls.io/github/venmo/VENCore) publicados.

### Documentação 

Uma vez que temos testes unitários e relatórios de cobertura, queremos agora garantir que conseguimos gerar a documentação para que seja mais fácil utilizar o nosso framework.

Caso você não seja familiar com o formato de documentação do Swift, recomendo dar uma olhada [neste post da Erika Sadun](http://ericasadun.com/2015/06/14/swift-header-documentation-in-xcode-7/). Talvez você tenha percebido que o arquivo `Zen.swift` já esta documentado com Swift Header Documentation.

Outro ponto importante de documentação é escrever um `README.md` bem explicativo para que as pessoas que encontrarem o seu framework saibam exatamente o que ele faz e todas as suas capacidades. O [Alamofire](https://github.com/Alamofire/Alamofire) tem um excelente `README.md` para se inspirar. 

#### Jazzy ♪♫

Para gerar documentação HTML e ser compatível com o [CocoaDocs](http://cocoadocs.org) vamos utilizar o [Jazzy](https://github.com/realm/jazzy). Uma vez documentado o seu framework, basta instalarmos o Jazzy e rodarmos o comando abaixo para gerar o HTML: 

`jazzy -x -scheme,Zen` 

![]({{site.baseurl}}/img/gsampaio/jazzy.png){:style="display: block; margin-left: auto; margin-right: auto;" }

### Integração contínua

Por fim, agora que temos testes e documentação, queremos ter um processo de integração contínua para nos ajudar em tasks relacionadas ao nosso framework. Para isso recomendo dar uma olhada no [Travis CI](https://travis-ci.org), no [Circle CI](https://circleci.com) ou no [Jenkins](https://jenkins-ci.org) para executar as suas tasks. 

#### Fastlane

Nesta última semana tivemos [um excelente post sobre o fastlane](http://equinocios.com/ios/2016/03/24/fastlane/) feito pelo [Fábio](http://www.twitter.com/fabintk). Assim, não entrarei muito em detalhes, apenas listarei algumas actions que podem ajudar no test/deploy do seu framework. 

| Action               | Descrição                                                                                   |
|----------------------|---------------------------------------------------------------------------------------------|
| `set_github_release` | Cria uma release nova no github. É útil para distribuir frameworks compilados pelo carthage |
| `pod_push`           | Faz deploy do seu podspec para o trunk ou para um repositório privado do cocoapods          |
| `slather`            | Executa o slather para gerar coverage reports                                               |
| `jazzy`              | Executa o jazzy para gerar documentação                                                     |
| `scan`               | Roda os unit tests da sua framework                                                         |

## Considerações finais

Gosto muito do conceito de criar frameworks pequenos e ir expandindo a sua aplicação em cima deles. Recomendo assistir a talk sobre [Library Oriented Programming](https://realm.io/news/justin-spahr-summers-library-oriented-programming/) do Justin Spahr Summers, que conta sobre desenvolver frameworks. 

Por fim, muitas das práticas que aplicamos nestes frameworks são aplicáveis ao seus aplicativos. Testes/Cobertura e Integração contínua são excelentes exemplos.

Se tiverem alguma dúvida me procurem no [twitter](https://twitter.com/gsampaio) ou no slack do [iOSDevBR](http://iosdevbr.herokuapp.com/).

Espero que tenham gostado!


