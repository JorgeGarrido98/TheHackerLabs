# 🔐 Resolución del CTF "Decryptor" | The HackersLabs

**Plataforma:** TheHackersLabs  
**Nombre de la máquina:** Decryptor  
**Dificultad:** Media  
**Enfoque:** Cracking de base de datos KeePass, enumeración, escalada manipulando /etc/passwd

---

## 🔍 1. Escaneo de puertos

```bash
nmap -p- -sS -sC -sV -Pn 192.168.1.127
```

Servicios detectados:

- 21 → FTP (vsftpd)
- 22 → SSH
- 80 → Apache HTTP Server

---

## 🌐 2. Análisis del servicio web

En el puerto 80 se encuentra un archivo que contiene un código ofuscado. Se trata de una cadena cifrada con Blowfish.  
La clave se descubre más adelante a través del FTP.

---

## 📂 3. Enumeración por FTP

Acceso permitido para el usuario `mario`. Dentro se encuentra un archivo `.kdbx`, que es una base de datos de KeePass.

---

## 🧠 4. Cracking del archivo KeePass

Usando `john` con un módulo específico, se obtiene la contraseña maestra del archivo KeePass.

La base de datos contiene acceso SSH para el usuario `chiquero`.

---

## 💻 5. Acceso al sistema como chiquero

```bash
ssh chiquero@192.168.1.127
```

Acceso exitoso con la contraseña obtenida.

---

## 🔍 6. Análisis de privilegios

```bash
sudo -l
```

Se observa que `chiquero` puede ejecutar `chown` con privilegios de root.

---

## 🧨 7. Escalada manipulando /etc/passwd

1. Se hace una copia de `/etc/passwd`.
2. Se reemplaza el hash de un usuario por uno creado con una contraseña conocida.
3. Se cambia la propiedad de `/etc/passwd` para poder sobrescribirlo:

```bash
sudo chown chiquero /etc/passwd
```

4. Se sobrescribe `/etc/passwd` con la versión modificada.
5. Se cambia a ese usuario con `su` usando la nueva contraseña.

---

## 🏁 8. Acceso como root

Una vez dentro, se puede usar `sudo su` u otro vector para obtener acceso root.

---

## ✅ Conclusión

- Enumeración FTP → base de datos KeePass
- Cracking con John → SSH como `chiquero`
- Escalada modificando `/etc/passwd` con `chown` (SUID abuse)

CTF técnico con un vector poco común para escalar privilegios, ideal para pentesters avanzados.

