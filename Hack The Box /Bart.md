# Bart

**Sistema Operativo**: Windows

**Dificultad**: Media

**Técnicas Vistas**: Subdomain Enumeration - Gobuster /
Information Leakage /
Username enumeration - Abusing the Forget Password Option /
Simple Chat Exploitation - Creating a new user /
Log Poisoning Attack - User Agent [RCE] /
Nishang Invoke-PowerShellTcp Shell /
Abusing SeImpersonatePrivilege [Privilege Escalation]

Te prepara para las **Certificaciones**: OSCP / 
eWPT / 
eWPTXv2 / 
OSWE<br><br>

![image](https://github.com/user-attachments/assets/24fb5316-aa4e-4ede-8680-945d598d1b6a)<br><br>

# Reconocimiento

Utilizando `nmap` escaneamos todos los puertos<br>

![image](https://github.com/user-attachments/assets/287b0f5c-f01c-4c3e-9dd8-df9ff7787bf9)

- `-p-` -> Escanea todos los puertos 
- `-sS` -> escaneo de tipo SYN
- `--open` -> Todos los puertos abiertos
- `--min-rate=5000` -> Manda mínimo 5000 paquetes por segundo
- `-Pn` -> Asume que el host especificado está activo
- `-vvv` -> Aplica triple Verbose 
- `-n` -> Que no aplique resolución DNS
- `-oG allports` -> manda todo a un archivo con nombre de "allports"<br><br>

Agregamos el Subdominio y Dominio en `/etc/hosts`<br>

![image](https://github.com/user-attachments/assets/0a336eee-e3d8-4aca-b1f8-3f5e9dbee17e)<br><br>


Vemos un nombre y apellido en los comentarios del código fuente<br>

![image](https://github.com/user-attachments/assets/a84561d5-0b83-429b-88b3-9964e1815f13)<br><br>


Utilizando `gobuster` vemos otro subdominio `monitor.bart.htb`<br>

![image](https://github.com/user-attachments/assets/6c806c41-d4b3-4f17-a2fa-85631ca44e28)<br>

- `vhost` ->  Buscar Subdominios 
- `-u` -> Indicar URL
- `-w` ->  Indicar diccionario que voy a utilizar
- `-t` -> Numero de hilos que voy a usar
- `--append-domain` -> Para que no salte error<br><br>

# Information Leakage 

Si ponemos `Harvey` de usuario y `Potter` de contraseña entramos<br>

![image](https://github.com/user-attachments/assets/adcd9b2c-c0a1-4e4f-996b-30145442afaa)<br>
Agregamos otro Subdominio<br>
![image](https://github.com/user-attachments/assets/11c73f01-5c1f-4235-95b8-45bfd1d8b521)<br><br>
Podemos pensar que `simple_chat` es un proyecto open source, si buscamos vamos a encontrar un github "https://github.com/magkopian/php-ajax-simple-chat/tree/master/simple_chat" Donde vemos una ruta `http://internal-01.bart.htb/simple_chat/register.php` <br>

![image](https://github.com/user-attachments/assets/d994fa97-c1b0-4d3c-8419-7a4d497208a2)<br><br>

Creamos un usuario `santiago` de contraseña `santiago123`<br>

![image](https://github.com/user-attachments/assets/cb7c3a3b-d00f-4bc5-ad21-63d072114a0e)<br><br>



# Explotación Log-Poissonging

En el codigo fuente vemos esta ruta, y podemos observar que crea un archivo `log.txt` el cual podemos visualizar en `http://internal-01.bart.htb/log/log.txt` Donde vemos que nos muestra nuestro  `User-Agent`.<br>

![image](https://github.com/user-attachments/assets/41ed80c6-cd09-4068-aa1d-418f43e77dd8)<br><br>

Con este pequeño Script mandamos codigo php para poder ejecutar comandos:<br>

![image](https://github.com/user-attachments/assets/c77d373e-7363-4128-815b-3d51f4ce5c05)<br><br>


Nos entablamos una reverse shell<br>

![image](https://github.com/user-attachments/assets/e6a13299-e018-4856-9292-96c5cf51fce1)<br><br>

hacemos un `locate nc` y luego `cp /usr/share/seclists/Web-Shells/FuzzDB/nc.exe .`, y ahi compartimos el recurso con `impacket-smbserver smbFolder $(pwd) -smb2support`, tambien nos ponemos en escucha `rlwrap nc -nlvp 443`.<br>

![image](https://github.com/user-attachments/assets/6c6c40ae-058b-440d-989e-dcb3c37f30ab)<br><br>



# Escalada de privilegios

`SeImpersonatePrivilege` es vulnerable

![image](https://github.com/user-attachments/assets/4e4efa86-635f-4350-bb05-c6d62fe764a2)

##### Descargamos binario

https://github.com/ohpe/juicy-potato/releases/tag/v0.1  -> Abusa de ese `SeImpersonatePrivilege` para ejecutarte una acción privilegiada.

Nos lo transferimos:

![image](https://github.com/user-attachments/assets/b0c40968-6dbb-45b1-bdf1-297e045cbdc3)

https://github.com/ohpe/juicy-potato/tree/master/CLSID  -> Buscamos un CLSID para windows 10 pro


Creamos un nuevo usuario: santiago:santiago123

![image](https://github.com/user-attachments/assets/5b005282-a9b2-4ae0-b2a1-c7304ac01ab5)


##### Nos descargamos netcat para windows

https://eternallybored.org/misc/netcat/  -> Lo transferimos al Windows como lo hicimos con Juicy-potato<br>


Ejecutamos el `nc.exe` privilegiado con el `juicy-potato` <br>

![image](https://github.com/user-attachments/assets/9d698d86-7b92-4006-8fd0-12cbd6c9934e)<br><br>

- rlwrap nc -nlvp 443<br><br>

Vemos la Flag del usuario:<br>

![image](https://github.com/user-attachments/assets/bfa7568b-ac3f-4192-a1b6-d6e9b8c64660)<br><br>


Vemos la Flag Final:<br>

![image](https://github.com/user-attachments/assets/80c7a7db-2c0e-4409-8d76-278c526bedad)<br><br>



