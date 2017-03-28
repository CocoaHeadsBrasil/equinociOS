---
layout:     post
title:      "Adotando Handoff em iOS e macOS"
subtitle:   "The Good, the Bad and the Ugly"
date:       2017-03-28 00:00:00
author:     "Rafael Nobre"
header-img: "img/nobre84/baton-bg.jpg"
category:   continuidade
---

> Rafael Nobre ([@nobre84](https://twitter.com/nobre84){:target="_blank"}) é desenvolvedor iOS desde 2009 e trabalha na [ProDoctor Software](https://prodoctor.net){:target="_blank"}. Odiava o Swift no lançamento, mas foi rapidamente conquistado pela abordagem _Swifty_ de resolver problemas.

# Continuidade 
Handoff faz parte de um conjunto de funcionalidades chamadas coletivamente pela Apple de **Continuidade**. Seu objetivo é fazer com que diversos dispositivos compatíveis possam se comunicar para que o usuário obtenha mais produtividade ou comodidade. São estes recursos que permitem, por exemplo, atender chamadas recebidas no iPhone pelo Mac, enviar e receber mensagens de texto pelo Messages, desbloquear o Mac pelo Apple Watch ou transferir arquivos via AirDrop, entre outros.

# Handoff
A função do Handoff é simples: permitir a continuação de uma _atividade_ iniciada em um device por outro, desde que esteja _próximo_ e vinculado à mesma conta iCloud. Essas restrições conferem *segurança* e *contexto*, impedindo que alguém com acesso físico a um de seus aparelhos possa visualizar ou continuar uma de suas atividades.

## Como começar
Para adotar o Handoff, o primeiro passo é mapear quais _atividades_ de seu app serão expostas para continuidade, ou seja, identifique funções de alto nível em seu aplicativo cujo estado seria relevante compartilhar com um segundo aparelho de forma transparente. Como exemplos, o Safari permite que uma página visualizada inicialmente no celular seja lida no Mac e vice-versa, assim como o Mail permite que uma mensagem iniciada em um aparelho possa ser alterada e enviada em outro.

## Como funciona
O **Bluetooth LE** é um dos elementos chave do Handoff: é ele quem confere ao recurso o fator _proximidade_. Quando uma _atividade_ é declarada pelo desenvolvedor como elegível ao Handoff, é feito um _broadcast_ pelo sistema para que dispositivos próximos - e conectados à mesma conta do iCloud - possam continuar a atividade atual.

O interessante é que até este momento, nenhuma informação foi de fato trafegada, apenas o pequeno pacote de _advertising_ do Bluetooth. Somente quando uma aplicação elegível - isto é, que tenha sido assinada pelo mesmo *Team ID* - for acionada pelo usuário para continuar a _atividade_ é que inicia a transferência de dados, via *WiFi*, para o dispositivo de destino.

# Colocando a mão na massa

### Primeiro passo: declarar atividades
Basta uma entrada no `Info.plist` do projeto indicando a lista de tipos que o app sabe lidar (recomenda-se nomenclatura estilo DNS reverso para evitar colisões).

~~~xml
<key>NSUserActivityTypes</key>
<array>
	<string>com.equinocios.Handoff.sample</string>
</array>
~~~

### Segundo passo: publicar uma atividade
Agora é preciso encapsular o estado da aplicação em um objeto `NSUserActivity`, que será transferido para um segundo app para dar continuidade à tarefa. Este app pode ser macOS, iOS, ou ainda um browser (para ativar este recurso utiliza-se o atributo `webpageURL`).

O exemplo abaixo é o _bare bones_ da implementação em um `UIViewController`, no iOS.

~~~swift
let userActivity = NSUserActivity(activityType: "com.equinocios.Handoff.sample")
userActivity.isEligibleForHandoff = true
userActivity.title = "Novo artigo"
userActivity.userInfo = ["user": "nobre84", "title": "Adotando Handoff em iOS e macOS", "body": "# Continuidade\nHandoff faz parte de ...", "cursorPosition": 110 ]
userActivity.webpageURL = URL(string: "http://equinocios.com/continuidade/2017/03/28/adotando-handoff")
self.userActivity = userActivity
~~~

O `userInfo` é o principal mecanismo utilizado para trafegar o estado da aplicação. Nele você deverá incluir tudo que for necessário para recriar o estado exato da _atividade_ do usuário em um dado momento. Um bom candidato para se encarregar disto é o seu `view model` ou outra forma de separação de responsabilidades, uma vez que é interessante que o _boilerplate_ que lida com a criação da `NSUserActivity` não conheça o estado de suas _views_. Além disso, este não é o único local onde estas informações se farão necessárias, como veremos mais adiante.

### Terceiro passo: continuar uma atividade
Para concluir o processo é preciso implementar o método `application(_:​continue:​restoration​Handler:​)` no _App Delegate_ da aplicação macOS ou iOS. É o mesmo método utilizado para receber informações a respeito de buscas utilizando a API de Spotlight ou Intents do Siri.  
Baseado no `activityType` da `userActivity`, você deve definir a tela a qual seu app será redirecionado, e seu conteúdo através do `userInfo`. Isto pode ser feito repassando a `userActivity` para os _childs_ desde o `rootViewController` usando o método `restoreUserActivityState`, sobrescrevendo-o em cada ponto da hierarquia e passando-a adiante, até alcançar o _controller_ relevante para atualizar seu estado; ou recriar a hierarquia de views do zero para atender o _atividade_ - o que na minha opinião faz mais sentido.
Para gerenciar este fluxo pode ser utilizado algum mecanismo de roteamento, que desacople do app delegate de forma flexível (e testável!) essa complexa lógica de **qual** _view controller_ exibir dado uma `NSUserActivity`. Dica: essa lógica pode ser similar à utilizada para definir o `rootViewController` da sua `Window` no `application(_:didFinishLaunchingWithOptions:)`.

~~~swift
func application(_ application: UIApplication, continue userActivity: NSUserActivity, restorationHandler: @escaping ([Any]?) -> Void) -> Bool {

	// Repassando a userActivity, cada um dos controllers na hierarquia deve sobrescrever o método abaixo e empurrando a userActivity para o próximo child
	self.window?.rootViewController?.restoreUserActivityState(userActivity)

	// Ou recriando toda a hierarquia para atender ao Handoff
	if let router = Router(activity: userActivity) {
		// Obtém uma pilha de controllers que represente o estado correto do app
		let hierarchy = router.buildHierarchy()
		// Substitui o rootViewController da Window e empilha os demais
		router.appendHierarchy(window: window)
	}

	return true

}

func application(_ application: UIApplication, willContinueUserActivityWithType userActivityType: String) -> Bool {
	// Sabe-se que irá ocorrer um Handoff, mas o userInfo ainda não foi transmitido.
	// Pode ser utilizado para exibir feedback ao usuário e/ou exibir um overlay para uma transição suave
	return true
}

func application(_ application: UIApplication, didFailToContinueUserActivityWithType userActivityType: String, error: Error) {
	// Pode-se exibir feedback de que um problema ocorreu e/ou desfazer um overlay apresentado no willContinueUserActivityWithType
}
~~~

# The Good
Pronto! Agora nossos apps são capazes de publicar uma atividade e continuá-la em outro device! Isto é realmente incrível, um rápido rascunho pode ser finalizado em um poderoso Mac à noite em casa, assim como um importante e-mail iniciado em casa pode ser enviado no meio do trânsito. A Apple vai te olhar com bons olhos, quem sabe até dê um feature pro seu app! A API é simples, e você já tem praticamente metade do esforço necessário para também adicionar suporte a Spotlight Searching pro seu app ao usar `NSUserActivity`'s. Mas, nem tudo são flores!

# The Bad
A API, apesar de simples, possui algumas _idiossincrasias_. Lembra do exemplo _bare bones_ que dei lá em cima? Ele não funciona. Faltaram alguns detalhes importantes, os quais a documentação não é muito clara a respeito. 
O ciclo de vida de uma `NSUserActivity` é atrelado a um `delegate`, que _UIKit/AppKit_ já cuidam para nós através de extensões de `UIResponder` - o próprio _controller_, bastando setar a propriedade `userActivity` do mesmo.
Ao setar a `userActivity` de um _controller_ no iOS, a mesma está ativa e fazendo _broadcast_ da _atividade_, mesmo que ela não seja mais relevante no seu app (o usuário navegou para outro _controller_ por exemplo). É necessário então cuidar do ciclo de vida - normalmente acompanhando o do próprio _controller_. Já o _macOS_ faz a coisa certa neste caso.  
Ambas plataformas, porém, proveem uma forma de manter o estado do `userInfo` de uma `NSUserActivity` o mais atualizado possível, através do método `updateUserActivityState(_:)`, onde você terá a chance de atualizar o `userInfo` instantes antes de sua _atividade_ ser transferida para outro dispositivo ou caso você sete manualmente a flag `needsSave` para `true`. Plausível, não é mesmo? O inesperado é que, parte da implementação deste método simplesmente **esvazia** todo o conteúdo do `userInfo`! Ou seja, se você não fizer _override_ deste método, nada funciona!  
Existe também uma propriedade chamada `requiredUserInfoKeys` que, digamos, indica que tudo que você adicionar ao `userInfo` e não adicionar aqui, será jogado fora! Astuto, não!? Tem uma pequena nota documentando este comportamento, mas até hoje não consigo ver o sentido. 
Corrigindo então nossa implementação ingênua para algo mais próximo da realidade:

~~~swift
override func viewDidAppear(_ animated: Bool) {
    super.viewDidAppear(animated)
    startUserActivity()
}

override func viewDidDisappear(_ animated: Bool) {
    super.viewDidDisappear(animated)
    stopUserActivity()
}

override func updateUserActivityState(_ userActivity: NSUserActivity) {
    userActivity.addUserInfoEntries(from: uiStateDictionary())
    super.updateUserActivityState(userActivity)
}

private func startUserActivity() {
    let userActivity = NSUserActivity(activityType: "com.equinocios.Handoff.sample")
    userActivity.isEligibleForHandoff = true
    userActivity.title = "Novo artigo"
    userActivity.userInfo = uiStateDictionary()
    if let keys = userActivity.userInfo?.keys {
        userActivity.requiredUserInfoKeys = Set<String>(keys.flatMap { $0 as? String })
    }
    userActivity.webpageURL = URL(string: "http://equinocios.com/continuidade/2017/03/28/adotando-handoff")
    self.userActivity = userActivity
}

private func stopUserActivity() {
    self.userActivity?.invalidate()
    self.userActivity = nil
}

private func uiStateDictionary() -> [ AnyHashable: Any ] {
    return ["user": "nobre84", "title": "Adotando Handoff em iOS e macOS", "body": "# Continuidade\nHandoff faz parte de ...", "cursorPosition": 110 ]
}
~~~

Outro aspecto documentado, mas que pode pegá-lo de surpresa, agora ao continuar a _atividade_, é que se por algum motivo o seu método `application(_:didFinishLaunchingWithOptions:)` retornar `false`, o sistema jamais chamará o método `application(_:​continue:​restoration​Handler:​)` e você não poderá concluir o processo. Logo, caso você manipule o retorno do primeiro método, terá de identificar se o app foi aberto por conta de uma _atividade_ inspecionando o dicionário `launchOptions` quanto à presença da chave `user​Activity​Dictionary`, retornando `true` neste caso.

# The Ugly
O Handoff apresenta em algumas ocasiões falhas que podem te deixar louco. Em muitos casos basta tentar novamente e tudo dá certo, porém não é incomum ter que deslogar e logar novamente no iCloud vezes para corrigir algum problema inexplicável, sobretudo quando se está utilizando versões diferentes do iOS (o recurso está disponível desde o iOS 8).  
Entretanto, a coroa vai pra um bug que pouca ou nenhuma informação encontrei a respeito na Web ([1](http://stackoverflow.com/questions/38462031/cant-handoff-from-mac-to-ios-even-though-handoff-from-ios-to-mac-works-fine?noredirect=1#comment72785636_38462031), [2](http://stackoverflow.com/questions/41720356/native-mac-app-fails-to-handoff-to-native-ios-app), [3](https://forums.developer.apple.com/search.jspa?q=handoff)), e que tenho um _DTS_ da Apple em aberto até hoje (reproduziram o problema mas não possuem solução), é o fato de que (ao menos em ambiente de desenvolvimento) um Handoff iniciado no Mac não aparece em outros dispositivos, enquanto do iOS para iOS ou iOS para Mac funciona normalmente. Nosso colega [Renan Protector](https://twitter.com/reprotector){:target="_blank"} utiliza Handoff no [Blogo](https://getblogo.com){:target="_blank"} e segundo ele, em produção tudo funciona. Espero compartilhar da mesma experiência em breve!


# Conclusão
Fora os pequenos grilos, é uma tecnologia muito interessante e útil para se adotar! A experiência do usuário é a privilegiada, mesmo que a custo de alguns cabelos brancos!  
Dúvidas, críticas e sugestões, enviem pelos comentários abaixo, ou diretamente no [Slack iOS Dev BR](http://iosdevbr.herokuapp.com) - `@nobre84`. Até a próxima!

# Referência oficial:
* [Handoff Programming Guide](https://developer.apple.com/library/content/documentation/UserExperience/Conceptual/Handoff/HandoffFundamentals/HandoffFundamentals.html#//apple_ref/doc/uid/TP40014338-CH3-SW1)
