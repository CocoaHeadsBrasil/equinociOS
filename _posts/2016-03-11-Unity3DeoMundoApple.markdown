---
layout:     post
title:      "Unity3D e o Mundo Apple"
subtitle:   "Introdução e Integração da engine com iOS"
date:       2016-03-01 00:00:00
author:     "Mauricio Cardozo"
header-img: "img/loloop/header.jpg"
category:   Categoria
---

<!-- galera com licença aí mas eu preciso centralizar e limitar umas coisas pro meu post --> 
<style>
video{
	width: 100%;
}

img{
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
category:   Categoria
---

-->

Escrever uma rápida introdução, talvez?

## Primeiros Passos

<img src="{{ site.baseurl }}/img/loloop/GetUnity.png">

Bom, vamos começar baixando a Unity né? A versão que eu usei pra escrever esse artigo é a 5.3.2, mas já saiu uma versão mais nova, a 5.3.3, mas tudo que eu vou fazer aqui deve funcionar nela sem o menor dos problemas. O download tá no [site da Unity](http://unity3d.com/download), e a Personal Edition é completa o suficiente pra funcionar com tudo que a gente vai usar.

<img src="{{ site.baseurl }}/img/loloop/BuildSupport.png">

Não se esqueça de marcar pro Download Assistant baixar o Build Support para as plataformas que vamos usar :)

### O Unity Remote

<img src="{{ site.baseurl }}/img/loloop/UnityRemote4.png">

[Download do UnityRemote na App Store](https://itunes.apple.com/us/app/unity-remote-4/id871767552?mt=8)
 
Testar builds em aparelhos iOS com a Unity é um processo demorado e chato. Para aqueles testes onde queremos apenas o input do celular, o Remote é uma 
ótima ferramenta de auxílio no desenvolvimento. Ele pode utilizar os sensores do seu celular, como acelerômetro e etc, mas não roda o jogo no hardware dele, apenas renderizando o jogo no Editor no computador, e enviando o vídeo pelo USB e captando os inputs feitos no device. 

### Git

<img src="{{ site.baseurl }}/img/loloop/EditorSettings.png">

Para deixar o mais amigável para se trabalhar com o Git, nós precisamos mudar a forma como ela trata com alguns arquivos lá em `Edit > Project Settings > Editor Settings`. Precisamos mudar o Version Control Mode para _Visible Meta Files_ e o Asset Serialization Mode para _Force Text_. Isso faz com que a Unity não esconda os arquivos de configuração dela, e todos os arquivos que ela puder deixar como texto, como cenas, prefabs e até os ProjectSettings sejam textos.

Fica bem mais fácil de identificar problemas assim, já que antes não era possível fazer diff dos arquivos, mas isso deixa os commits um número de linhas bem exagerado, como dá pra ver nos graphs do GitHub de um dos meus projetos

<img src="{{ site.baseurl }}/img/loloop/3MilhoesDeLinhas.png" style="margin: 0 auto">

### Player Settings

As outras configurações básicas de cada projeto estão no `Edit > Project Settings > Player Settings`, como lock de rotação do device, habilitar split screen multitasking para o iPad Air2 / iPad Pro, estilo e posicionamento da Status Bar, Ícone, Company Name, Bundle ID, Launch Screen, versão do iOS mínima e várias outras coisas. 

## Layouts e Canvas

### Layouts específicos para os devices

Vou começar falando da parte ruim da parte de UI da Unity, que é o fato dela não saber lidar muito bem com os layouts aparelhos de dpi diferentes e resoluções parecidas, infelizmente nos forçando a fazer algo parecido com isso:

~~~ csharp

if(UnityEngine.iOS.Device.generation.ToString().Contains("iPhone")){
	Debug.Log("Estou num iPhone!");
	//Carrega a UI específica do iPhone
} else {
	Debug.Log("Estou num iPad!");
	//Carrega a UI específica do iPad
}

~~~

<!-- http://docs.unity3d.com/ScriptReference/iOS.DeviceGeneration.html -->

### Canvas Settings

### UI Layouts

### Input Handling

#### Multitouch

A Unity possui suporte fácil a 

#### Swiping

#### Gyroscope, Accelerometer

<video src="https://zippy.gfycat.com/ShockedAthleticAlligatorsnappingturtle.mp4">

<span class="imageSubtitle">
`Input.accelerometer`do jeito que ele realmente é
</span>

<video src="https://zippy.gfycat.com/SarcasticDismalEidolonhelvum.mp4">

`Input.accelerometer` suavizado entre os frames

#### Vibração

Não dá pra ter muito controle da vibração do device mas é bem fácil de fazer ele vibrar, caso o device tenha como fazer isso. (Não adianta tentar num iPad, por exemplo :p )

~~~ csharp
void Vibrar(){
	Handheld.Vibrate();
}
~~~

### Integração com coisas nativas
#### GameCenter
#### Exemplo com a câmera


Colocar isso aqui no fim do post pra concluir!111!!! 
http://madewith.unity.com/games?type=featured&search=&platform=ios&genre=
E mostrar o projeto de exemplo lá github.com/loloop/equinox


FIM DO POST
------------------------------

# Perguntas importantes
### Qual é a história que eu quero contar aqui?
#### Como vou estruturar essa história?
### Quem é a pessoa que vai ler este artigo?

# O Projeto na Unity
Nome: Equinox

Precisa ter:
Multitouch, 3D, Gyro/Accelerômetro em iOS, Swiping. Câmera tb seria legal.

Minha solução? Vários minigames num proj só. Quem sabe depois a gente tenta encaixar isso direito em uma unidade. Ou corta features.


Links úteis:

https://littlebitesofcocoa.com/192-being-a-good-low-power-mode-citizen afetar settings do app com isso aqui seria bem legal

http://docs.unity3d.com/ScriptReference/WebCamTexture.html Uso de câmera

http://answers.unity3d.com/questions/232547/how-to-access-camera-feed-ios-and-android.html Uso de câmera

http://answers.unity3d.com/questions/550729/how-to-access-device-iphoneandroid-native-camera.html Uso de câmera

http://answers.unity3d.com/questions/221621/where-is-ios-camera-support-documented.html Uso de câmera

https://gist.github.com/Democide/beba5fd2603b268a8f72 Loading Screen Gist

https://github.com/dsoft20/psx_retroshader Esse shader lindo

https://github.com/nickgravelyn/UnityToolbag Ver coisas daqui, talvez tenha algo interessante

https://github.com/fholm/unityassets ver se os assets daqui tb podem se encaixar

https://github.com/RyanNielson/awesome-unity Varrer esta lista em busca de algo útil

https://www.reddit.com/r/gamedev/comments/3hst9l/best_unity_assets_by_keijiro_takahashi_including/ Shaders do Keijiro

http://kenney.nl/assets dar uma olhada nos assets do kenney

https://github.com/keijiro?tab=repositories

https://github.com/unity3d-jp

https://github.com/unity3d-jp/EasyFlightGame

Artigos que podem conter alguma informação importante que eu ainda não pensei em

http://www.raywenderlich.com/25205/beginning-unity-3d-for-ios-part-13

http://docs.unity3d.com/Manual/iphone-GettingStarted.html

https://prime31.com/docs#iosGCMP Plugins úteis para dev iOS

