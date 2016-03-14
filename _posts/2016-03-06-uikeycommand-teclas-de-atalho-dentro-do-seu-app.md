---
layout:     post
title:      "UIKeyCommand: Teclas de atalho dentro do seu app"
date:       2016-03-06 00:00:01
author:     "Douglas Fischer"
header-img: "img/dougfischer/header.jpg"
category:   "open-source"
---

> Douglas Fischer ([@DougDiskin](https://twitter.com/DougDiskin){:target="_blank"}) começou a desenvolver para iOS desde que ele ainda se chamava iPhoneOS. Ao passar dos anos trabalhou em diversos projetos para grandes empresas da Fortune 500 e hoje é Tech Lead Mobile e um dos sócios da Abacomm. Douglas se acha tão importante que escreveu todo esse parágrafo em terceira pessoa.


Quando a Apple apresentou o primeiro iPhone ao mundo, pelas mãos de Steve Jobs, não foi pouca a quantidade de pessoas que correram para falar que algo sem um teclado físico jamais poderia ser levado a sério. Steve Ballmer que o diga! 

<center><img src="{{ site.baseurl }}/img/dougfischer/ballmer.jpg"></img></center>

Algum tempo depois o aparelho já era uma febre e todos adoravam seus displays com touchscreen. Aí veio o iPad!

O iPad ficava naquele meio termo, entre a mobilidade de um celular e a robustez de um computador ou notebook. As pessoas pareciam não entender em qual cenário ele de fato seria útil.

Anos se passaram e já é consenso que as pessoas de fato se acostumaram com esses dispositivos com seus teclados virtuais, seja no aparelho que for. De fato, parece que em alguns casos até esquecemos que atualmente é possível usar teclados físicos, externos, junto ao iPhone ou iPad.

O uso de teclados físicos (via Bluetooth) é algo presente no dia a dia de milhares de usuários iOS ao redor do mundo. Mas será que você está dando atenção para esses usuários?


### Mas tem alguém usando?

Se você está por aqui lendo esse artigo, as chances de você ser envolvido de alguma forma como o mundo de desenvolvimento de apps é gigante. 

Nós somos desenvolvedores, pessoas que estão sempre com um computador por perto. Talvez para você, não faça sentido o uso de um teclado físico no iPad porque você está sempre a poucos segundos de um computador, com teclado, mouse e tudo mais necessário para uma boa usabilidade.

Porém as pessoas comuns, os seus usuários, nem sempre estão em situação similar.

<center><img src="{{ site.baseurl }}/img/dougfischer/ipadpro.png"></img></center>

A verdade é que as pessoas estão cada vez mais distantes de computadores. Muitos até deixaram de possuir um computador em casa. O primeiro acesso a internet de milhões de pessoas na grande maioria das vezes é através de um smartphone ou tablet.

Há algum tempo eu parei para pesquisar no Twitter se muita gente usava teclados externos com seus iPads e o número de resultados positivos me surpreendeu muito. É um público que você com certeza vai querer tratar com carinho.


### OK! O que eu preciso fazer? Achei que o iOS cuidava disso pra mim.

No que se refere a texto, o iOS garante que todas as metas estão sendo cumpridas (e eventualmente dobradas). A Apple cuida de tudo isso para você. Basta tratar os campos de input como você já faz para o teclado virtual.

Agora convenhamos, isso é muito pouco perto do que podemos fazer.

Desde o iOS 7, já é possível implementar atalhos de teclado dentro dos seus apps. Eles ficam a seu critério. São capazes de realizar qualquer coisa dentro do seu aplicativo. 


### Quando devo implementar?

As situações onde você pode usar atalhos de teclado ao seu favor são muitas. 

* Se seu app mantém registros de alguma informação você pode usar atalhos para inserir um novo registro no seu app. O mais famoso é o `CMD + N`.

* Navegar entre conteúdos do seu app, mostrando o próximo registro ao pressionar para a direita.

* Salvar com `CMD + S`. Descartar com `CMD + Delete`.

* Fechar um Modal com a tecla `ESC`.

* Navegar entre uma infinidade de campos de um formulário com as setas do teclado.

* Um editor de texto pode usar atalhos para texto rico. Por exempo usando `CMD + B` para atribuir negrito (Bold) a um texto.

* E não é só em utilitários que os atalhos tem uso. Imagine um jogo de estratégia ou RTS no iPad usando atalhos de teclado para dar velocidade ao jogador. Um atalho para criar aquele novo soldado, por exemplo. Uma troca de equipamento em um FPS usando `CMD + 1, 2, 3...`

Os cenários são infinitos.


### Me convenceu! Mas deve ser díficil implementar isso né, Douglas?

Sim! É tão díficil quanto configurar um array e adicionar dois meros métodos ao seu `UIViewController`.

Tudo funciona em torno da classe `UIKeyCommand`. Uma classe bastante simples. Que possui apenas 4 propriedades para você configurar.

* **input**: O caractere, em `NSString`, que você deseja reconhecer. Se você quiser uma tecla que não for texto, pode usar alguma das constantes: `UIKeyInputUpArrow`, `UIKeyInputDownArrow`, `UIKeyInputLeftArrow`, `UIKeyInputRightArrow` e `UIKeyInputEscape`.

* **modifierFlags**: A tecla modificadora para esse atalho. Não sabe o que é uma tecla modificadora? É a tecla que você pressiona de forma combinada ao atalho, tipo o `CMD` do `CMD + N`. 

	As opções disponíveis são: `UIKeyModifierShift`, `UIKeyModifierControl`, `UIKeyModifierAlternate`, `UIKeyModifierCommand`, `UIKeyModifierAlphaShift` e `UIKeyModifierNumericPad`. 

	*Essa última define que a tecla está no teclado número, e eu sinceramente não consigo imaginar um cenário onde você utilizaria isso dentro do iOS!*
	
* **action**: É o `@selector` que será chamado quando esse atalho for reconhecido. Aqui você vai apontar para o método da sua classe que de fato executa o que o atalho deve ativar.

* **discoverabilityTitle**: Um texto opcional descrevendo o nome do atalho. Essa propriedade foi adicionada apenas no iOS 9 e já veremos porque ela é tão importante para essa funcionalidade. Aguenta aí! 



### Ok! Já conheço a classe `UIKeyCommand`. Como faz agora? Estou ficando sem paciência! 

Vamos lá...

Vá até seu `UIViewController`, garanta que ele está habilitado para ser um `firstResponder` da sua aplicação.


~~~objc
- (BOOL)canBecomeFirstResponder {
    return YES;
}
~~~ 


Agora retorne um `NSArray` de `UIKeyCommand` no método `keyCommands`.


~~~objc
- (NSArray <UIKeyCommand *> *)keyCommands {
	return @[
		[UIKeyCommand keyCommandWithInput:@"N" modifierFlags:UIKeyModifierCommand action:@selector(addNewRecord:) discoverabilityTitle:@"Adicionar novo registro"]
	];
}
~~~ 


E claro, certifique-se que o método que será chamado realmente existe.


~~~objc
- (void)addNewRecord:(UIKeyCommand *)sender {
	// Seu código aleatório aqui...
}
~~~ 

Pronto! Agora seu app já reconhece o atalho `CMD + N` e chama o método de adicionar novo registro! Díficil né? Levou quanto tempo? 50 segundos?


### Muito legal... Mas como o usuário fica sabendo?

Ahá! Exatamente! Esse é o motivo porque os atalhos de teclado não eram muito comuns nos apps até recentemente. 

No iOS 7 e no iOS 8, o desenvolvedor precisava criar hints e outras formas de ensinar ao usuário que ele podia usar atalhos de teclado na sua aplicação. Cada app fazia de uma forma diferente e o resultado era um usuário pouco propenso a adotar os atalhos no seu dia a dia.

Com o iOS 9 e as diversas funcionalidades focadas em produtividade, a Apple criou o `Discoverability`, que entre outras funções, exibe uma tela em overlay, mostrando os atalhos disponíveis. Basta manter a tecla `Command` pressionada enquanto um teclado externo estiver conectado.

![]({{ site.baseurl }}/img/dougfischer/overlay-shortcuts.jpeg)
<span class="caption text-muted">Fica legal né?! ;-)</span>


### Faz sucesso!

Outro ponto legal de implementar essas funcionalidades, que não são primárias do app, mas que ajudam consideravelmente seus usuários mais ativos, é a relação que você constrói entre empresa e consumidor.

O Twitter está repleto de pessoas comemorando quando descobrem que algum app que eles gostam suporta atalhos desse tipo.

Essas reações colaboram muito com a divulgação do seu app nas redes sociais ou em matérias de blogs e afins. Até você mesmo pode divulgar isso como algum Easter Egg do seu produto! Tenho certeza que seu público vai gostar! 

É isso. Agora é com você! Vai lá e implementa isso logo no seu app!


### Referências

Links importantes:

* [Documentação Oficial da Apple](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIKeyCommand_class/)

* [Artigo do NSHipster sobre UIKeyCommand](http://nshipster.com/uikeycommand/)

Flw Vlw! 
