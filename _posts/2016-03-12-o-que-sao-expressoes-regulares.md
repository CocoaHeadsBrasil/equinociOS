---
layout:     post
title:      "Seja um desenvolvedor regular"
subtitle:   "Adicionando expressões regulares não seu dia-a-dia"
date:       2016-03-12 00:00:00
author:     "Diego Ventura"
header-img: "img/nomeDoUsuario/imagem.jpg" (imagem de cabeçalho)
category:   Categoria
---

> Diego Ventura ([@venturadiego](https://twitter.com/venturadiego){:target="_blank"}) é carioca daqueles que fala xêro, bixcoitu e caô. É desenvolvedor iOS desde 2012 e faz parte do time mobile da OLXBrasil. Acredita que Objective-C é amor e Swift é paixão. Pra ele, o equinociOS é mais uma forma de retribuir *para* a comunidade tudo o que ele aprendeu *pela* comunidade.

## O que são expressões regulares
Expressão regular, vulgo _regex_, é um padrão de caracter que tem como objetivo identificar em uma _string_ um padrão definido. Esse padrão pode ser palavras, caracteres específicos, números e expressões. Imagine o conjunto de palavras {sossego, prato, assadura, casa, passas} e você precisa separar todas as palavras que possuem `ss`. A primeira coisa que vem a mente é verificar palavra por palavra e checar se essa palavra contém `ss`. Com expressões regulares, você define um padrão e utiliza esse padrão para buscar as palavras que se encaixam nele. Se você for do tipo questionador, pode dizer: tá, mas isso eu faço com um `if` simples. Ok, concordo, mas e se quiser somente as palavras que contêm `ss` e que terminam com vogais (fica aí um exercício para quando terminar de ler o artigo)? É aí que entram as expressões regulares!

## Como funcionam as expressões regulares
As expressões regulares são formadas por _metacaracteres_ que, de acordo com a sua funcionalidade, dão mais poder às pesquisas. Elas formam padrões que, em alguns casos, são quase impossíveis de serem especificados utilizando caracteres comuns.

Os _metacaracteres_ são divididos em 4 (quatro) grupos: Especificadores, Quantificadores, Âncoras e Agrupamento. Para facilitar o entendimento, segue abaixo uma tabela com cada caracter e sua funcionalidade, separados por grupos.

#### Especificadores
Especificam (daí o nome) um conjunto de caracteres a partir de uma posição.

| Metacaracter | Funcionalidade |
|-----------------|---------------:|
| `.` | Qualquer caracter |
| `...` | Qualquer caracter incluído no conjunto |
| `^...` | Qualquer caracter *não* incluído no conjunto |
| `\d` | Qualquer caracter que seja um número (o mesmo que `[0-9]`) |
| `\D` | Qualquer caracter que *não* seja um número (o mesmo que `[^0-9]`)  |
| `s` | Qualquer caracter _branco_ (espaços, quebra de linha, _tabs_)  |
| `S` | Qualquer caracter *não* _branco_ |
| `w` | Qualquer caracter alfa numérico |
| `W` | Qualquer caracter *não* alfa numérico |
| `\` | Ignora o significado do próximo caracter na expressão. Ex.: Se quiser representar um `.` (ponto final), você utiliza `\.`. Dessa forma o `.` não vai representar qualquer caracter |

#### Quantificadores
Definem o número de repetições permitidas pela expressão anterior

| Metacaracter | Funcionalidade |
|-----------------|---------------:|
| `{n}` | Busca por exatamente `n` ocorrências. Ex.: A expressão `\d{4}` procura por um número de 4 dígitos |
| `{n,m}` | Busca por um mínimo de `n` e um máximo de `m` ocorrências |
| `{n,}` | Busca por no mínimo `n` ocorrências |
| `{,n}` | Busca por no máximo `n` ocorrências |
| `?` | Busca por 1 (uma) ou nenhuma ocorrência |
| `+` | Busca por 1 (uma) ou mais ocorrências |
| `*` | Busca por 0 (zero) ou mais ocorrências |

#### Âncoras
Definem as posições de referência para o _match_ da expressão regular.

| Metacaracter | Funcionalidade |
|-----------------|---------------:|
| `^` | Define a posição do início do texto ou de uma nova linha (em caso de textos multi-linhas) |
| `\A` | Define a posição do início do texto |
| `$` | Define a posição do fim do texto ou de uma linha (em caso de textos multi-linhas) |
| `Z` | Define a posição do fim do texto |
| `b` | Define a posição de borda, ou seja, logo antes do início de uma palavra ou logo após o fim da palavra |
| `B` | Define a posição de *não* borda |

#### Agrupamento
Definem grupos

| Metacaracter | Funcionalidade |
|-----------------|---------------:|
| `(...)` | Define um grupo para aplicação de um quantificador |
| `...|...` | Define uma alternativa para a aplicação da expressão. A expressão deve dar _match_ à direita ou à esquerda |
| `\n` | Recupa um _match_ no grupo _n_|

Viu como as expressões são bem simples e, na maioria dos casos, os caracteres até fazem algum sentido?

## Qual as vantagens das expressões regulares
A grande vantagem de usar expressões regulares é poupar tempo. Elas são uma maneira simples de encontrar padrões. Você poderia escrever um `if` gigantesco e varrear cada caracter de uma _string_ para validar o formato de um e-mail, por exemplo, ou pode usar um simples `[A-Z0-9a-z._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,6}`.

![Regex](https://imgs.xkcd.com/comics/regular_expressions.png)

## Onde usar expressões regulares
As aplicações regulares no dia a dia do desenvolvedor são muitas. Você pode aplicar expressões regulares desde o console de debug (sim, o Xcode te dá essa opção) até nas validações de formulários. Um caso bem comum de aplicação de expressões regulares é na validação de e-mails. Além de e-mails, inúmeros campos podem ser validados com expressões regulares: telefones, CEPs, senhas, nomes, endereços, número de cartões de crédito etc.

## Como usar expressões regulares no iOS
Usar expressão regular no iOS é muito simples. Existe uma classe (`NSRegularExpression`) que pede no seu construtor a expressão e as opções que serão aplicadas no momento do _match_. Assim, caso você queira uma função para validar um campo CEP, por exemplo, pode fazer:

~~~ swift
func validateZipCode(zipCode: String) -> Bool {
	let regex = try! NSRegularExpression(pattern: "^[0-9]{2}[0-9]{3}-[0-9]{3}$", options: [.CaseInsensitive])

	let regexResult = regex.firstMatchInString(zipCode, options:[], range: NSMakeRange(0, zipCode.characters.count)) != nil

	guard regexResult else {
		return false
	}

	return true
}
~~~

Uma dica para você praticar é o site [Regex Pal](http://www.regexpal.com). Nele, é possível definir uma _string_ e testar as suas expressões à vontade.

Se quiser, crie uma expressão para o exemplo dado na primeira seção do artigo e deixe nos comentários. Só não vale ler os comentários anteriores ;)