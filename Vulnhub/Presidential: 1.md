# Presidential: 1

Sistema Operativo: Linux

Dificultad: Media

Técnicas Vistas: Web Enumeration /
Information Leakage /
Virtual Hosting /
Subdomain Enumeration /
Abusing phpMyAdmin - LFI to RCE (abusing PHP ID sessions)
Cracking Hashes (User Pivoting) /
Abusing Capabilities (tar cap_dac_read_search+ep) [Privilege Escalation]

Te prepara para las Certificaciones: eWPT OSWE eWPTXv2 OSCP



![image](https://github.com/user-attachments/assets/8fae9c5e-6c42-408b-88e2-787910e75ed4)

<br>

# Reconocimiento

Aplicamos un barrido en la red local utilizando `arp-scan` para encontrar la IP de la maquina victima:



![image](https://github.com/user-attachments/assets/d4cb276c-d2a5-44bc-9c40-b776547074a4)
- `-I eth0` -> Mi interfaz grafica.
- `--localnet` -> Red local.
- `--ignoredups` -> Que ignore duplicados.



Utilizando `nmap` vemos los puertos abiertos.



![image](https://github.com/user-attachments/assets/7c0af8a0-5b9b-4c2c-be1c-cca46a1b7660)



- `-p-` -> Escanea todos los puertos
- `-sS` -> escaneo de tipo SYN
- `--open` -> Todos los puertos abiertos
- `--min-rate=5000` -> Manda mínimo 5000 paquetes por segundo
- `-Pn` -> Asume que el host especificado está activo
- `-vvv` -> Aplica triple Verbose
- `-n` -> Que no aplique resolución DNS
- `-oG allports` -> manda todo a un archivo con nombre de "allports"


Utilizando `nmap` lanzamos los scripts mas comunes.



![image](https://github.com/user-attachments/assets/9e298361-6368-47df-8590-c51330e18fdf)



- `OpenSSH 7.4` -> Versión SSH desactualizada
- `PHP 5.5.38` -> Versión PHP desactualizada

Utilizando `gobuster` encontramos varios directorios.



![image](https://github.com/user-attachments/assets/335036c9-c96a-465a-9c36-48cfeb5c2ff7)



- `dir` -> Buscar directorios
- `-u` -> Indicar URL
- `-w` -> Indicar diccionario que voy a utilizar
- `-t` -> Numero de hilos que voy a usar
- `-x` -> Extensiones que quiero que pruebe

En `config.php.bak` encontramos credenciales para una base de datos



![image](https://github.com/user-attachments/assets/39c5c121-5b26-4ecf-9597-6d8aedaac7bf)

Utilizando `gobuster` encontramos un subdominio "datasafe".



![image](https://github.com/user-attachments/assets/b61e30d8-5968-4096-b19f-6f444876a6bc)

Agregamos el dominio y subdominio en `/etc/hosts`



![image](https://github.com/user-attachments/assets/ca70c92a-4ae3-41d0-a23a-e7b41b3eaaff)



Entramos con las credenciales del `config.php.bak` que encontramos anteriormente.



![image](https://github.com/user-attachments/assets/73a251b2-4b27-4757-9ee7-db8aef0d9b6e)

------

# Explotacion PHPMYADMIN



### LFI Y RCE

Utilizando `searchsploit` vemos que el phpmyadmin 4.8.1 es vulnerable a un RCE:



![image](https://github.com/user-attachments/assets/0c022638-cf01-450b-9d9d-50ee44692a11)



En el código también muestra como explotar un LFI:



![image](https://github.com/user-attachments/assets/0f4296bf-36ce-4d48-9be0-11ab673b5c8d)



Explotación de RCE:



![image](https://github.com/user-attachments/assets/645921ad-39ec-4774-a6b0-a9d66b9012ce)



Para que el RCE funcione deben sacarle la "s" a "session" en el script.



![image](https://github.com/user-attachments/assets/dfb50f34-e097-427d-bd12-5d87a3b3b05e)


----

# Ataque de fuerza bruta

Encontramos un usuario `admin` con una contraseña encriptada, vamos a explotarla con el diccionario Rockyou



![image](https://github.com/user-attachments/assets/de612644-43cb-419b-a472-13c4f6c84790)


Utilizando `john` hacemos un ataque de fuerza bruta con el diccionario `Rockyou` a la contraseña de `admin`



![image](https://github.com/user-attachments/assets/6a0e7dfd-db38-42c8-bb52-e8ab7fda074d)


----

# Escalamos privilegios


Utilizamos `getcap` para encontrar capabilities.
`cap_dac_read_search+ep` -> Nos permite leer cualquier archivo del sistema, con tarS que funciona como tar.



![image](https://github.com/user-attachments/assets/046d89a8-49c2-4f03-b89a-09e901eb3e5e)



cd /tmp/ -> Nos dirigimos al directorio tmp.

tar -cv id_rsa.tar /root/.ssh/id_rsa -> Nos copiamos su clave privada

La creamos "id_rsa" y pegamos en nuestro directorio ssh



![image](https://github.com/user-attachments/assets/3a509cb0-46c3-427b-b7ef-45e1d6e2710e)



ssh -i id_rsa root@192.168.1.104 -p 2082 -> Nos conectamos

Flag Final:



![image](https://github.com/user-attachments/assets/e132d25d-0178-480d-b4c5-1fec26aab8d9)


