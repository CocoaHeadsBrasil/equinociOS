---
layout:     post
title:      "Interfaces assíncronas com AsyncDisplayKit"
subtitle:   "Uma introdução ao AsyncDispkayKit"
date:       2017-03-29 12:00:00
author:     “Guga Oliveira“
header-img: "img/gugaoliveira/layoutable-types.png"
category:   UI
---

> Sou o Guga Oliveira ([@gugaoliveira](https://twitter.com/gugaoliveira){:target="_blank"}), empreendedor, cursei engenharia eletrônica mas sempre trabalhei com softwares e programação. Abri uma empresa de desenvolvimento Web em Juiz de Fora no início de 2000, e em 2007 fundamos a [Handcom](http://www.handcom.com.br){:target="_blank"} focada em mobilidade. Trabalhamos com iOS desde o início, e criamos aplicativos em sua maioria focados no varejo e com foco empresarial. Em 2014 fundamos a startup [Microlocation](http://www.microlocation.com.br){:target="_blank"} que foi acelerada pelo Seed-MG e hoje faz parte de nosso solução para o varejo [Smart-Retail](http://www.smartretail.com.br){:target="_blank"}, atualmente estou estudando Swift e Análise de Dados, e participando bastante do desenvolvimento dos ecossistemas de Minas Gerais com o MGTI em BH e o Zer040 em Juiz de Fora.

## interface assíncrona? sem Interface Builder?

Fui apresentado ao [AsyncDisplayKit](http://asyncdisplaykit.org/) pelo Heberti Almeida, excelente dev iOS que conheci na WWDC de 2013. Ele estava usando o `ASDK` no desenvolvimento do novo app da [PostBeyond](https://postbeyond.com){:target="_blank"}, startup em que ele trabalha. No começo achei complicado apostar em uma biblioteca de terceiros para substituir as bibliotecas próprias do iOS como `UIKit`, XIBs e StroyBoards.

Mas na WWDC 2016, o Heberti me convidou para ir no evento do [Pinterest](http://www.pinterest.com){:target="_blank"} que acontecia em volda da WWDC. O evento foi promovido pelo Scott Goodsom [@ScottGoodson](https://twitter.com/ScottGoodson){:target="_blank"}, que é Head of Core Experience no Pinterest, e que idealizou a _AsyncDisplayKit_ para desenvolver o _Paper_ para o _Facebook_. 

[![WWDC 2016 AsyncDisplayKit event ](https://img.youtube.com/vi/8ngXakpE2x8/0.jpg)](https://www.youtube.com/watch?v=8ngXakpE2x8){:target="_blank"}<br/>
_WWDC 2016 AsyncDisplayKit event_

O que vi no evento me surpreendeu bastante, engenheiros de apps de famosos utilizando intensamente a biblioteca, com foco total em performance visual. O _Pinterest_ utiliza o _AsyncDisplayKit_ e possui engenheiros trabalhando de forma obsessiva na engenharia de renderezição das telas para que fiquem extremamente responsivas, com o mínimo _drop_ de _frames_ possível.

Como nosso aplicativo possui uma tela com muitas imagens de produtos e um fluxo quase infinito, como em uma timeline de produtos, resolvemos apostar na re-escrita com o AsyncDisplayKit nas próximas versões, além de quebrar um grande paradígma na empresa, que é o uso de Storyboard e Interface Builder.

Como é um framework complexo, e não por isso dificil de usar, este artigo pretende ser introdutório ao AsyncDisplayKit, tentarei complementar com assuntos mais avançados em breve.


## AsyncDisplayKit

O [AsyncDisplayKit](https://github.com/facebook/AsyncDisplayKit){:target="_blank"} é uma framework Open Source iOS feito no topo do UIKit com o objetivo de deixar até as interfaces visuais mais complexas com fluxo suave e responsivo. O Framework é _Open Source_ e está disponível via CocoaPods ou Carthage.  


O `UIKit` é um dos frameworks mais maduros do iOS, e foi desenvolvido quando os iPhones tinham recursos restritos de sistema, no início nem existia possibilidade de utilizar códigos assíncronos concorrentes. O problema é que mesmo atualmente o `UIKit` roda em uma única _thread_ e só funciona na _main queue_. Na maior parte dos casos funciona bem, mas quando envolve interfaces mais complexas ou necessita-se de mais recursos, a interface acaba não sendo tão responsiva. Para estes casos, trabalhar de forma assíncrona é extremamente necessário para criar uma interface lisa e responsiva com o usuário.

O iOS renderiza a interface com o usuário em 60 frames por segundo, o que nos dá apenas 16 milisegundos de CPU por frame, e caso você ultrapasse este mínimo intervalo de tempo o sistema operacional vai começar a liberar drop de frames imediatamente, tornando nossa interface muito menos responsiva e suave, principalmente agora que muitos apps utilizam de animações e efeitos para obter a atenção do usuário.

E nos tempos de autolayout e animações, existem várias tarefas para a _main thread_ realizar e consumir o pequeno tempo de frame que temos, como calcular a dimensão automática de cada célula, resolver os _constraints_ dos conteúdos das views, renderizar e decodificar imagens, criar sombras e contornos, efeitos como blur e shadow, redimensionamento de imagens para exibição e criação e manipulação de objetos do sistema.

O `ASDK` (_AsyncDisplayKit_) chegou para resolver esses problemas, ele move o máximo de trabalho de `UI` para ser executado em _background_. Por default ele permite que todas os cálculos de medidas, estruturações layout e renderizações sejam feitas de forma assíncrona, sem muitas otimizações um app pode experimentar uma grande redução do trabalho realizado na _main thread_.

O `ASDK` é incompatível com ***Interface Builder*** e ***Auto Layout***, a biblioteca foi escrita em _Objective-C_, mas o `ASDK` suporta completamente o _Swift_ . 

### Começando

A unidade fundamental do `ASDK` é o `node` ou `ASDisplayNode`, ele é uma abstração da `UIView`, que por conseguinte é uma abstração da `CALayer`. A `UIView` só pode ser manipulada na _main thread_, mas os `nodes` são _thread safe_ e podem ser instanciados e ter toda sua hierarquia configurada em _background_ . 

![Node - VIew - Layer]({{ site.baseurl }}/img/gugaoliveira/node-view-layer.png)

A idéia básica do `ASDK` é que você possa trabalhar com os `nodes` da mesma forma como já trabalha com as `views`, a grande maioria dos métodos e parâmetros da `UIView` e da `CALayer` possuem equivalentes nos `nodes`. 

Tudo o que aparece na tela do iOS é renderizado por um objeto `CALayer`, a `UIView` cria e mantém a referência do objeto `CALayer`, e os `nodes` extendem a `UIView` da mesma forma, como na imagem acima. Você pode acessar a _view_ e o _layer_ que estão abaixo diretamente com `node.view` e `node.layer`, desde que esteja na _main thread_.  

#### Containers de nodes 

Os `nodes` do `ASDK` devem ser usados dentro de _containers_ de `nodes`, o `ASDK` oferece alguns _containers_  que tem similares com classes do `UIKit` que estamos acostumados a trabalhar. Um `node` não deve ser adicionado diretamente a uma hirerarquia de `view` (se fizer isso é quase certo de você obter piscadas de tela). Um `node` deve ser adicionado como subnode de um _container_ de nodes. Esses _containers_ possuem a tarefa de gerenciar os subnodes para informá-los se está tudo pronto para renderizar na tela de forma mais eficiente possível.

Uma vantagem do uso dos _containers_  de `nodes` é que o _container_ automaticamente gerencia o pre-loading inteligente (_Inteligent Preloading_) dos `nodes` (`views`), que significa que toda tarefa de medição do layout, de carregamento de dados, decodificação e renderização serão feitos de forma assíncrona automaticamente. 

Os principais containers estão abaixo com os equivalentes no `UIKit`:

| Container `ASDK` | Equivalente `UIKit`  |
|:------:|:-:|
|[`ASViewController`](http://asyncdisplaykit.org/docs/containers-asviewcontroller.html){:target="_blank"}         |  `UIViewController` |
|[`ASTableNode`](http://asyncdisplaykit.org/docs/containers-astablenode.html){:target="_blank"}         | `UITableView`  |
|[`ASCollectionNode`](http://asyncdisplaykit.org/docs/containers-ascollectionnode.html){:target="_blank"}  |  `UICollectionView` |
|[ `ASPagerNode`](http://asyncdisplaykit.org/docs/containers-aspagernode.html){:target="_blank"}         |  `UIPageViewController`  |
|  `ASNavigationController` | `UINavigationController`  |
|  `ASTabBarController ` | `UITabBarController `  |

### Layout Engine 

A ***Layout Engine*** do `ASDK` é muito poderosa e oferece recursos únicos, ela é baseada no `CSS Box Model`, e fornece formas de declarar e especificar o tamanho e posição dos `subnodes`. Os `nodes` (`views`) são renderizados de forma concorrente por default, mas os cálculos de medidas e posição são executados de forma assíncrona ao se usar um `ASLayoutSpec` para cada `node`. Aqui está a maior diferença entre o `ASDK` e o `UIKit`, no `ASDK` o layout é declarativo e no `UIKit` é baseado em _constraint_ com o `Auto Layout`. 
 
A ***Layout Engine*** será assunto de um segundo artigo que sairá em breve, no entanto no fim do artigo deixarei referências.

### Instalando

O `ASDK` pode ser adicionado ao seu projeto via CocoaPods ou Carthage. 

#### CocoaPods 
Adicione o seguinte comando no seu `Podfile`:

~~~text
target 'MyApp' do
	pod "AsyncDisplayKit"
end
~~~

Saia do `XCode`, rode o comando dentro do diretório do projeto no Terminal: 

~~~text
> pod install
~~~

Para atualizar a versão do `ASDK`, rode o comando dentro do diretório do projeto no Terminal: 

~~~text
> pod update AsyncDisplayKit
~~~


#### Carthage 
Adicione o seguinte comando no seu `Cartfile` para usar a ***última versão*** do `ASDK`:

~~~text
github "facebook/AsyncDisplayKit"
~~~

Ou para usar a branch ***master***:

~~~text
github "facebook/AsyncDisplayKit" "master"
~~~

No terminal rode:

~~~text
> carthage update
~~~

Verifique no terminal se `AsyncDisplayKit`, `PINRemoteImage` and `PINCache` foram carregadas e compiladas. 

### importando o framework 

Importe o header do framework para usá-lo

~~~objective-c
#import <AsyncDisplayKit/AsyncDisplayKit.h>
~~~

Para usar com `swift` deve-se criar um [_bridge header_](https://developer.apple.com/library/content/documentation/Swift/Conceptual/BuildingCocoaApps/MixandMatch.html){:target="_blank"}.

## Mão na massa

A melhor maneira para iniciar é explorar os projetos de exemplo existentes no [GitHub](https://github.com/facebook/AsyncDisplayKit){:target="_blank"} do `ASDK`. A pasta ***examples*** possui diversos exemplos, inclusive um simulando o Instagram chamado ASDKgram. 

Vamos usar o exemplo ASViewController em Objective-c para nosso exemplo: 

<p align="center"><img style="border-radius: 3px;"  src="{{ site.baseurl }}/img/gugaoliveira/ASViewController.png"></p>

Para criar uma equivalente de `ViewController` vamos usar o container `ASViewController`

~~~objective-c
@interface ViewController : ASViewController
~~~

Nossa `ViewController` contem uma `TableView` que irá exibir as categorias de Imagens disponíveis para visualização, vamos então implementar a `TableView` usando o `ASTableNode`.

Declarando o array que contem os nomes da categoria e o `node` que reprensenta a `TableView`:

~~~objective-c
@property (nonatomic, copy) NSArray *imageCategories;
@property (nonatomic, strong, readonly) ASTableNode *tableNode;
~~~

Vamos configurar a `ViewController`, iniciando a `TableNode` e o `array` usando o `initWithNode`:

~~~objective-c
- (instancetype)init
{
    self = [super initWithNode:[ASTableNode new]];
    if (self == nil) { return self; }
    
    _imageCategories = @[@"abstract", @"animals", @"business", @"cats", @"city", @"food", @"nightlife", @"fashion", @"people", @"nature", @"sports", @"technics", @"transport"];
    
    return self;
}
~~~

Assim como na `TableView` para usar o `ASTableNode` deveremos nos conformar ao `delegate` e ao `datasource` da _TableNode_

~~~objective-c
....

@interface ViewController () <ASTableDataSource, ASTableDelegate>

.....

- (void)viewDidLoad
{
    [super viewDidLoad];
    
    ...
    
    self.node.delegate = self;
    self.node.dataSource = self;
}

....
- (void)dealloc
{
    self.node.delegate = nil;
    self.node.dataSource = nil;
}
~~~

Agora vamos implementar os `delegates` e `datasource` da `TableNode`

~~~ Objective-c 
- (NSInteger)tableNode:(ASTableNode *)tableNode numberOfRowsInSection:(NSInteger)section
{
    return self.imageCategories.count;
}

- (ASCellNodeBlock)tableNode:(ASTableNode *)tableNode nodeBlockForRowAtIndexPath:(NSIndexPath *)indexPath
{
    // Como o bloco é executado em outra thread em background temos que armazenar a imageCategory fora do bloco 
    NSString *imageCategory = self.imageCategories[indexPath.row];
    return ^{
        ASTextCellNode *textCellNode = [ASTextCellNode new];
        textCellNode.text = [imageCategory capitalizedString];
        return textCellNode;
    };
}

- (void)tableNode:(ASTableNode *)tableNode didSelectRowAtIndexPath:(NSIndexPath *)indexPath
{
    NSString *imageCategory = self.imageCategories[indexPath.row];
    DetailRootNode *detailRootNode = [[DetailRootNode alloc] initWithImageCategory:imageCategory];
    DetailViewController *detailViewController = [[DetailViewController alloc] initWithNode:detailRootNode];
    detailViewController.title = [imageCategory capitalizedString];
    [self.navigationController pushViewController:detailViewController animated:YES];
}
~~~

Reparem que os métodos são bem similares aos da `UITableView`, a `ASTextCellNode` é uma subclasse de `ASCellNode`, que é o equivalente do `ASDK` a `UITableViewCell` ou `UICollectionViewCell`. No caso a `ASTextCellNode` possui um label simples para exibição de texto. Mas existe uma diferença no método de exibição da célula.

~~~Objective-C

//UITableView
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath

//ASTableNode
- (ASCellNodeBlock)tableNode:(ASTableNode *)tableNode nodeBlockForRowAtIndexPath:(NSIndexPath *)indexPath
~~~

A diferença aqui é que o retorno é um bloco de código que pode ser executado em background pelo `ASDK`, neste caso a variável `indexPath` não pode ser usada dentro do bloco, pois os dados podem mudar antes do bloco ser executado. Não é necessário reusar as celulas, o `ASDK` faz o trabalho para você, basta iniciar o `ASCellNode` e retorná-lo ao fim do bloco.

Ao se selecionar uma linha da `TableNode` instanciamos a classe ***DetailRootNode***, que é uma subclasse de `ASDisplayNode` (o node fundamental do `ASDK`), com o inicializador que recebe o nome da categoria de imagens, depois instanciamos a ***DetailViewController*** que é uma subclasse de `ASViewController` com o `node` ***DetailRootControler***

Na ***DetailRootNode*** temos uma instância de `ASCollectionNode` o _container_ de `nodes` que é equivalente a `UIViewController`. Aqui vamos usar a capacidade de um `node` gerenciar os `subnodes`, automaticamente o `ASDK` irá adicionar a `CollectionView` à view ***DetailRootNode*** que é adicionada à DetailViewController.

~~~Objective-C
// DetailRootNode é um ASDisplayNode
@interface DetailRootNode : ASDisplayNode

//o ASDisplayNode possui uma ASCollectionNode (equivalente UIViewCollection)
@property (nonatomic, strong, readonly) ASCollectionNode *collectionNode;

....

// conforma-se os delegates e datasource da ASCollectionView

@interface DetailRootNode () <ASCollectionDataSource, ASCollectionDelegate>

....

- (instancetype)initWithImageCategory:(NSString *)imageCategory
{
    self = [super init];
    if (self) {
        // Habilita o gerenciamento automatico dos subnodes todos os nodes que 
        // estiverem referenciados no metodo layoutSpecThatFits: serão adicionados automaticamente
        self.automaticallyManagesSubnodes = YES;
        
        _imageCategory = imageCategory;

        // Cria a ASCollectionView. Não é necessário adiciocar como subnode explicitamente por que iremos configurar o parametro usesImplicitHierarchyManagement para YES
        UICollectionViewFlowLayout *layout = [[UICollectionViewFlowLayout alloc] init];
        _collectionNode = [[ASCollectionNode alloc] initWithCollectionViewLayout:layout];
        _collectionNode.delegate = self;
        _collectionNode.dataSource = self;
        _collectionNode.backgroundColor = [UIColor whiteColor];
    }
    
    return self;
}

... 

// adiciona e calcula automaticamente as dimensoes do DisplayNode com a CollectionView
- (ASLayoutSpec *)layoutSpecThatFits:(ASSizeRange)constrainedSize
{
    return [ASWrapperLayoutSpec wrapperWithLayoutElement:self.collectionNode];
}

...

#pragma mark - ASCollectionDataSource

- (NSInteger)collectionNode:(ASCollectionNode *)collectionNode numberOfItemsInSection:(NSInteger)section
{
    return 10;
}

- (ASCellNodeBlock)collectionNode:(ASCollectionNode *)collectionNode nodeBlockForItemAtIndexPath:(NSIndexPath *)indexPath
{
    NSString *imageCategory = self.imageCategory;
    return ^{
        DetailCellNode *node = [[DetailCellNode alloc] init];
        node.row = indexPath.row;
        node.imageCategory = imageCategory;
        return node;
    };
}

- (ASSizeRange)collectionNode:(ASCollectionNode *)collectionNode constrainedSizeForItemAtIndexPath:(NSIndexPath *)indexPath
{
    CGSize imageSize = CGSizeMake(CGRectGetWidth(collectionNode.view.frame), kImageHeight);
    return ASSizeRangeMake(imageSize, imageSize);
}

~~~

Como podem perceber a `ViewController` ***DetailViewController*** é criada com uma view que contem uma `CollectionView`, esta carrega 10 imagens de uma api em cada célula. A ***DetailCellNode*** é uma CollectionViewCell que é configurada em um bloco carregado em background pelo `ASDK` após as imagens serem carregadas. A `UI` em nenhum momento deixa de ser responsível ou fluida.

## Terminando

Este é um exemplo simples que demonstra como é intuitivo e fácil trabalhar com o `ASDK`, existem muitos tópicos avançados de como melhorar ainda mais a performance. Neste artigo não pudemos falar sobre Layout, Inteligent Preloading e outras conveniências. Ele serve como um start para a avaliação da biblioteca como substituta do UIKit para criar interfaces mais fluidas e responsivas. 

Agradeço a leitura e fico aberto a sugestões e críticas, este foi o meu primeiro artigo de desenvolvimento, e pretendo escrever mais.

Seguem meus contatos:

* [Twitter (@gugaoliveira)](https://twitter.com/gugaoliveira){:target="_blank"} 
* Slack do [iOSDevBR](http://iosdevbr.herokuapp.com/){:target="_blank"} (@gugaoliveira). 

Agradecimentos especiais ao [Heberti Almeida](https://twitter.com/hebertialmeida){:target="_blank"}, que já utiliza o `ASDK` há bastante tempo e me apresentou esta biblioteca fantástica e também seus criadores, foi realmente bacana poder conversar com os criadores e ter uma canal de comunicação direta.

---- 

## Referências 

* [AsyncDisplayKit Getting Starded](http://asyncdisplaykit.org/docs/getting-started.html){:target="_blank"}
* [Using AsyncDisplayKit to Develop Responsive UIs in iOS](http://www.appcoda.com/introduction-asyncdisplaykit-2-0/){:target="_blank"} ***[Ziad Tamim, 29/12/2016]***
* [AsyncDisplayKit 2.0 Tutorial: Getting Started](https://www.raywenderlich.com/124311/asyncdisplaykit-2-0-tutorial-getting-started){:target="_blank"} ***[Luke Parham, 5/12/2016]***
* [Slack do AsyncDisplayKit](http://asyncdisplaykit.org/slack.html){:target="_blank"}

### Referências de Layout
* [AsyncDisplayKit 2.0 Tutorial: Automatic Layout](https://www.raywenderlich.com/124696/asyncdisplaykit-2-0-tutorial-automatic-layout){:target="_blank"} ***[Luke Parham, 19/12/2016]***
* [Layout at Scale with AsyncDisplayKit 2.0](https://www.youtube.com/watch?v=sqkinHYXTuc){:target="_blank"} ***[NSMeetup 2016]***
* [ASStackLayout Game](http://huytnguyen.me/froggy-asdk-layout/){:target="_blank"}
