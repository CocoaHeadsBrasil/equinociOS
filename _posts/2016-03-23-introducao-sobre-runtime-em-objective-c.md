---
layout:     post
title:      "Introdução sobre Runtime em Objective-C"
subtitle:   ""
date:       2013-03-23 12:00:00 
author:     "Fernanda Gadeia Geraissate"
header-img: "img/fggeraissate/relogio.jpg" 
category:   "runtime"
---

# Introdução sobre Runtime em Objective-C

Alguns leitores estão familiarizados com bibliotecas como [Specta](https://github.com/specta/specta), [OCMock](http://ocmock.org/) e [ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa). O que num primeiro momento pode parecer mágica, estas ferramentas na verdade exploram ao máximo o fato do Objective-C ser uma linguagem dinâmica, isto é, o código em questão toma decisões em tempo de execução ("runtime") [sempre que possível](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40008048-CH1-SW1).

![]({{ site.baseurl }}/img/fggeraissate/bobruntime.jpg)

Isto fornece uma flexibilidade que pode ser aproveitada de diversas maneiras. Uma das mais conhecidas é consultar informações sobre um dado objeto, através de métodos como `isKindOfClass:`, `respondsToSelector:`, `conformsToProtocol:` e assim por diante. Pode-se também adicionar novos métodos, chamá-los e até mudar a sua implementação em runtime. Isto é comumente utilizado no OCMock, quando, por exemplo, um objeto "mockado" não mais depende de um serviço para obter a resposta de uma chamada, e sim, retorna um JSON que foi previamente setado.

Embora o runtime funcione na maior parte do tempo por debaixo dos panos, é interessante estudar mais sobre o assunto não só para entender melhor como a linguagem funciona mas também para evitar passos que muitas vezes são executados de forma corriqueira sem entender muito bem o por que.

Aqui será mostrado uma visão geral sobre o assunto, para mais detalhes é recomendado ler o “[Objective-C Runtime Programming Guide](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40008048-CH1-SW1)”.

##Algumas definições

Para saber como o runtime funciona é importante entender algumas definições. Selector, método e implementação podem num primeiro momento parecer que são a mesma coisa, mas na verdade são diferentes etapas de um processo feito em tempo de execução. Os termos mais importantes serão descritos a baixo. Não esqueça de implementar a biblioteca `<objc/runtime.h>` caso queira explorar os parâmetros a seguir.

###Selector

Um [selector](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/ObjCRuntimeRef/index.html#//apple_ref/c/tdef/SEL) (`typedef struct objc_selector *SEL`) nada mais é do que o nome de um método, como por exemplo `viewDidAppear:`, `setObject:forKey:`, etc. Note que o ":" faz parte do selector e serve para identificar quando é preciso passar parâmetros para um método. Para trabalhar diretamente com um selector, basta fazer, por exemplo:

~~~objc
SEL selector = @selector(viewDidAppear:)
//ou
SEL aSelector = NSSelectorFromString(@"viewDidAppear:")
~~~

###Method

Um [método](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/ObjCRuntimeRef/index.html#//apple_ref/c/tdef/Method) (`typedef struct objc_method *Method`) é a combinação de um selector e sua implementação. Para acessar um método de uma instância ou classe, basta fazer:

~~~objc
// Instance Method
Class class = [self class];
Method method = class_getInstanceMethod(class, selector);

// Class Method
Class class = object_getClass(self);
Method method = class_getClassMethod(class, selector);
~~~

O método `class_getInstanceMethod(class, selector)` retorna o método de instância que corresponde a implementação de um selector em uma dada classe, ou NULL, caso, por exemplo, a classe ou a classe pai não tiver o método de instância para um selector específico.

###Implementation

Uma [implementação](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/ObjCRuntimeRef/index.html#//apple_ref/c/tag/IMP) (`id (*IMP)(id, SEL, …`)) é basicamente o que está escrito dentro do bloco de um código. Um objeto do tipo IMP é um tipo de dado que aponta para o início da função que implementa o código. O primeiro argumento (id) aponta para a memória de uma dada instância de uma classe (ou no caso de um método de classe, um ponteiro para uma [metaclasse](http://www.cocoawithlove.com/2010/01/what-is-meta-class-in-objective-c.html)), também chamado de "receiver" (aquele que recebe o método), o segundo é o nome do método (SEL) e os restantes são os parâmetros que um método requere. A implementação pode ser adquirida da seguinte forma:

~~~objc
IMP implementation = method_getImplementation(method);
~~~

###Message

Enviar uma [mensagem](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/ObjCRuntimeRef/index.html#//apple_ref/doc/uid/TP40001418-CH1g-88778) é invocar um selector junto com os parâmetros que serão enviados para um "receiver". Por exemplo, ao fazer:

~~~objc
[button setTitle:@"title" forState:UIControlStateNormal];
~~~

o compilador chama a seguinte função:

~~~objc
objc_msgSend(button, @selector(setTitle:forState:), @"title", UIControlStateNormal);
~~~

e, assim, a mensagem enviada para o “receiver” button é o selector “setTitle:forState:” mais os argumentos “title” e “UIControlStateNormal”. É possível guardar uma mensagem em um objeto do tipo [NSInvocation](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSInvocation_Class/) para invocá-la posteriormente.

###Method Signature

A assinatura de um método ([NSMethodSignature](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSMethodSignature_Class/)) representa os tipos de dados que são aceitos e retornados por um método. Pode ser obtido por:

~~~objc
NSMethodSignature *signature = [receiver methodSignatureForSelector:selector];
~~~

onde o receiver é o objeto que implementa o método e o selector é o nome do método, como já foi discutido anteriormente.

###Invocation

Um objeto do tipo [NSInvocation](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSInvocation_Class/) é usado para guardar e enviar mensagens para um dado objeto. Ele contém todos os elementos necessários para enviar uma mensagem: um "receiver", um selector, parâmetros de envio e o valor que será retornado. Um exemplo de como implementar um objeto desse tipo é:

~~~objc
void invokeSelector(id receiver, SEL selector, NSArray *arrayArguments) {
    
    if (receiver != nil && [receiver respondsToSelector:selector]) {
        
        NSMethodSignature *signature = [receiver methodSignatureForSelector:selector];
        NSInvocation *invocation = [NSInvocation invocationWithMethodSignature:signature];
        [invocation setTarget:receiver];
        [invocation setSelector:selector];
        
        for(int i = 0; i < [signature numberOfArguments] - 2; i++) {
            id arg = [arrayArguments objectAtIndex:i];
            [invocation setArgument:&arg atIndex:i+2]; // The first two arguments are the hidden arguments self and _cmd
        }
        
        [invocation invoke]; // Invoke the selector
    }
}

- (void)invocationExample {

    NSMutableDictionary *dictionary = [NSMutableDictionary new];
    invokeSelector(dictionary, NSSelectorFromString(@"setObject:forKey:"), @[@"keyString", @"valueString"]);
}
~~~

Note que os argumentos são setados a partir do índice 2. Isto porque os índices 0 e 1 são reservados para os argumentos self e _cmd, respectivamente, e devem ser setados explicitamente como mostrado a cima, onde self é igual ao receiver e _cmd é o selector.

Se o leitor estiver familiarizado com testes unitários e usou o OCMock, já deve ter se deparado com a seguinte situação:

~~~objc
[[[managerMock expect] andDo:^(NSInvocation *invocation) {

        void (^successBlock)(NSString *aString) = nil;
        [invocation getArgument:&successBlock atIndex:2];
        
        successBlock(@"42");
        
}] successBlock:[OCMArg any] errorBlock:[OCMArg any]];
~~~

No caso, o objeto manager terá o retorno do bloco de resposta "mockado" toda vez que o método `successBlock:errorBlock:` for chamado. Note que o primeiro argumento é referenciado com o índice 2, o segundo a 3 e assim por diante (caso houvesse), devido ao fato do que foi discutido a cima.

##Juntando o quebra-cabeça

Com estas definições em mente fica mais fácil entender como o processo em runtime funciona e como estes conceitos estão relacionados. 

No Objective-C, a estrutura de uma classe possui:

* Um ponteiro para a classe pai (ou superclass)
* Uma "dispatch table", onde cada entrada associa um selector a uma implementação.

Os objetos instanciados por sua vez possuem um ponteiro para a estrutura de classe, chamado `isa`, que dá ao objeto acesso a sua classe e, por meio da classe, a todas as classes que herda. Ao enviar uma mensagem a um objeto, a função `objc_msgSend(receiver, selector, arg1, arg 2, etc…)` segue o isa que aponta para a estrutura de classe e tenta encontrar o selector na "dispatch table". Caso não encontre, a função segue o ponteiro que aponta para a superclass e tenta encontrar o selector na “dispatch table” dela. Falhas sucessivas fazem com que `objc_msgSend` vá subindo na hierarquia de classes até chegar na classe NSObject. Uma vez localizado o selector, o `objc_msgSend` chama o método correspondente e repassa os parâmetros, caso contrário, ocorre uma exceção. Dessa forma, as implementações são escolhidas em tempo de execução.

![]({{ site.baseurl }}/img/fggeraissate/messaging.gif)
<span class="caption text-muted">Créditos: https://developer.apple.com</span>

Para acelerar o processo, o sistema possui um cache para cada classe, que associa os selectors às implementações assim que vão sendo usadas. Quando uma mensagem for enviada, a função `objc_msgSend` checa primeiro esse cache antes de verificar a "dispatch table". Assim quanto mais tempo o programa for executado, mais rapidamente as mensagens serão enviadas.

##Método Swizzling

Uma das formas de se aplicar esses conceitos sobre runtime é através do método Swizzling, que consiste em modificar a "dispatch table" trocando selectors e implementações de dois métodos entre si.

Imagine que se queira, por exemplo, adicionar um log toda vez que uma tela aparece:

~~~objc
- (void)viewDidAppear:(BOOL)animated {
    
    [super viewDidAppear:animated];
    
    [self writeLogWhenTheViewAppeared];
}

- (void)writeLogWhenTheViewAppeared {
    
    NSLog(@"The view of viewController %@ appeared", self);
}
~~~

Agora imagine repetir este processo para uma UIViewController, UITableViewController, UINavigationViewController, etc. A técnica mostrada a seguir visa evitar esse tipo de repetição.

Resumidamente, será implementado o método `extensionViewDidAppear:` (que conterá a implementação do log) e será trocado a implementação dele com o do `viewDidAppear:`. Assim quando for enviado a mensagem para a viewController com o selector `viewDidAppear:`, será na verdade executado a implementação de `extensionViewDidAppear:`. Além do log, o método `extensionViewDidAppear:` irá chamar `[self extensionViewDidAppear:]`. Como os selectors estão trocados, a implementação de `viewDidAppear:` será executada e o processo poderá continuar normalmente. O código a seguir pode ser encontrado no [Github](https://github.com/fggeraissate/FGSwizzlingExample).

O primeiro passo é criar uma categoria e implementar a biblioteca `<objc/runtime.h>`:

~~~objc
#import "UIViewController+Swizzling.h"

#import <objc/runtime.h>

@implementation UIViewController (Swizzling)
~~~

O método a seguir aplicará a técnica Swizzling propriamente dita:

~~~objc
#pragma mark - Swizzling viewDidAppear:

+ (void)swizzlingViewDidAppear {
}
~~~

Dentro deste método será escrito:

~~~objc
Class class = [self class];
    
SEL selectorOriginal = @selector(viewDidAppear:);
SEL selectorSwizz = @selector(extensionViewDidAppear:);
    
Method methodOriginal = class_getInstanceMethod(class, selectorOriginal);
Method methodSwizz = class_getInstanceMethod(class, selectorSwizz);

 /*
     Class class = object_getClass(self);
     
     SEL selectorOriginal = @selector(...);
     SEL selectorSwizz = @selector(...);
     
     Method methodOriginal = class_getClassMethod(class, selectorOriginal);
     Method methodSwizz = class_getClassMethod(class, selectorSwizz);
*/
~~~

Primeiro é resgatado a classe do objeto no qual o método `viewDidAppear:` é implementado. Depois, guarda-se os selectors e métodos de `viewDidAppear:` e `extensionViewDidAppear:`. As linhas comentadas a cima mostram como fazer este mesmo processo para métodos de classe. Feito isso será pego a implementação dos respectivos métodos:

~~~objc
IMP implementationOriginal = method_getImplementation(methodOriginal);
IMP implementationSwizz = method_getImplementation(methodSwizz);
~~~

Agora será feito uma tentativa de adicionar à classe UIViewController (class) um método com o nome de `viewDidAppear:` (selectorOriginal), mas com a implementação de `extensionViewDidAppear:`. O último parâmetro trata-se de [um array de caracteres que correspondem aos tipos dos argumentos que serão passados para o método](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtTypeEncodings.html).

~~~objc
BOOL didAddMethod = class_addMethod(class,
                                    selectorOriginal,
                                    implementationSwizz,
                                    method_getTypeEncoding(methodSwizz));
~~~

Em caso de sucesso, o próximo passo será fazer com que a classe em questão adicione um método com o nome de `extensionViewDidAppear:` mas com a implementação do método original. Caso contrário, significa que a classe já contém uma implementação com o dado nome. Neste caso, é necessário apenas trocas suas implementações:

~~~objc
if (didAddMethod) {
        class_replaceMethod(class,
                            selectorSwizz,
                            implementationOriginal,
                            method_getTypeEncoding(methodOriginal));
        
} else {
        method_exchangeImplementations(methodOriginal, methodSwizz);
}
~~~

Agora é preciso escrever o método `extensionViewDidAppear:` propriamente dito:

~~~objc
#pragma mark - viewDidAppear: extension

- (void)extensionViewDidAppear:(BOOL)animated {
    
    NSLog(@"The view of viewController %@ appeared", self);
    
    [self extensionViewDidAppear:animated];
}
~~~

Lembre-se que após terminar a implementação é preciso chamar o método original. Isto será feito ao escrever `[self extensionViewDidAppear:]`.

Para finalizar, é necessário chamar o método `swizzlingViewDidAppear` no método `load`, que por sua vez é chamado assim que a classe for inicializada:

~~~objc
#pragma mark - Load

+ (void)load {
    
    static dispatch_once_t onceToken;
    dispatch_once (&onceToken, ^{
        
        [self swizzlingViewDidAppear];
    });
}
~~~

Como o método Swizzling muda o estado global da classe é preciso garantir que o código seja executado apenas uma vez, e por isso está sendo utilizado `dispatch_once`.

##Cuidados a serem tomados

Embora os conceitos a cima forneçam ferramentas poderosas, deve-se tomar o máximo de cautela antes de aplicá-los. Além da maior parte do tempo não ser necessário usá-los explicitamente, não entender a fundo como o processo funciona, ou caso haja alguma modificação interna da linguagem, pode fazer com que o aplicativo quebre quando menos se espere. No caso do método Swizzling, há o problema de haver conflito com métodos com mesmo nome, além de tornar o código mais difícil de debugar. A dica final é: pense em todas as possibilidades antes de utilizar os exemplos citados a cima.
