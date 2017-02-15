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
- **01/03/2017:** 
- **02/03/2017:** 
- **03/03/2017:** 
- **04/03/2017:** 
- **05/03/2017:** 
- **06/03/2017:** 
- **07/03/2017:** 
- **08/03/2017:** 
- **09/03/2017:** 
- **10/03/2017:** 
- **11/03/2017:** 
- **12/03/2017:** 
- **13/03/2017:** 
- **14/03/2017:** 
- **15/03/2017:** 
- **16/03/2017:** 
- **17/03/2017:** 
- **18/03/2017:** 
- **19/03/2017:** 
- **20/03/2017:** 

## Artigos extras


## Contato
Para desenvolvedores que acharam a iniciativa interessante e quiserem se juntar e conhecer a comunidade de desenvolvedores iOS brasileira, se cadastre no [Slack do iOSDevBr](http://iosdevbr.herokuapp.com/).


(*) Em março, ocorre o equinócio de outono no hemisfério sul. No hemisfério norte, na mesma data ocorre o equinócio de primavera.
