---
layout:     post
title:      "Experiências na modularização de um projeto de um SDK"
date:       2017-03-11 00:00:00
author:     "Tales Pinheiro"
category:   "arquitetura"
---

> Tales Pinheiro ([@talesp](https://twitter.com/talesp){:target="_blank"}) é mestre em computação pelo IME-USP, trabalhou por 8 anos com backend bancário programando principalmente em C, quando em 2007 (antes do anúncio do primeiro iPhone!) resolveu aprender Objective-C. Atualmente é um dos lideres do capitulo iOS na Concrete Solutions, onde continua em um projeto Objective-C, mas ajudando outros em Swift

# A história inicial

Nos último pouco mais de dois anos, a consultoria em que trabalho vem desenvolvendo diversos aplicativos para uma das maiores empresas do Brasil. Nós fomos inicialmente responsáveis por reescrever como código nativo e melhorar os dois principais aplicativos desse cliente, pois as versões iniciais eram desenvolvidas numa tecnologia hibrida.

Para facilitar a nossa vida e por questões de segurança (menos pessoas conhecendo áreas criticas do cliente), resolvemos criar um SDK/Framework com as partes comuns aos aplicativos, incluindo nele as partes comuns aos diversos aplicativos.

Os projetos cresceram, o SDK cresceu, os times aumentaram em tamanho e em número e, é claro, a complexidade aumentou em todas as pontas. Precisavamos novamente simplificar a nossa vida.

Eu passei um tempo em um time responsável por manter o funcionamento do fluxo e desenvolvimento, que garante a coisas como documentação de _coding style_, cobertura de código, qualidade de código, _merges_ solicitados pelos diversos times, entre outras coisas da qualidade geral do projeto, de um dos aplicativos (temos outro time responsável pelo menos no outro grande aplicativo desse cliente), mas atualmente estou no time do SDK.

Esse é um projeto menor, tocado por menos pessoas, mas bastante complexo, inicialmente desenvolvido por apenas um time, mas que já engloba dois (o meu time e um time de outra consultoria, especializada em segurança), com um terceiro entrando em breve.

Por questões estratégicas, normalmente clientes desse tamanho não querem/podem depender de uma única consultoria, e ao longo do tempo algumas consultorias foram adicionadas ao projeto.

Com o aumento da complexidade dos projetos e da quantidade de pessoas e empresas - com culturas e opiniões diferentes sobre coisas como arquitetura dos projetos - algo precisava ser feito para manter a sanidade dos times. Modularizar apenas com um SDK comum aos projetos não era suficiente.

Além disso, precisei fazer uma prova de conceito usando o SDK e esbarrei em um problema: ele depende do `UIKit`. Por ser uma base de código menor, resolvemos modularizar ainda mais, começando pelo SDK.

## Cenário inicial do SDK

Como disse antes, o SDK é comum a vários apps gerenciados pelo meu time (a parte _core_) com outro time de outra consultoria desenvolvendo pequenas bibliotecas da parte de segurança do aplicativo, que são enviadas para nós como bibliotecas que são _linkadas_ estaticamente ao nosso SDK. Esporadicamente algumas pessoas direto do cliente ainda fazem um _bypass_ no nosso time/processo em pequenas partes do projeto, para adicionar uma _feature_ para um dos projetos principais, mas esquecendo do outro, causando _bug_ (identificado rapidamente pelos times do outro aplicativo ainda em desenvolvimento). E está entrando um time de outra consultoria para mexer no SDK. 

Os aplicativos atualmente suportam o iOS 7, e isso exige que a distribuição do SDK seja como biblioteca para linkagem estática. Já temos no cliente aplicativos sendo desenvolvidos com suporte apenas ao iOS 9+, sendo escrito em Swift, mas como dependências em Swift devem ser _frameworks_ dinâmicos, esse projeto teve inicialmente que usar CocoaPods para adicionar o SDK estaticamente, e o Carthage _linkando_ dinamicamente as dependências de terceiros escritas em Swift, mas o time de _devops_ do cliente barrou o Carthage - por enquanto, apenas CocoaPods pode ser usado nesse cliente.

Como disse anteriormente, o SDK atual compreende então três grandes áreas, todas distribuídas como um "framework" - não no sentido de _framework_ dinâmico criado pelo Xcode, mas numa estrutura similar criada via script, que embute os _assets_ (_storyboards_, `XIB`s e imagens) e funcionalidades das três áreas abaixo:

- Comunicação
- Segurança
- UI Comum aos projetos

Além disso, o time do SDK mantém um projeto Sample, para que os times dos apps saibam como usar o SDK. Esse sample tem o código-fonte aberto internamente, mas o SDK é distribuído compilado.


## SDK "atual"

![center]({{ site.baseurl }}/img/talesp/iphone-sample-ios.png)

Além dos problemas acima, esbarramos em um novo recentemente. Um grande redesign está ocorrendo nos apps, melhorando o design e a usabilidade geral dos apps, mas que não serão lançados ao mesmo tempo. Como o SDK também inclui telas, agora telas antigas que deverão ter suporte e novas em desenvolvimento, uma solução paleativa foi criada: criar um _proxy_ que determina qual a interface usada. Então, num exemplo simples, se tinhamos uma classe/tela `LoginViewController`, tivemos que adicionar a "nova" `LoginViewControllerNew`, e o proxy determina quem será usado. Além de duplicar classes, temos também _assets_ duplicados. E como pretendemos (pelo menos o time de SDK está se adiantando para isso, ainda não tem previsão de desenvolvimento) flexibilizar o SDK, poderia dar à área de negócios a decisão de lançar para outras plataformas Apple (watchOS, tvOS, macOS), hoje estamos intimamente "presos" ao UIKit.

# Nossa proposta

Pensamos então em separar os três componentes em 3 bibliotecas:

- _core_: menor base de código e mais simples de manutenção;
- segurança: poderá ser mantida por outro time/consultoria;
- UI: com versionamento do design simplificado, podendo criar novas "versões" com interfaces para macOS e watchOS;

Para iniciar a modularização, resolvemos focar em separar inicialmente apenas a parte de UI, que agora passará a ter duas versões _major_: a versão `1.0.0` seria a versão atual, e a versão `2.0.0` será a versão do redesign. 

Futuramente pretendemos modularizar as maiores aplicações, o que eliminará o _merge hell_, vai facilitar a verificação de pontos críticos e _feature_ problemáticas, além de uma melhor granulação dos indicadores da cobertura de qualidade, facilitando a manutenção, administração e evolução do produto.

Nossa proposta então é tornar o SDK independente da UI, podendo ser criado um aplicativo _Sample_ contendo apenas as funcionalidades "_core_", como abaixo:

![]({{ site.baseurl }}/img/talesp/proposed-arch-core.png)

Ou um Sample que inclui a interface, com a biblioteca de UI tendo a biblioteca _core_ como dependência via CocoaPods.
![]({{ site.baseurl }}/img/talesp/proposed-arch-full.png)


# Primeiro passo: Separar a UI

O processo geral - já efetuado - pode ser descrito de forma bastante simples. Foi:

1. Criado um novo projeto, que contém apenas código de UI e _assets_
2. Movido código de UI para novo projeto (mas inicialmente não foi movido o código de testes de UI)
3. Removido TODOS os arquivos de UI do projeto _core_ e eliminada a dependência do UIKit
4. Removido Sample (que está se tornando um novo "app", já em desenvolvimento) do _workspace_ do projeto do SDK.

Mas temos aqui um problema:

Algumas telas incluídas no _core_ são chamadas por métodos do _core_, não pelos apps principais. Por exemplo, alguns erros de comunicação já são exibidos para o usuário via chamada do _core_. Mas o _core_ não pode depender de UI. Além disso, para times que usam a nova biblioteca de UI, desejamos que a comunicação deve ser transparente e sem configuração.

Como ter duas bibliotecas se comúnicando, com o _core_ chamando métodos de UI sem saber da existência desses ou do UIKit?


# Protocolos, ao resgate

Na nossa solução, a biblioteca _core_ define alguns protocolos de exibição de interface, que são então adotados pela biblioteca de UI - que tem o _core_ como dependência.

Mas queriamos que a biblioteca de UI fosse apenas adicionada ao Podfile dos aplicativos. Não queria ter que adicionar ao SDK a responsabilidade de registrar as classes de UI, nem ao desenvolvedor dos aplicativos chamar algum método de configuração. Alguns truqes de _runtime_ do Objective-C (válidos em Swift se classe herda de `NSObject`) podem nos ajudar com isso. Mas antes...


# Pequeno desvio: vamos falar sobre inicialização de objetos, bibliotecas e _frameworks_

Para melhor entender a solução, vou explicar um pouco dobre o processo de inicialização de bibliotecas e de objetos em Objective-C.

## Um pouco sobre inicialização de objetos

Praticamente toda classe da Foundation e do UIKit herda de `NSObject` (com exceção de algumas que herdam de `NSProxy`, mas não vem ao caso para nossa solução). E classes que herdam de `NSObject` podem implementar dois métodos que podem ser bastante úteis nesse problema. Para entender melhor os dois, recomendo altamente o artigo [Friday Q&A 2009-05-22: Objective-C Class Loading and Initialization](https://www.mikeash.com/pyblog/friday-qa-2009-05-22-objective-c-class-loading-and-initialization.html), do Mike Ash (inclusive, recomendo fortemente o blog dele como um todo. ele está meio parado, mas o conteúdo é riquissimo).

- `+load`
	- esse método é chamado quando biblioteca é carregada em memória. Se implementar esse método numa classe de seu aplicativo, o método será chamado na inicialização do aplicativo, mesmo que a classe nunca seja referenciada durante o ciclo de vida do aplicativo. Por exemplo, se você tem uma classe que cuida de impressão, mas o usuário nunca mandar imprimir nada, e você implementar esse método nessa classe, ainda sim o método será chamado.
	- Se o método é implementado em uma biblioteca de terceiros, depende do tipo de _linkagem_ da biblioteca com seu aplicativo:
		- linkagem estática: na inicialização do aplicativo
		- linkagem dinâmica: na primeira chamada de um método da biblioteca
- `+initialize`
	- Esse método é um pouco mais _lazy_, e só é chamado quando primeiro método da classe é chamado. Assim, se durante a vida da aplicação, uma referência qualquer - chamada de método de classe, instância ou algo como `NSStringFromClass` ou `NSClassFromString` - não for chamada, esse método não é chamado. Além disso, é garantido pelo runtime que esse método será chamado apenas uma vez por class, na primeira referência da mesma por terceiros.

O método `+load` tem ainda uma característica especial em relação ao _runtime_: se a classe e uma ou mais categorias implementar o método `+load`, todos serão executados


### Observação sobre `initialize` em Swift - Xcode 8.3 beta 3

O documento de _Release Notes_ do Xcode 8.3 beta 3 inclui a seguinte observação.

> Swift will now warn when an NSObject subclass attempts to override the class `initialize` method. Swift doesn't guarantee that references to class names trigger Objective-C class realization if they have no other side effects, leading to bugs when Swift code attempts to override initialize. (28954946)

É bom tomar cuidado com isso, caso esteja usando Swift e pensar em uma implementação parecida

E já que temos falado um pouco sobre _linkagem_ estática e dinámica, e sobre bibliotecas e _frameworks_, vamos explicar um pouco sobre elas.

## "Linkagem" estática vs dinâmica

_Linkar_ uma biblioteca ou _framework_ ao seu projeto funciona como a forma mais básica de gerenciamento de dependências e distribuição/reutilização de código - seu ou de terceiros - em multiplos projetos. Mas a forma como você _linka_ a biblioteca ou framework impacta no tamanho do executável, como a biblioteca ou _framework_ é distribuído e em que momento eles são carregados em memória e seu conteúdo é executado.


### _Linkagem_ estática

Vamos começar explicando a _linkagem_ estática. Nela, o conteúdo executável da biblioteca ou _framework_ que você está _linkando_ ao seu projeto é anexado diretamente ao binário final. No caso de um aplicativo iOS, por exemplo, não é ao arquivo `meuapp.ipa`, mas ao executável embutido dentro do arquivo ipa. Para quem não sabe, o arquivo ipa é na verdade um arquivo comprimido, contendo seu aplicativo, informações da App Store e de _code signing_. Você pode alterar a extensão para `.zip`, descomprimir e abrir a pasta `Payload`, e lá estará seu aplicativo em um pacote com a extensão `.app`. Se clicar com o direito, e selecionar a opção "Mostrar conteúdo do pacote", vai encontrar dentro desta pasta coisa como _assets_, uma pasta chamada `_CodeSignature` com a assinatura/hash dos arquivos embutidos no pacote (para anti-tampering, e evitar que arquivos sejam modificados embutindo código malicioso) e seu binário efetivamente, no formato de executável Unix.

Assim, _linkar_ estáticamente uma biblioteca ou framework (falaremos um pouco mais sobre este daqui a pouco) ao seu projeto deixa este executável maior, consequentemente com maior tempo de inicialização do aplicativo em memória.
Mas até o iOS 8, esta era a única forma de _linkar_ código de terceiro ao seu projeto

Na imagem abaixo podemos ver que o _linker_ pega os diversos arquivos compilados e as diversas bibliotecas que você inclui no seu projeto e gera um binário final. Esse binário contém seu código e as bibliotecas compiladas,e carrega tudo no _heap_ de memória durante a execução
![]({{ site.baseurl }}/img/talesp/mono-static.png)

### Linkagem dinamica

A segunda forma  de _linkar_ código de terceiro ao seu projeto é a dinâmica: Nesse, nenhum código ou conteúdo executável é anexado diretamente ao binário principal. O _linker_ apenas adiciona referências ao conteúdo executável da biblioteca dinâmica, como endereços de memória e referência ao arquivo onde o executável deve buscar o código objeto e conteúdo executável.

Assim, a principio, carregar o aplicativo em memória é mais rápido, pois a biblioteca é carregada dinamicamente quando um de seus simbolos é requisitado. No iOS, watchOS e tvOS não é possível, mas no macOS você pode distribuir uma nova versão da biblioteca sem ter que distribuir uma nova versão do aplicativo, já que o novo código é buscado/carregado na nova inicialização do aplicativo. Além disso, as bibliotecas dinâmicas possuem ciclo de vida e processo de inicialização e finalização/limpeza de memória própios.

Mas não é porque o binário final do aplicativo fica menor que podemos sair adicionando diversas bibliotecas ao projeto no iOS: no processo de inicialização, a assinatura do seu aplicativo e de todas as bibliotecas é verificada, o que é um processo também demorado.

Na imagem abaixo podemos ver que, diferente da _linkagem_ estática, aqui apenas a referência as bibliotecas dinâmicas é adicionada ao seu aplicativo, e que as bibliotecas são carregadas, durante a execução, na pilha de memória, e não no _heap_ junto com o aplicativo.
![]({{ site.baseurl }}/img/talesp/mono-dynamic.png)

## Bibliotecas (_libraries_) vs Arcabouços (_Frameworks_)

Temos falado aqui de bibliotecas, mas e os _frameworks_?

De forma mais técnica, uma biblioteca é um arquivo em formato binário [Mach-O](https://developer.apple.com/library/content/documentation/DeveloperTools/Conceptual/MachOTopics/0-Introduction/introduction.html), que dependendo da forma de _linkagem_, é carregado na inicialização do aplicativo ou dinamicamente de acordo com a necessidade.

Bibliotecas dinâmicas evitam a repetição/cópia de código entre a aplicação e extensões - como extensões de teclado, _Today extenions_ e da Siri - pois o binário existe em apenas um local - com a linkagem estática, o conteúdo da biblioteca é copiado dentro de cada extensão. 

Ao compilar um ou mais arquivos fontes, cada um gera um arquivo `*.o`, chamado de arquivo objeto, que contém o código executável daquele arquivo. Uma biblioteca é um recipiente para um conjunto de arquivos objetos. Bibliotecas estáticas usam a extensão `.a`, gerado ao `ar`quivar um conjunto de arquivos objeto, e bibliotecas dinâmicas possuem a extensão `.dylib` (aqui vale uma nota, lembrada pelo [Ronaldo F. Lima](https://medium.com/@ronaldolima) durante o processo de revisão - `dylib` é usada apenas pelo [Darwin](https://pt.wikipedia.org/wiki/Darwin_(sistema_operacional)), base BSD do macOS; outras vasriações de Unix e o Linux costumam usar a extensão `.so`, de _shared object_).

Uma característica do processo de _linkagem_ é que na compilação o _linker_ só pode usar arquivos objeto de uma arquitetura. Por isso existem dois "formatos" de biblioteca estática, chamados de _container_:

1. arquivos objeto de mesma arquitura em único `ar`_chive_
2. binário _fat_ Mach-O, criado com comando `lipo`

Com o comando `lipo` é possível pegar diversos arquivos de biblioteca com os mesmo arquivos objetos mas em diferentes arquiteturas, e incluir tudo em um único "binário gordo", facilitando a distribuição.

Já os _frameowrks_ são análogos a bibliotecas, mas funcionam como um pacote, onde é possível incluir _assets_ (sons, imagens, vídeos, fontes, etc), _storyboards_, arquivos `NIB` (a versão compilada do `XIB`) entre outras coisas. Frameworks dinâmicos pode também ser [versionados](https://developer.apple.com/library/content/documentation/MacOSX/Conceptual/BPFrameworks/Concepts/VersionInformation.html#//apple_ref/doc/uid/20002255-BCIECADD)

Infelizmente o Xcode não possui um _template_ para criação de bibliotecas estáticas, mas é possível criá-las via _script_, sendo assim possível distribuir o nosso SDK como um _framework_, mas para linkagem estática, distribuindo assim um conjunto as imagens e storyboards comuns aos aplicativos. De modo geral, para distribuir os _assets_ é necessário adicionar um _taget_ do tiplo _Bundle_ ao seu projeto - mas aqui, uma atenção: mesmo que seu projeto seja para iOS, este deve ser um target "__OS X__", e para criar o _framework_ deve ser criada uma estrutura especial de pastas, com uma pasta raiz com a "extensão" `.framework`. Para mais detalhes de como criar um frameowrk estático, veja [esse link](https://github.com/kstenerud/iOS-Universal-Framework).

# Voltando a programação normal

Nosso SDK _core_ consiste agora de algumas classes _singleton_, modelos, _categories_, protocolos e constantes, além das bibliotecas de segurança mantidas por terceiros. Essas classes podem ser acessadas diretamente dos aplicativos ou como direto pelo SDK de UI, já que este também - intencionalmente - depende da biblioteca _core_.

Como a biblioteca _core_ é garantida de já estar em memória quando a biblioteca de UI é inicializada - e ambas são inicializadas durante a inicialização do aplicativo por causa da _linkagem_ estática - o que fazemos é definir um protocolo no SDK _core_, adotar e implementar esse protocolo na biblioteca de UI.

Vamos ver um exemplo de como isso funciona. Um exemplo de conteúdo de interface que é disparado pelo _core_ são alertas e erros - de conexão ou do _backend_. Assim, tínhamos no _core_ - agora na biblioteca de UI - classes de _view_ ou _view controller_ para exibir essas telas. Mas o _core_ não sabe mais essas classes - e nem deve ser responsabilidade dele saber (agora temos a flexibilidade de alterar todo o fluxo) - o que fazemos é declarar um protocolo com um método responsável por exibir a interface. Por exemplo, podemos declarar na biblioteca _core_ o protocolo `TPAAlertDelegate` como abaixo:

~~~objectivec
@protocol TPAAlertDelegate

+ (void)presentInterfaceWithTitle:(NSString *) title message:(NSString *)message;

@end
~~~

E, por exemplo, na interface da classe de rede - _singleton_ - que inicialmente mandava exibir a _view_ ou _view controller_ declaramos uma variável que armazenará a referência fraca para a classe que exibirá efetivamente a interface.

~~~objectivec
@interface TPANetworkManager: NSObject
+ (instancetype)sharedManager;

@property (nonatomic, weak) Class<TPAAlertDelegate>alertInterfaceDelegate;
@end
~~~

Perceba aqui que o tipo declarado é `Class<TPAAlertDelegate>` e não `id<TPAAlertDelegate>` como normalmente fazemos. Já explicamos por quê.

Agora digamos que a interface a ser exibida é uma _view controller_ chamada aqui de `TPAAlertViewController`. Para fazer as duas interagirem de forma transparente e sem a interação/configuração do desenvolvedor dos aplicativos - já que esses são SDKs desenvolvidos e mantidos pelo time de SDK, eu gostaria de evitar jogar essa responsabilidade para outro - implementamos o método `+load` na classe `TPAAlertViewController` como a seguir.

~~~objectivec
@implementation TPAAlertViewController

+ (void)load {
	[TPANetworkManager sharedManager].alertInterfaceDelegate = self;
}
@end
~~~

Aqui, `self` está referenciando a classe `TPAAlertViewController`, e não a uma instância desta, já que esse método é chamado, como dissemos quando a biblioteca que contém esta classe está sendo carregada em memória - ou seja, na inicialização do aplicativo. Por isso a _property_ `alertInterfaceDelegate` deve ser declarada como `Class<TPAAlertDelegate>`. Além disso, como a biblioteca de UI depende da biblioteca _core_, já temos a classe `TPANetworkManager` carregada na pilha de memória, e o _singleton_ pode ser carregado/inicializado normalmente e configurado sem intervenção de quem "consome" esses SDKs.

Com tudo configurado podemos, na implementação do método do SDK _core_, que anteriormente inicializava uma _view_ ou _view controller_, no caso de um erro de requisição, podemos chamar o método do _delegate_.

~~~objectivec
@implementation TPANetworkManager
- (void)someMethodCalledByMainApp {
	[AFNetworking request...
		if (error != nil) {
			[self.alertInterfaceDelegate presentInterfaceWithTitle:@"Erro"
			                                               message:error.message];
		}
	];
}
@end
~~~

Aqui temos um exemplo simples, mas é interessante tratar de alguma forma de que `self.alertInterfaceDelegate` está configurado - se não estiver, é erro do desenvolvedor dos aplicativos, que não importaram a biblioteca de UI como dependência. Podemos então fazer o tratamento de erro desejado para erro do programador - como gerar uma exceção com uma finalização anormal do app, algo que seja detectado instantaneamente pelo desenvolvedor, não algo que vai ser detectado pelo usuário do aplicativo.

# _Sample_, _Core_, _UI_...how to handle it?

Ok, antes tinhamos um único _workspace_ contendo 3 projetos: _Sample_, _Pods_ usados pelo SDK e o SDK propriamente, mas agora temos 3 projetos separados, sendo as bibliotecas _core_ e de UI distribuídas via CocoaPods. Agora imaginemos a seguinte situação: estamos alterando a biblioteca de UI, adicionando uma nova tela que será chamada pelo _core_, e para isto precisamos alterar este último para fazer a nova chamada da tela sob determinada condição. Devemos então fazer a alteração na biblioteca _core_, gerar uma nova versão, publicar esta, voltar no projeto de UI, efetuar o `pod update` e seguir com o desenvolvimento da UI, certo? Errado, isso é trabalhoso demais.

Pods...local pods FTW!

mantémos os dois repositórios localmente, e no `Podfile` da biblioteca de UI, adicionamos o _core_ como dependência, mas adicionamos o atributo `:path` apontado para o caminho local.

~~~ruby
target 'UI' do
	...
	pod 'SDKCore', :path => 'path/to/sdk-core/'
end
~~~

Agora não precisamos publicar toda vez que uma alteração é feita na biblioteca _core_, facilitando muito. Mas ainda tem uma melhoria, proposta pelo artigo [CocoaPods: Working With Internal Pods Without Hassle](http://albertodebortoli.com/blog/2014/03/11/cocoapods-working-with-internal-pods/), user um pouco de `git submodule`!

Um exemplo de como usar isso no projeto da biblioteca de UI seria:

1. No projeto UI (_core_ como dependência)
	- `$ mkdir Vendor`
	- `$ cd Vendor`
	- `$ git submodule add server/path/to/sdk-core.git`
2. crie um grupo no projeto `UI`, `ctrl`+click, _Add Files do UI.xcodeproj_
	- navegue até `Vendor/sdk-core` e adicione o projeto `sdk-core.xcodeproj`
	- navegue até `Vendor/sdk-core/Pods` e adicione o projeto `Pods.xcodeproj`

E podemos deixar o `podfile` do projeto de UI como o modelo a seguir:

~~~ruby
target 'UI' do
	...
	pod 'SDKCore', :path => './Vendor/sdk-core/'
end
~~~

Agora podemos executar `pod install` no projeto UI que ele irá compilar com a biblioteca _core_, mas no `.podspec` deixamos o _core_ sendo baixado do servidor. Para quem desenvolve as duas bibliotecas, temos uma única janela e um único local de alteração, com alterações entre projetos facilitadas.

Qual problema é resolvido com a combinação _local pods_ + `git submodule`? Um resumo seria:

- Situação: 2 projetos independentes
	- duas janelas do Xcode
	- UI exige alteração do _core_
		1. Altera o projeto _core_
		2. Gera nova versão
		3. Sobe para repositório local de _pods_ internos
		4. `pod update sdk-core` dentro do projeto UI
		5. burocrático e demorado

Mas com a dupla `git submodule` e CocoaPods temos:

- Fácil manutenção
	- uma única área de trabalho, dois projetos independentes
	- compilado da forma tradicional 
	- para quem precisa dar manutenção nos dois projetos
		1. Clonar o projeto UI
		2. `$ git submodule update --init` - UI baixa as dependências de submodulos
		3. Alterar dentro do workspace o projeto  
- Fácil distribuição
- Fácil versionamento
	- projetos podem ser versionados individualmente para quem vai usar, e não desenvolver

Ao terminar as alterações dois dois projetos, devemos:

1. efetuar o `commit` da biblioteca _core_;
2. efetuar o `commit` da biblioteca de UI

O `push` pode ser feito a qualquer momento. Mas é importante que o `commit` da biblioteca de UI seja feito após o `commit` da biblioteca _core_, pois será feita uma referência ao `commit` exato do _core_ do momento que o é feito o `commit` do projeto de UI. Isso é válido se você mudar de _branch_ no projeto _core_, por exemplo.

Agora no projeto _Sample_ basta fazer algo semelhante, porém adicionando o repositório da biblioteca de UI como dependência, e para baixar todos os submodules recursivamente, executar o comando:

`$ git submodule update --init --recursive`

Que irá baixar o código da biblioteca de UI e da biblioteca _core_.

A estrutura de diretórios de nossos projetos agora fica similar a esta:


~~~
~/Sample
    └── .git
    └── .gitsubmodules
    └── Podfile
    └── Pods
    └── Sample.xcodeproj
    └── Sample.xcworkspace
    └── Sources
    └── Vendor
        └── SDKUI (submodule)
            └── .git
            └── .gitsubmodules
            └── Podfile
            └── Pods
            └── SDKUI.podspec
            └── SDKUI.xcodeproj
            └── SDKUI.xcworkspace
            └── Sources
            └── Vendor
                └── SDKCore (submodule)
                    └── .git
                    └── Podfile
                    └── Pods
                    └── SDKCore.podspec
                    └── SDKCore.xcodeproj
                    └── SDKCore.xcodeproj
                    └── Sources
~~~

E a estrutura dos projetos dentro do Xcode ficam assim:

~~~
▼ Sample (projeto)
  ▶ Source
  ▶ Products
  ▶ Frameworks
  ▶ Pods
  ▼ Vendor
    ▼ SDKUI
      ▶ Source
      ▶ Products
      ▶ Frameworks
      ▶ Pods
      ▼ Vendor
        ▼ SDKCore
          ▶ Source
          ▶ Products
          ▶ Frameworks
          ▶ Pods
        ▶ Pods (Projeto)
    ▶ Pods (Projeto)
▶ Pods (Projeto)
~~~

# Resultados

Após algumas _sprints_, o resultado foi:

SDK (apenas o _core_):

- Redução das linhas de código: de 28000* para 13500 (sendo 7000 linhas de testes)
- Cobertura de testes: de 85% para 83%
	- enquanto esse processo era feito, outro desenvolvedor do time chegou a 95% de cobertura na principal _branch_ de desenvolvimento
	- Estes 95% parecem ser um "máximo teórico", pois as bibliotecas de segurança são escritas majoritariamente em `C`, impedindo/dificultando a injeção de dependência e _mocks_/_stubs_

UI:

- Projeto novo, iniciado com aproximadamente 9500 linhas de código (sendo 5500 linhas de testes)
	- o _storyboard_ deve ser convertido para arquivo `xib` ou código-fonte em breve
- Após migração dos testes da versão anterior da biblioteca _core_, já nasceu com 82% de cobertura de código
- Pode ser facilmente versionada e distribuída com CocoaPods
- Times dos apps só precisam adicionar ao Podfile - nenhuma configuração necessária
- Podemos criar bibliotecas de UI para macOS/watchOS/tvOS - aplicativo só tem que adicionar a biblioteca de UI correta

Sample:

- Projeto simplificado
	- Redução de linhas de código: de 28000* para 2100

# Proxímos passos

Com a primeira experiência na definição de um núcleo comum e na integração das dependências, podemos agora gerar a nova versão da biblioteca de interface de forma muito mais simples e sem duplicação de código e recursos.

Além disso, podemos agora partir para a modularização mais fina desse projeto, como a separação melhor da camada de segurança.

Por último, pretendemos expor a solução de forma que os outros times dos aplicativos possam também modularizar as funcionalidades, ganhando assim a simplificação do trabalho do time de integração, mais detalhes de qualidade de código por times e áreas do aplicativo, facilidade na evolução de áreas separadas dos mesmos e maior estabilidade geral dos aplicativos.

# Referências:

- [Overview of Dynamic Libraries](https://developer.apple.com/library/content/documentation/DeveloperTools/Conceptual/DynamicLibraries/100-Articles/OverviewOfDynamicLibraries.html)
- [Static and Dynamic Libraries](https://pewpewthespells.com/blog/static_and_dynamic_libraries.html)
- [Match-O Programming Topics](https://developer.apple.com/library/content/documentation/DeveloperTools/Conceptual/MachOTopics/0-Introduction/introduction.html)
- [Git submodule tutorial – Cocoapods might not be the solution](https://idevzilla.com/2015/09/21/git-submodule-tutorial-cocoapods-might-not-be-the-solution/)
- [CocoaPods: Working With Internal Pods Without Hassle](http://albertodebortoli.com/blog/2014/03/11/cocoapods-working-with-internal-pods/)
- [Objective-C Class Loading and Initialization](https://www.mikeash.com/pyblog/friday-qa-2009-05-22-objective-c-class-loading-and-initialization.html)
- [Using CocoaPods to Modularize a Big iOS App](http://product.hubspot.com/blog/architecting-a-large-ios-app-with-cocoapods)
- [Framework Programming Guide](https://developer.apple.com/library/content/documentation/MacOSX/Conceptual/BPFrameworks/Frameworks.html)