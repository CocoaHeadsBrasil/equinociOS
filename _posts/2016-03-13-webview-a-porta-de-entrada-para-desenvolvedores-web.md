WebView
A Porta de entrada pra desenvolvedores web

- Introdução

Os aplicativos chegaram pra valer nas empresas, só que elas estão acabando de se tornar fluentes em mobile web. É natural ver o caminho de colocar o site mobile dentro de uma app, - Já tenho um site que parece um App! Porque não usar o mesmo? Eu acredito que em boa parte dos casos isso pode ser feito, mas é preciso ficar de olhos nos detalhes de implementação e principalmente a expectativa do usuário, afinal de contas é uma App e uma App de ver voar, no mínimo.

- Ponte de comunicação Javascript/Objective-C

O premeiro desafio é fazer essa conversa acontecer, a velha UIWebView não apresenta uma forma objetiva de executar javascript, e dessa forma nada de conversa fácil ente código nativo e Javascript.
 - ObjC to JS
Para enviar um javascript para a página, será necessário incluir o código no método 'webView: shouldStartLoadWithRequest: navigationType:', assim antes do carregamento da página é possível incluir no seu contexto qualquer código JS.

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

 - JS to ObjC
O inverso não é tão simples, não é possível chamar o código ObjC diretamente, é preciso estabelecer um protocolo de comunicação via url, por exemplo: JStoObjC://title=equinociOS.
Esse padrão deverá ser identificado no mesmo método 'webView: shouldStartLoadWithRequest: navigationType:' e aí sim executar o código nativo.

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

O Projeto 'WebViewJavascriptBridge' de 'Marcus Westin' faz o trabalho descrito acima de uma maneira genérica permitindo a execução dos scripts a qualquer momento.

O formato acima só seria mandatório para atender a ~6% de base de dispositivos que ainda rodam o iOS7, entretanto para os dispositivos com as versões do iOS 8+ está disponível a WebKit WebView, que é inclusive uma recomendação de uso da Apple para essas versões de iOS. A comunicação Javascript/Código Nativo já está em um nível bem superior.

-Javascript
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

-JStoObjC
A parte pesada aqui fica por conta do setup, no qual será necessário instanciar o 'WKUserContenetController' e adicionar o 'messageHandler', e com a implementação do método 'userContentController:didReceiveScriptMessage:'.
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
-ObjCtoJS
```objc
-(void)webView:(WKWebView *)webView didFinishNavigation:(WKNavigation *)navigation{

    NSString *jsPath = [[NSBundle mainBundle] pathForResource:@"wk_script" ofType:@"js"];
    NSString *js = [NSString stringWithContentsOfFile:jsPath encoding:NSUTF8StringEncoding error:NULL];
    [self.wkWebView evaluateJavaScript:js completionHandler:nil];

}
```

- Trabalhando com Cookies

Mas nem tudo são flores, o WebView do WebKit não consegue usar de forma satisfatória o 'NSHTTPCookieStorage', e nesse caso o potencial Javascript deve ser utilizado no processo de manipulação de cookies. Caso o seu projeto tenha por exemplo, um login nativo e que precise passar o token para a página para manar o usuário logado na web você vai precisar escrever, deletar ou ler cookies da Webview.
Começando com a UIWebView, a manipulação de cookies se dá de forma muito eficiente utilizando o 'NSHTTPCookieStorage'.

 - Gravando um Cookie
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

 - Deletando um Cookie
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

 - Obtendo um Cookie
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

Já para Manipular o cookie na WK precisaremos trabalhar com uma implementação javascript, vejo isso como um benefício já que o time de web poderá fazer implementações otimizadas de acordo com sua necessidade. Para criar um cookie é necessário que, além de executar o script de criação do Cookie, que página seja carregada na sua totalidade para que o cookie seja criado/deletado efetivamente.

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

- Cache e Performance

Nesse ponto que as coisas começam a complicar, o que se espera de um aplicativo é que seja performático e uma Webview nem sempre entrega isso de forma aceitável, caso seu conteúdo seja complexo, com muitas imagens, fontes customizadas, chamadas ajax etc isso tende a degradar o carregamento das páginas e não haverá cache que ajudará um segundo carregamento, já que além da obtenção dos dados o que torna uma página web rápida é também como ela foi construída.
A política de cache padrão de um request é a 'NSURLRequestUseProtocolCachePolicy' a imagem a baixo (obtida da própria referência da apple) descreve seu comportamento. Existem algumas outras políticas de para os diversos casos: Cache parcial sem cache etc.

 - Request com política de Cache
 ```objc
 -(void)loadWKWebViewWithUrl:(NSString *)absoluteUrl{
    NSURL *url = [NSURL URLWithString:absoluteUrl];
    NSURLRequest *request = [NSURLRequest requestWithURL:url cachePolicy:NSURLRequestReloadIgnoringLocalAndRemoteCacheData timeoutInterval:1.0];
    [_wkWebView loadRequest:request];
}
 ```

No caso de utilização de webview é notório o consumo de memória, em específico da UIWebView em iOS 8+, e limpar o cache em caso de MemoryWarning ajudará a manter o bom funcionamento do seu aplicativo.

"In apps that run in iOS 8 and later, use the WKWebView class instead of using UIWebView. Additionally, consider setting the WKPreferences property javaScriptEnabled to false if you render files that are not supposed to run JavaScript." UIWebView Reference

 - Limpar cache
 ```objc
 - (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    [[NSURLCache sharedURLCache] removeAllCachedResponses];
    [[NSURLCache sharedURLCache] setDiskCapacity:0];
    [[NSURLCache sharedURLCache] setMemoryCapacity:0];
}
 ```

E mesmo que a escolha seja a de colocar o html embarcado ele ainda terá o passo atrás de renderizar HTML+CSS+Javascript e não uma View da plataforma.

```objc
-(void)loadUIWebViewWithLocalData{
    NSString *htmlFile = [[NSBundle mainBundle] pathForResource:@"index" ofType:@"html"];
    NSString* htmlString = [NSString stringWithContentsOfFile:htmlFile encoding:NSUTF8StringEncoding error:nil];
    [self.uiWebView loadHTMLString:htmlString baseURL:[NSURL URLWithString:@"http://equinocios.com"]];
}
```

- Performance do HTML
Preocupar-se com a performance do código web para uma webview é ainda mais relevante além de ela ser uma versão piorada do navegador, estarmos em um dispositivo que precisa otimizar o consumo de bateria em alguns momentos. Então turbinar seu código vai ajudar substancialmente a sua webview rodar suave. A idéia que o código seja escrito de maneira minimizar reflows e repaints e obviamente scripts que bloqueiem a interação do usuário.

https://www.youtube.com/watch?v=ZTnIxIA5KGw

- Performance
Embora a WKWebView tenha sido lançada com o iOS8 em 2014 o Google Chrome por exemplo só foi adotá-la no início desse ano e só usa para iOS9+. E como era de se esperar a diferença de performance é gritante. Segue abaixo um comparativo da UIWebView vs WKWebView. Um dos motivos que foi citado pelo google pra não utilização do WK é não ter um caminho obvio para gerenciar cookies.

[Chrome-48-for-iOS.001-640x470]

[vídeo]

- Ferramentas de inspeção
E para um desenvolvedor web treinada nada é mais fundamental do que o inspect do navegador, e para a webview isso continua igual, obviamente que é a ferramenta do Safari. E de simples utilização, basta habilitar o modo desenvolvedor do Safari e o menu desenvolvedor ficará disponível.

[Imagens]

- Browser inApp.
E para os aplicativos que querem manter seu usuário ainda no contexto do seu aplicativo já que está disponível para iOS9+ o Safari View Controller. A SafariViewController apresenta a experiência consistente com o próprio safari levando o auto-preenchimento de formulários cookies, ou seja se o usuário logou no safari e estará logado na safari view controller.

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

- Conclusão

Existem ótimas experiências na web, muitas delas superam muitos apps por aí, mas é muito interessante entender até onde soluções web podem chegar e principalmente até onde uma webview pode solucionar o problema proposto. Podem existir casos em que a solução pode parecer um aplicativo a ponto de um usuário treinado não conseguir identificar. Mas em outros caos, nos quais o conteúdo é complexo demais pra funcionar com fluidez é bem mais interessante já deixar para o usuário que se trata de um acesso a web e isso já calibrará a expectativa do usuário com a app.
E claro, desenvolvedor web, tire um tempinho para aprender as plataformas nativas, dê uma chance e você pode se surpreender e entender que construir um aplicativo pode ser bem interessante também. E obviamente recomendo fortemente essa série de artigos do equinociOS.

- Agradecimentos

Agradeço Solli pela inciativa do projeto que celebra o Equinócio e a todos os membros da comunidade do cocoaheads que prontamente absorveu a sugestão e em poucos dias já deixaram tudo preparado para um mês de artigos. Pra mim é um previlégio.

- Referências
AppStore - https://developer.apple.com/support/app-store/
WebViewJavascriptBridge - https://github.com/marcuswestin/WebViewJavascriptBridge
NSHTTPCookieStorage - https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSHTTPCookieStorage_Class/
Minimizing browser reflow - https://developers.google.com/speed/articles/reflow#guidelines
Rendering: repaint, reflow/relayout, restyle - http://www.phpied.com/rendering-repaint-reflowrelayout-restyle/
Using JavaScript with WKWebView in iOS 8 -http://www.joshuakehn.com/2014/10/29/using-javascript-with-wkwebview-in-ios-8.html
A faster, more stable Chrome on iOS - http://blog.chromium.org/2016/01/a-faster-more-stable-chrome-on-ios.html
Use WKWebView on iOS 9+ - https://bugs.chromium.org/p/chromium/issues/detail?id=423444

--
Emiliano Eloi
tim: (BH) 9451-0018
fb : http://fb.com/emilianoeloi
twt: @emilianoeloi
