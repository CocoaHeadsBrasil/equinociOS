---
layout:     post
title:      Testes de Aceitação em iOS
subtitle:   Como escrever testes de aceitação em iOS.
date:       2016-03-05 14:26
author:     Felipe B. Valio
category:   ios

tags:
 - testes de aceitação
 - swift
---

É comum classificar testes automatizados em três tipos: testes unitários, de aceitação e de UI. Existe um concenso de que um balanço ideal entre estes três tipos se dá pela seguinte pirâmide:

![]({{ site.baseurl }}/img/felipebvalio/TestsPyramid.png)

Testes unitários formam a base porque são os mais numerosos. Costumam ser os mais aceitos pelos desenvolvedores por serem fáceis de se escrever e não requererem uma estrutura de código muito específica. Também ajudam a criar um código mais limpo e bem organizado. Porém a sua fraqueza reside no fato desses testes serem isolados e não garantirem a coesão do código em um nível mais elevado. É possível garantir que um método ou classe funcionam corretamente, mas não um fluxo inteiro de navegação.

Testes de UI residem no topo porque com poucos deles é possível abrangir grandes partes do app, porém escrevê-los costuma requerer muito esforço, utilização de frameworks externos e a sua execução demora bastante (pode passar de uma hora). Quanto mais testes de UI um projeto possui, maior a dificuldade em criar mais. Por isso é comum criar poucos testes, apenas abrangendo os fluxos mais básicos.

Chegamos por fim na camada intermediária. Testes de aceitação costumam ser os mais desconhecidos, e por isso, injustamente ignorados. Em minhas experiências com testes automatizados, os de aceitação são os mais valiosos, pois conseguem garantir que qualquer funcionalidade do app continue funcionando ao longo da evolução do app. São bastante flexíveis, abrangentes e, se for seguida uma estrutura de código bem definida, escrevê-los torna-se uma tarefa simples. Podemos, por exemplo, escrever testes para garantir o fluxo de navegação de uma tela em um iPad, iOS 8, orientação horizontal, sem conexão com internet.

Testes de aceitação são também conhecidos como testes subcutâneos porque, fazendo uma analogia com as camadas da pele de uma pessoa, a camada mais externa (e fina) fica excluída dos testes. Nesta camada fica o código difícil de testar, como detalhes de UI e chamadas para o sistema. É muito importante que esta camada seja o mais fina possível, pois assim teremos uma boa cobertura de código sob os testes.

# Escrevendo testes de aceitação

Para escrever os testes de aceitação propostos aqui dois conceitos são necessários: Conductor e Repositórios. O conceito de Conductor é uma extensão do tradicional Model-View-Controller (MVC) e foi criado para remover lógica de dentro dos view controllers. Desta forma, um view controller pode focar apenas em manipular a interface. 

Já os repositórios são usados para garantir um bom encapsulamento de qualquer gerador/consumidor de dados, facilitando assim manipular os dados (_mock_) durante a execução dos testes.

# Conductor em iOS

Vamos construir o conductor para um exemplo prático: uma tela de login. 

Considere uma tela com dois campos de texto (email e senha) e um botão de confirmação. O usuário entra com seus dados, confirma e aguarda a resposta, que pode ser de sucesso (é direcionado para a próxima tela) ou de falha (um pop-up é exibido). O fluxo de telas segue abaixo:

![]({{ site.baseurl }}/img/felipebvalio/SignInFlow.png)


Nossa idéia aqui é remover qualquer lógica do view controller e colocá-la no conductor. Uma boa heurística para avaliar se um view controller possui lógica é verificar se ele possui IFs.

> Um bom view controller é um view controller sem IFs

Vamos atribuir ao view controller e ao conductor as seguintes tarefas:

| View Controller                   | Conductor                             |   
| --------------------------------- | ------------------------------------: |
| Capturar input do usuário         | Validar e-mail e senha                |
| Exibir alertas                    | Comunicar com back-end                |
| Exibir um indicador de atividades | Decidir se o login foi bem-sucedido   |
| Navegar para outras telas         |                                       |

<br>
O diálogo entre o view controller e o conductor será da seguinte forma:

![]({{ site.baseurl }}/img/felipebvalio/ViewController-Conductor.png)

<!-- 
View Controller -> usuário confirmou com o seguinte email e senha: -> Conductor
View Controller <- - - -  exiba o indicador de atividades <- - - - -  Conductor
View Controller <- - - - esconda o indicador de atividades <- - - - - Conductor
View Controller <- -  exiba um alerta com a seguinte mensagem: <- - - Conductor
View Controller <- - - - navegue para a tela de bem-vindo <- - - - -  Conductor
 -->

Vejamos como o nosso conductor pode ser:

{% highlight swift %}
class SignInConductor: NSObject {
    var showActivityIndicator: Void -> Void = {}
    var hideActivityIndicator: Void -> Void = {}
    var navigateToWelcomeScreen: Void -> Void = {}
    var showAlertWithMessage: String -> Void = {message in}
    
    func didConfirmWithEmail(email: String, password: String) {
        
    }
}
{% endhighlight %}

E agora como o view controller utiliza o conductor:

{% highlight swift %}
class SignInViewController: UIViewController {
    @IBOutlet weak var emailTextField: UITextField!
    @IBOutlet weak var passwordTextField: UITextField!

    let signInConductor = SignInConductor()

    override func viewDidLoad() {
        super.viewDidLoad()

        signInConductor.showActivityIndicator = {
            MBProgressHUD.showHUDAddedTo(self.view, animated: true)
        }
        
        signInConductor.hideActivityIndicator = {
            MBProgressHUD.hideHUDForView(self.view, animated: true)
        }
        
        signInConductor.showAlertWithMessage = { message in
            let alert = UIAlertController(title: "", message: message, preferredStyle: UIAlertControllerStyle.Alert)
            alert.addAction(UIAlertAction(title: "Ok", style: UIAlertActionStyle.Default, handler: nil))
            self.presentViewController(alert, animated: true, completion: nil)
        }
        
        signInConductor.navigateToWelcomeScreen = {
            if let welcomeController = self.storyboard?.instantiateViewControllerWithIdentifier("WelcomeViewController") {
                self.showViewController(welcomeController, sender: nil)
            }
        }
    }

    @IBAction func didConfirm() {
        signInConductor.didConfirmWithEmail(emailTextField.text!, password: passwordTextField.text!)
    }
}
{% endhighlight %}

Agora vamos incluir um pouco da lógica necessária para o conductor funcionar:

{% highlight swift %}
class SignInConductor: NSObject {
    var showActivityIndicator: Void -> Void = {}
    var hideActivityIndicator: Void -> Void = {}
    var navigateToWelcomeScreen: Void -> Void = {}
    var showAlertWithMessage: String -> Void = {message in}
    
    func didConfirmWithEmail(email: String, password: String) {
        if !isValidEmail(email) {
            showAlertWithMessage("The email is invalid")
        }
        else if !isValidPassword(password) {
            showAlertWithMessage("The password is invalid")
        }
        else {
            showActivityIndicator()
            performSignInInBackEnd(email: email, password: password) { success in
                self.hideActivityIndicator()
                if success {
                    self.navigateToWelcomeScreen()
                }
                else {
                    self.showAlertWithMessage("Sign in failed")
                }
            }
        }
    }
    
    private func isValidEmail(email: String) -> Bool {
        return email.rangeOfString("@") != nil
    }
    
    private func isValidPassword(password: String) -> Bool {
        return password.characters.count >= 6
    }
    
    private func performSignInInBackEnd(email email: String, password: String, onComplete: Bool -> Void) {
        onComplete(true)
    }
}
{% endhighlight %}

Perceba que temos estruturas bem diferentes no view controller e no conductor. O view controller não toma decisões, apenas executa tarefas, todas simples e isoladas, especificamente focadas em UI. Já o conductor concentra toda tomada de decisão e orienta o view controller. Se existe algum furo na nossa lógica, é no conductor que ele está, e por isso é no conductor que iremos focar os testes.

## Testando um Conductor

Vamos escrever um teste para verificar o fluxo quando o email não for preenchido. Qual seria o fluxo neste caso? Ao pressionar o botão de login, um alerta é apresentado pedindo o email. Não deve ser exibido um indicador de atividades nem deve prosseguir para a tela de bem-vindo. Vejamos o teste para isso:

{% highlight swift %}
class SignInConductorTests: XCTestCase {
    func testSignInWithEmptyEmail() {
        var didShowAlert = false
        let signInConductor = SignInConductor()
        
        signInConductor.showAlertWithMessage = { message in
            didShowAlert = true
        }
        
        signInConductor.navigateToWelcomeScreen = {
            XCTFail("This action should not be executed")
        }
        
        signInConductor.showActivityIndicator = {
            XCTFail("This action should not be executed")
        }
        
        signInConductor.hideActivityIndicator = {
            XCTFail("This action should not be executed")
        }
        
        signInConductor.didConfirmWithEmail("", password: "")
        XCTAssertTrue(didShowAlert)
    }
}
{% endhighlight %}

Qualquer comportamento diferente de exibir um alerta resultará em falha. Este teste não se preocupa saber qual é a mensagem de erro nem qual o motivo do mesmo porque isso não nos interessa agora. Podemos escrever testes para diversos casos, explorando combinações de email e senha válidos e inválidos, mas como todos esses testes seguirão a mesma estrutura, vamos deixar isto como exercício.

Vamos escrever agora um teste para um fluxo de sucesso:

{% highlight swift %}
func testSignInWithValidCredentials() {
    var didShowActivityIndicator = false
    var didHideActivityIndicator = false
    var didNavigateToWelcomeScreen = false
    let signInConductor = SignInConductor()
    
    signInConductor.showAlertWithMessage = { message in
        XCTFail("This action should not be executed")
    }
    
    signInConductor.navigateToWelcomeScreen = {
        didNavigateToWelcomeScreen = true
    }
    
    signInConductor.showActivityIndicator = {
        didShowActivityIndicator = true
    }
    
    signInConductor.hideActivityIndicator = {
        didHideActivityIndicator = true
    }
    
    signInConductor.didConfirmWithEmail("abc@def.ghi", password: "abcdef")
    XCTAssertTrue(didShowActivityIndicator)
    XCTAssertTrue(didHideActivityIndicator)
    XCTAssertTrue(didNavigateToWelcomeScreen)
}
{% endhighlight %}

Temos agora o caso oposto. Não deve ser exibido o alerta de erro e todas as outras ações devem ser tomadas. Podemos tornar a validação ainda mais complexa, por exemplo, para garantir que o indicador de atividades não é ocultado antes de ser exibido, mas vamos deixar as coisas simples por enquanto.

# Repositórios

Vamos agora olhar para o método performSignInInBackEnd. Na prática ele fará uma requisição para algum lugar, mas como isso é feito não nos interessa neste artigo. O que queremos, na verdade, é testar como o conductor se comporta para as diferentes situações que podem decorrer dessa requisição. Para isso iremos fazer um _mock_ da requisição para o back-end, e para isso é necessário que seja trivial substituir tal código por outro bem conhecido e bem comportado.

Usaremos um conceito conhecido como repositório. Um repositório é basicamente um gerador ou consumidor de informações. Acesso ao Keychain, banco de dados, requisições remotas e informações do sistema são exemplos de possíveis repositórios. Um repositório segue uma regra básica: ele deve ser tão bem encapsulado a ponto ponto de ser trivial substituí-lo.

> É necessário ser trivial substituir um repositório por outro equivalente

O nosso SignInConductor se comporta de acordo quando não há conexão com a rede? Vamos escrever um teste para verificar isso.

Primeiro definimos o repositório:

{% highlight swift %}
typealias StatusCode = Int

protocol SignInRepository {
    func trySignInWithEmail(email: String, password: String, onComplete: StatusCode -> Void)
}
{% endhighlight %}

Em segundo lugar, modificamos o conductor para utilizar este repositório:

{% highlight swift %}
class SignInConductor: NSObject {
    let signInRepository: SignInRepository
    
    init(signInRepository: SignInRepository) {
        self.signInRepository = signInRepository
    }
    
    private func performSignInInBackEnd(email email: String, password: String, onComplete: Bool -> Void) {
        signInRepository.trySignInWithEmail(email, password: password) { statusCode in
            onComplete(statusCode == 200)
        }
    }
}
{% endhighlight %}

Terceiro, criamos uma classe concreta para esse repositório, uma que tenha como comportamento sempre falhar com erro 503 (Service unavailable):

{% highlight swift %}
class NoConnectionSignInRepository: SignInRepository {
    func trySignInWithEmail(email: String, password: String, onComplete: StatusCode -> Void) {
        onComplete(503)
    }
}
{% endhighlight %}

E finalmente, utilizamos este repositório no teste:

{% highlight swift %}
func testSignInOffline() {
    var didShowActivityIndicator = false
    var didHideActivityIndicator = false
    var didShowAlert = false
    let signInConductor = SignInConductor(signInRepository: NoConnectionSignInRepository())
    
    signInConductor.showAlertWithMessage = { message in
        didShowAlert = true
    }
    
    signInConductor.navigateToWelcomeScreen = {
        XCTFail("This action should not be executed")
    }
    
    signInConductor.showActivityIndicator = {
        didShowActivityIndicator = true
    }
    
    signInConductor.hideActivityIndicator = {
        didHideActivityIndicator = true
    }
    
    signInConductor.didConfirmWithEmail("abc@def.ghi", password: "abcdef")
    XCTAssertTrue(didShowActivityIndicator)
    XCTAssertTrue(didHideActivityIndicator)
    XCTAssertTrue(didShowAlert)
}
{% endhighlight %}

Temos, com este teste, a garantia de que será exibido um indicador de atividades ao tentar efetuar o login e, ao falhar, uma mensagem será apresentada. Se existirem tratamentos diferentes para erros diferentes, podemos facilmente escrever testes semelhantes, com repositórios equivalentes.

Como também não pode faltar, criaremos um repositório real que será utilizado normalmente pelo app:

{% highlight swift %}
class RemoteSignInRepository: SignInRepository {
    func trySignInWithEmail(email: String, password: String, onComplete: StatusCode -> Void) {
        // do here the code to request a sign-in
    }
}
{% endhighlight %}

Como este último repositório realiza uma chamada assíncrona, possivelmente demorada e dependente de um ambiente de homologação elaborado, não é comum escrever testes de aceitação utilizando-o, o que implica em possuirmos um código não coberto por testes. Por isso é importante manter o máximo possível de código sob o conductor, deixando tanto os view controllers como os repositórios magros e burros.



