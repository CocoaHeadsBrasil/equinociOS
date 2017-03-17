---
layout:     post
title:      "Enums para o nosso bem"
subtitle:   "código mais seguro, otimizado e legível"
date:       2017-03-17 00:00:00
author:     "Daniel Bonates"
header-img: "img/dbonates/enum_header.png"
category:   ios
---


> [Daniel Bonates](https://bonates.com){:target="_blank"}, problem solver & iOS guy at Peixe Urbano. Minhas paixões digitais são [design](https://dribbble.com/dbonates){:target="_blank"} e desenvovimento de [apps](http://bonates.com/index.html#easybalance){:target="_blank"} para a plataforma Apple. Vivo dessas coisas desde 1998 e meu primeiro app foi para o MacOS 9, em 1991, um player de audio ;) Fiquei desviado produzindo backend e frontend para web, até que a chegada do iPhone me salvou mais uma vez, quando então comprei um Mac e voltei a ser feliz, hoje focado em MacOS, iOS e derivados!

#### `Disclaimer 1`: Estarei tratando aqui do assunto *Enum* usado **Swift**. 

## Introdução
Sempre que falo de Enums, o faço com um sentimento de gratidão por um recurso que já fez muito por mim e pelo código que escrevo. Antes dele, era quase tudo literal, sabe...frágeis strings e constantes para todo lado! Nesse artigo enfrentarei 3 desafios: passar conteúdo útil, ser pragmático e objetivo, e testar minha capacidade de síntese escrevendo o mínimo de blábláblá possível.

### Sobre a minha querida Objective-C:
![Amor eterno.](https://media.giphy.com/media/13tHtQriubWlHy/giphy.gif)

_mas voltando ao assunto..._

## Enum fundamental
Começando do começo, usamos enums para enumerar casos possíveis (ou se preferir, cases ou ainda, statements):

Considere essa estrutura:

{% highlight swift %}
struct Oferta {
    let titulo: String
    let tipo: String
}
{% endhighlight %}

Essa _struct_ possui um título que provavelmente é único. Mas será que podemos dizer o mesmo do _tipo_? Vejamos o caso desse array:

{% highlight swift %}
let ofertas = [
    Oferta(titulo: "Agua Mineral", tipo: "normal"),
    Oferta(titulo: "Macarrão", tipo: "extra"),
    Oferta(titulo: "Vassoura", tipo: "normal"),
    Oferta(titulo: "Shampoo", tipo: "normal"),
    Oferta(titulo: "Sabonente", tipo: "extra"),
    Oferta(titulo: "Coleira", tipo: "extra"),
    Oferta(titulo: "Nutella", tipo: "relampago")
]
{% endhighlight %}

Repetição, margem de erro, retardamento no entendimento de todos os tipos... tudo isso eu sou capaz de sofrer. Então, que tal enumerar os tipos de ofertas? Sugestão:

{% highlight swift %}
enum DescontoTipo {
    case normal
    case extra
    case relampago
}
{% endhighlight %}


Daí nossa struct ficaria ao invés de usar String em _tipo_, usa nosso enum _DescontoTipo_:

{% highlight swift %}
struct Oferta {
    let titulo: String
    let tipo: DescontoTipo
}
{% endhighlight %}

daí o nosso array de ofertas fica assim:

{% highlight swift %}
let ofertas = [
    Oferta(titulo: "Agua Mineral", tipo: .normal),
    Oferta(titulo: "Macarrão", tipo: .extra),
    Oferta(titulo: "Vassoura", tipo: .normal),
    Oferta(titulo: "Shampoo", tipo: .normal),
    Oferta(titulo: "Sabonente", tipo: .extra),
    Oferta(titulo: "Coleira", tipo: .extra),
    Oferta(titulo: "Nutella", tipo: .relampago)
]
{% endhighlight %}

Simples, efetivo e **strongly typed**! Sempre que formos criar uma `Oferta`, o compilador não vai aceitar nada que seja diferente de **DescontoTipo** na propriedade _tipo_, prevenindo erros, entre outras vantagens que já citei.

## Reforçando tipos e legibilidade

**Enums** também podem tornar seu código muito mais legível e flexível. Por exemplo, e se nossa `Oferta` possuir um status de disponibilidade? Talvez o nosso primeiro impulso seria o de criar uma propriedade do tipo _Bool_:

{% highlight swift %}
struct Oferta {
    let titulo: String
    let tipo: DescontoTipo
    let status: Bool
}
{% endhighlight %}

É ok trabalhar aqui com **true/false**. Vai dar conta de boa, certo?! Depende! 

Fazendo isso você acaba de condenar a coitada da `Oferta` a estar disponível ou não, apenas 2 estados! Quer ver? E se a turma de produto chega depois e diz que a oferta pode estar disponível, indisponível, suspensa, encerrada, etc?


![Tudo é relativo!](http://i.giphy.com/l0MYNLZmHQbryIMFO.gif)


Veja como o uso de um _enum_ no lugar de Bool, vai te dar muito mais flexibilidade:

{% highlight swift %}
enum OfertaStatus {
    case disponivel
    case expirada
    case suspensa
    case encerrada
}
{% endhighlight %}

Nossa `Oferta` fica assim:

{% highlight swift %}
struct Oferta {
    let titulo: String
    let tipo: DescontoTipo
    let status: OfertaStatus
}
{% endhighlight %}

daí para criar uma uma oferta:
{% highlight swift %}
let oferta = Oferta(titulo: "Nutella", tipo: .normal, status: .disponivel)
{% endhighlight %}

nice 😊!

Acredito que até aqui, já deu pra ver que usar **enums** é uma obrigação se você pretende manter seu código legível e eficiente. Então vamos agora a um case bem mais comum e pragmático.

----

## Enum na rota, um caso mais "PRO"

Diz-se que em __Swift__ _enums_ tem super poderes. E com razão. São value types, podem ter extensions, variaveis computadas, receber parametros, ter metodos etc. Vamos explorar algumas dessas ferramentas e mostrar como isso pode ser bom pra gente:

A partir daqui:

> Nosso **cenário**: vamos simular que nosso app consome uma API da web, que possui várias rotas e versões.

> Nossa **missão**: criar um **enum** `Router` para gerenciar todas as rotas do nosso app.

#### `Disclaimer 2`: Sobre a 'sacada' que vamos abordar:

Embora a solução final que vou apresentar aqui não seja de minha autoria, acho que vale como caso real pois foi justamente num app super complexo de ecommerce que a vi implementada. Achei uma solução sensacional assim que vi, e a partir daí adotei (com pequenas modificações) em todo app que trabalho. O autor dessa sacada genial foi o [Cassius Pacheco](https://twitter.com/CassiusPacheco_){:target="_blank"} que havia deixado sua marca no app pouco antes de eu chegar na empresa. Vamos destrinchar passo a passo essa solução pra entender como ele conseguiu resolver um sistema complexo de rotas de uma API explorando todo poder de um **Enum** em _Swift_.

Mas vamos começar devagar, com uma versão bem simples da _API_ e vamos aumentando a complexidade para ver até onde o uso de um **enum** pode nos ajudar.

Nossa primeira versão da _API_ é `https://bonates.com/usuarios`, que retorna um _json_ com a lista dos usuários.

Essa é a `func` que usaremos para testar a construção da nossa rota:
{% highlight swift %}
// essa é a nossa função que carrega dados da web
func loadData(request: URLRequest) {
    URLSession.shared.dataTask(with: request) { _,_,_ in
        print("loaded!")
    }
}
{% endhighlight %}

Para a primeira versão da _API_, esse código dá conta de fazer o download dos dados:

{% highlight swift %}
let usersURL = URL(string: "https://bonates.com/usuarios")!
let request = URLRequest(url: usersURL)

// carregando a lista de usuários
loadData(request: request)
{% endhighlight %}

_Usei **force unwrap** aqui! Relaxa que já vai passar!_

Isso pro seu teste num _Playground_ tá ok, mas num app, consumindo uma API de verdade, uma url só não é nossa realidade, confere?

E tem mais! Agora precisamos acessar agora uma nova rota da nossa api que retorna a lista de ofertas: `https://bonates.com/ofertas`. Bastaria criar uma nova e fedorenta variavel `ofertasURL`, certo? Não! Vamos criar nosso _enum_ para começar a cuidar das rotas:

{% highlight swift %}
enum Router: String {
    case usuarios = "https://bonates.com/usuarios"
    case ofertas = "https://bonates.com/ofertas"
}
{% endhighlight %}

Daí basta usar nosso _Router_ pra buscar a url da vez:

{% highlight swift %}
let usersURL = URL(string: Router.usuarios.rawValue)! // force unwrap de novo :boom:
{% endhighlight %}

Pronto, de novo nos livramos de literais e quem olhar essa linha de código vai saber que estamos recuperando uma url relacionada a _usuarios_. Bem mais profissional, não?! (Eu sei, mas ainda tem a exclamação do mal ali, calma, já disse)

Mas isso é muito pouco perto do que ainda podemos fazer pra esse código ficar bom. E tem mais uma novidade: nossa api retorna também ofertas favoritas do usuário logado. O link é esse: `https://bonates.com/ofertas/favoritas`

**Enums** aceitam variáveis computadas, então, ao invés de criar outro case `favoritas` e retornar toda a **url inteira**, podemos torna-la mais dinâmica gerando-a a partir de seus componentes. Isso fica assim:

{% highlight swift %}
enum Router {
    case usuarios
    case ofertas
    case favoritas
    
    // host comum a todos
    var basePath: String {
        return "https://bonates.com"
    }
    
    // o path que varia de acordo com o case
    var path: String {
        switch self {
        case .usuarios: return "usuarios"
        case .ofertas: return "ofertas"
        case .favoritas: return "ofertas/favoritas"
        }
    }
       
    // o request que nos interessa
    var request:URLRequest? {
        
        let baseUrlPath = "\(basePath)/\(path)"
        let baseURL = URLComponents(string: baseUrlPath)
        
        guard
            let url = baseURL?.url else {
                print("Impossível iniciar request com essa url.")
                return nil
        }
        
        let request = URLRequest(url: url)
        return request
    }
}
{% endhighlight %}

Nosso `Router` agora nos retorna um **request** pronto, bastando pra isso dizer qual request queremos, inclusive com a ajuda do _auto complete_ para os mais preguiçosos, me included o/

<img src="{{ site.baseurl }}/img/dbonates/enums-autocomplete.png">

So far, so good ;) Olha como agora um simples *if-let* resolve nossos problemas de request válido e uso de *force unwrap*. No código a seguir, o **request** vai me retornar `https://bonates.com/ofertas/favoritas` lindamente, safe and sound!

{% highlight swift %}
if let request = Router.favoritas.request {
    loadData(request: request)
}
{% endhighlight %}

### Passando parametros e query strings

Nossa api agora tem um recurso que lista apenas os usuários ativos: `https://bonates.com/usuarios?status=ativos`. Daí, evoluindo nosso enum:

1 - mudamos o case `.usuarios` para aceitar parametros:

{% highlight swift %}
case usuarios([String:Any])
{% endhighlight %}

2 - adicionamos uma variável computada:

{% highlight swift %}
var parametros: [String: Any] {
    switch self {
    case .usuarios(let urlParams):
        return urlParams
    default:
        return [:]
    }
}
{% endhighlight %}

3 - E fazemos uso dela na geração do _request_:

{% highlight swift %}
...
baseURL?.queryItems = parametros.map{ qParamter in
    return URLQueryItem(
        name: qParamter.key, 
        value: "\(qParamter.value)"
    )
}
...
{% endhighlight %}

Agora qualquer quantidade de parametros passados como `String:Any` vai ser aceito e entrar como _query string_:

{% highlight swift %}
// request vai retornar: 
// https://bonates.com/usuarios?cidade=rio-de-janeiro&status=ativo
let parametros = ["status":"ativo", "cidade": "rio-de-janeiro"]
if let usuariosAtivosReq = Router.usuarios(parametros).request {
    loadData(request: request)
}
{% endhighlight %}

### Metodo HTTP

Perto do que já fizemos, incluir a capacidade de atribuir um método http de acordo com o caso acaba sendo trivial. Basta adicionar uma variável no _Router_ com essa finalidade:

{% highlight swift %}
var metodo: String {
    switch self {
    default:
        return "GET"
    }
}
{% endhighlight %}


Só deixei o `GET` implementado como case padrão. Incrementar um `POST, DELETE, etc` é com você!

![](https://media.giphy.com/media/yoJC2K6rCzwNY2EngA/giphy.gif)

O Router até aqui ficou assim:

{% highlight swift %}
enum Router {
    case usuarios([String:Any])
    case ofertas
    case favoritas
    
    var basePath: String {
        return "https://bonates.com"
    }
    
    var path: String {
        switch self {
        case .usuarios: return "usuarios"
        case .ofertas: return "ofertas"
        case .favoritas: return "ofertas/favoritas"
        }
    }
    
    var parametros: [String: Any] {
        switch self {
        case .usuarios(let urlParams):
            return urlParams
        default:
            return [:]
        }
    }
    
    var metodo: String {
        switch self {
        default:
            return "GET"
        }
    }
    
    var request:URLRequest? {
        
        let baseUrlPath = "\(basePath)/\(path)"
        var baseURL = URLComponents(string: baseUrlPath)
        
        baseURL?.queryItems = parametros.map{ qParamter in
            return URLQueryItem(name: qParamter.key, value: "\(qParamter.value)")
        }
        
        guard
            let url = baseURL?.url else {
                print("Impossível iniciar request com essa url.")
                return nil
        }
        
        
        let request = URLRequest(url: url)
        return request
    }
}
{% endhighlight %}

Pronto! Cobrimos um caso real explorando os recursos de enumeração disponíveis em **Swift** para criarmos um sistema de rotas fácil de escalar e fazer manutenção. Tem mais possibilidades, mas basicamente elas seriam uma combinação do que já fizemos aqui ;)

## Enums + Generics - o poder não tem limites!

E pra fechar, você sabia que uma variável optional na real implementa o enum Optional da biblioteca padrão do Swift?

{% highlight swift %}
// Optional simplificado
enum Optional<T> {
    case none
    case some(T)
}
{% endhighlight %}

ou seja, quando definimos uma variável como *optional*, basicamente estamos fazendo isso:

{% highlight swift %}
let variavel1: Int? = nil // nil
let variavel2: Optional<Int> = .none // nil
let variavel3: Int? = 42 // 42
let variavel4: Optional<Int> = .some(42) //42
{% endhighlight %}

Viu só?! **Enum** está no coração do Swift 😊!

Usando essa mesma idéia, podemos criar por exemplo um recurso muito útil para tratar erro e sucesso em requests:

{% highlight swift %}
enum RequestResult<T> {
    case successo(T)
    case erro(Error)
}
{% endhighlight %}

A dica aqui é sua rotina retornar um enum _RequestResult_, verificando o resultado da operação:

{% highlight swift %}
WebService.json(from: request) { result in
    switch result {
    case .erro(let erro):
        // tratar o erro
        
    case .successo(let json):
        // usar os dados retornados
    }
}
{% endhighlight %}

Fica fácil, né?!

## Considerações finais

Bem, esses são os meus argumentos pra te convencer de que enumerar seus dados usando os super poderes da nossa linguagem do coração, vai deixar seu código lindo, seus coleguinhas vão curtir quando precisarem dar manutenção e provavelmente o Crashlytics vai dar uma relaxada porque você não tipou errado! Espero ter contribuído com algumas idéias e uma inspiração a mais para tornar seu código mais seguro, organizado e fácil de ler.

Dúvidas, comentários, whatever, pode tacar por aqui, me parar na rua, ou me procurar lá no [Slack da comunidade iOS](http://iosdevbr.herokuapp.com), onde estou sempre presente, ainda que apenas invisível só acompanhando as _tretas_! 

Abraços e até logo!




