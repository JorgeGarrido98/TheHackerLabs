# 🍔 Resolución del CTF "Goiko" | The HackersLabs

**Plataforma:** TheHackersLabs  
**Nombre de la máquina:** Goiko  
**Dificultad:** Media  
**Enfoque:** SMB enumeration, cracking ZIP e ID_RSA, acceso a MariaDB, escalada con cronjob

---

## 🔍 1. Escaneo de puertos

```bash
nmap -p- -sS -sC -sV -Pn 192.168.1.127
```

Puertos abiertos:

- 22 → SSH
- 139, 445 → SMB
- 3306 → MariaDB
- 8080 → Servicio web

---

## 📂 2. Enumeración por SMB

Con `smbclient` se accede a múltiples comparticiones:

- `/food`
- `/dessert`
- `/cafe`

Archivos descargados:

- `creds.txt` con nombres como `cafesinleche`, `burgerwithoutcheese`, etc.
- Archivo ZIP `burgerwithoutcheese.zip`

---

## 🔐 3. ZIP protegido y cracking

El ZIP `burgerwithoutcheese.zip` contiene una clave privada (`id_rsa`) cifrada.

1. Se crackea la contraseña del ZIP con `john`.
2. Se extrae `id_rsa`.
3. El `id_rsa` está protegido con passphrase → también se crackea con `john`.

---

## 🔑 4. Acceso SSH como `gurpreet`

Con la clave privada y passphrase:

```bash
ssh -i id_rsa gurpreet@192.168.1.127
```

---

## 🧾 5. Nota interesante

En el home de gurpreet se encuentra una nota que menciona posibles credenciales para MariaDB.

---

## 🛢️ 6. Acceso a MariaDB

Accedemos con las credenciales encontradas y listamos hashes de usuarios.  
Se crackea el hash de `nika`.

---

## 🧠 7. Escalada a `nika` y privilegios

Con la contraseña de `nika` accedemos o escalamos localmente.  
Revisando con `sudo -l`, se identifican permisos interesantes o cronjobs automatizados.

---

## 🧨 8. Escalada a root

Se detecta un script malicioso que se ejecuta como root desde `/opt/porno`.  
Se aprovecha para inyectar comandos y conseguir shell como root.

---

## ✅ Conclusión

1. Enumeración por SMB
2. Cracking de ZIP e ID_RSA protegida
3. Acceso como `gurpreet`, luego `nika`
4. Enumeración en MariaDB
5. Escalada con cronjob malicioso

Un reto bien diseñado para practicar enumeración de servicios de red, cracking, pivoting entre usuarios y escalada local en Linux.

