---
layout:     post
title:      "Protocol-Oriented Programming"
subtitle:   "Vivendo em um mundo abstrato."
author:     "Lourenço Marinho"
date:       2016-03-16 00:00:00
header-img: "img/post-exemplo.jpg"
category:   ios
---

> **Equinócio**
>
> Fenômeno onde a duração do dia é idêntica à da noite e os hemisférios Norte e Sul recebem a mesma quantidade de luz.

O objetivo desse artigo é mostrar como podemos usar _Protocol Oriented Programming_ para melhorarmos as aplicações, criando bibliotecas mais reutilizáveis e genéricas.

Apesar de todo o hype criado pela [palestra](https://developer.apple.com/videos/play/wwdc2015-408/) na WWDC de 2015 sobre protocolos, esse não é um assunto novo no mundo de desenvolvimento iOS. Sempre usamos protocolos quando queremos por exemplo usar uma `UITableView` ou uma `UICollectionView` e é um padrão adotado em várias outras bibliotecas de código aberto.

Mesmo que Swift e Objective-C usem protocolos de forma extensa, as duas linguagens usam protocolos de forma bem diferente. Objective-C usa protocolos geralmente como uma maneira de diminuir o acoplamento entre objetos. Por exemplo, uma `UITableView` não precisa saber quais tipos de objetos são responsáveis por ser sua _data source_ ou _delegate_, tudo que ela precisa saber é que esse objetos implementam os protocolos `UITableViewDataSource` e `UITableViewDelegate`.

~~~ objc
@interface ViewController : UIViewController <UITableViewDataSource, UITableViewDelegate>
	@property (strong, nonatomic) IBOutlet UITableView *tableView;
@end
~~~

~~~ objc
@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];

    self.tableView.dataSource = self;
    self.tableView.delegate = self;
}

- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section {
    return numberOfCellsForTable;
}

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:@"Cell" forIndexPath:indexPath];

    // Customização da célula

    return cell;
}

@end
~~~

### Protocolos em Swift
Swift também incorpora o conceito de desacoplamento de objetos através do uso de protocolos, mas adiciona uma nova dimensão a estes. Um exemplo é quando são usados para generalizar código,  através de  _Associated Types_ ou com o uso  de **Self** na definição de métodos.

Vamos começar analisando o uso de _Associated Types_ com o protocolo [`GeneratorType`](http://swiftdoc.org/v2.1/protocol/GeneratorType/) e como ele encapsula a interação de objetos dentro de uma [sequência](http://swiftdoc.org/v2.1/protocol/SequenceType/). Para isso podemos separar sua definição em duas partes: primeiro ele define um _Associated Type_ `Element`, que terá seu tipo definido posteriormente por quem adote esse protocolo; em seguida, define a função `next()` que retorna um valor opcional do tipo `Element`. 

~~~ swift
protocol GeneratorType {
	typealias Element
	mutating func next() -> Self.Element?
}
~~~

Para ilustrar o uso de _Associated Types_,  criamos um  *Generator*  que produz constantes quando seu método `next()` é executado. 

~~~swift
class ConstantGenerator: GeneratorType {
	typealias Element = Int

	func next() -> Int? {
		return 100
	}
}
~~~

~~~ swift
let generator = ConstantGenerator()

while let c = generator.next() {
	print("Gerando a constante \(c) ao infinito e além!")  
	//  Gerando a constante 100 ao infinito e além!
}
~~~

### Code Smell
Quando trabalhamos com APIs antigas, nem sempre podemos contar com o suporte de genéricos, o que vai contra a nova filosofia que [Swift traz](https://github.com/apple/swift/blob/master/docs/Generics.rst#motivation).

> _Swift is intended to be a small, expressive language with great support for building libraries. We'll need generics to be able to build those libraries well._

Um exemplo dessa diferença é observado quando usamos [CoreData](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/CoreData/index.html).  Para obtermos registros em nossa base de dados, usamos um [NSFetchRequest](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/CoreDataFramework/Classes/NSFetchRequest_Class/) que retorna um _Array_ de _AnyObject_.

~~~ swift
let fetchRequest = NSFetchRequest(entityName: "Person")
fetchRequest.predicate = NSPredicate(format: "name = %@", "João")
        
let results = try! context.executeFetchRequest(fetchRequest)
print(results.dynamicType) // Array<AnyObject>
~~~

Como vemos no código acima, precisamos usar uma *String* para indicar qual a entidade vamos buscar na base de dados e posteriormente é necessário um _casting_ para acessarmos os atributos do tipo `Person`.

~~~ swift
let fetchRequest = NSFetchRequest(entityName: "Person")
fetchRequest.predicate = NSPredicate(format: "name = %@", "João")
        
let results = try! context.executeFetchRequest(fetchRequest) as! [Person]
 
print(results.dynamicType) // Array<Person>
print(results[0].name) // "João"
~~~

Quando vemos muitos _casting_ ou uso de literais de forma desordenada podemos considerar isso um [*code smell*](https://en.wikipedia.org/wiki/Code_smell). Felizmente com o uso de *Protocol-Oriented Programming* podemos deixar esse código um pouco mais Swifty.  

Boa parte dos aplicativos que desenvolvemos usa algum tipo de persistência de dados local e um dos *frameworks* mais usados para atender esse requisito é o *CoreData*.
Para o exemplo que vamos mostrar nesse artigo, vamos precisar entender outra maneira de como declarar protocolos genéricos. 

### Self Requirement
Podemos criar protocolos genéricos usando a palavra reservada **Self** na declaração de métodos e/ou variáveis. Ao incluir **Self** na declaração de métodos,  estamos dizendo que aquele valor é apenas um *placeholder* e será substituído pelo tipo que adota este protocolo.

Para entender melhor o uso da palavra **Self**, vamos imaginar um protocolo que representa tipos de dados que serão lidos de alguma fonte de dados. 

~~~ swift
protocol Readeable {
	static func byName(name: String) -> [Self]
}
~~~

~~~ swift
class Person: Readeable {
	static func byName(name: String) -> [Person] {
		let appDelegate = UIApplication.sharedApplication().delegate as! AppDelegate
		let context = appDelegate.managedObjectContext

		let fetchRequest = NSFetchRequest(entityName: "Person")
		fetchRequest.predicate = NSPredicate(format: "name = %@", name)

		return try! context.executeFetchRequest(fetchRequest) as! [Person]
	}
}
~~~

Quando adotamos o protocolo `Readeable` na classe `Person` podemos substituir o retorno da função `byName(name:)` pelo tipo que implementa o protocolo. 

### Protocol Extension
Até o Swift 1.2 não podíamos fornecer uma implementação genérica para protocolos, o que muitas vezes resultava em duplicação de código. No exemplo abaixo se quisermos que a classe `Pet` tenha as funções de `Readable`, temos basicamente os mesmo código com pequenas modificações.

~~~ swift
class Pet: Readeable {
	static func byName(name: String) -> [Pet] {
		let appDelegate = UIApplication.sharedApplication().delegate as! AppDelegate
		let context = appDelegate.managedObjectContext

		let fetchRequest = NSFetchRequest(entityName: "Pet")
		fetchRequest.predicate = NSPredicate(format: "nickname = %@", name)

		return try! context.executeFetchRequest(fetchRequest) as! [Pet]
	}
}
~~~

Felizmente Swift 2.0 possibilitou através de *Protocol Extension* implementações padrão para métodos ou propriedades de protocolos, possibilitando a criação de código mais genérico e reutilizável.

### Active Record
Nosso objetivo final nesse artigo é criar uma biblioteca que implemente parte do padrão arquitetural [*Active Record*](https://en.wikipedia.org/wiki/Active_record_pattern), utilizando CoreData como nossa base de dados.

Então vamos em frente e adicionar mais um protocolo que será responsável pela persistência dos dados em nossa base.

~~~ swift
protocol Writeable {
    func save()
}
~~~

Um outro protocolo que irá ser responsável por deletar objetos da nossa base de dados.

~~~ swift
protocol Deletable {    
    func destroy()
}
~~~

E vamos modificar o protocolo `Readable` deixando ele mais completo.

~~~ swift
protocol Readable {
	static func all() -> [Self]
	static func find(predicate: NSPredicate) -> [Self]
}
~~~

Como já foi dito, iremos usar *CoreData* como forma de persistência, porém vale ressaltar que os protocolos `Writeable`, `Deletable`, `Readable` não fazem nenhuma premissa de qual base de dados vamos usar ao longo do projeto.

> A separação de operações em protocolos nos possibilita a criação de outros tipos que contenham apenas algumas funcionalidades.

Logo em seguida vamos usar herança entre protocolos para compor um novo protocolo chamado `ActiveRecordType`. Este por sua vez, agrega as operações que um *Active Record* pode realizar.

~~~ swift
protocol ActiveRecordType: Writeable, Deletable, Readable {
    
}
~~~

Abstraindo ainda mais, vamos definir o protocolo `ModelType`, representando tipos que são armazenados em uma base de dados. Esse protocolo herda as funcionalidades de `ActiveRecordType`, declara um *Associated Type* `Context`, representando o contexto usado em operações na base de dados, e por fim uma variável para acessar esse contexto.

~~~ swift
protocol ModelType: ActiveRecordType {
	typealias Context
	static var context: Self.Context { get }
}
~~~

Por fim, criamos o protocolo `CoreDataModel` que herda de `ModelType` e define o tipo de `Context` que será utilizado. Já que estamos trabalhando com **CoreData**, naturalmente, nosso `Context` será um `NSManagedObjectContext`

~~~ swift
protocol CoreDataModel: ModelType {
    typealias Context = NSManagedObjectContext
}
~~~

### Revisando nossa arquitetura
Agora é uma boa hora para revisar nossa hierarquia antes de irmos em frente. Definimos protocolos para leitura (`Readable`), escrita (`Writeable`), remoção (`Deleteable`)  e agregamos todas as operações em um `ActiveRecordType`. No final da hierarquia temos os protocolos `ModelType`e `CoreDataModel` que representam tipos que usam um contexto para armazenar dados.

<img src="{{site.baseurl}}/img/lourencomarinho/hierarquia.png">


### Extension Time
Criando uma *extension* de [NSManagedObject](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/CoreDataFramework/Classes/NSManagedObject_Class/), podemos nos "livrar" de *Strings* com nomes de classe quando criamos um *NSFetchRequest*. Para isso, vamos adicionar a variável estática `className` que retorna o nome da classe.

~~~ swift
extension NSManagedObject {
    static var className: String {
        return String(self)
    }
}
~~~

Agora vamos adicionar a implementação padrão para o protocolo `CoreDataModel`. A primeira **Protocol Extension** que vamos criar vai definir qual tipo de contexto vamos usar e como ele será obtido. Extensões podem ser adicionadas em qualquer arquivo `.swift` do projeto.

~~~ swift
extension CoreDataModel where Self: NSManagedObject {
    static var context: NSManagedObjectContext {
        let appDelegate = UIApplication.sharedApplication().delegate as! AppDelegate
        return appDelegate.managedObjectContext
    }
}
~~~

Vamos começar simples com uma extensão que implementa o método `save()` do protocolo `Writeable`. Esta implementação é bem fácil - tudo que ela faz é acessar a variável estática **context** que definimos anteriormente e invocar o método `save()` de `NSManagedObjectContext`. 

~~~ swift
extension CoreDataModel where Self: NSManagedObject {
    func save() {
        try! Self.context.save()
    }
}
~~~

Nossa segunda extensão também é bastante simples. Vamos implementar agora o método`destroy()` do protocolo `Deletable`. Como fizemos anteriormente, vamos acessar a variável **context** e invocar o método  `deleteObject(_:)` e salvar a alteração.

~~~ swift
extension CoreDataModel where Self: NSManagedObject {
    func destroy() {
        Self.context.deleteObject(self)
        try! Self.context.save()
    }
}
~~~

 No método `destroy()` também vemos **self**, dessa vez com letra minúscula, representando a instância do objeto que será deletado da base de dados.

Agora vamos criar a extensão para os métodos do protocolo `Readable` que adiciona a implementação de `all()` e `find(predicate:)`. Nessa extensão, usamos **Self** em dois casos distintos: para acessar a variável `className` e posteriormente fazer o *casting* para o tipo do objeto que está sendo pesquisado.

~~~ swift
extension CoreDataModel where Self: NSManagedObject {
    static func all() -> [Self] {
        let fetchRequest = NSFetchRequest(entityName: Self.className)
        fetchRequest.predicate = NSPredicate(value: true)
        
        return try! context.executeFetchRequest(fetchRequest) as! [Self]
    }
    
    static func find(predicate: NSPredicate) -> [Self] {
        let fetchRequest = NSFetchRequest(entityName: Self.className)
        fetchRequest.predicate = predicate
        
        return try! context.executeFetchRequest(fetchRequest) as! [Self]
    }
}
~~~

Agora basta que nosso tipo adote o protocolo `CoreDataModel`. Assim temos todos os métodos definidos em nossos protocolos, para qualquer tipo que adote `CoreDataModel`. E o melhor de tudo, agora temos uma forma de acessar nossos dados de forma genérica, tipada e sem literais espalhados pelo código.

~~~ swift
final class Person: NSManagedObject, CoreDataModel {
    @NSManaged var name: String
}
~~~

~~~ swift
let people = Person.all()
print(results.dynamicType) // Array<Person>
~~~

~~~ swift
let namePredicate = NSPredicate(format: "name CONTAINS[cd] %@", "João")
let people = Person.find(namePredicate)

if let person = people.first {
	print(person.name)
}
~~~

Isso é tudo pessoal! Espero ter ajudado nossa comunidade a crescer um pouco mais, mostrando como Protocol-Oriented Programming pode melhorar nossas vidas no dia-a-dia. Também criei um exemplo que pode ser acessado [aqui](https://github.com/lourenco-marinho/CoreDataModel).

---
Podemos conversar mais aqui [@lopima](https://twitter.com/lopima) e [aqui](http://iosdevbr.herokuapp.com)!




