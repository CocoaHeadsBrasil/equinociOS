---
layout:     post
title:      "Minimizando o acoplamento entre Views e ViewControllers"
subtitle:   "Uma opção para configurar Views"
date:       2016-03-08 00:00:00
author:     "Diogo Tridapalli"
header-img: "img/diogot/plainMVVM.png"
category:   Arquitetura
---

O excesso de responsabilidades do `ViewController` é algo que vem me incomodando há algum tempo. Já [escrevi](http://invariante.com/2015/10/20/todo-view-controller-deveria-ter-delegate/) um pouco sobre a relação entre `ViewControllers`, hoje quero falar sobre a relação entre as `Views` e o `ViewController`.

`Views`, de maneira geral, são genéricas. Não contêm lógica e podem ser reutilizadas em diferentes contextos. Por exemplo, `UILabel` possui uma série de customizações como `text`, `font` e `textColor`, que permitem que ela seja usada em diversos contextos. Até aqui tudo certo[^1]. Na maioria dos casos nem todas as configurações são alteradas, o mais comum é alterar as 3 citadas. Em alguns casos é necessário definir uma configuração para o mesmo valor, por exemplo `textColor`, cujo o valor padrão é `blackColor` mas no *app* todos os textos são `grayColor`, para evitar a replicação de código algumas vezes eu já fiz uma subclasse de `UILabel` com configurações padrão diferentes, mas isso sempre me pareceu estranho. Essa questão de onde deve ficar o código que define as configurações é um ponto desconfortável. Por exemplo o `text`, normalmente quem atribui uma *string* é o `ViewController` que é um estranho pois normalmente ele tira essa *string* de um `Model`. Quem deve ter o propósito de definir essas configurações? 

[^1]: certo?

Primeiramente o que são essas configurações? São um conjunto de customizações de um componente que correspondem a um resultado final específico, vamos chamar esse conjunto de configurações de **estado**. Note que a definição de **estado** aqui é a mesma usada em física[^2], uma descrição que define de maneira única e não ambígua a configuração de um sistema. Por exemplo, se o sistema for a localização de algo em uma rua, um estado é definido pelo número da casa. Se o sistema for a localização de algo em alguma cidade na terra, temos várias descrições (ou *modelos*) de estados possíveis, por exemplo [País, cidade, rua, número da casa] ou [latitude, longitude, altitude].

[^2]: ¯\\_(ツ)\_/¯

Então cada componente possui um número quase infinito de estados diferentes, pois, no caso do `UILabel`, para cada `text` diferente temos um estado diferente. Se cada estado é único e corresponde a um conjunto de configurações, porque não criar uma classe responsável por definir o estado de uma `View`? Precisamos de um nome para essa descrição de estado, como devemos fazer uma *modelagem* de quais as configurações são relevantes acho que `ViewModel` pode ser um nome adequado ;-)

Um `LabelViewModel` simplificado seria:

~~~ swift
struct LabelViewModel {
    let text: String
    let textColor: UIColor
    let font: UIFont
}
~~~

A `ViewModel` deve ser imutável, então um `struct` parece mais adequado.
Para definir uma configuração padrão é só definir um `init`:

~~~ swift
init(text: String = "", 
     textColor: UIColor = UIColor.blackColor(),
     font: UIFont = UIFont.systemFontOfSize(17))
{
    self.text = text
    self.textColor = textColor
    self.font = font
}
~~~

E uma `extension` do `UILabel` faz a conexão:

~~~ swift
extension UILabel {
    var viewModel: LabelViewModel {
        set(viewModel) {
            self.text = viewModel.text
            self.textColor = viewModel.textColor
            self.font = viewModel.font
        }
        get {
            return LabelViewModel(
                text: self.text ?? "",
                textColor: self.textColor,
                font: self.font)
        }
    }
}
~~~

A ideia parece interessante, mas esse exemplo dá uma impressão de excesso de complexidade. Concordo, realmente estamos trocando 6 por meia dúzia.
Vamos tomar um exemplo mais real. Temos uma `View` que contém uma `UILabel` e um `UIButton`. Sua interface pública seria:

~~~ swift
public class View : UIView {

    public var viewModel: ViewModel

    internal let label: UILabel
    internal let button: UIButton
}
~~~

Note que as *subviews* não são expostas e a única maneira de alterar o *estado* dessa `View` é alterando o `ViewModel`, que seria assim:

~~~ swift
public struct ViewModel {

    public let attributedLabel: NSAttributedString
    public let attributedButtonText: NSAttributedString
    public let backgroundColor: UIColor
    public let onTap: ((viewModel: ViewModel) -> Void)?

    public init(
        attributedLabel: NSAttributedString = NSAttributedString(string: "Label"),
        attributedButtonText: NSAttributedString = NSAttributedString(string: "Button"),
        backgroundColor: UIColor = .grayColor(),
        onTap: ((viewModel: ViewModel) -> Void)? = nil)
    {
        self.attributedLabel = attributedLabel
        self.attributedButtonText = attributedButtonText
        self.backgroundColor = backgroundColor
        self.onTap = onTap
    }
}
~~~

Os atributos que podem ser alterados são o`NSAttributedString` da `label`, `NSAttributedString` do `UIButton` em `UIControlState.Normal`, a `backgroundColor` e um *closure* que é executado quando o botão tem um *tap*.

O *estado* dessa `View` depende de um `Model`:

~~~ swift
public struct Model {

    public enum Emoji: String {
        case 👍, 👎, 👊
    }

    public let name: String
    public let emoji: Emoji

}
~~~

Dado que a configuração da `View` é necessariamente feita de maneira a representar visualmente o `Model` o `ViewModel` funciona como um *[adapter](https://pt.wikipedia.org/wiki/Adapter)*, uma interface com apenas com as configurações relevantes. Isso é representado por uma `extension` do `ViewModel`:

~~~ swift
public extension ViewModel {

    public init(fromModel model: Model, onTap: (viewModel: ViewModel)->Void) {

        let name: NSAttributedString =
        NSAttributedString(
           string: model.name,
           attributes: [NSForegroundColorAttributeName: UIColor.redColor()])

        let text: String = model.emoji.rawValue

        self.init(
            attributedLabel: NSAttributedString(string: text),
            attributedButtonText: name,
            onTap: onTap)
    }

}
~~~

Aqui vemos mais uma grande vantagem dessa maneira de separar o código.
Testar como será a representação da tela dado um modelo fica muito simples, não é necessário instanciar `View` nem `ViewController`, um simples teste unitário nessa `extension` é suficiente. Isso porque toda a lógica está isolada em apenas um lugar!

Já que falamos de `ViewController`, qual seria seu papel aqui? Muito simples:

~~~ swift
let viewModel = ViewModel(fromModel: model) { viewModel in
    print("tap")
}
aView.viewModel = viewModel
~~~

Cria o `ViewModel` à partir do `Model` e define qual a ação que será tomada quando o botão for acionado. Menos código na `ViewController`, menos complexidade para testar porque tudo está isolado, mais uma vitória do bem!

Dessa forma todo o fluxo de informação entre a `View` e o `ViewController` necessariamente passa pelo `ViewModel`.

Um exemplo completo pode ser encontrado no repositório [PlainMVVM](https://github.com/diogot/PlainMVVM).

---

O nome `ViewModel` não foi usado aqui por mero acaso, ele vem de um padrão de arquitetura conhecido como `MVVM` ([Model-View-ViewModel](https://en.wikipedia.org/wiki/Model–view–viewmodel)). Ele tem se tornado popular, principalmente num contexto de programação reativa, inclusive em alguns artigos do [equinociOS](http://equinocios.com).
Mas pouco se fala fora desse contexto, a [NatashaTheRobot](https://twitter.com/NatashaTheRobot) deu uma palestra sobre [Protocol Oriented MVVM](http://www.slideshare.net/natashatherobot/protocoloriented-mvvm-extended-edition), com algumas idéias interessantes para organizar o código usando `MVVM`. A idéia que eu descrevi aqui pode não corresponder à um `MVVM` formal, mas ilustra de uma maneira razoavelmente simples como separar mais as responsabilidades facilitando testes e reaproveitamento de código.

---

Criticas, sugestões e comentários são sempre bem vindos, é só me *pingar* no [@diogot](https://twitter.com/diogot) ou no [Slack do iOS Dev BR](http://iosdevbr.herokuapp.com).

---
Diogo Tridapalli <br />
[@diogot](https://twitter.com/diogot)
