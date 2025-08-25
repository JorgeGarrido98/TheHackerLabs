# 🔐 Resolución del CTF "Decryptor" | TheHackersLabs

**Plataforma:** TheHackersLabs  
**Nombre de la máquina:** Decryptor  
**Dificultad:** Fácil  
**Enfoque:** Cracking de base de datos KeePass, enumeración, escalada manipulando /etc/passwd

---

## 🔍 1. Escaneo de puertos

```bash
nmap -p- --open -sS -sC -sV --min-rate 5000 -n -vvv -Pn 192.168.1.144 -oN escaneo.txt
```

<img width="826" height="527" alt="1-nmap" src="https://github.com/user-attachments/assets/97e36286-1299-4dd1-93d4-1476074b1867" /><br>

Servicios detectados:

- 22 → SSH
- 80 → Apache HTTP Server
- 2121 → FTP (vsftpd)

---

## 🌐 2. Análisis del servicio web

En el puerto 80 se encuentra es el código fuente un mensaje encriptado:

<img width="1166" height="246" alt="2-puerto80-codigofuente" src="https://github.com/user-attachments/assets/5967e25f-e7fd-443f-b7e4-c5fcbe0d31d6" /><br>

Se trata de una cadena cifrada con BrainFuck que con una herramienta en python creada por mi la desencriptamos:

<img width="872" height="62" alt="3-decrypt-bf" src="https://github.com/user-attachments/assets/063d2abd-3bec-4cea-85e2-6a3d658607f1" /><br>

Herramienta -> https://github.com/JorgeGarrido98/bf-decoder.py/blob/main/bf-decoder.py

---

## 📂 3. Enumeración por FTP

Accedemos con el usuario `mario` y la password desencriptada. 

Dentro se encuentra un archivo `.kdbx`, que es una base de datos de KeePass.

<img width="1156" height="238" alt="4-ftp-mario" src="https://github.com/user-attachments/assets/b5e901db-2fc5-41bb-91fd-278ede3227f8" />

---

## 🧠 4. Cracking del archivo KeePass

Usando `john` con un módulo específico, se obtiene la contraseña maestra del archivo KeePass.

<img width="1154" height="146" alt="5-john-crack-keepass" src="https://github.com/user-attachments/assets/6f740424-c84c-4ca6-8a18-670a14c63298" /><br>

La base de datos contiene acceso SSH para el usuario `chiquero`:

<img width="580" height="354" alt="6-bbdd-keepass" src="https://github.com/user-attachments/assets/6cdb6cfb-a62a-4815-84f9-cee4ba044e7c" />

---

## 💻 5. Acceso al sistema como chiquero

```bash
ssh chiquero@192.168.1.144
```

<img width="453" height="127" alt="7-login-ssh-chiquero" src="https://github.com/user-attachments/assets/12d7ade8-0e3c-4907-820f-fdf27630bf40" /><br>

Acceso exitoso con la contraseña obtenida.

---

## 🔍 6. Análisis de privilegios

```bash
sudo -l
```

Se observa que `chiquero` puede ejecutar `chown` con privilegios de root.

---

## 🧨 7. Escalada manipulando /etc/passwd

Lanzo `sudo -l` y detecto que se puede usar el binario `chown` con privilegios de root:

<img width="1155" height="59" alt="9-sudo -l" src="https://github.com/user-attachments/assets/b530eaf1-f1fb-4987-9344-7a4f2d231979" /><br>

<img width="455" height="117" alt="10-gtfobins" src="https://github.com/user-attachments/assets/9e2f6389-9731-476f-af30-a697b0043239" /><br>

1. Doy permisos de escritura en `/etc/passwd`.
2. Se saca el hash de `password1`.

<img width="1154" height="68" alt="11-chown-passwd-sacamos-hash-password1" src="https://github.com/user-attachments/assets/2f2c3fd8-8a05-4006-b4f2-b4f1c2a52bbf" /><br>

3. Se crea un usuario `jgarri` con el hash de `password1`

<img width="1152" height="286" alt="12-change-passwd-user-jgarri-hash-password1" src="https://github.com/user-attachments/assets/dbb67ac3-1ed5-46d0-987b-0110cc63b170" /><br>

4. Se cambia a ese usuario con `su` usando la nueva contraseña.
   
<img width="1159" height="53" alt="13-escalada-root" src="https://github.com/user-attachments/assets/fa46500a-81c1-46f0-97af-f10bc74aee2d" />

---

## ✅ Conclusión

- Enumeración FTP → base de datos KeePass
- Cracking con John → SSH como `chiquero`
- Escalada modificando `/etc/passwd` con `chown` (SUID abuse)

CTF técnico con un vector poco común para escalar privilegios, ideal para pentesters avanzados.

