---
layout:     post
title:      "Unity3D e o Mundo Apple"
subtitle:   "Subtitulo?"
date:       2016-03-11 00:00:00
author:     "Mauricio Cardozo"
header-img: "img/Unity3DeoMundoApple/header.jpg"
category:   Categoria
---

# TO-DO
* trocar a pasta de imagens de nome do artigo para username

* Desenvolver o projeto na Unity
* * Procurar assets livres para uso aberto no GitHub

* Escrever o Artigo
* Quebrar o TO-DO em tasks menores



# Perguntas importantes
### Qual é a história que eu quero contar aqui?
#### Como vou estruturar essa história?
### Quem é a pessoa que vai ler este artigo?

# O Projeto na Unity
Nome: Equinox

Precisa ter:
Multitouch, 3D, Gyro/Accelerômetro em iOS e tvOS, Swiping, controlável por mouse e teclado. Câmera tb seria legal.

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

http://www.dannygoodayle.com/2013/02/14/setting-up-your-system-for-unity-ios-development/

https://prime31.com/docs#iosGCMP Plugins úteis para dev iOS

# Possíveis Tópicos
## First Steps


![Unity3D Download Assistant Screenshot](img/Unity3DeoMundoApple/BuildSupport.png)

Não se esqueça de marcar pro Download Assistant baixar o Build Support para as plataformas que vamos usar :)

## Unity Remote
`<Unity Remote 4 na App Store. Link e Screenshot>`
 
Testar builds em aparelhos iOS com a Unity é um processo demorado e chato. Para aqueles testes onde queremos apenas o input do celular, o Remote é uma 
ótima ferramenta de auxílio no desenvolvimento. Ele pode utilizar os sensores do seu celular, como acelerômetro e etc, mas não roda o jogo no hardware dele, apenas renderizando o jogo no Editor no computador, e enviando o vídeo pelo USB e captando os inputs feitos no device. 

## Projeto Compatível Apple Stuff
## Multiplat
### Canvas Settings

## iOS
### Input Handling
#### Multitouch
#### Swipes
#### Gyroscope, Accelerometer
### Integração com coisas nativas
#### GameCenter
#### Exemplo com a câmera
## OSX
### Input Handling de teclado e mouse
## tvOS
### Siri Remote Input Handling
