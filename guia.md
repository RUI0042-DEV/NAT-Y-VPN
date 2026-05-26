# Pràctica UD9.AA4: NAT i VPN

## 1. Configuracions Inicials i Preparació del Servidor
L'objectiu d'aquesta fase és preparar la nostra màquina de la xarxa interna (Zorin OS) perquè actuï com a servidor web i servidor SSH.

Primer, instal·lem i verifiquem els serveis al servidor Zorin (IP 192.169.11.11):

    sudo apt install apache2 openssh-server
    sudo systemctl restart apache2

A continuació, editem l'arxiu index.html per defecte d'Apache perquè mostri un missatge distintiu (en aquest cas, "Servidor de Rui"):

    sudo nano /var/www/html/index.html

![Edició de l'index.html](pics/Captura%20de%20pantalla%202026-05-21%20191643.png)
![Codi de l'index.html](pics/Captura%20de%20pantalla%202026-05-21%20192308.png)

Comprovem localment que el servidor web funciona accedint a localhost des del propi Zorin:
![Comprovació en localhost](pics/Captura%20de%20pantalla%202026-05-21%20192343.png)

## 2. Configuració de Destination NAT (DNAT)
En aquesta fase configurarem el tallafocs IPFire per permetre l'accés remot des de l'exterior (xarxa NAT) als serveis interns de Zorin mitjançant Port Forwarding.

Accedim a la interfície web d'IPFire i anem a Firewall > Reglas del Cortafuegos. Creem dues regles:
1. Regla HTTP: Origen RED -> NAT destí (DNAT) -> Destí 192.169.11.11 -> Protocol Preestablert HTTP (Port 80).
2. Regla SSH: Origen RED -> NAT destí (DNAT) -> Destí 192.169.11.11 -> Protocol Preestablert SSH (Port 22).

Apliquem els canvis i verifiquem que ambdues regles estan actives:
![Regles DNAT a IPFire](pics/Captura%20de%20pantalla%202026-05-21%20194018.png)

### Comprovacions del DNAT des del Client (Exterior)
Per demostrar que l'enrutament funciona, ens n'anem a la màquina Client situada a la xarxa externa i ataquem a la IP pública de l'IPFire (10.0.2.17).

Prova Web:
Accedim a http://10.0.2.17 des del navegador del Client i verifiquem que ens carrega la pàgina interna:
![Prova Web DNAT](pics/Captura%20de%20pantalla%202026-05-21%20194740.png)

Prova SSH:
Des de la terminal del Client, iniciem connexió contra la IP pública del firewall:

    ssh rui@10.0.2.17

Verifiquem que aconseguim entrar al servidor intern i confirmem amb un ip a que, efectivament, estem dins de la màquina 192.169.11.11:
![Prova SSH DNAT 1](pics/Captura%20de%20pantalla%202026-05-21%20194946.png)
![Prova SSH DNAT 2](pics/Captura%20de%20pantalla%202026-05-21%20195101.png)

## 3. Configuració del Servidor VPN (OpenVPN)
Per a un accés més segur i professional, configurarem una VPN perquè el client extern obtingui una IP virtual i treballi com si estigués dins de la LAN.

### 3.1 Generació de Certificats
A IPFire, anem a Servicios > OpenVPN i generem els certificats Root/Host especificant el nostre domini:
* Nom d'organització: ipfire
* Nom de host: ipfire.foodlogistic.test

![Formulari Certificats](pics/Captura%20de%20pantalla%202026-05-21%20200126.png)

Confirmem que els certificats s'han creat correctament a la taula d'Autoritats Certificadores:
![Certificats Generats](pics/Captura%20de%20pantalla%202026-05-21%20201210.png)

### 3.2 Configuració Global i Arrancada
A la secció de Configuracions Globals:
1. Marquem OpenVPN en RED.
2. Definim la subxarxa d'OpenVPN (diferent de la GREEN): 10.25.213.0/255.255.255.0.
3. Guardem i iniciem el servidor.

### 3.3 Creació de la Connexió Client (Roadwarrior)
Afegim una nova connexió del tipus Host-to-Net (Roadwarrior) per al nostre client extern:
* Nom: vpnrui
* Generem un certificat nou introduint una contrasenya PKCS12 segura.
* Guardem la configuració.

![Configuració de l'usuari VPN](pics/Captura%20de%20pantalla%202026-05-26%20184932.png)

Comprovem que l'usuari s'ha creat correctament i descarreguem el paquet de configuració (.zip) utilitzant la icona del disquet a la columna Acción:
![Usuari vpnrui creat](pics/Captura%20de%20pantalla%202026-05-26%20185159.png)

## 4. Configuració del Client VPN i Prova Final
Traslladem l'arxiu .zip descarregat a la nostra màquina Client externa.

### 4.1 Resolució DNS Local
Abans de connectar, és necessari que la màquina client sàpiga resoldre el domini del servidor VPN. Editem l'arxiu hosts del client (Linux):

    sudo nano /etc/hosts

Afegim la línia apuntant a la IP pública de l'IPFire:
10.0.2.17 ipfire.foodlogistic.test

![Edició arxiu hosts](pics/Captura%20de%20pantalla%202026-05-21%20191034.png)

### 4.2 Connexió i Comprovació Final
Instal·lem el client OpenVPN i executem l'arxiu de configuració extret del .zip (introduint la contrasenya PKCS12 quan se'ns sol·liciti).

Una vegada establerta la connexió VPN, el nostre equip client ja forma part de la xarxa virtual. Per demostrar-ho, obrim el navegador i accedim directament a la IP privada del Zorin (192.169.11.11), aconseguint carregar la web correctament sense necessitat d'utilitzar la IP pública ni regles DNAT:

![Accés VPN a xarxa interna](pics/Captura%20de%20pantalla%202026-05-26%20190823.png)

---

[Tornar enrere](README.md)
