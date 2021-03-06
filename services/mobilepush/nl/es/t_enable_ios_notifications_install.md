# Inicialización de SDK Push para apps iOS
{: #enable-push-ios-notifications-install}

Para un proyecto Xcode existente, puede configurar el SDK del cliente de Bluemix Mobile Services mediante la herramienta de gestión de dependencias de CocoaPods. Una alternativa es instalar el SDK manualmente.

**Nota**: Para ver el archivo readme Push de Swift, vaya a https://github.com/ibm-bluemix-mobile-services/bms-clientsdk-swift-push/tree/master

##Instalación de CocoaPods

1. Instale CocoaPods utilizando el siguiente mandato en su terminal Mac.
```
$ sudo gem install cocoapods
```
2. Especifique el mandato siguiente en el terminal para inicializar CocoaPods. Cuando emita este mandato, asegúrese de ejecutarlo en el directorio en el que se encuentra el proyecto de Xcode. El mandato `pod init` creará un título de archivo.  
```
$ pod init
```
3. En el Podfile generado, añada las dependencias de SDK que necesita. Copie el siguiente Podfile.

   Objective-C

    ```
    source 'https://github.com/CocoaPods/Specs.git'
	Copy the following list as is and remove the dependencies you do not need
	pod 'IMFCore'
	pod 'IMFPush'
	```

   Swift

	```
	source 'https://github.com/CocoaPods/Specs.git'
	// Copy the following list as is and remove the dependencies you do not need.
	use_frameworks!

	target 'MyApp' do
	    platform :ios, '8.0'
	    pod 'BMSCore'
	    pod 'BMSPush'
	end
	```
3. Desde el terminal, vaya a la carpeta del proyecto e instale las dependencias con el mandato siguiente:
```
$ pod update
```
Este mandato instala las dependencias y crea un nuevo espacio de trabajo Xcode.  **Nota**: Asegúrese de abrir siempre el nuevo espacio de trabajo de Xcode, en lugar del archivo de proyecto Xcode original:

	```
	$ open App.xcworkspace
	```
El espacio de trabajo contiene el proyecto original y el proyecto Pods que contiene las dependencias. Si desea modificar una carpeta fuente de Bluemix Mobile Services, puede encontrarla en el proyecto Pods, en `Pods/yourImportedSourceFolder`, por ejemplo: `Pods/IMFGoogleAuthentication`.

##Utilización de infraestructuras importadas y carpetas fuente

Haga referencia al SDK del código.


### Objective-C

Escriba directivas #import para las cabeceras relevantes, por
ejemplo:

```
//Objective-C
#import <IMFCore/IMFCore.h>
#import <IMFPush/IMFPush.h>
```

**Nota**: La actualización del proyecto Pods utilizando los mandatos CocoaPods `pod install` o `pod update` puede alterar temporalmente las carpetas fuente de Bluemix Mobile Services. Si desea conservar las versiones personalizadas
de los archivos originales, asegúrese de que se les ha hecho la copia de seguridad antes de emitir uno de estos
mandatos.

###Swift

**Requisitos previos**

- iOS 8.0 o superior
- Xcode 7


Escriba directivas #import para las cabeceras relevantes, por
ejemplo:

```
//swift
import BMSCore
import BMSPush
```


##Crear configuración

Vaya a **Xcode > Crear configuración > Opciones de creación y Establecer la habilitación de Bitcode** en **No**.

**Atención**: A partir de iOS 9, los cambios a la característica de App Transport Security (ATS) pueden afectar a la forma de manejar el proceso de autenticación. Las siguientes publicaciones de blog describen más información sobre los cambios:[ATS y Bitcode en iOS 9](https://developer.ibm.com/mobilefirstplatform/2015/09/09/ats-and-bitcode-in-ios9/) y [Conecte la app de iOS 9 a Bluemix ahora mismo](https://developer.ibm.com/bluemix/2015/09/16/connect-your-ios-9-app-to-bluemix/)




# Inicialización de SDK Push para apps iOS
{: #enable-push-ios-notifications-initialize}

Un lugar común para colocar el código de inicialización se encuentra en el delegado de aplicación para la aplicación iOS.
Pulse el enlace **Opciones móviles** en el Panel de control de aplicaciones de Bluemix para obtener la ruta de la aplicación y el GUID.

##Inicialización del SDK principal

###Objective-C

```
// Initialize the SDK for Object-C with IBM Bluemix GUID and route
IMFClient *imfClient = [IMFClient sharedInstance];
[imfClient initializeWithBackendRoute:"add_your_applicationRoute_here" backendGUID:"add_your_appId_here"];
```

###Swift

```
// Initialize the Core SDK for Swift with IBM Bluemix GUID, route, and region
let myBMSClient = BMSClient.sharedInstance

myBMSClient.initializeWithBluemixAppRoute("BluemixAppRoute", bluemixAppGUID: "APPGUID", bluemixRegion:"Location where your app Hosted")
myBMSClient.defaultRequestTimeout = 10.0 // Timput in seconds
```


##Inicialización del SDK Push del cliente

###Objective-C

```
//Initialize client Push SDK for Objective-C
IMFPushClient _pushService = [IMFPushClient sharedInstance];
```

###Swift

```
//Initialize client Push SDK for Swift
let push = BMSPushClient.sharedInstance
```

## Ruta, GUID, y región Bluemix

**appRoute**

Especifica la ruta que se asigna a la aplicación de servidor que ha creado en Bluemix.

**GUID**

Especifica la clave exclusiva asignada a la aplicación que ha creado en Bluemix. Este valor distingue entre mayúsculas y minúsculas.

**bluemixRegionSuffix**

Especifica la ubicación en la que se aloja la app. El parámetro `bluemixRegion` especifica qué despliegue de Bluemix está utilizando. Puede establecer este valor con una propiedad estática `BMSClient.REGION` y utilizar uno de estos tres valores:

- BMSClient.REGION_US_SOUTH
- BMSClient.REGION_UK
- BMSClient.REGION_SYDNEY




# Registro de dispositivos y aplicaciones iOS
{: #enable-push-ios-notifications-register}


Debe registrar una aplicación (app) con APNs para recibir notificaciones remotas, que normalmente se producen después de instalar la app en un dispositivo. Después de que la app reciba la señal de dispositivo que ha generado APNs, debe volver a pasarse al Servicio de notificaciones Push.

Para registrar las aplicaciones y los dispositivos de iOS:

1. Cree una aplicación de fondo
2. Pase la señal a las Notificaciones Push


##Cree una aplicación de fondo

Cree una aplicación de fondo en el catálogo Bluemix® de la sección de Contenedores modelo, que enlaza automáticamente el servicio Push a esta aplicación. Si ya ha creado una app de fondo, asegúrese de que enlace la app al Servicio de notificaciones Push.

###Objective-C

```
	//Para Objective-C
	- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
	if ([[[UIDevice currentDevice] systemVersion] floatValue] >= 8.0){
	    [[UIApplication sharedApplication] registerUserNotificationSettings:[UIUserNotificationSettings settingsForTypes:(UIUserNotificationTypeSound | UIUserNotificationTypeAlert | UIUserNotificationTypeBadge) categories:categories]];
	    [[UIApplication sharedApplication] registerForRemoteNotifications];
	    }
	    else{
	    [[UIApplication sharedApplication] registerForRemoteNotificationTypes:
	    (UIUserNotificationTypeBadge | UIUserNotificationTypeSound | UIUserNotificationTypeAlert)];
	    }
	    return YES;
	}
```

###Swift

```
	//For Swift
	func application(application: UIApplication, didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool {
		let notificationTypes: UIUserNotificationType = UIUserNotificationType.Badge | UIUserNotificationType.Alert | UIUserNotificationType.Sound
		let notificationSettings: UIUserNotificationSettings = UIUserNotificationSettings(forTypes: notificationTypes, categories: categories)
		application.registerUserNotificationSettings(notificationSettings)
		application.registerForRemoteNotifications()
	}
```

##Pase la señal a las Notificaciones Push

Después de recibir la señal desde APNs, pase la señal a Notificaciones Push como parte del método `registerDevice:withDeviceToken`.

###Objective-C

```
//Para Objective-C
-( void) application:( UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:( NSData *)deviceToken{

   IMFClient *client = [IMFClient sharedInstance];

 [client initializeWithBackendRoute:@"your-backend-route-here" backendGUID:@"Your-backend-GUID-here"];


 // get Push instance
IMFPushClient* push = [IMFPushClient sharedInstance];
[push registerDeviceToken:deviceToken completionHandler:^(IMFResponse *response,  NSError *error) {
   if (error){
     [ self  updateMessage:error .description];
  }  else {
    [ self updateMessage:response .responseJson .description];
}
}];
```

###Swift

Después de recibir la señal desde APNS, pase la señal a Notificaciones Push como parte del método `didRegisterForRemoteNotificationsWithDeviceToken`.

```
func application (application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: NSData){
   let push =  BMSPushClient.sharedInstance
   push.registerDeviceToken(deviceToken) { (response, statusCode, error) -> Void in
        if error.isEmpty {
            print( "Response during device registration : \(response)")
            print( "status code during device registration : \(statusCode)")
        }
        else{
            print( "Error during device registration \(error) ")
            Print( "Error during device registration \n  - status code: \(statusCode) \n Error :\(error) \n")
        }
    }

}
```



# Recepción de notificaciones push en dispositivos iOS
{: #enable-push-ios-notifications-receiving}

Recibir notificaciones push en dispositivos iOS.

##Objective-C
Para recibir notificaciones push en dispositivos iOS, añada el método Objective-C siguiente en la aplicación delegada de la aplicación.

```
// Para Objective-C
-(void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo {
//El diccionario userInfo contendrá datos que ha enviado el servidor.
}
```

##Swift
Para recibir notificaciones push en dispositivos iOS, añada el método Swift siguiente a la aplicación delegada de la aplicación.

```
 // Para Swift
func application(application: UIApplication, didReceiveRemoteNotification userInfo: [NSObject : AnyObject], fetchCompletionHandler completionHandler: (UIBackgroundFetchResult) -> Void) {
       //El diccionario UserInfo contendrá datos que ha enviado el servidor
   }

```



# Envío de notificaciones push básicas
{: #push-send-notifications}

Una vez que haya desarrollado sus aplicaciones, puede enviar notificaciones push básicas (sin utilizar etiquetas, identificadores, cargas útiles adicionales o archivos de sonido).


Enviar notificaciones push básicas.

1. En **Elegir la audiencia**, seleccione una de las siguientes audiencias:
      **Todos los dispositivos**, o por plataforma: **Sólo dispositivos iOS** o
      **Sólo dispositivos Android**. 

	**Nota**: Cuando seleccione la opción **Todos los dispositivos**, todos los dispositivos que se hayan suscrito a las notificaciones push recibirán la notificación.

	![pantalla Notificaciones](images/tag_notification.jpg)

2. En **Crear la notificación**, escriba el mensaje y pulse **Enviar**.
3. Verifique que los dispositivos hayan recibido la notificación.

	La captura de pantalla siguiente muestra un recuadro de alerta que maneja una notificación push
en el primer plano en un dispositivo Android e iOS.

	![Notificación push en primer plano en Android](images/Android_Screenshot.jpg)

	![Notificación push en primer plano en iOS](images/iOS_Screenshot.jpg)

	La captura de pantalla siguiente muestra una notificación push en segundo plano para Android.
	![Notificación push en el fondo en Android](images/background.jpg)




# Pasos siguientes
{: #next_steps_tags}

Una vez que haya configurado correctamente las notificaciones básicas, puede configurar las notificaciones basadas en código y las opciones avanzadas.

Añada estas características del servicio de notificaciones push a su app.
Para utilizar notificaciones basadas en código, consulte [Notificaciones basadas en código](c_tag_basednotifications.html).
Para utilizar opciones de notificaciones avanzadas, consulte [Notificaciones push avanzadas](t_advance_notifications.html).
