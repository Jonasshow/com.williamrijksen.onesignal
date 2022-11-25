# Titanium OneSignal [![Build Status](https://travis-ci.org/williamrijksen/com.williamrijksen.onesignal.svg?branch=master)](https://travis-ci.org/williamrijksen/com.williamrijksen .onesignal)

Este módulo oferece a possibilidade de integrar o OneSignal em seu aplicativo Appcelerator para Android ou iOS. É até possível segmentar pessoas registrando tags.

### Onde conseguir:
Verifique https://github.com/williamrijksen/com.williamrijksen.onesignal/releases para baixar as versões mais recentes do módulo.

## Gerar credenciais

Antes de configurar o Titanium SDK, você deve gerar as credenciais apropriadas para a(s) plataforma(s) em que está lançando:

- iOS - [Gerar um iOS Push Certificate](https://documentation.onesignal.com/docs/generate-an-ios-push-certificate)
- ANDROID - [Gerar uma chave de API do servidor do Google](https://documentation.onesignal.com/docs/generate-a-google-server-api-key)

## Siga o Guia

### Configurar

1. Integre o módulo na pasta `modules` e defina-os no arquivo `tiapp.xml`:

    ```xml
    <modules>
      <module platform="iphone" version="2.1.1">com.williamrijksen.onesignal</module>
      <module platform="android" version="2.1.1">com.williamrijksen.onesignal</module>
    </modules>
    ```
1. Configure seu aplicativo no painel Configurações do aplicativo para a plataforma certa (Android e/ou iOS).
1. Para usar o OneSignal em dispositivos iOS, registre o OneSignal-appId em `tiapp.xml`:

    ```xml
    <property name="OneSignal_AppID" type="string">[App-id]</property>
    ```
1. Para usar o OneSignal em dispositivos Android, registre também alguns metadados:

    ```xml
    <meta-data android:name="onesignal_app_id"
                   android:value="[App-id]" />
    ```
1. Para usar notificações avançadas no iOS 10, você precisa adicionar uma extensão ao seu aplicativo.
    Para isso consulte:
   - [https://documentation.onesignal.com/docs/ios-sdk-setup#section-1-add-notification-service-extension](https://documentation.onesignal.com/docs/ios-sdk-setup#section-1-add-notification-service-extension)
   - [http://docs.appcelerator.com/platform/latest/#!/guide/Creating_iOS_Extensions_-_Siri_Intents](http://docs.appcelerator.com/platform/latest/#!/guide/Creating_iOS_Extensions_-_Siri_Intents)

### Uso
1. Registre o dispositivo para notificações push

   ```js
       // This registers your device automatically into OneSignal
       var onesignal = require('com.williamrijksen.onesignal');
   ```
1. No iOS, você precisará solicitar permissão para usar as notificações:
   ```js
       onesignal.promptForPushNotificationsWithUserResponse(function(obj) {
           alert(JSON.stringify(obj));
       });
   ```
1. Para adicionar a possibilidade de segmentar pessoas para notificações, envie uma tag:

   ```js
       onesignal.sendTag({ key: 'foo', value: 'bar' });
   ```
1. Excluir marca:

   ```js
       onesignal.deleteTag({ key: 'foo' });
   ```
1. Obter etiquetas:

    ```js
        onesignal.getTags(function(e) {
            if (!e.success) {
                Ti.API.error("Error: " + e.error);
                return
            }

            Ti.API.info(Ti.Platform.osname === "iphone"? e.results : JSON.parse(e.results));
        });
    ```
1. Definir ID de usuário externo:

    ```js
        onesignal.setExternalUserId('your_db_user_id');
    ```
1. Remover ID de usuário externo:

    ```js
        onesignal.removeExternalUserId();
    ```
1. Obter se o usuário estiver inscrito (Booleano):

    ```js
        var subscribed = onesignal.retrieveSubscribed();
    ```
1. Obtenha um ID de jogador de sinal (String):

    ```js
        var res = onesignal.retrievePlayerId();
    ```
1. Obtenha um token de sinal (string):

    ```js
        var token = onesignal.retrieveToken();
    ```
1. Obter estado de assinatura de permissão (somente iOS por enquanto):

    ```js
        var res = onesignal.getPermissionSubscriptionState();
        /* res example:
            {
                "subscriptionStatus": {
                    "userSubscriptionSetting": true,
                    "subscribed": false,
                    "userId": "123-123-123-123-123456789",
                    "pushToken": null
                },
                "permissionStatus": {
                    "status": 2,
                    "provisional": false,
                    "hasPrompted": true
                },
                "emailSubscriptionStatus": {
                    "emailAddress": null,
                    "emailUserId": null
                }
            }
        */
    ```
1. postNotification (Por enquanto apenas para iOS):

    ```js
        //You can use idsAvailable for retrieving a playerId
        onesignal.postNotification({
            message:'Titanium test message',
            playerIds:["00000000-0000-0000-0000-000000000000"]
        });
    ```
1. Definir nível de registro (somente iOS por enquanto):

    ```js
        onesignal.setLogLevel({
            logLevel: onesignal.LOG_LEVEL_DEBUG,
            visualLevel: onesignal.LOG_LEVEL_NONE
        });
    ```
1. Ouvinte aberto:
    O conteúdo retornado corresponde à carga útil disponível no OneSignal:
   - [iOS](https://documentation.onesignal.com/docs/ios-native-sdk#section--osnotificationpayload-)
   - [Android](https://documentation.onesignal.com/docs/android-native-sdk#section--osnotificationpayload-)

    ```js
    onesignal.addEventListener('notificationOpened', function (evt) {
        alert(evt);
        if (evt) {
            var title = '';
            var content = '';
            var data = {};

            if (evt.title) {
                title = evt.title;
            }

            if (evt.body) {
                content = evt.body;
            }

            if (evt.additionalData) {
                if (Ti.Platform.osname === 'android') {
                    // O Android o recebe como uma string JSON
                    data = JSON.parse(evt.additionalData);
                } else {
                    data = evt.additionalData;
                }
            }
        }
        alert("Notification opened! title: " + title + ', content: ' + content + ', data: ' + evt.additionalData);
    });
    ```

1. Ouvinte recebido:
     O conteúdo retornado corresponde à carga útil disponível no OneSignal:
   - [iOS](https://documentation.onesignal.com/docs/ios-native-sdk#section--osnotificationpayload-)
   - [Android](https://documentation.onesignal.com/docs/android-native-sdk#section--osnotificationpayload-)

   ```js
   onesignal.addEventListener('notificationReceived', function(evt) {
       console.log(' ***** Received! ' + JSON.stringify(evt));
   });
   ```

Felicidades!

## Construa você mesmo

### iOS

Se você já tem o Titanium instalado, pule os 2 primeiros passos, senão vamos instalar o Titanium localmente.

1. `brew install yarn --without-node` para instalar o yarn sem depender de uma versão específica do Node
1. No diretório raiz, execute `yarn install`
1. Entre no diretório `ios`
1. Se você deseja atualizar o OneSignal SDK:
   - Executar `carthage update`
   - Arraste e solte o `OneSignal.framework` de `Carthage/Build/iOS` para `platform`
1. Altere o `titanium.xcconfig` para compilar com o SDK preferido
1. Para construir o módulo, execute `rm -rf build && ../node_modules/.bin/ti build -p ios --build-only`
### Android

1. `brew install yarn --without-node` para instalar o yarn sem depender de uma versão específica do Node
1. No diretório raiz, execute `yarn install`
1. Entre no diretório android
1. Copie `build.properties.dist` para `build.properties` e edite para corresponder ao seu ambiente
1. Para construir o módulo, execute `rm -rf build && mkdir -p build/docs && ../node_modules/.bin/ti build -p android --build-only`

#### Google Play Services

Desde o Titanium 7.x, este módulo depende de [https://github.com/appcelerator-modules/ti.playservices](ti.playservices)

Se você ainda precisar oferecer suporte ao Titanium 6.xe precisar alterar a versão usada do Google Play Services, execute as seguintes ações:
1. Instale o Google Play Services em seu sistema:

   ```bash
   sdkmanager "extras;google;m2repository"
   ```
1. Obtenha os 4 arquivos *.aar necessários do caminho do SDK `extras/google/m2repository/com/google/android/gms`
   - base
   - basement
   - gcm
   - idd
   - location

   Para a versão que você deseja usar.
1. Extraia o arquivo *.aar e renomeie o `classes.jar` para `google-play-services-<part>.jar`.
1. Atualize os jars usados ​​na pasta `lib`.
1. Atualize a pasta res com aquela do `google-play-services-basement.jar`
