# Union

**Sistema Operativo**: Linux

**Dificultad**: Media

**Técnicas Vistas**: SQLI (SQL Injection) - UNION Injection
SQLI - Read Files
HTTP Header Command Injection - X-FORWARDED-FOR [RCE]
Abusing sudoers privilege [Privilege Escalation]

Te prepara para las **Certificaciones**: eWPT eJPT<br>

![union](https://github.com/user-attachments/assets/35840fb3-9cda-4654-8869-4220dd6a23e1)<br><br>


# Reconocimiento

Utilizando `nmap` escaneamos todos los puertos<br>

![image](https://github.com/user-attachments/assets/8a084d9a-a5f6-457d-8094-2be11e983463)<br>

- `-p-` -> Escanea todos los puertos 
- `-sS` -> escaneo de tipo SYN
- `--open` -> Todos los puertos abiertos
- `--min-rate=5000` -> Manda mínimo 5000 paquetes por segundo
- `-Pn` -> Asume que el host especificado está activo
 - `-vvv` -> Aplica triple Verbose 
- `-n` -> Que no aplique resolución DNS
- `-oG allports` -> manda todo a un archivo con nombre de "allports"<br><br>


Utilizando `gobuster` encontramos directorios<br>

![image](https://github.com/user-attachments/assets/d357456a-49ae-433f-954d-3cac10f6f789)<br>

 - `dir` ->  Buscar directorios 
- `-u` -> Indicar URL
- `-w` ->  Indicar diccionario que voy a utilizar
- `-t` -> Numero de hilos que voy a usar
- `-x` -> Extensiones que quiero que pruebe<br><br>

# SQLI

Vulnerable SQLI.<br>

![image](https://github.com/user-attachments/assets/858cfdfa-57f4-458b-926e-263f2c2da5e6)<br><br>

Encontramos dos tablas<br>
![image](https://github.com/user-attachments/assets/ee5d402c-f300-462f-a7e2-4435e1b350c9)<br><br>


Encontramos una Flag en la base de datos **november** y la tabla **flag**<br>
![image](https://github.com/user-attachments/assets/63ebc394-c5b4-42be-8749-c4a8f455e52d)<br><br>

Nos abren el puerto SSH<br>
![image](https://github.com/user-attachments/assets/8af3f7ea-0404-4e4b-b7cc-8e7e6a29b942)<br><br>


Leemos el archivo `/var/www/html/config.php` donde encontramos la contraseña de uhc.<br>
![image](https://github.com/user-attachments/assets/2e0ccc75-e734-495e-b9a5-126ebd47b1d0)<br><br>


# Escalada de privilegios

Nos conectamos por ssh al usuario `uhc`<br>
![image](https://github.com/user-attachments/assets/1495ba29-b3ac-4ecb-893e-df45e2ecb896)<br><br>

Flag uno:<br>
![image](https://github.com/user-attachments/assets/0918d6cf-cb8d-4e1a-83b9-598019aa53bf)<br><br>


Vemos la Cabezera `X-FORWARDED-FOR` vulnerable.<br>
![image](https://github.com/user-attachments/assets/fb9672c0-8d88-4b8a-a6c0-4af2e4815760)<br><br>

El servidor valida si la cabezera `X-FORWARDED-FOR` existe y tiene contenido.
En la variable `ip` almacena lo que en esa variable hallamos puesto.
Ejemplo ` X-FORWARDED-FOR: 1.1.1.1  ` 
Luego a esa IP:

le hace `system("sudo /usr/sbin/iptables -A INPUT -s" . $ip . " -j ACCEPT"); ` -> y esto es un comando de linux, el cual nosotros podemos usar `curl` para definir la cabezera y el comando a ejecutar.<br><br>



Con esta query podemos ver si tenemos privilegios.<br>
![image](https://github.com/user-attachments/assets/bd424270-2a53-406e-9e7d-60e60463d8ab)<br><br>


Vemos podemos que podemos ejecutar cualquier comando como root.<br>
![image](https://github.com/user-attachments/assets/57f0480f-b88d-45e7-ab03-cbc2f7f5cf41)<br><br>


Le indicamos privilegios SUID a la `/bin/bash`<br>
![image](https://github.com/user-attachments/assets/9e7ebedc-f842-446c-89d2-2f31a7c8fee9)<br>

- `bash -p` -> Somos root<br><br>

Flag final:<br>
![image](https://github.com/user-attachments/assets/4d044c72-1109-4544-9f3f-b3264483e1ac)<br><br>
