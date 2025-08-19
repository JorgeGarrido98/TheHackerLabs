# 🛕 Resolución del CTF "Templo" | TheHackersLabs

**Plataforma:** TheHackersLabs  
**Nombre de la máquina:** Templo  
**Dificultad:** Baja 
**Enfoque:** LFI, bypass de validación, ROT13, cracking de ZIP, escalada con LXD

---

## 🔍 1. Escaneo de puertos

```bash
nmap -p- --open -sS -sC -sV --min-rate 5000 -n -vvv -Pn 192. 168.1.137 -oN escaneo.txt
```

**Puertos abiertos:**
- 22 → SSH
- 80 → Apache HTTP

<img width="1120" height="543" alt="1-nmap" src="https://github.com/user-attachments/assets/9bde1cfe-d0c4-4dde-8f08-f45989716b96" />

---

## 🧭 2. Enumeración web

Lanzamos gobuster y encontramos un directorio llamado `wow`:

<img width="579" height="245" alt="2-gobuster-wow" src="https://github.com/user-attachments/assets/44fff3b9-a88c-4747-8258-ad54478f5d6d" /><br>

Entramos y hay un archivo llamado `clue.txt`:

<img width="296" height="115" alt="3-wow-clue txt" src="https://github.com/user-attachments/assets/ccee2082-3db7-468e-8ef8-664ae39d1f59" /><br>

Nos indica que tenemos que ir al directorio `/opt`.

---

## 🛠️ 3. Directorio NAMARI

Dando muchas vueltas a la página web vemos un mensaje que pone: 'NAMARI lo es todo, solo debes probar'.

<img width="1126" height="476" alt="4-puerto80-NAMARI" src="https://github.com/user-attachments/assets/3b8d5698-52cb-4996-b045-af9075b0a85e" /><br>

Entramos dentro de /NAMARI y hay un formulario para cargar un archivo:

<img width="1129" height="532" alt="6-NAMARI-uploads" src="https://github.com/user-attachments/assets/3fc2278d-a6fb-4247-9e76-13edfa99ea39" /><br>

Probamos a cargar un archivo .php con una reverse shell.

Hacemos Fuzzing sobre el directorio NAMARI para ver si hay alguna carpeta donde guarde los archivos cargados:

<img width="605" height="196" alt="5-gobuster-namari" src="https://github.com/user-attachments/assets/ad1cb1c7-8b2f-4ca7-8f93-adae339ceec5" /><br>

Vemos que existe un directorio dentro de NAMARI -> `/uploads`.

Ejecutamos el archivo con la reseverse shell en /uploads pero no hace nada.

---

## 🧨 4. Reverse shell y LFI

Probamos en el formulario a lanzar `/etc/passwd` y nos devuelve el contenido sacando que existe un usuario llamado `rodgar`:

<img width="1126" height="534" alt="7-LFI-passwd" src="https://github.com/user-attachments/assets/d48741d8-478c-4300-88cf-3903dba07bcd" /><br>

Probamos a ejecutar un comando para devolvernos el contenido del index.php codificado en base64 para poder descifrarlo y ver el funcionamiento de la subida de archivos:

<img width="1118" height="463" alt="10-curl-codigofuente-index php" src="https://github.com/user-attachments/assets/6b7a8338-f252-4414-8415-88ec0e5de20e" /><br>

Lo desciframos:

<img width="1116" height="578" alt="11-decode-index php-codificado-nombrearchivo" src="https://github.com/user-attachments/assets/00bc7178-7479-48b3-9eb1-aa847254327f" /><br>

Vemos que el nombre que tenía nuestro archivo lo encriptan en ROT13 pero mantienen la extensión, probamos a encriptar el nombre de nuestro archivo (revshell):

<img width="267" height="24" alt="12-encrypt-revshell-rot13" src="https://github.com/user-attachments/assets/e84ebdf5-f6a7-4a3a-8310-acda5927851b" /><br>

Ejecutamos nuestra reverse shell con el nombre `erifuryy.php`:

<img width="351" height="32" alt="13-ejecutamos-revshell" src="https://github.com/user-attachments/assets/3595652c-3959-49d9-bdca-76feafa1e3df" /><br>

<img width="692" height="116" alt="14-acceso-inicial" src="https://github.com/user-attachments/assets/6a75b442-da2e-4b85-81fd-c81d74373c29" />

Accedemos a la máquina!

## 📦 5. ZIP protegido

En `/opt/` se encuentra `backup.zip` que solicita contraseña.

<img width="300" height="104" alt="15-opt-backup zip" src="https://github.com/user-attachments/assets/f46b8b94-ef1b-496f-bc26-3cfc582f2bb9" /><br>

Lo sacamos a nuestra máquina Kali abriendo el puerto 8080 en la máquina víctima:

```bash
python3 -m http.server 8080
```

Desde nuestra máquina kali:

```bash
wget http://192.168.1.137/backup.zip
```

Se crackea con `john`:

<img width="551" height="141" alt="17-cracking-john" src="https://github.com/user-attachments/assets/4cd501f3-a796-492c-bd96-244b8eebb287" /><br>

<img width="356" height="164" alt="18-password-rodgar" src="https://github.com/user-attachments/assets/d517b11e-ac43-4763-a78b-4c618cc3687e" /><br>

Entramos como rodgar:

<img width="241" height="61" alt="19-login-rodgar" src="https://github.com/user-attachments/assets/8efbca66-3235-46d9-952b-add09aa3c082" />

---

## 📈 7. Enumeración con linpeas

Dando muchas vueltas y sin sacar nada probando `sudo -l`, Binarios SUID y Cron jobs, decido bajarme linpeas.sh y ejecutarlo:

- LinPEAS muestra que el sistema tiene instalado **LXD versión 5.21.3 LTS**. y que el usuario rodgar tiene permisos de sudo

<img width="553" height="248" alt="20-linpeas" src="https://github.com/user-attachments/assets/3cf18a23-205c-41a2-838c-6282f7e684b7" />

---

## 🚀 8. Escalada a root con LXD

Se utiliza la técnica clásica de escalada con contenedor:

1. Importar una imagen Alpine (`alpine/3.18`) en local.
2. Crear contenedor con volumen montado a `/root` del sistema.
3. Obtener shell root al acceder desde el contenedor al host.

<img width="955" height="296" alt="22-iniciando-lxd" src="https://github.com/user-attachments/assets/299a778d-0833-4a2e-a0dc-56e8b43601d4" /><br>

<img width="976" height="82" alt="23-escalada-lxd" src="https://github.com/user-attachments/assets/3a99bbb9-0f5f-4533-acaf-982d6b3fe073" />

---

## 🏁 9. Conclusión

- 🗂️ LFI con bypass por codificación
- 🧠 ROT13 como técnica para ocultar nombre de archivos
- 🔐 Crackeo de ZIP protegido
- ⚙️ Escalada con LXD

Un CTF muy completo, ideal para practicar técnicas combinadas de explotación web y post-explotación en Linux.
