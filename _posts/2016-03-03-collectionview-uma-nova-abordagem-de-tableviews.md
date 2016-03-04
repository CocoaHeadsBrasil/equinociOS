---
layout:     post
title:      "UICollectionView - Uma nova abordagem de TableView?"
subtitle:   "Por onde começar a utilizar collectionview"
date:       2016-03-03 00:00:00
author:     "Vinicius Carvalho"
header-img: "img/viniciuscarvalho/background-header.jpg"
category:   CollectionView
---

> Olá pessoal, essa é minha primeira postagem aqui no EquinociOS e então farei uma breve apresentação. Me chamo [Vinicius Carvalho](www.twitter.com/viniciusc70), sou desenvolvedor mobile há pouco mais de três anos e já trabalhei com backend usando Ruby on Rails. Feita essa introdução, vamos ao que interessa: UICollectionView!

Vamos estruturar o artigo em quatro partes:

* O que é uma CollectionView;
* Como funciona;
* Elementos de uma CollectionView;
* Utilização em diversos aplicativos;

Este post contém uma base para você começar a utilizar `UICollectionView` em suas aplicações. Não é um guideline, pois a Apple tem um muito bom [Documentação CollectionView](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UICollectionView_class/). Vou apenas tentar dar um norte com as dificuldades que tive recentemente.

`UICollectionView` foi introduzido no iOS 6 e, ao longo desse tempo, tornou-se um dos elementos de UI mais apreciados e utilizados por desenvolvedores iOS. Sua característica básica é de um layout em formato de Grid, o que se parece muito com o elemento `GridView` do Android.

![]({{ site.baseurl }}/img/viniciuscarvalho/layout-flow-cv.png)

A gama de aplicações feitas com CollectionViews é vasta e podemos observar nos próprios aplicativos da Apple como calendário, a câmera, música, a própria AppleStore, entre outros.

Como na `UITableView`, a `UICollectionView` é uma subclasse de `UIScrollView` que gerencia todos os artigos e elementos encomendados. Os itens são geridos por um *data source*, que fornece uma *collection view cell*, que representa um elemento em um determinado *index path*.

Ao contrário da `UITableView`, a `UICollectionView` não está restrita a um layout vertical, de coluna única. Ao invés disso, a collection view tem um *layout object*, que determina a posição de cada *subview*, similar ao *data source* em alguns aspectos.

Veremos como o `UICollectionViewDataSource` (número de itens, seções, cells, ...) e `UICollectionViewDelegate` são eficientes, a separação do layout da camada de model, que é a grande diferença para as `UITableView`s, onde não possui esses layout objects.
Além disso tudo, a customização das células é um ponto importante com as subclasses de `UICollectionViewCell`. Possuem diferentes métodos de seleção das células e diferentes modos de animação.
Veremos quase tudo sobre `UICollectionView`. O nosso amigo Rodrigo Borges também vai explorar mais esse elemento aqui no blog e nos trazer muito conhecimento.

### Como Funciona a Collection View

Como mostrado anteriormente, podemos ter o layout simples dividido em colunas e mostrado como default ou podemos explorar a flexibilidade e criar um layout customizado da collection view. Vamos abordar um pouco dessas customizações:
Vou criar um projeto para ilustrar bem essas funcionalidades.

Ao criarmos o projeto temos a seguinte estrutura de arquivos,

<center><img src="/img/viniciuscarvalho/imagem-inicial.png" alt="" /></center>

Foi criado um novo `Controller`, `HomeViewController`, para receber as instruções da nossa CollectionView. Feito isso vamos trabalhar um pouco no nosso Storyboard.

Nossa Storyboard está estruturada da seguinte forma:

![]({{ site.baseurl }}/img/viniciuscarvalho/storyboard-inicial.png)

Explicando melhor a figura acima, primeiro criamos um View Controller e colocamos uma `View`. Como imagem de fundo, foi inserido um `UIImageView` para ocupar toda a tela, com as Constraints de 0,0,0,0.
Em seguida foi adicionado um Visual Effect with Blur em cima da imagem, com a aparência Light. Por cima de tudo isso foi finalmente adicionado o nosso CollectionView. A View terá o tamanho de 600 Width e 480 Height, o tamanho da `cell` será de 200 Width e 380 Height, Min Spacing de 10 For Cells e 20 For Lines, Section Insets 20 para esquerda e direita.

Normalmente o scroll das CollectionViews é para baixo. Vamos fazer diferente e aproveitar tudo de customização e fazer o scroll lateral.

<center><img src="/img/viniciuscarvalho/direcao-scroll.png" alt="" /></center>

Feito isso, vamos para a customização do interior da nossa célula. Vamos adicionar uma imagem dentro dela e seus Constraints 0,0,0,0. 

![]({{ site.baseurl }}/img/viniciuscarvalho/nova-customizacao.png)

Depois, vamos adicionar outro efeito Blur no canto inferior da `ImageView` e adicionar seus Constraints também.

<center><img src="/img/viniciuscarvalho/constraints-segunda-img.png" alt="" /></center>

Logo em seguida, adicionamos uma Label dentro do Visual Effect Blur e seus respectivos paddings nos quatro cantos: superior, trailing à direita/esquerda e bottom.
Um fato extremamente importante é não esquecer de ligar os Outlets da CollectionView ao nosso `HomeViewController` ou referenciá-los via código no nosso `viewDidLoad()`. Atenção a este ponto.

Feito tudo isso, vamos finalmente para o código. UFA!

Primeiro vamos abordar nosso `HomeViewController`,

```swift
import UIKit

class HomeViewController: UIViewController, UICollectionViewDataSource, UICollectionViewDelegate { 
    
    //MARK: - IBOutlets
    
    @IBOutlet weak var backgroundImage: UIImageView!
    @IBOutlet weak var collectionView: UICollectionView!
   
    //MARK: - UICollectionViewDataSource
    
    private var interests = Interest.createInterests()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
    }
    
    //MARK: - CollectionView
    
    func numberOfSectionsInCollectionView(collectionView: UICollectionView) -> Int {
        return 1
    }
    
    func collectionView(collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
        return interests.count
    }

    func collectionView(collectionView: UICollectionView, cellForItemAtIndexPath indexPath: NSIndexPath) -> UICollectionViewCell {
        
        let cellIdentifier = "InterestCell"
        
        let cell = collectionView.dequeueReusableCellWithReuseIdentifier("InterestCell", forIndexPath: indexPath) as! InterestCollectionViewCell
        
        cell.interest = self.interests[indexPath.item]
        
        return cell
    }
}

```

Veja que na `CollectionView` também temos elementos semelhantes do `UITableView`, como `numberOfSections`, `numberOfItems` e o tratamento da célula. Isso tudo facilita sua compreensão ao trabalhar com `CollectionView`.

Foi criada a variável `interests`, para pegar os dados e popular nossas células. Além disso, não podemos esquecer de referenciar a sua célula que esteja trabalhando. No nosso caso é o `InterestCell`.

Para popularmos nossas células, foi criado o Model `Interest`:

```swift
class Interest {

    var title = ""
    var description = ""
    var numberOfMembers = 0
    var numberOfPosts = 0
    var featuredImage: UIImage!
    
    init(title: String, description: String, featuredImage: UIImage!) {
        self.title = title
        self.description = description
        self.featuredImage = featuredImage
        numberOfMembers = 1
        numberOfPosts = 1
    }
    
    static func createInterests() -> [Interest] {
        
        return [
            Interest(title: "Quero viajar o mundo inteiro", description: "Estamos planejando fazer uma eurotrip nas férias deste ano, vamos passar por todo o leste europeu", featuredImage:UIImage(named: "img2")!),
            Interest(title: "Estou trabalhando muito!", description: "Passei esta semana trabalhando mais de oito horas por dia, foi extremamente exaustivo. Que semana!", featuredImage: UIImage(named: "img5")!),
            Interest(title: "Precisamos de desenvolvedores", description: "Estamos criando uma startup com um produto revolucionário e estamos contratando devs que queiram participar desta jornada, basta enviar o portifólio e pretensão salarial", featuredImage: UIImage(named: "img6")!),
            Interest(title: "Dev iOS", description: "Procuramos um desenvolvedor iOS sênior, com mais de 3 anos de experiência e seja muito f***", featuredImage: UIImage(named: "img4")!),
            Interest(title: "Quero me tornar um empreendedor", description: "No Brasil é bem difícil se tornar um empreendedor e abrir seu próprio negócio e por isso vamos te ajudar a desmitificar essa ideia", featuredImage: UIImage(named: "img3")!),
        
        ]
    }
}
```

E por último, na nossa aplicação, temos a `InterestCollectionViewCell`,

```swift
class InterestCollectionViewCell: UICollectionViewCell {
 
    var interest: Interest! {
        didSet {
            updateUI()
        }
    }
    
    //MARK: - Private
    @IBOutlet weak var featuredImageView: UIImageView!
    @IBOutlet weak var interestTitleLabel: UILabel!
    
    private func updateUI() {
        interestTitleLabel?.text! = interest.title
        featuredImageView?.image! = interest.featuredImage
    }
}
```
Veja que fizemos um tratamento do label e da image, mas caso não queira pode apenas adicionar as propriedades de `IBOutlet` normalmente.

### Elementos de uma CollectionView

Passado todo o nosso projeto, tem mais algumas coisas interessantes que podemos fazer com os *CustomLayouts* das `UICollectionView`s.
Caso você queira inserir um *fade-in* ou *fade-out* em uma célula, você pode customizar completamente o modo de inserção. Por exemplo, podemos inicializar o *alpha* como 0, que pode se encontrado na barra lateral da direita caso você esteja usando o Storyboard. Isso irá criar uma animação de *fade-in*. Colocando a translação e escala com os mesmo valores e ao mesmo tempo, deixará com que a célula aumente de tamanho, assim dando aquela sensação de que ela está vindo por cima da outra.
Este mesmo princípio é usado para deletar alguma célula. Agora a animação vai sair do estado normal para o final de acordo com o layout attributes que você irá implementar. Vamos listar alguns métodos que servem para sua customização:

*`initialLayoutAttributesForAppearingItemAtIndexPath:`
*`initialLayoutAttributesForAppearingSupplementaryElementOfKind:atIndexPath:`
*`initialLayoutAttributesForAppearingDecorationElementOfKind:atIndexPath:`
*`finalLayoutAttributesForDisappearingItemAtIndexPath:`
*`finalLayoutAttributesForDisappearingSupplementaryElementOfKind:atIndexPath:`
*`finalLayoutAttributesForDisappearingDecorationElementOfKind:atIndexPath:`

#### Trocando de layouts

Para fazer a troca animada de layouts entre collection view, teremos mais ou menos o mesmo conceito anterior de *fade in* e *fade out*. Quando chamamos o método `setCollectionViewLayout:animated`, ele irá consultar o novo layout e suas células, fazendo assim a animação em cada célula do seu antigo para o novo layout. Você não precisa fazer mais nada. Bem legal, né?

### Conclusões

Bem falado isso tudo, tentei abordar um pouco de como comecei com as `CollectionView` e o que podemos aproveitar desta classe. O nosso amigo [Rodrigo Borges](http://www.twitter.com/rdgborges) vai abordar este assunto mais afundo, então aqui foi apenas uma introdução :)
O link do projeto completo está no meu [github](www.github.com/viniciuscarvalho), então se quiser discutir alguma alteração ou propor alguma mudança, será sempre bem-vindo.

 
Vou deixar alguns links interessantes e que me ajudaram bastante. Um deles foi o [Invariante](http://invariante.com) dos amigos [Diogo Tridapalli](http://www.twitter.com/diogot) e [Bruno Koga](http://www.twitter.com/brunokoga) que tem um artigo interessante também falando sobre uma implementação de `UICollectionView` e outros mais. Vale a pena dar uma conferida.

Um abraço e até a próxima!

Links interessantes:

[Talk WWDC'15](https://developer.apple.com/videos/play/wwdc2015-225/)

[Ray Wenderlich - tutorial CollectionView Pinterest](http://www.raywenderlich.com/107439/uicollectionview-custom-layout-tutorial-pinterest)

[Tutorial Ray Wenderlich](https://www.youtube.com/watch?v=_490cFFGDjw)

[Jared Davidson](https://www.youtube.com/watch?v=cCAfeTqsE_Y)

[objc-io](https://www.objc.io/issues/3-views/collection-view-layouts/#animation)

[Ash Furrow](https://ashfurrow.com/uicollectionview-the-complete-guide/)
