---
layout:     post
title:      "LLDB é seu amigo"
subtitle:   "Como usar LLDB para melhorar seu workflow"
date:       2017-03-08 00:00:00
author:     "Fernando Cezar Bunn"
header-img: "img/bunn/header.jpg"
category:   ios
---
> Fernando Bunn ([@fcbunn](https://twitter.com/fcbunn){:target="_blank"}) começou a trabalhar com mobile como um hobby desde 2006 na época que os apps eram feitos em J2ME. Dedicou sua carreira ao iOS desde 2008. Já fez apps para empresas Fortune 500, foi CEO de uma empresa focada em mobile apps, atualmente trabalha como Mobile Lead Developer e nas horas vagas gosta de reclamar do cocoapods no [Slack](http://iosdevbr.herokuapp.com){:target="_blank"}.

## Motivação
Esse dezembro dei uma palestra na [primeira conferência nacional do CocoaHeads](http://cocoaheadsconference.com.br){:target="_blank"} (Valeu pelo convite, pessoal) e decidi falar sobre LLDB. Fiquei um pouco preocupado no início por talvez ser um assunto "batido", mas tive um feedback muito positivo tanto pessoalmente quanto no twitter [1](https://twitter.com/guibayma/status/805104900464111616){:target="_blank"}, [2](https://twitter.com/walmyrcarvalho/status/805104805748346880){:target="_blank"}, [3](https://twitter.com/emannuel_oc/status/805101998924320768){:target="_blank"}... e graças a pressão social do [Rambo](https://twitter.com/_inside){:target="_blank"} e [Holanda](https://twitter.com/tholanda){:target="_blank"}, resolvi escrever esse blog post sobre o assunto aqui no EquinociOS.

## Introdução
LLDB é um assunto extenso, ao invés de um blog post daria para escrever um livro sobre esse tema, mas a ideia aqui é passar algumas dicas de como tenho usado o LLDB ao longo desses anos alguns exemplos práticos para realmente ajudar o dia a dia. Para uma visão aprofundada e detalhada sobre LLDB, recomendo o [site do projeto](http://lldb.llvm.org){:target="_blank"}

## O que é LLDB?
LLDB é basicamente o debugger default que vem nas novas versões do Xcode. Ele veio para substituir o antigo GDB e uma das coisas bacanas dele é que ele utiliza a LLVM para expression parse, algo muito útil que vamos ver mais à frente.

Curiosidade: O LLDB no [site oficial do projeto](http://lldb.llvm.org){:target="_blank"} suporta C/C++, RenderScript e Objective-C. A versão que vem no Xcode está no [Github da Apple](https://github.com/apple/swift-lldb){:target="_blank"}. Isso pode ser conferido ao rodar o comando `language` que lista as linguagens suportadas.

~~~
(lldb) language
     Commands specific to a source language.

Syntax: language <language-name> <subcommand> [<subcommand-options>]

The following subcommands are supported:

      cplusplus    -- Commands for operating on the C++ language runtime.
      objc         -- Commands for operating on the Objective-C language
                      runtime.
      renderscript -- Commands for operating on the RenderScript runtime.
      swift        -- A set of commands for operating on the Swift Language
                      Runtime.

For more help on any particular subcommand, type 'help <command> <subcommand>'.
~~~

E comparando os comment headers do runtime do [Swift](https://github.com/apple/swift-lldb/blob/68b732cd24e2f2376c53a6117f114d8e07c51964/source/Target/SwiftLanguageRuntime.cpp){:target="_blank"} e do [Objectiive-C](https://github.com/apple/swift-lldb/blob/68b732cd24e2f2376c53a6117f114d8e07c51964/source/Target/ObjCLanguageRuntime.cpp){:target="_blank"}

## Só um "testezinho"
Quantas vezes você já alterou o seu código apenas para fazer "um testezinho"? Coisas simples como situações onde você quer saber o valor de uma variável e acaba fazendo algo como isso

~~~objective-c
NSLog(@"WTF %@", whatIsThis);
~~~
Ou precisou forçar uma condição para testar um fluxo específico e fez isso aqui?

~~~objective-c
if (myCondition || YES) {
    //do stuff
}
~~~
Sem contar que toda vez você tem que recompilar, rodar o app, esperar o simulador ou device e chegar no lugar no qual você quer testar (que com sorte é fácil de chegar), e aí sim ver o resultado. Fazer isso uma vez ou outra até que não parece o fim do mundo, mas somando pequenas iterações como essa no final do dia é muito tempo jogado fora. 

Mas qual o pior problema? Alteração de código!
Esse tipo de alteração é altamente problemática, é muito fácil esquecer uma mudança no código e acabar mandando pra production.
Inclusive o `NSLog` que a princípio é inofensivo, pode gerar muita dor de cabeça se for enviado para a app store. Já vi alguns apps que logavam a minha senha no console durante a fase de login, e tudo ficava salvo em plain text no seu aparelho.

![]({{ site.baseurl }}/img/bunn/senha_console.png)

## Breakpoints
OK, alterar código é ruim, mas qual a alternativa para o "testezinho"? **Breakpoints!**
Um breakpoint nada mais é do que uma maneira de pausar a execução e te dar controle sobre a continuação do fluxo do seu app.
Para adicionar um breakpoint basta clicar na linha que você quer que a execução pare.

![]({{ site.baseurl }}/img/bunn/breakpointSS.jpg)

Por trás dos panos o que está acontecendo nada mais é do que uma chamada como essa:

~~~
breakpoint set --file ViewController.m --line 15
~~~

Inclusive, se você decidir rodar esse comando no CLI do LLDB o breakpoint vai ser criado. A diferença é que fazendo assim o breakpoint não fica salvo entre diferentes sessões, ao contrário do Xcode que salva seu breakpoint até que você decida removê-lo. Importante perceber que existe uma relação direta com o que você faz na UI do Xcode e os comandos disponíveis do LLDB.

Xcode e LLDB são dois projetos distintos, ou seja, tudo que você faz no Xcode existe uma "alternativa" diretamente no CLI, porém, nem tudo que você consegue fazer no CLI está disponível no Xcode. Vamos ver mais à frente alguns exemplos quando eu comentar das "extensões" do LLDB.

Assim que o seu breakpoint for adicionado e a execução do seu app passar por ele, o console do LLDB vai aparecer na parte inferior do Xcode.

![]({{ site.baseurl }}/img/bunn/xcode_lldb.png)

Logo no header do console você pode ver alguns botões que servem para controlar o fluxo da sua aplicação, da esquerda para direita:

* **Hide debug area**: Serve para dar hide/show do CLI do LLDB;
* **Deactivate breakpoints**: É um botão toggle que vai ligar/desligar todos os seus breakpoints sem removê-los;
* **Continue**: Continua a execução normalmente do seu app;
* **Step Over**: Executa a linha onde o programa está esperando e para na próxima instrução;
* **Step Into**: Entra na declaração de um método;
* **Step Out**: Sai da declaração de um método e volta para quem o chamou.

## Comandos LLDB

Como escrevi anteriormente, os comandos do Xcode que interagem com o LLDB sempre possuem uma contraparte no CLI, os botões de Continue, Step Over, Step Into e Step Out vistos acima também podem ser executados diretamente no CLI através dos respectivos comandos: `continue`, `step`, `stepi` e `finish`.

OK, o programa está parado lá no seu breakpoint, você consegue ir passo a passo e investigar o que está acontecendo, mas seria legal ter maneiras de interagir com seus dados e fluxo de execução. Pra isso existem alguns comandos no LLDB pra te ajudar, aqui é uma breve lista de alguns comandos que vamos ver com mais detalhes mais para frente:

* **Expression**: O expression vai basicamente rodar *(evaluate)* uma expressão que você digitar. Essa expressão pode ser, por exemplo, uma linha de código exatamente como você digitaria no Xcode, inclusive é permitido **alterar** valores em runtime. Expression é tão importante que uma boa parte de comandos são apenas um alias do expression com certos parâmetros;
* **Print**: O comando Print serve para printar uma variável, geralmente utilizado para printar tipos primitivos;
* **Print Object**: Parecido com o print, mas geralmente é utilizado para printar objetos. Ele vai retornar o objeto formatado de uma maneira mais fácil de entender pois o resultado vem do método `debugDescription`  e faz fallback para o `description`. Inclusive é possível customizar esses resultados fazendo override desses métodos;
* **Watchpoint**: Esse aqui é legal para acompanhar se um valor foi alterado. Digamos que você tem uma variável e quer ser avisado de toda vez que ela sofrer uma alteração.

Para ver mais comandos do LLDB basta digitar `help` no CLI. [Este site](https://developer.apple.com/library/content/documentation/IDEs/Conceptual/gdb_to_lldb_transition_guide/document/lldb-command-examples.html
){:target="_blank"} também tem uma lista legal comparando os comandos do GDB com do LLDB.

Vale notar que o LLDB faz matching do prefixo dos comandos, ou seja, você pode digitar apenas `p` ao invés de `print`, `e` ao invés de `expression`, etc. 

## Printing

Digamos que você quer saber o valor de uma variável, um `NSInteger` por exemplo. Ao invés de escrever um `NSLog(@"Integer %li", counter);` você pode simplesmente adicionar um breakpoint no local de interesse, e quando o CLI do LLDB abrir apenas digite:

~~~
(lldb) p counter
(NSInteger) $0 = 1
~~~

Pronto, seu valor `1` foi retornado sem nenhuma alteração no seu código. Vamos tentar a mesma coisa para uma `UILabel`:

~~~
(lldb) p countLabel
(UILabel *) $1 = 0x00007fefb650a7f0
~~~

Hmm, esse resultado não parece muito útil se o nosso objetivo é saber o valor dessa label. Isso acontece porque o `print` está retornando o ponteiro da nossa `UILabel`. Como a `UILabel` é um objeto, para pegarmos um resultado mais interessante basta usarmos o `po`:

~~~
(lldb) po self.countLabel
<UILabel: 0x7fefb650a7f0; frame = (122 274; 76.5 20.5); text = 'Count = 1'; opaque = NO; autoresize = RM+BM; userInteractionEnabled = NO; layer = <_UILabelLayer: 0x618000088f70>>
~~~

Muito melhor, mas o que aconteceria se usarmos o mesmo `po` no nosso `self` que nesse caso é um `UIViewController` ?

~~~
(lldb) po self
<SampleViewController: 0x7fba51e03db0>
~~~

O que está acontecendo de errado? `SampleViewController` é um objeto, estou usando um `po` que deveria dar mais informações úteis desse objeto, assim como aconteceu com o `UILabel`. 
O problema é que o `po` vai retornar o resultado do método `debugDescription` fazendo fallback para o `description`. Nenhum desses métodos estão implementados no `SampleViewController`, então vamos mudar isso:

~~~objective-c
- (NSString *)debugDescription {
    return [NSString stringWithFormat:@"Debug description from ViewController [%@], Current count [%li], Label Value [%@]", self, (long)_counter, self.countLabel.text];
}
~~~

E esse é o novo resultado quando rodamos o `po`:

~~~
(lldb) po self
Debug description from ViewController [SampleViewController], Current count [1], Label Value [Count = 1]
~~~

Recomendo fortemente sempre implementar o `debugDescription` pois isso facilita muito a vida de quem está manuseando seus objetos.

## Watching

Considere uma situação onde você tem uma variável qualquer que está sendo alterada e não deveria, mas você não sabe quem, quando ou onde ela está mudando. Uma solução seria caçar todos os lugares do seu projeto, mas isso dá muito trabalho. Outra possibilidade seria fazer um override do setter dessa variável (caso ele não exista), mas isso altera código, precisa compilar de novo, e todos aqueles problemas que já comentei anteriormente.

Para isso existe uma solução muito boa chamada `watchpoint` e para usar é muito simples

~~~
(lldb) watchpoint set variable _counter
Watchpoint created: Watchpoint 1: addr = 0x7fdf52c00d88 size = 8 state = enabled type = w
    watchpoint spec = '_counter'
    new value: 0
~~~

E a próxima vez que o seu `counter` for alterado, o Xcode vai parar no frame onde isso acontece, e no CLI você vai ver o resultado com o valor antigo e o valor novo:

~~~
Watchpoint 1 hit:
old value: 0
new value: 1
~~~

Para listar todos os seus watchpoints use o comando `list`:

~~~
(lldb) watchpoint list
Number of supported hardware watchpoints: 4
Current watchpoints:
Watchpoint 1: addr = 0x7fdf52c00d88 size = 8 state = enabled type = w
    watchpoint spec = '_counter'
    old value: 0
    new value: 1
~~~

E assim que não precisar dele, apenas delete o watchpoint desejado com `delete`
Nesse caso o nosso watchpoint tem o número 1, então para deletar ele:

~~~
(lldb) watchpoint delete 1
1 watchpoints deleted.
~~~

## Interacting

Já vimos como inspecionar suas variáveis e fluxo da aplicação, mas e situações onde você quer interagir com a execução, mudar variáveis, criar outras, como fazer isso tudo sem ter que mudar seu código, compilar e tudo mais?

Usando o `expression` você pode fazer tudo isso, vamos ver alguns exemplos práticos de como ele pode ser utilizado.

Imagine uma situação onde você tenha um counter e queira testar um caso onde esse counter seja `100` porém ele é incrementado apenas de 1 em 1 ao apertar um botão.

Você poderia ir diretamente no código e alterar o valor para 100, mas novamente, isso altera código, adiciona bugs, precisa ser recompilado. Não é uma solução válida.

~~~
(lldb) p counter
(NSInteger) $0 = 1
(lldb) e counter = 100
(NSInteger) $1 = 100
(lldb) p counter
(NSInteger) $2 = 100
~~~

Usando o `expression` ou `e`  foi possível mudar o valor que antes era 1 para 100, isso tudo sem precisar alterar uma única linha de código.

Outra situação bem comum, você tem uma tela com um background meio feio, quer fazer alguns testes mas sempre precisa fazer uma alteração no código como essa:

~~~objective-c
self.view.backgroundColor = [UIColor purpleColor];
~~~

Porém, se você lembrar que o LLDB é seu amigo (roll credits) você pode pegar exatamente o mesmo trecho de código e rodar no `expression`

~~~
(lldb) e self.view.backgroundColor = [UIColor purpleColor];
(UICachedDeviceRGBColor *) $3 = 0x00006080000735c0
~~~

Pronto! O background color da sua `UIView` foi alterado durante a execução do seu projeto, assim fica bem fácil te testar várias cores diferentes sem ter que compilar o código a cada tentativa.

Você pode ter notado que o background só mudou a cor depois de você continuar a execução do seu projeto, pois antes ele estava preso no frame do seu breakpoint. Uma maneira de forçar a alteração da view sem precisar dar `continue` no LLDB é rodar o seguinte comando

~~~
e [CATransaction flush]
~~~

Outra coisa bem legal de fazer com o expression é a criação de novas variáveis ou até mesmo hierarquias de views:

~~~
(lldb) e [[UIView alloc] initWithFrame:CGRectMake(10, 10, 100, 100)]
(UIView *) $6 = 0x00007fd968409620
~~~

Assim você tem uma nova `UIView` criada diretamente no CLI do LLDB. E se você tem prestado atenção, toda vez que uma `expression` com retorno é executada, na linha logo abaixo sempre vemos um `${número}` como `$6` nesse código acima. Isso são variáveis que o LLDB está criando para você, então podemos continuar a criação da nossa `UIView` a partir disso.

~~~
(lldb) e $6.backgroundColor = [UIColor redColor];
(UICachedDeviceRGBColor *) $7 = 0x000061800007a180
(lldb) e [self.view addSubview:$6];
(lldb) e [CATransaction flush]
~~~
Com isso você já pode ver um quadrado vermelho no seu simulador sem nem ao menos ter que sair do modo de debugger. 

Acho que agora dá para ter uma ideia legal de como o LLDB pode ser útil para diversas tarefas no seu dia-a-dia, as possibilidades são grandes e estamos apenas na "ponta do iceberg", para ilustrar outra situação bem comum e diferente do que já vimos, é como simular um memory warning no seu device.

No simulador nós podemos apenas ir no `Simulate Memory Warning`. Mas e no device? 

~~~
(lldb) e [[UIApplication sharedApplication] performSelector:@selector(_performMemoryWarning)]
~~~
Chamando essa linha no LLDB seu app vai se comportar exatamente como se você desse um click no `Simulate Memory Warning` no seu simulador. Então fica notável a versatilidade do LLDB em diversas situações diferentes.

## Breakpoint Actions

Agora sabemos como criar os comandos para pegar mais informações do seu app ou até mesmo alterar comportamento sem mudar uma linha do seu código, mas seria bom se tivesse uma maneira de salvar essas funções para não ter que escrever o tempo todo. Por exemplo, você sempre quer saber o valor de uma variável X, fica muito ruim ficar fazendo um `p` ou `po` o tempo todo. Pra resolver isso nós temos o breakpoint actions.

Pare acessar o breakpoint action você dá um duplo click no seu breakpoint ou acessa pelo atalho Command + Alt + click.

![]({{ site.baseurl }}/img/bunn/breakpoint_action.png)

Aqui você consegue definir alguns parâmetros para seu breakpoint, como:

* **Condition**: Caso você queira fazer uma condição especial para seu breakpoint ser executado, por exemplo, durante um parser, você quer parar quando uma variável X for igual a Y. O código que você usa aqui é praticamente o mesmo que você escreveria no programa;
* **Ignore**: Caso você queira ignorar o seu breakpoint num número X de vezes, por exemplo num loop;
* **Action**: Aqui você seleciona qual ação seu breakpoint vai executar, vamos ver com detalhes mais pra frente;
* **Options**: Basicamente uma opção para continuar, caso você não queira que o programa pare nesse breakpoint, útil para logs, por exemplo. (Não sei porque está no plural se é apenas um checkbox).

## Actions

No painel de Breakpoint quando você clickar no `actions` uma lista com várias opções vai aparecer, essas são as opções de que ações você quer que rode assim que seu breakpoint for executado.

![]({{ site.baseurl }}/img/bunn/breakpoint_action_list.png)

* **Apple Script**: Executar um Apple Script toda vez que o breakpoint é ativado. Uma utilidade seria ter um script que mande um e-mail ou uma slack message enquanto roda num CI;
* **Capture GPU Frame**: Mais utilizado caso você esteja fazendo algo com OpenGL;
* **Debugger Command**: Todos os comandos que vimos anteriormente podem ser rodados aqui, então é uma das actions mais utilizadas no meu dia-a-dia;
* **Log Message**: Gosto bastante desse também porque ele provê uma maneira simples de adicionar logs no seu console, assim não precisa mais dos prints;
* **Shell Command**: Parecido com o Apple Script, mas nesse é focado em shell mesmo;
* **Sound**: Pode ser útil caso você tenha alguma situação com muita coisa acontecendo e quer ser notificado, nada mais é do que mais uma forma de você receber uma "notificação" do Xcode.

E você pode encadear vários no mesmo breakpoint, por exemplo rodar um log message e um som ao mesmo tempo depois que um debugger command for executado.

## Auto Login

Vamos pegar o breakpoint actions para resolver um problema que todos nós já tivemos que fazer uma vez ou outra: Login no seu app. Toda vez que você roda um app que precisa de login você tem que perder momentos preciosos da sua vida para

* Digitar o usuário;
* Digitar a senha;
* Apertar o botão de login.

Pode parecer pouco, mas fazer isso, toda vez que você roda o app, sendo que você roda o app inúmeras vezes em apenas um dia, rapidamente dá pra ver que é um fluxo que consome bastante tempo. Como resolvemos isso?

~~~objective-c
self.loginTextField.text = @"email@email.com";
self.passwordTextField.text = @"senhaSuperSegura";
~~~
Não! Nee! Niet! Nein! Imagina se um código desse vai pra production? Imagina se todos seus usuários do dia pra noite tem um usuário hardcoded porque você esqueceu de remover esse código antes de mandar para a app store.

Para resolver esse problema, podemos criar um breakpoint (no lugar de sua preferência, mas aqui eu decidi usar o `viewDidAppear`) com duas actions de **debugger command** e selecionar a opção de automatically continue:

Assim que você rodar o app é possível ver no console suas duas expressions rodando

~~~
(__NSCFString *) $0 = 0x0000610000055420 @"email@email.com"
(__NSCFString *) $1 = 0x000060000005b450 @"senhaSuperSegura"
~~~

Resolvido, não temos mais o problema de enviar uma senha hardcoded para production e também não precisamos mais perder momentos preciosos de nossa vida escrevendo a mesma coisa várias vezes ao dia. Mas e esse botão de login? Vamos automatizar isso também, afinal, ninguém tem tempo para ficar clickando no mesmo botão o tempo todo.

![]({{ site.baseurl }}/img/bunn/breakpoint_automatic_login.png)

Nada mais fiz do que adicionar mais um **debugger command** chamando manualmente o método de login, e também adicionei um **Log Message** para deixar claro o que está acontecendo. 

Próxima vez que seu breakpoint for chamado seu campo de login e password vão ser automaticamente preenchidos e além disso, o login vai ser chamado sem você precisar fazer mais nada, e esse é o output do console:

~~~
(__NSCFString *) $0 = 0x000060800005c500 @"email@email.com"
(__NSCFString *) $1 = 0x000060000005b450 @"senhaSuperSegura"
Automatic Login Enabled
~~~

E caso você não queira mais essas ações, você pode ou deletar seu breakpoint ou apenas desativá-lo momentaneamente. 

## Sharing
Outra coisa legal que é possível fazer com breakpoints é compartilhar ele no seu projeto. Um motivo interessante para fazer share de um breakpoint é quando você for criar um novo bug no [Bug Reporter](https://developer.apple.com/bug-reporting/){:target="_blank"} da Apple.
Com isso você cria o projeto demo e já adiciona os breakpoints com log messages ou até chamando os métodos que tem que ser chamados para causar o bug.

Outra situação é simplesmente ter logs padrões ou breakpoints para facilitar a vida do desenvolvedor diretamente no seu repositório. Mas cuidado, ao fazer isso as informações do breakpoint vão ficar salvas em plain text dentro do seu `xcshareddata`. Ou seja, você não quer adicionar nenhuma informação sensível num shared breakpoint.

Para fazer share do seu breakpoint basta ir no breakpoint navigator (⌘ + 7), clicar com o botão direito no seu breakpoint e selecionar "Share".

![]({{ site.baseurl }}/img/bunn/breakpoint_share.png)

## LLDB plugins

É possível escrever plugins para o [LLDB em Python](https://lldb.llvm.org/python-reference.html){:target="_blank"} com isso você pode escrever seus próprios comandos, o que é muito prático pois permite a criações de plugins excelentes como o [Chisel](https://github.com/facebook/chisel){:target="_blank"}.

O Chisel nada mais é do que uma coleção de novos comandos para você usar no seu LLDB. Após instalar o Chisel, ao escrever `help` no CLI do LLDB, você vai ver uma série de novos comandos dentro da section `Current user-defined commands`.

Recomendo fortemente dar uma conferida em todos os comandos do Chisel, mas vou listar apenas alguns que uso diariamente.

### mask
Quantas vezes já alteramos o código com alguma coisa do tipo:

~~~objective-c
self.loginTextField.backgroundColor = [UIColor redColor];
~~~
apenas para ver certinho os bounds que essa view está usando?
Com o **mask** você pode apenas fazer:

~~~
(lldb) mask self.loginTextField
~~~
![]({{ site.baseurl }}/img/bunn/chisel_mask.png)

E para remover:

~~~
(lldb) unmask self.loginTextField
~~~

### mwarning
Lembra quando eu usei o seguinte comando para simular um memory warning?

~~~
(lldb) e [[UIApplication sharedApplication] performSelector:@selector(_performMemoryWarning)]
~~~

Então, com o Chisel tudo que precisa ser feito é:

~~~
(lldb) mwarning
~~~

### pvc
Faz o print recursivo de um `UIVIewController`, por exemplo esse UINavigationController:

~~~
(lldb) pvc self.navigationController
<UINavigationController: 0x7ffd6801e200; view = <UILayoutContainerView; 0x7ffd67e04290>; frame = (0, 0; 320, 568)>
   | <LoginViewController: 0x7ffd67f08160; view = <UIView; 0x7ffd67e102c0>; frame = (0, 64; 320, 504)>
   | <SecondViewController: 0x7ffd69d01600; view = <UIView; 0x7ffd69e00d80>; frame = (0, 0; 320, 568)>
~~~

## Conclusão

É notável a diferença que uma boa aplicação do LLDB faz no nosso fluxo de trabalho. Sem contar que nos dá uma segurança muito melhor pois sabemos que nenhum log errado, nenhum código de "testezinho" vai ser enviado para nossos clientes. Isso sem contar nos inúmeros "build & runs" que se tornam completamente desnecessários e que certamente nos salvam incontáveis horas no decorrer de pouco tempo.

Sempre que você pensar em alterar um código para fazer "um testezinho", sempre que você for apertar aquele botão de run do Xcode para compilar e rodar seu projeto, lembre-se do LLDB, talvez ele consiga resolver esse problema para você, afinal, o LLDB é seu amigo 😊
