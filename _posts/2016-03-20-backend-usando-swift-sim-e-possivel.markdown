---
layout:     post
title:      "Backend usando Swift ? Sim, é possível!"
subtitle:   "Entenda um pouco sobre o projeto open source que vai te ajudar a ver o desenvolvimento de software para servidores de outra forma usando Swift!"
date:       2016-03-20 00:00:00
author:     "Thiago Holanda"
header-img: "img/tholanda/milky-way-graceful-arc.jpg"
category:   open-source
---

> Thiago Holanda ([@tholanda](https://twitter.com/tholanda){:target="_blank"}), desenvolvedor de software há um bocado de tempo, já trabalhou em tudo que foi tipo de projeto. Começou no backend, se meteu a besta com banco de dados, iniciou no iOS, se embrenhou pelo client side, voltou pro iOS e agora voltou a namorar o backend, ufa... Marido da Carolina, pai de duas trombadinhas felinas e quando tem um tempinho dá uns rolés de skate por aí, que é a terceira maior paixão, depois da Carolina e da programação!
<br>

### Um pequeno overview

Recentemente, enquanto preparava uma apresentação sobre *"Swift no Backend"* para o encontro de desenvolvedores de uma grande empresa, conversei com alguns amigos no [Slack](https://iosdevbr.herokuapp.com), nos corredores da empresa e percebi que nos dias de hoje temos excelentes desenvolvedores móveis, que por muitas vezes não conhecem outras tecnologias, senão *iOS* ou *Android*.

Pessoas que acompanharam a popularização dessas novas tecnologias através da Apple e do Google, investiram tempo e dinheiro nesse segmento, e deixaram de lado o restante das engrenagens que fazem esse grande relógio chamado [World Wide Web](https://en.wikipedia.org/wiki/World_Wide_Web){:target="_blank"} funcionar.

Eu mesmo, durante alguns anos, deixei o backend de lado e esquecido acreditando que parte de um novo futuro estava nas mãos de empresas que estavam surgindo sob uma nova sigla: [BaaS](https://en.wikipedia.org/wiki/Mobile_backend_as_a_service){:target="_blank"} *(Backend as a Service)* . Achei que o *[Parse](https://www.parse.com){:target="_blank"}* era a grande solução, quase uma bala de prata. Desenvolvi alguns sistemas usando a ferramenta e não posso dizer que me arrependi.

Depois de um tempo trilhando esse caminho, comecei a perceber quão intrusivo era o [Parse](https://www.parse.com){:target="_blank"}. Queria me livrar das garras dele e então renasceu a idéia de me dedicar mais ao backend e buscar alternativas de código aberto, que fosse semelhante ao que o [Parse](https://www.parse.com){:target="_blank"} poderia me oferecer. Nessa busca, tinha em mente algumas coisas: 1) open source, 2) [Swift](https://www.swift.org){:target="_blank"}, 3) backend, e é claro, iniciar o próximo projeto em [Swift](https://www.swift.org){:target="_blank"}. 

Por falta de tempo ou vontade de sair da zona de conforto, esse projeto em [Swift](https://www.swift.org){:target="_blank"} não foi iniciado. Finalmente, no início de Dezembro de 2015, a *Apple* libera o código fonte do *[Swift](https://www.swift.org){:target="_blank"}*, o que foi ótimo. Em Janeiro, conversando com o [Ricardo Borelli](https://twitter.com/rabc){:target="_blank"}, percebemos que havia uma vontade mútua de aprender o *[Swift](https://www.swift.org){:target="_blank"}*, mas não para trabalhar no *iOS*. Queríamos aprender a linguagem de programação, não o *Framework*. A pergunta nesse momento, era: "Como?".

As formas que tínhamos para iniciar nosso aprendizado no *[Swift](https://www.swift.org){:target="_blank"}* eram: 

- Desenvolvendo uma aplicação iOS (fora do nosso objetivo inicial)
- Contribuir com projeto open source escrito em *[Swift](https://www.swift.org){:target="_blank"}*

Por termos trabalhado alguns anos como desenvolvedores Backend, a segunda opção era mais atraente, então iniciamos as buscas por projetos open source que nos permitissem desenvolver *web applications* usando *[Swift](https://www.swift.org){:target="_blank"}*.

Primeiro chegamos ao [CocoaPods App](https://github.com/CocoaPods/CocoaPods-app){:target="_blank"}. Desencantamos, pois era um projeto com diversas classes escritas em *Objective-C*. Em seguida, encontramos o [Perfect](http://www.perfect.org/){:target="_blank"}, o [Vapor](https://github.com/qutheory/vapor){:target="_blank"}, e então o [Ricardo](https://twitter.com/rabc){:target="_blank"} encontrou o [Zewo](http://zewo.io){:target="_blank"}. A partir daí começou de fato o envolvimento com um projeto 100% open source para servidor, onde aprenderíamos [Swift](https://www.swift.org){:target="_blank"}.

<br>

### O que é o Zewo?

O *[Zewo](http://zewo.io){:target="_blank"}* é um conjunto de bibliotecas voltadas para o desenvolvimento de software para servidores. Diversas bibliotecas do *[Zewo](http://zewo.io){:target="_blank"}* são nada mais do que wrappers de classes, que foram escritas em *[C](https://en.wikipedia.org/wiki/C_(programming_language)){:target="_blank"}*.

<br>

### A História do Zewo

O *[Zewo](http://zewo.io){:target="_blank"}* nasceu em torno de Junho de 2015, na época em que a *Apple* anunciou que abriria o código fonte do *[Swift](https://www.swift.org){:target="_blank"}*. O primeiro nome dele foi *[SwiftHTTPServer](https://github.com/paulofaria/SwiftHTTPServer){:target="_blank"}* e foi construído pelo *[Paulo Faria](https://twitter.com/paulofaria){:target="_blank"}* - um dev brasileiro - que manteve a construção do *[SwiftHTTPServer](https://github.com/paulofaria/SwiftHTTPServer){:target="_blank"}* em constante crescimento, até pouco antes da *Apple* finalmente liberar o código fonte do *[Swift](https://www.swift.org){:target="_blank"}*, quando ele decidiu reiniciar tudo. E então surgiu o *[Zewo](http://zewo.io){:target="_blank"}* com uma proposta diferente: "não ser apenas um web framework, mas uma plataforma de desenvolvimento para tudo que fosse relacionado a servidor". E tudo isso graças ao fato de compilar para Linux. 

<br>

### Se é uma plataforma para servidores, me dá uns exemplos do que eu posso fazer com o Zewo.

Você pode trabalhar com praticamente tudo que você quiser no servidor. 
Alguns poucos exemplos:

- Crawlers
- Loggers
- Renders 
- Database (MySQL e Postgres)
- Scripts
- Web Applications 
- Conexões remotas usando TCP/UDP

E o mais interessante: sem precisar da [Foundation](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/ObjC_classic/){:target="_blank"}, que ainda não possui o código fonte liberado.

<br>

### Zewo + Swift X

Desde o início, modularização foi um dos focos principais do [Zewo](http://zewo.io){:target="_blank"}. Isso faz com que a reutilização de código ocorra de forma muito mais fácil. Quando Vapor (um dos frameworks web para swift) resolveu se integrar aos módulos do Zewo, surgiu a idéia de criar um padrão para compartilhamento de código entre projetos Swift. Nasceu então o [Swift X](https://github.com/swiftx/docs){:target="_blank"} (Swift Cross Project Standards). O [Swift X](https://github.com/swiftx/docs){:target="_blank"} é uma coleção de protocolos e tipos básicos, passando por dados binários, requisições e respostas *HTTP*, *URIs*, até alcançar um padrão para servidores e clientes *HTTP*, roteadores *HTTP*, *middlewares*, etc.

O *[Swift X](https://github.com/swiftx/docs){:target="_blank"}* ainda é bem novo, mas já tem alguns frameworks iniciando o suporte a seus padrões. A próxima versão do *[Zewo](http://zewo.io){:target="_blank"}* já vai fazer uso do *SX*, facilitando ainda mais o compartilhamento de código e integração entre a comunidade *[Swift](https://www.swift.org){:target="_blank"}*.

<br>

### Vamos começar?

Você precisa de algumas ferramentas. *Vâmo* lá, receita de bolo 😜:

- [Homebrew](http://brew.sh)
- Um editor de texto 
    - [CodeRunner](https://coderunnerapp.com){:target="_blank"}
    - [Sublime Text](https://www.sublimetext.com/){:target="_blank"}
    - [Atom](https://atom.io){:target="_blank"}
    - [TextMate](https://macromates.com/){:target="_blank"}
    - Você não manda em mim, vou usar o TextEdit 😂

Depois de instalado, será necessário adicionar a fórmula do *[Zewo](http://zewo.io){:target="_blank"}* para o [Homebrew](http://brew.sh). No *terminal*, digite o seguinte comando

```sh
$ brew tap zewo/tap
$ brew install zewo
```
<br>

### Toolchain

Para iniciar os trabalhos de forma simples, usando o *Mac OS X*, precisaremos do *toolchain* do *[Swift](https://www.swift.org){:target="_blank"}*.

> Se você já tem tudo isso instalado, vá direto para _[Criando o Hello World](#criando-um-hello-world)_

Faça o download do toolchain de 08 de Fevereiro em *[https://swift.org/download/#apple-platforms](https://swift.org/download/#apple-platforms){:target="_blank"}*

![Swift Download Page]({{ site.baseurl }}/img/tholanda/swift-download-page.png)

Finalizado o download, execute o `swift-DEVELOPMENT-SNAPSHOT-2016-02-08-a-osx.pkg`

Será necessário que você adicione à sua variável de ambiente, o path do toolchain que foi instalado.

Abra o *terminal* e digite o seguinte comando

```sh
$ open ~/.bash_profile
```

Adicione o path padrão de instalação do *[Swift](https://www.swift.org){:target="_blank"}* (caso você não tenha alterado o diretório de instalação)

```sh
/Library/Developer/Toolchains/swift-latest.xctoolchain/usr/bin
```

O resultado final da variável de ambiente *PATH* deverá ser algo semelhante ao screenshot. Salve a modificação no *.bash_profile*.

![]({{ site.baseurl }}/img/tholanda/environment-variable-path-textedit.png)

Feito isso, será necessário executar o *.bash_profile*, usando o comando abaixo

```sh
$ source ~/.bash_profile
```
<br>

### Criando um Hello World

Vamos voltar ao *terminal*

```sh
$ cd ~/
$ mkdir hello-world
$ cd hello-world
$ swift-build --init
Creating Package.swift
Creating .gitignore
Creating Sources/
Creating Sources/main.swift
Creating Tests/
```

Com um editor de texto, edite o arquivo *Package.swift* e adicione os repositórios [HTTPServer](https://github.com/Zewo/HTTPServer){:target="_blank"}, [Router](https://github.com/Zewo/Router){:target="_blank"} e [LogMiddleware](https://github.com/Zewo/LogMiddleware){:target="_blank"} à nossa lista de dependências


*Package.swift* 

```swift
import PackageDescription

let package = Package(
    name: "hello-world",
    dependencies: [
        .Package(url: "https://github.com/Zewo/HTTPServer.git", majorVersion: 0, minor: 3),
        .Package(url: "https://github.com/Zewo/Router.git", majorVersion: 0, minor: 3),
        .Package(url: "https://github.com/Zewo/LogMiddleware.git", majorVersion: 0, minor: 3)
    ]
)
```

*Sources/main.swift*

```swift
import HTTPServer
import Router
import LogMiddleware

let log = Log()
let logMiddleware = LogMiddleware(log: log)

let router = Router(middleware: logMiddleware) { route in
    route.get("/") { request in
        return Response(body: "Hello World!")
    }
}

try Server(port: 8080, responder: router).start()
```

Após implementar essas ~~milhares~~ poucas linhas de código volte ao *terminal*, no diretório do projeto, e execute o compilador do *[Swift](https://www.swift.org){:target="_blank"}*

```sh
$ swift-build
```

Na primeira vez que você executar o `swift-build` no diretório do seu projeto, o *[Swift Package Manager](https://github.com/apple/swift-package-manager){:target="_blank"}* fará o clone dos projetos que adicionamos na chave `dependencies`, no `Package.swift` e suas subdependências, em seguida os sources serão compilados. Por fim, o resultado será um executável gerado pelo compilador

```sh
$ swift-build
Compiling Swift Module 'helloworld' (1 sources)
Linking Executable:  .build/debug/hello-world
```

Agora que o compilador disse pra gente o que compilou e onde gerou o executável, fica fácil saber o que executar. 

Assumindo que você está no diretório `hello-world`, digite o seguinte no *terminal*

```sh
$ .build/debug/hello-world



                             _____
     ,.-``-._.-``-.,        /__  /  ___ _      ______
    |`-._,.-`-.,_.-`|         / /  / _ \ | /| / / __ \
    |   |ˆ-. .-`|   |        / /__/  __/ |/ |/ / /_/ /
    `-.,|   |   |,.-`       /____/\___/|__/|__/\____/ (c)
        `-.,|,.-`           -----------------------------

================================================================================
Started HTTP server, listening on port 8080.
```

Agora que temos o nosso servidor rodando devidamente no *terminal*, abra o browser e digite a seguinte URL: `http://127.0.0.1:8080` e você verá algo como o screeshot abaixo

![]({{ site.baseurl }}/img/tholanda/hello-world-running-on-browser.png)

Por ter configurado  o [LogMiddleware](https://github.com/Zewo/LogMiddleware){:target="_blank"} no `Package.swift`, as requisições disparadas contra o servidor serão mostradas no console.

> Em diversos web frameworks, mostrar os detalhes da requisição no console é parte da configuração padrão. Como no *[Zewo](http://zewo.io){:target="_blank"}* separamos todos os módulos, deixamos essa configuração como opção para o usuário. No *Hello World* não estamos persistindo esses dados, mas caso seja interessante, você pode usar o [File](https://github.com/Zewo/File){:target="_blank"} para guardá-los em um arquivo.

![]({{ site.baseurl }}/img/tholanda/hello-world-showing-log-on-console.png)

Enfim, depois de tanta explicação, esse é o nosso *Hello World* usando *[Swift](https://www.swift.org){:target="_blank"}* e *[Zewo](http://zewo.io){:target="_blank"}*. 
Por que não incrementar um pouco mais esse exemplo ? Fique tranquilo que a partir de agora não serei tão detalhista! ~~será?~~ 😌

<br>

### Criando um render html

Legal, bacana, temos o *Hello World* funcionando. Mas se estamos falando sobre montar uma *web application*, então é necessário renderizar um *HTML* com informações vindas do sevidor, não é mesmo ? Por isso, vamos usar o [Sideburns](https://github.com/Zewo/Sideburns){:target="_blank"} *- uma pequena camada sobre o [Mustache](https://github.com/Zewo/Mustache){:target="_blank"} que é um fork do [GRMustache](https://github.com/groue/GRMustache.swift){:target="_blank"} -* e o [HTTPFile](https://github.com/Zewo/HTTPFile){:target="_blank"} que é a implementação do protocolo *ResponderType* e será usado para expor o conteúdo da pasta `public`. 

Ao adicionar o [Sideburns](https://github.com/Zewo/Sideburns){:target="_blank"} à lista de `dependencies`, o`Package.swift` deverá ficar assim

```swift
import PackageDescription

let package = Package(
    name: "hello-world",
    dependencies: [
        .Package(url: "https://github.com/Zewo/HTTPServer.git", majorVersion: 0, minor: 3),
        .Package(url: "https://github.com/Zewo/Router.git", majorVersion: 0, minor: 3),
        .Package(url: "https://github.com/Zewo/LogMiddleware.git", majorVersion: 0, minor: 3),
        .Package(url: "https://github.com/Zewo/HTTPFile.git", majorVersion: 0, minor: 3),
        .Package(url: "https://github.com/Zewo/Sideburns.git", majorVersion: 0, minor: 2)
    ]
)
```

Show! Logo após adicionar os novos pacotes, o ideal seria voltar ao *terminal* e rodar o `swift-build`, mas podemos esperar um pouquinho mais e fazer tudo de uma vez.
Mas se paciência não for muito seu forte, manda ver! Mal não vai fazer! 😜

Na raiz do seu projeto vamos criar uma pasta chamada `public` e vamos adicionar alguns arquivos simples. No *terminal* digite

```sh
$ mkdir -p public/assets
```

crie um arquivo `main.css` dentro da pasta `assets`

```sh
$ touch public/assets/main.css
```

edite o `main.css` e vamos adicionar uma simples propriedade, apenas para que a nossa página não fique completamente sem estilo

```css
body {
    font-family: "verdana";
    font-size: 10px;
}
```
crie na raiz da pasta `public` um arquivo chamado `hello.html`

```sh
$ touch public/hello.html
```

edite o `hello.html` e adicione o seguinte bloco ao arquivo

> caso você tenha dúvidas sobre a forma de trabalhar com o *Mustache*, acesse a documentação da ferramenta no link [https://github.com/groue/GRMustache](https://github.com/groue/GRMustache){:target="_blank"}.

```html
{% raw %}<html>
    <head>
        <title>{{ title }}</title>
        <meta charset="UTF-8">
        <link rel="stylesheet" type="text/css" href="/assets/main.css">
    </head>
    <body>
        <div>
            <h1>
            {{ description }}
            </h1>
            <ul>
                {{# messages }}
                <li> {{ author }} - {{ message }} </li>
                {{/ messages }}
            </ul>
        </div>
    </body>
</html>{% endraw %}
```

O que adicionaremos agora ao nosso `Sources/main.swift`:

- Estrutura de dados para ser renderizado no `html`, que será o response do endpoint "/"
- Response com path do arquivo que será usado para renderizar o dicionário que acabamos de adicionar 
- `FileResponder`, que será responsável por expor os arquivos da pasta `public` de forma estática


O resultado final do nosso `main.swift`

```swift
import Zewo
import HTTPFile
import Mustache
import Sideburns

let log = Log()
let logMiddleware = LogMiddleware(log: log)

let router = Router(middleware: logMiddleware) { route in
    route.get("/") { request in
        let data: [String: Any] = [
            "title": "Título da Página",
            "description": "Renderizando HTML usando Sideburns",
            "messages": [ 
                [
                    "author": "Patrick", 
                    "message": "Lorem ipsum dolor sit amet, consectetur adipiscing elit."
                ],
                [
                    "author": "Senhor Sirigueijo", 
                    "message": "Morbi luctus urna vel lacus malesuada posuere."
                ],
                [
                    "author": "Lula Molusco", 
                    "message": "Praesent dignissim nunc a convallis posuere."
                ],
                [
                    "author": "Bob Esponja", 
                    "message": "Nam facilisis arcu at consequat sagittis."
                ]
            ]
        ]
        return try Response(templatePath: "public/hello.html", templateData: data)
    }
    route.fallback = FileResponder(basePath: "public")
}

try Server(port: 8080, responder: router).start()
```

Ok, terminamos de incrementar o nosso *Hello World* com a renderização de algumas informações do servidor. Nesse caso, os dados estão todos *"mockados"*, mas poderiam vir diretamente de um banco (que é assunto para um próximo artigo 😉). Agora vamos compilar as nossas mudanças e executar o binário gerado pelo compilador. Para isso, volte ao *terminal* e execute

```sh
$ swift-build
$ .build/debug/hello-world
```

Abra o browser e digite o endereço `http://127.0.0.1:8080`. Ao carregar a tela, esse deve ser o resultado

![]({{ site.baseurl }}/img/tholanda/beautiful-rendering-browser.png)

e o `html` será parecido com que está listado abaixo

```html
<html>
    <head>
        <title>Título da Página</title>
        <meta charset="UTF-8">
        <link rel="stylesheet" type="text/css" href="/assets/main.css">
    </head>
    <body>
        <div>
            <h1>
            Renderizando HTML usando Sideburns
            </h1>
            <ul>
                <li> Patrick - Lorem ipsum dolor sit amet, consectetur adipiscing elit. </li>
                <li> Senhor Sirigueijo - Morbi luctus urna vel lacus malesuada posuere. </li>
                <li> Lula Molusco - Praesent dignissim nunc a convallis posuere. </li>
                <li> Bob Esponja - Nam facilisis arcu at consequat sagittis. </li>
            </ul>
        </div>
    </body>
</html>
```

e por aqui, finalizamos mais um exemplo de como podemos renderizar valores em um `html`. 

<br>

### Criando uma API simples

Agora, vamos usar um repositório que já tem como dependência diversos outros que precisamos para *web applications* - o [Zewo](https://github.com/Zewo/Zewo){:target="_blank"} - um repositório que podemos chamar de *umbrella package*, fazendo uma referência a [umbrella header](https://developer.apple.com/library/mac/documentation/MacOSX/Conceptual/BPFrameworks/Tasks/IncludingFrameworks.html){:target="_blank"}.

> Por conta de diversas libraries ainda estarem sendo escritas, principalmente no caso de rotas RESTful, nesse artigo veremos apenas as rotas de consulta mais simples. Em breve escreverei outro post abordando APIs mais complexas, prometo 😉🙏!

Nesse exemplo criaremos uma API ridiculamente simples. O objetivo não é ser complexo e por isso não faria sentido criar muitos métodos. 

Então vamos começar criando um novo pacote para a nossa API ? De volta ao *terminal* crie uma nova pasta com o nome `todo-api`

```sh
$ cd ~/
$ mkdir todo-api
$ cd todo-api
$ swift-build --init
Creating Package.swift
Creating .gitignore
Creating Sources/
Creating Sources/main.swift
Creating Tests/
```

vamos editar o arquivo `Package.swift` e adicionar a única dependência que precisaremos, o [Zewo](https://github.com/Zewo/Zewo){:target="_blank"}

> Se você quiser conhecer quais são as dependências usadas por esse repositório, acesse essa url: [https://github.com/Zewo/Zewo/blob/master/Package.swift](https://github.com/Zewo/Zewo/blob/master/Package.swift){:target="_blank"}

```swift
import PackageDescription

let package = Package(
    name: "todo-api",
    dependencies: [
        .Package(url: "https://github.com/Zewo/Zewo", majorVersion: 0, minor: 3)
    ]
)
```

agora nós precisamos criar de fato as nossas rotas e respostas, sendo assim, edite o arquivo `Sources/main.swift` e cole o conteúdo do bloco abaixo

```swift
import HTTPServer
import Router
import LogMiddleware
import HTTPFile
import ContentNegotiationMiddleware
import LogMiddleware
import JSONMediaType
import InterchangeData

let log                 = Log()
let logMiddleware       = LogMiddleware(log: log)
let contentNegotiaton   = ContentNegotiationMiddleware(mediaTypes: JSONMediaType())

let router = Router(middleware: logMiddleware, contentNegotiaton) { route in
    
    var items: [InterchangeData] = [
        ["id": "1", "description": "Comprar frutas"],
        ["id": "5", "description": "Pagar condomínio"],
        ["id": "3", "description": "Despachar encomenda"],
        ["id": "6", "description": "Assistir Deadpool"],
        ["id": "4", "description": "Ir ao banco"],
        ["id": "2", "description": "Ligar para operadora"]
    ]
    
    func get(id: String) throws -> InterchangeData {
        let list = items.filter({ $0["id"]!.string == id })

        guard let todo = list.first else {
            return nil
        }
        
        return todo
    }
    
    // cria o novo registro de uma tarefa
    route.post("/todo") { request in
        let body        = request.content!
        
        let id          = InterchangeData.from(String(arc4random()))
        let description = body["description"]
        
        let todo: InterchangeData = [
            "id"          : id,
            "description" : description!
        ]
        
        items.append(todo)
        
        return Response(status: .Created, content: todo)
    }
    
    // Lista todas os registros de tarefas
    route.get("/todo") { request in
        items = items.sort({ $0["id"]!.string < $1["id"]!.string })
        
        guard items.count > 0 else {
            return Response(status: .NoContent)
        }
        
        return Response(content: InterchangeData.from(items))
    }
    
    // busca os dados de uma tarefa baseado no id
    route.get("/todo/:id") { request in
        let path = request.pathParameters
        
        let todo = try get(path["id"]!)
        guard todo != nil else {
            return Response(status: .NotFound)
        }
        
        return Response(content: todo)
    }
    
    // atualiza uma tarefa baseado no id
    route.put("/todo/:id") { request in
        let path = request.pathParameters
        let body = request.content!
        
        var todo = try get(path["id"]!)
        guard todo != nil else {
            return Response(status: .NotFound)
        }
        
        let index = items.indexOf(todo)!
        todo["description"] = body["description"]
        items[index] = todo
        
        return Response(status: .NoContent)
    }
    
    // exclui uma tarefa baseado no id
    route.delete("/todo/:id") { request in
        let path = request.pathParameters
        
        let todo = try get(path["id"]!)
        guard todo != nil else {
            return Response(status: .NotFound)
        }
        
        let index = items.indexOf(todo)!
        guard items.removeAtIndex(index) != nil else {
            return Response(status: .NotFound)
        }
        
        return Response(status: .NoContent)
    }
}

try Server(port: 8080, responder: router).start()
```

É claro que você, um expert em Swift, pode criticar diversos pontos do bloco acima, infelizmente ainda não faço parte desse ~~seleto~~ time, mas estamos à caminho.

Neste exemplo, criamos:

- Criação de uma tarefa
- Listagem das tarefas
- Detalhe de uma tarefa
- Atualização de uma tarefa
- Exclusão de uma tarefa

Em princípio, para que não ficasse com mais dependências (ex: banco de dados), usamos um array mutável para persistir e listar as tarefas. É claro que essa não chega nem a ser uma forma decente de persistência, mas atende ao nosso propósito educativo. 


> **ContentNegotiationMiddleware**
> 
> Esse middleware tem como função básica o tratamento do tipo de informação recebida no request. É baseado no `Content-Type` e no `Accept`. No nosso exemplo, trabalharemos apenas com `JSON`. Caso haja necessidade, você pode optar por usar o [URLEncodedForm](https://github.com/Zewo/URLEncodedForm){:target="_blank"}.

Depois de rodar o exemplo anterior, você pode estar perguntando: *como proteger a nossa humilde API ?* Bom, temos à disposição diversas ferramentas para nos auxiliar nesse ponto, mas por enquanto, apenas um está implementado de forma nativa no [Zewo](https://github.com/Zewo){:target="_blank"}: o [BasicAuthMiddleware](https://github.com/Zewo/BasicAuthMiddleware){:target="_blank"} [(RFC 2617)](https://tools.ietf.org/html/rfc2617){:target="_blank"}.

Infelizmente, o [HTTP Basic Authentication](https://tools.ietf.org/html/rfc2617){:target="_blank"} não é seguro em relação ao dado trafegado. Encriptar os dados requer *[Base64](https://en.wikipedia.org/wiki/Base64){:target="_blank"}*, uma encriptação de duas vias, ou seja, a segurança é tão fraca que, qualquer pessoa que interceptar a sua conexão com o servidor, facilmente descobrirá qual usuário e senha está sendo transmitido. Se você usa *HTTPS*, a vida de quem está tentando interceptar os dados trafegados complica um pouco mais.

<br>

### Como adicionar o BasicAuth nas minhas rotas ?

Em outras ocasiões, você precisaria adicionar o pacote *[BasicAuthMiddleware](https://github.com/Zewo/BasicAuthMiddleware){:target="_blank"}* na lista de dependências do `Package.swift`, mas nós estamos usando o [Zewo](https://github.com/Zewo/Zewo){:target="_blank"}, que é o nosso *umbrella package* e ele já possui essa dependência. Então, ao invés de trabalhar com diversas dependências, nós vamos voltar a editar o `Sources/main.swift`. 

Importe o `BasicAuthMiddleware` no topo do `main.swift`

```swift
import BasicAuthMiddleware
```

vamos instanciar o `BasicAuthMiddleware`

```swift
let basicAuth = BasicAuthMiddleware { username, password in
    if username == "admin" && password == "password" {
        return .Authenticated
    }

    return .AccessDenied
}
```

e configurar no nosso `Router`. Basta adicionar a variável `basicAuth` ao parâmetro `middleware` e deve ficar assim

```swift
let router = Router(middleware: logMiddleware, contentNegotiaton, basicAuth) { route in 
```

O que precisávamos para "segurar" a nossa API era basicamente esses dois passos. O resultado disso é simples: se você não informar as credenciais no momento da requisição, a API recusará o seu acesso.

![]({{ site.baseurl }}/img/tholanda/api-secured-without-credentials.png)

Ao informar credenciais válidas, a API devolverá os dados normalmente é efetuará as devidas ações, como: insert, update, delete e quantos mais você configurar

![]({{ site.baseurl }}/img/tholanda/api-secured-with-credentials.png)

e finalmente, terminamos por aqui o terceiro e último exemplo do nosso artigo.

<br>

### Considerações finais

Como você pode ver, o desenvolvimento de *web applications* não é realmente um monstro de sete cabeças como talvez você estivesse imaginando. É claro que no desenvolvimento de software tudo pode piorar, e pode apostar que vai. Com um pouco de estudo, dedicação, fonte de pesquisa (tá bom, Stack Overflow vale também 😛) e uma boa documentação, você pode e DEVE ir longe. 

Durante o tempo em que estive escrevendo esse post, a *Apple* liberou mais um *toolchain* com diversas novas possibilidades que, com toda certeza, nos ajudarão muito daqui pra frente. Pelo fato de agora o [Swift](https://www.swift.org){:target="_blank"} também compilar *[C](https://en.wikipedia.org/wiki/C_(programming_language)){:target="_blank"}*, podemos fazer deploy das aplicações escritas em [Swift](https://www.swift.org){:target="_blank"} para o [Heroku](https://www.heroku.com){:target="_blank"}, mas isso também ficará para uma próxima vez.

Os códigos dos exemplos estão todos no meu perfil do Github, na url [https://github.com/unnamedd/equinocios-backend](https://github.com/unnamedd/equinocios-backend){:target="_blank"}

Hoje, dia 20 de Março, seria o nosso último dia de [EquinociOS](http://equinocios.com){:target="_blank"}, mas devido a tantos ótimos desenvolvedores se interessarem pelo projeto, foi decidido que prorrogaríamos até o final do mês. Isso quer dizer que até o dia 31 de Março, você vai poder acompanhar artigos inéditos sobre o mundo de desenvolvimento da *Apple*.

<br><br>

Esse post teve a participação direta de diversos amigos, sugerindo, corrigindo ou ensinando.

Meus agradecimentos ao *[Paulo Faria](https://twitter.com/paulofaria){:target="_blank"}*, ao *[Ricardo Borelli](https://twitter.com/rabc){:target="_blank"}* e a minha amada Carolina.

> A foto de capa é da Via Láctea numa belíssima noite! 
