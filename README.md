#equinociOS

No dia 20 de março acontece o [Equinócio](https://pt.wikipedia.org/wiki/Equinócio)*! Para celebrarmos este evento, iremos escrever, a partir do primeiro dia do mês, 20 artigos sobre temas de conhecimento elemental que qualquer desenvolvedor iOS deve dominar.

## Workflow de colaboração
Para colaborar com algum artigo, o autor deve seguir o seguinte fluxo:

1. Abrir uma [issue](https://github.com/CocoaHeadsBrasil/equinociOS/issues) contendo:
	- Título do artigo
	- Descrição sucinta do artigo
	- Data de publicação
2. Escrever o artigo e fazer o Pull Request para esse repositório, no branch `gh-pages`
3. Certificar-se de que o artigo se encontra sob a licença [Creative Commons](https://br.creativecommons.org/)

## Como fazer Pull Request do artigo
- Faça um fork, baixe ou clone o repositório `https://github.com/CocoaHeadsBrasil/equinociOS.git`
- Escreva seu artigo dentro do diretório [`_posts`](https://github.com/CocoaHeadsBrasil/equinociOS/tree/gh-pages/_posts) (que contém todos os posts do blog e que por sua vez são escrito na linguagem de marcação markdown).
- Faça o Pull Request

## Estrutura do post
**Todos os posts** devem ter o seguinte nome: `2016-02-15-welcome-to-jekyll.markdown`, onde `YYYY-MM-DD-nome-do-artigo.markdown`.

**Todos os posts** devem conter o seguinte cabeçalho:

	---
	layout:     post
	title:      "Titulo do Artigo"
	subtitle:   "Subtitulo do Artigo"
	date:       YYYY-MM-DD 12:00:00
	author:     "Nome do Autor"
	header-img: "img/nomeDoUsuario/imagem.jpg" (imagem de cabeçalho)
	category:   Categoria
	---

### Como salvar imagens
Caso seu post tenha imagens, você deve adiciona-las no diretório [`img`](https://github.com/CocoaHeadsBrasil/equinociOS/tree/gh-pages/img). **Porém não insira a imagem na raíz do diretório!** Crie um novo diretório com o nome do seu usuário e salve suas imagens nele. ;)

Sempre que você for utilizar a imagem, insira o caminho dela: `img/nomeDoUsuario/imagem.jpg`

### Como utilizar as imagens nos posts
Utilize `{{ site.baseurl }}` para concatenar com o diretório de imagem, como no exemplo abaixo: 

`<img src="{{ site.baseurl }}/img/nomeDoUsuario/imagem.jpg">`

### Como editar markdown
Para escrever seu artigo, você pode utilizar editores markdown como o [MacDown](http://macdown.uranusjr.com/) ou [Atom](https://atom.io/packages/markdown-writer)!

## Revisão dos artigos
É importante os colaboradores revisarem os artigos para não serem publicados com erros ortográficos ou erros técnicos!

Você já pode ir fazendo Pull Request do seu artigo dentro do diretório [`_posts`](https://github.com/CocoaHeadsBrasil/equinociOS/tree/gh-pages/_posts), pois ele só será exibido quando a data que foi especificada no cabeçalho chegar!

**É muito importante seu artigo estar pronto alguns dias antes da data de publicação, caso contrário medidas serão tomadas para manter as publicações diárias.**

## Como rodar localmente
1. Pelo terminal, vá ao diretório raíz onde seu equinociOS está localizado
2. Caso não possua o Bundler instalado, execute `sudo gem install bundler`
2. Se for a primeira vez que você irá rodar esse projeto, execute `bundle install` para garantir que todas as dependências que o projeto utiliza existem. Caso negativo, o download será efetuado
2. Execute `jekyll serve` ou simplesmente `jekyll s`
3. Confira qual foi o *Server Address* gerado pelo jekyll ![](img/jekyll-path.png)
4. Abra o navegador e entre no endereço


## Data de Publicação
- **01/03/2016:** O mundo é mais que seu umbigo, por [Marcelo Fabri](https://github.com/marcelofabri)
- **02/03/2016:** iOS nativo - load e parse de json da web sem framework de terceiros, por [Daniel Bonates] (https://github.com/dbonates)
- **03/03/2016:** CollectionView: Uma nova abordagem de TableViews, por [Vinicius Carvalho](https://github.com/Viniciuscarvalho)
- **04/03/2016:** Scene Kit Overview, por [Lucas Farris](https://github.com/luksfarris)
- **05/03/2016:** Testes de Aceitação em iOS, por [Felipe Valio](https://github.com/felipe-valio-movile)
- **06/03/2016:** UIKeyCommand: Teclas de atalho dentro do seu app, por [Douglas Fischer](https://github.com/DougFischer)
- **07/03/2016:** Desmistificando Storyboards, por [Rafael Nobre] (https://github.com/nobre84)
- **08/03/2016:** Desenvolvendo para Apple TV - compartilhando código e dependências entre plataformas, por [Tales Pinheiro](https://github.com/talesp)
- **09/03/2016:** Em busca de um layout bonito e adaptativo: UICollectionView, Auto Layout e Size Classes, por [Rodrigo Borges] (https://github.com/rdgborges)
- **10/03/2016:** Minimizando o acoplamento de Views e ViewControllers, por [Diogo Tridapalli](https://github.com/diogot)
- **11/03/2016:** Unity3D e o Mundo Apple, por [Mauricio Cardozo] (https://github.com/loloop)
- **12/03/2016:** Seja um desenvolvedor regular: Adicionando expressões regulares não seu dia-a-dia, por [Diego Ventura] (https://github.com/diegoventura)
- **13/03/2016:** WebView: A porta de entrada para desenvolvedores web, por [Emiliano E. S. Barbosa] (https://github.com/emilianoeloi)
- **14/03/2016:** Programação Reativa com RxSwift, por [Bruno Koga](https://github.com/brunokoga)
- **15/03/2016:** Generics + Functional Programming + ReactiveProgramming, por [Bruno Bilescky](https://github.com/brunogb)
- **16/03/2016:** Protocol-Oriented Programming, por [Lourenço Marinho](https://github.com/lourenco-marinho)
- **17/03/2016:** Bibliotecas, por [Igor Ferreira](https://github.com/igorcferreira)
- **18/03/2016:** iBeacon, por [Gabriel Oliva](https://github.com/gabrieloliva)
- **19/03/2016:** Usando Swift no desenvolvimento do seu backend usando Zewo, por [Thiago Holanda](https://github.com/unnamedd)
- **20/03/2016:** A vida de um desenvolvedor indie, por [Ricardo Borelli](https://github.com/rabc)

## Artigos extras
- **21/03/2016:** Resolvendo UI complexas com Auto Layout usando a linguagem Visual Format, por [Daniel Bonates](https://github.com/dbonates)
- **22/03/2016:** Implementando o Facebook iOS SDK, por [Gabriel Ribeiro](https://github.com/gabrielribeiro)
- **23/03/2016:** Método Swizzling, por [Fernanda Geraissate](https://github.com/fggeraissate)
- **24/03/2016:** Fastlane, por [Luiz Alberto da Silva Oliveira] (https://github.com/betodr)
- **25/03/2016:** Optionals e o Gato de Schrödinger, por [Francesco Perrotti-Garcia](https://github.com/fpg1503)
- **26/03/2016:**
- **27/03/2016:** App Extensions - Today Notification, por [Renato Matos](https://github.com/renatosarro)
- **28/03/2016:** Swift in the sky with types, por [Matheus Brasil](https://github.com/mabrasil)
- **29/03/2016:** Reactivecocoa + MVVM, por [Alessandro Santos](https://github.com/delarge77)
- **30/03/2016:** Testes Unitários, por [Solli Honorio] (https://github.com/shonorio)

## Contato
Para desenvolvedores que acharam a iniciativa interessante e quiserem se juntar e conhecer a comunidade de desenvolvedores iOS brasileira, se cadastre no [Slack do iOSDevBr](http://iosdevbr.herokuapp.com/).


(*) Em março, ocorre o equinócio de outono no hemisfério sul. No hemisfério norte, na mesma data ocorre o equinócio de primavera.
