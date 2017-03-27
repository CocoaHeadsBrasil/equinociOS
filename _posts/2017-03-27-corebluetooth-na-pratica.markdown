---
layout:     post
title:      "CoreBluetooth na Prática"
subtitle:   "Entendendo o CoreBluetooth framework e aplicando a exemplo real."
date:       2017-03-27 00:00:00
author:     "Leonardo Cardoso"
header-img: "img/leonardocardoso/main.jpg"
category:   bluetooth
---

> Leonardo Cardoso ([@leocardz](https://twitter.com/leocardz){:target="_blank"}) começou a trabalhar com mobile desde 2010 com a plataforma do robozinho verde, mas veio para a maçã em 2014, atualmente lidera o time iOS da [1aim](https://1aim.com){:target="_blank"}, em Berlim. É também um entusiasta open source e desenvolveu alguns projetos que podem ser acessados no seu perfil do [GitHub](https://github.com/leonardocardoso){:target="_blank"}.

# Intro

É muito comum hoje estarmos em contato com dispositivos sem fio no nosso dia-a-dia. Seja um *headphone*, um *mouse*, um teclado, ou o famigerado *Apple Watch*. A lista é extensa... Todos eles, porém, utilizam basicamente uma mesma tecnologia que os torna de fato *wireless*: **Bluetooth®**.

Aqui vamos aprender brevemente os papeis executados e propriedades difundidas numa conexão via *Bluetooth*, e os modos de execução desses papeis utilizando o *framework* **Core Bluetooth**, que nos dá o suporte à versão 4.0 (*a.k.a smart, a.k.a. low-energy*).

Primeiramente vamos aos modos de execução.

# Modos de Execução

O **Core Bluetooth**, ou CB, nos proporciona dois modos de execução, *Foreground* e *Background*. Assim, podemos definir bem se queremos nossos *apps* "conectáveis" apenas quando o usuário estiver interagindo com ele ou não. Alguns aspectos definem bem esses modos, são eles:

## Foreground

Se o app é minimizado ou outro app se torna o principal utilizado pelo usuário, nosso suporte *Bluetooth* vai junto. Assim, no estado suspenso, seu *app* não é capaz de realizar tarefas relacionadas a *Bluetooth*, nem fica ciente de eventos relacionados a *Bluetooth*. Porém a lista de eventos recebidos enquanto o *app* estava em *background* é empilhada e esses eventos são entregues quando o usuário resumir o *app*, tornando-o ativo novamente.

Explicaremos mais abaixo a diferença entre os papeis e as propriedades, mas nesse modo caso o aplicativo não esteja ativo, o periférico não consegue enviar propagandas (*advertising*) e a central não consegue difundir seu sinal (*scanning*), porém a execução de *advertising* e *scanning* são respondidas sob demanda, praticamente sem atrasos.

A vantagem de usarmos em *Foreground* é que temos o todos os recursos do sistema ao nosso alcance, como por exemplo, atualização de *views* e tarefas sem tempo limite de execução. Ainda nesse modo, nossa *stack* de dispositivos é bem distrinchada, inclusive tendo a opção de escolher se queremos escanear o mesmo periférico por alguma razão específica. Isso é feito através de uma propriedade `CBCentralManagerScanOptionAllowDuplicatesKey`, o que no modo *Background* é ignorada, assim como outras propriedades são tratadas diferentes quando executadas em modos *Foreground* e *Background* e elas podem ser achadas nas [Referências](#referncias).

## Background

Aqui já começamos dizendo que os recursos de sistemas são limitados. Temos também _10 segundos_ no máximo para executar uma tarefa nesse modo. Mas creio que isso é mais do que suficiente. O intervalo de execução de *advertising* e *scanning* pode ser mais demorado. Isso é feito em prol de economizar bateria do seu dispositivo. Aqui, eventos do mesmo dispositivo são combinados num único evento com seus _serviços e características_, ou seja, a opção de escanear o mesmo dispositivo mais de uma vez é descartada.

A vantagem desse modo é permitir que nosso *app* não precisa estar ativo para trocarmos dados entre centrais e periféricos. Então, só para esclarecer, o sistema acorda nosso *app* de um estado suspenso, permitindo ler, escrever, inscrever em características, escutar eventos. Tudo em *background*. Mas mesmo que tenhamos permissão de rodar em *background*, nosso *app* não vai rodar sempre. Isso pode acontecer porque eventualmente o sistema pode finalizar nosso *app* para liberar memória para o *app* que está no estado ativo. A partir do *iOS7*, é permitido salvar os estados dos dispositivos, conexões pendentes, etc., antes de uma interrupção e então restaurá-los depois.

Como um exemplo para essa restauração de estados, vamos imaginar que você tenha desenvolvido um dispositivo de segurança que se comunica com uma fechadura eletrônica equipada com *Bluetooth*. O *app* e a fechadura podem interagir para que a fechadura se tranque automaticamente quando o usuário saia da casa e destranque quando o usuário voltar. Tudo isso com o *iPhone* no bolso. Quando o usuário sair de casa, o *iPhone* pode eventualmente sair do alcance da fechadura e, então, perder conexão. Nesse caso, o usuário pode simplesmente chamar a função para conectar o dispositivo, mesmo não estando no raio de alcance e, assim, a conexão será refeita quando eles estiverem de novo ao alcance. Isso acontece porque a requisição de conexão de dispositivos não expira.

Agora imagina que o usuário está fora de casa em uma viagem. Se o *app* é finalizado pelo sistema enquanto o usuário está fora, o *app* não vai ser capaz de se reconectar com a fechadura quando o usuário retornar. Assim, a fechadura não vai destrancar. Para apps como estes, é definitivamente necessário usar a opção de salvamento e restauração de estados.



Observação: *macOS* e *iOS* funcionam um pouco diferente nos papeis de periférico e central. Por exemplo, *macOS* não aceita a *flag* `CBCentralManagerOptionRestoreIdentifierKey`, porque o *background* do *macOS* não finaliza *apps* como o *iOS* faz, em teoria.

Para usar o modo *Background*, precisamos ainda definir *flags* no `Info.plist`.

- `bluetooth-peripheral`: Se quisermos usar um periférico em *background* 
- `bluetooth-central`: Se quisermos usar uma central em *background* 

Se adicionarmos o suporte via `Target > Capabilities > Background Modes`, selecionando `Uses Bluetooth LE accessories (central)` e/ou `Act as Bluetooth LE accessory (peripheral)`, elas são adicionadas automaticamente no `Info.plist`.

**Importante**: Não esqueça de adicionar a *flag* `Privacy - Bluetooth Peripheral Usage Description` para que o usuário permita que esteja utilizando o dispositivo como periférico, assim como fazemos quando queremos utilizar a câmera ou localização.

# Papeis

Nesse jogo de conexões *Bluetooth*, nós temos dois papeis distintos e bem especificados que funcionam como uma abordagem **Client-Server**. São eles: **Periférico** e **Central**.

## Periférico

Um periférico é o dispositivo que compartilha seus dados para a central. Nesse modo, o sistema acorda nosso app para processar tarefas de leitura, escrita, inscrições de eventos vindos da central. É o dispositivo que funciona como *Server*, ou seja, tem os dados que se desejam.

Quando um periférico está sendo executado em *Background*, seus serviços são realocados para uma área de "*overflow*" especial, ou seja, todos os *UUIDs* dos serviços contidos na valor da propriedade `CBAdvertisementDataServiceUUIDsKey`. Sendo assim, ele só pode ser descoberto por um dispositivo que esteja explicitamente escaneando por ele, procurando por alguma de sua característica, basicamente seu indentificador.

## Central

Uma central é o dispositivo que requer informações de dispositivos periféricos. Nesse caso, ela escaneia, connecta, obtém dados e envia dados, os explora. O *Client*.

O sistema acorda nossa central quando eventos de mudança de estado ocorrem com o periférico, tais como conexão estabelecida ou cortada com o periférico, quando periférico atualiza informações de suas caractéristicas, ou ainda quando nossa central está perto de ser finalizada e também restaurada. 

### Exemplo

Imagina o funcionamento de um medidor de batimento cardíaco de praticantes de esportes. Nesse caso, o periférico seria o dispositivo medidor que está alocado abaixo do peito do corredor. E um *app* no *iPhone* poderia ler esse valor e mostrar para o usuário em tempo real, fazendo o *iPhone* funcionar como central, numa ação em *Foreground*. Agora imagina que o corredor não vai querer ficar olhando para seu *iPhone* todo o tempo, então, no modo *Background*, o *iPhone* captura dados dos batimentos e guarda num *log* no qual o usuário pode checar as variações quando terminar seu exercício. 

<img src="{{ site.baseurl }}/img/leonardocardoso/communication.png">

# Conexão

A conexão entre uma central e um periférico é feita através de escaneamento e propaganda. Basicamente esse é o fluxo:

- Um periférico difunde um sinal que pode ser conectado usando pacotes de propaganda;
- Enquanto a central difunde um sinal que está procurando por periférico;
- Quando uma central encontra um periférico, ele pode explorar primeiro requisitar a conexão, o que pode ser rejeitada pelo periférico ou não. Essa conexão pode ser encriptada com a encriptação nativa que o *Core Bluetooth* nos provê. Se a conexão encriptada for desejada, então um código aparece num dos dispositivos para ser digitado no outro e assim criar pares de criptografia administrados pelo próprio sistema, tornando-os dispositivos confiáveis. Se nenhuma criptografia for requerida, então a conexão é feita automática;
- Após a conexão ser feita, a central pode então ordenar que o periférico descubra serviços, basicamente a central está explorando os serviços do periférico;
- Após serviços descobertos, a central pode então ordenar que o periférico descubra características, basicamente a central está explorando as características de cada serviço do periférico;
- Descobertas as características, a central pode então ler valores das características estaticamente ou se increver naquela característica e caso o periférico atualize o valor dela, a central será notificada com o novo valor.
- A conexão pode ser finalizada, se for o caso.

<img src="{{ site.baseurl }}/img/leonardocardoso/advertisingdiscovery.png">


# Serviços e Características

As trocas de dados feitas por dispositivos conectados são através de propriedades e elas são serviços e características já comentadas em algumas partes anteriormente.

Um serviço é uma coleção de dados e comportamentos associados para completar uma tarefa ou uma função de um disposito, ou partes de um dispositivo. Esses comportamentos são chamados de características. Uma característica provê mais detalhes sobre um serviço de um periférico. Parece basicamente uma descrição daqueles verbetes que você procura em um dicionário. Uma definição te leva pra outra e você fica num eterno *loop*. Mas vamos tentar explicar melhor mais abaixo.

Serviços podem ter outros serviços relacionados, como dependências, apontando para *UUIDs* de outros serviços. Cada característica também tem *UUID* que a possa indentificar.

<img src="{{ site.baseurl }}/img/leonardocardoso/servicescharacterisctics.png">

Então nesse exemplo nós temos dois sensores que funcionam diferentes um do outro mas juntos eles produzem um serviço, que é o Batimento Cardíaco. Para que ele funcione corretamente, o sensor cardíaco deve estar posicionado no local ideal.

Ainda utilizando o nosso exemplo anterior, suponha que no dispositivo de batimento cardíaco poderíamos ter dois serviços, um com duas caracterícticas e outro com apenas uma:

- Serviço 1: Batimentos Cardíacos
    - Característica 1: Medição dos Batimentos Cardíacos
    - Característica 2: Localização Adequada do Dispositivo
- Serviço 2: Status
    - Característica 1: Nível de Bateria

Mas um periférico é nada se não estiver enviando propagandas. Um pacote de propaganda é relativamente um pequeno agrupamento de dados que tem informações sobre o que o dispositivo periférico para um reconhecimento inicial. Dados como nome do dispositivo, *UUID* e *RSSI*, que é uma informação de quão forte é o sinal do periférico.

# Pra onde vamos nós?

Nós vamos exemplificar aqui o seguinte:

* Papeis
    * *iOS* como Periférico
    * *macOS* como Central
* Modos    
    * *Foreground*

Para conferir o modo *background*, confira no [repositório desse projeto no GitHub](https://github.com/LeonardoCardoso/BLE{:target="_blank"}), no *branch* `background`.     

# Code Snippets

Devemos primeiro ligar nosso app com o `CoreBluetooth`. Então, vá `Project` -> `Targets` -> `Build Phases` -> `Link Binary with Libraries` -> Procura por `CoreBluetooth.framework` e então o adiciona.

## Foreground

Primeiro, vamos criar um protocolor para receber os eventos escutados do `BluetoothManager` e atualizar as views dos nossos `ViewController`s. Vamos chamá-lo de `BlueEar`. E tem uma versão para a `Central` e outra pro `Peripheral`. Assim como o `BlueEar`, teremos uma classe que será a administradora da nossa conexão *Bluetooth* e é a `BluetoothManager`.

### iOS

#### BlueEar

~~~swift

protocol BlueEar {

    func didStartConfiguration()

    func didStartAdvertising()

    func didSendData()
    func didReceiveData()

}

~~~

#### BluetoothManager

~~~swift

class BluetoothManager: NSObject {

    // MARK: - Properties
    let peripheralId: String = "62443cc7-15bc-4136-bf5d-0ad80c459215"
    let serviceUUID: String = "0cdbe648-eed0-11e6-bc64-92361f002671"
    let characteristicUUID: String = "199ab74c-eed0-11E6-BC64-92361F002672"
    let localName: String = "Peripheral - iOS"

    let properties: CBCharacteristicProperties = [.read, .notify, .writeWithoutResponse, .write]
    let permissions: CBAttributePermissions = [.readable, .writeable]

    var bluetoothMessaging: BlueEar?
    var peripheralManager: CBPeripheralManager?

    var serviceCBUUID: CBUUID?
    var characteristicCBUUID: CBUUID?

    var service: CBMutableService?

    var characterisctic: CBMutableCharacteristic?

    // MARK: - Initializers
    convenience init (delegate: BlueEar?) {

        self.init()

        self.bluetoothMessaging = delegate

        guard
            let serviceUUID: UUID = NSUUID(uuidString: self.serviceUUID) as UUID?,
            let characteristicUUID: UUID = NSUUID(uuidString: self.characteristicUUID) as UUID?
            else { return }

        self.serviceCBUUID = CBUUID(nsuuid: serviceUUID)
        self.characteristicCBUUID = CBUUID(nsuuid: characteristicUUID)

        guard
            let serviceCBUUID: CBUUID = self.serviceCBUUID,
            let characteristicCBUUID: CBUUID = self.characteristicCBUUID
            else { return }

        // Configuring service
        self.service = CBMutableService(type: serviceCBUUID, primary: true)

        // Configuring characteristic
        self.characterisctic = CBMutableCharacteristic(type: characteristicCBUUID, properties: self.properties, value: nil, permissions: self.permissions)

        guard let characterisctic: CBCharacteristic = self.characterisctic else { return }

        // Add characterisct to service
        self.service?.characteristics = [characterisctic]

        self.bluetoothMessaging?.didStartConfiguration()

        // Initiate peripheral and start advertising
        self.peripheralManager = CBPeripheralManager(delegate: self, queue: nil, options: nil)

    }

}

~~~

#### CBPeripheralManagerDelegate

~~~swift

// MARK: - CBPeripheralManagerDelegate
extension BluetoothManager: CBPeripheralManagerDelegate {

    func peripheralManagerDidUpdateState(_ peripheral: CBPeripheralManager) {

        print("peripheralManagerDidUpdateState")

        if peripheral.state == .poweredOn {

            guard let service: CBMutableService = self.service else { return }

            self.peripheralManager?.removeAllServices()
            self.peripheralManager?.add(service)

        }

    }

    func peripheralManager(_ peripheral: CBPeripheralManager, didAdd service: CBService, error: Error?) {

        print("\ndidAdd service")

        let advertisingData: [String: Any] = [
            CBAdvertisementDataServiceUUIDsKey: [self.service?.uuid],
            CBAdvertisementDataLocalNameKey: "Peripheral - iOS"
        ]
        self.peripheralManager?.stopAdvertising()
        self.peripheralManager?.startAdvertising(advertisingData)

    }

    func peripheralManagerDidStartAdvertising(_ peripheral: CBPeripheralManager, error: Error?) {

        print("peripheralManagerDidStartAdvertising")
        self.bluetoothMessaging?.didStartAdvertising()

    }

    // Listen to dynamic values
    // Called when CBPeripheral .setNotifyValue(true, for: characteristic) is called from the central
    func peripheralManager(_ peripheral: CBPeripheralManager, central: CBCentral, didSubscribeTo characteristic: CBCharacteristic) {

        print("\ndidSubscribeTo characteristic")

        guard let characterisctic: CBMutableCharacteristic = self.characterisctic else { return }

        do {

            // Writing data to characteristics
            let dict: [String: String] = ["Hello": "Darkness"]
            let data: Data = try PropertyListSerialization.data(fromPropertyList: dict, format: .binary, options: 0)

            self.peripheralManager?.updateValue(data, for: characterisctic, onSubscribedCentrals: [central])
            self.bluetoothMessaging?.didSendData()

        } catch let error {

            print(error)

        }

    }

    // Read static values
    // Called when CBPeripheral .readValue(for: characteristic) is called from the central
    func peripheralManager(_ peripheral: CBPeripheralManager, didReceiveRead request: CBATTRequest) {

        print("\ndidReceiveRead request")

        if let uuid: CBUUID = self.characterisctic?.uuid, request.characteristic.uuid == uuid {

            print("Match characteristic for static reading")

        }

    }

    // Called when receiving writing from Central.
    func peripheralManager(_ peripheral: CBPeripheralManager, didReceiveWrite requests: [CBATTRequest]) {

        print("\ndidReceiveWrite requests")

        guard
            let characteristicCBUUID: CBUUID = self.characteristicCBUUID,
            let request: CBATTRequest = requests.filter({ $0.characteristic.uuid == characteristicCBUUID }).first,
            let value: Data = request.value
            else { return }

        // Send response to central if this writing request asks for response [.withResponse]
        print("Sending response: Success")
        self.peripheralManager?.respond(to: request, withResult: .success)

        print("Match characteristic for writing")

        do {

            if let receivedData: [String : String] = try PropertyListSerialization.propertyList(from: value, options: [], format: nil) as? [String: String] {

                print("Written value is: \(receivedData)")
                self.bluetoothMessaging?.didReceiveData()

            } else {

                return

            }

        } catch let error {

            print(error)

        }

    }
    
    func peripheralManager(_ peripheral: CBPeripheralManager, central: CBCentral, didUnsubscribeFrom characteristic: CBCharacteristic) {
        
        print("\ndidUnsubscribeFrom characteristic")
        
        
    }
    
    func peripheralManager(_ peripheral: CBPeripheralManager, willRestoreState dict: [String : Any]) {
        
        print("willRestoreState")
        
    }
    
    func peripheralManagerIsReady(toUpdateSubscribers peripheral: CBPeripheralManager) {
        
        print("peripheralManagerIsReady")
        
    }
    
}

~~~

#### ViewController

~~~swift

import UIKit

class ViewController: UIViewController {

    // MARK: - IBOutlet
    @IBOutlet var label: UILabel!

    // MARK: - Lifecyle
    override var preferredStatusBarStyle: UIStatusBarStyle { return .lightContent }

    // MARK: - Properties
    var manager: BluetoothManager?

    override func viewDidAppear(_ animated: Bool) {

        super.viewDidAppear(animated)

        self.manager = BluetoothManager(delegate: self)

    }

}

// MARK: - BlueEar
extension ViewController: BlueEar {

    func didStartConfiguration() { self.label.text = "Start configuration 🎛" }

    func didStartAdvertising() { self.label.text = "Start advertising 📻" }

    func didSendData() { self.label.text = "Did send data ⬆️" }

    func didReceiveData() { self.label.text = "Did received data ⬇️" }
    
}

~~~

#### View

<img src="{{ site.baseurl }}/img/leonardocardoso/iOS.png">


#### Notas:

- No nosso exemplo, o Periférico vai começar a fazer propaganda quando iniciar o `app`.
- Você só pode fazer alguma coisa com o periférico quando a função `peripheralManagerDidUpdateState(: CBPeripheralManager)` for chamada e o estado do periférico estiver `.poweredOn`.
- Leituras de valores dinâmicos pela Central usando a função `.setNotifyValue(true, for: characteristic)` disparam a função `peripheralManager(_: CBPeripheralManager, : CBCentral, : CBCharacteristic)` do periférico, já leituras de valores estáticos usando a função `.readValue(for: characteristic)` pela Central, disparam `peripheralManager(_: CBPeripheralManager, :CBATTRequest)` do periférico.

### macOS

#### BlueEar

~~~swift

protocol BlueEar {

    func didStartConfiguration()

    func didStartScanningPeripherals()

    func didConnectPeripheral(name: String?)
    func didDisconnectPeripheral(name: String?)

    func didSendData()
    func didReceiveData()

    func didFailConnection()

}

~~~

#### BluetoothManager

~~~swift

class BluetoothManager: NSObject {

    // MARK: - Properties
    let serviceUUID: String = "0cdbe648-eed0-11e6-bc64-92361f002671"
    let characteristicUUID: String = "199ab74c-eed0-11e6-bc64-92361f002672"

    var serviceCBUUID: CBUUID?
    var characteristicCBUUID: CBUUID?

    var blueEar: BlueEar?

    var centralManager: CBCentralManager?

    var discoveredPeripheral: CBPeripheral?

    // MARK: - Initializers
    convenience init (delegate: BlueEar) {

        self.init()

        self.blueEar = delegate

        guard
            let serviceUUID: UUID = NSUUID(uuidString: self.serviceUUID) as UUID?,
            let characteristicUUID: UUID = NSUUID(uuidString: self.characteristicUUID) as UUID?
            else { return }

        self.serviceCBUUID = CBUUID(nsuuid: serviceUUID)
        self.characteristicCBUUID = CBUUID(nsuuid: characteristicUUID)

    }

    // MARK: - Functions
    func scan() {

        self.centralManager = CBCentralManager(delegate: self, queue: nil, options: nil)
        self.blueEar?.didStartConfiguration()

    }

}

~~~

#### CBCentralManagerDelegate

~~~swift

// MARK: - CBCentralManagerDelegate
extension BluetoothManager: CBCentralManagerDelegate {

    func centralManagerDidUpdateState(_ central: CBCentralManager) {

        print("\ncentralManagerDidUpdateState \(Date())")

        if central.state == .poweredOn {

            guard let serviceCBUUID: CBUUID = self.serviceCBUUID else { return }

            self.blueEar?.didStartScanningPeripherals()
            self.centralManager?.scanForPeripherals(withServices: [serviceCBUUID], options: nil)

        }

    }

    func centralManager(_ central: CBCentralManager, didDiscover peripheral: CBPeripheral, advertisementData: [String : Any], rssi RSSI: NSNumber) {

        // We must keep a reference to the new discovered peripheral, which means we must retain it.
        self.discoveredPeripheral = peripheral

        print("\ndidDiscover:", self.discoveredPeripheral?.name ?? "")

        self.discoveredPeripheral?.delegate = self

        guard let discoveredPeripheral: CBPeripheral = self.discoveredPeripheral else { return }
        self.centralManager?.connect(discoveredPeripheral, options: nil)
        
    }

    func centralManager(_ central: CBCentralManager, didConnect peripheral: CBPeripheral) {

        print("\ndidConnect", self.discoveredPeripheral?.name ?? "")

        self.blueEar?.didConnectPeripheral(name: peripheral.name ?? "")

        guard let serviceCBUUID: CBUUID = self.serviceCBUUID else { return }

        self.discoveredPeripheral?.discoverServices([serviceCBUUID])

    }

    func centralManager(_ central: CBCentralManager, willRestoreState dict: [String : Any]) {

        print("willRestoreState")

    }

    func centralManager(_ central: CBCentralManager, didRetrievePeripherals peripherals: [CBPeripheral]) {

        print("\ndidRetrievePeripherals")


    }

    func centralManager(_ central: CBCentralManager, didFailToConnect peripheral: CBPeripheral, error: Error?) {

        print("\ndidFailToConnect")

        self.blueEar?.didFailConnection()

    }

    func centralManager(_ central: CBCentralManager, didRetrieveConnectedPeripherals peripherals: [CBPeripheral]) {

        print("\ndidRetrieveConnectedPeripherals")


    }

    func centralManager(_ central: CBCentralManager, didDisconnectPeripheral peripheral: CBPeripheral, error: Error?) {

        print("\ndidDisconnectPeripheral", self.discoveredPeripheral?.name ?? "")
        self.blueEar?.didDisconnectPeripheral(name: peripheral.name ?? "")

    }

}

~~~

#### CBPeripheralDelegate

~~~swift

// MARK: - CBPeripheralDelegate
extension BluetoothManager: CBPeripheralDelegate {

    func peripheral(_ peripheral: CBPeripheral, didDiscoverServices error: Error?) {

        print("\ndidDiscoverServices")

        if let service: CBService = self.discoveredPeripheral?.services?.filter({ $0.uuid == self.serviceCBUUID }).first {

            guard let characteristicCBUUID: CBUUID = self.characteristicCBUUID else { return }

            self.discoveredPeripheral?.discoverCharacteristics([characteristicCBUUID], for: service)

        }
        
    }

    func peripheral(_ peripheral: CBPeripheral, didWriteValueFor characteristic: CBCharacteristic, error: Error?) {

        print("\ndidWriteValueFor \(Date())")

        // After we write data on peripheral, we disconnect it.
        self.centralManager?.cancelPeripheralConnection(peripheral)

        // We stop scanning.
        self.centralManager?.stopScan()

    }

    func peripheral(_ peripheral: CBPeripheral, didDiscoverCharacteristicsFor service: CBService, error: Error?) {

        print("\ndidDiscoverCharacteristicsFor")

        if let characteristic: CBCharacteristic = service.characteristics?.filter({ $0.uuid == self.characteristicCBUUID }).first {

            print("Matching characteristic")

            // To listen and read dynamic values
            self.discoveredPeripheral?.setNotifyValue(true, for: characteristic)

            // To read static values
            // self.discoveredPeripheral?.readValue(for: characteristic)
            
        }
        
        
    }

    func peripheral(_ peripheral: CBPeripheral, didUpdateValueFor characteristic: CBCharacteristic, error: Error?) {

        print("\ndidUpdateValueFor")

        // We read
        if let value: Data = characteristic.value {

            do {

                let receivedData: [String: String] = try PropertyListSerialization.propertyList(from: value, options: [], format: nil) as! [String: String]

                print("Value read is: \(receivedData)")
                self.blueEar?.didReceiveData()

            } catch let error {

                print(error)

            }

        }

        // We write
        do {

            print("\nWriting on peripheral.")

            let dict: [String: String] = ["Yo": "Lo"]
            let data: Data = try PropertyListSerialization.data(fromPropertyList: dict, format: .binary, options: 0)

            self.discoveredPeripheral?.writeValue(data, for: characteristic, type: .withResponse)
            self.blueEar?.didSendData()
            
        } catch let error {
            
            print(error)
            
        }
        
    }

    func peripheralDidUpdateRSSI(_ peripheral: CBPeripheral, error: Error?) {

        print("\nperipheralDidUpdateRSSI")
        print(self.discoveredPeripheral?.rssi ?? "")
        
    }

    func peripheralDidUpdateName(_ peripheral: CBPeripheral) {

        print("\nperipheralDidUpdateName")

    }

    func peripheral(_ peripheral: CBPeripheral, didWriteValueFor descriptor: CBDescriptor, error: Error?) {

        print("\ndidWriteValueFor")

    }

    func peripheral(_ peripheral: CBPeripheral, didModifyServices invalidatedServices: [CBService]) {

        print("\ndidModifyServices")

    }

    func peripheral(_ peripheral: CBPeripheral, didUpdateValueFor descriptor: CBDescriptor, error: Error?) {

        print("\ndidUpdateValueFor")

    }

    func peripheral(_ peripheral: CBPeripheral, didDiscoverIncludedServicesFor service: CBService, error: Error?) {

        print("\ndidDiscoverIncludedServicesFor")

    }
    
    func peripheral(_ peripheral: CBPeripheral, didDiscoverDescriptorsFor characteristic: CBCharacteristic, error: Error?) {
        
        print("\ndidDiscoverDescriptorsFor")
        
    }
    
    func peripheral(_ peripheral: CBPeripheral, didUpdateNotificationStateFor characteristic: CBCharacteristic, error: Error?) {
        
        print("\ndidUpdateNotificationStateFor")
        
    }
    
}

~~~

#### ViewController

~~~swift

import Foundation
import Cocoa

class ViewController: NSViewController {

    // MARK: - IBOutlet
    @IBOutlet weak var label: NSTextField!
    @IBOutlet weak var button: NSButton!

    // MARK: - Properties
    var manager: BluetoothManager?

    // MARK: - Lifecyle
    @IBAction func discover(_ sender: Any) {

        self.manager = BluetoothManager(delegate: self)
        self.manager?.scan()

    }

}

// MARK: - BlueEar
extension ViewController: BlueEar {

    func didStartConfiguration() { self.label.stringValue = "Start configuration 🎛" }

    func didStartScanningPeripherals() { self.label.stringValue = "Start scanning peripherals 👀" }

    func didConnectPeripheral(name: String?) { self.label.stringValue = "Did connect to: \(name ?? "") 🤜🏽🤛🏽" }

    func didDisconnectPeripheral(name: String?) { self.label.stringValue = "Did disconnect: \(name ?? "") 🤜🏽🤚🏽" }

    func didSendData() { self.label.stringValue = "Did send data ⬆️" }

    func didReceiveData() { self.label.stringValue = "Did received data ⬇️" }

    func didFailConnection() { self.label.stringValue = "Connection failed 🤷🏽‍♂️" }
    
}

~~~

#### View

<img src="{{ site.baseurl }}/img/leonardocardoso/macOS.png">

#### Notas:

- No nosso exemplo, a Central vai começar a escanear por periféricos quando clicarmor num botão chamado `Tap`.
- Você só pode fazer alguma coisa com a central quando a função `centralManagerDidUpdateState(: CBCentralManager)` for chamada e o estado do periférico estiver `.poweredOn`.
- Note que o `CBPeripheralDelegate` é diferente do `CBPeripheralManagerDelegate` usado no periférico. Ambos são relacionados ao periférico, porém o `CBPeripheralDelegate` permite que a central escute eventos sobre o periférico.
- Ao optar por ler resultados dinâmicos, a central se inscreve em características do periférico, assim sendo alertada por meio de notificações se o periférico alterar seu valor no qual ela esteja inscrita.
- Quando a central descobre um periférico, devemos manter uma referência para ele, ou seja, devemos retê-lo fazendo com que uma variável dentro do nosso código o receba.
- Os dados são trasmitidos em formato de `Data`, então você deve transformar o que quer transmitir neste tipo de dados para poder ser recebido corretamente no outro lado.

### Resultado

Na imagem, temos o respectivo: *Log* do periférico, *iOS*, *macOS*, *Log* da central.

<a href="{{ site.baseurl }}/img/leonardocardoso/result.mp4" alt="Click to see a video smoother than this GIF" target="_blank">
<img src="{{ site.baseurl }}/img/leonardocardoso/result.gif">
</a>

---

### Referências

- [CoreBlueetooth](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/AboutCoreBluetooth/Introduction.html){:target="_blank"}

---

### Agradecimentos

Um agradecimento pra galera do [Slack iOS Devs BR](http://iosdevbr.slack.com){:target="_blank"} pela oportunidade.

Um abraço, e até próxima.

Leo
