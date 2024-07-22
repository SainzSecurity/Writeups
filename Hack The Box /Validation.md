# Validation

**Sistema Operativo**: Linux

**Dificultad**: Facil

**Técnicas Vistas**: SQLI (Error Based) /
SQLI -> RCE (INTO OUTFILE) /
Information Leakage 

Te prepara para las **Certificaciones**: eJPT /
eWPT<br>


![Validation-1024x775](https://github.com/user-attachments/assets/eeb1f235-07db-438f-9cf3-d9bec7d44d7d)<br><br>


# Reconocimiento

Utilizando `nmap` descubrimos puertos abiertos<br>

![image](https://github.com/user-attachments/assets/e6b39bbd-1bf8-4019-aedc-72b1c2f82758)<br>

- `-p-` -> Escanea todos los puertos 
- `-sS` -> escaneo de tipo SYN
- `--open` -> Todos los puertos abiertos
- `--min-rate=5000` -> Manda mínimo 5000 paquetes por segundo
- `-Pn` -> Asume que el host especificado está activo
- `-vvv` -> Aplica triple Verbose 
- `-n` -> Que no aplique resolución DNS
- `-oG allports` -> manda todo a un archivo con nombre de "allports"<br><br>


# SQLI

Con esta query creamos e insertamos un `probando.php` en `/var/www/html`<br>

![image](https://github.com/user-attachments/assets/4c84d539-2071-4f7a-97d6-1e4f8c594f0d)<br><br>

La cual usamos para ganar acceso a la maquina<br>

![image](https://github.com/user-attachments/assets/f3bf9a0c-ff38-4ce7-bc84-16349312d89b)<br><br>


# Escalar privilegios

En `config.php` nos muestran el usuario y contraseña de la base de datos.<br>

![image](https://github.com/user-attachments/assets/287fa9b2-b292-47b3-b8a7-1aaaa436d027)<br><br>

Probas esa contraseña para el usuario root:<br>

![image](https://github.com/user-attachments/assets/48f95afa-f1f7-461e-ac31-2b4e3891ddfe)<br><br>


