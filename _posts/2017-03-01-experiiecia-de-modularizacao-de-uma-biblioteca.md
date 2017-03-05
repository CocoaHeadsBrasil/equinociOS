---
layout:     post
title:      "Experiências na modularização de um projeto de um SDK"
date:       2017-03-01 00:00:00
author:     "Tales Pinheiro"
category:   "arquitetura"
---

> Tales Pinheiro ([@talesp](https://twitter.com/talesp){:target="_blank"}) é mestre em computação pelo IME-USP, trabalhou por 8 anos com backend bancário programando principalmente em C, quando em 2007 (antes do anúncio do primeiro iPhone!) resolveu aprender Objective-C. Ataulemte é um dos lideres do capitulo iOS na Concrete Solutions, onde continua em um projeto Objective-C, mas ajudando outros em Swift

# Experiências na modularização de um projeto de um SDK



# Cenário inicial

- SDK comum a vários apps gerenciados por um time
	- inclui bibliotecas estáticas desenvolvidas por outra empresa (segurança)
	- além do time que gerencia, mais times internos "desconhecidos" alterando o código
- Cada app usando o SDK desenvolvido por um ou mais times, de uma ou mais consultorias
	- Questões estratégicas do cliente
- Merges dificeis, manutenção complicada, versionamento insuficiente
- Apps com suporte ao iOS 7, exigindo _linkagem_ estática


## SDK "atual"

- Comunicação
- Segurança
- UI Comum aos projetos
	- Redesign ocorrendo nos apps, exigindo _proxy_ determinando qual interface usada
		- `LoginViewController` vs `LoginViewControllerNew`
	- Problema adicional: SDK unico atrelado a dependencia de UIKit, impedindo porte para watchOS e macOS
- Sample (costuma estar desatualizado)


## SDK "atual"

![]({{ site.baseurl }}/img/talesp/iphone-sample-ios.png)

# Proposta

- Separar em 3 componentes principais
	- _core_
		- menor e mais simples
	- segurança
		- será mantido por outro time/consultoria
	- UI
		- versionamento do design simplificado
- Distribuição e versionamento individual via CocoaPods
- Futuramente modularizar o app inteiro
	- Elimiação do _merge hell_
	- facil verificação de pontos criticos e features "problemáticas"
	- granulação dos indicadores de qualidade


![]({{ site.baseurl }}/img/talesp/proposed-arch-core.png)


![]({{ site.baseurl }}/img/talesp/proposed-arch-full.png)


# Primeiro passo: Separar a UI

1. Criado novo projeto, que deve conter apenas código de UI
2. Movido código de UI para novo projeto (mas não de testes de UI)
3. Removido TODOS os arquivos de UI do projeto _core_
4. Removido Sample (irá se tornar novo "app", já em desenvolvimento

__Problemas:__ 
- telas incluídas no _core_ são chamadas por métodos de rede, não pelos apps principais
- _core_ não pode depender de UI
- para times que usam novas libs, comunicação deve ser transparente e sem configuração


# Protocolos, ao resgate

- _core_ define protocolos de exibição de interface
- biblioteca de UI adota e implementa os métodos
- truqes de _runtime_ do Objective-C (validos em Swift se classe herda de `NSObject`)


# Pequeno desvio: vamos falar sobre inicialização de objetos, bibliotecas e _frameworks_


# Um pouco sobre inicialização de objetos

- magia do _runtime_ Objective-C
- `+load`
	- chamado quando biblioteca é carregada em memória
		- linkagem estática: na inicialização do app
		- linkagem dinâmica: na primeira chamada de um método da biblioteca
- `+initialize`
	- chamado quando primeiro método da classe é chamado
	- _lazy_
	- seria necessário time dos apps chamarem método de configuração para cada classe


### Observação sobre `initialize` em Swift - Xcode 8.3 beta 3


> Swift will now warn when an NSObject subclass attempts to override the class initialize method. Swift doesn't guarantee that references to class names trigger Objective-C class realization if they have no other side effects, leading to bugs when Swift code attempts to override initialize. (28954946)


## "Linkagem" estática vs dinâmica

- funciona como forma de gerenciamento de dependencias
- determinam quando o código objeto da biblioteca/_framework_ é carregado em memória
- determinam como o código objeto da biblioteca/_framework_ é distribuído e incluído no projeto final


## "Linkagem" estática vs dinâmica

- linkagem estática:
	- código objeto é incluído diretamento no binário alvo
	- Binários maiores e de inicialização mais lenta
	- padrão pré-iOS 8


## Linkagem dinamica

- '_linkagem_' dinâmica:
	- nenhum código da biblioteca é adicionado ao binário
	- carregado em memória em tempo de execução no instante que um de seus simbolos é requisitado
	- como código não é anexado ao binário, pode ser atualizado sem recompilação
	- Possuem processo de inicialização e finalização/limpeza próprios


## dinâmico
![]({{ site.baseurl }}/img/talesp/mono-dynamic.png)



### Bibliotecas (_libraries_) vs Arcabouços (_Frameworks_)
- bibliotecas:
	- Binário Match-O carregado na inicialização ou execução
	- evita repetição da cópia do código executável entre aplicação e extensões - código só existe em um local
	- responsável por avisar ao _linker_ ocódigo adicional necessário
	- usam a extensão `.dylib`
- _frameworks_:
	- analogo a bibliotecas
	- embutidos em um pacote
	- permite versionamento da biblioteca
	- pode incluir _assets_

^ match-o: https://developer.apple.com/library/content/documentation/DeveloperTools/Conceptual/MachOTopics/0-Introduction/introduction.html


## linkagem estática

- inclui o código objeto diretamente no binário alvo
- aumento do tamanho do binário alvo
- aumento do tempo de inicialização do app
- alteração na biblioteca exige recompilação do binário alvo
- unico método de incluir código de terceiros até iOS 8


### Bibliotecas estática vs _Frameworks_ estático
- biblioteca:
	- recipiente de conjunto de código objeto - arquivo `*.o` gerado pelo compilador
	- usam extensão `.a` - `ar`_chive_ junta
	- ideal para incluir um conjunto de arquivos objetos de uma unica biblioteca de código


### Bibliotecas estática vs _Frameworks_ estático
- biblioteca:
	- _linker_ só pode usar arquivos objetos de uma arquitetura:
		- dois formatos _container_:
			1. arquivos objeto de mesma arquitura em unico `ar`_chive_
			2. binário _fat_ Match-O, criado com comando `lipo`

^ binario gordo: formato simples q armazena multiplos arqs de diff arquiteturas


### Bibliotecas estática vs _Frameworks_ estático
- _framework_:
	- pacote contendo um arquivo de biblioteca estática
	- forma conveniente de publicar _assets_ como imagens, fontes, arquivos de áudio, etc
- Criado manualmente ou via script - Xcode não tem template


## estático

![]({{ site.baseurl }}/img/talesp/mono-static.png)


# Voltando a programação normal

- _core_ consiste de alguns _singletons_
- podem ser usado na biblioteca de UI para configuração automática
	- biblioteca de UI depende da biblioteca _core_ 
		- _core_ é inicializado antes


# Definindo protocolos na biblioteca _core_

- Exemplo de interface padrão: _warnings_ e _alerts_
- na biblioteca _core_:

~~~objectivec
@protocol TPAAlertDelegate

+ (void)presentInterfaceWithTitle:(NSString *) title message:(NSString *)message;

@end
~~~


# UI configurando o uso

~~~objectivec
@implementation TPAAlertViewController

+ (void)load {
	[TPANetworkManager sharedManager].alertInterfaceDelegate = self;
}
@end
~~~

^ atenção para metodo de classe


# Singleton usando _delegate_

~~~objectivec
@interface TPANetworkManager: NSObject
+ (instancetype)sharedManager;

@property (nonatomic, weak) Class<TPAAlertDelegate>alertInterfaceDelegate;
@end
~~~

^ protocolo de classe, não de intancia


# Singleton usando _delegate_

No _core_

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

^ atenção para o uso, se UI ñ importado, app não faz nada: erro do programador do time


# Sample, Core, UI...how to handle it

- Um projeto/workspace para cada
- pods...local pods FTW!
	- E um pouco de `git submodule`

1. No projeto UI (_core_ como dependencia)
	- `$ mkdir Vendor`
	- `$ cd Vendor`
	- `$ git submodule add path/to/sdk-core.git`
2. crie um grupo no projeto `UI`, `ctrl`+click, _Add Files do UI.xcodeproj_
	- navegue até `Vendor/sdk-core` e adicione o projeto `sdk-core.xcodeproj`
	- navegue até `Vendor/sdk-core/Pods` e adicione o projeto `Pods.xcodeproj`


### Qual problema é resolvido com a combinação _local pods_ + `git submodule`?

- Situação: 2 projetos independentes
	- duas janelas do Xcode
	- UI exige alteração do _core_
		1. Altera o projeto _core_
		2. Gera nova versão
		3. Sobe para repositório local de _pods_ internos
		4. `pod update sdk-core` dentro do projeto UI
		5. burocrático e demorado


### Por que `git submodule` e CocoaPods

- Fácil manutenção
	- uma unica área de trabalho, dois projetos independentes
	- compilado da forma tradicional 
	- para quem precisa dar manutenção nos dois projetos
		1. Clonar o projeto UI
		2. `$ git submodule update` - UI baixa as dependencias de submodulos
		3. Alterar dentro do workspace o projeto  
- Fácil distribuição
- Fácil versionamento
	- projetos podem ser versionados individualmente para quem vai usar, e não desenvolver


## Podfile do projeto UI

~~~ruby
target 'UI' do
	...
	pod 'SDKCore', :path => './Vendor/sdk-core/'
end
~~~

Pode executar `pod install` no projeto UI
- _core_ usado como dependencia via CocoaPods, facilita distribuição
- alterações entre projetos facilitado


# Resultados

SDK (apenas o core):
- Linhas de código: de ____ para 4600
- Cobertura de testes: de 85% para 82%, atualmente em 95%
	- aparentemente o máximo teórico
		- funções em `C` de segurança impedem Injeção de Dependencia
		- Separação da lib de segurança pode jogar baixa cobertura do SDK para segurança

UI:
- Pode ser facilmente versionada edistribuída com CocoaPods
- Times dos apps só precisam adicionar ao Podfile - nenhuma configuração necessária

# Proxímos passos

- Criar a nova versão da lib de UI com o redesign
- Transformar as maiores features em apps/módulos

# Bonus: Convertendo biblioteca estática em dinâmica

- impactos de performance quando muitas libs dinâmicas são carregadas
	- _daemon_ `amfi` le e valida conteúdo e assinatura do _app_ e das bibliotecas
- conversão manual ou automática


## Conversão manual

- biblioteca de terceiro a ser integrada em app compatível com iOS 8+

1. Identificar formato
	- `$ lipo -indo libMyComp.a`
	- `Architectures in the fat file: libMyComp.a are: x86_64 arm64`
2. Extraír cada arquitetura do binário gordo
	- `$ mkdir -p x86_64`
	- `$ lipo libMyComp.a -extract x86_64 -output ./x86_64/libfoo.a`
3. Verificar se parte extraída tem arquitetura correta
	- `$ lipo -info ./x86_64/libfoo.a` 
	- `input file libfoo.a is not a fat file`
	- `Non-fat file: libfoo.a is architecture: x86_64`
4. Verificar que é um arquivo `ar`
	- `$ file ./x86_64/libfoo.a`
	- `libfoo.a: current ar archive random library`
5. Extrair os arquivos objetos
	- `$ cd x86_64`
	- `$ ar -x libfoo.a`
6. Contruír nova biblioteca dinâmica
	- `$ libtool -dynamic *.o -o libfoo_dynamic-x86_64.dylib -framework CoreFoundation -lSystem`
7. Repita para cada arquitetura
8. Crie uma biblioteca dinâmica gorda
	- $ lipo -create ./x86_64/libfoo_dynamic-x86_64.dylib ./arm64/libfoo_dynamic-arm64.dylib -output ./libfoo_dynamic.dylib`

## Conversão automática

- Empacotar biblioteca estática em uma dinâmica
	- mais confiável e mais fácil

1. Crie um novo projeto
2. Escolha _Cocoa Touch Framework_ como _target_
3. Adicione a biblioteca estática, _headers_ e recursos
4. Nas configurações do projeto, em _Build Settings_ da sua biblioteca dinâmica:
	- adicione a _flag_ `--all_load` na opção `OTHER_LDFLAGS`
	- adicione os caminhos da biblioteca estática nos campos `FRAMEWORK_SEARCH_PATH` e `LIBRARY_SEARCH_PATH`
	- se biblioteca estática não for binário gordo, adicione a _flag_ `-no_arch_warnings` no campo _OTHER_LDFLAGS_
5. Nas opções de _Build phases_ da biblioteca dinâmica:
	- adicione a biblioteca estática na fase _Link Binary With Libraries_
	- adicione os _headers_ que deseja exportar como publicos na fase _Headers_
	- adicione os recursos que deseja incluir na fase _Copy bundle resources_
6. Efetue o _build_ do projeto
7. Caso ocorram erros de simbolos inexistentes, volte em _Build Phases_, expanda a fase _Link Binary With Libraries_ e adicione as bibliotecas que a biblioteca estática espera ser _linkada_


# Referências:

- [Overview of Dynamic Libraries](https://developer.apple.com/library/content/documentation/DeveloperTools/Conceptual/DynamicLibraries/100-Articles/OverviewOfDynamicLibraries.html)
- [Static and Dynamic Libraries](https://pewpewthespells.com/blog/static_and_dynamic_libraries.html)
- [Match-O Programming Topics](https://developer.apple.com/library/content/documentation/DeveloperTools/Conceptual/MachOTopics/0-Introduction/introduction.html)
- [Git submodule tutorial – Cocoapods might not be the solution](https://idevzilla.com/2015/09/21/git-submodule-tutorial-cocoapods-might-not-be-the-solution/)
- [CocoaPods: Working With Internal Pods Without Hassle](http://albertodebortoli.com/blog/2014/03/11/cocoapods-working-with-internal-pods/)
- [Objective-C Class Loading and Initialization](https://www.mikeash.com/pyblog/friday-qa-2009-05-22-objective-c-class-loading-and-initialization.html)
- [Using CocoaPods to Modularize a Big iOS App](http://product.hubspot.com/blog/architecting-a-large-ios-app-with-cocoapods)