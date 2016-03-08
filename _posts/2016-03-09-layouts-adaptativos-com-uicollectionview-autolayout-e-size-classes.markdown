---
layout:     post
title:      "Layouts adaptativos com UICollectionView, Size Classes e Auto Layout"
date:       2016-03-09 00:00:00
author:     "Rodrigo Borges"
header-img: "img/borges/viva-cover.jpg"
category:   iOS
---

> Meu nome é Rodrigo Borges ([@rdgborges](https://www.twitter.com/rdgborges)). Sou manaura e moro em São Paulo. Passei uma temporada em BH, onde fiz um mestrado em Computação Ubíqua. Já trabalhei com Maemo, MeeGo, Symbian, Windows Phone e Android. Atualmente sou desenvolvedor iOS no [VivaReal](https://itunes.apple.com/br/app/vivareal-imoveis-casas-condominios/id655245656?mt=8) e desenvolvedor/co-fundador no [Meatless](https://itunes.apple.com/us/app/meatless-sao-paulo-sp/id1080837095?l=pt&ls=1&mt=8), onde estou tentando reduzir o consumo de carne no mundo  com alguns amigos.

Quando entrei no VivaReal, um dos meus primeiros desafios foi desenvolver a nova página de detalhes do imóvel (PDP, do inglês, Property Details Page). Para isso, eu precisei utilizar UICollectionView, Auto Layout, Size Classes, capturar eventos de mudança de tamanho da tela, criar um layout específico para o iPad e suportar Multitasking. Um pequeno detalhe: eu nunca tinha trabalhado com nada disso. 🤓👍

O objetivo desse artigo é mostrar que com um esforço relativamente pequeno, você consegue aproveitar ao máximo o tamanho da tela do device do seu usuário (seja qual iDevice for) para criar uma interface bonita e adaptativa.

##Property Details Page

No [app do VivaReal](https://itunes.apple.com/br/app/vivareal-imoveis-casas-condominios/id655245656?mt=8), a PDP é um View Controller que mostra detalhes de um imóvel selecionado pelo usuário. Ela é composta por várias seções com informações como preços do aluguel e condomínio, localização, descrição, formulário para contato e outros detalhes.

<center><b>PDP no iPad</b></center>
![]({{ site.baseurl }}/img/borges/viva-ipad1.png)

<center><b>PDP no iPhone</b></center>
<center><img src="/img/borges/viva-iphone1.png" alt="" /></center>

Toda Collection View possui um objeto do tipo **UICollectionViewLayout**, que é o responsável por definir o tamanho, posição e outros atributos das células e seções. A PDP antiga utilizava o layout padrão para Collection Views disponibilizado no UIKit, o **UICollectionViewFlowLayout**.

> Apesar de ser customizável em vários aspectos, um dos principais problemas desse layout é não poder posicionar as células de outra forma que não seja um grid, isto é, baseado em linhas e colunas. 

Outro problema que identifiquei foi o tamanho das células em uma mesma linha. Geralmente, a altura de uma linha é determinada pela célula de maior altura. Dessa forma, na PDP antiga, haviam vários espaços em branco entre células de linhas diferentes, pois as células costumam variar bastante de tamanho.

##Custom Layouts

Para resolver esse problema, chegamos a uma alternativa: **Custom Layouts**. Segundo a [Apple](https://developer.apple.com/library/ios/documentation/WindowsViews/Conceptual/CollectionViewPGforIOS/CreatingCustomLayouts/CreatingCustomLayouts.html), Custom Layouts devem ser considerados se:

1. O layout não se parece com um grid, um layout baseado em linhas ou suporta scroll em mais de uma direção;
2. Você precisa mudar a posição das células constantemente.

No nosso caso, percebemos que satisfazíamos ambos os pontos. 

- Primeiro, o nosso layout para uma tela maior, como a do iPad, não tem o conceito de linhas, apesar de ter o de colunas. As células deveriam ser posicionadas nas colunas logo abaixo das últimas células adicionadas, sem se importar com o tamanho destas.

- Segundo, de acordo com o tamanho da tela, as células da nossa PDP devem ter posições diferentes para facilitar a visualização das informações. Por exemplo, no iPad em *portrait*, a célula de localização deve vir logo após a célula de resumo de preço e características. Já em *landscape*, a célula de descrição do imóvel é que vem logo abaixo da célula de resumo.

Todos esses pontos nos levaram a utilizar um Custom Layout para implementar nossa nova PDP. 👌

###Criando a subclasse da UICollectionViewLayout

A primeira coisa a se fazer quando implementando um Custom Layout é criar uma **subclasse** da UICollectionViewLayout. Ela deve implementar 3 métodos principais, que são os responsáveis por indicar à Collection View os tamanhos e posições das células.

- prepareLayout
- collectionViewContentSize
- layoutAttributesForElementsInRect:

O método prepareLayout possui a implementação mais complexa dos 3, pois ele varia de acordo com a complexidade do layout da sua Collection View. Os outros 2 métodos apenas retornam objetos que calculamos no prepareLayout, como veremos mais a frente.

###Método prepareLayout

No método prepareLayout, deve-se determinar a posição de cada célula. No fim desse método, é preciso ter o mínimo de informação para definir a área total do conteúdo da Collection View (não somente a área visível).

Criamos algumas variáveis globais do layout para nos auxiliar e evitar recalcular o layout a cada pequena interação do usuário com a Collection View:

- **attributesCache**: Array que armazena os atributos do layout de cada célula;
- **contentWidth** e **contentHeight**: Armazenam a largura e altura do conteúdo da Collection View. São atualizados sempre que a posição de uma célula é determinada.

~~~swift
override func prepareLayout() {

    let numberOfColumns = columnsBasedOnScreen()
    attributesCache = []
    contentHeight = 0.0

    // 1    
    // Initializes offsets arrays
    let columnWidth = contentWidth / CGFloat(numberOfColumns)
    var yOffset = [CGFloat](count: numberOfColumns, repeatedValue: 0)

    var xOffset = [CGFloat]()
    for column in 0 ..< numberOfColumns {
        xOffset.append(CGFloat(column) * columnWidth)
    }

    var column = 0

    for item in 0 ..< collectionView!.numberOfItemsInSection(0) {
        // 2    
        // Calculates cell frame
        let indexPath = NSIndexPath(forItem: item, inSection: 0)
        let itemHeight = delegate.heightForItemAtIndexPath(indexPath, withWidth: columnWidth)

        var frame: CGRect
        // 3
        if item == 0 {
            // if first cell, cell width == content width
            frame = CGRect(x: xOffset[column], y: yOffset[column], width: contentWidth, height: itemHeight)
        } else {
            // cell width == column width
            frame = CGRect(x: xOffset[column], y: yOffset[column], width: columnWidth, height: itemHeight)
        }

        // 4    
        // Append cell attributes to cache
        let attributes = UICollectionViewLayoutAttributes(forCellWithIndexPath: indexPath)
        attributes.frame = frame
        attributesCache.append(attributes)

        // 5    
        // Updates contentHeight
        contentHeight = max(contentHeight, CGRectGetMaxY(frame))

        // 6
        // Updates y position of next cell
        if item == 0 {
            for i in 0..<yOffset.count {
                yOffset[i] = itemHeight
            }
        } else {
            yOffset[column] = yOffset[column] + itemHeight
        }

        // 7
        // Determines column where next cell will be placed
        if item == 0 {
            column = 0
        } else {
            column = column >= (numberOfColumns - 1) ? 0 : ++column
        }

    }
}
~~~


1. Inicialização dos Arrays xOffset e yOffset. A partir desses Arrays, calculamos a posição de cada célula na Collection View. Eles são atualizados cada vez que a posição de uma célula é calculada para sabermos onde posicionar a próxima célula da lista;
2. Cálculo do frame da célula baseado na largura da coluna e na sua altura;
3. Caso o item seja o primeiro (célula das fotos), a largura é igual a largura da Collection View. Caso contrário, é igual a largura da coluna;
4. Após o frame da célula ser definido, o objeto de atributos da célula é adicionado ao cache. Este será utilizado posteriormente no método layoutAttributesForElementsInRect:;
5. Atualiza-se a altura da área de conteúdo da Collection View, o qual será utilizada no método collectionViewContentSize;
6. Atualiza a posição em que a próxima célula da coluna será posicionada, de acordo com a altura e posição da célula atual.
7. Determina a coluna onde a próxima célula será colocada;

Para definir o posicionamento das células, decidimos criar um método que analisa a **orientação** do device e **Size Class** da tela e retorna o número de colunas em que o layout será dividido.

~~~swift
func columnsBasedOnScreen() -> Int {
    let orientation = UIApplication.sharedApplication().statusBarOrientation
    if orientation == UIInterfaceOrientation.Portrait || orientation == UIInterfaceOrientation.PortraitUpsideDown {
        // portrait
        return 1

    } else {
        // landscape
        let horizontalSizeClass = self.collectionView?.traitCollection.horizontalSizeClass
        let verticalSizeClass = self.collectionView?.traitCollection.verticalSizeClass

        if horizontalSizeClass == UIUserInterfaceSizeClass.Regular && verticalSizeClass == UIUserInterfaceSizeClass.Regular {
            let applicationDelegate = UIApplication.sharedApplication().delegate

            guard let delegate = applicationDelegate else {
                return 1
            }

            guard let window = delegate.window else {
                return 1
            }

            let isFullscreen = CGRectEqualToRect(window!.frame, window!.screen.bounds)

            if isFullscreen {
                return 2
            } else {
                return 1
            }
        } else {
            return 1
        }

    }
}
~~~

- Caso o device esteja em portrait, o layout da Collection View sempre será baseado em 1 coluna.
- Caso esteja em landscape, as Size Classes horizontais e verticais são verificadas:
    - Se são ambas Regular, significa que estamos em uma tela grande, como o iPad. Nesse caso, precisamos checar também se o usuário está utilizando Multitasking ou não. Isso é feito facilmente comparando o window.frame com o window.screen.bounds.
    - Caso contrário, estamos em uma tela menor, e, por isso, utilizamos o layout normal baseado em 1 coluna.

###Método collectionViewContentSize

Com as informações calculadas no método prepareLayout, é possível definir a área total do conteúdo da Collection View. O retorno do método é um objeto do tipo CGSize.

~~~swift
override func collectionViewContentSize() -> CGSize {
    return CGSize(width: contentWidth, height: contentHeight)
}
~~~

No nosso caso, à medida que calculamos as posições e tamanhos das células no método **prepareLayout**, atualizamos duas variáveis globais da classe: **contentWidth** e **contentHeight**. Dentro do método collectionViewContentSize() criamos uma CGSize e a retornamos.

###Método layoutAttributesForElementsInRect:

Esse método é chamado pela Collection View para saber os atributos das células que estão dentro de um retângulo passado como parâmetro. O retorno do método deve ser um **Array de UICollectionViewLayoutAttributes**.

~~~swift
override func layoutAttributesForElementsInRect(rect: CGRect) -> [UICollectionViewLayoutAttributes]? {
    var layoutAttributes = [UICollectionViewLayoutAttributes]()

    for attributes in attributesCache {
        if CGRectIntersectsRect(attributes.frame, rect){
            layoutAttributes.append(attributes)
        }
    }
    return layoutAttributes
}
~~~

Os atributos de cada célula são informações que também podemos calcular e armazenar durante a execução do método **prepareLayout**. Neste método, criamos um Array de UICollectionViewLayoutAttributes e, à medida que definimos as posições das células, vamos adicionando seus atributos no Array.

Feito isso, no layoutAttributesForElementsInRect:, basta iterarmos sobre o Array de atributos e utilizar o método CGRectIntersectsRect para identificar quais células cujos frames intersectam o retângulo passado como parâmetro.

###Definindo o novo layout da Collection View

Uma vez que temos o nosso Custom Layout implementado, precisamos setá-lo como layout da nossa Collection View. Para isso, bastamos selecioná-la no Storyboard, abrir o Attributes Inspector, selecionar o tipo do layout como **Custom** e a classe do layout.

![]({{ site.baseurl }}/img/borges/viva-storyboard5.png)


###Atualizando o layout após a transição de tamanho de tela

Precisamos avisar a collection View para recalcular a posição das células e área do conteúdo toda vez que houver uma mudança de tamanho ou orientação da tela.

Para isso, basta-se fazer um override do método **viewWillTransitionToSize** da nossa View Controller e chamar o método **invalidateLayout** no layout da nossa Collection View. Assim, o método **prepareLayout** será chamado novamente, atualizando todas as informações de layout e posicionamento das células.

~~~swift
override func viewWillTransitionToSize(size: CGSize, withTransitionCoordinator coordinator: UIViewControllerTransitionCoordinator) {
    super.viewWillTransitionToSize(size, withTransitionCoordinator: coordinator)

    self.collectionView.collectionViewLayout.invalidateLayout()
    self.collectionView.reloadData()
}
~~~

##Mostrando Views de acordo com a Size Class

Outra característica interessante da PDP é que, em telas menores, o formulário de contato com o anunciante deve ser implementado como última célula da Collection View. Já em telas maiores (iPad), ele deve ficar fixo no lado direito do layout, independente da Collection View.

![]({{ site.baseurl }}/img/borges/viva-ipad4.png)

###Size Classes

Desde a criação das [Size Classes](https://developer.apple.com/library/ios/recipes/xcode_help-IB_adaptive_sizes/chapters/AboutAdaptiveSizeDesign.html), o desenvolvimento de interfaces universais no iOS deixou de ser dividido em iPhone e iPad para levar em consideração duas classes: **Regular** e **Compact** combinadas com as duas dimensões: **Width** e **Height**.

Cada combinação possível entre elas define um tipo de tamanho de tela, levando em conta também a orientação do device. Por exemplo, **Compact Height** define a tela dos iPhones em landscape. Já **Regular Width/Regular Height** define a tela dos iPads, seja em portrait ou landscape.

Quando você baseia o layout em Size Classes, a interface do seu app pode se comportar bem em qualquer tipo de tela, seja ela de um iPhone, iPhone Plus, de um iPad ou iPad Pro, em portrait ou landscape.

###Modificando constraints para uma Size Class

No Storyboard é possível alterar a Size Class do layout na barra que fica logo acima da Debug area. Inicialmente, a Size Class selecionada é a Any Width/Any Height, o que significa que as constraints e Views configuradas no atual Storyboard serão aplicadas a todas as Size Classes.

<center><img src="/img/borges/viva-storyboard4.png" alt="" /></center>

Quando mudamos para uma outra Size Class, podemos configurar como nossa interface se comportará neste tamanho específico da tela, sem modificar o seu comportamento no restante das classes.

No nosso caso, queremos que a View de formulário de contato apareça apenas quando o usuário estiver em um tamanho de tela grande como o do iPad. Logo, selecionamos Regular Width/Regular Height como nossa Size Class e modificamos a interface para tornar o formulário visível.

A modificação foi alterar as **constraints do Auto Layout** que definem o espaço à direita da Collection View e a largura do formulário. Assim, no iPad, a Collection View acompanhará a largura do formulário, se mantendo à esquerda dele, enquanto que, nos outros tamanhos de tela, ela ocupará toda a extensão do View Controller.

| Any Width, Any Height  | Regular Width, Regular Height |
------------- | -------------
| ![]({{ site.baseurl }}/img/borges/viva-storyboard2.png) | ![]({{ site.baseurl }}/img/borges/viva-storyboard1.png)|

###Instalando uma View de acordo com a Size Class

Você deve estar se perguntando: O que acontece com a nossa View lateral de formulário no iPhone? 🤔 Ela é criada e ocupa memória mesmo nunca sendo utilizada lá? 😱

Felizmente o Xcode fornece um meio de configurar se uma View será computada ou não de acordo com a Size Class. Para isso, precisamos apenas selecionar a View em questão no Storyboard, selecionar o Attributes Inspector e, lá no final, adicionar as Size Classes em que a View será "instalada". Dessa forma, a View fará parte da hierarquia de Views do layout apenas nas Size Classes em que aparece, sem gastar recursos do sistema nas outras.

<center><img src="/img/borges/viva-storyboard3.png" alt="" /></center>

O formulário é instalado apenas quando em Regular Width/Regular Height. Já o botão de "Contatar anunciante", que aparece na parte inferior da tela, é instalado em todas as Size Classes, exceto nas Regular.

##Multitasking

Quando você desenvolve sua app baseando-se em Size Classes e Auto Layout, ela se adaptará automaticamente durante o uso de Multitasking! 🎉

No fim das contas, durante as mudanças de tamanho no Multitasking, a app estará apenas trocando de Size Class. Como sua Collection View está escutando as mudanças de Size Class, recalculando seu layout e suas Views estão sendo instaladas conforme a Size Class, não precisa fazer mais nada para adaptar seu layout.   

![]({{ site.baseurl }}/img/borges/viva-ipad3.png)

![]({{ site.baseurl }}/img/borges/viva-ipad2.png)

Segundo a [Apple](https://developer.apple.com/library/prerelease/ios/documentation/WindowsViews/Conceptual/AdoptingMultitaskingOniPad/index.html):

> From a development perspective, the biggest change is about resource management.

Então o maior problema a ser tratado durante o Multitasking é o gerenciamento de recursos e memória. Conceitos como instalação ou não de Views baseados em Size Classes ajudam a utilizar o mínimo de recursos possíveis e compartilhá-los com outras apps.

##Conclusão

Vimos aqui como foi implementada a PDP do app do VivaReal, onde o desafio foi criar a melhor experiência pro usuário que o tamanho de tela do seu device permitisse.

Utilizamos UICollectionViews e Custom Layouts para definir a lógica de criação do nosso layout, nos baseando nas Size Classes para saber como posicionar as células e quais Views aparecem ou não em cada tamanho de tela.

Não deixe de ler e compartilhar também os outros artigos do EquinociOS, uma excelente iniciativa da comunidade do [CocoaHeads Brasil](http://www.cocoaheads.com.br)! 👏🤓

##Referências
1. [Projeto Demo](https://github.com/rdgborges/VivaRealPDPExample). Disponibilizado no meu perfil do Github.
2. [UICollectionViews Tutorial](http://www.raywenderlich.com/78550/beginning-ios-collection-views-swift-part-1). Ray Wenderlich.
3. [UICollectionView Custom Layout Tutorial](http://www.raywenderlich.com/107439/uicollectionview-custom-layout-tutorial-pinterest). Ray Wenderlich.
4. [Creating Custom Layouts](https://developer.apple.com/library/ios/documentation/WindowsViews/Conceptual/CollectionViewPGforIOS/CreatingCustomLayouts/CreatingCustomLayouts.html). Apple iOS Developer Library.
5. [About Designing for Multiple Size Classes](https://developer.apple.com/library/ios/recipes/xcode_help-IB_adaptive_sizes/chapters/AboutAdaptiveSizeDesign.html). Apple iOS Developer Library.
6. [Installing and Uninstalling Views for a Size Class](https://developer.apple.com/library/ios/recipes/xcode_help-IB_adaptive_sizes/chapters/EnableAndDisableViews.html). Apple iOS Developer Library.
7. [Adopting Multitasking Enhancements on iPad](https://developer.apple.com/library/prerelease/ios/documentation/WindowsViews/Conceptual/AdoptingMultitaskingOniPad/index.html). Apple iOS Developer Library.
