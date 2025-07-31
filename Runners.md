# 🏃 Resolución del CTF "Runners" | The HackersLabs

**Plataforma:** TheHackersLabs  
**Nombre de la máquina:** Runners  
**Dificultad:** Media/Alta  
**Enfoque:** SQLi, exfiltración de archivos, cracking, uso de pspy, escalada con Docker

---

## 🔍 1. Escaneo de puertos

```bash
nmap -p- -sS -sC -sV --min-rate 2000 -n -vvv -Pn 192.168.1.127
```

Se detectan:

- Puerto 22 → OpenSSH
- Puerto 80 → Apache con sitio web activo

---

## 🌐 2. Enumeración web

Con `gobuster` se encuentra `/post.php?id=1`, lo que sugiere posible SQLi:

```bash
gobuster dir -u http://192.168.1.127 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

---

## 💥 3. Explotación SQLi con sqlmap

Confirmamos vulnerabilidad:

```bash
sqlmap -u "http://192.168.1.127/post.php?id=1" --dbs
```

Se descubre la base de datos `blog`, y dentro de ella la tabla `users` con usuarios y hashes.

---

## 🔓 4. Cracking de hash

Extraemos el hash y lo crackeamos:

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

🔑 Contraseña obtenida: `maradona`

---

## 💻 5. Acceso SSH

```bash
ssh runner@192.168.1.127
# Contraseña: maradona
```

Se obtiene acceso como usuario `runner`.

---

## 📂 6. Revisión del sistema

Explorando el sistema encontramos `/var/www/html/credenciales.php`. Lo copiamos con `scp` y extraemos un hash de contraseña ofuscado.

Este hash es crackeado nuevamente con `john`, y accedemos a un fichero `.xlsx` con más credenciales.

---

## 🔑 7. Análisis de credenciales.xlsx

El Excel contiene credenciales de múltiples usuarios. Se accede a otro usuario `ian` con credenciales obtenidas.

---

## 📈 8. Enumeración con pspy

Se observa que un script `backup.sh` se ejecuta periódicamente por un usuario con más privilegios. Se modifica el script para inyectar una reverse shell.

---

## 🚪 9. Acceso privilegiado y nueva shell

Escalada al usuario que ejecuta el script desde el puerto `2222`. Revisando los archivos encontramos la flag de usuario.

---

## 🛠️ 10. Escalada a root vía Docker

Con `id` y búsqueda de binarios SUID, se identifica que el usuario pertenece al grupo `docker`.

Se lanza un contenedor Docker interactivo montando el sistema y accediendo como root:

```bash
docker run -v /:/mnt --rm -it ubuntu chroot /mnt
```

---

## 🏁 11. Flag final

Una vez dentro del contenedor, accedemos a `/root/root.txt` y conseguimos la flag.

---

## ✅ Conclusión

Pasos clave:

1. SQLi → enumeración de base de datos
2. Cracking de hashes → acceso SSH
3. Revisión de ficheros y extracción de credenciales
4. Uso de pspy → escalada mediante script vulnerable
5. Acceso como root vía Docker

Un reto técnico y realista, ideal para poner a prueba tus habilidades de post-explotación.

