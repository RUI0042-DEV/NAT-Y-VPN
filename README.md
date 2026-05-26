### **Activitat: UD9.AA4 NAT i VPN**

**Objectius de l'activitat**
*   Configurar l'accés remot a un servei a través de DNAT.
*   Configurar l'accés remot a través d'una VPN.
*   Comparar les característiques d'ambdues opcions d'accés.

---

#### **1. Requisits i Configuració de l'Escenari**
Per a aquesta activitat es requereixen tres màquines virtuals (VM):
*   **Tallafocs (IPFire):** Amb dues interfícies. La interfície `eth0` (RED) connectada a la xarxa externa (xarxa NAT) en mode DHCP, i la interfície `eth1` (GREEN) connectada a la xarxa interna.
*   **Servidor Local (Zorin OS):** Connectat a la xarxa interna amb una IP estàtica dins del rang `192.169.x.0/24`.
*   **Equip Client (Windows o Linux):** Connectat a la xarxa externa (xarxa NAT) simulant estar a l'exterior.

#### **2. Configuracions Inicials**
Abans d'aplicar regles d'enrutament, has de preparar el servidor intern:
1.  Comprova que tens connectivitat a Internet, tant des del client com des del Zorin.
2.  Instal·la al Zorin els serveis de **SSH** i el servidor web **Apache**.
3.  Edita l'arxiu `index.html` per defecte d'Apache perquè mostri un missatge distintiu que inclogui el teu nom.
4.  Comprova localment (des de la pròpia màquina Zorin) que els dos serveis funcionen correctament.

#### **3. Fase A: Accés mitjançant Destination NAT (DNAT)**
En aquesta fase has d'exposar els serveis interns a l'exterior utilitzant regles de *Port Forwarding*:
1.  Configura i documenta a l'IPFire la regla necessària per tal que ens puguem **connectar via SSH** des de l'equip Client extern al Zorin intern.
2.  Configura i documenta la regla necessària per tal que puguem **accedir via web** (HTTP) des de l'equip Client a la pàgina allotjada al Zorin.
3.  **Comprovació:** Demostra des de la màquina Client que aconsegueixes connectar-te a ambdós serveis utilitzant únicament la IP "pública" (la de la xarxa RED) de l'IPFire.

#### **4. Fase B: Accés mitjançant Xarxa Privada Virtual (VPN)** 
En aquesta fase configuraràs un túnel segur OpenVPN perquè el client extern accedeixi a la xarxa local com si hi fos físicament:
1.  **Generació de certificats:** Accedeix a IPFire i genera els certificats Root i Host per al teu domini.
2.  **Configuració del Servidor VPN:** Configura una subxarxa dedicada per als clients VPN (per exemple, `10.25.213.0/24`) i inicia el servidor OpenVPN a la interfície RED.
3.  **Creació d'usuari:** Afegeix una nova connexió client del tipus *Host-to-Net (Roadwarrior)* i descarrega el seu paquet de configuració (.zip).
4.  **Preparació del Client:** A la màquina Client externa, edita l'arxiu de resolució DNS local (`hosts`) perquè el domini de la VPN apunti a la IP pública de l'IPFire.
5.  **Connexió i Comprovació final:** Instal·la el client OpenVPN a la màquina externa, importa els certificats i connecta't. Demostra que ara **pots accedir al servidor web del Zorin introduint directament la seva IP privada** de la xarxa interna, sense utilitzar les regles DNAT.

---

[Guia](guia.md)
