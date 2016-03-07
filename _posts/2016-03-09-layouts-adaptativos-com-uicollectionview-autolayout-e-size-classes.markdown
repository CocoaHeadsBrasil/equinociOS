---
layout:     post
title:      "Layouts adaptativos com UICollectionView, Size Classes e Auto Layout"
subtitle:   ""
date:       2016-03-09 12:00:00
author:     "Rodrigo Borges Soares"
header-img: "img/nomeDoUsuario/imagem.jpg" (imagem de cabeçalho)
category:   iOS
---

> Meu nome é Rodrigo (mais conhecido como Borges). Sou manaura e moro em São Paulo, depois de uma temporada em BH, onde fiz um mestrado em Computação Ubíqua. Já trabalhei com Maemo, MeeGo, Symbian, Windows Phone e Android. Atualmente sou desenvolvedor iOS na VivaReal e desenvolvedor/co-fundador no Meatless, onde estou tentando reduzir o consumo de carne no mundo  com alguns amigos.

Quando entrei no VivaReal, um dos meus primeiros desafios foi desenvolver a nova página de detalhes do imóvel (PDP, do inglês, Property Details Page). Para isso, eu precisaria utilizar UICollectionView, Auto Layout, Size Classes, capturar eventos de mudança de tamanho da tela, criar um layout específico para o iPad e suportar Multitasking. Um pequeno detalhe: eu nunca tinha trabalhado com nada disso.

O objetivo desse artigo é mostrar que com um esforço relativamente pequeno, você consegue aproveitar ao máximo o tamanho da tela do device do seu usuário (seja qual iDevice for) para criar uma interface bonita e adaptativa.

##Property Details Page

A PDP é um View Controller que mostra detalhes de um imóvel selecionado pelo usuário. Ela é composta por várias seções com informações como preços do aluguel e condomínio, localização, descrição e outros detalhes.

![](../img/borges/viva-ipad1.png)

Toda Collection View possui um objeto do tipo UICollectionViewLayout, que é o responsável por definir o tamanho, posição e outros atributos das células e seções. A PDP antiga utilizava o layout padrão para Collection Views disponibilizado no UIKit, o UICollectionViewFlowLayout. Um dos principais problemas desse layout é não poder posicionar as células de outra forma que não seja um grid, isto é, baseado em linhas e colunas. 

Outro problema que identifiquei foi o tamanho das linhas e das células em uma mesma linha. Geralmente, a altura de uma linha é determinada pela célula de maior altura. Dessa forma, na PDP antiga, haviam vários espaços em branco entre células de linhas diferentes, pois as células costumam variar bastante de tamanho (célula que mostra um preview do mapa e a célula que mostra a descrição, por exemplo).

##Custom Layouts 

Para resolver esse problema, chegamos a uma alternativa: Custom Layouts. Segundo a Apple, Custom Layouts devem ser utilizados se:

O layout não se parece com um grid, um layout baseado em linhas ou suporta scroll em mais de uma direção;
Você precisa mudar a posição das células constantemente.

No nosso caso, percebemos que satisfazíamos ambos os pontos. 

Primeiro, o nosso layout para uma tela maior, como a do iPad, não tem o conceito de linhas, apesar de ter o de colunas. As células deveriam ser posicionadas nas colunas logo abaixo das últimas células adicionadas, sem se importar com o tamanho destas.

Segundo, de acordo com o tamanho da tela, as células da nossa PDP devem ter posições diferentes para facilitar a visualização das informações. Por exemplo, no iPad em portrait, a célula de localização deve vir logo após a célula de resumo de preço e características. Já em landscape, logo abaixo da célula de resumo vem a de descrição. Além disso, enquanto a célula de fotos do imóvel permanece ocupando duas colunas, todas as outras células fazem parte de apenas uma coluna cada.

Todos esses pontos nos levaram a utilizar um Custom Layout para implementar nossa nova PDP.

###Criando sua subclasse da UICollectionViewLayout 

A primeira coisa a se fazer quando implementando um Custom Layout é criar uma subclasse da UICollectionViewLayout. Ela deve implementar 3 métodos principais, que são os responsáveis por indicar à Collection View os tamanhos e posições das células.

- prepareLayout
- collectionViewContentSize
- layoutAttributesForElementsInRect:

###O método prepareLayout 

possui a implementação mais complexa dos 3, pois ele varia de acordo com a complexidade do layout da sua Collection View. Os outros 2 métodos apenas retornam objetos que calculamos no prepareLayout, como veremos mais a frente.

Método prepareLayout No método prepareLayout, deve-se determinar a posição de cada célula. No fim desse método, é preciso ter o mínimo de informação para definir a área total do conteúdo da Collection View (não somente a área visível).

###Método collectionViewContentSize 

Com as informações calculadas no método prepareLayout, é possível definir a área total do conteúdo da Collection View. O retorno do método é um objeto do tipo CGSize.

```swift
override func collectionViewContentSize() -> CGSize { 
    return CGSize(width: contentWidth, height: contentHeight) 
}
```

No nosso caso, à medida que calculamos as posições e tamanhos das células no método prepareLayout, atualizamos duas variáveis globais da classe: contentWidth e contentHeight. Dentro do método collectionViewContentSize() criamos uma CGSize e a retornamos.

###Método layoutAttributesForElementsInRect: 

Esse método é chamado pela Collection View para saber os atributos das células que estão dentro de um retângulo que é mandado como parâmetro. O retorno do método deve ser um Array de UICollectionViewLayoutAttributes.

```swift
override func layoutAttributesForElementsInRect(rect: CGRect) -> [UICollectionViewLayoutAttributes]? { 
    var layoutAttributes = [UICollectionViewLayoutAttributes]() 

    for attributes in attributesCache { 
        if CGRectIntersectsRect(attributes.frame, rect){
            layoutAttributes.append(attributes) 
        }
    } 
    return layoutAttributes 
}
```

Os atributos de cada célula são mais informações que podemos aproveitar e cacheá-las durante a execução do método prepareLayout. Neste método, criamos um Array de UICollectionViewLayoutAttributes e, à medida que definimos as posições das células, vamos adicionando seus atributos no Array.

Feito isso, no layoutAttributesForElementsInRect:, basta iterarmos sobre o Array de atributos e utilizar o método CGRectIntersectsRect para identificar quais células cujos frames intersectam o retângulo passado como parâmetro.

##Mostrando Views de acordo com a Size Class 

Outra característica interessante da PDP é que, em telas menores, o formulário de contato com o anunciante deve ser implementado como última célula da Collection View. Já em telas maiores (iPad), ele deve ficar fixo no lado direito do layout, independente da Collection View.

###Size Classes 

Desde o momento da criação de Size Classes, o desenvolvimento de interfaces universais no iOS deixou de ser dividido em iPhone e iPad para levar em consideração duas classes: Regular e Compact.

Quando você baseia o layout em Size Classes, a interface do seu app pode se comportar bem em qualquer tipo de tela, seja ela de um iPhone, iPhone Plus, de um iPad ou iPad Pro, em portrait ou landscape.

###Modificando constraints para uma Size Class 

No Storyboard é possível alterar a Size Class do layout na barra que fica logo acima da Debug area. 

Inicialmente, a Size Class selecionada é a Any Width/Any Height, o que significa que as constraints e Views configuradas no atual Storyboard serão aplicadas a todas as Size Classes.

Quando mudamos para uma outra Size Class, podemos configurar como nossa interface se comportará em um tamanho específico de tela, sem modificar o seu comportamento no restante das classes.

No nosso caso, queremos que a View de formulário de contato apareça apenas quando o usuário estiver em um tamanho de tela grande como o do iPad. Logo, selecionamos Regular Width/Regular Height como nossa Size Class e modificamos a interface para tornar o formulário visível. No nosso Storyboard, a modificação foi alterar as constraints do Auto Layout que definem o espaço à direita da Collection View e a largura do formulário. Assim, no iPad, a Collection View acompanhará a largura do formulário, se mantendo à esquerda dele, enquanto que, nos outros tamanhos de tela, ela ocupará todo o View Controller.

| Any Width, Any Height  | Regular Width, Regular Height |
------------- | -------------
| ![](../img/borges/viva-storyboard2.png) | ![](../img/borges/viva-storyboard1.png)|

###Instalando uma View de acordo com a Size Class 

Você deve estar se perguntando: O que acontece com a nossa View lateral de formulário no iPhone? Ela é criada e ocupa memória mesmo nunca sendo utilizada lá?

Felizmente o Xcode fornece um meio de configurar se uma View será computada ou não de acordo com a Size Class. Para isso, precisamos apenas selecionar a View em questão no Storyboard, selecionar o Attributes Inspector e, lá no final, adicionar as Size Classes em que a View será "instalada". Dessa forma, a View fará parte da hierarquia de Views do layout apenas nas Size Classes em que aparece, sem gastar recursos do sistema nas outras.

![](../img/borges/viva-storyboard3.png)

##Multitasking 

Quando você desenvolve sua app baseando-se em Size Classes e Auto Layout, ela se adaptará automaticamente durante o uso de Multitasking no iPad. No fim das contas, durante as mudanças de tamanho durante o Multitasking, a app estará apenas trocando de Size Class.

![](../img/borges/viva-ipad3.png)

![](../img/borges/viva-ipad2.png)

Segundo a Apple:

> From a development perspective, the biggest change is about resource management.

Então o maior problema a ser tratado durante o Multitasking é o gerenciamento de recursos e memória. Conceitos como instalação ou não de Views baseados em Size Classes ajudam a utilizar o mínimo de recursos possíveis e compartilhá-los com outras apps.

##Referências 
1. [Projeto Demo](https://github.com/rdgborges/VivaRealPDPExample). Disponibilizado no meu perfil do Github.
2. [UICollectionViews Tutorial](http://www.raywenderlich.com/78550/beginning-ios-collection-views-swift-part-1). Ray Wenderlich.
3. [UICollectionView Custom Layout Tutorial](http://www.raywenderlich.com/107439/uicollectionview-custom-layout-tutorial-pinterest). Ray Wenderlich.
4. [Creating Custom Layouts](https://developer.apple.com/library/ios/documentation/WindowsViews/Conceptual/CollectionViewPGforIOS/CreatingCustomLayouts/CreatingCustomLayouts.html). Apple iOS Developer Library.
5. [About Designing for Multiple Size Classes](https://developer.apple.com/library/ios/recipes/xcode_help-IB_adaptive_sizes/chapters/AboutAdaptiveSizeDesign.html). Apple iOS Developer Library.
6. [Installing and Uninstalling Views for a Size Class](https://developer.apple.com/library/ios/recipes/xcode_help-IB_adaptive_sizes/chapters/EnableAndDisableViews.html). Apple iOS Developer Library.
7. [Adopting Multitasking Enhancements on iPad](https://developer.apple.com/library/prerelease/ios/documentation/WindowsViews/Conceptual/AdoptingMultitaskingOniPad/index.html). Apple iOS Developer Library.