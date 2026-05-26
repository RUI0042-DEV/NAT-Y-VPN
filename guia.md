# Práctica UD9.AA4: NAT i VPN

## 1. Configuraciones Iniciales y Preparación del Servidor
El objetivo de esta fase es preparar nuestra máquina de la red interna (Zorin OS) para que actúe como servidor web y servidor SSH.

Primero, instalamos y verificamos los servicios en el servidor Zorin (IP 192.169.11.11):

    sudo apt install apache2 openssh-server
    sudo systemctl restart apache2

A continuación, editamos el archivo index.html por defecto de Apache para que muestre un mensaje distintivo (en este caso, "Servidor de Rui"):

    sudo nano /var/www/html/index.html

![Edición del index.html](Captura de pantalla 2026-05-21 191643.png)
![Código del index.html](Captura de pantalla 2026-05-21 192308.png)

Comprobamos localmente que el servidor web funciona accediendo a localhost desde el propio Zorin:
![Comprobación en localhost](Captura de pantalla 2026-05-21 192343.png)

## 2. Configuración de Destination NAT (DNAT)
En esta fase configuraremos el cortafuegos IPFire para permitir el acceso remoto desde el exterior (xarxa NAT) a los servicios internos de Zorin mediante Port Forwarding.

Accedemos a la interfaz web de IPFire y vamos a Firewall > Reglas del Cortafuegos. Creamos dos reglas:
1. Regla HTTP: Origen RED -> NAT destino (DNAT) -> Destino 192.169.11.11 -> Protocolo Preestablecido HTTP (Puerto 80).
2. Regla SSH: Origen RED -> NAT destino (DNAT) -> Destino 192.169.11.11 -> Protocolo Preestablecido SSH (Puerto 22).

Aplicamos los cambios y verificamos que ambas reglas están activas:
![Reglas DNAT en IPFire](Captura de pantalla 2026-05-21 194018.png)

### Comprobaciones del DNAT desde el Cliente (Exterior)
Para demostrar que el enrutamiento funciona, nos vamos a la máquina Cliente situada en la red externa y atacamos a la IP pública del IPFire (10.0.2.17).

Prueba Web:
Accedemos a http://10.0.2.17 desde el navegador del Cliente y verificamos que nos carga la página interna:
![Prueba Web DNAT](Captura de pantalla 2026-05-21 194740.png)

Prueba SSH:
Desde la terminal del Cliente, iniciamos conexión contra la IP pública del firewall:

    ssh rui@10.0.2.17

Verificamos que logramos entrar al servidor interno y confirmamos con un ip a que, efectivamente, estamos dentro de la máquina 192.169.11.11:
![Prueba SSH DNAT 1](Captura de pantalla 2026-05-21 194946.png)
![Prueba SSH DNAT 2](Captura de pantalla 2026-05-21 195101.png)

## 3. Configuración del Servidor VPN (OpenVPN)
Para un acceso más seguro y profesional, configuraremos una VPN para que el cliente externo obtenga una IP virtual y trabaje como si estuviera dentro de la LAN.

### 3.1 Generación de Certificados
En IPFire, vamos a Servicios > OpenVPN y generamos los certificados Root/Host especificando nuestro dominio:
* Nombre de organización: ipfire
* Nombre de host: ipfire.foodlogistic.test

![Formulario Certificados](Captura de pantalla 2026-05-21 200126.png)

Confirmamos que los certificados se han creado correctamente en la tabla de Autoridades Certificadoras:
![Certificados Generados](Captura de pantalla 2026-05-21 201210.png)

### 3.2 Configuración Global y Arranque
En la sección de Configuraciones Globales:
1. Marcamos OpenVPN en RED.
2. Definimos la subred de OpenVPN (distinta a la GREEN): 10.25.213.0/255.255.255.0.
3. Guardamos e iniciamos el servidor.

### 3.3 Creación de la Conexión Cliente (Roadwarrior)
Añadimos una nueva conexión del tipo Host-to-Net (Roadwarrior) para nuestro cliente externo:
* Nombre: vpnrui
* Generamos un certificado nuevo introduciendo una contraseña PKCS12 segura.
* Guardamos la configuración.

![Configuración del usuario VPN](Captura de pantalla 2026-05-26 184932.png)

Comprobamos que el usuario se ha creado correctamente y descargamos el paquete de configuración (.zip) usando el icono del disquete en la columna Acción:
![Usuario vpnrui creado](Captura de pantalla 2026-05-26 185159.png)

## 4. Configuración del Cliente VPN y Prueba Final
Trasladamos el archivo .zip descargado a nuestra máquina Cliente externa.

### 4.1 Resolución DNS Local
Antes de conectar, es necesario que la máquina cliente sepa resolver el dominio del servidor VPN. Editamos el archivo hosts del cliente (Linux):

    sudo nano /etc/hosts

Añadimos la línea apuntando a la IP pública del IPFire:
10.0.2.17 ipfire.foodlogistic.test

![Edición archivo hosts](Captura de pantalla 2026-05-21 191034.png)

### 4.2 Conexión y Comprobación Final
Instalamos el cliente OpenVPN y ejecutamos el archivo de configuración extraído del .zip (introduciendo la contraseña PKCS12 cuando se nos solicite).

Una vez establecida la conexión VPN, nuestro equipo cliente ya forma parte de la red virtual. Para demostrarlo, abrimos el navegador y accedemos directamente a la IP privada del Zorin (192.169.11.11), logrando cargar la web correctamente sin necesidad de usar la IP pública ni reglas DNAT:

![Acceso VPN a red interna](Captura de pantalla 2026-05-26 190823.png)
