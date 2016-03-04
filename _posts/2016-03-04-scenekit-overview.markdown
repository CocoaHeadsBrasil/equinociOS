---
layout:     post
title:      "SceneKit Overview"
subtitle:   "Introdução aos principais conceitos do framework"
date:       2016-03-04 00:10:00
author:     "Lucas Farris"
header-img: "img/farris/bg.jpg"
category:   Tutorial
---

> Este artigo pertence à série de artigos equinociOS, e aqui iremos tratar do framework [SceneKit](https://developer.apple.com/library/ios/documentation/SceneKit/Reference/SceneKit_Framework/), que é uma bibiliteca para desenvolvimento de gráficos 3d de alta performance. O código será escrito em `Swift`, e um exemplo completo do projeto pode ser encontrado [neste repositório](https://github.com/luksfarris/carRush). O presente artigo está licenciado como [CC - Creative Commons](https://creativecommons.org/).
Você irá, acompanhando o texto, escrever umas `150` linhas de código, o tempo médio da leitura é de `30` minutos. 

---

### Prólogo: Declarações iniciais e criando o projeto.

Durante este texto iremos recriar juntos uma versão *minimalista* do fantástico jogo [2 Cars](https://itunes.apple.com/en/app/2-cars/id936839198?mt=8"), mas em um ambiente tridimensional. Com isso aprenderemos sobre:

- Física e colisões
- Texturas e modelos 3d
- Sistemas de partícula
- Animações e interação com o usuário

Para acompanhar não é necessário conhecimento prévio de `Swift`, apenas de programação básica, algumas noções de geometria, e um pouco de conhecimento do `XCode`.


Comece tendo certeza que seu XCode está atualizado, pelo menos na versão `Version 7.2`. Crie um novo projeto, do tipo `Game`, escolha `Swift` para a linguagem, `SceneKit` como tecnologia, e `Universal` nos dispositivos. Salve onde preferir.

![]({{ site.baseurl }}/img/farris/img1.png)

No projeto criado, voce poderá encontrar o arquivo `GameViewController.swift`. Abra ele e vamos comecar!

### Capítulo 1: Luzes, Câmera e Ação!

###### No qual aprendemos a criar câmeras, posicionar elementos, criar materiais e adicionar objetos à cena.
Apague tudo na classe `GameViewController`, e deixe apenas:

```Swift
import UIKit
import QuartzCore
import SceneKit
class GameViewController: UIViewController {
}
```
Em seguida, adicione variáveis pra câmera, pro chão e pra nossa cena:

```Swift
var camera:SCNNode!
var ground:SCNNode!
var scene:SCNScene!
var sceneView:SCNView!
```
Adicione uma função para criar a cena:

```Swift
func createScene () {
  scene = SCNScene()
  sceneView = self.view as! SCNView
  sceneView.scene = scene
  sceneView.allowsCameraControl = true
  sceneView.showsStatistics = true
  sceneView.playing = true
  sceneView.autoenablesDefaultLighting = true
}
```

Adicione uma função responsável por criar a câmera. Note que `.position` é a propriedade que define a posição tridimensional dela, e `eulerAngles` (medidos em radianos) definem a orientação (pra onde ela aponta). Os fotógrafos amadores poderão se divertir com os [demais parâmetros disponíveis para as lentes](http://flexmonkey.blogspot.com/2015/05/depth-of-field-in-scenekit.html).

```Swift
func createCamera () {
  camera = SCNNode()
  camera.camera = SCNCamera()
  camera.position = SCNVector3(x: 0, y: 25, z: -18)
  camera.eulerAngles = SCNVector3(x: -1, y: 0, z: 0)
  scene.rootNode.addChildNode(camera)
}
```

Adicione uma função responsável por criar o chão. `SCNFloor` cria um plano infinito fixado inicialmente na origem. Note que vamos dar uma tonalidade amarela pra ele usando um `SCNMaterial`.

```Swift
func createGround () {
  let groundGeometry = SCNFloor()
  groundGeometry.reflectivity = 0.5
  let groundMaterial = SCNMaterial()
  groundMaterial.diffuse.contents = UIColor.yellowColor()
  groundGeometry.materials = [groundMaterial]
  ground = SCNNode(geometry: groundGeometry)
  scene.rootNode.addChildNode(ground)
}
```

E junte tudo no `ViewDidLoad()`:

```Swift
override func viewDidLoad() {
  super.viewDidLoad()
  createScene()
  createCamera()
  createGround()
}
```

Compile e rode e veja nosso cenário inicial. Use gestos para circular pelo terreno tridimensional.

![]({{ site.baseurl }}/img/farris/img2.png)

### Capítulo 2: A jornada do herói.

###### No qual aprendemos criar ou importar objetos tridimensionais, animá-los e a interagir com o usuário.

Vamos criar um tímido cenário? Faremos uma faixa na nossa rodovia! Adicione este método e chame-o no `viewDidLoad`:

```Swift
func createScenario() {
  for i in 20...70 {
    let laneMaterial = SCNMaterial()
    if i%5<2 { // se a divisao de i por 5 for igual a 0 ou 1
      laneMaterial.diffuse.contents = UIColor.clearColor()
    } else { // se a divisao de i por 5 for 2,3 ou 4
      laneMaterial.diffuse.contents = UIColor.blackColor()
    }
    let laneGeometry = SCNBox(width: 0.2, height: 0.1, length: 1, chamferRadius:0)
    laneGeometry.materials = [laneMaterial]
    let lane = SCNNode(geometry: laneGeometry)
    lane.position = SCNVector3(x: 0, y: 0, z: -Float(i))
    scene.rootNode.addChildNode(lane)
    let moveDown = SCNAction.moveByX(0, y:0 , z: 5, duration: 0.3)
    let moveUp = SCNAction.moveByX(0, y: 0, z: -5, duration: 0)
    let moveLoop = SCNAction.repeatActionForever(SCNAction.sequence([moveDown, moveUp]))
    lane.runAction(moveLoop)
  }
}
```

Ok, tem muita coisa acontecendo aqui, vamos por partes. Estamos dentro de um *loop*, no qual `i` vai assumir todos os valores inteiros entre `20` e `70`. Em cada iteração, colocamos um pequeno tijolinho, `preto` ou `transparente`, dependendo de `i`. Note que isso vai colocar 3 tijolinhos pretos, e 2 transparentes.
Em seguida, adicionamos uma animação ao conjunto. Todos os tijolinhos estão sujeitos a duas animações: `moveUp` e `moveDown`. A animação `moveLoop` combina as duas (usando o método `sequence`), e as repete para sempre (usando `repeatActionForever`). Por fim, `runAction`, que pode ser chamado a qualquer `SCNNode`, aplica a animação em cada um de nossos tijolinhos. Como cada faixa tem 3 tijolinhos pretos + 2 transparentes, nós andamos `5` pra baixo em `0.3` segundos, e instantaneamente subimos `5` pra dar a impressão de que é um movimento contínuo. Tente remover `moveUp` como experimento. Eis o resultado até agora:

![]({{ site.baseurl }}/img/farris/gif1.gif)

Vamos adicionar nosso personagem principal? Adicione esta variável junto com as outras:

```Swift
var car:SCNNode!
```

Em seguida adicione a função `createPlayer`, e <b>chame-a</b> no `viewDidLoad`:

```Swift
func createPlayer(){
  car = SCNNode(geometry: SCNBox(width: 3, height: 2, length: 3, chamferRadius: 0.2))
  let material = SCNMaterial()
  material.reflective.contents = UIColor.blueColor()
  material.diffuse.contents = UIColor.lightGrayColor()
  car.geometry!.materials = [material]
  scene.rootNode.addChildNode(car)
  car.position = SCNVector3(-4,1,-25) // colocamos ele na frente da camera
}
```

Note que precisamos fazer um ajuste de translação para que nosso modelo se encaixasse no cenário. Rode o código, veja o carrinho aparecendo. Vamos adicionar um escapamento? Clique com o botão direito na pasta de seu projeto, vá em `Novo Arquivo... -> Recurso -> SceneKit Particle System` e use o template `Smoke` ou fumaça. Brinque como quiser com os parametros, segue um print de como deixar o sistema bacaninha:

![]({{ site.baseurl }}/img/farris/img3.png)

Agora adicione este código no final da função `createPlayer`:

```Swift
let particleSystem = SCNParticleSystem(named: "SmokeParticles", inDirectory: nil)
let exausterNode = SCNNode(geometry: SCNBox(width: 0, height: 0, length: 0, chamferRadius: 1))
exausterNode.position = SCNVector3(0,0,1.5)
exausterNode.addParticleSystem(particleSystem!)
car.addChildNode(exausterNode)
```

Vamos interagir com ele? Adicione a seguinte variavel `var onLeftLane:Bool = true`, e adicione este código no seu `viewDidLoad`:

```Swift
let tapGestureRecognizer = UITapGestureRecognizer(target: self, action:"move:")
let swipeGestureRecognizer = UISwipeGestureRecognizer(target: self, action: "move:")
scnView.addGestureRecognizer(tapGestureRecognizer)
scnView.addGestureRecognizer(swipeGestureRecognizer)
```

Em seguida, vamos implementar o `move:`:

```Swift
func move(sender: UITapGestureRecognizer){
    let position = sender.locationInView(self.view) //localizacao do gesto
    let right = position.x > self.view.frame.size.width/2 // foi na esquerda ou direita?
    if right == onLeftLane { // Pra onde vamos
        let moveSideways:SCNAction = SCNAction.moveByX((right ? 8:-8), y: 0, z: 0, duration: 0.2)
        moveSideways.timingMode = SCNActionTimingMode.EaseInEaseOut // suaviza a animacao
        car.runAction(moveSideways)
        onLeftLane = !right // atualiza a posicao do carro
    }
}
```

Rode. O resultado deve ser algo como:

![]({{ site.baseurl }}/img/farris/gif2.gif)

### Capítulo 3: Obstáculos e recompensas!

###### No qual aprendemos a criar inimigos, física e colisões.

Vamos começar definindo quem serão nossas entidades capazes de interagir fisicamente entre si. Insira esse `enum` em seu `ViewController`:

```Swift
enum PhysicsCategory: Int {
    case Player=1, Mob=2, Ground=4, Wall=8
}
```

Veja que os valores são binários, pois estamos simulando máscaras de bits. Em seguida, na função `createGround`, vamos dar um formato e um corpo pro nosso chão:

```Swift
func createGround () {
    let groundGeometry = SCNFloor()
    groundGeometry.reflectivity = 0.5
    let groundMaterial = SCNMaterial()
    groundMaterial.diffuse.contents = UIColor.yellowColor()
    groundGeometry.materials = [groundMaterial]
    ground = SCNNode(geometry: groundGeometry)
    ground.physicsBody = SCNPhysicsBody(type: .Static, shape: SCNPhysicsShape(geometry: groundGeometry, options: nil))
    ground.physicsBody!.categoryBitMask = PhysicsCategory.Ground.rawValue
    ground.physicsBody!.contactTestBitMask = PhysicsCategory.Mob.rawValue
    ground.physicsBody!.collisionBitMask = PhysicsCategory.Mob.rawValue
    scene.rootNode.addChildNode(ground)
}
```

Vamos rever nossos conceitos. `SCNFloor`, que é uma subclasse de `SCNGeometry`, contém uma descrição geométrica (uma equação paramétrica, no caso) que serve para desenhar o objeto na tela. 
`SCNNode` é a classe que nos ajuda a compor nossa cena, estabelecendo uma hierarquia entre os objetos tridimensionais. `SCNPhysicsShape` é a casca do objeto, é o que será usado para que as
colisões sejam testadas, simulando um volume sólido. `SCNPhysicsBody` é o corpo físico, onde podemos atribuir campos gravitacionais, eletromagnéticos, atrito, velocidade, aceleração e outras
propriedades físicas.

No nosso `groundBody` criamos 3 máscaras:

- `categoryBitMask`: nos ajuda a definir a qual categoria o objeto pertence. 
- `contactTestBitMask`: define com quais objetos os testes de contato são feitos (veremos isso mais adiante). 
- `collisionBitMask`: contra quais outras categorias esse objeto colide.

Vamos criar alguns inimigos então? Adicione o método `spawnEnemyMob()`:

```Swift
func spawnEnemyMob() {
    let enemyMaterial = SCNMaterial()
    enemyMaterial.reflective.contents = UIColor.redColor()
    let enemy = SCNNode(geometry: SCNBox(width: 3, height: 3, length: 3, chamferRadius: 0.2))
    enemy.geometry!.materials = [enemyMaterial]
    enemy.physicsBody = SCNPhysicsBody(type: .Dynamic, shape: SCNPhysicsShape(geometry: enemy.geometry!, options: nil))
    enemy.physicsBody!.velocity = SCNVector3Make(0, 0, 30)
    enemy.position = SCNVector3(Int(arc4random_uniform(2)*8)-4,2,-100)
    enemy.physicsBody!.categoryBitMask = PhysicsCategory.Mob.rawValue
    enemy.physicsBody!.contactTestBitMask = PhysicsCategory.Player.rawValue
    enemy.physicsBody!.collisionBitMask = PhysicsCategory.Player.rawValue | PhysicsCategory.Ground.rawValue
    scene.rootNode.addChildNode(enemy)
}
```

Quase nada de novo aqui. `Velocity` é a velocidade inicial que nosso objeto se encontrará quando aparecer na cena. Vamos invocar esses inimigos?
Chame no seu `viewDidLoad()`:

```Swift
spawnEnemyMob()
NSTimer.scheduledTimerWithTimeInterval(7, target: self, selector: "spawnEnemyMob", userInfo: nil, repeats: true)
```

Note que criamos um inimigo, e programamos pra adicionar outro a cada 7 segundos. Rode o código, voce deverá ver algo como:

![]({{ site.baseurl }}/img/farris/gif3.gif)

Notou que o bloco passou atravessando o carro? Precisamos adicionar um corpo ao nosso jogador. Adicione este código na sua função `createPlayer` (antes de `car.addChildNode(exausterNode)`):

```Swift
car.physicsBody = SCNPhysicsBody(type: .Kinematic, shape: SCNPhysicsShape(node: car, options: nil))
```

Rode de novo, veja que existe a colisão. Vamos adicionar um objeto agora para capturar os inimigos e engatilhar a lógica de criação dos próximos. Adicione a variável `var wall:SCNNode!`, a chamada `createWall()` no seu `viewDidLoad`, e crie a função:

```Swift
func createWall () {
    wall = SCNNode(geometry:SCNBox(width: 200, height: 200, length: 3, chamferRadius: 0))
    wall.physicsBody = SCNPhysicsBody(type: .Static, shape: SCNPhysicsShape(geometry: wall.geometry!, options: nil))
    wall.physicsBody!.categoryBitMask = PhysicsCategory.Wall.rawValue
    wall.physicsBody!.contactTestBitMask = PhysicsCategory.Mob.rawValue
    wall.physicsBody!.collisionBitMask = PhysicsCategory.Mob.rawValue
    scene.rootNode.addChildNode(wall)
}
```



Vamos agora detectar as colisões. Adicione a interface `SCNPhysicsContactDelegate` ao seu view controller, assim:

```Swift
class GameViewController: UIViewController, SCNPhysicsContactDelegate
```

Em seguida, vamos criar a função que recebe os avisos de colisões:

```Swift
func physicsWorld(world: SCNPhysicsWorld, didBeginContact contact: SCNPhysicsContact) {
    if (contact.nodeA != ground && contact.nodeB != ground) {
        if (contact.nodeA == car || contact.nodeB == car) {
            let enemyNode = contact.nodeA == car ? contact.nodeB : contact.nodeA
            if (enemyNode.parentNode != nil) {
                enemyNode.removeFromParentNode()
                spawnEnemyMob()
            }
        } else if (contact.nodeA == wall || contact.nodeB == wall) {
            let enemyNode = contact.nodeA == wall ? contact.nodeB : contact.nodeA
            if (enemyNode.parentNode != nil) {
                enemyNode.removeFromParentNode()
                spawnEnemyMob()
            }
        }
    }
}
```

Estamos verificando se a colisão é com nosso carro, ou com o muro que está escondido atrás da câmera. Se for com um deles, removemos o inimigo e criamos outro.
Rode novamente, desvie dos inimigos!

### Epílogo: Pra onde ir agora.

Como desafio, sugiro as seguintes modificações:

- Mostrar o score na tela;
- Adicionar _swag_ no movimento do carrinho;
- Desligar o `autoenablesDefaultLighting` da cena, e adicionar farois ao carrinho;
- Criar um modo POV onde a camera vai parar dentro do carrinho;
- Adicionar mais faixas, mais inimigos, bonus ou até mais um carro (como é o jogo 2 Cars);

Espero que tenha gostado do texto, fique ligado nos demais artigos dessa série. Qualquer dúvida, reclamação, sugestão, o repositório [https://github.com/luksfarris/carRush](https://github.com/luksfarris/carRush) é o melhor lugar para me achar. Abra uma `Issue`, faça um `Pull Request`, brinque com o código, enfim: Divirta-se!

> Lucas Farris desenvolve jogos desde 2006, entrou no mercado mobile em 2011. Atualmente mora na Polonia. 
