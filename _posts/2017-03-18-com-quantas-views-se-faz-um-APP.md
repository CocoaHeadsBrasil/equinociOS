---
layout:     post
title:      "Com quantas views se faz uma APP?"
subtitle:   "Com view code e criatividade esse número pode ser menor do que você pensa"
date:       2017-03-18 00:13:00
author:     "Ronan Rodrigo Nunes"
header-img: "img/ronanrodrigo/building-boat.jpg"
category:   view-code
---

> Me chamo Ronan Rodrigo Nunes sou catarinense, não sulista e já usei Adobe Flex. Comecei minha carreira fazendo algumas gambiarras no Java, embarquei no trem do Ruby, desembarquei em Python, peguei conexão para Xamarin/C# e agora estou vivendo emoções com Swift! Sou desenvoledor iOS na ([Concrete Solutions](http://www.concretesolutions.com.br){:target="_blank"}, estou aprendendo a andar de skate, escrevo na revista [Cocoa Academy](http://medium.com/cocoaacademymag){:target="_blank"} e sou companheiro e marido de Nayara que me fez gostar de psicanálise e filosofia.

# Storyboards? XIBs? View Code?

Tanto meus primeiros passos dentro do Xcode quanto alguns trabalhos profissionais foram feitos com o Storyboard. Eu não conseguia imaginar como alguém em sã consciência poderia substituir o arraste e solte do Interface Builder por código. Afinal, é muito simples.

Até que os Storyboards começaram a me incomodar e em seguida os XIBs também. Antes de preferir views construídas com código, eu era muito resistente à esta mudança. Mas um dia comecei a refletir e concluí que isso era preguiça da minha parte. Afinal, quando fullstack eu escrevia HTML ao invés de arrastar e soltar componentes. Mas ainda assim, o Visual Format language era algo estranho.

Então participei de algumas talks do Thiago Lioy [@tpLioy](http://twitter.com/tpLioy){:target="_blank"} sobre view code. Nessas talks ele mostrou como [migrou um projeto de Storyboard para views criadas com código](https://medium.com/cocoaacademymag/migrating-an-app-to-view-code-ffe3f1510408#.523t9mw60){:target="_blank"} utilizando Snapkit. E o resultado, para minha surpresa, foi bonito, simples e fácil. Foi assim que comecei a gostar de view code, obrigado [@tpLioy](http://twitter.com/tpLioy){:target="_blank"}. Mas ainda queria aprender a utilizar o que a Apple fez pra nós. Foi então que conheci o Anchor Layout.

# NSLayoutAnchor

A documentação da Apple descreve `NSLayoutAnchor` como sendo uma fábrica de classe, destinada a gerar objetos `NSLayoutConstraint` através de uma API mais expressiva. Sendo os objetos do tipo `NSLayoutConstraint` os responsáveis por possibilitar o Auto Layout.

Resumindo, a classe `NSLayoutAnchor` facilitará a criação de constraints utilizadas no auto layout. Ao invés criar as constraints utilizando o construtor da classe `NSLayoutConstraint` são utilizados métodos das `UIView`s. A vantagem consistirá em um código mais limpo e de brinde uma checagem de tipo para evitar a criação de constraints inválidas. Essa checagem de tipo não permitirá, por exemplo, misturar atributos do eixo X com atributos do eixo Y.

> Ainda que a classe `NSLayoutAnchor` tenha a checagem adicional de tipo, ainda é possível criar constraints inválidas. Por exemplo, o compilador permite que você configure a constraint `leadingAnchor` de uma view com a `leftAnchor` de outra view, pois ambas são instâncias de `NSLayoutXAxisAnchor`. Entretanto, Auto Layout não permite constraints que misturem atributos de `leading` e `trailing` com atributos de `left` ou `right`. Como resultado, as constraints vão quebrar em tempo de execução. [NSLayoutAnchor](https://developer.apple.com/reference/uikit/nslayoutanchor){:target="_blank"}

# Hora da aventura

Para demonstrar como utilizar o auto layout na prática, eu criei um [Playground](https://github.com/ronanrodrigo/autolayout){:target="_blank"}. Nesse playground existirá uma lista de cartões de visita. Onde os cartões têm uma exibição diferenciada conforme o plano que o usuário anunciante pagou. Com isso em mente, criei o layout abaixo.

<p align="center"><img style="border-radius: 3px;"  src="{{ site.baseurl }}/img/ronanrodrigo/business-cards-list.png"></p>

## A raiz de tudo

Como podem observar, todas as células da tabela demonstrada no protótipo possuem os mesmos dados. Mas, diferentemente organizados em cada célula. Se você já lidou com front-end Web, é como se tivéssemos um único HTML, com um CSS diferente para cada layout. Sendo assim, vou criar uma classe contendo todos os elementos do componente de cartão.

~~~swift
import UIKit

final class BusinessCardComponents {

    let photoImageView: UIImageView = {

        // Aqui não vamos nos preocupar com tamanhos.
        // Isso vai ficar por conta da "folha de estilos".
        let imageView = UIImageView(frame: .zero)

        // Se desejamos calcular dinamicamente os tamanhos
        // e posições das views, devemos alterar o valor dessa propriedade
        // para false. Por default, toda view criada via código
        // possui esse valor como true. Se criada via Interface Builder
        // o valor default é false.
        imageView.translatesAutoresizingMaskIntoConstraints = false

        return imageView
    }()

    let nameLabel: UILabel = {
        let label = UILabel(frame: .zero)
        label.translatesAutoresizingMaskIntoConstraints = false
        return label
    }()

    let titleLabel: UILabel = {
        let label = UILabel(frame: .zero)
        label.translatesAutoresizingMaskIntoConstraints = false
        return label
    }()

    let emailLabel: UILabel = {
        let label = UILabel(frame: .zero)
        label.translatesAutoresizingMaskIntoConstraints = false
        return label
    }()

    let phoneLabel: UILabel = {
        let label = UILabel(frame: .zero)
        label.translatesAutoresizingMaskIntoConstraints = false
        return label
    }()

}
~~~

Neste momento vamos ter código repetindo. Mas, calma, no final do post esse código vai estar refatorado e com as repetições eliminadas. E essa é uma das vantagens quando construímos views com código, temos liberdade para colocar em prática nossa criatividade e evoluir o que está sendo feito.

Pra fazer o layout do card referente ao plano Senior utilizarei o design pattern  Dependency Injection. O objeto que será injetado vai ser um `BusinessCardComponents` que vai receber instruções de apresentação na `SeniorBusinessCardView`.

~~~swift
final class SeniorBusinessCardView: UIView {

    private let businessCardComponents: BusinessCardComponents
    private let imageHeight: CGFloat = 100

    init(businessCardComponents: BusinessCardComponents) {
        self.businessCardComponents = businessCardComponents
        super.init(frame: .zero)
        setupViewHierarchy()
        setupConstraints()
    }

    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    private func setupViewHierarchy() {
        addSubview(businessCardComponents.photoImageView)
        addSubview(businessCardComponents.nameLabel)
        addSubview(businessCardComponents.titleLabel)
        addSubview(businessCardComponents.phoneLabel)
        addSubview(businessCardComponents.emailLabel)
    }

    private func setupConstraints() {
        businessCardComponents.photoImageView.topAnchor.constraint(equalTo: topAnchor).isActive = true
        businessCardComponents.photoImageView.leadingAnchor.constraint(equalTo: leadingAnchor).isActive = true
        businessCardComponents.photoImageView.trailingAnchor.constraint(equalTo: trailingAnchor).isActive = true
        businessCardComponents.photoImageView.heightAnchor.constraint(equalToConstant: imageHeight).isActive = true

        businessCardComponents.nameLabel.topAnchor.constraint(equalTo: businessCardComponents.photoImageView.bottomAnchor).isActive = true
        businessCardComponents.nameLabel.leadingAnchor.constraint(equalTo: leadingAnchor).isActive = true
        businessCardComponents.nameLabel.trailingAnchor.constraint(equalTo: trailingAnchor).isActive = true

        businessCardComponents.titleLabel.topAnchor.constraint(equalTo: businessCardComponents.nameLabel.bottomAnchor).isActive = true
        businessCardComponents.titleLabel.leadingAnchor.constraint(equalTo: leadingAnchor).isActive = true
        businessCardComponents.titleLabel.trailingAnchor.constraint(equalTo: trailingAnchor).isActive = true

        businessCardComponents.phoneLabel.topAnchor.constraint(equalTo: businessCardComponents.titleLabel.bottomAnchor).isActive = true
        businessCardComponents.phoneLabel.leadingAnchor.constraint(equalTo: leadingAnchor).isActive = true
        businessCardComponents.phoneLabel.trailingAnchor.constraint(equalTo: trailingAnchor).isActive = true

        businessCardComponents.emailLabel.topAnchor.constraint(equalTo: businessCardComponents.phoneLabel.bottomAnchor).isActive = true
        businessCardComponents.emailLabel.leadingAnchor.constraint(equalTo: leadingAnchor).isActive = true
        businessCardComponents.emailLabel.trailingAnchor.constraint(equalTo: trailingAnchor).isActive = true
        businessCardComponents.emailLabel.bottomAnchor.constraint(equalTo: bottomAnchor).isActive = true
    }
}
~~~

Embora sejam poucas constraints, o código ficou um tanto quanto extenso. Por conta da dor de ter que fazer esse código várias vezes criei uma extension. Abaixo, mostro como ficou a extension e como ficou o código depois.

~~~swift
extension UIView {

    @discardableResult func topAnchor(equalTo anchor: NSLayoutYAxisAnchor, constant: CGFloat = 0) -> Self {
        topAnchor.constraint(equalTo: anchor, constant: constant).isActive = true
        return self
    }

    @discardableResult func bottomAnchor(equalTo anchor: NSLayoutYAxisAnchor, constant: CGFloat = 0) -> Self {
        bottomAnchor.constraint(equalTo: anchor, constant: constant).isActive = true
        return self
    }

    @discardableResult func leadingAnchor(equalTo anchor: NSLayoutXAxisAnchor, constant: CGFloat = 0) -> Self {
        leadingAnchor.constraint(equalTo: anchor, constant: constant).isActive = true
        return self
    }

    @discardableResult func trailingAnchor(equalTo anchor: NSLayoutXAxisAnchor, constant: CGFloat = 0) -> Self {
        trailingAnchor.constraint(equalTo: anchor, constant: constant).isActive = true
        return self
    }

    @discardableResult func heightAnchor(equalTo height: CGFloat) -> Self {
        heightAnchor.constraint(equalToConstant: height).isActive = true
        return self
    }

    @discardableResult func widthAnchor(equalTo height: CGFloat) -> Self {
        widthAnchor.constraint(equalToConstant: height).isActive = true
        return self
    }

}
~~~

~~~swift
final class SeniorBusinessCardView: UIView {

    private let defaultMargin: CGFloat = 5

    // ...

    private func setupConstraints() {
        businessCardComponents.photoImageView
            .topAnchor(equalTo: topAnchor)
            .leadingAnchor(equalTo: leadingAnchor)
            .trailingAnchor(equalTo: trailingAnchor)
            .heightAnchor(equalTo: imageHeight)

        businessCardComponents.nameLabel
            .topAnchor(equalTo: businessCardComponents.photoImageView.bottomAnchor, constant: defaultMargin)
            .leadingAnchor(equalTo: leadingAnchor, constant: defaultMargin)
            .trailingAnchor(equalTo: trailingAnchor, constant: defaultMargin)

        businessCardComponents.titleLabel
            .topAnchor(equalTo: businessCardComponents.nameLabel.bottomAnchor, constant: defaultMargin)
            .leadingAnchor(equalTo: leadingAnchor, constant: defaultMargin)
            .trailingAnchor(equalTo: trailingAnchor, constant: defaultMargin)

        businessCardComponents.phoneLabel
            .topAnchor(equalTo: businessCardComponents.titleLabel.bottomAnchor, constant: defaultMargin)
            .leadingAnchor(equalTo: leadingAnchor, constant: defaultMargin)
            .trailingAnchor(equalTo: trailingAnchor, constant: defaultMargin)

        businessCardComponents.emailLabel
            .topAnchor(equalTo: businessCardComponents.phoneLabel.bottomAnchor, constant: defaultMargin)
            .bottomAnchor(equalTo: bottomAnchor)
            .leadingAnchor(equalTo: leadingAnchor, constant: defaultMargin)
            .trailingAnchor(equalTo: trailingAnchor, constant: defaultMargin)
    }
}
~~~

Ocupou mais linhas pois achei conveniente quebra-las. Mas ao todo foi uma redução de quase 800 caracteres. E para além da economia de caracteres, ficou um código mais fácil de ler. Continuando, a próxima tela é referente ao plano Full.

Agora, quando o nosso aplicativo exibir cartões de visita do plano Full, deve mostrar a imagem ao lado dos dados, ao invés de em cima. Para isso, criei uma view chamada `rightContentContainer` que vai conter o conteúdo à direita da imagem, ou seja, com seu `leading` igual ao `trailing` da imagem.

~~~swift
final class FullBusinessCardView: UIView {

    private let businessCardComponents: BusinessCardComponents
    private let imageSize: CGFloat = 80
    private let defaultMargin: CGFloat = 5

    private let rightContentContainer: UIView = {
        let view = UIView(frame: .zero)
        view.translatesAutoresizingMaskIntoConstraints = false
        return view
    }()

    init(businessCardComponents: BusinessCardComponents) {
        self.businessCardComponents = businessCardComponents
        super.init(frame: .zero)
        setupViewHierarchy()
        setupConstraints()
    }

    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    private func setupViewHierarchy() {
        addSubview(businessCardComponents.photoImageView)
        addSubview(rightContentContainer)
        rightContentContainer.addSubview(businessCardComponents.nameLabel)
        rightContentContainer.addSubview(businessCardComponents.titleLabel)
        rightContentContainer.addSubview(businessCardComponents.emailLabel)
        rightContentContainer.addSubview(businessCardComponents.phoneLabel)
    }

    private func setupConstraints() {
        businessCardComponents.photoImageView
            .topAnchor(equalTo: topAnchor)
            .leadingAnchor(equalTo: leadingAnchor, constant: defaultMargin)
            .widthAnchor(equalTo: imageSize)
            .heightAnchor(equalTo: imageSize)

        rightContentContainer
            .topAnchor(equalTo: businessCardComponents.photoImageView.topAnchor)
            .heightAnchor(equalTo: imageSize)
            .leadingAnchor(equalTo: businessCardComponents.photoImageView.trailingAnchor, constant: defaultMargin)
            .trailingAnchor(equalTo: trailingAnchor, constant: defaultMargin)

        businessCardComponents.nameLabel
            .topAnchor(equalTo: rightContentContainer.topAnchor)
            .leadingAnchor(equalTo: rightContentContainer.leadingAnchor)
            .trailingAnchor(equalTo: rightContentContainer.trailingAnchor)

        businessCardComponents.titleLabel
            .topAnchor(equalTo: businessCardComponents.nameLabel.bottomAnchor)
            .leadingAnchor(equalTo: rightContentContainer.leadingAnchor)
            .trailingAnchor(equalTo: rightContentContainer.trailingAnchor)

        businessCardComponents.phoneLabel
            .topAnchor(equalTo: businessCardComponents.titleLabel.bottomAnchor)
            .leadingAnchor(equalTo: rightContentContainer.leadingAnchor)
            .trailingAnchor(equalTo: rightContentContainer.trailingAnchor)

        businessCardComponents.emailLabel
            .topAnchor(equalTo: businessCardComponents.phoneLabel.bottomAnchor)
            .leadingAnchor(equalTo: rightContentContainer.leadingAnchor)
            .trailingAnchor(equalTo: rightContentContainer.trailingAnchor)
    }
}
~~~

Agora para fazer a view do plano Junior, vou sugerir uma forma diferente. Utilizando Stack Views ao invés do Anchor Layout. Particularmente, tanto com Interface Builder quanto através de código, faço o layout com Constraints e depois altero para usar Stack Views quando adequado.

## Stack Views

Como o nome sugere, é um objeto que vai empilhar views na horizontal ou vertical. Por conta dessa estrutura de pilha, evitamos a criação de algumas constraints. Tudo o que foi demonstrado anteriormente usando Anchor Layout pode ser simplificado ainda mais com o uso das Stack Views.

> A stack view está de acordo com Auto Layout (é baseada no layout de constraints do sistema) para organizar  e alinhar uma lista de views de acordo com sua especificação. Para tirar proveito da stack view, você precisa saber o básico sobre as contraints do Auto Layout, como descrito no [Guia do Auto Layout](https://developer.apple.com/library/etc/redirect/xcode/content/1189/documentation/UserExperience/Conceptual/AutolayoutPG/index.html#//apple_ref/doc/uid/TP40010853){:target="_blank"}. [NSStackView](https://developer.apple.com/reference/appkit/nsstackview){:target="_blank"}

O código da `JuniorBusinessCardView` ficou ainda mais simples e com menos linhas na parte que define as constraints. Além do posicionamento das views, também é possível alterar tamanho de fontes, cores e assim por diante, conforme código abaixo.

~~~swift
final class JuniorBusinessCardView: UIView {

    private let businessCardComponents: BusinessCardComponents
    private let imageSize: CGFloat = 50
    private let defaultMargin: CGFloat = 5
    private let defaultFontSise: CGFloat = 10

    private let stackContentContainer: UIStackView = {
        let stack = UIStackView(frame: .zero)
        stack.axis = .vertical
        stack.translatesAutoresizingMaskIntoConstraints = false
        return stack
    }()

    init(businessCardComponents: BusinessCardComponents) {
        self.businessCardComponents = businessCardComponents
        super.init(frame: .zero)
        setupViewHierarchy()
        setupConstraints()
        setupTextFonts()
    }

    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    private func setupViewHierarchy() {
        addSubview(businessCardComponents.photoImageView)
        addSubview(stackContentContainer)
        stackContentContainer.addArrangedSubview(businessCardComponents.nameLabel)
        stackContentContainer.addArrangedSubview(businessCardComponents.titleLabel)
        stackContentContainer.addArrangedSubview(businessCardComponents.emailLabel)
        stackContentContainer.addArrangedSubview(businessCardComponents.phoneLabel)
    }

    private func setupConstraints() {
        businessCardComponents.photoImageView
            .topAnchor(equalTo: topAnchor)
            .leadingAnchor(equalTo: leadingAnchor, constant: defaultMargin)
            .widthAnchor(equalTo: imageSize)
            .heightAnchor(equalTo: imageSize)

        stackContentContainer
            .topAnchor(equalTo: businessCardComponents.photoImageView.topAnchor)
            .leadingAnchor(equalTo: businessCardComponents.photoImageView.trailingAnchor, constant: defaultMargin)
            .trailingAnchor(equalTo: trailingAnchor, constant: defaultMargin)
            .bottomAnchor(equalTo: businessCardComponents.photoImageView.bottomAnchor)
    }

    private func setupTextFonts() {
        [businessCardComponents.nameLabel,
         businessCardComponents.titleLabel,
         businessCardComponents.emailLabel,
         businessCardComponents.phoneLabel].forEach { $0.font = .systemFont(ofSize: defaultFontSise) }
    }
}
~~~

## Bonus - Trafegando dados

Poderia ser utilizado uma Entity/Value Object qualquer pra dentro do componente que possui a foto e as labels. Mas para fazer um código mais limpo, devemos solicitar somente os dados necessários e geralmente as Entities possuem dados além do que vai ser exibido.

Além de não utilizar todos os dados de uma Entity, o(a) próximo(a) programador(a) não vai saber o que daquela entity deve ser preenchido para que a view funcione corretamente. A menos que ele(a) abra a classe `BusinessCardComponents` e descubra todos os valores utilizados. Pra resolver essa dor tenho duas sugestões.

A primeira sugestão é colocar na função que vai popular a tela, todos os parâmetros utilizados. O ruim disso é que podemos ter uma função com uma assinatura muito extensa. Espia:

~~~swift
final class BusinessCardComponents {
    // ...
    init(photo: UIImage, name: String, title: String, phone: String, email: String ... ∞) {
         // ...
    }
}
~~~

E a segunda alternativa eu tenho usado em diversos momentos. Criar um `typealias` daquilo que estaria na assinatura do método. Assim a assinatura fica menor e ainda podemos contar com o help (option + click) do Xcode.

~~~swift
typealias BusinessCardParams = (photo: UIImage, name: String, title: String, phone: String, email: String)

final class BusinessCardComponents {
    // ...
    init(params: BusinessCardParams) {
         // ...
    }
}

let params = BusinessCardParams(photo: somePhoto, name: "Juca", title: "Desenvolvedor de Software", phone: "(11) 1111-1111", email: "oi@juca.com")
let seniorBsinessCardComponent = BusinessCardComponents(params: params)
~~~

<p align="center"><img style="border-radius: 3px;" src="{{ site.baseurl }}/img/ronanrodrigo/xcode-help.png"></p>


# Conclusão

Quando se trata de views construídas com código temos diversas soluções e "não existe bala de prata". O lado positivo é que podemos trabalhar com nossa criatividade para sermos mais produtivos. Acredito que o Interface Builder acaba nos limitando e nos acomodando.

Em termos práticos se a escolha fosse utilizar Storyboard ou XIBs teríamos três telas desenhadas com pouco reaproveitamento. Além disso, ficaria mais difícil de criar testes e o tráfego de dados também ficaria mais oneroso.

As views com código requerem um novo aprendizado. Mas é uma curva de aprendizado baixa. Experimente sair da zona de conforto do Interface Builder, aprenderás mais sobre UIKit, vai aprimorar sua criatividade e futuramente, aos poucos, aumentará sua produtividade.

### Agradecimentos

Agradeço aos companheiros da Concrete Solutions por me ajudarem a oxigenar meu cérebro. Agradeço também à minha companheira Nayara. Ela quem me incentivou a buscar minha especialização fora da minha cidade natal e fisicamente longe dela, estou com saudades ❤️.
