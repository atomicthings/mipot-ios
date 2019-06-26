# SDK para MiPOT iOS
# Versión actual: 0.0

Este es el repositorio oficial para las versiones de SDK del dispositivo MiPOT


## MiPOT ya cuenta con Basic Auth

ENDPOINT: Solicitar por correo a ricardo@atomicthings.com

Agregar header "Authentication" en la petición
y como valor el base64 de "Basic packageName:KEY"

Donde:
* packageName es el Bundle Identifier de la aplicación
* KEY es una cadena suministrada por Atomic Things.


Este SDK aplica para la versión de firmware de MiPOT v0.96L en adelante.

### Guía de inicio

Para iniciar, únicamente es necesario descargar el archivo MiPOT.framework

### Prerequisitos

* Desarrollo en Swift 5
* Xcode 10
* MacOS Mojave 10.14

### Installing
En esta sección se enumeran los pasos para colocar el archivo MiPOT.framework en un lugar adecuado para su consumo.

1) En la carpeta raíz de tu proyecto (ejemplo: ~/Documents/iOSDev/MiApp) crea una carpeta llamada "Frameworks". Aunque este paso es opcional, es muy recomendable para mantener orden en tu proyecto.
2) Coloca en una carpeta o subcarpeta de tu proyecto (ejemplo: ~/Documents/iOSDev/MiApp/Frameworks) el archivo MiPOT.framework.

(imagen)


### Integración

En esta sección se detallarán los pasos necesarios para que MiPOT funcione con tu
aplicación.

1) Definir los permisos necesarios en nuestro archivo Info.plist.

Para integrar MiPOT a tu aplicación se requiere agregar un permiso en la categoría "Information Property List". Para ello:

a) Ir al área de navegación.
b) Abrir el archivo Info.plist y damos click secundario sobre la categoría "Information Property List" 
c) Seleccionar opción "Add row" del menú emergente.
d) Localizar el permiso llamado "Privacy - Camera Usage Description" y seleccionarlo.
e) Escribir en el campo "Value" de este permiso en cuestión la siguiente descripción: "Esta app utilizará la cámara para escanear códigos QR".
(imagen)

2) Insertar el framework en tu proyecto.

a) Ir al área de navegación.
b) Mostrar la lista de proyectos y targets y seleccionar el target.
c) Seleccionar botón "General" en la parte superior del área de edición.
d) Encontrar el apartado "Embeded Binaries".
e) Dar click en el botón de agregar.
f) Dar click en el botón "Add Other..."
g) Dirigirse a la dirección en la que se encuentra el archivo MiPOT.framework.
h) Seleccionar archivo y dar click en "Open".
i) Verificar que en la ventana emergente se encuentren seleccionadas las opciones "Copy items if needed" y "Create folder references".

3) Realizar una prueba de verificación.

Esta prueba es opcional pero es muy útil para verificar que el framework se logró integrar correctamente. Con fines de aislar esta prueba realizamos los siguientes pasos.

a) Creamos un nuevo archivo .swift en nuestro proyecto y le daremos el nombre de "Prueba.swift"
b) Escribimos en la sección de importaciones del archivo "import MiP".

Si en las opciones de autocompletado aparece MiPOT, el framework se ha añadido exitosamente al proyecto. De ser así, el archivo "Prueba.swift" se puede eliminar del proyecto.

## Lector QR  

**Paso 1: Leer QR**
Para abrir el escaner con la acción de un botón tenemos el siguiente código:

```
    @IBAction func testCamera(_ sender: Any) {
        let escanearQR = QR.init()
        escanearQR.presentQrController(self)
    }
```
Atención!: Vale la pena recalcar que el framework de MiPOT no provee los permisos de cámara. Estos deben ser añadidos en el archivo "Info.plist" como se indica en el punto 1 del apartado "Integración".

El código anterior abrirá la cámara y buscará un QR válido para conectarse al dispositivo MiPOT. La respuesta se obtendrá implementando una extensión. Esta extensión se aplica a la clase tipo UIViewController de la pantalla en la que se implementa el lector QR y hereda del protocolo didFinishReceiveCode. 

```
extension ViewController:didFinishReceiveCode{
    func codeSended(code: String) {
        print("Code sended \(code)")
    }
}
```

## Conexión BLE
Para la conexión BLE es importante:
- Importar el framework CoreBluetooth
- Contar con la cadena recuperada por la función ´´´codeSended(code: String)´´´.
- Que el UIViewController de la pantalla en cuestión herede del protocolo CBCentralManagerDelegate.

(PROTOCOLS DE ESTADO BLE)

____
**Ciclo de vida de la actividad y el servicio**
El siguiente bloque de código sirve para establecer la comunicación
con el servicio de BLE
En *gattIntentFilter()* se agregan las respuestas que se esperan en el BroadcastReceiver

```
protected void onCreate(Bundle savedInstanceState) {
       super.onCreate(savedInstanceState);
       ...
       Intent miPotService = new Intent(this, BLEService.class);
       bindService(miPotService,serviceConnection,BIND_AUTO_CREATE);
}

@Override
 protected void onResume() {
     super.onResume();
     registerReceiver(receiver,gattIntentFilter());
 }

 @Override
 protected void onPause() {
     super.onPause();
     unregisterReceiver(receiver);
 }

 @Override
 protected void onDestroy() {
     super.onDestroy();
     unbindService(serviceConnection);
     if(miPot != null)
         miPot.disconnectMiPOT();
     miPot = null;
 }

 private static IntentFilter gattIntentFilter() {
    IntentFilter intentFilter = new IntentFilter();
    intentFilter.addAction(BLEService.ACTION_GATT_CONNECTED);
    intentFilter.addAction(BLEService.ACTION_GATT_DISCONNECTED);
    intentFilter.addAction(BLEService.ACTION_GATT_SERVICES_DISCOVERED);

    intentFilter.addAction(BLEService.ACTION_VERIFICATION_DATA);
    intentFilter.addAction(BLEService.ACTION_WALLET_MIPOT_SEND_DATA);
    intentFilter.addAction(BLEService.ACTION_WALLET_DEVICE_VERIFICATION);
    return intentFilter;
}

```


**Paso 2: Obtener Barcode**
El siguiente bloque de código se agrega en el onCreate y sirve para obtener el
barcode del QR leído y verificar el Bluetooth encendido.
```
// Variable global
String barcode;

Intent dataBarcode = getIntent();
if(dataBarcode.hasExtra(EXTRA_BARCODE_DATA)){
  BluetoothManager bluetoothManager = (BluetoothManager) Objects.requireNonNull(getApplicationContext()).getSystemService(Context.BLUETOOTH_SERVICE);
  BluetoothAdapter mBluetoothAdapter = bluetoothManager.getAdapter();
  if (mBluetoothAdapter == null || !mBluetoothAdapter.isEnabled()) {
    startActivityForResult(new Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE), BLUETOOTH_REQUEST);
  }else{
    barcode = dataBarcode.getStringExtra(EXTRA_BARCODE_DATA);
  }
}
```
**Paso 3 y 4: Establecer conexión**
Si se puede inicializar una conexión (paso 3) se realiza la conexión (paso 4)
```
if(miPot.miPotInitializeConnection()){
  miPot.miPotConnect(barcode);
}
```

**Paso 5: Solicitar id MiPOT para validación**
Se debe solicitar el id de MiPOT para verificar que el dispositivo
sea autentico
```
if(!miPot.miPotGetAuthData()){
  // Si entra aquí se muestra un mensaje de error
}
```

**Paso 6: Recibiendo la respuesta del id**
En el switch del BroadcastReceiver caerá la respuesta de MiPOT
al haber solicitado el id. Se tiene un tipo de dato HashMap<String,String>
```
case BLEService.ACTION_VERIFICATION_DATA:
```

**Paso 7: Verificando cadena con id en el servidor**
Una vez que se tiene la cadena, se debe enviar una solicitud al servidor
de AtomicThings para verificar MiPOT
#POST
- header ->  Content-Type:application/x-www-form-urlencoded
- body   ->  message: Hashmap en formato JSON (new JSONObject(map))



![alt text](https://img.icons8.com/color/48/000000/error.png "Alerta") **Cambio en la nueva versión 4/4**
*  Mejoras en seguridad


**Paso 8: El resultado de la petición POST**
Este paso consiste en solo tener control del resultado de la petición.
La respuesta tiene el siguiente formato:
*{message:cadenaCifrada, token:tokenDeCifrado}*

Se debe de obtener la cadenaCifrada y el token para agregar como parámetros al
método miPotValidation(cadenaCifrada,tokenDeCifrado)


**Paso 9: Regresando respuesta a MiPOT**
Para seguir con el proceso se requiere mandar la información a MiPOT
Esto se hace con miPotValidation
```
if(!miPot.miPotValidation(cadenaCifrada,tokenDeCifrado)){
  // Si entra aquí se muestra un mensaje de error
}
```

**Paso 10: Validando MiPOT**
MiPOT se puede desconectar. Algunas razones son:
- El dispositivo MiPOT fue alterado o es un clon del original
- Hemos detectado comportamiento extraño y se tomó la decisión de bloquearlo
Se manda un código de error a
```
case BLEService.ACTION_WALLET_DEVICE_VERIFICATION:
```
para conocer el estado del sistema.


**Paso 11 y 12**
Estos pasos corresponden a la petición de tu Wallet para hacer un intento de cobro
El paso 12 corresponde a la obtención del resultado listo para mandarlo a MiPOT


**Paso 13: MiPOT muestra el resultado**
Para envíar resultado a MiPOT es necesario usar
```
if(!miPot.miPotSendData(true,monto)){
  // Si entra aquí se muestra un mensaje de error (No fue posible mandar mensaje)
}
```
donde el primer parámetro consiste en el estado de la transacción y el segundo en
el monto. El primer parámetro es un boolean y el segundo un String

**MiPOT enciende en este paso**


**Paso 14: Fin de la comunicación**
Se puede obtener un último mensaje en
```
case BLEService.ACTION_WALLET_MIPOT_SEND_DATA
```
que sirve para conocer, si es que hubo algún error, la razón por la cual
MiPOT no lograra encender.


## Contribuciones
Próximamente

## Autores

* **ReivajGP**  [Reivaj](https://github.com/ReivajGP)

## Licencia

Próximamente

## Agradecimientos

Próximamente
