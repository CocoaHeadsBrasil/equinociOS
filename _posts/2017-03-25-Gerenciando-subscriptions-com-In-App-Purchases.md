---
layout:     post
title:      "Gerenciando subscriptions com In-App Purchases"
subtitle:	"Porque parece complicado, mas não é."
date:       2017-03-25 09:00:00
author:     "Renan Protector"
header-img: "img/reprotector/subscription-header.jpeg"
category:   "subscription"
---

> Renan Protector ([@reprotector](https://twitter.com/reprotector){:target="_blank"}) é desenvolvedor iOS desde 2009. Atualmente Indie Dev, é Co-Founder do ([Blogo](https://getblogo.com){:target="_blank"}) e do ([Space Coworking](https://spacecoworking.com){:target="_blank"}). 


Em Junho de 2016 a Apple modificou sua política de subscriptions na App Store, permitindo que várias categorias de aplicativos possam utilizar esse modelo de negócios. A assinatura funciona como um In-App Purchase e toda a parte de pagamento é feito pela própria App Store. O foco desse artigo está na gestão das assinaturas, por isso não falaremos como implementar IAP (In-App Purchase) dentro do seu aplicativo. [Clique Aqui](https://developer.apple.com/in-app-purchase/){:target="_blank"} para saber mais sobre IAP.

Iremos falar sobre duas possibilidades: __Assinatura apenas no app__ e __Assinatura em vários ambientes__

## Apenas em 1 App
Quando a assinatura é apenas para um único app iOS, não é preciso fazer muita coisa. O próprio app deve gerenciar a assinatura do usuário. Após completar a compra do IAP, o método `- (void)completeTransaction:(SKPaymentTransaction *)transaction` será chamado e a partir daí as funcionalidades já podem ser liberadas para o usuário.

Para verificar se o usuário é assinante e se a assinatura está em dia, basta verificar o receipt[*](#what_is_receipt) do aplicativo. Recomendo a utilização do [VerifyStoreReceiptiOS](https://github.com/rmaddy/VerifyStoreReceiptiOS){:target="_blank"}, pois facilita muito a leitura dos campos do receipt dentro do aplicativo. 

Para saber como validar um receipt, veja o item: [O que verificar nesse receipt?](#validate_receipt)


## Múltiplos apps e Web

Quando a assinatura vai ser usada em vários ambientes, é necessário que você sincronize os dados da App Store. Você precisará de um serviço web para gerenciar as assinaturas e validá-las. Isso significa que seu aplicativo precisará de login para identificar seus usuários e suas respectivas assinaturas.

### Chega de papo, como faço isso no meu app?

Ao finalizar a compra do IAP, o seguinte método será chamado: `- (void)completeTransaction:(SKPaymentTransaction *)transaction`. 

Nesse momento, o receipt do seu app já estará atualizado com os dados de compra do IAP e você deverá enviá-lo para sua API. Sua API deve validar os dados junto à App Store para se certificar que o receipt é verdadeiro e original. Para saber mais sobre a validação do receipt na App Store, [clique aqui.](https://developer.apple.com/library/content/releasenotes/General/ValidateAppStoreReceipt/Chapters/ValidateRemotely.html#//apple_ref/doc/uid/TP40010573-CH104-SW1){:target="_blank"}

Em Ruby, a validação pode ser feito com o código abaixo, não esqueça de adicionar seu Token no request:

```ruby
	def validate_receipt_remote(receipt_base64)  
	  receipt_base64.gsub! ' ', '+'
	  params_json = "{ \"receipt-data\": \""+receipt_base64+"\", \"password\": \”SEU TOKEN AQUI\”}”
	  uri = URI("https://buy.itunes.apple.com") # Use "https://buy.itunes.apple.com" for production
	  Net::HTTP.start(uri.host, uri.port, :use_ssl => uri.scheme == 'https') do |http|
	    response = http.post('/verifyReceipt', params_json)	    
	    return JSON.parse(response.body)
	  end
	end
```

Um exemplo de resposta dessa requisição (mantive apenas os campos que considero importante):

```
{ 
   "status"   =>0,
   "environment"   =>"Production",
   "receipt"   =>   { 
      "receipt_type"      =>"Production",
      "app_item_id"      =>977160124,
      "bundle_id"      =>"com.Blogo.ios",
      "application_version"      =>"251",
      "receipt_creation_date"      =>"2017-03-12 11:22:41      Etc/GMT",
      "request_date"      =>"2017-03-24 17:22:53      Etc/GMT",
      "original_purchase_date"      =>"2016-10-16 03:54:25      Etc/GMT",
      "original_application_version"      =>"215",
      "in_app"      =>      [          
         { 
            "product_id"            =>"com.blogo.pro.ios.month",
            "transaction_id"            =>"120000262038036",
            "original_transaction_id"            =>"120000262038036",
            "purchase_date"            =>"2016-10-16 05:45:23            Etc/GMT",
            "original_purchase_date"            =>"2016-10-16 05:45:24            Etc/GMT",
            "expires_date"            =>"2016-11-16 06:45:23            Etc/GMT",
            "is_trial_period"            =>"true"
         }
      ]
   },
   "latest_receipt_info"   =>   [ 
      { 
         "product_id"         =>"com.blogo.pro.ios.month",
         "transaction_id"         =>"120000262038036",
         "original_transaction_id"         =>"120000262038036",
         "purchase_date"         =>"2016-10-16 05:45:23         Etc/GMT",
         "original_purchase_date"         =>"2016-10-16 05:45:24         Etc/GMT",
         "expires_date"         =>"2016-11-16 06:45:23         Etc/GMT",
         "is_trial_period"         =>"true"
      }
   ],
   "latest_receipt"   =>"MIIXOwYJKoZIhvcNAQc….”
}
```

### <a name="validate_receipt"></a>O que verificar nesse receipt?

A primeira coisa que deve ser verificada é se os campos `bundle_id` e `app_item_id` conferem com o do seu aplicativo. 

Nos dados do IAP, você deve validar a data de expiração e deve atrelar o `transaction_id` da assinatura com o usuário. O `transaction_id` deve ser um dado único por assinatura, por usuário. Por que isso? Se não for salvo dessa forma, o usuário pode dar logout no aplicativo, logar com outro usuário e assinar o IAP novamente. Como já está pago na App Store, a Apple vai retornar para o método `completeTransaction` como se a transação tivesse sido bem sucedida. Isso é __MUITO IMPORTANTE__ para evitar fraudes no seu app.

__Importante!__ Sempre use os dados do `latest_receipt_info`. O receipt é um arquivo local e os dados do `latest_receipt_info` são os últimos dados encontrados no serviço da Apple.

Depois de validado, você deve guardar o arquivo de receipt na sua nuvem. Ele será usado para verificar se o usuário renovou a assinatura.

### Como sei se o usuário é um assinante?

Seu app deve se comunicar com sua base de dados online, perguntando se ele é um assinante ou não. Lembre-se que o usuário pode ter assinado em outro dispositivo, veja qual é o momento ideal para perguntar pro seu servidor se ele está ativo ou não. Em meus apps iOS, costumo verificar logo na abertura do app ou logo que o dispositivo tenha acesso a internet, caso estivesse sem interent no momento da abertura. Também costumo verificar novamente antes de entrar no processo de assinatura.

### Ótimo, já sei que o usuário assinou meu IAP. E quando acabar o período, como sei que ele renovou?

As assinaturas são renovadas 1 dia antes delas expirarem, mas pode ocorrer problemas no pagamento e o processo de renovação pode levar mais tempo. Minha experiência mostra que assinaturas podem ser renovadas automaticamente até 1 dia depois dela expirar. Portanto, não bloqueie as funcionalidades da assinatura assim que ela expirar. Espere pelo menos 1 dia pra ter certeza que ela não foi renovada.

Para verificar se a assinatura foi renovada ou não, devemos pegar o receipt que foi salvo no momento da assinatura e fazer o mesmo processo de validação. Faça a validação do receipt pelo link da Apple e verifique o `latest_receipt_info`.  Caso a assinatura tenha sido renovada, um novo item de IAP terá sido criado. Basta, então, adicionar o novo transaction_id no seu banco de dados e liberar o uso das funcionalidades no seu aplicativo.

Abaixo um exemplo de um receipt com assinaturas renovadas. Repare que o usuário entrou em trial no dia 16/10/16, renovou automaticamente a assinatura em 16/11/16 e não renovou no período de Janeiro a Março. Em 12/03/17 o usuário se inscreveu novamente em uma assinatura.

```
"latest_receipt_info"   =>   [ 
      { 
         "product_id"         =>"com.blogo.pro.ios.month",
         "transaction_id"         =>"120000262038036",
         "original_transaction_id"         =>"120000262038036",
         "purchase_date"         =>"2016-10-16 05:45:23         Etc/GMT",
         "original_purchase_date"         =>"2016-10-16 05:45:24         Etc/GMT",
         "expires_date"         =>"2016-11-16 06:45:23         Etc/GMT",
         "is_trial_period"         =>"true"
      },
      { 
         "product_id"         =>"com.blogo.pro.ios.month",
         "transaction_id"         =>"120000269815092",
         "original_transaction_id"         =>"120000262038036",
         "purchase_date"         =>"2016-11-16 06:45:23         Etc/GMT",
         "original_purchase_date"         =>"2016-10-16 05:45:24         Etc/GMT",
         "expires_date"         =>"2016-12-16 06:45:23         Etc/GMT",
         "is_trial_period"         =>"false"
      },
      { 
         "product_id"         =>"com.blogo.pro.ios.month",
         "transaction_id"         =>"120000303300763",
         "original_transaction_id"         =>"120000262038036",
         "purchase_date"         =>"2017-03-12 11:22:40         Etc/GMT",
         "original_purchase_date"         =>"2016-10-16 05:45:24         Etc/GMT",
         "expires_date"         =>"2017-04-12 11:22:40         Etc/GMT",
         "is_trial_period"         =>"false"
      }
   ]
```

## Conclusão

A gestão de subscriptions com a App Store não é tão complicada quanto parece ser. Quando a assinatura é pra apenas um app, não é preciso fazer muitas coisa e o processo de validação é bem simples. Já para múltiplos ambientes, a documentação da Apple não é muito clara sobre como verificar se um usuário renovou a assinatura ou não. Esse artigo tenta ajudar a desvendar o mistério do `latest_receipt_info` que não é muito claro na documentação da Apple. Dúvidas e sugestões podem ser colocadas no Slack iOSDevBr, não deixe de entrar lá!

#### Notas de Rodapé:

<a name="what_is_receipt"></a>
__Receipt__: Todo aplicativo, seja ele pago ou gratuito, possui um “recibo de compra” embutido. Esse arquivo contém dados importantes que devem ser usados para evitar fraudes. [Clique aqui](https://developer.apple.com/library/content/releasenotes/General/ValidateAppStoreReceipt/Chapters/ReceiptFields.html#//apple_ref/doc/uid/TP40010573-CH106-SW1){:target="_blank"} para saber mais sobre os campos de um receipt.
