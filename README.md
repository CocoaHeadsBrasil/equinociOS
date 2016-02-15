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
- Vá ao diretório [`_posts`](https://github.com/CocoaHeadsBrasil/equinociOS/tree/gh-pages/_posts) (que contém todos os posts do blog, que por sua vez são escrito na linguagem de marcação markdown) e adicione seu artigo, mas fique atento as regras:
	- O título do arquivo deve manter o seguinte padrão: `2016-03-01-dinosaurs.markdown`, onde temos `ano-mês-dia-nomeDoArtigo.markdown`.
- Faça o Pull Request

## Características do arquivo do post
O cabeçalho de cada arquivo do post deve conter as seguintes informações:	
	
	layout:     post
	title:      "Titulo do Artigo"
	subtitle:   "Subtitulo do Artigo"
	date:       2016-03-01 12:00:00
	author:     "Fulano de Tal"
	header-img: "img/nomeDaImagem.jpg"
	category:   Categoria

Fique atento para:

- A data deve ser no formato ano-mes-dia
- Não se esqueça de inserir a imagem no diretório [`img`](https://github.com/CocoaHeadsBrasil/equinociOS/tree/gh-pages/img) e certificar-se que o parâmetro `header-img` possui o mesmo nome da imagem.

Para escrever seu artigo, você pode utilizar editores markdown como o [MacDown](http://macdown.uranusjr.com/) ou [Atom](https://atom.io/packages/markdown-writer)!


## Data de Publicação

- **01/03/2016:** O mundo é mais que seu umbigo, por [Marcelo Fabri](https://github.com/marcelofabri)
- **02/03/2016:** iOS nativo - load e parse de json da web sem framework de terceiros, por [Daniel Bonates] (https://github.com/dbonates)
- **03/03/2016:** (A definir), por [Diogo Tridapalli](https://github.com/diogot)
- **04/03/2016:** CollectionView: Uma nova abordagem de TableViews, por [Vinicius Carvalho](https://github.com/Viniciuscarvalho)
- **05/03/2016:** Fastlane, por [Luiz Alberto da Silva Oliveira] (https://github.com/betodr)
- **06/03/2016:** Generics + Functional Programming + ReactiveProgramming
- **07/03/2016:** (A definir), por [Rafael Nobre] (https://github.com/nobre84)
- **08/03/2016:** Desenvolvendo para Apple TV - compartilhando código e dependências entre plataformas, por [Tales Pinheiro](https://github.com/talesp)
- **09/03/2016:** Em busca de um layout bonito e adaptativo: UICollectionView, Auto Layout e Size Classes, por [Rodrigo Borges] (https://github.com/rdgborges)
- **10/03/2016:** Scene Kit Overview, por [Lucas Farris](https://github.com/luksfarris)
- **11/03/2016:** Unity3D e o Mundo Apple, por [Mauricio Cardozo] (https://github.com/loloop)
- **12/03/2016:** Seja um desenvolvedor regular: Adicionando expressões regulares não seu dia-a-dia, por [Diego Ventura] (https://github.com/diegoventura)
- **13/03/2016:** WebView: A porta de entrada para desenvolvedores web, por [Emiliano E. S. Barbosa] (https://github.com/emilianoeloi)
- **14/03/2016:** Programação Reativa com RxSwift, por [Bruno Koga](https://github.com/brunokoga)
- **15/03/2016:** A vida de um desenvolvedor indie, por [Ricardo Borelli](https://github.com/rabc)
- **16/03/2016:** Protocol-Oriented Programming, por [Lourenço Marinho](https://github.com/lourenco-marinho)
- **17/03/2016:** Bibliotecas, por [Igor Ferreira](https://github.com/igorcferreira)
- **18/03/2016:** iBeacon, por [Gabriel Oliva](https://github.com/gabrieloliva)
- **19/03/2016:** Usando Swift no desenvolvimento do seu backend usando Zewo, por [Thiago Holanda](https://github.com/unnamedd)
- **20/03/2016:** Testes Unitários, por [Solli Honorio] (https://github.com/shonorio)

(*) Em março, ocorre o equinócio de outono no hemisfério sul. No hemisfério norte, na mesma data ocorre o equinócio de primavera.
