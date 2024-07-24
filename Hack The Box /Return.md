# Return

**Sistema Operativo**: Windows<br>

**Dificultad**: Facil<br>

**Técnicas Vistas**: Abusing Printer /
Abusing Server Operators Group /
Service Configuration Manipulation<br>

Te prepara para las **Certificaciones**: eJPT /
OSCP (Escalada)<br>

![image](https://github.com/user-attachments/assets/4b594586-1c7d-451e-b143-9ab8e16d73be)<br><br>


# Reconocimiento

Utilizando `nmap` descubrimos los puertos abiertos<br>

![image](https://github.com/user-attachments/assets/b8ddc2f4-6ed8-4f19-807f-c181058e1905)<br>

- `-p-` -> Escanea todos los puertos 
- `-sS` -> escaneo de tipo SYN
- `--open` -> Todos los puertos abiertos
- `--min-rate=5000` -> Manda mínimo 5000 paquetes por segundo
- `-Pn` -> Asume que el host especificado está activo
- `-vvv` -> Aplica triple Verbose 
-  `-n` -> Que no aplique resolución DNS
- `-oG allports` -> manda todo a un archivo con nombre de "allports"<br><br>

Nos podemos mandar la peticion a nuestra IP, ponte en escucha con `nc -nlvp 389`<br>
![image](https://github.com/user-attachments/assets/4d6cb516-a6c7-4f2d-83a7-9e7892f309df)<br><br>

Nos da la contraseña de `svc-printer`<br>
![image](https://github.com/user-attachments/assets/e3db947c-5386-4e93-9a7a-f1db4e748b27)<br><br>

# Explotación

Las credenciales son validas<br>
![image](https://github.com/user-attachments/assets/838df5bd-cd20-491a-ab44-0344a6f03e9d)<br>

- `-u` -> Usuario
- `-p` -> Contraseña<br><br>

Esta en el grupo `Remote Management Users`, nos podemos conectar al `winrm` -> Ya que el puerto 5985 esta abierto<br>
![image](https://github.com/user-attachments/assets/d6ad7790-9fd5-452b-8792-2770885f9e1c)<br><br>


Nos conectamos utilizando `evil-winrm`<br>
![image](https://github.com/user-attachments/assets/96e33822-352a-47ef-ace7-4a2b9e06ea73)<br><br>


# Escalada de privilegios

Vemos el grupo `Server Operators`<br>
![image](https://github.com/user-attachments/assets/0dc65d07-2f13-426f-8b82-5f0106bdbe47)<br><br>


Vemos los servicios:<br>
![image](https://github.com/user-attachments/assets/c3677f1e-7f28-40a6-be89-a297a5241989)<br><br>


Me traigo el nc.exe a mi directorio actual.<br>
![image](https://github.com/user-attachments/assets/af492af1-d547-4612-a950-a928e160d421)<br><br>


Me lo descargo en el Windows<br>
![image](https://github.com/user-attachments/assets/5771b524-ba42-41d9-afbc-422e97f127d2)<br><br>



- Cambiamos la configuración de un servicio expuesto, que nos abra el netcat y nos envie una revershell a nuestra ip y puerto.
- Luego paramos el servicios, y lo volvemos a encender **Siempre escuchado por el puerto 443 en nuestra maquina de atacante**.<br>
![image](https://github.com/user-attachments/assets/cec2fea5-3067-4dc5-a81b-c9185d42d4f0)<br><br>

Flag Final:<br>
![image](https://github.com/user-attachments/assets/0b83d98a-65b4-481d-a196-b4991d5208f6)<br><br>

