---
layout:     post
title:      "Persistência de dados usando Core Data"
subtitle:   "Uma introdução de como persistir dados no iOS usando o framework nativo"
date:       2017-03-30 12:00:00
author:     "Douglas Taquary"
header-img: "img/douglastaquary/database_blocks.png"
category:   Banco de Dados
---

> Olá pessoal, esse é meu primeiro artigo aqui no EquinociOS e então farei uma breve apresentação.
>  Me chamo [Douglas Taquary](www.twitter.com/viniciusc70https://twitter.com/bluesprogrammer) sou desenvolvedor iOS um pouco mais de 3 anos, fundador e líder do Cocoaheads Teresina desde de 2015, nasci e me criei dentro da eletrônica do meu [Pai](https://www.facebook.com/profile.php?id=100009371490482&fref=ts) 📺🔌🔧 por isso a paixão também por IoT, eletrônica e seus derivados. Sou [guitarrista](https://www.youtube.com/watch?v=J0imW0Xu6nY) (apesar de não estar mais na atividade já algum tempo, somente vez ou outra), sou fã de blues, Jimi Hendrix,  e gosto de beber cerva e cozinhar ao mesmo tempo.
> Feita essa introdução, vamos nessa: **Core Data**!


## Motivação

Há uns dois meses atrás me deparei com uma situação em que precisaria persistir dados em um aplicativo. Eu nunca fui muito fã de trabalhar com banco de dados, talvez por experiências anteriores ou por causa de várias configurações chatas que você tem que fazer pra deixar tudo funcionando.
Acredito que em determinadas tecnologias, não era eu ou você quem deveria estar fazendo esse tipo de configuração(*o framework ou qualquer outra coisa que você esteja usando*) tem que deixar tudo pronto somente para você ir lá e usar. Devo me preocupar com minha lógica de negócios, que no momento é o mais importante.

Bom, mas o jeito foi engolir o choro. ¯\\_(ツ)\_/¯

## O que é o Core Data

O [CoreData](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CoreData/index.html?utm_source=iosstash.io) é um framework nativo como qualquer outro da **Apple**: *UIKit, Foundation* e alguns outros. Ele é usado para persistir, manipular dados no iOS/OSX e usa o `SQLite` por baixo dos panos, mas você não precisar saber `SQL` para usá-lo, ele interage com o *sql* sem você ver ou precisar saber o que está acontecendo.
Usamos o esquema de *key/value* para acessar nossos objetos persistidos.

Algumas de várias características que esse framework possui:

- Gerenciamento e manipulação de objetos(CRUD)
- Buscas sofisticadas. Em vez de escrever SQL, você pode criar consultas complexas associando um objeto *NSPredicate* a uma solicitação de busca.
- Ferramentas de migração de esquema
- Filtragem e gerenciamento de dados na memória e na interface de usuário
- Controle de versão

E várias outras que você pode conferir [aqui](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CoreData/index.html?utm_source=iosstash.io).

Sei que existem outras tecnologias *open source* como o `Realm`, que também podem ser usadas para persistir dados no iOS e que podem ser adicionadas em seu projeto através do `CocoaPods` ou via `módulo`. Mas não é o intuito desse artigo falar sobre prós e contras de tecnologias. Cabe a você escolher a tecnologia que faz sentido no seu projeto.

Blá, blá, blá... então vamos lá. 🎉

## Explicando

Há um tempo atrás, como eu falei, eu criei um [app](https://github.com/douglastaquary/mymovies) para estudar o **Core Data**.

![]({{ site.baseurl }}/img/douglastaquary/mymovies.png)

>O código do app ainda pode melhorar com algumas *refactors* .

No app você pode inserir nomes de filmes favoritos, e pesquisar por eles através de uma *API* chamada [Open Movie Database](http://www.omdbapi.com). A ideia é salvar as informações sobre os filmes e poder vê-las offline, aqui que o *framework entra em ação*. Nosso foco é somente o *Core Data*. ;)

## Vamos lá

Para poder usar o *Core Data* no seu projeto, é necessário habilitá-lo no Xcode no momento que você esta criando um novo projeto, mas ele pode ser adicionado também depois do projeto criado. Além disso, basta fazer `import CoreData` do módulo nas suas classes e pronto. Isso será uma das poucas configurações que você precisará fazer para sair usando.

![]({{ site.baseurl }}/img/douglastaquary/start_coredata.png)

Olha que maravilha?! Mais uma vez o *teorema do taquary* agindo: nosso trabalho não é configurar, é usar, criar.. e blá, blá..

Com projeto criado, abrindo a sua class `AppDelegate.swift`, lá no final do arquivo tem uma variável chamada `var persistentContainer: NSPersistentContainer`.
O Xcode cria essa implementação automaticamente. Ela é basicamente o meio de acesso ao seu banco de dados dentro do Core Data. Ele retorna um `container` se seu banco de dados existe, caso contrário ele retornará um erro, o seu app `crasha` e o Xcode imprime um `log` no console.

Somente isso que precisamos para criarmos nossas `queries`.

Com a chegada do [Swift 3]() as coisas ficaram bem mais fáceis para se trabalhar com o *Core Data*. Menos código e mais eficiência. Como eu disse, deixa o framework cuidar de configurações dele mesmo.

Para mostrar o quanto se trabalhar com `Core Data` depois do `Swift 3` melhorou eu tenho dois arquivos aqui que o *Xcode* gera no `AppDelegate`.



O primeiro com **Swift 2**:

~~~swift
    lazy var applicationDocumentsDirectory: NSURL = {
        let urls = NSFileManager.defaultManager().URLsForDirectory(.DocumentDirectory, inDomains: .UserDomainMask)
        return urls[urls.count-1]
    }()

    lazy var managedObjectModel: NSManagedObjectModel = {
        let modelURL = NSBundle.mainBundle().URLForResource("DATAMODELNAME", withExtension: "momd")!
        return NSManagedObjectModel(contentsOfURL: modelURL)!
    }()
        // Create the coordinator and store
        let coordinator = NSPersistentStoreCoordinator(managedObjectModel: self.managedObjectModel)
        let url = self.applicationDocumentsDirectory.URLByAppendingPathComponent("PROJECTNAME.sqlite")
        var failureReason = "There was an error creating or loading the application's saved data."
        do {
            try coordinator.addPersistentStoreWithType(NSSQLiteStoreType, configuration: nil, URL: url, options: nil)
        } catch {
            // Report any error we got.
            var dict = [String: AnyObject]()
            dict[NSLocalizedDescriptionKey] = "Failed to initialize the application's saved data"
            dict[NSLocalizedFailureReasonErrorKey] = failureReason

            dict[NSUnderlyingErrorKey] = error as NSError
            let wrappedError = NSError(domain: "YOUR_ERROR_DOMAIN", code: 9999, userInfo: dict)
            NSLog("Unresolved error \(wrappedError), \(wrappedError.userInfo)")
            abort()
        }
        
        return coordinator
    }()

    lazy var managedObjectContext: NSManagedObjectContext = {
        // Returns the managed object context for the application (which is already bound to the persistent store coordinator for the application.) This property is optional since there are legitimate error conditions that could cause the creation of the context to fail.
        let coordinator = self.persistentStoreCoordinator
        var managedObjectContext = NSManagedObjectContext(concurrencyType: .MainQueueConcurrencyType)
        managedObjectContext.persistentStoreCoordinator = coordinator
        return managedObjectContext
    }()
~~~


Com **Swift 3**:


~~~swift
    lazy var persistentContainer: NSPersistentContainer = {

        let container = NSPersistentContainer(name: "CoreData_app")
        container.loadPersistentStores(completionHandler: { (storeDescription, error) in
            if let error = error as NSError? {
                fatalError("Unresolved error \(error), \(error.userInfo)")
            }
        })
        return container
    }()
~~~

Que diferença hein?! Pois é... imagina isso em Objective C?! 🙏🏼


## Criando modelos

Outra coisa que podemos observar no nosso projeto, é que temos um arquivo com extensão `.xcdatamodel` que no nosso caso, é chamado `CoreData_app.xcdatamodel`.

![]({{ site.baseurl }}/img/douglastaquary/tree2x.png)

Vamos analisar esse arquivo gerado, aqui criamos e gerenciamos nossas tabelas usando uma interface bem fácil e que poupa muito trabalho. Sem contar que quando você criar uma nova tabela `+ Add Entity` e popular com seus atributos, assim que você `buildar` seu projeto, o Xcode magicamente gera as classes referentes à *estrutura/esquema/modelo* da sua tabela de dados criada. Olha que beleza 2! 🏄🏻

**Aqui os fãs de geradores de código piram** 😅

>O *Xcode* agora suporta a geração automática de subclasses de `NSManagedObject` na ferramenta de modelagem.  

Inspetor de entidade:

![]({{ site.baseurl }}/img/douglastaquary/coredata_panel.png)

## O Mito do App Delegate

Já é de costume os `devs` deixarem esse código onde Xcode o cria. Algo como: eu não vou nem tocar para não quebrar... Isso é **MITO**!!😅
O melhor que você faz é tirar essa implementação do `AppDelegate` e criar algo como `DataManager.swift` e colar ele lá.

## NSManagedObject

O `NSManagedObject` é uma classe genérica que implementa todo o comportamento básico para um objeto no modelo que o Core Data espera. Não é possível usar instâncias de subclasses diretas de `NSObject` (ou qualquer outra classe que não herde de `NSManagedObject`) com um contexto de um objeto gerenciado.
Você pode criar subclasses customizadas do `NSManagedObject`, embora isso nem sempre seja necessário, como eu falei o framework "*gera*" as classes refente as suas tabelas e propriedades.🍺

## Adicionar Dados

No app temos um tela para inserir títulos de filmes. Então, precisamos persistir um atributo *name* do tipo `String` na tabela `Title`. Para eu inserir essa informação, eu simplesmente carrego meu `persistentContainer` através da minha classe `DataManager` no meu `ViewController`

`var dataManager = DataManager()`

e implemento minha função para inserir dados:



~~~swift
    func saveNameOfMovie(with name: String) {
        let managedContext = dataManager.persistentContainer.viewContext
        let entity = NSEntityDescription.entity(forEntityName: "Title", in: managedContext)!
        let movie = NSManagedObject(entity: entity, insertInto: managedContext)
        
        movie.setValue(name, forKey: "title")
        
        do {
            try managedContext.save()
            didEnd()
        } catch let error as NSError {
            print("Could not save. \(error), \(error.userInfo)")
        }
    }

~~~

Aqui nós instanciamos nosso container usando o `managedContext`, depois pegamos a referência da tabela(*você pode ter várias tabelas*) com `entity` e criamos o objeto que será persistido:

`let name = NSManagedObject(entity: entity, insertInto: managedContext)`

E aí eu salvo de fato meu dado usando a funcão `save()` padrão do framework.

`try managedContext.save()`

Tranquilo demais!

## NSFetchRequest

`NSFetchRequest` é um *tipo parametrizado* baseado no novo protocolo `NSFetchRequestResult` adicionado no *Swift 3*. As principais *APIs do Core Data* agora se referem a `NSFetchRequest` *parametrizado* no `Objective C` e no `Swift`.

Além disso, no `Swift`, `NSManagedObjectContext` oferece variantes parametrizadas de `fetch(:)`, que antes do Swift 3 era chamado de `executeFetchRequest: error:`, e `count(:)`.

~~~swift

public func fetch<T : NSFetchRequestResult>(_ request: NSFetchRequest<T>) throws -> [T]

public func count<T : NSFetchRequestResult>(for request: NSFetchRequest<T>) throws -> Int

~~~

É com ele que vamos recuperar nossos dados. 🍷


## Listar nossos dados

Para a minha `tableview` mostrar meus registros de filmes favoritos é mais simples ainda.

Para recuperar os dados persistidos, nós usamos a instância de  `NSFetchRequest`, onde temos que passar o nome da tabela.

~~~swift
func loadData() {
        let fetchRequest = NSFetchRequest<NSFetchRequestResult>(entityName: "Title")
        let context = dataManager.persistentContainer.viewContext
        do{
            let results = try context.fetch(fetchRequest)
            namesOfMovies = results as! [NSManagedObject]
            tableView.reloadData()
        }catch{
            fatalError("Error is retriving titles items")
        }
    }
}

~~~
Depois eu chamo a funcão `fetch()` usando o `context` e passando o `fetchRequest` como parâmetro.

Dai é só transformar o resultado da sua busca em uma `array` de `[NSManagedObject]`, nesse caso, atribuído à *variável* `namesOfMovies` e dar reload na `tableview`.

Então lá na função `cellForRowAt` da `tableView` eu consigo fazer isso:

~~~swift
cell.name.text = nameOfMovie.value(forKey: "title") as? String
~~~
e listar meus objetos persistidos, moleza hein?!

Bom, então é isso, essa foi uma breve introdução.

Existem muito mais operações que podem ser feitas com esse poderoso framework nativo que está ali, só no ponto pra você usar. Espero que esse artigo tenha ajudado você a entender melhor o `Core Data` e algumas de suas estruturas.
Quem sabe em um próximo podemos ir mais a fundo e brincar um pouco mais com ele.

*Quer saber mais*:

[Core Data documentation](https://developer.apple.com/reference/coredata)

[What`s New In Core Data](https://developer.apple.com/library/content/releasenotes/General/WhatNewCoreData2016/ReleaseNotes.html)

[What Is Core Data?](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CoreData/index.html?utm_source=iosstash.io)
