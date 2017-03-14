---
layout:     post
title:      "Metaprogramação para o resto de nós"
subtitle:	"Pare de escrever boilerplate"
date:       2017-03-14 00:15:03
author:     "Francesco Perrotti-Garcia"
header-img: "img/fpg1503/metaprogramming.jpg"
category:   "metaprogramming"
---


> Francesco Perrotti-Garcia ([@fpg1503](https://twitter.com/fpg1503){:target="_blank"}) é desenvolvedor iOS. Atualmente trabalha na [Moobie](https://www.moobie.com.br){:target="_blank"}, já trabalhou em aplicativos como PlayKids, iFood e SpoonRocket. Programa desde os 12 anos e nos últimos 6 está cada vez mais próximo do desenvolvimento iOS. Swift mudou sua maneira de ver o mundo e até de como programar em Objective-C. Adora gatos e nas horas vagas gosta de viajar, cozinhar e tirar fotos. 🐥


# O Que é?
Metaprogramação é uma palvara difícil simplesmente para dizer *programas que manipulam programas*. Especificamente neste artigo iremos falar sobre programas que escrevem programas (geração de código).


# Por que fazer?
Você já se deparou copiando e colando código ou então fazendo um código extremamente verboso, com pouco signficado e propenso a erros? Já copiou e colou a chave errada ao serializar um JSON? Já consertou um bug em um lugar do código e depois descobriu que aquela trecho estava replicado por vários lugares e teve que sair caçando eles por aí?

Para mim programar é criar abstrações de problemas do mundo real e metaprogramação é **a arte de abstrair uma abstração**.
Ao usar metaprogramação você escreve menos código repetitivo e, com isso, menos bugs.  


# Exemplo simples
Se você é como eu provavelmente sempre tem um Playground, e algo como o [regexr.com](http://regexr.com) aberto para evitar ter que ficar replicando código. //TODO: FINISH

# Sourcery
[Sourcery](https://github.com/krzysztofzablocki/Sourcery) é uma ferramenta open source mantida pelo [Krzysztof Zabłocki](https://github.com/krzysztofzablocki) que alavanca metaprogramação em Swift usando templates. Não se preocupe, vou explicar o que isso significa nos próximos parágrafos.

## O que é um *template*?
*Template* é uma palavra em inglês que pode ser traduzida como **modelo** ou **gabarito. Imagine um template como um cortador de biscoitos, algo que dá forma ao que é colocado dentro, você pode usar um cortador de biscoitos em massinha de crianças e isso não vai trasnformar ela em biscoito.
<img src="{{ site.baseurl }}/img/fpg1503/template.jpg">

## Por que *templates*?
//TODO: FINISH

## Stencil
[Stencil](http://github.com/kylef/Stencil) é uma linguagem de templates para Swift criada e mantida pelo [Kyle Fuller](https://github.com/kylef), a ideia é criar uma maneira de expressar a apresentação de algo. Farei uma introdução rápida ao Stencil porém encorajo você a dar uma lida na [documentação oficial](http://stencil.fuller.li)!

{% raw %}
- `{{ ... }}`: imprime variáveis
- `{% ... %}`: funciona para tags (mais sobre elas abaixo)
{% endraw %}

## Tags
As duas tags mais importantes são `for` e `if`:

### `for`
Suponhamos que há uma lista de usuários (chamada `users`) e queremos listar todos eles um embaixo do outro:

{% highlight html %}
{% raw %}
{% for user in users %}
{{ user.name }}
{% endfor %}
{% endraw %}
{% endhighlight %}

Além disso podemos usar a tag `empty` para lidar com listas vazias:

{% highlight html %}
{% raw %}
{% for user in users %}
{{ user.name }}
{% empty %}
Não há usuários :(
{% endfor %}
{% endraw %}
{% endhighlight %}

E por final temos à nossa disposição o contexto `forloop` que possui três variáveis: 

- `first`: boleano que indica se é a primeira iteração do loop
- `last`: boleano que indica se é a última iteração do loop
- `counter`: iteração atual do loop

### `if`
O `if` avalia uma variável para verdadeira se um dos abaixo for válido:
- presente no contexto
- coleções: não vazias
- boleano: verdadeiro
- número: maior que zero
- string: não vazia

{% highlight html %}
{% raw %}
{% if users.count == 42 and users.first.name == "Admin" %}
Lista de usuários interessante
{% else %}
Lista de usuários padrão
{% endif %}
{% endraw %}
{% endhighlight %}

### Filtros
Além disso Stencil possui alguns filtros (e Sourcery adiciona outros bem interessantes):

- `capitalize`: deixa a primeira letra da string em caixa alta e as demais em caixa baixa (`Taylor Swift -> Tayor swift`)
- `uppercase`: deixa todas as letras da string em caixa alta (`Taylor Swift -> TAYLOR SWIFT`)
- `lowercase`: deixa todas as letras da string em caixa baixa (`Taylor Swift -> taylor swift`)

Filtros são aplicados a uma variável usando o pipe (`|`), por exemplo: `{{ "Taylor Swift"|lowercase }}`

Há filtros que possuem parâmetros, esses parâmetros devem ser incluídos na forma `variável|filtro:parâmetro`, um exemplo é o filtro `join` em listas:

{% highlight html %}
{% raw %}
// myList = ["Uma", "lista", "com", "várias", "palavras"]
{{ myList|join:" 🥑 "}}
// Imprime "Uma 🥑 lista 🥑 com 🥑 várias 🥑 palavras"
{% endraw %}
{% endhighlight %}

Além disso podemos usar as e filtros [adicionadas pelo Sourcery](https://github.com/krzysztofzablocki/Sourcery#custom-stencil-tags-and-filter) e nas últimas versões você também pode usar os [exportados pelo StencilSwfitKit](https://github.com/SwiftGen/StencilSwiftKit).

### Anotações
Infelizmente Swift não possui suporte a anotações de código, no entanto Sourcery traz uma alternativa para isso: comentários com `/// sourcery`, veja o exemplo abaixo onde o `struct User` é anotado `AutoEquatable`: 

{% highlight swift %}
{% raw %}
/// sourcery: AutoEquatable
struct User {
    let id: UUID
    let name: String
    let email: String
}
{% endraw %}
{% endhighlight %}

Falaremos mais de como aproveitar anotações mais abaixo porém por enquanto é interessante saber que podemos usar um filtro de Stencil (`annotated`) para encontrar apenas coisas anotadas, ou seja, se quisermos escrever algum código apenas para todas as variáveis anotadas `AutoInject` entro de um tipo `type` faríamos assim:

{% highlight html %}
{% raw %}
{% for variable in type.variables|annotated:"AutoInject" %}
{{ variable }}
{% endfor %}
{% endraw %}
{% endhighlight %}

A beleza das anotações serem inseridas em comentários é que códigos anotados ainda são códigos Swift válidos que compilam normalmente!


## Instalação

Instalar o Sourcery é muito simples e para brincar com ele é interessante [baixar o binário](https://github.com/krzysztofzablocki/Sourcery/releases) e usar. Porém, para usá-lo dentro de projetos e garantir que todos estão na mesma versão você pode usar CoocaPods ou Swift Package Manager:

### CocoaPods
Simplesmente adicione `pod 'Sourcery'` na sua Podfile e `$PODS_ROOT/Sourcery/bin/sourcery {source} {templates} {output}` em uma Build Phase de Script.

### Swift Package Manager
Adicione a dependência e rode `.build/debug/sourcery {source} {templates} {output}.`

Para os exemplos a baixo eu recomendo que você tenha um binário preparado (na última versão) e três arquivos:

- Input.swift
- Template.stencil
- Output.swift

Eu sempre faço assim e crio um Script em shell para facilitar minha vida, seu conteúdo é somente:

{% highlight shell %}
sourcery Input.swift Template.stencil Output.swift --watch
{% endhighlight %}

Note que a flag `--watch` acompanha seus arquivos e automaticamente regera a saída baseado nas mudanças, é mágico! Note que para isso funcionar eu tenho o sourcery na minha `PATH`, caso você não tenha será necessário fornecer o caminho para o binário.

## Editores de texto
Para poder acompanhar as mudanças em tempo real aconselho que você divida sua tela em duas partes: código sendo editado (template/fonte) e saída gerada automaticamente. Usando a flag `--watch` que comentei acima basta salvar o arquivo que as mudanças são refletidas automaticamente

### *Sublime Text*
Infelizmente o Sublime Text não atualiza arquivos abertos automaticamente quando há mudanças no disco então não recomendo o uso dele. Você poderia instalar algum plugin para isso porém a falta dessa feature inviabiliza o uso dele junto para visualizar mudanças automaticamente.

### *Atom*
O Atom funciona incrivelmente bem para isso, além de permitir que você divida sua tela em vários panes em sentidos diferentes simultâneamente!
<img src="{{ site.baseurl }}/img/fpg1503/atom.gif">


### *VSCode*

<img src="{{ site.baseurl }}/img/fpg1503/vscode.gif">

# Casos de Uso
Para todos os exemplos abaixo farei implementações simples, elas não cobrem todos os casos porém estará explicito em quais casos elas funcionam e uma referência para uma implementação que lida com todos os *edge cases*.

## `Equatable`
Uma maneira simples de pensar em igualdade de tipos concretos é: todas as suas propriedades não computadas devem ser iguais. Essa implementação não lida com: Optionals, Enums, Arrays, Herança. Exemplo de implementação mais completa: [AutoEquatable](https://github.com/krzysztofzablocki/Sourcery/blob/master/Templates/AutoEquatable.stencil).

{% highlight html %}
{% raw %}
{% for type in types.implementing.AutoEquatable %}
extension {{ type.name }}: Equatable {}
{{ type.accessLevel }} func == (lhs: {{ type.name }}, rhs: {{ type.name }}) -> Bool {
    {% for variable in type.storedVariables %}
    guard lhs.{{ variable.name }} == rhs.{{ variable.name }} else { return false }
    {% endfor %}
    return true
}
{% endfor %}
{% endraw %}
{% endhighlight %}

Para o `struct User`:

{% highlight swift %}
{% raw %}
struct User: AutoEquatable {
    let id: UUID
    let name: String
    let email: String
}
{% endraw %}
{% endhighlight %}

Foi gerado o código:
{% highlight swift %}
{% raw %}
extension User: Equatable {}
internal func == (lhs: User, rhs: User) -> Bool {
    guard lhs.id == rhs.id else { return false }
    guard lhs.name == rhs.name else { return false }
    guard lhs.email == rhs.email else { return false }
    return true
}
{% endraw %}
{% endhighlight %}

## `AutoInjectable`
Imagine que você possui um `struct` com diversas propriedades porém quer que algumas delas sejam injetadas automaticamente e não quer perder o construtor que você ganhou, podemos para isso inserir uma anotação `AutoInjectable`, nosso `struct` ficaria assim:

{% highlight swift %}
{% raw %}
struct ApiService: AutoInjectable {

    /// sourcery: AutoInject
    let requestManager: RequestManager
    /// sourcery: AutoInject
    let sessionManager: SessionManager
    
    let baseURL: URL
    let serviceName: String
}
{% endraw %}
{% endhighlight %}

Podemos escrever um template assim:

{% highlight html %}
{% raw %}
{% for type in types.all.implementing:"AutoInjected" %}
extension {{type.name}} {
    convenience init({% for variable in type.variables|instance|!annotated:"AutoInject" %}{{variable.name}}: {{variable.typeName}}{% if not forloop.last %}, {% endif %}{% endfor %}) {
    {% for variable in type.variables|instance|stored|annotated:"AutoInject" %}
        let {{variable.name}}: {{variable.typeName}} = try! autoInject()
    {% endfor %}
        init({% for variable in type.variables %}{{variable.name}}: {{variable.name}}{% if not forloop.last %}, {% endif %}{% endfor %})
}

{% endfor %}

func autoInject<T>() throws -> T {
    //TODO: Sua lógica de injeção de dependências aqui!
}
{% endraw %}
{% endhighlight %}

Código gerado:

{% highlight swift %}
{% raw %}
extension ApiService {
    convenience init(baseURL: URL, serviceName: String) {
        let requestManager: RequestManager = try! autoInject()
        let sessionManager: SessionManager = try! autoInject()
        init(requestManager: requestManager, sessionManager: sessionManager, baseURL: baseURL, serviceName: serviceName)
}


func autoInject<T>() throws -> T {
    //TODO: Sua lógica de injeção de dependências aqui!
}
{% endraw %}
{% endhighlight %}

No exemplo acima somento criamos um construtor de conveniência que chama nossa função capaz de prover dependências e junta isso com os parâmetros não injetados numa chamada para o construtor designado.

## Desserialização de JSONs
Para serialização de JSONs usaremos o protocolo `JsonCreatable` que consiste de coisas que podem ser criadas a partir de um dicionário:

{% highlight swift %}
{% raw %}
protocol JsonCreatable {
    init?(json: [String: Any])
}
{% endraw %}
{% endhighlight %}

Nesse caso assumiremos que todas as propriedades não `Optional` são obrigatórias, que todas as variáveis sem um nome pré-definido tem seu próprio nome no JSON e não lidamos com tipos diferentes de Números, Strings e Booleanos. 

Nosso `struct`:

{% highlight swift %}
{% raw %}
struct User: AutoJsonCreatable {
    /// sourcery: JsonName = "fullName"
    let name: String
    let favoriteNumber: Int
    let isCool: Bool
    let numberOfTaylorSwiftAlbums: Int
    let favoriteQuote: String?
}
{% endraw %}
{% endhighlight %}

Template:

{% highlight html %}
{% raw %}
{% for type in types.all.implementing:"AutoJsonCreatable" %}
extension {{type.name}}: JsonCreatable {
    init?(json: [String: Any]) {
        {% for variable in type.variables|instance|stored %}
        {% ifnot variable.isOptional %}guard {% endif %}let {{variable.name}} = json["{{variable.annotations.JsonName|default:variable.name}}"] as? {{variable.unwrappedTypeName}}{% ifnot variable.isOptional %} else { return nil }{% endif %}
        {% endfor %}

        {% for variable in type.variables|instance|stored %}
        self.{{variable.name}} = {{variable.name}}
        {% endfor %}
    }
}
{% endfor %}
{% endraw %}
{% endhighlight %}

Código gerado:

{% highlight swift %}
{% raw %}
extension User: JsonCreatable {
    init?(json: [String: Any]) {
        guard let name = json["fullName"] as? String else { return nil }
        guard let favoriteNumber = json["favoriteNumber"] as? Int else { return nil }
        guard let isCool = json["isCool"] as? Bool else { return nil }
        guard let numberOfTaylorSwiftAlbums = json["numberOfTaylorSwiftAlbums"] as? Int else { return nil }
        let favoriteQuote = json["favoriteQuote"] as? String

        self.name = name
        self.favoriteNumber = favoriteNumber
        self.isCool = isCool
        self.numberOfTaylorSwiftAlbums = numberOfTaylorSwiftAlbums
        self.favoriteQuote = favoriteQuote
    }
}
{% endraw %}
{% endhighlight %}

A beleza de usarmos templates é que não precisamos nos limitar a Swfit! Imagine que seu colega Android vai ter que implementar tudo de novo então você poderia escrever um template para ele! 

{% highlight html %}
{% raw %}
{% for type in types.all.implementing:"AutoJsonCreatable" %}
package com.equinocios.{{type.name}};

import com.google.gson.annotations.SerializedName;

import java.io.Serializable;

public class {{type.name}} implements Serializable {

    {% for variable in type.variables|instance|stored %}
    @SerializedName("{{variable.annotations.JsonName|default:variable.name}}")
    private {{variable.unwrappedTypeName}} {{variable.name}} = null;

    {% endfor %}

    {% for variable in type.variables|instance|stored %}
    public {{variable.unwrappedTypeName}} get{{variable.name|swiftIdentifier}}({{variable.unwrappedTypeName}} {{variable.name}}) {
        return {{variable.name}};
    } 

    public void set{{variable.name|swiftIdentifier}}({{variable.unwrappedTypeName}} {{variable.name}}) {
        this.{{variable.name}} = {{variable.name}};
    }

    {% endfor %}
}
{% endfor %}
{% endraw %}
{% endhighlight %}


Esse template gera o código abaixo:

{% highlight java %}
{% raw %}
package com.equinocios.User;

import com.google.gson.annotations.SerializedName;

import java.io.Serializable;

public class User implements Serializable {

    @SerializedName("fullName")
    private String name = null;

    @SerializedName("favoriteNumber")
    private Int favoriteNumber = null;

    @SerializedName("isCool")
    private Bool isCool = null;

    @SerializedName("numberOfTaylorSwiftAlbums")
    private Int numberOfTaylorSwiftAlbums = null;

    @SerializedName("favoriteQuote")
    private String favoriteQuote = null;


    public String getName(String name) {
        return name;
    } 

    public void setName(String name) {
        this.name = name;
    }

    public Int getFavoriteNumber(Int favoriteNumber) {
        return favoriteNumber;
    } 

    public void setFavoriteNumber(Int favoriteNumber) {
        this.favoriteNumber = favoriteNumber;
    }

    public Bool getIsCool(Bool isCool) {
        return isCool;
    } 

    public void setIsCool(Bool isCool) {
        this.isCool = isCool;
    }

    public Int getNumberOfTaylorSwiftAlbums(Int numberOfTaylorSwiftAlbums) {
        return numberOfTaylorSwiftAlbums;
    } 

    public void setNumberOfTaylorSwiftAlbums(Int numberOfTaylorSwiftAlbums) {
        this.numberOfTaylorSwiftAlbums = numberOfTaylorSwiftAlbums;
    }

    public String getFavoriteQuote(String favoriteQuote) {
        return favoriteQuote;
    } 

    public void setFavoriteQuote(String favoriteQuote) {
        this.favoriteQuote = favoriteQuote;
    }

}
{% endraw %}
{% endhighlight %}

Lindo, não? O único problema é que o código acima não compila pois os tipos que usamos tem nomes diferentes em Java, poderíamos criar um [filtro customizado do Stencil](https://github.com/kylef/Stencil/blob/master/docs/custom-template-tags-and-filters.rst) (provavelmente chamado `javaTypeName`) que fizesse essa conversão. O ponto deste exemplo é mostrar o quão flexível templates nos permitem ser!


## Cliente HTTP
//TODO


# Outras ferramentas
//TODO

## SwiftGen
//TODO

# Imagens
Todas as imagens usadas nesse artigo são CC0.