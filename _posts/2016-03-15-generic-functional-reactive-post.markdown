---
layout:     post
title:      "Programação funcional e reativa é para todos"
subtitle:   "Como inserir RxSwift + Programação funcional naquele seu projeto que esta 90% concluído"
date:       2016-03-15 00:00:00
author:     "Bruno Bilescky"
header-img: "img/bgondim/header_programming.jpg"
category:   "functional"
---
> #### Bruno Bilescky ([@bgondim](https://twitter.com/bgondim){:target="_blank"}) 
> `let currentJob = "desenvolvedor backend(java, ruby, c#) ".map { _ in "Desenvolvedor frontend(js) " }. map { _ in "desenvolvedor mobile (Objective-C, Java, C#" }.map { _ in "desenvolvedor iOS(Swift)" }`.
> Estudante das artes de programação funcional e reativa

Muitos desenvolvedores quando escutam falar de programação funcional e programação reativa assumem uma postura defensiva e a conversa tende a não sair do lugar. _"Mas o meu aplicativo ja esta 50% desenvolvido, não vale a pena mudar a arquitetura dele agora."_, ou então _"Eu só estou dando manutenção neste aplicativo, não tenho tempo para alterar a arquitetura dele."_, ou ainda _" Não posso mudar a arquitetura do projeto, porque ninguém mais da equipe sabe essa magia negra aí"_ são frases frequentes que vamos encontrar ao tentar sugerir a adoção desses paradigmas.

Mas o que esses desenvolvedores não sabem é que programação funcional/reativa **não** necessita que você altere a arquitetura do seu projeto.

**A programação funcional tem muito mais a ver com paradigmas a serem utilizados do que com frameworks e arquiteturas de projetos**.

Mas obviamente existem diversas arquiteturas e frameworks que facilitam a adoção desses paradigmas, porém elas não são obrigatórias e você pode **sim** começar hoje mesmo a inserir códigos funcionais e reativos no seu projeto.

Neste artigo vamos fazer extenso uso de [RxSwift](https://github.com/ReactiveX/RxSwift){:target="_blank"}, uma biblioteca de programação Reativa.

Agora, caso este seja o seu primeiro contato com esta biblioteca, ou você ainda não se sinta confortável com os paradigmas funcionais, ou ainda você é um marinheiro de primeira viagem neste blog, eu recomendo que você leia  ["RxSwift: Como vim parar aqui"](http://equinocios.com/2016/03/14/rxswift-como-eu-vim-parar-aqui/){:target="_blank"}, publicado pelo [Bruno Koga](https://twitter.com/brunokoga){:target="_blank"} aqui no EquinociOS. Ou ainda, visite a [documentação](https://github.com/ReactiveX/RxSwift/blob/master/Documentation/GettingStarted.md) do projeto.

___

### Fazendo mais com menos
Bom esta é a ideia deste post. Mostrar como podemos utilizar o paradigma da programação funcional, junto com RxSwift e com uma pitada de **Generics** para tornar o seu código mais reativo e funcional.

E para isso eu desenvolvi um aplicativo de gerenciamento de dispesas ([github](https://github.com/brunogb/ExpenseTracker){:target="_blank"}) para ilustrar alguns pontos que quero abordar. Eu sugiro que você baixe o código, de uma olhada no projeto, brinque um pouco com o aplicativo e depois volte para continuarmos.

### Bom, agora que você voltou podemos começar...
Sobre a arquitetura do projeto, além do **RxSwift** para a programação reativa, estou utilizando o **[Realm](https://realm.io)** para gerenciar o banco de dados local, o **[Hue](https://github.com/hyperoslo/Hue)** para o gerenciamento de cores, o **[SnapKit](https://github.com/SnapKit/SnapKit)** para usar autolayout no código e o **[NibDesignable](https://github.com/mbogh/NibDesignable)** para gerenciar as telas e deixar mais leve nosso storyboard.

Um dos recursos mais interessantes do **Realm**, a lista com _live update_ permite que façamos uma query no banco, que está sempre atualizada, incluindo atualizações que transações futuras podem efetuar na base, facilitando manter nossa UI sempre fresca e atualizada.

Porém o **Realm** não tem suporte nativo para RxSwift. No entanto, é muito simples criar nossas próprias sequências de dados (**[Observables](https://github.com/ReactiveX/RxSwift/blob/master/Documentation/GettingStarted.md#observables-aka-sequences)**, como são chamados no RxSwift) e podemos nós mesmos adicionar esse suporte.

**[@fpillet](https://twitter.com/fpillet)**(um usuário bem ativo da comunidade de RxSwift) fez um [gist](https://gist.github.com/fpillet/4ceb477eeb2705fb5159) adicionando este recurso ao **Realm** e nós vamos utilizar este gist no nosso projeto.

Bom vamos ao código:

>#### Talk is cheap. Show me the code
>_Torvalds, Linus_

~~~swift

import UIKit
import RealmSwift
import RxSwift

struct AppState {
    
    private let disposeBag = DisposeBag()
    static let current = AppState()
    let currentCategory = Variable<ExpenseCategory?>(nil)
    let entries = Variable<[Expense]>([])
    let allEntries: Observable<[Expense]>
    let allExpenses: Observable<[Expense]>
    let currentExpensesTotal: Observable<Double>
    let currentTintColor: Observable<UIColor>
    
    private init() {
        let observableCategory = self.currentCategory.asObservable()
        
        observableCategory.map { (category) -> Int in
            guard let selectedCategory = category else {
                return -1
            }
            return selectedCategory.rawValue
        }.distinctUntilChanged().map { (categoryNumber) -> Observable<[Expense]> in
            let realm = try! Realm()
            var results = realm.objects(Expense)
            if categoryNumber >= 0 {
                results = results.filter("_category == \(categoryNumber)")
            }
            results = results.sorted("date", ascending: false)
            return results.asObservableArray()
        }.switchLatest().bindTo(self.entries).addDisposableTo(disposeBag)
        self.allEntries = self.entries.asObservable()
        
        self.allExpenses = self.allEntries.map { (expenses) -> [Expense] in
            return expenses.filter { expense in
                return expense.type == ExpenseType.Outcome
            }
        }
        
        self.currentExpensesTotal = self.allExpenses.map { (list) -> Double in
            return list.reduce(0.0, combine: { (total, expense) -> Double in
                return total + expense.amount
            })
        }
        
        self.currentTintColor = observableCategory.map { (category) -> UIColor in
            return UIColor.hex(category?.tintColor ?? ExpenseCategory.allColors)
        }
    }
    
}
~~~

Basicamente este é um objeto que representa o **_estado atual_** do aplicativo. Ele é responsável por rastrear as mudanças no banco de dados e expor os valores ja mapeados para serem consumidos pelas outras classes e funções. Vamos então analisar as partes deste código:

### Mantendo sempre o "estado atual" atualizado...

~~~swift
let currentCategory = Variable<ExpenseCategory?>(nil)

let entries = Variable<[Expense]>([])
let allEntries: Observable<[Expense]>
let allExpenses: Observable<[Expense]>
let currentExpensesTotal: Observable<Double>
let currentTintColor: Observable<UIColor>
~~~
Estas são as propriedades disponíveis para serem observadas, e basicamente todas são dependentes de `currentCategory`. Quando a categoria atual é alterada, todas as outras propriedades também são atualizadas. Conseguimos este feito com o seguinte código:

~~~swift
let observableCategory = self.currentCategory.asObservable()        
observableCategory.map { (category) -> Int in
    guard let selectedCategory = category else {
        return -1
    }
    return selectedCategory.rawValue
}.distinctUntilChanged().map { (categoryNumber) -> Observable<[Expense]> in
    let realm = try! Realm()
    var results = realm.objects(Expense)
    if categoryNumber >= 0 {
        results = results.filter("_category == \(categoryNumber)")
    }
    results = results.sorted("date", ascending: false)
    return results.asObservableArray()
}.switchLatest().bindTo(self.entries).addDisposableTo(disposeBag)
~~~
Primeiro estamos mapeando a categoria selecionada(`ExpenseCategory`) para um `Int` (Caso nenhuma categoria esteja selecionada, mapeamos o resultado para `-1`). Por que um `Int`? Porque este é o valor mapeado no banco de dados do Realm. 

Mas nós não queremos atualizar a query quando selecionarmos uma nova categoria que seja igual a categoria atual (A nossa lista já é auto-atualizável, não precisamos fazer nada aqui). Essa é a função do `distinctUntilChanged`. Ele filtra resultados que sejam iguais ao último enviado.

Depois, utilizamos este `Int` para fazer nossa query no banco de dados (`"_category == \(categoryNumber)"`), e utilizando aquele [gist](https://gist.github.com/fpillet/4ceb477eeb2705fb5159), retornamos uma sequência observável destes registros encontrados pela nossa query.

Perceba que nesse passo estamos gerando um `Observable` que produz um `Observable` que produz uma lista de Expenses(`<[Expense]>`). Confuso? A princípo pode parecer que sim, mas para tentar entender, **vamos fazer uma analogia com uma impressora 3D. Pense que um `Observable` é como uma impressora 3D especializada em imprimir um objeto de um tipo `E`. No nosso caso, nós estamos criando uma impressora 3D que imprime outra impressora 3D especializada em imprimir uma lista de `E`** (é como se toda vez que mudamos `self.currentCategory` criamos uma nova impressora 3D que imprime os itens). Mas nós estamos interessados em observar a lista de Expenses e não as impressoras que geram elas. E como fazemos isso? Utilizando o `switchLatest`. Com este operador nós utilizamos os resultados sempre da _última impressora que imprime a lista de itens_ que for gerada pela nossa impressora matriz. Com isso nós criamos uma Impressora que imprime uma lista de `E`, ou `Observable<[E]>`

>Assim: `Observable<Observable<[Expense]>>.switchLatest = Observable<[Expense]>`

Espero que essa analogia tenha facilitado um pouco as coisas.

E por fim salvamos uma referencia a esse `Observable` em `self.entries`. Com isso, toda vez que `self.currentCategory` receber um novo valor distinto do anterior, iremos refazer nossa query e com isso atualizar a variável `self.entries` com os novos registros. 
E como utilizamos aquele gist, `self.entries` vai sempre se manter atualizada com os registros filtrados do banco, mesmo após sua inicialização.

### ... Para sempre manter nossa UI atualizada.

~~~swift
tableView.registerNib(R.nib.expenseDisplayTableViewCell)
let cellIdentifier = R.nib.expenseDisplayTableViewCell.identifier
AppState.current.allEntries.bindTo(tableView.rx_itemsWithCellIdentifier(cellIdentifier, cellType: ExpenseDisplayTableViewCell.self)) {(index, item, cell) in
	let number = NSNumber(double: item.amount)
	let value = self.numberFormatter.stringFromNumber(number) ?? ""
	cell.amountLabel.text = value
	cell.categoryNameLabel.text = item.category?.description ?? "Sem categoria"
	cell.applyColor(UIColor.hex(item.category?.tintColor ?? ExpenseCategory.allColors))
}.addDisposableTo(disposeBag)
~~~
Aqui estamos criando um datasource dinâmico e anônimo, já configurado para exibir todos os items de `AppState.current.allEntries`. Se a lista de itens de `allEntries` mudar, nossa `tableView` também será atualizada automaticamente. 

E para manter nossos labels atualizados e a tela com a cor da categoria:

~~~swift
AppState.current.currentTintColor.subscribeNext(self.applyColor).addDisposableTo(disposeBag)
AppState.current.currentCategory.asObservable().map { (category) -> String in
    return category?.description ?? "Todas as categorias"
}.bindTo(amountType.rx_text).addDisposableTo(disposeBag)
AppState.current.currentExpensesTotal.map({ (value) -> String in
    return self.numberFormatter.stringFromNumber(NSNumber(double: value)) ?? ""
}).bindTo(self.amountLabel.rx_text).addDisposableTo(disposeBag)
~~~
Com esse código nossa UI vai estar sempre atualizada com a última categoria selecionada.

### Ok, este RxSwift parace interessante mesmo, mas e a tal da programação funcional? Onde vamos utilizar?
Na verdade nós já estamos utilizando e talvez você não tenha percebido. Quando executamos: 

~~~swift
observableCategory.map { (category) -> Int in
    guard let selectedCategory = category else {
        return -1
    }
    return selectedCategory.rawValue
}
~~~

estamos criando uma **função anônima** e passando ela como parâmetro para a função `map`. Porém, poderíamos ter criado uma função assim:

~~~swift
func mapExpenseCategoryToInt(category: ExpenseCategory?) -> Int {
        guard let selectedCategory = category else {
            return -1
        }
        return selectedCategory.rawValue
    }
~~~
e utilizado esta função, nossa chamada do `map` ficaria deste jeito:

~~~swift
observableCategory.map(mapExpenseCategoryToInt)
~~~
Como já estávamos fazendo na função que altera a cor do nosso cabeçalho:

~~~swift
AppState.current.currentTintColor.subscribeNext(self.applyColor).addDisposableTo(disposeBag)
~~~
Na programação funcional, as funções que criamos são cidadãs de primeira classe, assim como objetos e value types, e podem ser passadas como parâmetros, serem referenciadas e executadas arbitráriamente.


### Hummm, interessante, mas e o Generics? Onde ele se encaixa nessa história toda?
Generics te ajuda a escrever menos código e abranger mais situações. Ao escrever funções genericas você está aumentando o escopo onde estas funções podem ser utilizadas. E isso faz todo o sentido quando falamos de RxSwift e programação funcional.

Além do que temos menos código para testar e manter.

Praticamente todos os operadores que utilizamos do RxSwift são genéricos. `switchLatest`, `map`, `filter`, todos podem ser utilizados de maneira genérica, desde que os tipos de retorno desses `Observable`s conformem com os protocolos específicos.

### Legal, mas tudo que você mostrou eu consigo fazer sem RxSwift, onde mais ele pode facilitar a minha vida?
Um fato bem interessate da programação reativa é que nós podemos juntar as sequências, criando um fluxo bem complexo a partir de fluxos mais simples e fáceis de testar. E aí a programação reativa começa a brilhar, pois para juntar estes fluxos de maneira imperativa seria necessario muito mais códigos, além de refatoramentos e mudanças nas APIs dos métodos.

Vamos, como exemplo, rastrear o status de conexão com a internet. Veja esta classe:

~~~swift
import ReachabilitySwift
import RxSwift

let Reachable = _Reachable()
struct _Reachable {
    
    let reachability: Reachability
    let disposeBag = DisposeBag()
    let internetStatus = Variable<Reachability.NetworkStatus>(.NotReachable)
    
    private init() {
        reachability = try! Reachability.reachabilityForInternetConnection()
        createObservable().bindTo(self.internetStatus).addDisposableTo(disposeBag)
    }
    
    private func createObservable()-> Observable<Reachability.NetworkStatus> {
        return Observable.create { observer in
            let cancel = AnonymousDisposable {
                self.reachability.stopNotifier()
            }
            let reachableUpdateBlock: (Reachability-> Void) = { r in
                observer.onNext(r.currentReachabilityStatus)
            }
            
            self.reachability.whenReachable = reachableUpdateBlock
            self.reachability.whenUnreachable = reachableUpdateBlock
            
            try! self.reachability.startNotifier()
            return cancel
        }
    }
}
~~~
Aqui estamos criando uma sequencia que informa o estado atual da conexão com a internet. Se quisermos apenas consultar o ultimo valor enviado podemos utilizar `Reachable.internetStatus.value`.

Mas podemos criar funções que operem em cima desse `Observable`.

~~~swift
func filterInternetIsActive()-> Observable<Reachability.NetworkStatus> {
    return Reachable.internetStatus.asObservable().filter { status in
        switch status {
        case .NotReachable:
            return false
        default:
            return true
        }
    }
}

func filterInternetIsOffline()-> Observable<Reachability.NetworkStatus> {
    return Reachable.internetStatus.asObservable().filter { status in
        switch status {
        case .NotReachable:
            return true
        default:
            return false
        }
    }
}
~~~
Agora para  exibir um alerta sempre que a conexão cair podemos utilizar: `filterInternetIsOffline().subscribeNex(funcToShowAlert)`

Ou ainda podemos fazer uma requisição no servidor, e caso estejamos sem internet, ou ela caia no meio da conexão, podemos tentar novamente quando a conexão voltar:

~~~swift
extension Observable {
    func retryOnInternetBecomeActive() -> Observable<E> {
        return self.retryWhen({ (error) -> Observable<Reachability.NetworkStatus> in
            return filterInternetIsActive()
        })
    }
}
~~~
Agora com esse método podemos fazer nossa chamada na API e garantir que caso nossa conexão caia, quando ela voltar esta chamada será refeita:

~~~swift
func tryConnection()-> Observable<NSData> {
        let request = NSURLRequest(URL: NSURL(string: "http:path_to_api")!)
        return NSURLSession.sharedSession().rx_data(request).retryOnInternetBecomeActive()
    }
~~~

Depois basta executar: `tryConnection().subscribeNext(funcToParseJSON)` e pronto, temos aqui nossa requisição tolerante a quedas de internet.

### Finalizando

Bom RxSwift tem ainda diversos outros recursos, e infelizmente não vamos conseguir cobrir todos aqui. Espero que os recursos apresentados aqui, junto com os exemplos fornecidos possam lhe mostrar como pode ser simples e fácil de adicionar programação funcional/reativa no seu código ja existente, sem que você tenha que alterar a arquitetura do seu projeto.

E isso é tudo pessoal! Até o próximo post! E acompanhem o desenvolvimento do [ExpenseTracker](https://github.com/brunogb/ExpenseTracker), pois pretendo evoluí-lo com o tempo adicionando mais recursos funcionais e reativos.

### # Agradecimentos
- A equipe do RxSwift pela ótima lib que eles entregam
- A equipe do CocoaHeads BR pela iniciativa! (Valeu [Solli](https://github.com/shonorio)!!)


### # Referências
- [RxSwift](https://github.com/ReactiveX/RxSwift) - [Documentação](https://github.com/ReactiveX/RxSwift/blob/master/Documentation/) 
- [Slack da comunidade RxSwift](http://slack.rxswift.org/)
- [Turn Realm auto-updating Results into an RxSwift Observable sequence](https://gist.github.com/fpillet/4ceb477eeb2705fb5159) by [@fpillet](https://twitter.com/fpillet)
- [ExpenseTracker](https://github.com/brunogb/ExpenseTracker) by [@bgondim](https://twitter.com/bgondim)