---
layout:     post
title:      "WebView"
subtitle:   "A Porta de entrada pra desenvolvedores web"
date:       2016-03-13 00:00:00
author:     "Emiliano Barbosa"
header-img: "img/emilianoeloi/bg.jpg"
category:   WebView
---

## Introdução

O desenvolvimento de aplicativos chegaram pra valer nas empresas, só que elas estão acabando de se tornar fluentes em mobile web e é natural ver como caminho viável colocar o site mobile dentro de uma app, afinal - Já tenho um site que parece um App, Porque não usar o mesmo? Eu acredito que em boa parte dos casos isso pode ser feito, mas é preciso ficar de olhos nos detalhes de implementação e principalmente a expectativa do usuário, que espera na maioria dos casos um desempenho superior ao de um site.
A WebView pode ser implementada utilizando a WKWebView (WK) e a UIWebView (UI), essa última acompanha o sistema desde sua versão 2, a WK foi introduzida com a versão 8 e apresenta um desempenho muito superior. Sua implementação de uma reserva alguns desafios para as soluções que demandem comunicação do código nativo com o web, gerenciamento de cookies etc.
Todos os exemplos de código desse artigo fazem parte desse projeto no Github: [equinociOS-WebView](https://github.com/emilianoeloi/equinociOS-WebView).

### Ponte de comunicação Javascript/Objective-C

Se eu estou dentro de uma App é claro que queremos usar o melhor dos dois mundos e é por isso que não raro a comunicação entre as plataforma será necessária. A velha UIWebView não apresenta uma forma objetiva de executar javascript, e dessa forma nada de comunicação fácil.

* ObjC to JS

Para enviar um javascript para a página será necessário incluir o código no método `'webView: shouldStartLoadWithRequest: navigationType:'`, assim antes do carregamento da página o código será incluído no contexto da página.

```javascript
(function(){
    window.isInnerEquinocios = function(){
        return document.location.hostname == "equinocios.com"
    }
    window.hideHeader = function(){
        var nav = document.querySelector("nav");
        if(nav && window.isInnerEquinocios()){
            document.body.removeChild(nav);
        }
    }
    window.changeNavTitle = function(){
        document.location.href = "JStoObjC://title="+document.title;
    }
    window.onload = function(){
        window.hideHeader();
        window.changeNavTitle();
    }
})();
```

```objc
- (void)injectJavascript:(NSString *)resource {
    NSString *jsPath = [[NSBundle mainBundle] pathForResource:resource ofType:@"js"];
    NSString *js = [NSString stringWithContentsOfFile:jsPath encoding:NSUTF8StringEncoding error:NULL];

    [self.uiWebView stringByEvaluatingJavaScriptFromString:js];
}
-(BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType{

    [self injectJavascript:@"scripts"];

    return YES;

}
```

*JS to ObjC

Obviamente o inverso não se trata de enviar código nativo para a App, é preciso estabelecer um protocolo de comunicação via url, por exemplo: `JStoObjC://title=equinociOS`, esse padrão deverá ser identificado do lado da App no método `webView: shouldStartLoadWithRequest: navigationType:` e então executar o código nativo.

```objc
-(BOOL)isJStoObjcSchema:(NSString *)url{
    return [url rangeOfString:@"JStoObjC://"].location != NSNotFound;
}
-(NSString *) titleWithUrl:(NSString *)url{
    NSString *title;
    NSArray *urlParts = [url componentsSeparatedByString:@"="];
    if (urlParts) {
        title = urlParts[1];
        title = [title stringByRemovingPercentEncoding];

    }
    return title;
}
-(BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType{

    NSString *absoluteUrl = [request URL].absoluteString;

    if([self isJStoObjcSchema:absoluteUrl]){
        self.navigationItem.title = [self titleWithUrl:absoluteUrl];
        return NO;
    }

    [self injectJavascript:@"scripts"];
    NSLog(@"shoulrStart: %@",[request URL]);
    return YES;
}
```

O Projeto [WebViewJavascriptBridge](https://github.com/marcuswestin/WebViewJavascriptBridge) faz o trabalho descrito acima de uma maneire bem mais completa.

#### WKWebView

O formato acima só seria obrigatório para atender a ~6% de base de dispositivos que ainda rodam o iOS7. Já para os aparelhos com iOS8+ a WKWebView apresentações uma solução bem mais elegante, veja

* Javascript

```javascript
(function(){
    window.isInnerEquinocios = function(){
        return document.location.hostname == "equinocios.com"
    }
    window.hideHeader = function(){
        var nav = document.querySelector("nav");
        if(nav && window.isInnerEquinocios()){
            document.body.removeChild(nav);
        }
    }
    window.changeNavTitle = function(){
        setTimeout(function(){
            window.webkit.messageHandlers.observe.postMessage(document.title);
        },1000);
    }
    window.onload = function(){
        window.hideHeader();
        window.changeNavTitle();
    }
})()
```

* JS to ObjC

A parte pesada aqui fica por conta do setup, no qual será necessário instanciar o `WKUserContenetController`, adicionar o `messageHandler` e implementar o método que vai receber a mensagem vinda da página `userContentController:didReceiveScriptMessage:`.

```objc
@interface ViewController () <WKNavigationDelegate, WKUIDelegate, UIWebViewDelegate, WKScriptMessageHandler>
```

```objc
-(void)setupWKWebView{
    WKWebViewConfiguration *theConfiguration = [[WKWebViewConfiguration alloc] init];
    WKUserContentController *controller = [[WKUserContentController alloc]init];
    [controller addScriptMessageHandler:self name:@"observe"];

    [theConfiguration setUserContentController:controller];
    self.wkWebView = [[WKWebView alloc] initWithFrame:CGRectZero configuration:theConfiguration];

    self.wkWebView.navigationDelegate = self;
    self.wkWebView.UIDelegate = self;
}
```

```objc
-(void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message{
    self.navigationItem.title = message.body;
}
```

* ObjC to JS
```objc
-(void)webView:(WKWebView *)webView didFinishNavigation:(WKNavigation *)navigation{

    NSString *jsPath = [[NSBundle mainBundle] pathForResource:@"wk_script" ofType:@"js"];
    NSString *js = [NSString stringWithContentsOfFile:jsPath encoding:NSUTF8StringEncoding error:NULL];
    [self.wkWebView evaluateJavaScript:js completionHandler:nil];

}
```

### Trabalhando com Cookies

Mas nem tudo são flores, o WKWebView não consegue usar de forma satisfatória o 'NSHTTPCookieStorage', e nesse caso o potencial do Javascript deve ser utilizado no processo de manipulação de cookies. Caso o seu projeto tenha por exemplo, um login nativo e que precise passar o token para a página para mandar o usuário logado na web você vai precisar escrever, deletar ou ler cookies da WebView.

Na UI a manipulação de cookies é feito via `NSHTTPCookieStorage`.

* Gravando um Cookie

```objc
-(void)saveCookie:(NSString *)key value:(NSString *)value{
    NSMutableDictionary *cookieProperties = [NSMutableDictionary dictionary];
    [cookieProperties setObject:key forKey:NSHTTPCookieName];
    [cookieProperties setObject:value forKey:NSHTTPCookieValue];
    [cookieProperties setObject:@"equinocios.com" forKey:NSHTTPCookieDomain];
    [cookieProperties setObject:@"equinocios.com" forKey:NSHTTPCookieOriginURL];
    [cookieProperties setObject:@"/" forKey:NSHTTPCookiePath];
    [cookieProperties setObject:@"0" forKey:NSHTTPCookieVersion];

    NSHTTPCookie *cookie = [NSHTTPCookie cookieWithProperties:cookieProperties];
    [[NSHTTPCookieStorage sharedHTTPCookieStorage] setCookie:cookie];

}
```

* Deletando um Cookie

```objc
-(void)deleteCookie:(NSString *)key{
    NSHTTPCookieStorage *storage = [NSHTTPCookieStorage sharedHTTPCookieStorage];
    for (NSHTTPCookie *cookie in [storage cookies]) {
        if ([cookie.name isEqualToString:key]) {
            [storage deleteCookie:cookie];
        }
    }
    [[NSUserDefaults standardUserDefaults] synchronize];
}
 ```

* Obtendo um Cookie

```objc
-(NSString *)cookie:(NSString *)key{
    NSArray *httpCookies = [[NSHTTPCookieStorage sharedHTTPCookieStorage] cookies];
    for (NSHTTPCookie *cookie in httpCookies) {
        if([[cookie name] isEqualToString:key]){
            return [cookie value];
        }
    }
    return nil;
}
```

Já para Manipular o cookie na WK precisaremos trabalhar com uma implementação javascript, vejo isso como um benefício já que o desenvolvedor web poderá fazer implementações otimizadas de acordo com sua necessidade.

```javascript
window.cookieMng = {
        "set": function(cname,value){
            document.cookie=cname+"="+value;
        },
        "get": function(cname){
            var name = cname + "=";
            var ca = document.cookie.split(';');
            for(var i=0; i<ca.length; i++) {
                var c = ca[i];
                while (c.charAt(0)==' ') c = c.substring(1);
                if (c.indexOf(name) == 0)
                    return c.substring(name.length,c.length);
            }
            return "";
        },
        "delete": function(cname){
            document.cookie=cname+"=";
        }
    }
```

```objc
-(void)wkSaveCookie:(NSString *)key value:(NSString *)value{
    NSString *js = [NSString stringWithFormat:@"window.cookieMng.set('%@','%@');",key,value];
    [self.wkWebView evaluateJavaScript:js completionHandler:nil];
}
-(void)wkDeleteCookie:(NSString *)key{
    NSString *js = [NSString stringWithFormat:@"window.cookieMng.delete('%@');",key];
    [self.wkWebView evaluateJavaScript:js completionHandler:nil];
}
-(void)wkCookie:(NSString *)key completion:(WKCookieCompletion)completion{
    NSString *js = [NSString stringWithFormat:@"window.cookieMng.get('%@');",key];
    [self.wkWebView evaluateJavaScript:js completionHandler:^(id jsReturn, NSError * error) {
        NSString *local = [NSString stringWithFormat:@"%@",jsReturn];
        if ([local isEqualToString:@""]) {
            completion(nil);
        }
        completion(jsReturn);
    }];
}
```

### Performance

Nesse ponto que as coisas começam a complicar, o que se espera de um aplicativo é que seja performático e uma WebView nem sempre entrega isso de forma aceitável, caso seu conteúdo seja complexo, com muitas imagens, fontes customizadas, chamadas ajax etc isso tende a degradar o carregamento das páginas e não haverá cache que ajudará um segundo carregamento, já que além da obtenção dos dados o que torna uma página web rápida é também como ela foi construída.

#### Cache

A política de cache padrão de um request é a `NSURLRequestUseProtocolCachePolicy` a imagem a baixo (obtida da própria referência da apple) descreve seu comportamento. Existem algumas outras políticas de para os diversos casos: Cache parcial sem cache etc.

<img src="{{ site.baseurl }}/img/emilianoeloi/cache_policy.png">

* Request com política de Cache

 ```objc
 -(void)loadWKWebViewWithUrl:(NSString *)absoluteUrl{
    NSURL *url = [NSURL URLWithString:absoluteUrl];
    NSURLRequest *request = [NSURLRequest requestWithURL:url cachePolicy:NSURLRequestReloadIgnoringLocalAndRemoteCacheData timeoutInterval:1.0];
    [_wkWebView loadRequest:request];
}
 ```

* Limpar Cache

No caso de utilização de WebView é notório o consumo de memória, em específico da UIWebView em iOS 8+, e limpar o cache em caso de MemoryWarning ajudará a manter o bom funcionamento do seu aplicativo.

> In apps that run in iOS 8 and later, use the WKWebView class instead of using UIWebView. Additionally, consider setting the WKPreferences property javaScriptEnabled to false if you render files that are not supposed to run JavaScript. UIWebView Reference

 ```objc
 - (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    [[NSURLCache sharedURLCache] removeAllCachedResponses];
    [[NSURLCache sharedURLCache] setDiskCapacity:0];
    [[NSURLCache sharedURLCache] setMemoryCapacity:0];
}
 ```

 * HTML embarcado

Existe também a opção de carregar o HTML previamente embarcado no aparelho.

```objc
-(void)loadUIWebViewWithLocalData{
    NSString *htmlFile = [[NSBundle mainBundle] pathForResource:@"index" ofType:@"html"];
    NSString* htmlString = [NSString stringWithContentsOfFile:htmlFile encoding:NSUTF8StringEncoding error:nil];
    [self.uiWebView loadHTMLString:htmlString baseURL:[NSURL URLWithString:@"http://equinocios.com"]];
}
```

### HTML

Preocupar-se com a performance do código web para uma WebView é ainda mais relevante, além de ela ser uma versão piorada do navegador, estarmos em um dispositivo que precisa otimizar o consumo de bateria. Então turbinar seu código vai ajudar substancialmente a sua WebView rodar suave. A idéia que o código seja escrito de maneira minimizar reflows, repaints e todo script que possa bloquear a interação do usuário.

### WebKit

Embora a WKWebView tenha sido lançada com o iOS8 em 2014 o Google Chrome, por exemplo só foi adotá-la no início desse ano e só usa para iOS9+. E como era de se esperar a diferença de performance é gritante. Segue abaixo um comparativo da UIWebView vs WKWebView. Um dos motivos que foi citado pelo Google pra não utilização do WK é não ter um caminho obvio para gerenciar cookies.

Observe no consumo de recursos da comparação abaixo:

<img src="http://emiliano.bocamuchas.org/__ui_to_gif_final.gif">
<img src="http://emiliano.bocamuchas.org/__wk_to_gif_final.gif">

## Ferramenta de inspeção

E para um desenvolvedor web treinada nada é mais fundamental do que o inspect do navegador, e para a WebView isso continua igual, obviamente que é a ferramenta do Safari. E de simples utilização, basta habilitar o modo desenvolvedor do Safari e o menu desenvolvedor ficará disponível.

<img src="{{ site.baseurl }}/img/emilianoeloi/inspect.png">

## Browser inApp.

E para os que querem manter seu usuário ainda no contexto do seu aplicativo, já que está disponível para iOS9+ o Safari View Controller, que é uma experiência completa de um browser dentro da sua App. Ele apresenta uma experiência consistente com o próprio Safari levando o auto-preenchimento de formulários cookies, ou seja se o usuário estiver logado no Safari estará logado na SVC.

```objc
-(BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType{

    NSString *absoluteUrl = [request URL].absoluteString;

    if([self isJStoObjcSchema:absoluteUrl]){
        self.navigationItem.title = [self titleWithUrl:absoluteUrl];
        return NO;
    }

    if (![self isInnerURL:absoluteUrl] && navigationType == UIWebViewNavigationTypeLinkClicked) {
        SFSafariViewController *svc = [[SFSafariViewController alloc]initWithURL:request.URL];
        [self presentViewController:svc animated:YES completion:^{

        }];
        return NO;
    }

    [self injectJavascript:@"ui_script"];
    return YES;
}
```

## Conclusão

A WebView integra a App e seus recursos nativos à web, ou seja você pode ter o melhor dos dois mundo ao seu favor.
Existem soluções para web mobile que beiram o inacreditável de tão boa de usar, muitas delas superam muitas Apps por aí, mas é muito interessante entender até onde soluções web podem chegar e principalmente até onde uma WebView pode solucionar o problema proposto.
Existem cenários em que a solução pode parecer tanto um aplicativo que um usuário treinado não conseguir identificar, mas isso não será verdade em todos os casos, nos quais o conteúdo é complexo demais pra funcionar com fluidez, e o melhor para esses cacos é já deixar claro para o usuário que se trata de um acesso a web e isso já calibrará a expectativa dele.
E essa série de artigos do CocoaHeads é uma ótima oportunidade para desenvolvedores web se envolverem com a plataforma e entender que é tão interessante quanto a web e poder ter mais insumos para desenvolver soluções para Mobile.

### Agradecimentos

Agradeço [Solli](https://github.com/shonorio) pela inciativa do projeto que celebra o Equinócio e a todos os membros da comunidade do CocoaHeads que prontamente absorveu a sugestão e em poucos dias já deixaram tudo preparado para um mês de artigos. Pra mim é um privilégio.

### Referências

1. [AppStore](https://developer.apple.com/support/app-store/)
2. [WebViewJavascriptBridge](https://github.com/marcuswestin/WebViewJavascriptBridge)
3. [NSHTTPCookieStorage](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSHTTPCookieStorage_Class/)
5. [Minimizing browser reflow](https://developers.google.com/speed/articles/reflow#guidelines)
6. [Rendering: repaint, reflow/relayout, restyle](http://www.phpied.com/rendering-repaint-reflowrelayout-restyle/)
7. [Using JavaScript with WKWebView in iOS 8](http://www.joshuakehn.com/2014/10/29/using-javascript-with-wkwebview-in-ios-8.html)
8. [A faster, more stable Chrome on iOS](http://blog.chromium.org/2016/01/a-faster-more-stable-chrome-on-ios.html)
9. [Use WKWebView on iOS 9+](https://bugs.chromium.org/p/chromium/issues/detail?id=423444)
