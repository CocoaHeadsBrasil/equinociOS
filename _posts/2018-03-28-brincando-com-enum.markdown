---
layout:     post
title:      "Brincando com Enum"
subtitle:   "Usando valor associado e RawValue"
date:       2018-03-28 12:00:00
author:     "Eduardo Tolmasquim"
header-img: "img/eduardoTolmasquim/list.jpg"
category:   Categoria
---
>Olá pessoal, como é a primeira vez que publico no EquinociOS, vou me apresentar. Sou o Eduardo Tolmasquim e entrei no mundo iOS pelo curso BEPiD em 2015 na PUC-rio (que hoje em dia se chama Apple Developer Academy). Atualmente sou dev iOS na Fulllab e nos tempos livres mantenho um blog chamado [doidiOS](https://www.doidios.com). A maior revolução na minha vida foi quando aprendi a ler no ônibus sem ficar enjoado. As vezes até torço para ter trânsito 🤣. Ok, não chega a tanto... Mas o podcast é hors concours na hora de lavar a louça.

####Agora vamos brincar com enum?

Os `enum`s são perfeitos para modelar dados onde coisas são (como diz o nome), enumeráveis.

Por exemplo neste caso onde tenho dois tipos de roupas:

~~~
enum Clothes {
    case hat
    case shoe
}
~~~
Agora podemos usar os valores

~~~
let myBeautifulHat = Clothes.hat
print(myBeautifulHat)
// hat
~~~

Tudo está ótimo, até que você percebe que não adianta nada ter um sapato quando não se sabe o tamanho dele.

Então você pensa: Sem problemas! Vou criar uma variável opcional para guardar o tamanho.

~~~
enum Clothes {
    case hat
    case shoe
}
var shoeSize: Int?
~~~

Porém, quando você vai usar o `enum`, aparece o desconforto

~~~
let myBeautifulShoe = Clothes.shoe
shoeSize = 45 // Sim, em calço 45 👞

if let size = shoeSize {
    print(myBeautifulShoe, size)
}
//shoe 45
~~~

Não sei se você está sentindo, mas o _code smell_ está muito forte 🤢

1. Sempre que formos usar o sapato temos que fazer `unwrap` da variável _shoeSize_? Boring! 🙄
2. Se criarmos um sapato mas esquecermos de criar um valor, teremos um sapato sem tamanho, o que não faz sentido.


###A solução é simples: Enum com valor associado! 🎉

Fazendo...

~~~
enum Clothes {
    case hat
    case shoe(size: Int)
}
~~~

Podemos usar

~~~
let hat = Clothes.hat
let shoe = Clothes.shoe(size: 45)

print(shoe)
//shoe(40)
~~~

Se não soubermos o tamanho do sapato, podemos até usar a função que o `enum` nos dá gratuitamente, para usar depois

~~~
let makeShoe = Clothes.shoe
let ShoeSize45 = makeShoe(45)
~~~


###Enum com RawValue

 
Agora vamos ver outra funcionalidade legal do `enum`.

Muitas vezes `enum`s são relacionados a outro tipo de valor (o `rawValue`), geralmente para inicializar com valores vindos de outros sistemas.

Podemos usar qualquer tipo que seja `RawRepresentable`. Por exemplo `Int` ou `Strings`. (`Double` não!)

~~~
enum Drink: String {
    case water
    case soda
}
~~~

O uso

~~~
let myDeliciousDrink = Drink(rawValue: "water")

let sodaName = Drink.soda.rawValue
~~~

Assim ganhamos de graça um `init` e ainda podemos acessar o valor `String` correspondente aos valores do `enum`.

Mas atenção, este init pode falhar!

~~~
let coldDrink = Drink(rawValue: "mineirinho")
~~~

Infelizmente não temos mineirinho no enum, então a variável _coldDrink_ será nula

~~~
print(coldDrink ?? "nil")
//nil
~~~

Agora podemos juntar tudo que aprendemos e fazer um super `enum` com `rawValue` e valor associado!

~~~
enum Clothes: String {
    case hat
    case shoe(size: Int)
}
~~~

Mas aí, erro de compilação...

![]({{ site.baseurl }}/img/eduardoTolmasquim/compilationError.png)

Pois é... `enum` com `rawValue` não pode ter valor associado 😵

Bom, faz sentido. O `enum` não tem como saber se `Clothes.shoe(size: 45).rawValue`  deveria ser _"shoe45"_, _"shoe-45"_, somente _"shoe"_, `nil` ou qualquer outra coisa.

### Mas nada está perdido. Nós podemos implementar um enum com rawValue e valor associado 💪

Veja este exemplo

~~~
// Primeiro criamos um enum 
// como se só tivesse valor associado

enum Clothes {
    case hat
    case shoe(Int)
    
    // Criamos um outro enum aqui dentro, 
    // agora sim com rawValue.
    
    private enum ValueFromString: String {
        case hat
        case shoe
    }
    
    // Este é o init falhável, onde a gente diz 
    // como que a String vira enum. 
    // Neste caso queremos que só "hat" possa ser
    // inicializada com String.
    
    init?(rawValue someString: String) {
        guard let type = ValueFromString(rawValue: someString) else {
            return nil
        }
        switch type {
        case .hat: self = .hat
        case .shoe: return nil
        }
    }
    
    // Criamos uma variável RawValue que vai converter
    // o enum para String. 
    // Neste caso queremos que shoe seja traduzido 
    // para somente "shoe", independente do número.
    
    var rawValue: String {
        switch self {
        case .hat:
            return ValueFromString.hat.rawValue
        case .shoe:
            return ValueFromString.shoe.rawValue
        }
    }
}
~~~

Como usar:

~~~
let myOtherHat = Clothes.init(rawValue: "hat")

let shoeName = Clothes.shoe(45).rawValue
~~~


###Discutindo essa abordagem


####Comentários

* Neste caso optamos por não inicializar _shoe_ com `String`, mas poderíamos dar um valor padrão se quiséssemos.
* Optamos por retornar somente _"shoe"_ no `rawValue`, mas também poderíamos retornar algo como _"shoe-45"_

####O lado trabalhoso ⛏

Sempre que a gente quiser adicionar um caso no `enum`, teremos que adicionar em quatro lugares

1. o `enum` _Clothes_
2. o `enum` _ValueForString_
3. mais um caso no `switch` do `init`
4. mais um caso no `switch` do `rawValue`

####Mas pelo menos é seguro 🔒

Apesar de termos que adicionar os novos casos em quatro lugares, não corremos o risco de esquecer um lugar, pois isso geraria erro de compilação. Isso se deve ao fato de que o `switch` é obrigado a lidar com todos os casos.

Vale ressaltar que se alguém colocar um caso `default` no `switch`, ele se torna vulnerável novamente. 

---

O `enum` é uma ferramenta muito útil e poderosa do Swift, que permite escrevermos códigos mais organizados e legíveis. O que você achou desta abordagem? Conte nos comentários!

---
Crédito da imagem:

<a href='https://www.freepik.com/free-vector/flat-check-list-design_953317.htm'>Designed by Freepik</a>
