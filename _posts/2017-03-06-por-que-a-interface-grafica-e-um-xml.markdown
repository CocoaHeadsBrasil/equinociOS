---
layout:     post
title:      "Por que a interface gráfica é um XML?"
subtitle:   "Entenda um pouco da programação por definições"
date:       2017-03-06 00:00:00
author:     "Ronaldo Faria Lima"
header-img: "img/ronflima/background.jpg"
category:   ios
---

Antes da interface gráfica o mundo era feito de caracteres. A interface primária
com os computadores era através de uma tela de texto. À medida em que os
sistemas foram aumentando a quantidade de funcionalidades, as telas simples
tornavam-se um estorvo para o usuário. Assim, surgiram as aplicações orientadas
a menus. A barra de menus, que normalmente ficava no topo da tela, foi uma
grande novidade durante um tempo. Os menus começaram a ficar complicados, cheios
de níveis, de opções, de teclas de atalho. A interface caractere havia atingido
seu limite.

Até então, criar telas era simples. Era relativamente fácil criar funções para
desenhar janelas, controlar o mouse. Sem dúvida, exigia programação de baixo
nível. Por exemplo, para iniciar o driver do mouse era necessário realizar uma
interrupção de software que provocava a ativação do mouse. Para criar sombras
abaixo de menus, era necessário acessar a RAM de vídeo na página 0 e mudar os
atributos dos caracteres, como o valor de intensidade luminosa. Apesar de serem
práticas de baixo nível, não era difícil fazer isso. Uma vez escrita a função,
era fácil reutilizá-la.

Linguagens como dBase III, Foxbase e Clipper já vinham com uma série de funções
prontas para traçar a interface com o usuário. Basicamente, a interface era
criada via código. Nos sistemas unix havia, e ainda há, uma biblioteca chamada
_curses_ que permite ao programador controlar aspectos interessantes de um
terminal texto. Os terminais texto usam sequências de escape, ou seja,
sequências binárias que criam comportamentos interessantes, como ajustar a cor
de fundo de um texto ou o brilho sem a necessidade de gerar, manualmente, todas
as sequências binárias necessárias.

E aí chegou a interface gráfica. As telas deixaram de ser retangulares. Há
programas que criam janelas em formatos diversos. Porém, programar a interface
de um programa tornou-se uma tarefa hedionda. Escrever por código tudo o que era
necessário para criar efeitos como sombras, profundidade, textura, animações
ficou aburdamente complicado. Bibliotecas gráficas e SDKs como a Motiff, Win32,
MFC e OWL tentavam ajudar, mas mesmo assim a tarefa de escrever código era bem
complexa, apesar de ser uma repetição interminável.

Bem, agora chegamos onde queríamos: por que minha interface gráfica virou um
XML? Este blablablá todo foi para contextualizar a dificuldade que é criar uma
interface gráfica. Deveria haver alguma forma de simplificar este trabalho dos
infernos. E foi aí que alguém teve a brilhante ideia de começar a usar
representações para governar o comportamento do software que traçava os gráficos
da interface com o usuário.

## Borland x Microsoft

A Microsoft percebeu que seus programadores sofriam horrores com o seu SDK para
fazer aplicações para o seu ambiente operacional gráfico. Sim, estou falando do
Windows antes de tornar-se um sistema operacional de fato. Foi quando surgiram
os arquivos _res_. Os scripts de recursos permitiam ao programador criar ícones,
menus e janelas. A ideia era simplificar o trabalho do programador deixando-o
com uma linguagem mais simples que, ao ser interpretada, permitisse que a
interface fosse criada com mais facilidade.

Os arquivos _res_ eram efetivamente compilados e geravam todo o código
necessário para que seu conteúdo fosse exibido corretamente para o usuário
final. Isto foi uma grande evolução, mas a interface ainda continuava bem
complicada para ser criada. Começaram a surgir editores gráficos que geravam os
arquivos _res_, mas eram de uso complexo.

Com o avanço das linguagens visuais, em particular com o Visual Basic, os
arquivos de representação começaram a ser usados com mais frequência. A Borland,
indo nesta onda, lançou em 1995 o Borland Delphi que levou ao uso dos arquivos
de representação para outro nível. As telas eram criadas em arquivos-texto ou
binários que eram lidos pelo _run-time_, interpretados e apresentados ao
usuário. Criar interfaces gráficas havia se tornado um exercício de arrastar e
soltar.

## NeXT e o Interface Builder

Quase 10 anos antes do Delphi sair quebrando tudo no mercado de desenvolvimento,
em 1986, um francês chamado Jean-Marie Hullot, escreveu um editor de interfaces
gráficas usando a linguagem LISP. Este editor também usava arquivos de
representação para descrever como a interface gráfica deveria ser construída em
_run-time_. Steve Jobs, em 1987, conheceu Jean-Marie e incorporou esta
facilidade no _toolbox_ de desenvolvimento da NeXT, para simplificar o
desenvolvimento de interfaces gráficas para os programadores que desenvolviam
para o NeXTStep.

Assim, os programadores podiam usar uma ferramenta gráfica para criar seus
projetos e depois incorporá-los nos seus sistemas sem a necessidade de escrever
toneladas de código, a grande maioria código repetitivo e hoje conhecido como
_boiler plate_.

## Por que XML?

Até então cada solução usava uma forma proprietária para descrever os dados
necessários para fornecer às bibliotecas para que estas pudessem realizar seu
trabalho. Porém, em 1996, apareceu o XML como uma solução simples e portável
para criação de documentos estruturados, qualquer que fosse a estrutura desejada
para organizar a informação.

Com o aparecimento do XML e, consequentemente dos _parsers_, diversos frameworks
começaram a usá-lo para configuração ou processamento de instruções. O XML é
suficientemente simples e igualmente completo no que diz respeito às
representações possíveis com a linguagem. Assim, pode-se salvar virtualmente
qualquer representação em uma estrutura XML.

## XIB? NIB?

Bem, depois desse tanto de história, sabemos por que a Apple usa o formato XML
para os arquivos XIB. Estes arquivos contém representações que determinam como
uma view ou janela serão criadas, bem como o que será chamado quando houver
algum tipo de interação com o usuário. Porém, o que diferencia o NIB do XIB?

O arquivo NIB tem um formato binário, formato este que era usado antes do
surgimento do XML no mercado. A princípio, por uma questão de compatibilidade, o
XIB é convertido em NIB e é o NIB quem alimenta os frameworks gráficos da
Apple. No entanto, há outro motivo para haver esta transformação: O XML, apesar
de muito flexível, tende ter um processo de interpretação complexo, que pode
consumir CPU e memória. Apesar do NIB ser um formato antigo, trata-se de um
formato muito mais eficiente em termos de interpretação e execução em relação ao
XML.

Atualmente existem os _push parsers_ XML que são mais eficientes e econômicos
que os _parsers_ baseados em DOM. No entanto, nada é mais eficiente que uma
representação binária que pode ser lida em blocos, como é o caso dos arquivos
NIB. 

# O poder da representação

A ideia de usar representações para programar não é novidade. Em seu excelente
livro _"The Practice of Programming"_, Brian W. Kernighan, um dos pais da
linguagem C, apresenta algumas ideias sobre o uso de representações em
linguagens de programação, em particular em algumas funções da biblioteca STDC e
da linguagem de scripting _awk_.

A ideia é simples: dada uma representação arbitrária, um código que a interprete
terá um comportamento bem definido se esta representação fizer parte da
gramática reconhecida por este código. Assim é como funcionam os arquivos
XIB/NIB: eles definem sua interface gráfica em termos de estrutura visual e
comportamento, permitindo que métodos arbitrários pertencentes à classes
arbitrárias possam ser chamados quando houver algum tipo de interação com o
usuário.

O que se consegue com isto é um código amplamente flexível, que pode ser
facilmente customizado e usado em diversos contextos diferentes sem, contudo,
exigir reprogramação do mesmo. Como exemplo, veja a função da bibilioteca STDC
_printf_. É possível fazer virtualmente qualquer saída formatada com esta
função. O comportamento da formatação está descrito na string de máscara que tem
uma gramática simples e muito limitada mas não menos poderosa.

Este mesmo efeito é observado no Cocoa e no Cocoa Touch no que diz respeito ao
processamento de arquivos XIB/NIB. É possível construir uma combinação enorme de
telas sem a necessidade de customizar as classes que representam as views. Isto
faz com que o código torne-se bastante flexível a ponto de existirem casos de
desenvolvedores que nunca precisaram customizar a classe de uma view para
desenhar alguma coisa que o framework não suportava desenhar.

Sem dúvida, cedo ou tarde todo desenvolvedor precisará especializar alguma
classe visual do UIKit. No entanto, é possível criar comportamentos altamente
customizados apenas usando as classes fornecidas, o que demonstra o poder que
uma representação pode ter.

# Concluindo...

A sua interface gráfica é construída com base em um XML por que a linguagem de
marcação é simples, poderosa e permite representar virtualmente qualquer
combinação possível de elementos. Dentro do universo Apple, o Xcode ainda
permite representar nos seus XMLs classes customizadas que podem ter suas
propriedades editadas no Interface Builder.

Ao representar em um documento como a sua interface deve ficar, economiza-se
muito código, deixando para o framework todo o _heavy lifting_ necessário para
desenhar e dar comportamento à interface gráfica.
