---
layout:     post
title:      "Sincronização de dados com CloudKit"
subtitle:	"Colocando seu app na nuvem"
date:       2017-03-24 12:00:00
author:     "Guilherme Rambo"
header-img: "img/insidegui/cloudkit/cloudkit-header.jpg"
category:   "cloudkit"
---


> Guilherme Rambo ([@_inside](https://twitter.com/_inside){:target="_blank"}) é desenvolvedor iOS na [Peixe Urbano](https://peixeurbano.com){:target="_blank"}, criador do [BrowserFreedom](https://getbrowserfreedom.com), [ChibiStudio](https://itunes.apple.com/app/chibistudio/id1135307199) e criador de diversos projetos open-source, incluindo o [app da WWDC para macOS](https://github.com/insidegui/WWDC).

O CloudKit foi lançado pela Apple em 2014 e desde então recebeu diversas melhorias, como a possibilidade de utilizá-lo fora das plataformas da Apple, a liberação do seu uso em apps fora da App Store no macOS e a disponibilidade do framework no watchOS, completando o círculo de plataformas da Apple suportadas.

Acredito que a tecnologia ainda não esteja sendo explorada em todo o seu potencial pelos desenvolvedores nas plataformas da Apple, seja por medo ou desconhecimento. Pretendo ajudar a mudar isso com este artigo.

# Posso usar CloudKit?

Infelizmente, a primeira pergunta que todos se fazem quando juntamos as palavras "Apple" e "nuvem" na mesma frase é: "dá pra usar sem medo?". De fato, a Apple não tem um histórico muito bom quando o assunto é serviços na nuvem, desde [o fiasco do Mobile.me](http://gizmodo.com/5033442/steve-jobss-entire-mobileme-is-fail-email) até [o problema do iCloud Core Data](http://www.imore.com/debug-12-icloud-core-data-sync).

A boa notícia é que em se tratando de CloudKit, a coisa é bem diferente. Eu arrisco afirmar que talvez seja a tecnologia da Apple na nuvem mais confiável de todas. Mas como eu sei que apenas minhas palavras não serão o suficiente para convencê-lo de que uma tecnologia da Apple na nuvem é confiável, vou deixar que você mesmo decida. Se você usa o aplicativo Notas da Apple, saiba que ele usa CloudKit. Outros exemplos de uso do CloudKit pela própria Apple são o compartilhamento de atividades no Apple Watch, o app Fotos e o iCloud Drive.

Se os apps citados acima funcionam bem pra você, então pode ficar tranquilo e usar o CloudKit sem medo. Se eles não funcionam bem pra você, eu não vou conseguir convencê-lo de que a tecnologia é confiável 😅

Um detalhe importante a ser colocado é que a confiabilidade do CloudKit também depende muito da implementação. Como a API não automatiza muita coisa, cabe a cada desenvolvedor usá-la da melhor maneira para criar uma experiência agradável e confiável ao usuário, o que nos leva ao próximo ponto:

# Devo usar CloudKit?

Mesmo que você esteja convencido de que pode usar o CloudKit sem medo, isso não significa que você deva usá-lo, afinal existem aplicações para as quais ele é mais indicado e outras para as quais existem ferramentas melhores, não estamos falando aqui de uma bala de prata.

## Onde usar CloudKit

Estas são as situações para as quais o CloudKit é altamente indicado:

### Sincronizar dados privados dos usuários entre vários dispositivos

Este talvez seja o uso mais óbvio, que é a sincronização dos dados privados de um usuário entre os vários dispositivos daquele usuário.

**Exemplo:** um aplicativo de notas ou todo list onde o usuário pode criar e ler notas em qualquer dispositivo que possua associado à sua conta do iCloud (Mac, iPhone, iPad, Apple Watch, etc) ou até mesmo numa interface web.

*Alternativas: Realm Mobile Platform, Firebase, iCloud KVS ou iCloud Documents (dependendo do caso)*

### Armazenar configurações remotas do aplicativo

Utilizando o banco de dados público do seu container é possível armazenar dados que são compartilhados por todas as instâncias do seu app, por todos os usuários, mesmo que não estejam autenticados com uma conta do iCloud. 

**Exemplo:** um aplicativo que em épocas festivas (Carnaval, Natal, etc) muda as cores do seu tema poderia armazenar os códigos das cores no banco de dados público do CloudKit. Desta maneira, as cores poderiam ser alteradas remotamente sem a necessidade de um servidor próprio ou atualização do app.

*Alternativas: Realm Mobile Platform, Firebase ou servidor próprio*

### Sincronizar dados entre vários apps do mesmo desenvolvedor

Se você tem uma família de apps e quer que os dados dos seus usuários sejam acessíveis em todos os seus apps, é possível utilizar um container compartilhado entre todos os apps desta família de modo que os usuários tenham acesso aos mesmos dados em todos os apps. Isto também se aplica a configurações (item acima), poderiam ser compartilhadas por todos os apps que utilizam o mesmo container.

*Alternativas: servidor próprio (devem existir outros serviços que suportem isto, mas não tenho conhecimento para indicar algum)*

### Utilizar a conta do iCloud do usuário como forma de autenticação

Você pode utilizar a conta do iCloud do usuário apenas como meio de autenticação para algum outro app ou serviço

### Enviar notificações

Sim, é possível enviar notificações usando o CloudKit, eliminando a necessidade de utilizar um serviço terceirizado ou um servidor próprio para este fim.

*Alternativas: Firebase ou servidor próprio*

## Onde NÃO usar CloudKit

Agora que já vimos alguns exemplos de onde o CloudKit é extremamente indicado, vamos a dois exemplos de onde ele é altamente contra-indicado:

### NÃO: Armazenamento e sincronização de documentos

Se o seu app trabalha primariamente com documentos, o CloudKit não é a ferramenta mais indicada para armazenamento e sincronização dos mesmos. Neste caso o ideal seria usar iCloud Drive, Dropbox ou outros serviços similares. É possível armazenar arquivos grandes como fotos e vídeos no CloudKit, mas ele pode não ser a melhor solução se a função principal do seu app é lidar com documentos.

**Exemplo:** editores de texto estilo Pages, editores de imagens estilo Pixelmator, Sketch, etc

### NÃO: Sincronização de preferências do usuário

Para armazenar simples preferências do usuário ou quantidades muito pequenas de informação, utilize [iCloud KVS](https://developer.apple.com/library/content/documentation/General/Conceptual/iCloudDesignGuide/Chapters/DesigningForKey-ValueDataIniCloud.html) (`NSUbiquitousKeyValueStore`).

**Exemplo:** seu app tem uma opção para mostrar ou esconder uma barra de ferramentas e você quer que esta configuração seja propagada para todos os dispositivos do usuário

## Por que não usar alguma alternativa?

O CloudKit é uma tecnologia da própria Apple que já vem instalada em todos os dispositivos, não requer uma autenticação além da conta do iCloud que os usuários já possuem, tem funcionalidades muito poderosas e tem uma grande chance de continuar existindo por um bom tempo. Estes são os principais motivos que me fazem preferir o CloudKit em vez de soluções de terceiros.

# Quanto custa?

<img src="{{ site.baseurl }}/img/insidegui/cloudkit/free-with-limits.gif">

Esta é uma dúvida comum quando estamos falando de serviços de sincronização e com o CloudKit não é diferente. Esta questão do preço é comumente confundida pelos desenvolvedores, então vou explicar aqui da forma mais simples possível.

[Como disse Craig Federighi na introdução do CloudKit](https://www.youtube.com/watch?v=w87fOAG8fjk&t=1h36m36s):

> O CloudKit é grátis... (cof cof) com limites (cof cof) 🙊

Mas o que isso significa?

Colocando de forma simples: **O CLOUDKIT É GRÁTIS, PONTO**

O que a Apple fez foi criar um sistema que previna abusos, dessa forma, se você fizer um uso 'normal' do serviço, ele será sempre grátis.

Conforme foi dito na session 231 da WWDC de 2014 (tradução livre):

> Nós não queremos previnir uso legítimo
>
> Nós só queremos evitar que alguém abuse do CloudKit
>
> Os números que nós informamos aqui aumentam com o número de usuários que você tem

E tem mais: se você utiliza apenas o **banco de dados privado** do CloudKit (que é o uso mais comum), o uso dele conta para a cota do usuário, ou seja, quem paga é o usuário e não você.

Os limites para uso do banco de dados público do CloudKit aumentam com o número de usuários do seu app, conforme pode ser visto na simulação abaixo que fiz no site da Apple:

<img src="{{ site.baseurl }}/img/insidegui/cloudkit/quota.png">

Lembrando que estes limites são para o banco de dados público, que é compartilhado por todos os usuários do seu app, o uso do banco de dados privado de cada usuário conta para a cota daquele usuário no iCloud, ou seja, jamais terá custo algum para você.

# Mão na massa

Passada esta introdução, hora de colocarmos a mão na massa. Vou explicar vários conceitos sobre o CloudKit e ao mesmo tempo apresentar pequenos exemplos de como são usados na prática 😉

<h2 style="color:#C86D6C">Ativando o CloudKit no seu projeto</h2>

O primeiro passo para usarmos o CloudKit é habilitar ele no painel Capabilities do Xcode.

<img src="{{ site.baseurl }}/img/insidegui/cloudkit/cloudkit-enable.gif">

Ao habilitarmos o CloudKit no projeto, o Xcode se comunica com os servidores da Apple para atualizar o nosso provisioning profile e também para criar o container padrão para o app.

Perceba que ao ativar o CloudKit, o Xcode ativou automaticamente Push Notifications, porque as subscriptions do CloudKit utilizam push notifications (falarei mais sobre isso depois).

## Container

Um container nada mais é do que uma caixinha onde você coloca os dados dos seus usuários. O container é o pai de todos os dados e geralmente será diferente para cada aplicativo seu, embora seja possível compartilhar um container entre vários apps. Por padrão, quando você habilita CloudKit no seu projeto, o Xcode cria um container com o bundle identifier do seu app. Outro detalhe a ser apontado é que um único app pode acessar vários containers.

Containers são representados por objetos do tipo `CKContainer`.

<h3 style="color:#C86D6C">Acessando o container padrão</h3>

Acessar o container padrão do app é bem fácil: basta utilizar o método `default` de `CKContainer`.

~~~swift
let container = CKContainer.default()
~~~

<h3 style="color:#C86D6C">Criando e acessando um container personalizado</h3>

Se você pretende compartilhar dados no CloudKit com outros apps desenvolvidos por você ou por versões para diferentes plataformas do mesmo app (ex: entre macOS e iOS), você deve criar um container próprio para isso.

O nome do container deve seguir o mesmo esquema dos bundle identifiers: DNS reverso. No meu caso, criei um container chamado `iCloud.br.com.guilhermerambo.KitchenContainer`.

<img src="{{ site.baseurl }}/img/insidegui/cloudkit/custom-container.gif">

Com o container criado, basta usar o inicializador de `CKContainer` que aceita um `identifier` como parâmetro e passar o nome que demos ao nosso container.

~~~swift
let containerIdentifier = "iCloud.br.com.guilhermerambo.KitchenContainer"
let secondContainer = CKContainer(identifier: containerIdentifier)
~~~

Se você for usar somente o container padrão do app, basta usar `CKContainer.default()`, nos demais exemplos deste artigo utilizarei o container default para que o código fique mais breve.

## Database

Database é o banco de dados onde você irá armazenar os registros que seu app usa. Bancos de dados são representados  por objetos do tipo `CKDatabase`.

Todo container do CloudKit contém três bancos de dados:

### Private database

Este é o banco de dados privado, onde os dados privados dos seus usuários ficarão armazenados. Somente um dispositivo do usuário autenticado na conta do iCloud consegue ter acesso aos registros armazenados neste banco de dados. 

Sendo assim, você como desenvolvedor do app não consegue ver os dados dos seus usuários. O dashboard do CloudKit apenas informa o número de registros para cada tipo e você consegue ver os dados da sua própria conta, mas não dos outros.

Para acessar o banco de dados privado do usuário, utilizamos a propriedade `privateCloudDatabase` de `CKContainer`.

### Public database

Este é o banco de dados público, onde você pode armazenar dados globais do seu app ou dados criados pelos usuários que devem ser acessíveis pelos outros usuários.

Apesar do banco de dados ser público, é possível restringir quem tem acesso aos registros armazenados nele utilizando Security Roles, mas não falarei sobre elas neste artigo.

Para acessar o banco de dados público, utilizamos a propriedade `publicCloudDatabase` de `CKContainer`.

### Shared database

Este é o banco de dados de compartilhamento. Com o lançamento do iOS 10 e do macOS Sierra a Apple liberou também a função de compartilhamento do CloudKit, pela qual usuários podem compartilhar registros específicos nos seus bancos de dados privados com contatos, permitindo com que ambos vejam e editem os registros simultaneamente. O banco de dados compartilhado é usado para armazenar esses registros, mas seu app não irá lidar com ele diretamente.

## Zone

Uma zona dentro do CloudKit é como se fosse uma pasta onde você salva seus registros. Todo banco de dados do CloudKit tem uma zona padrão, a `Default Zone`. Você pode utilizar a zona padrão ou criar zonas novas para organizar seus registros. Só é possível criar zonas no banco de dados privado, o banco de dados público só permite o uso da zona padrão.

Algumas funcionalidades do CloudKit só são possíveis em zonas customizadas. Se você quiser salvar vários objetos relacionados ao mesmo tempo e precisa que a operação falhe caso algum deles não possa ser salvo, precisa usar uma zona customizada. O novo recurso de compartilhamento também requer a utilização de uma zona customizada.

Zonas são representadas por objetos do tipo `CKZone`.

<h3 style="color:#C86D6C">Obtendo uma lista de zonas</h3>

Para listar a zonas disponíveis em um banco de dados, utilizamos o método `fetchAllRecordZones` de `CKDatabase`.

<script src="https://gist.github.com/insidegui/f27e48e6eda5bf53d993e67d3fbfaa92.js"></script>

## Record

Records são os registros que estão salvos no banco de dados do CloudKit. Registros são representados por objetos do tipo `CKRecord`, que são basicamente dicionários onde podemos adicionar as chaves que quisermos, que se tornarão campos nas "tabelas" do servidor.

É importante ressaltar que, apesar do banco de dados do CloudKit ser schemaless (você não precisa definir os campos previamente), isso só é verdade no ambiente de desenvolvimento, após colocar seu app em produção você terá que fazer alterações no seu schema no dashboard ou através do seu app em ambiente de desenvolvimento e depois exportar essas alterações para o ambiente de produção.

### Tipos de dados aceitos

Embora `CKRecord` seja basicamente um dicionário, não significa que possamos salvar qualquer tipo de dado no CloudKit. Estes são os tipos de dados que podemos colocar nas chaves de um `CKRecord`:

- `String`: a Apple recomenda `String` para pequenas quantidades de texto
- `NSNumber`: tipos numéricos do Swift são convertidos automaticamente
- `Data`: um exemplo de uso para um campo tipo `Data` seria armazenar objetos próprios, codificados usando `NSCoding`
- `Date`: datas e horas podem ser armazenadas diretamente no CloudKit
- `CLLocation`: muito útil para apps que trabalham com localização, até porque é possível fazer queries com base em localização (mais sobre isso depois)
- `CKAsset`: este objeto do CloudKit representa um arquivo que pode conter uma grande quantidade de dados (fotos e vídeos, por exemplo)
- `CKReference`: este objeto do CloudKit representa uma referência que aponta para outro `CKRecord` no banco de dados

Além de poder utilizar os tipos citados acima por si só, qualquer chave em um `CKRecord` pode conter também um array desses tipos, desde que o array contenha objetos de um único tipo.

<h3 style="color:#C86D6C">Criando um registro</h3>

Vamos supor que estamos criando um aplicativo onde usuários podem cadastrar filmes, provavelmente teríamos um registro chamado `Movie`. Neste caso, `Movie` é o nosso `recordType`.

Para criar um registro de um filme, inicializamos um `CKRecord`:

~~~swift
let record = CKRecord(recordType: "Movie")
~~~

Com o objeto criado, basta começar a setar as propriedades. Dentro do view controller do app onde o usuário entra com os dados do filme, poderíamos atualizar o record na action de um `UITextField`, por exemplo:

<script src="https://gist.github.com/insidegui/399eb8f10315b8c87ee1926e3ccd9807.js"></script>

Agora você deve estar se perguntando: "o que diabos é `CKRecordValue`? 🤔".

`CKRecordValue` é um protocolo adotado por objetos suportados pelo CloudKit. O problema é que, usando Swift, o compilador não compreende que `String` adota `CKRecordValue` e por isso temos que fazer esse casting feio.

<h3 style="color:#C86D6C">Melhorando nosso código: custom subscript</h3>

Para melhorar essa situação, sempre que trabalho com CloudKit eu crio enums com os campos dos meus registros e adiciono uma extensão em `CKRecord` com um custom subscript que aceita esse enum. Explicando assim parece complicado, mas no código é bem simples.

Primeiro, o enum dos campos do nosso registro:

<script src="https://gist.github.com/insidegui/a66fd48d4e27a737718e9ddcd3b10fb9.js"></script>

Agora podemos criar a extensão em `CKRecord`:

<script src="https://gist.github.com/insidegui/7c862fa6226d8ecac72a4f137f699c46.js"></script>

Agora, para modificar os valores dos nossos registros, nosso código fica bem mais limpo, podemos fazer simplesmente assim:

~~~swift
record[.title] = title
record[.releaseDate] = date
record[.rating] = rating
// e assim por diante...
~~~

Vale ressaltar que com este subscript customizado, continuamos só podendo colocar no nosso registro valores suportados pelo CloudKit. Se tentarmos colocar algum valor não suportado, o campo ficará nulo.

Outro detalhe importante: `CKRecord` **não** é um value type, então quando você passa objetos do tipo `CKRecord` você está passando uma referência e, caso tenha `didSet` em propriedades do tipo `CKRecord`, o `didSet` não será chamado quando algum campo do registro for alterado 😉

Na "vida real", você deve utilizar seus próprios models (provavelmente value types) e convertê-los de/para `CKRecord` quando estiver lidando com o CloudKit.

<p style="background-color:#f5f5f5;border-left:5px solid #3461A6;padding:10px">O código completo desta parte você encontra no <a href="https://github.com/insidegui/CloudKitchenSink/blob/master/CloudKitchenSink/SimpleRecordTableViewController.swift">arquivo SimpleRecordTableViewController.swift do projeto de exemplo</a>.</p>

## CloudKit Dashboard

Agora que já sabemos como criar registros no CloudKit, seria interessante termos uma forma de ver o que está acontecendo no servidor quando salvamos nossos registros.

A Apple criou uma ferramenta para isto, o CloudKit Dashboard. No dashboard nós temos acesso a todos os nossos containers, bancos de dados, tipos de registros e muito mais.

Primeiramente, usando o app de exemplo, vamos criar um registro:

<img src="{{ site.baseurl }}/img/insidegui/cloudkit/simple-record.gif">

Agora que temos um registro criado, vamos acessar o dashboard. O primeiro passo é selecionar no menu do canto superior esquerdo com qual container queremos trabalhar.

<img src="{{ site.baseurl }}/img/insidegui/cloudkit/dashboard-container-menu.png">

Com o container selecionado, vemos inicialmente uma lista dos tipos de registro que temos. Todo banco de dados do CloudKit já vem por padrão com um tipo `User`, que armazena informações sobre usuários.

<img src="{{ site.baseurl }}/img/insidegui/cloudkit/dashboard-record-types.png">

Para visualizarmos o registro salvo no banco de dados público, selecionamos a opção "Default Zone" em "Public Data". Se tivéssemos outras zonas, elas apareceriam nesta mesma lista.

<img src="{{ site.baseurl }}/img/insidegui/cloudkit/dashboard-records.png">

Agora o dashboard está nos avisando que ele precisa de um index no ID do registro para fazer uma listagem deles. Basta clicar em "Add Record ID Query Index" e o dashboard irá então mostrar o registro que criamos usando o app.

<img src="{{ site.baseurl }}/img/insidegui/cloudkit/dashboard-records-2.png">

Note que, além dos dados que nós inserimos, o registro contém alguns metadados que o CloudKit adiciona automaticamente:

- `Record Name`: este é o ID único do registro, usado para localizar registros no banco de dados. Nós podemos definir este ID ou deixar que o CloudKit defina um automaticamente, neste caso ele usa um `UUID`.
- `Created`: a data de criação do registro. Pode ser acessada através da propriedade `creationDate` de `CKRecord`.
- `Created By`: o ID do usuário que criou o registro. Pode ser acessado através da propriedade `creatorUserRecordID` de `CKRecord`.
- `Modified`: a data da última alteração do registro. Pode ser acessada através da propriedade `modificationDate` de `CKRecord`.
- `Modified By`: o ID do usuário que realizou a última alteração no registro. Pode ser acessado através da propriedade `lastModifiedUserRecordID` de `CKRecord`.

## User Records

Como vimos na seção sobre o dashboard, o CloudKit cria automaticamente registros para os usuários do nosso app. Esses registros são do tipo `User` e contém por padrão somente um identificador único do usuário. Esse identificador é único por container, quer dizer que um usuário terá o mesmo identificador único em todos os bancos de dados e zonas dentro do seu container, mas se você utilizar mais de um container, verá IDs diferentes para o mesmo usuário em cada um deles.

O CloudKit nos permite fazer diversas coisas com o registro do usuário:

- Descobrir se o usuário está logado no iCloud ou não
- Obter o registro do usuário no container
- Obter o nome completo do usuário
- Obter os identificadores dos contatos do usuário que possuem registros correspondentes no mesmo container
- Atualizar o registro com dados úteis para o nosso app
- Ser notificado de mudanças no status da conta do iCloud

<p style="background-color:#f5f5f5;border-left:5px solid #3461A6;padding:10px">Todos os exemplos desta parte você encontra na íntegra no <a href="https://github.com/insidegui/CloudKitchenSink/blob/master/CloudKitchenSink/UserViewController.swift">arquivo UserViewController.swift do projeto de exemplo</a>.</p>

<h3 style="color:#C86D6C">Descobrindo se o usuário está logado</h3>

Em muitos casos é importante sabermos se o usuário está logado no iCloud no dispositivo atual para decidirmos se ativamos determinada funcionalidade do app ou até mesmo para impedir que o usuário utilize o app até que esteja logado.

**Importante:** caso você decida não permitir com que o usuário utilize o app sem estar logado no iCloud, lembre-se que poderá ter que prestar esclarecimentos para o time de review da Apple, afinal eles não gostam de apps que exigem algum tipo de login sem uma boa justificativa. Seu app precisa ter um bom motivo para exigir o login, do contrário poderá ser rejeitado.

Para obter o status da conta do iCloud, usamos o método `accountStatus` de `CKContainer`.

<script src="https://gist.github.com/insidegui/5523796b377133a637855e837acfbb7d.js"></script>

<h3 style="color:#C86D6C">Obtendo o ID do registro do usuário</h3>

Para obter o registro do usuário do CloudKit, primeiro precisamos saber o ID dele. Para isso, utilizamos o método `fetchUserRecordID` de `CKContainer`.

<script src="https://gist.github.com/insidegui/f0140a3255a6a033d84cf4417998b6f9.js"></script>

<a name="user-record-fetch"></a>
Agora que nós temos um `CKRecordID` referente ao registro do usuário, podemos usar o método `fetch` de `CKDatabase` no banco de dados público para baixar o registro completo do usuário.

<script src="https://gist.github.com/insidegui/42787a138c7213dd18e23b0ad633150c.js"></script>

No exemplo eu estou usando `publicCloudDatabase`, mas poderia usar `privateCloudDatabase`. Qual banco de dados usar aqui vai depender da sua aplicação, como o app de exemplo usa somente o banco de dados público, optei por usar este.

É possível ter dois registros diferentes para o mesmo usuário: um no banco de dados público e outro no banco de dados privado, embora ambos tenham o mesmo identificador, os dados contidos em cada um deles podem ser diferentes. Você pode por exemplo utilizar o registro do usuário no banco de dados público para salvar informações como avatar e apelido e usar o registro no banco de dados privado para e-mail, endereço e outros dados sigilosos.

<h3 style="color:#C86D6C">Obtendo o nome completo do usuário</h3>

Podemos obter o nome completo do usuário autenticado, mas isso requer a permissão do mesmo.

Para solicitar essa permissão, utilizamos o método `requestApplicationPermission` de `CKContainer`, passando o parâmetro `.userDiscoverability`. Aparecerá um alert para o usuário solicitando permissão.

Após obtermos permissão, utilizamos o método `discoverUserIdentity` de `CKContainer` para obter a identidade do usuário, que contém o seu nome completo na forma de `PersonNameComponents` que podemos formatar através de um `PersonNameComponentsFormatter`.

<script src="https://gist.github.com/insidegui/4a282db0d8843b1503087f85fe8c3039.js"></script>

<h3 style="color:#C86D6C">Descobrindo contatos do usuário que utilizam o mesmo app</h3>

Para obter uma lista de registros dos amigos do usuário que utilizam o app, ou seja, que tem registro no mesmo container, utilizamos o método `discoverAllIdentities` de `CKContainer`.

<script src="https://gist.github.com/insidegui/63ad1684f37b2dc3701029d17d625313.js"></script>

<h3 style="color:#C86D6C">Inserindo dados adicionais no registro do usuário</h3>

No nosso app de exemplo, vamos supor que nós queremos listar os filmes cadastrados e junto ao filme exibir o nome e avatar do usuário que cadastrou aquele filme. Infelizmente a Apple não fornece uma opção para obter o avatar do usuário do iCloud, mas nós podemos oferecer esta funcionalidade no nosso próprio app, utilizando um campo no user record.

Para isto, vamos aprender também sobre uma nova classe: `CKAsset`.

`CKAsset` é um objeto usado para armazenar arquivos grandes no CloudKit. A Apple recomenda que qualquer campo que seja maior do que alguns kilobytes seja armazenado usando `CKAsset`.

Trabalhar com `CKAsset` é muito simples: basta inicializá-lo com a URL para um arquivo que queremos armazenar no CloudKit e apontar um campo de um `CKRecord` para o `CKAsset` criado.

Quando o registro for salvo, o CloudKit cuidará de enviar o arquivo junto dele. No caso do nosso app estamos enviando imagens para serem usadas como avatar, no meu código de exemplo não há nenhum tipo de tratamento quanto ao tipo de imagem ou tamanho do arquivo, mas se fosse um app real eu provavelmente iria validar e redimensionar a imagem para evitar desperdício de espaço no iCloud e consumo de banda desnecessário.

Eu adicionei um botão na interface que abre um `UIImagePickerController` para que o usuário possa selecionar uma foto da biblioteca para servir de avatar. O exemplo abaixo mostra a implementação do que acontece após o usuário selecionar uma imagem:

<script src="https://gist.github.com/insidegui/8c960b15f65aa123fe9e20fe2ee07226.js"></script>

No exemplo acima, `imageURL` é uma `URL` para um arquivo salvo localmente, se você tentar inicializar um `CKAsset` com algum outro tipo de `URL` haverá uma exception e seu app irá travar.

O método `save` é usado tanto para criar quanto para atualizar registros. Ao passar um registro com um ID que já existe no servidor, o CloudKit irá atualizar por padrão apenas os campos que foram alterados desde o último salvamento.

Como podem ver, é muito simples adicionar um campo personalizado ao registro do usuário e enviar arquivos para o CloudKit.

<img src="{{ site.baseurl }}/img/insidegui/cloudkit/avatar.gif">

<h3 style="color:#C86D6C">Observando mudanças no status da conta do iCloud</h3>

Algo que pode acontecer enquanto seu app está rodando é que o usuário pode abrir as preferências do iCloud e trocar de conta, não estar logado e então logar ou estar logado e fazer logoff.

Em qualquer um desses casos, se o seu app varia de acordo com a conta do iCloud ativa, você precisa atualizar o estado do app para refletir esta mudança.

Para ser notificado de mudanças na conta do iCloud, basta registrar um observer para a notificação `.CKAccountChanged`:

<script src="https://gist.github.com/insidegui/0868e826fb781a07958963802c069e44.js"></script>

No meu exemplo, ao receber esta notificação, estou chamando o método do view controller que faz o processo de descoberta de todos os dados do usuário. Assim, a interface ficará sempre em sincronia com o status do iCloud no dispositivo.

<img src="{{ site.baseurl }}/img/insidegui/cloudkit/account.gif">

## Queries

Agora que já vimos como salvar informações no CloudKit, vamos ver como fazemos para buscar essas informações. A forma mais simples de obter um registro do CloudKit é através de uma simples busca com base em um ID, <a href="#user-record-fetch">como já vimos antes</a>.

Se quisermos fazer buscas mais avançadas, filtradas com base em outros campos do registro, precisaremos utilizar a classe `CKQuery`. Com ela, podemos especificar um predicate que irá definir um filtro para a busca.

Se você não tem familiaridade com `NSPredicate`, recomendo que [dê uma lida na documentação](https://developer.apple.com/reference/foundation/nspredicate), pois trata-se de uma classe muito poderosa e que é usada em diversas APIs da Apple.

Para rodarmos uma query no CloudKit, iremos utilizar uma `CKQueryOperation`. Realizar queries é apenas uma dentre muitas funcionalidades do CloudKit que são expostas na forma de operations.

<h3 style="color:#C86D6C">Obtendo todos os registros de um determinado tipo</h3>

Este é o exemplo mais simples. Vamos executar uma query para obter todos os filmes que temos registrados no nosso banco de dados público. O primeiro passo é construir uma query com um tipo de registro e um predicate, como queremos todos os registros, basta criarmos uma query especificando o tipo `Movie` e um predicate com o valor `true`:

<script src="https://gist.github.com/insidegui/839813b1754e31b648aaf47c50ae636a.js"></script>

Agora que temos uma operação, antes de executá-la, precisamos definir os callbacks que serão chamados para que sejamos informados de novos resultados e possíveis erros na operação:

<script src="https://gist.github.com/insidegui/c3338649c87ae8535294dfa710532928.js"></script>

`CKQueryOperation` tem dois callbacks: um que é chamado para cada registro obtido do CloudKit e outro que é chamado no final da operação, após todos os registros terem sido baixados.

Há dois parâmetros neste último callback que merecem comentário: `cursor` é um objeto do tipo `CKCursor` que poderá estar presente ao final da operação caso haja mais resultados a serem obtidos. Se sua query for retornar uma quantidade muito grande de registros, você terá que executar várias `CKQueryOperation`s para conseguir obter todos eles, passando o `cursor` nas queries subsequentes.

O parâmetro `error` também é importantíssimo, a partir dele é possível saber se houve um erro na operação e qual a natureza do erro. Alguns erros no CloudKit são recuperáveis, isso significa que não basta simplesmente exibir um alerta ao usuário no primeiro erro encontrado, você precisa lidar com o erro. Dependendo de qual foi o erro, o CloudKit irá informar até mesmo em quantos segundos é recomendado que a operação seja tentada novamente (e você deve definitivamente seguir essa recomendação).

Finalmente, após configurarmos nossa operação, basta executá-la, adicionando-a ao banco de dados:

<script src="https://gist.github.com/insidegui/e2834c0cb17d19a405a538ef2829683b.js"></script>

Se quiséssemos executar a operação no banco de dados privado, bastaria substituir `publicCloudDatabase` por `privateCloudDatabase`. Existem algumas outras operações do CloudKit que são executadas no nível do container, nesses casos você chamaria o método `add` de `CKContainer`.

<h3 style="color:#C86D6C">Realizando uma busca textual</h3>

Outro tipo de query muito comum de se fazer é a busca textual. No nosso exemplo, usuários podem querer buscar filmes pelo título. Felizmente o CloudKit sabe lidar muito bem com isso e podemos construir um predicate simples que dá conta do recado:

<script src="https://gist.github.com/insidegui/5806c2078648de7425b1e59d6d2c49fb.js"></script>

O predicate `self contains %@` significa "busque este valor em todos os parâmetros do registro que contenham texto".

<h3 style="color:#C86D6C">Realizando uma busca por localização geográfica</h3>

Parece mesmo que a Apple pensou em tudo, afinal podemos fazer até mesmo queries baseadas em localização usando o CloudKit. No meu exemplo utilizei a localização atual do dispositivo para buscar filmes que tenham sido gravados em locações num raio de 500km.

Para fazermos uma query baseada em localização, o predicate fica desta forma:

<script src="https://gist.github.com/insidegui/23d254910cc315d70143db0f8dfd232d.js"></script>

Onde `location` dentro da string se refere à chave no nosso registro, `currentLocation` é um objeto `CLLocation` com a localização atual e `radius` é um `Float` contendo o tamanho do raio incluído na pesquisa (em quilômetros).

Aqui está uma demonstração dos três tipos de query explicados acima:

<img src="{{ site.baseurl }}/img/insidegui/cloudkit/queries.gif">

<p style="background-color:#f5f5f5;border-left:5px solid #3461A6;padding:10px">O código completo desta parte você encontra no <a href="https://github.com/insidegui/CloudKitchenSink/blob/master/CloudKitchenSink/QueryTableViewController.swift">arquivo QueryTableViewController.swift do projeto de exemplo</a>.</p>

Fazer queries no banco de dados nos dá total flexibilidade para obtermos apenas as informações mais relevantes ao contexto do nosso app, mas o CloudKit tem algo ainda mais legal que isso. Com ele nós podemos criar queries persistentes, executadas a cada alteração no banco de dados e que notificam nosso app via push. Essas queries persistentes são chamadas de subscriptions.

## Subscriptions

Lembram que eu falei anteriormente sobre enviar notificações usando o CloudKit? É exatamente isso que subscriptions nos permitem fazer.

Através de subscriptions, nós registramos o interesse de um determinado dispositivo ser notificado toda vez que alguma alteração ocorrer no banco de dados. Então, se um novo registro for inserido, o CloudKit irá enviar uma notificação para aquele dispositivo. Essas notificações podem ser apenas de `content-available` (silenciosas), ou notificações normais que mostram um alerta no dispositivo e/ou colocam números no badge do app.

Através de notificações silenciosas, podemos garantir um update imediato em todos os dispositivos do usuário sempre que o mesmo fizer alguma alteração em outro dispositivo, afinal este tipo de notificação dá ao nosso app a oportunidade de efetuar um background fetch.

<h3 style="color:#C86D6C">Criando subscription para envio de notificações</h3>

Para criar uma subscription que envia notificações, primeiro precisamos obter permissão do usuário para que ele receba as notificações do nosso app e dizer para o sistema que queremos utilizar notificações remotas.

<script src="https://gist.github.com/insidegui/d9510bb2b567b3f01552bfd303fa0a04.js"></script>

Se você for usar somente notificações silenciosas (`content-available`), só precisa fazer a última chamada (`registerForRemoteNotifications`). No app de exemplo estamos usando notificações de alerta com som, por isso precisamos solicitar permissão através do `UNUserNotificationCenter`.

Tendo permissão do usuário para enviar notificações, podemos criar nossa subscription com o CloudKit. A subscription é um objeto `CKSubscription`, que criamos desta forma:

<script src="https://gist.github.com/insidegui/3228e3d30cd257f3cc2b27dfc824016d.js"></script>

Vamos ver o que cada coisa significa:

#### `recordType` 

O tipo de registro para o qual queremos receber notificações.

#### `predicate`

Qual query será executada para determinar se uma notificação será disparada ou não (neste exemplo, qualquer registro). Como mencionei na parte anterior, subscriptions são como queries persistentes que ficam rodando no servidor a cada alteração no banco de dados, é através deste parâmetro que nós determinamos que query será essa.

Lembram da query com localização que fizemos no exemplo anterior? Poderíamos registrar uma subscription com aquela query, fazendo com que o usuário seja notificado sempre que um filme gravado perto da sua cidade for registrado no sistema.

#### `options`

Uma lista definindo em quais circunstâncias a notificação será disparada. Podemos pedir para que a notificação seja disparada quando um novo registro for criado, um registro existente for alterado ou excluído (ou as três opções ao mesmo tempo).

<h4 style="color:#C86D6C">Definindo como será a notificação</h4>

Com nossa subscription criada, precisamos definir como será a notificação que essa subscription irá gerar. Para isso, vamos criar um objeto `CKNotificationInfo`:

<script src="https://gist.github.com/insidegui/d607ab29d2c60ddb6b297d51bbfa1265.js"></script>

#### `alertLocalizationKey`

Este parâmetro é uma chave no nosso `Localizable.strings` que será o formato do alerta. É necessário usar este parâmetro quando você precisa incluir dados do registro no texto do alerta. No meu caso eu estou incluindo o título do filme que foi criado:

`"movie_registered_alert" = "%@ has been registered, check it out!";`

#### `alertLocalizationArgs`

As chaves do registro que serão usadas para popular os placeholders no texto do alerta. No exemplo estou usando a chave `title`, que é o título do filme.

#### `desiredKeys`

As chaves do registro que serão enviadas junto da notificação, que podemos usar no app para fazer uma query e localizar o registro.

<h4 style="color:#C86D6C">Salvando a subscription</h4>

Agora que criamos a subscription e definimos como serão as notificações, basta salvá-la como se fosse um registro qualquer:

<script src="https://gist.github.com/insidegui/84c92632f108899c84e1de8d56df9654.js"></script>

Lembrando que a subscription deverá ser salva no banco de dados para o qual você deseja ser notificado. Como o app de exemplo usa o banco de dados público, estou usando `publicCloudDatabase`.

Com a configuração acima, ao criar um registro em outro dispositivo, meu iPhone e meu Apple Watch receberam a seguinte notificação:

<img src="{{ site.baseurl }}/img/insidegui/cloudkit/notification_watch.png">

<img src="{{ site.baseurl }}/img/insidegui/cloudkit/notification_phone.png">

<p style="background-color:#f5f5f5;border-left:5px solid #3461A6;padding:10px">O código completo desta parte você encontra no <a href="https://github.com/insidegui/CloudKitchenSink/blob/master/CloudKitchenSink/SubscriptionViewController.swift">arquivo SubscriptionViewController.swift do projeto de exemplo</a>.</p>

# Arquitetura para sincronização

O que nós vimos até agora no artigo foram apenas os conceitos básicos de como utilizar as funcionalidades do CloudKit. Para criar um app que sincroniza de forma eficiente e correta os dados do usuário, há muito mais a ser feito.

Para tentar ajudar quem precisa dessa funcionalidade, criei um simples app para criação de notas (estilo Notes da Apple) que usa uma arquitetura offline-first para sincronização.

O fluxo do app é mais ou menos assim:

<img src="{{ site.baseurl }}/img/insidegui/cloudkit/sync_flow.gif">

As notas são salvas localmente num banco de dados [Realm](https://realm.io). Através de [Realm Notifications](https://realm.io/docs/swift/latest/#notifications), o motor de sincronização sabe sempre que uma nota é adicionada, alterada ou removida no banco de dados local. Essas alterações são reconhecidas pelo motor de sincronização, que transforma os models em `CKRecord`s e envia para o CloudKit.

O mesmo processo acontece inverso quando a alteração ocorre no CloudKit: o app recebe uma notificação remota, pega as informações sobre quais registros foram adicionados/alterados/removidos e replica essas alterações no banco de dados local. A alteração no banco de dados local causa um update na interface.

Um cuidado que precisa ser tomado neste caso é no sentido de evitar loops de sincronização. Se uma alteração no Realm provoca um salvamento no CloudKit e uma alteração no CloudKit provoca um download e alteração no Realm, temos um loop. Para evitar este problema, o motor de sincronização registra um notification token com o Realm para que as alterações provocadas por ele não façam com que as notificações sejam enviadas para ele mesmo.

## Tratamento de erros

É muito importante observar a ocorrência de erros nas operações do CloudKit e tentar lidar com eles da melhor forma possível. Muitos desenvolvedores estão acostumados a simplesmente colocar um `print` quando ocorre um erro ou mostrar um alerta para o usuário, mas nem sempre esta é a melhor alternativa.

A primeira coisa que você deve fazer é verificar se o erro é recuperável. Existem dois casos muito comuns que causam erros recuperáveis quando estamos trabalhando com CloudKit:

### Erro temporário / timeout / internet ruim / rate limit

Às vezes pode ocorrer um pequeno *glitch* na conexão ou nos servidores da Apple que causa um erro temporário. Também pode acontecer do seu app estar chamando o CloudKit com muita frequência, neste caso o servidor irá recusar alguns requests para aliviar o excesso de carga. Nesses casos, o `error` retornado no callback da operação será do tipo `CKError`, que contém uma propriedade `retryAfterSeconds`. Se esta propriedade não for `nil`, use o valor contido nela como um delay para tentar novamente a operação que falhou.

Nos meus projetos com CloudKit eu sempre tenho uma função mais ou menos assim:

<script src="https://gist.github.com/insidegui/8c726b4b705ddfe2fe01f24672c9a792.js"></script>

Esta função facilita o tratamento de erros recuperáveis do CloudKit. Ela recebe um erro retornado de uma operação do CloudKit, um bloco a ser executado caso o erro seja recuperável e retorna um erro caso a operação não possa ser tentada novamente.

### Conflitos

Outro caso que pode acontecer é um conflito entre duas alterações no banco de dados. O usuário pode ter modificado um registro em um dispositivo enquanto o mesmo estava offline e depois fez uma alteração diferente em outro dispositivo. Neste caso, o salvamento irá falhar e o CloudKit irá retornar um erro do tipo `server​Record​Changed`.

O `userInfo` desse erro irá conter o registro original antes da modificação, o registro atual no servidor e o registro atual no cliente. Com estas informações, cabe ao seu app decidir o que fazer para solucionar o conflito. Alguns apps exibem um painel para que o usuário decida o que fazer, outros fazem um merge do conteúdo automaticamente e outros apenas salvam o registro que foi modificado mais recentemente.

<p style="background-color:#f5f5f5;border-left:5px solid #3461A6;padding:10px">O código completo desta parte você encontra no <a href="https://github.com/insidegui/NoteTaker">projeto NoteTaker, no meu Github</a>.</p>

# Conclusão

Com isto, chegamos ao fim deste <strike>pequeno</strike> artigo sobre CloudKit. Espero que este artigo tenha te ajudado a entender melhor o que é o CloudKit e tenha te dado ideias de como poderá utilizá-lo em seus projetos.

# Sugestões de estudo

- [Documentação da Apple sobre o CloudKit](https://developer.apple.com/reference/cloudkit)
- [Designing for CloudKit](https://developer.apple.com/library/content/documentation/General/Conceptual/iCloudDesignGuide/DesigningforCloudKit/DesigningforCloudKit.html)
- [CloudKit Sharing](https://medium.com/@kwylez/cloudkit-sharing-series-creating-the-ckshare-40e420b94ee8#.7snggpqs1)
- [Todas as sessions da WWDC sobre CloudKit](https://developer.apple.com/search/?type=Videos&q=CloudKit)