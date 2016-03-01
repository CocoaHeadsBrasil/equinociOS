---
layout:     post
title:      "Resolvendo suas dependências"
subtitle:   "dados da web e parse de json por sua conta"
date:       2016-03-02 15:58:00
author:     "Daniel Bonates"
header-img: "img/dbonates/header_bg.jpg"
category:   ios
---


Em primeiro lugar, vamos esclarecer alguns pontos que, pelas conversas que tenho tido ultimamente sobre esse assunto, sempre são levantados.

## A que me refiro quando digo “sem dependências”?

Me refiro a concluir uma solução usando ferramentas entregues pelo kit nativo do iOS, sem recorrer a frameworks de terceiros como por exemplo, aquelas bibliotecas lindas que costumamos adicionar aos nossos projetos usando CocoaPods e Carthage.

## É condenavel o uso de bibliotecas nos projetos iOS?

Não e sim! Explico:
Não, por conta do fato de que muito esforço pode ser poupado, tempo de produção e ganho de resultados podem ser alcançados mais facilmente se usarmos o código certo para uma determinada solução. É muito mais rápido e prático quando aplicado com assertividade.
Sim, quando o uso de bibliotecas prontas passa a ser a primeira opção do desenvolvedor. Esse problema foi sentido também na comunidade de desenvolvedores Ruby faz alguns anos, com as famosas gems. Com a chegada do gerenciador de dependências CocoaPods, tornou-se previsível que isso também poderia ser uma armadilha no mundo iOS. E tem sido.

### Quando é legal usar?
1. Quando você sabe que poderia resolver o resolver o problema (mesmo!);
2. Você entende o que a lib está fazendo e que recursos ela está utilizando;
3. A lib realmente vai resolver seu problema, e não criar outros;
4. A lib será usada apenas para acelerar a solução. Exemplo: CoreData;
5. Quando trata-se de algum recurso muito pontual que você acredita que não precisará, ao menos por hora, se aprofundar. Exemplo: Bluetooth.

### Quando NÃO é legal usar?
1. Quando não se faz idéia de como se resolveria o problema e que frameworks podem ser acionados para isso;
2. Muito código e muita dependência será acrescentada à sua base, para pouca funcionalidade realmente utilizada;
3. Quando trata-se de uma operação básica e você não sabe como fazê-la na mão. Exemplos: puxar dados da web, interpretar *JSON*.

### Comparando na prática:

| Usando soluções de terceiros | Desenvolvendo a sua solução |
|-----------------|---------------:|
| pode conseguir solução pontual mais rápida  | tempo = o que você sabe vs. o que precisará saber |
| pode precisar de um tempo pra descobrir qual solução se encaixa melhor no seu problema      |    tempo que você deixa de aprimorar seu conhecimento e suas habilidades como desenvolvedor de soluções  |
| É provável que você terá que lidar com 3 situações:  | Como foi você que fez, nenhum problema, you always win! |
| conviver com uma limitação do framework| -  |
| encarar o código pra entende-lo e fazer ajustes para sua necessidade ou ainda, | - |
| mudar o que você achava que seria a solução ideal, para que tenha condições de usar a biblioteca "ideal". | - |
| aos poucos você vai perceber o quanto está distante de alguns conceitos básicos da plataforma sobre a qual você deveria ser O cara! | Além de saber o que está fazendo, você estará em dia com os rumos da plataforma para a qual você se dispôs a ser um solucionador de problemas. |

Além desses pontos, depender de um framework de terceiro pode se tornar um caso ainda mais crítico nos eventos de updates, principalmente do iOS. Na virada de chave do Swift 1.2 para 2.0, por exemplo, muitos projetos abandonaram o suporte às versões inferiores à 8.0 do iOS, e isso só aumentou o impacto e esforço necessário para essa conversão em apps que dependiam desses frameworks e precisam manter compatibilidade com iOS 7, um problema considerável em projetos grandes. Lá se foram horas e horas pra resolver essas pendências mantendo esses requerimentos por conta própria.

Enfim, tudo isso que pontuei foi apenas pra defender esse ponto de vista: o desenvolvedor tem que se envolver com a plataforma e saber tirar dela o caldo que ela dá com as ferramentas que ela oferece. Isso significa consistência e solidez nas soluções. Usar uma bilblioteca pronta tem que ser para acelerar a solução, sendo sempre assertivo na aplicação, e nunca para suprir uma falta de conhecimento ou domínio do assunto que ela trata. 

> o desenvolvedor tem que se envolver com a plataforma e saber tirar dela o caldo que ela dá com as ferramentas que ela oferece.

> Usar uma bilblioteca pronta tem que ser para acelerar a solução, sendo sempre assertivo na aplicação, e nunca para suprir uma falta de conhecimento ou domínio do assunto que ela trata. 



## Caso clássico: load and parse de *JSON* da web

Vou tentar mostrar agora um caso clássico, presente em muitos testes para vagas de iOS, e que, para minha surpresa, muitos colegas acabam travando se for solicitado para que não seja utilizada dependências externas. O que deveria ser uma solicitação básica acaba virando um problema e um balizador do quão distante o programador está realmente da plataforma que ele usa pra desenvolver.


### O que precisamos?

O processo é tão simples, que basicamente vamos precisar apenas de `NSURLSession`, `NSJSONSerialization` e mais algum código básico para fazer o load dos dados da web e parse dos mesmos.

Eis como a gente pode resolver o load simples de url (_async_):

```swift
let session = NSURLSession.sharedSession()

let task = session.dataTaskWithRequest(request) { (data, response, error) -> Void in
    
    // checar conexão e dados recebidos.
}

task.resume()
```

E para interpretar o *JSON* recebido:

```swift
do {
    if let json = try NSJSONSerialization.JSONObjectWithData(rawData, options: NSJSONReadingOptions.AllowFragments) as? [[String:AnyObject]] {
         return json // conseguimos nosso json :)
    } else {
        print("não foi possível serializar os dados no formato especificado.")
    }
} catch let error as NSError {
    print(error.description)
}
```

Acredite, é basicamente isso. Acredite se quiser, mas vejo muitos devs que desconhecem essa dupla. O que vou propor agora é só dar uma incrementada mas sem adicionar complexidade. O objetivo é tornar essas operações mais genéricas e reutilizáveis, aprimorando o tratamento de erros e resposta, deixando nosso código mais reutilizável:


#### Parte 1: o load

Primeiro, proponho uma estrutura pra facilitar nossa leitura de erros, tratando casos comuns de resposta do servidor:

```swift
enum NetError : ErrorType, CustomStringConvertible {
    case NotFound(Int)
    case Forbidden(Int)
    case ServerResponseError(Int)
    case FatalError(String)
    case Unknown
    
    var description:String {
        
        switch self {
            
        case let .NotFound(statusCode):
            return "Página não encontrada (Erro \(statusCode))"
        case let .Forbidden(statusCode):
            return "Acesso não permitido (Erro \(statusCode))"
        case let .ServerResponseError(statusCode):
            return "Servidor não está respondendo no momento (Erro \(statusCode))"
        case let .FatalError(errorDescription):
            return "Fatal error: \(errorDescription)"
        default:
            return "Erro desconhecido"
        }
    }
}
```

Agora o nosso request, inserimos dentro de uma `func` usando um _callback_ para retornar os dados ou o erro quando for o caso:

```swift
func requestData(request:NSMutableURLRequest, callback:(AnyObject?, NetError?)-> ()) {
    
    let session = NSURLSession.sharedSession()
    
    let task = session.dataTaskWithRequest(request) { (data, response, error) -> Void in
        
        if error != nil {
            callback(nil, NetError.FatalError((error?.localizedDescription)!))
            return
        }
        
        if let response = response as? NSHTTPURLResponse {
            
            switch response.statusCode {
                
            case 200..<300:
                callback(data ?? nil, nil)
            case 401:
                callback(nil, NetError.Forbidden(response.statusCode))
            case 404:
                callback(nil, NetError.NotFound(response.statusCode))
            case let x where x >= 500:
                callback(nil, NetError.ServerResponseError(response.statusCode))
            default:
                callback("alguma coisa deu errado e eu não estou apurando esse caso ainda!", NetError.Unknown)
            }
        } else {
            callback("nenhuma resposta do servidor.", NetError.Unknown)
        }
    }
    
    task.resume()
    
}
```

O `switch` deixa o código um pouco mais extenso, mas o que fazemos aqui é simples, em qualquer situação que não consigamos uma resposta *OK* (statusCode == 200, por exemplo), reportamos o erro, caso contrario, passamos os dados recebidos no _callback_ para que o responsável por esses dados faça o parse do *JSON* recebido.

#### Parte 2: o parse

Agora precisamos extrair um *JSON* dos dados que recuperamos. Atribuindo o processo de parse a uma estrutura também nos dá mais mobilidade. Aqui fazemos isso de uma forma bem simples:

```swift
struct Parser {
    
    typealias StringObjectArrayDataFormat = [[String:AnyObject]]
    
    static func parseData(rawData: NSData) -> StringObjectArrayDataFormat? {
        
        do {
            if let json = try NSJSONSerialization.JSONObjectWithData(rawData, options: NSJSONReadingOptions.AllowFragments) as? StringObjectArrayDataFormat {
                 return json
            } else {
                print("cannot serialize data returned in especified format")
            }
        } catch let error as NSError {
            print(error.description)
        }
        
        return nil
    }
}
```

Note que aqui poderíamos ainda fazer uso de _`Generics`_ no Swift ao invés de `typealias`, o que seria uma opção para tornar o parser ainda mais abrangente para transformar *JSON* em outros objetos, algo tipo isso:

```swift
struct Parser {
    
    static func parseData<T>(rawData: NSData) -> T? {
        
        do {
            if let json = try NSJSONSerialization.JSONObjectWithData(rawData, options: NSJSONReadingOptions.AllowFragments) as? T {

   ...
```

Mas esse assunto é mais extenso, cabe num tópico só sobre _Generics_ e sai bem do escopo do objetivo desse artigo.

#### Parte 3: nosso plano em ação:

Isso posto, vamos para um exemplo de uso dessa proposta. Note que estarei usando um array apenas para facilitar o entendimento:

```swift
if let url = NSURL(string: "https://gist.githubusercontent.com/dbonates/f3d0c4896941c9d0be31/raw/bc8a3f6fcc022fbc8fd38e9aa01d506e838f5451/demodata.json") {
    
    let request:NSMutableURLRequest = NSMutableURLRequest(URL:url)
    
    var namesArray:[String] = []
    
    requestData(request, callback: { (data, error) -> () in
        
        if let error = error {
            print(error)
            return
        }
        
        if let data = data as? NSData {
            if let json = Parser.parseData(data) {
                
                print("aqui está seu json:\n\(json)")
                
                for user in json {
                    if let userFullName = user["user_fullname"] as? String {
                        namesArray.append(userFullName)
                    }
                }
                print(namesArray)
            }
        } else {
            print("nenhum json para intepretar.")
        }
        
    })
    
} else {
    print("url inválida")
}
```

Pronto, está feito! Caso de load e *JSON* resolvido sem precisar de Alamofire, swiftJSON etc...

Como eu sei que é bem provável que você queira na verdade fazer o parse do *JSON* e retorná-lo como um objeto pronto. Dou uma sugestão de como faço isso mais adiante e no Playground vai essa implementação também.

Poderiámos encerrar esse artigo por aqui, mas...

## Dica PRO

Dá pra ficar melhor? 
Sempre! E para fechar segue uma _pro-tip_:

Para fazer o parse do *JSON* retornando um objeto, normalmente eu crio um protocolo `JSONParselable` e nele defino uma função:

```swift
protocol JSONParselable {
    static func withJSON(json: [String:AnyObject]) -> Self?
}
```

Daí é questão de implementar esse protocolo no próprio objeto, de preferência em uma extensão pra separar visualmente as responsabilidades.

O Model User do exemplo é esse:

```swift
struct User {
    var id:Int = 0
    var userFullname:String = ""
    var userAvatar:String = ""
}
```

A implementação do parse nesse model, retornando um User válido apenas se os dados não opcionais sejam encontrados no *JSON*:

```swift
static func withJSON(json: [String:AnyObject]) -> User? {
    
    guard
    let id = int(json, key: "id"),
    userFullname = string(json, key: "user_fullname"),
    userAvatar = string(json, key: "user_avatar")
    else {
        return nil
    }
    
    let user = User(
        id: id,
        userFullname: userFullname,
        userAvatar: userAvatar
    )
    
    return user
}
```

Agora podemos capturar os _users_ usando a implementação do protocolo no User fazendo apenas isso:

```swift
for user in usersJson {
	if let validUser = User.withJSON(user) {
		allUsersList.append(validUser)
	}
}
```

Veja como conseguimos um objeto User válido pra lista apenas usando:

```swift 
User.withJSON(user)
```

Passando para esse método um bloco com os dados do user extraídos do *JSON*. 


That's it! Implementando esse protocolo para cada objeto, você consegue personalizar o tipo de dados e parse para cada objeto.

Não se esqueça de conferir o arquivo Playground desse artigo. Ele contém duas páginas, cada uma contendo uma versão básica e uma mais avançada. As páginas estão acessíveis pelo Project Navigator (**⌘+1**):

<img src="{{ site.baseurl }}/img/dbonates/project_navigator.png">


[O Playground está nesse link](https://github.com/dbonates/load-parse-playground)


## Conclusão
Tentei ser prático em minha peroração. Não tenho a pretensão de ter mostrado o melhor ou mais apurado código, menos ainda uma solução pronta, pois essa demanda é sua! Apenas tentei ser didático sobre uma atitude de se aventurar um pouco mais no iOS. Meu conselho é, sempre que puder, evite uma salada de dependências, e que essa aplicação seja feita com sabedoria e consciencia do valor que ela estará de fato acrescentando ao projeto, incluindo o custo de manutenção e dependência que o uso dela poderá gerar, se for o caso, a longo prazo.

Com esse artigo, espero ter contribuído ao menos 1 byte pra seu conhecimento e tendo sido útil ou não, seu feedback é muito importante e gostaria de encoraja-lo a dizer o que achou desse artigo e contribuir com suas críticas e sugestões.

Um forte abraço a todos e até a próxima!

> Daniel Bonates<br>designer & developer -  <a href="http://bonates.com" target="_blank">bonates.com</a>

