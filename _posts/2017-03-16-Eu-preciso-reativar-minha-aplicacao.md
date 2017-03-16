---

layout:     post
title:      "Eu preciso reativar minha aplicação?"
subtitle:   "Utilização do Reactive Cocoa no nosso dia a dia."
date:       2017-03-16 12:00:00
author:     "Vinicius Carvalho"
header-img: "img/viniciuscarvalho/earth.jpg"
category:   Reativo

---

# Eu preciso reativar minha aplicação?

> Vinicius Carvalho ([@viniciusc70](https://twitter.com/viniciusc70)) | Trabalho com mobile desde 2013 e iOS desde 2014. Comecei a carreira em Fortaleza e hoje estou buscando novos ares em São Paulo. Nas horas vagas andar de bicicleta sem rumo é o que faço. Você pode me encontrar também no [Slack](http://iosdevbr.herokuapp.com) @viniciuscarvalho do iOSDevBr.

## O porquê

Estava trabalhando em um projeto com Objective-C ( *Isso mesmo objC que você leu :/ *), e me deparo com ações de toque em diversas células distintas de uma `UITableview`, uma tela de formulário. Agora você se pergunta, onde que entra o `Reactive` ai? Os eventos de toques na tela são codificados de diversas maneiras, *actions*, *delegates*, *KVO*, *callbacks*. O [ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa) define uma interface padrão para tais eventos, justamente para que eles possam ser facilmente encadeados, filtrados e compostos. A sua utilização me ajudou a entender e compreender o porque e quando usar tal ferramenta.

## Introdução

ReactiveCocoa é um paradigma que combina programação funcional e reativa, há diversos frameworks que o abordam, os seus principais são [RxSwift](https://github.com/ReactiveX/RxSwift) e o do ReactiveCocoa, como foi falado acima. Procurarei abordar mais o de porque usar e como me foi útil em um momento, mas nem sempre é necessário a sua utilização e você poderá acabar dando um tiro no pé ou querendo utilizar somente por estar na moda e você tenha lido muitos artigos sobre tal, um ponto sobre isso é você ler o artigo do [Bruno Mazzo](http://equinocios.com/arquitetura/2017/03/03/Introducao-a-arquitetura-evolutiva/), onde há um tópico que aborda esse ponto de utilização de ferramentas demais em nossas aplicações para desenvolver algo que era pra ser rápido se desenvolvido nativamente.

## Vamos lá?

![](http://i.giphy.com/xUA7b1XtsUC0P6aMY8.gif)

Antes de começar tudo eu tive que entender o que havia no código que estava me deparando naquele momento, por isso fui ver cada um dos eventos que o ReactiveCocoa possuía.

No Reactive, ele fornece uma interface padrão para lidar com o fluxo de disparo de eventos. Essa interface padrão tem como representação a classe **`RACSignal`**, onde possuímos três tipos, `next`, `error` e `completed`. Não vou passar afundo cada um, mas você pode saber mais sobre tais na [documentação](https://github.com/ReactiveCocoa/ReactiveSwift/blob/master/Documentation/FrameworkOverview.md#signals)

### Mas o que é um event?

Vejamos esse trecho de código para entender como funcionam,

```objc
[[[self.nameTextField.rac_textSignal
    map:^id(NSString *text) {
    	return @(text.length);
    }]

    filter:^BOOL(NSNumber *length) {
    	return [length integerValue] > 5;
    }]

    subscribeNext:^(id x) {
    	NSLog(@"%@", x);
    }];
```
Quando você vê o print no console, observará que ele começa a contar a partir do 6 em diante, para cada evento que ele recebe, ele executa o bloco e emite um valor de retorno como um próximo evento. Confuso não? Eu também fiquei quando comecei a ler sobre o assunto.
No código acima, o `map` pega o NSString de entrada junto com seu comprimento, o que resulta em um NSNumber sendo retornado com a função de `filter`, basicamente após o `rac_textSignal` quando ainda tinhamos o NSString, após o `map` ele recebe uma instância de NSNumber. Assim, podemos usar o `map` para transformar o dado em qualquer coisa, mas desde que seja um objeto!

Se quiser conhecer mais um pouco sobre `map` e `filter`, tem um artigo bacana sobre tal [aqui](http://equinocios.com/swift/2017/03/13/Introducao-e-casos-de-uso-Map-Filter-e-Reduce/).

## Baby steps

Vamos criar agora dois campos um de usuário e outro de senha e verificar se eles são válidos.

```objc
RACSignal *validUsernameSignal =
  [self.usernameTextField.rac_textSignal
    map:^id(NSString *text) {
      return @([self isValidUsername:text]);
   }];

RACSignal *validPasswordSignal =
  [self.passwordTextField.rac_textSignal
    map:^id(NSString *text) {
      return @([self isValidPassword:text]);
   }];
```

Como vemos novamente, o `map` transforma o `rac_textSignal` de cada campo, em um NSNumber que armazena um booleano. Beleza até aí?

O próximo passo é transformar esses sinais para que eles forneçam a mudança do background de cada `UITextField`.
Basicamente, você assina o signal e usa o resultado para atualizar a cor do fundo dos campos. Veremos uma opção com o seguinte trecho de código;

```objc
[[validPasswordSignal
  map:^id(NSNumber *passwordValid) {
    return [passwordValid boolValue] ? [UIColor clearColor] : [UIColor yellowColor];
  }]
  subscribeNext:^(UIColor *color) {
    self.passwordTextField.backgroundColor = color;
  }];
```

Vamos entender o código, estamos atribuindo a saída desse signal a propriedade do `backgroundColor` do campo de texto. Só que, esse código está com expressões como se tivessem passado!

![](http://i.giphy.com/3otPouMUsmarhYbpaE.gif)

Mas calma, vamos resolver isso!

O bendito do ReactiveCocoa possui uma macro que permite transformar esse código de uma maneira elegante e tirar essa dubiedade do nosso código,

```
RAC(self.passwordTextField, backgroundColor) =
  [validPasswordSignal
    map:^id(NSNumber *passwordValid) {
      return [passwordValid boolValue] ? [UIColor clearColor] : [UIColor yellowColor];
    }];

RAC(self.usernameTextField, backgroundColor) =
  [validUsernameSignal
    map:^id(NSNumber *passwordValid) {
     return [passwordValid boolValue] ? [UIColor clearColor] : [UIColor yellowColor];
    }];
```

Observando o código acima, temos a macro RAC que permite que atribuamos a saída de um signal para a propriedade de um objeto. É necessário passar dois argumentos, o primeiro é objeto que contém a propriedade e o segundo é o nome da propriedade. Cada vez que o signal manda um próximo evento, o valor que passa agora é atribuído a determinada propriedade. Entendido?

Mas você deve estar se perguntando o porquê de criar dois signals separados não é? Acaba que torna mais fácil o entendimento e a segmentação dos mesmos. Uma vez que você pode fazer validações distintas e inclusive facilitando a criação de teste unitário.

## Conclusões

Passamos por alguns pontos importantes do ReactiveCocoa, comecei a escrever esse artigo visto que não conhecia e não sabia alguns desses pontos que foram passados aqui, tem muito mais lá no github do projeto e com Swift é bem mais encorpado a linguagem... Mas, entretanto, todavia, nem tudo são flores nobres amigos!
Você pode estar achando que vai chegar hoje ou amanhã e começar a implementar Reactive em todas as suas classes não é? Não não!

![](http://i.giphy.com/JYZ397GsFrFtu.gif)

Estude bem antes de começar a inserir signals, maps, filters em sua aplicação, você pode estar inserindo complexidade onde não há!
Não tenhas as mesmas experiências traumáticas que eu. Quando vi esse rac_signal comecei a imaginar que não sabia mais programar. Pois, nunca tinha visto isso e era muito complicado.
Pare, pense e veja se isso é o melhor pra você naquele momento da aplicação.

No mais eu vou deixar dois artigos, para estudo e conhecer mais sobre ReactiveCocoa!

(https://medium.com/swift-programming/reactive-swift-3b6050375534#.fiyfof6j2)
(https://www.raywenderlich.com/126522/reactivecocoa-vs-rxswift)

Até a próxima e obrigado! \o/
