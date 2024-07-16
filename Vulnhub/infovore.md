# Infovore

**Sistema Operativo**: Linux

**Dificultad**: Media

**Técnicas Vistas**: Web Enumeration /
LFI (Local File Inclusion) /
Abusing file_uploads visible in info.php (LFI2RCE via phpinfo() + Race Condition) /
Cracking Protected Private SSH Key /
Abusing ssh key pair trust to escape the container /
Abusing docker group [Privilege Escalation]

Te prepara para las **Certificaciones**: eWPT eWPTXv2 OSWE OSCP


![image](https://github.com/user-attachments/assets/27dd21bb-aec2-4791-9985-2d49739042ab)
# <br>

# Reconocimiento


Aplicamos un barrido en la red local utilizando `nmap` para encontrar la IP de la maquina victima:

![image](https://github.com/user-attachments/assets/edf64c8d-e57d-4682-82fc-5d56922f7eb5)
# <br>

Utilizando `nmap` vemos los puertos abiertos.

![image](https://github.com/user-attachments/assets/dd5f0d0d-33a5-439a-a45f-f6e90d04d86e)

- `-p-` -> Escanea todos los puertos 
- `-sS` -> escaneo de tipo SYN
- `--open` -> Todos los puertos abiertos
- `--min-rate=5000` -> Manda mínimo 5000 paquetes por segundo
- `-Pn` -> Asume que el host especificado está activo
- `-vvv` -> Aplica triple Verbose 
- `-n` -> Que no aplique resolución DNS
- `-oG allports` -> manda todo a un archivo con nombre de "allports"

# <br>

Utilizando `nmap` lanzamos los scripts mas comunes.
![image](https://github.com/user-attachments/assets/d4669c55-1276-48bc-9670-bed983515298)

# <br>

Utilizamos `gobuster` para encontrar el directorio "/info.php"

![image](https://github.com/user-attachments/assets/58b93d5b-ab7d-4470-b7ed-3aeebe64fdcc)

- `dir` ->  Buscar directorios 
- `-u` -> Indicar URL
- `-w` ->  Indicar diccionario que voy a utilizar
- `-t` -> Numero de hilos que voy a usar
- `-x` -> Extensiones que quiero que pruebe

# <br>

Podemos subir archivos, ya que `file_uploads` esta habilitado

![image](https://github.com/user-attachments/assets/9bcc3c64-7849-4b72-91fc-80945abe9696)


# Explotación File_uploads

### LFI, RACE CONDITION Y RCE

En la ruta "/info.php" hacemos la prueba de subir un archivo y vemos que lo sube, pero lo borra inmediatamente.
Hay que abusar de una **Condición de Carerra**
![image](https://github.com/user-attachments/assets/3b64ae17-9e3e-49ad-a48a-8c2df7e7ebf8)

# <br>

Utilizando `wfuzz` encontramos donde subir el archivo:
![image](https://github.com/user-attachments/assets/2775485b-3e50-4c61-ba72-dbd72edfb94c)
- `-c` -> Para que se vea en colores
- `--hl=136`  -> Para que me oculte los resultados de 136 líneas
- `-t 200` -> Para usar 200 hilos
- `-w` -> Diccionario que voy a usar.
- `-u` -> Especificamos URL
- `FUZZ` -> Donde quiero aplicar la fuerza bruta

<br>

Aplicamos el **LFI**



![image](https://github.com/user-attachments/assets/950c9647-9a2d-4faa-baab-496fade01f07)


<br>
Aplicamos la **Race Condition** junto con el **RCE**

hackstricks
[https://www.insomniasec.com/downloads/publications/phpinfolfi.py](https://www.insomniasec.com/downloads/publications/phpinfolfi.py)


Estos son algunos cambios que tenemos que aplicar en el Script.



![image](https://github.com/user-attachments/assets/d90e9f45-6cf0-49ef-9856-996da3e9a5b1)
![image](https://github.com/user-attachments/assets/2fae5737-21b0-422f-972f-63853ffd52f9)<br><br>



Ganamos acceso a la maquina<br>
![image](https://github.com/user-attachments/assets/30462227-87d5-47fb-acd0-d8a296700f64)<br>



# Escalada de privilegios

Tiene un archivo escondido en la raiz<br>
![image](https://github.com/user-attachments/assets/1cc36bc4-e33e-4ade-8102-e71b2cedfc0a)<br>
- `cp .oldkeys.tgz /tmp/`
- `tar -xf .oldkeys.tgz`


Es una clave privada pero encriptada<br>
![image](https://github.com/user-attachments/assets/b56d411e-ea22-4d0d-9688-ac958cfa5da9)<br>

Vemos que estamos  en un contenedor<br>
![image](https://github.com/user-attachments/assets/4b148222-708d-48ac-9071-d31353b9ac4e)<br>

Vemos que la maquina host tiene el puerto ssh abierto.<br>
![image](https://github.com/user-attachments/assets/65e5e9b4-f3f8-406e-9df7-dc11c685427e)<br>


- `vim id_rsa` -> Pegamos la contraseña ssh
- `python2.7 /usr/share/john/ssh2john.py id_rsa > hash` -> Nos devuelve un hash
- `john -w:/usr/share/wordlists/rockyou.txt hash` -> Rompe la contraseña

Tenemos la contraseña privada de ssh<br>
![image](https://github.com/user-attachments/assets/1b777447-a056-446e-9b6f-84dfc0d2b16f)<br>

su root
choclate93

ssh admin@192.168.150.1 
<br><br>


# Escalada de privilegios Docker

Creamos una imagen de docker, luego le cargamos toda la raiz al directorio "/mnt/root", entramos al contenedor y nos dirigimos al "/mnt/root" donde le cargamos toda la raiz.<br>
![image](https://github.com/user-attachments/assets/5bd2c0ff-9f1b-486c-961f-a439a7423fc1)<br>

- cd bin
- chmod u+s bash -> Da permisos SUID, no solo en el contenedor.
- exit


Flag Final:<br><br>
![image](https://github.com/user-attachments/assets/74648ad7-1f3a-49a6-b7c3-dbff19660067)
