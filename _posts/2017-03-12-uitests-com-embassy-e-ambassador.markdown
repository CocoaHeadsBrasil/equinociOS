---
layout:     post
title:      "UITests com Embassy e Ambassador"
subtitle:   "Mockando a sua API no target de testes"
date:       2017-03-12 00:00:00
author:     "Emannuel Carvalho"
header-img: "img/emannuelOC/header.jpg"
category:   Testes
---

# UITest com Embassy & Ambassador

## Objetivo

Esse post tem objetivos bem modestos. A ideia é simplesmente falar sobre um desafio que eu enfrentei em um projeto recente (testar a UI de maneira isolada e independente) e apresentar as ferramentas que eu encontrei e que foram muito úteis ([Embassy](https://github.com/envoy/Embassy){:target="_blank"} e [Ambassador](https://github.com/envoy/Ambassador){:target="_blank"}).

Foi uma experiência legal e até meio surpreendente de certa forma. Espero que isso possa ser útil pra outras pessoas também 😊

## Motivação

Algo que sempre me chamou a atenção é a interação entre pessoas de diferentes áreas e com diferentes backgrounds. Eu costumo me agradar muito com as iniciativas que têm como objetivo tornar essa interação mais agradável e produtiva. Foi esse pensamento que me motivou a implementar testes de UI no projeto que eu estava trabalhando nos últimos meses. 

Diferente dos testes unitários, os testes de UI tem essa característica de observar o funcionamento do app "pelo lado de fora" e isso pode ser uma porta de entrada para "não-programadores" se envolverem mais com o processo de desenvolvimento, uma vez que não é necessário conhecer a implementação do app pra testar. Além disso, o código em um teste de UI costuma ser bem mais legível (menos assustador? 🤔) pra quem não está acostumado a codar. 

Nesse projeto eu resolvi escrever os meus primeiros UITests. E eu precisei pesquisar como eu faria pra que os meus testes não tocassem no servidor. Naquele momento nós só tínhamos o servidor de homologação então de cara havia duas opções: (1) criar um servidor para os testes e passar uma url diferente como variável de ambiente no launch; ou (2) mockar as respostas do servidor na própria camada de network e retornar essas respostas quando estiver rodando os testes de UI.

A primeira opção não me pareceu uma boa ideia. Os testes dependeriam da conexão e do servidor, de modo que qualquer falha em um desses dois acarretaria em uma alteração no resultado dos testes.  
A segunda opção não me parecia tão problemática, mas ainda não era o ideal. O que me fez pensar duas vezes foi o fato de que haveria bastante código no app que não servia pro app em si, mas sim pros meus testes. 

Eu fui procurar uma terceira opção e acabei encontrando [este artigo](https://envoy.engineering/embedded-web-server-for-ios-ui-testing-8ff3cef513df#.t2rr3w9db){:target="_blank"}. Com o Embassy e o Ambassador, a ideia é que o seu servidor rode no próprio target de testes e você pode definir a resposta que você quer retornar no próprio método onde a UI será testada. Além disso, é possível verificar os dados que o app está enviando para o servidor a partir de ações na interface.

Eu gostei muito da ideia. Os testes vão ficar isolados e a definição do que o app está recebendo fica logo acima dos asserts que verificam se a interface está como deveria, tornando bem mais visível (mesmo pra quem não está acostumado com o código) a relação entre o que o app está esperando de retorno da API e a interface.

## Configurando o projeto

Para que o nosso app tenha acesso ao servidor local, é preciso fazer algumas configurações. Em primeiro lugar, você deve incluir o `Embassy` e o `Ambassador` no projeto. Eu fiz isso usando cocoapods:

```
target 'JustDevsUITests' do
    inherit! :search_paths

    pod 'Embassy', '~> 3.0'
    pod 'EnvoyAmbassador', '~> 3.0'
end
```

Para as configurações do servidor, eu criei uma _UITestBase_ com as configurações necessárias: 

```swift
import XCTest

import Embassy
import EnvoyAmbassador

class UITestBase: XCTestCase {
    
    let port = 8088
    var router: Router!
    var eventLoop: SelectorEventLoop!
    var server: HTTPServer!
    var app: XCUIApplication!
    
    var eventCondition: NSCondition!
    var eventLoopThread: Thread!
        
    override func setUp() {
        super.setUp()
        setupWebApp()
        setupApp()
    }
    
    private func setupWebApp() {
        eventLoop = try! SelectorEventLoop(selector: try! KqueueSelector())
        router = Router()
        server = DefaultHTTPServer(eventLoop: eventLoop, port: port, app: router.app)
        
        try! server.start()
        
        eventCondition = NSCondition()
        eventLoopThread = Thread(target: self, selector: #selector(UITestBase.runEventLoop), object: nil)
        eventLoopThread.start()
    }
    
    private func setupApp() {
        app = XCUIApplication()
        app.launchEnvironment["webserviceURL"] = "http://[::1]:8088"
    }
    
    override func tearDown() {
        super.tearDown()
        app.terminate()
        server.stopAndWait()
        eventLoop.stop()
    }
    
    @objc func runEventLoop() {
        eventLoop.runForever()
        eventCondition.lock()
        eventCondition.signal()
        eventCondition.unlock()
    }
    
}

```

No método `setupWebApp()` eu inicializo o server e crio o `router`, que será usado para definir as respostas do servidor para cada request.

No `setupApp()` eu crio o `XCUIApplication` e passo a url para o app. No código do app, a única mudança que eu preciso fazer é uma verificação se essa url foi passada, se tiver sido, ela se torna a _base url_ da API.

```swift
let baseURL = ProcessInfo.processInfo.environment["webserviceURL"] ?? "http://realurlgoeshere.com"
```  

Ok! Servidor configurado! Vamos ver os testes em ação.

## App de exemplo

O app de exemplo é bem simples. É uma nano-rede social para desenvolvedores iOS, onde você pode se cadastrar e publicar uma única frase ou citação que diz algo sobre você. Além disso você pode ver os usuários cadastrados e as frases que eles publicaram. Ele apresenta uma lista de Devs e ao selecionar um deles, o usuário vai pra uma tela de detalhe que mostra a frase publicada.

<img src="{{ site.baseurl }}/img/emannuelOC/list_devs.png">

A primeira tela é uma tableView em que cada célula mostra no `textLabel` o nome do dev naquela posição.

A segunda tela contém somente uma label com a frase referente ao dev selecionado e o seu nome no title da `navigationBar`.

<img src="{{ site.baseurl }}/img/emannuelOC/detail_screen.png">

## Testando a UI

No nosso teste, quando o app abrir vai ser feita uma chamada para o servidor que deve retornar a lista de desenvolvedores cadastrados. 

O retorno com a lista para o nosso "micro-exemplo" ficou mais ou menos assim:

```swift
let mockData = [
    [
        "id": 123,
        "name": "Emannuel Carvalho",
        "quote": "O equinociOS está demais! 🚀"
    ],
    [
        "id": 214,
        "name": "Francesco Perrotti-Garcia",
        "quote": "Vc não quer um pato 🐥, vc quer um Quackable!"
    ],
    [
        "id": 235,
        "name": "Kaique D'amato",
        "quote": "Computação molecular não é complicado!!!"
    ],
    [
        "id": 236,
        "name": "Fernando Bunn",
        "quote": "Parei de ler o artigo quando vi \"Cocoapods\"."
    ],
    [
        "id": 237,
        "name": "Rafael Feroli",
        "quote": "O que eu to fazendo aqui?! Eu sou designer!"
    ],
    [
        "id": 238,
        "name": "Steph Curry",
        "quote": "🏀 Lock in #DubNation"
    ],
    [
        "id": 241,
        "name": "Ezequiel França",
        "quote": "Não é stalking, é uma espécie de pesquisa sociológica dos devs do séc. XXI"
    ],
    [
        "id": 242,
        "name": "Guilherme Rambo",
        "quote": "Curti! Vou fazer uma versão pro macOS... [3 minutes later] Pronto ✅"
    ],
    [
        "id": 243,
        "name": "Douglas Fischer",
        "quote": "To ➡️pagando⬅️ pra ver o que vão escrever de mim. Pelo menos não tem mention no blog! 😅"
    ]
]
```
Para definir os dados que serão retornados, basta retornar um dicionário ou array contendo as informações no bloco da `JSONResponse` - que por sua vez pode estar dentro de uma `DelayResponse`, caso você queira simular um atraso na resposta.

Podemos então escrever o nosso código com base nas informações que nós sabemos que virão do servidor. Segundo o retorno definido, a tableView deve mostrar uma lista com os nomes.

```swift 
func testShowingItems() {
        
    router["/developers"] = DelayResponse(JSONResponse(handler: { [unowned self] _ -> Any in
        return self.mockData
    }), delay: .delay(seconds: 0.0))
    
    app.launch()

    
    XCTAssert(app.tables.cells.children(matching: .staticText)["Emannuel Carvalho"].exists)
    XCTAssert(app.tables.cells.children(matching: .staticText)["Francesco Perrotti-Garcia"].exists)
    XCTAssert(app.tables.cells.children(matching: .staticText)["Kaique D'amato"].exists)
    XCTAssert(app.tables.cells.children(matching: .staticText)["Fernando Bunn"].exists)
    XCTAssert(app.tables.cells.children(matching: .staticText)["Rafael Feroli"].exists)
    XCTAssert(app.tables.cells.children(matching: .staticText)["Steph Curry"].exists)
    
    app.tables.element.swipeUp()
    
    XCTAssert(app.tables.cells.children(matching: .staticText)["Ezequiel França"].exists)
    XCTAssert(app.tables.cells.children(matching: .staticText)["Guilherme Rambo"].exists)
    XCTAssert(app.tables.cells.children(matching: .staticText)["Douglas Fischer"].exists)
    

}
```

Se tudo ocorrer como deveria, a tableView deve conter uma célula com o nome de cada uma das pessoas listadas. O `swipeUp()` é para garantir que mesmo num device com a tela menor, todos os itens poderão ser exibidos.

No nosso exemplo, ao selecionar um nome da lista o app vai pra uma nova tela com os detalhes da pessoa selecionada. Pra isso, é feita uma chamada para a nossa API, passando o `id` do desenvolvedor. Assim, é possível testar se ao selecionar uma pessoa da lista o app está realmente fazendo a chamada adequada.

No próprio bloco onde é definida a resposta pra um dado caminho é possível testar se as informações passadas estão corretas.

```swift
func testSelectingDeveloper() {
        
    router["/developers"] = DelayResponse(JSONResponse() { [unowned self] _ -> Any in
        return self.mockData
    }, delay: .delay(seconds: 0.2))
    
    router["/developer/details"] = JSONResponse() { [unowned self] environ -> Any in
        
        if let input = environ["swsgi.input"] as? SWSGIInput {
            JSONReader.read(input, handler: { (json) in
                if let info = json as? [String: Any] {
                    // o `info` contém os parâmetros passados na chamada
                    XCTAssert(info["developer_id"] as? Int == 214)
                }
            })
        }
        
        return self.userDetailsData
    }
    
    app.launch()
    
    XCUIApplication().tables.staticTexts["Francesco Perrotti-Garcia"].tap()
    
    XCTAssert(app.staticTexts["Francesco Perrotti-Garcia"].exists)
    XCTAssert(app.staticTexts["Vc não quer um pato 🐥"].exists)
    
}
```

Além de garantir que as os parâmetros certos estão sendo enviados, o teste também verifica se o app está mostrando a frase correta - afinal de contas, esse é um teste de UI.

Aqui certamente cabe uma discussão sobre se um teste de _UI_ é o local para verificar os parâmetros passados na chamada http. Eu vou deixar a discussão mais profunda pra quem é mais "gente grande". Nesse post fica só a apresentação da funcionalidade - que no meu caso foi bem útil!

## Concluindo

O `Embassy` e o `Ambassador` me ajudaram bastante com os testes de UI no último projeto em que eu trabalhei e eu espero que possa ser útil pra alguns de vocês também. Vale lembrar que são ferramentas abertas! Nós podemos clonar os projetos, contribuir e propor e implementar novas funcionalidades.

Agradeço a leitura!

Quaisquer críticas e/ou sugestões são mais que bem-vindas. Você pode me encontrar no [Twitter](https://twitter.com/emannuel_oc){:target="_blank"} e no [Slack do iOSDevBR](http://iosdevbr.herokuapp.com){:target="_blank"}.
 
Grande abraço! 🙃



