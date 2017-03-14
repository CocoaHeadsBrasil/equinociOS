---
layout:     post
title:      "Começando com VIPER"
subtitle:   "Entendendo e aplicando sem muito sofrimento"
date:       2017-03-14 00:00:00
author:     "Vitor Santos Mesquita"
header-img: "img/vitormesquita/viper-monster.jpg"
category:   viper
---

Antes de começarmos gostaria de dizer que sou um entusiasta sobre VIPER e o post é contando um pouco sobre o que sei e entendo e a minha experiência sobre o assunto.

---

# Intro

VIPER já é uma arquitetura bastante conhecida e utilizada pela comunidade iOS.
E para quem está começando a pesquisar alguma arquitetura além do famoso MVC pode se surpreender com ela.

Eu comecei a pesquisar sobre VIPER pois tinha algumas grandes dificuldades sobre a arquitetura MVC como: 

- Desacoplamento de código 
- Classes muito extensas 
- Classes muito complexas 
- Dificuldade na hora de dar manutenção no código
- Dificuldade em testar

Se você tem algumas desses dificuldades, seja com seu código ou não, VIPER pode te ajudar a soluciona-las.
Mas lembrando que ela não irá operar milagres no seu código, na meu caso me ajudou a pensar mais, arquitetar mais, desacoplar mais e assim aumentar meu nível de código.

---

VIPER é baseada na arquitetura Clean disseminada pelo famoso “tio” Bob (Uncle Bob), que por sua vez é baseada na arquitetura de cebola (Onion architecture) com junção de mais outras 3, onde separa sua aplicação em camadas distintas que são: 

## Camada de apresentação (presentation)

É a camada mais superficial dá sua aplicação. Onde o usuário irá interagir, no caso do iOS seria nossas Views, ViewController, TabBarViewContoller …

É a camada responsável por dar “cara” na nossa aplicação, montar tela, mudar cor, mostrar ou esconder elementos entre outras várias coisas relacionadas.

## Camada de domínio (domain)

É a camada do coração dá sua aplicação, pois ela é responsável por conter todas as regras de negócios e casos de uso dá sua aplicação.

## Camada de dados (data)

É a camada responsável por conversar com o externo dá sua aplicação. Seja API ou Banco de dados, ela que irá manusear toda essa interação com o externo.

<img src="{{ site.baseurl }}/img/vitormesquita/clean.png">

---

# Entendido Clean vamos entender o VIPER.

Cada letra do nome representa um tipo de objeto/classe que iremos ter são:

- View
- Interactor
- Presenter
- Entity
- Router

### Formando V.I.P.E.R!!

<img src="{{ site.baseurl }}/img/vitormesquita/Viper-Module.png">

## View
Representa tudo onde o usuário irá interagir.<br>
Sua responsabilidade é mostrar o que o <b>Presenter</b> manda ela mostrar, e lidar com interações do usuario com a tela passando-as para o Presenter.

```swift
import UIKit

/*Protocolo que define as interações Presenter -> View */
protocol ViewProtocol: class {
    func setVisibilityActivityIndicator(isVisible: Bool)
}

class ViewController: UIViewController {
    
    var presenterProtocol: PresenterProtocol!
    
    @IBOutlet weak var activityIndicator: UIActivityIndicatorView!
    @IBOutlet weak var tableView: UITableView!
    
    let dispose = DisposeBag()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        presenterProtocol.viewDidLoad()
        bind()
    }
    
    private func bind() {
        presenterProtocol.repositoriesChanged.bindNext {[weak self] (_) in
            guard let strongSelf = self else {return}
            strongSelf.tableView.reloadData()
        }
        .addDisposableTo(dispose)
    }
}

extension ViewController: ViewProtocol {
    
    func setVisibilityActivityIndicator(isVisible: Bool) {
        activityIndicator.isHidden = !isVisible
        if isVisible {
            activityIndicator.startAnimating()
        } else {
            activityIndicator.stopAnimating()
        }
    }
}
```

## Presenter
Representa toda a lógica dá view.<br>
Ele recebe eventos da View e reage a eles.<br>
Ele recebe dados do <b>Interactor</b>, aplica a lógica da View e manda mostrar o que for preciso.

```swift
import UIKit
import RxSwift
import RxCocoa

/* Protocolo que define as interações View -> Presenter */
protocol PresenterProtocol {
    var numberOfRows: Int {get}
    var repositoriesChanged: Observable<Void> {get}
    
    func viewDidLoad()
    func getItem(indexPath: IndexPath) -> Repository?
}

class Presenter {
    
    weak var viewProtocol: ViewProtocol?
    var interactorProtocol: RepositoryInput!
    
    var repositories = Variable<[Repository]>([])
}

extension Presenter: PresenterProtocol {
    
    var numberOfRows: Int {
        return self.repositories.value.count
    }
    
    var repositoriesChanged: Observable<Void> {
        return repositories.asObservable().map({ (_) -> Void in
            
        })
    }
    
    func viewDidLoad() {
		viewProtocol?.setVisibilityActivityIndicator(isVisible: true)
		interactorProtocol.searchRepositories(query: "alguma lib")
    }
    
    func getItem(indexPath: IndexPath) -> Repository? {
        return repositories.value[indexPath.row]
    }
}

/* Protocolo de output do interactor */
extension Presenter: RepositoryOutput {
    
    func foundRepositories(repositories: [Repository]?) {
        guard let repositories = repositories else {return}
        viewProtocol?.setVisibilityActivityIndicator(isVisible: false)
        self.repositories.value = repositories
    }
}
```

## Interactor
Representa um caso de uso.<br>
Ele que é responsável por aplicar a lógica de determinado caso.

```swift 
import UIKit
import RxSwift

/* Protocolo que define as interações Presenter -> Interactor */
protocol RepositoryInput {
    func searchRepositories(query: String)
}

/* Protocolo que define as interações Interactor -> Presenter */
protocol RepositoryOutput: class {
    func foundRepositories(repositories:[Repository]?)
}

class RepositoryInteractor {

    weak var output: RepositoryOutput?
    let dispose = DisposeBag()
}

extension RepositoryInteractor: RepositoryInput {
    
    func searchRepositories(query: String) {
        APIClient
            .getRepositories(byName: query)
            .bindNext {[weak self] (repositories) in
                guard let strongSelf = self else { return }
                strongSelf.output?.foundRepositories(repositories: repositories)
            }
            .addDisposableTo(dispose)
    }
}
```

## Entity
Representa a entidade da API e/ou do Banco

```swift 
import ObjectMapper

/* Repositorio pego da API do Github */
class Repository: Mappable {
    
    var id: Int?
    var name: String?
    var htmlURLString: String?
    var description: String?
    var starsCount: Int?
    var language: String?
    var forksCount: Int?
    
    required init?(map: Map) {}
    
    func mapping(map: Map) {
        id <- map["id"]
        name <- map["name"]
        htmlURLString <- map["html_url"]
        description <- map["description"]
        starsCount <- map["stargazers_count"]
        language <- map["language"]
        forksCount <- map["forks_count"]
    }
}
```

## Router
Representa toda a lógica de navegação da sua aplicação.<br>
Ele que sabe para qual tela tem que ir e para qual tem que voltar.<br>
<b>É um objeto composto por 2 (Presenter e WireFrame).</b>

#### Wireframe
É encarregado de instanciar todas as dependências necessárias.<br>
E ele que também é encarregado de chamar outro WireFrame para assim instanciar sua tala/feature.

```swift
import UIKit

/**/
class WireFrame: NSObject {
    
    let viewController = ViewController(nibName:"ViewController", bundle: nil)
    let presenter = Presenter()
    let interactor = RepositoryInteractor()

    override init() {
        super.init()
        viewController.presenterProtocol = presenter
        presenter.viewProtocol = viewController
        presenter.interactorProtocol = interactor
        interactor.output = presenter
    }
    
    func present(window: UIWindow) {
        window.rootViewController = viewController
    }
	
	func showRepositoryDetails(repository: Repository) {
		let wireFrameDetails = RepoDetailsWireFrame(repository: repository)
		wireFrameDetails.present(withVC: viewController)
	}
}
```
 
#### Presenter
Nesse caso é encarregado de falar para o WireFrame fazer algum tipo de navegação, seja para outra tela, como desistanciar a tela corrente. 

---

# Dicas 

Quando comecei a implementar essa arquitetura tive alguns problemas que deixaram o desenvolvimento um pouco mais complicado, mas com essas dicas pode ficar bem mais simples desenvolver.

Como sabemos no desenvolvimento iOS sempre temos que cuidar a memoria que nossa aplicação consome (famoso Memory Leak). E com o VIPER por ter muitas dependências e dependências ciclicas, tive muito problema em desalocar meu ViewController e suas dependências.

### Com a `weak var` esse problema pode ser facilmente resolvido. 

Colocando ponteiros fracos nas dependências ciclicas como segue o diagrama:

<img src="{{ site.baseurl }}/img/vitormesquita/weak-diagram.png">

Um simples ponteiro fraco causa desalocamento em cadeia e assim não tendo que se preocupar em ficar desalocando dependencia por dependencia na mão.

Mas para criar ponteiros fracos de `protocol` é necessário falar para o protocolo que somente classes podem implementa-las.

```swift
protocol ViewProtocol: class {
}
```
--

Para quem está começando com VIPER pode ser meio confuso abstrair o que cada parte deve fazer e/ou implementar. Para isso comecei a separar as camadas do clean em pacotes, como:

<img src="{{ site.baseurl }}/img/vitormesquita/folders.png">

### Presentation 

Nesse pacote deverá conter as dependências `view`, `presenter` e `router (wireFrame)` separados por `features`.

<img src="{{ site.baseurl }}/img/vitormesquita/presentation.png">

### Domain

Nesse pacote deverá conter o `interactor`.

<img src="{{ site.baseurl }}/img/vitormesquita/domain.png">

### Data

Nesse pacote deverá conter as `entities` e todos as implementações de `APIClient` ou `DataBase`

<img src="{{ site.baseurl }}/img/vitormesquita/data.png">


<b>OBS: projeto exemplo que está no meu github</b> 
--

## E agora?

Muitas pessoas falam que é muito `overhead` fazer um software em VIPER, e de fato digo é mesmo, se sua aplicação é pequena.

Mas antes de julgar ou adotar o VIPER, analise bem seu projeto, verique se ele é um projeto que será escalavél e com varias funcionalidades. Se for o VIPER poderá ser uma arma muito útil que irá facilitar sua vida. Se não, viper poderá atrapalhar sua vida e pode acabar em um `spaghetti code` o seu projeto. 

### VIPER exige um empenho muito grande de quem está programando para se entender e se aplicar de forma correta.

Abaixo deixarem uns links para ajudar você que está começando em VIPER a se orientar e praticar.

- [https://www.objc.io/issues/13-architecture/viper/](https://www.objc.io/issues/13-architecture/viper/)
- [https://www.ckl.io/blog/ios-project-architecture-using-viper/](https://www.ckl.io/blog/ios-project-architecture-using-viper/)
- [https://github.com/vitormesquita/Cocoaheds](https://github.com/vitormesquita/Cocoaheds)

E sempre que precisar de alguma ajuda sobre o assunto pode me contactar no iOSDevBR <b>@vitormes</b>.

Obrigado e espero que gostem!
