---
layout:     post
title:      "Unity3D e o Mundo Apple"
subtitle:   "Introdução e Integração da engine com iOS"
date:       2016-03-01 00:00:00
author:     "Mauricio Cardozo"
header-img: "img/loloop/header.jpg"
category:   gamedev
---

<!-- galera com licença aí mas eu preciso centralizar e limitar umas coisas pro meu post --> 
<style>
video, iframe{
	width: 100%;
}

img, iframe{
	margin: 0 auto;
}

</style>
<!--
HEADER ORIGINAL

---
layout:     post
title:      "Unity3D e o Mundo Apple"
subtitle:   "Introdução e Integração da engine com iOS"
date:       2016-03-11 00:00:00
author:     "Mauricio Cardozo"
header-img: "img/loloop/header.jpg"
category:   gamedev
---

-->
![O fps mais inovador que eu já joguei em anos]({{site.baseurl}}/img/loloop/muchocaliente.png)

Nascida no OS X em 2005 e portada para o resto do mundo todo, a Unity é uma das maiores game engines da atualidade, e uma das melhores escolhas que se pode fazer quando o assunto é gamedev para aparelhos mobile. Com suporte a tantas plataformas que eu não duvidaria que ela funciona até em torradeiras, e isto naturalmente traz aquela dúvida que todo framework que promete mil e uma plataformas traz: Mas realmente funciona?

Sim, funciona. Diferente de apps híbridos, que na maior parte do tempo são várias Web Views com uns hooks para acessar algumas funcionalidades nativas, o core da Unity é todo em C++ ([E o build que ela gera para o iOS também](http://blogs.unity3d.com/2015/05/06/an-introduction-to-ilcpp-internals/)) e ela implementa o [Metal](http://blogs.unity3d.com/2015/04/17/ios-64-bit-and-metal-update/) no iOS, e assim tem uma performance ótima mesmo quando comparada com o SceneKit

Agora que você já conhece um pouco da Unity e o core dela, vamos começar? :)


## Primeiros Passos

![Página Inicial da Unity3D]({{ site.baseurl }}/img/loloop/GetUnity.png)

Bom, vamos começar baixando a Unity né? Não precisa se preocupar muito se ela vai rodar bem no seu Mac, pois estou usando um [MacBook Air de 2010](https://support.apple.com/kb/sp618?locale=en_US) para escrever este artigo. A versão que eu usei pra escrever ele é a 5.3.2, mas já saiu uma versão mais nova, a 5.3.3, mas tudo que eu fizer aqui deve funcionar nela sem o menor dos problemas. O download tá no [site da Unity](http://unity3d.com/download), e a Personal Edition é completa o suficiente pra funcionar com tudo que a gente vai usar.

<img src="{{ site.baseurl }}/img/loloop/BuildSupport.png">

Não se esqueça de marcar pro Download Assistant baixar o Build Support para as plataformas que vamos usar :)

### O Unity Remote

![Unity Remote 4 na App Store]({{ site.baseurl }}/img/loloop/UnityRemote4.png)
<span class="caption text-muted">[Download do UnityRemote na App Store](https://itunes.apple.com/us/app/unity-remote-4/id871767552?mt=8)</span>
 
Testar builds em aparelhos iOS com a Unity é um processo demorado e chato. Para aqueles testes onde queremos apenas o input do celular, o Remote é uma 
ótima ferramenta de auxílio no desenvolvimento. Ele pode utilizar os sensores do seu celular, como acelerômetro e etc, mas não roda o jogo no hardware dele, apenas renderizando o jogo no Editor no computador, e enviando o vídeo pelo USB e captando os inputs feitos no device. 

### Git

<img src="{{ site.baseurl }}/img/loloop/EditorSettings.png">

Para deixar o mais amigável para se trabalhar com o Git, nós precisamos mudar a forma como ela trata com alguns arquivos lá em `Edit > Project Settings > Editor Settings`. Precisamos mudar o Version Control Mode para _Visible Meta Files_ e o Asset Serialization Mode para _Force Text_. Isso faz com que a Unity não esconda os arquivos de configuração dela, e todos os arquivos que ela puder deixar como texto, como cenas, prefabs e até os ProjectSettings sejam textos.

Fica bem mais fácil de identificar problemas assim, já que antes não era possível fazer diff dos arquivos, mas isso deixa os commits um número de linhas bem exagerado, como dá pra ver nos graphs do GitHub de um dos meus projetos

![3mi linhas de código]({{ site.baseurl }}/img/loloop/3MilhoesDeLinhas.png)

### Player Settings

As outras configurações básicas de cada projeto estão no `Edit > Project Settings > Player Settings`, como lock de rotação do device, habilitar split screen multitasking para o iPad Air2 / iPad Pro, estilo e posicionamento da Status Bar, Ícone, Company Name, Bundle ID, Launch Screen, versão do iOS mínima e várias outras coisas. 

## Layouts e Canvas

### Layouts específicos para os devices

Vou começar falando da parte ruim do sistema de UI da Unity, que é o fato dela não saber lidar muito bem com os layouts aparelhos de dpi diferentes e resoluções parecidas, infelizmente nos forçando a fazer algo parecido com isso:

~~~ csharp

//O Device.generation retorna um enum com a versão específica do device, então também é possível utilizar ele para saber se o usuário tem um iPad Pro ou um iPad Air, por exemplo
if(UnityEngine.iOS.Device.generation.ToString().Contains("iPhone")){
	Debug.Log("Estou num iPhone!");
	//Carrega a UI específica do iPhone
} else {
	Debug.Log("Estou num iPad!");
	//Carrega a UI específica do iPad
}

~~~

<!-- http://docs.unity3d.com/ScriptReference/iOS.DeviceGeneration.html -->

### Canvas

A Unity, na versão 4.6, ganhou um sistema novo de UI ([open source, btw](https://bitbucket.org/Unity-Technologies/ui)), com  suporte muito melhor para lidar com múltiplas resoluções, aspectos e variações de DPI. Um dos componentes mais importantes desse sistema é o `Canvas Scaler`, responsável por coordenar a escala dos elementos no aparelho. Existem algumas formas de configurar ele:

* `Constant Pixel Size`: É a configuração padrão do Scaler, e nela os elementos retém o tamanho original deles, e pode ser bem enganadora. É o que causa uma interface a ficar boa em um iPad 2, mas minúscula em um iPad Air. Tem seus usos, mas tome cuidado ao decidir ficar nela.

* `Scale With Screen Size`: Nesta configuração a Unity escala os elementos de acordo com a resolução da tela, e você tem uma resolução virtual para trabalhar com os seus elementos.

* `Constant Physical Size`: Na última configuração, os valores de pontos da Unity são trocados por valores em dp. Esta configuração depende do device reportar adequadamente o dpi de sua tela, e você pode especificar um valor de DPI padrão como fallback.

A configuração mais "segura" de todas é usar o `Scale With Screen Size`, com resolução virtual de 1080p, e o `Match Width or Height` configurado para 0.5. 

<iframe src='https://gfycat.com/ifr/RewardingCreepyHowlermonkey' frameborder='0' scrolling='no' width='600' height='340' allowfullscreen ></iframe>

Também é possível incluir o Canvas dentro do mundo do seu jogo, como um elemento que faz parte dele. Usar isto é útil para quando você quiser colocar um nome em cima dos jogadores, por exemplo, mas desabilita completamente qualquer tipo de escala de acordo com o tamanho da tela.

### Input Handling

A classe `Input` controla todo tipo de input feito no jogo na Unity3D, 

#### Multitouch

O `Input` possui suporte fácil ao multitouch, com a propriedade `Input.touches`, 

#### Swiping

Infelizmente a Unity não tem nada para detecção de swipe, então nesse caso nós temos duas alternativas: Usar uma lib externa, como o [TouchKit](https://github.com/prime31/TouchKit), ou escrever todo o código nós mesmos. 

#### Aceleração

O input de aceleração é reportado como um `Vector3` que representa a aceleração em valores de força G que o device está recebendo. É possível capturar e utilizar estes dados de forma bem simples, com o `Input.acceleration`: 

~~~ csharp

        IEnumerator NonSmoothedAccelerate(){
            move = true;
            while(move){
                transform.localPosition = Input.acceleration * strength;
                yield return new WaitForEndOfFrame();                
            }
        }

~~~

O problema de usar o valor direto assim é esse aqui:

<iframe src='https://gfycat.com/ifr/ShockedAthleticAlligatorsnappingturtle' frameborder='0' scrolling='no' width='600' height='340' allowfullscreen ></iframe>
<span class="caption text-muted">`Input.acceleration`do jeito que ele realmente é</span>

Você muito provavelmente vai querer suavizar estes valores pro jogador não achar que ele tem alguma tremedeira ou coisa do tipo, e é bem simples, é só usar um `SmoothDamp` do próprio `Vector3` que a gente pode definir o valor que vai ser suavizado, a "força" com que ele vai ser suavizado, e o delay da suavização, fazendo algo parecido com o seguinte código:

~~~ csharp

/*
Eu ainda tenho de investigar essa solução, não tô muito confiante no translate desse jeito, acho que ele nem funciona direito?
*/

        IEnumerator Accelerate(){
            move = true;
            Vector3 lastFramePosition = transform.localPosition;
            Vector3 velocity = Vector3.zero;            
            while(move){
                baseline = Input.acceleration;
                Vector3 accelData = Vector3.SmoothDamp(lastFramePosition, baseline, ref velocity, 0.1f); //Inputs do sensor devem ser "limpos", senão ele fica pulando loucamente por aí              
                Vector2 ad = (Vector2) accelData;  
                transform.Translate(ad * strength * Time.deltaTime);
                lastFramePosition = transform.localPosition;            
                yield return new WaitForEndOfFrame();                
            }
        }

~~~

<iframe src='https://gfycat.com/ifr/SarcasticDismalEidolonhelvum' frameborder='0' scrolling='no' width='600' height='340' allowfullscreen ></iframe>
<span class="caption text-muted">`Input.acceleration` com suavização</span>


#### Vibração

Não dá pra ter muito controle da vibração do device mas é bem fácil de fazer ele vibrar, caso o device tenha como fazer isso. (Não adianta tentar num iPad, por exemplo :p )

~~~ csharp
void Vibrar(){
	Handheld.Vibrate();
}
~~~



### Integração com o mundo nativo

`<Interoperabilidade com o código Obj-C e Swift>`

#### GameCenter

As APIs da Unity já provem um nível bem básico de integração com o GameCenter, utilizando a classe [Social](http://docs.unity3d.com/ScriptReference/Social.html), que possui as funcionalidades básicas de autenticação, leaderboards e achievements.

~~~ csharp

        void Start () {
            Social.localUser.Authenticate(SocialUserLogin);
        }
        
        void SocialUserLogin(bool success){
            if(success){
                Debug.Log("do stuff with logged user here");
            } else {                
                Debug.Log("disable gamecenter features until player logs in");
            }
        }
        
~~~

#### Low Power Mode

Com a interoperabilidade de código, podemos fazer coisas interessantes, como reduzir o framerate do nosso jogo quando o iOS9 está em modo Low Power para economizar bateria:

~~~ objc

[[NSProcessInfo processInfo] isLowPowerModeEnabled]


~~~ 

https://developer.apple.com/library/ios/documentation/Performance/Conceptual/EnergyGuide-iOS/LowPowerMode.html

~~~ csharp
class DeviceManager{
	public static bool isLowPowerActive{
		get{
			return true; //implementar método
		}
	}
}
~~~

E aí, preferencialmente no startup do nosso jogo:

~~~ csharp
void Start(){
	Application.targetFrameRate = DeviceManager.isLowPowerActive? 30: 60;
}
~~~


#### Exemplo com a câmera

##### O jeito fácil

`<Vídeo do projeto, pode ser no editor mesmo>`

`WebCamTexture`

<!--
Links úteis

http://docs.unity3d.com/ScriptReference/WebCamTexture.html Uso de câmera

http://answers.unity3d.com/questions/232547/how-to-access-camera-feed-ios-and-android.html Uso de câmera

http://answers.unity3d.com/questions/221621/where-is-ios-camera-support-documented.html Uso de câmera
-->

##### O jeito correto

`<Vídeo do projeto """buildado""" capturando input da câmera e colocando no jogo>`

`UIImagePickerView` nativo

<!--
Links úteis: dá uma olhada lá no teu Pocket, Mauricio
-->


## Conclusão 

Com esse material todo aí vocês já tem uma boa base para desenvolver o jogo de iOS que tanto queriam. Mal posso esperar para ver o que vocês vão fazer!

Todas as imagens e vídeos utilizados para ilustrar os code samples estão no meu [GitHub](https://github.com/loloop/equinox), embalados num pacote bonitinho que tem todo o texto aqui escrito, com uns minigames bem mini divertidinhos pra vocês :D

A imagem do Header é do jogo [**Firewatch**](http://www.firewatchgame.com), desenvolvido pela empresa americana [Campo Santo](http://www.camposanto.com) com Unity3D para PS4 e PC, a imagem que abre o post é de [**SUPER HOT**](https://superhotgame.com), também desenvolvido com Unity3D, para PC. A Unity tem um espaço de showcase dos jogos desenvolvidos usando ela, o [Made With Unity](http://madewith.unity.com/games?type=featured&search=&platform=ios&genre=). Se você quiser conhecer mais jogos desenvolvidos com ela, é só dar um pulinho lá :)