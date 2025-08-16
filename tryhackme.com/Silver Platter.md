# Silver Platter

![image.png](./imagenes/SilverPlatter/image.png)

Maquinas que vamos a utilizar Kali Linux y https://tryhackme.com/room/silverplatter

Nuestra IP: `10.21.203.172`

![image.png](./imagenes/SilverPlatter/image%201.png)

IP maquina victima: `10.10.141.31`

![image.png](./imagenes/SilverPlatter/image%202.png)

---

- Hacemos una enumeración de puerto con nmap.
    
    ```bash
    nmap -p- -sS -sV -sC --open -min-rate 5000 -n -vvv -Pn 10.10.141.31
    ```
    
    ![image.png](./imagenes/SilverPlatter/image%203.png)
    

- Vamos a mirar lo que corre por el puerto 80
    
    ![image.png](./imagenes/SilverPlatter/image%204.png)
    
    En la sección Contacto encontramos un el nombre de una aplicacion y un posible nombre de usuario.
    
    ![image.png](./imagenes/SilverPlatter/image%205.png)
    
    Project manager en `Silverpeas`. 
    
    Posible nombre de usuario"`scr1ptkiddy`".
    
- Miramos el código de la web
    
    `CTRL+U`
    
    ![image.png](./imagenes/SilverPlatter/image%206.png)
    
    No se ve nada destacado
    

- Miramos en el puerto 8080
    
    ![image.png](./imagenes/SilverPlatter/image%207.png)
    
    No carga nada
    
- Hacemos Fuzzing Web con Gobuster
    
    ```bash
    gobuster dir -u [http://10.10.141.31/](./imagenes/SilverPlatter/http://10.10.141.31/) -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,sh,html,txt,rm
    ```
    
    ![image.png](./imagenes/SilverPlatter/image%208.png)
    
    No parece que por aquí pueda hacer algo.
    
- Hacemos Fuzzing Web con Gobuster pero sobre el puerto 8080
    
    ```bash
    gobuster dir -u [http://10.10.141.31:8080/](./imagenes/SilverPlatter/http://10.10.141.31:8080/) -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,sh,html,txt,rm
    ```
    
    ![image.png](./imagenes/SilverPlatter/image%209.png)
    
    Encontramos dos directorios pero no parecen accesibles.
    
    ![image.png](./imagenes/SilverPlatter/image%2010.png)
    
    ![image.png](./imagenes/SilverPlatter/image%2011.png)
    
- Me encuentro un poco atacado y prueba con el nombre de Project manager haber si es una carpeta. `Silverpeas`
    
    `http://10.10.141.31:8080/silverpeas/`
    
    ![image.png](./imagenes/SilverPlatter/image%2012.png)
    
    Nos encontramos con formulario de Login 
    
- Busco CVE para  silverpeas y encuentro CVE-2024-36042: Silverpeas authentication bypass
    
    [https://gist.github.com/ChrisPritchard/4b6d5c70d9329ef116266a6c238dcb2d](./imagenes/SilverPlatter/https://gist.github.com/ChrisPritchard/4b6d5c70d9329ef116266a6c238dcb2d)
    
    Explica como soltarse la autentificación
    
    ![image.png](./imagenes/SilverPlatter/image%2013.png)
    

- Iniciamos BurpSuite y capturamos la petición
    
    ![image.png](./imagenes/SilverPlatter/image%2014.png)
    
    ![image.png](./imagenes/SilverPlatter/image%2015.png)
    
    ![image.png](./imagenes/SilverPlatter/image%2016.png)
    
    Eliminamos la parte del password y lanzamos la petición
    
    ![image.png](./imagenes/SilverPlatter/image%2017.png)
    

- Hemos conseguido acceso y vemos que tiene una notificación que vamos a investigar
    
    ![image.png](./imagenes/SilverPlatter/image%2018.png)
    

![image.png](./imagenes/SilverPlatter/image%2019.png)

El mensaje veo que tiene un ID, así que voy a probar a jugar con  el  haber si sale algún mensaje interesante. Copio la URL en una pestaña nueva del navegador para poder editarlo.

![image.png](./imagenes/SilverPlatter/image%2020.png)

Sorpresa en el ID 6 nos encontramos con un usuario y una contraseña de SSH

`Username: tim`

`Password: cm0nt!md0ntf0rg3tth!spa$$w0rdagainlol`

- Vamos a probarlas
    
    ```bash
    ssh [tim@10.10.141.31](./imagenes/SilverPlatter/mailto:tim@10.10.141.31) 
    ```
    
    ![image.png](./imagenes/SilverPlatter/image%2021.png)
    
    `pwd`
    
    `ls`
    
    `cat user.txt`
    
    ![image.png](./imagenes/SilverPlatter/image%2022.png)
    
    Ya tenemos la primera Flag: `THM{c4ca4238a0b923820dcc509a6f75849b}`
    

- Intentamos ver los permisos con sudo -l , pero no tenemos permiso para sudo
    
    ![image.png](./imagenes/SilverPlatter/image%2023.png)
    

- Intentamos leer el **/etc/passwd**
    
    `cat /etc/passwd`
    
    ![image.png](./imagenes/SilverPlatter/image%2024.png)
    
    Encontramos que hay otro usuario llamado `tyler`
    

- Miramos nuestro Id para ver los permisos de nuestro usuario
    
    `id`
    
    ![image.png](./imagenes/SilverPlatter/image%2025.png)
    
    Vemos que estamos en el grupo de adm por tanto tenemos ciertos permisos para ver registro en el sistema.
    
- Vamos a la carpeta  /var/log/ para ver si podemos encontrar alguna contraseña .
    
    `cd /var/log/`
    `ls`
    
    ![image.png](./imagenes/SilverPlatter/image%2026.png)
    

- Hacemos una búsqueda en todos los archivo de la palabra **password**
    
    `grep -iR "Password”`
    
    <aside>
    💡
    
    - **`grep`** es una herramienta para buscar texto en archivos.
    - **`i`** hace que la búsqueda no distinga entre mayúsculas y minúsculas (ignora si es "password", "Password" o "PASSWORD").
    - **`R`** (o **`r`**) indica que la búsqueda es recursiva, es decir, buscará el texto dentro de todos los archivos en el directorio actual y en todos sus subdirectorios.
    - **`'Password'`** es el texto que estás buscando.
    </aside>
    

![image.png](./imagenes/SilverPlatter/image%2027.png)

Usuario: `tyler`

Pass: `_Zd_zx7N823/`

- Cambiamos de usuario:
    
    `su tyler`
    
    ![image.png](./imagenes/SilverPlatter/image%2028.png)
    

- Miramos si podemos escalar privilegios con sudo -l
    
    `sudo -l`
    
    ![image.png](./imagenes/SilverPlatter/image%2029.png)
    
    Como vemos podemos ser root directamente sin contraseña.
    
- Escalamos root
    
    `sudo su`
    
    `whoami`
    
    ![image.png](./imagenes/SilverPlatter/image%2030.png)
    

- Nos vamos a la carpeta root para la segunda flag. que se encuentra en el archivo **root.txt**
    
    `cat root.txt`
    
    ![image.png](./imagenes/SilverPlatter/image%2031.png)
    
    Segunda Flag: `THM{098f6bcd4621d373cade4e832627b4f6}`