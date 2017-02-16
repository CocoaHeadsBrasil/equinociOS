---
layout:     post
title:      "Começando com VIPER"
subtitle:   "Entendendo e aplicando sem muito sofrimento"
date:       2017-03-14 00:00:00
author:     "Vitor Santos Mesquita"
header-img: "img/agradecimento/sunset-header.jpg"
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

---

# Entendido Clean vamos entender o VIPER.

Cada letra do nome representa um tipo de objeto/classe que iremos ter são:

- View
- Interactor
- Presenter
- Entity
- Router

### Formando V.I.P.E.R!!

<!--TODO IMAGEM VIPER-->

##View
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

##Presenter
Representa toda a lógica dá view.<br>
Ele recebe eventos da View e reage a eles.<br>
Ele recebe dados do <b>Interactor</b>, aplica a lógica da View e manda mostrar o que for preciso.

##Interactor
Representa um caso de uso.
Ele que é responsável por aplicar a lógica de determinado caso.

##Entity
Representa a entidade da API e/ou do Banco

##Router
Representa toda a lógica de navegação da sua aplicação.
Ele que sabe para qual tela tem que ir e para qual tem que voltar.
É um objeto composto por 2 (Presenter e WireFrame).
O WireFrame é encarregado de instanciar todas as dependências necessárias. E ele que também é encarregado de chamar outro WireFrame para assim instanciar sua tale de dependências.