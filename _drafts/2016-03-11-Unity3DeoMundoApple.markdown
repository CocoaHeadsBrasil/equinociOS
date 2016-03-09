---
layout:     post
title:      "Unity3D e o Mundo Apple"
subtitle:   "Introdução e Integração da engine com iOS"
date:       2016-02-11 00:00:00
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
ótima ferramenta de auxílio no desenvolvimento. Ele pode utilizar os sensores do seu celular, como acelerômetro e etc, mas não roda o jogo no hardware dele, apenas renderizando o jogo no Editor no computador, e enviando o vídeo pelo USB e captando os inputs feitos no device. Um downside é que não dá pra capturar a câmera do device com o remote.

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

A classe `Input` controla todo tipo de input feito no jogo na Unity3D, desde o apertar de teclas no teclado, a movimentação dos analógicos em um controle e claro, por que não, os toques em uma tela. 

#### Multitouch

O `Input` possui suporte fácil ao multitouch, com a propriedade `Input.touches`, que nada mais é do que um array de [toques](http://docs.unity3d.com/ScriptReference/Touch.html), com a posição, delta de movimento e tempo, posição, pressão no caso de aparelhos como o 6S e até se ele veio do dedo do usuário ou de uma [Apple Pencil](http://docs.unity3d.com/ScriptReference/Touch-type.html).

#### Swiping

`<TO-DO>`

Infelizmente a Unity não tem nada para detecção de swipe, então nesse caso nós temos uma alternativa bem legal: Usar uma lib externa, o [TouchKit](https://github.com/prime31/TouchKit)


#### Aceleração

O input de aceleração é reportado como um `Vector3` que representa a aceleração em valores de força G que o device está recebendo. É possível capturar e utilizar estes dados de forma bem simples, com o `Input.acceleration`: 

~~~ csharp

        IEnumerator NonSmoothedAccelerate(){
            while(true){
                transform.localPosition = Input.acceleration * strength;
                yield return new WaitForEndOfFrame();                
            }
        }

~~~

O problema de usar o valor direto assim é esse aqui:

<iframe src='https://gfycat.com/ifr/ShockedAthleticAlligatorsnappingturtle' frameborder='0' scrolling='no' width='600' height='340' allowfullscreen ></iframe>
<span class="caption text-muted">`Input.acceleration`do jeito que ele realmente é</span>

Você muito provavelmente vai querer suavizar estes valores pro jogador não achar que ele tem alguma tremedeira ou coisa do tipo, e é bem simples, é só usar o `SmoothDamp` do próprio `Vector3` que a gente pode definir o valor que vai ser suavizado, a "força" com que ele vai ser suavizado, e o delay da suavização, fazendo algo parecido com o seguinte código:

~~~ csharp

/*
Eu ainda tenho de investigar essa solução, não tô muito confiante no translate desse jeito, acho que ele nem funciona direito?
*/

        IEnumerator Accelerate(){
            Vector3 lastFramePosition = transform.localPosition;
            Vector3 velocity = Vector3.zero;            
            while(true){
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

A Unity permite que o código C# interaja com várias linguagens de programação específicas de outros sistemas, como no nosso caso, o Objective-C, através dos [Native Plugins](http://docs.unity3d.com/Manual/NativePlugins.html). Não só isso, como ela também disponibiliza alguns wrappers básicos, para que você não se perca na burocracia de trazer o resultado do Obj-C para o C#.

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

Com a interoperabilidade de código, podemos fazer coisas interessantes, como reduzir o framerate do nosso jogo quando o iOS9 está em modo Low Power para economizar bateria.

Esse exemplo é bem simples, e mostra como é fácil misturar o Objective C com o C#. Em um arquivo .mm, tenho o seguinte código: 

~~~ objc
extern "C" bool getLowPowerMode(){
    return [[NSProcessInfo processInfo] isLowPowerModeEnabled];
}
~~~ 

E eu acesso esse código na minha classe pelo namespace `System.Runtime.InteropServices`, apenas declarando o nome e o retorno do método no corpo da classe:

~~~ csharp
using System.Runtime.InteropServices;

namespace CocoaHeadsBR{   
    public class DeviceManager {        
        [DllImport ("__Internal")] private static extern bool getLowPowerMode();
        
        public static bool isLowPowerActive{
            get{
                return getLowPowerMode();
            }
        }
    }
}
~~~

E aí, preferencialmente no startup do nosso jogo, modificamos o framerate alvo do jogo, que será menor caso o Low Power Mode esteja ativo:

~~~ csharp
void Start(){
	Application.targetFrameRate = DeviceManager.isLowPowerActive? 30: 60;
}
~~~


#### Câmera

Como com os nossos outros exemplos, a Unity dá uma forma bem fácil de acessar a câmera também. Usando a classe `WebCamTexture`.

<iframe src='https://gfycat.com/ifr/GloomySpitefulAmericanwigeon' frameborder='0' scrolling='no' width='600' height='340' allowfullscreen ></iframe>

Na minha cena tenho algumas texturas que eu quero que sejam fotos que eu vou tirar na hora (Aqueles quadrados brancos ali no meio dos estandartes), e é bem simples de fazer isso funcionar.

Para capturar a imagem, as ações dos botões na tela são estas:

~~~ csharp
        public GameObject cameraOverlay;
        public RawImage camImage; 
        
        WebCamTexture wct;
        
        void Awake(){
            wct = new WebCamTexture();                                               
        }     

        public void StartCamera(){
            camImage.texture = wct;            
            camImage.material.mainTexture = wct;            
            wct.Play(); //Começa a modificar a RawImage que aparecerá como overlay na tela
            cameraOverlay.SetActive(true); //Abre o overlay
        }
        
        public void CaptureImage(){
            cameraOverlay.SetActive(false); //Fecha o overlay
            wct.Stop(); //Para o RawImage no último frame capturado
            CameraSceneManager.sharedInstance.PostPictureIntoStandard(camImage.texture);     
        }
~~~

Para colocar a imagem dentro do jogo, na textura, o `CameraSceneManager` faz isto:

~~~ csharp
        public void PostPictureIntoStandard(Texture photo){
            foreach(Standard s in standards){
                s.ApplyPhoto(photo);    
            }
        }        
~~~

E cada um dos estandartes na cena só faz isso:

~~~ csharp
        public MeshRenderer meshToApplyPhoto; //Esse MeshRenderer é colocado direto no Inspector da Unity
        
        public void ApplyPhoto(Texture photo){
            meshToApplyPhoto.material.mainTexture = photo;
        }
~~~

Fazer funcionar direito é outra história. A texture que a Unity retornou no `WebCamTexture` está invertida, mas como com o exemplo da aceleração, é possível brincar o suficiente com esses settings para deixar o input da câmera bem legalzinho :)


## Conclusão 

Com esse material todo aí vocês já tem uma boa base para desenvolver o jogo usando as features do iOS que tanto queriam. Mal posso esperar para ver o que vocês vão fazer!

O projeto que eu usei para fazer todas as imagens e vídeos utilizados para ilustrar os code samples estão no [GitHub](https://github.com/loloop/equinox) :D

A imagem do Header é do jogo [**Firewatch**](http://www.firewatchgame.com), desenvolvido pela empresa americana [Campo Santo](http://www.camposanto.com) com Unity3D para PS4 e OS X, a imagem que abre o post é de [**SUPER HOT**](https://superhotgame.com), também desenvolvido com Unity3D, para OS X. A Unity tem um espaço de showcase dos jogos desenvolvidos usando ela, o [Made With Unity](http://madewith.unity.com/games?type=featured&search=&platform=ios&genre=). Se você quiser conhecer mais jogos desenvolvidos com ela, é só dar um pulinho lá :)