# SDK para MiPOT iOS
# Versión actual: 0.0

Este es el repositorio oficial para las versiones de SDK del dispositivo MiPOT


## MiPOT ya cuenta con Basic Auth

ENDPOINT: Solicitar por correo a ricardo@atomicthings.com

Agregar header "Authentication" en la petición.
y como valor el base64 de "Basic packageName:KEY".

Donde:
* packageName es el Bundle Identifier de la app.
* KEY es una cadena suministrada por Atomic Things.


Este SDK aplica para la versión de firmware de MiPOT v0.96L en adelante.

### Guía de inicio

Para iniciar, únicamente es necesario descargar el archivo MiPOT.framework.

### Prerequisitos

* Desarrollo en Swift 5
* Xcode 10
* MacOS Mojave 10.14

### Instalación
En esta sección se enumeran los pasos para colocar el archivo "MiPOT.framework" en un lugar adecuado para su consumo.

  Crear una carpeta llamada "Frameworks" en la carpeta raíz de tu proyecto (ejemplo: ~/Documents/iOSDev/MiApp). Este paso es opcional así como recomendable para mantener orden en tu proyecto.
  Colocar el archivo "MiPOT.framework" en una carpeta o subcarpeta de tu proyecto (ejemplo: ~/Documents/iOSDev/MiApp/Frameworks).

(imagen)


### Integración

En esta sección se detallarán los pasos necesarios para que MiPOT funcione con tu
aplicación.

#####1) Definir los permisos necesarios en nuestro archivo Info.plist.

Para integrar MiPOT a tu aplicación se requiere agregar un permiso en la categoría "Information Property List". Para ello:

  Ir al área de navegación.
    Abrir el archivo Info.plist y damos click secundario sobre la categoría "Information Property List".
    Seleccionar opción "Add row" del menú emergente.
    Localizar el permiso llamado "Privacy - Camera Usage Description" y seleccionarlo.
    Escribir en el campo "Value" de este permiso en cuestión la siguiente descripción: "Esta app utilizará la cámara para escanear códigos QR".
(imagen)

#####2) Insertar el framework en tu proyecto.

  Ir al área de navegación.
  Mostrar la lista de proyectos y targets y seleccionar el target.
  Seleccionar botón "General" en la parte superior del área de edición.
  Encontrar el apartado "Embeded Binaries".
  Dar click en el botón de agregar.
  Dar click en el botón "Add Other...".
  Dirigirse a la dirección en la que se encuentra el archivo MiPOT.framework.
  Seleccionar archivo y dar click en "Open".
  Verificar que en la ventana emergente se encuentren seleccionadas las opciones "Copy items if needed" y "Create folder references".

#####3) Realizar una prueba de verificación.

Esta prueba es opcional pero es muy útil para verificar que el framework se logró integrar correctamente. Con fines de aislar esta prueba realizamos los siguientes pasos.

  Crear un nuevo archivo .swift en nuestro proyecto y le daremos el nombre de "Prueba.swift"
  Escribir en la sección de importaciones del archivo "import MiP".

Si en las opciones de autocompletado aparece MiPOT, el framework se ha añadido exitosamente al proyecto. De ser así, el archivo "Prueba.swift" se puede eliminar del proyecto.

## Lector QR  

#####Paso 1: Leer QR
Para abrir el escaner con la acción de un botón tenemos el siguiente código:

```
    @IBAction func testCamera(_ sender: Any) {
        let escanearQR = QR.init()
        escanearQR.presentQrController(self)
    }
```
**¡Atención!**: Vale la pena recalcar que el framework de MiPOT no provee los permisos de cámara. Estos deben ser añadidos en el archivo "Info.plist" como se indica en el punto 1 del apartado "Integración".

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
- Importar el framework CoreBluetooth.
- Contar con la cadena recuperada por la función *codeSended(code: String)*.
- Que el UIViewController de la pantalla en cuestión herede del protocolo CBCentralManagerDelegate.