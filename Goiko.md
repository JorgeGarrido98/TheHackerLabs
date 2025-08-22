# 🍔 Resolución del CTF "Goiko" | TheHackersLabs

**Plataforma:** TheHackersLabs  
**Nombre de la máquina:** Goiko  
**Dificultad:** Media  
**Enfoque:** SMB enumeration, cracking ZIP e ID_RSA, acceso a MariaDB, escalada con Script que se ejecuta como `root`

---

## 🔍 1. Escaneo de puertos

```bash
nmap -p- -- open -sS -sC -sV -- min-rate 5000 -n -vvv -Pn 192. 168.1.143 -oN escaneo. txt
```

Puertos abiertos:

- 22 → SSH
- 139, 445 → SMB
- 10021 → FTP

---

## 📂 2. Enumeración por SMB

Con `smbclient` se accede a múltiples comparticiones:

<img width="1300" height="286" alt="2-smbclient" src="https://github.com/user-attachments/assets/7e7e6db9-1626-40f2-a206-0258692f72b2" /><br>

- `/food`
- `/dessert`
- `/cafe`

Hay varios archivos pero el importante es `.cafesinleche`:

<img width="412" height="132" alt="8-cafesinleche" src="https://github.com/user-attachments/assets/4e9893ee-9097-4b32-8ab6-31b0cf3ffc87" /><br>

---

## 🔑 3. Acceso SSH como `marmai`

Busco archivos .txt y .zip y encuentro un zip llamado `burgerwithoutcheese.zip`, lo descargo en mi kali porque está protegido:

<img width="710" height="388" alt="10-burgerwithoutcheese zip" src="https://github.com/user-attachments/assets/85cc2331-0f2e-46c6-b30c-adad4669bcf2" /><br>

<img width="1042" height="278" alt="11-zip-pide-password" src="https://github.com/user-attachments/assets/dcbc9f16-bf6d-4a14-9bf1-963b91f31427" />

---

## 🔐 4. ZIP protegido y cracking

El ZIP `burgerwithoutcheese.zip` pide contraseña:

1. Se crackea la contraseña del ZIP con `john`:

<img width="1102" height="348" alt="12-crackeo-password-john" src="https://github.com/user-attachments/assets/ea38071c-508e-46d4-a5fb-b07a946f7f08" /><br>

<img width="832" height="348" alt="13-extraccion-zip" src="https://github.com/user-attachments/assets/aff0b31d-3f5b-4f35-8c9b-999e49769b08" /><br>

2. Se extrae `id_rsa` y `users`:

<img width="848" height="1028" alt="14-cat-id_rsa-users" src="https://github.com/user-attachments/assets/edd8fe35-7309-4328-9325-091a50e1ef24" /><br>

<img width="130" height="95" alt="image" src="https://github.com/user-attachments/assets/f10e508e-c590-40b3-918d-848b4a620613" /><br>

3. El `id_rsa` está protegido con passphrase → también se crackea con `john`:

<img width="354" height="60" alt="15-id_rsa-passphrase" src="https://github.com/user-attachments/assets/dd47a580-6c79-4c6f-9d16-bf3a61182e64" /><br>

<img width="940" height="242" alt="16-crackeo-john-passphrase-id_rsa" src="https://github.com/user-attachments/assets/b2842971-e458-4da7-a80d-15e3a888da2a" />

---

## 🔑 5. Acceso SSH como `gurpreet`

Con la clave privada y passphrase se prueba con los usuarios encontrados en el fichero `users`:

```bash
ssh -i id_rsa gurpreet@192.168.1.127
```

<img width="894" height="426" alt="17-ssh-login-gurpreet" src="https://github.com/user-attachments/assets/95f9cd34-3045-4e8f-a640-3813c706e2f6" />

---

## 🧾 6. Nota interesante

En el home de gurpreet se encuentra una nota que menciona posibles credenciales para MariaDB:

<img width="910" height="154" alt="18-cat-nota" src="https://github.com/user-attachments/assets/040cd3d7-4a9f-4af2-b07e-a6397679c663" />

---

## 🛢️ 7. Acceso a MariaDB

Accedemos con las credenciales encontradas de `gurpreet` y listamos hashes de usuarios:

<img width="397" height="238" alt="19-acceso-mariadb" src="https://github.com/user-attachments/assets/55a45654-56ae-45d7-9edb-c991c3d2d1b1" /><br>

<img width="341" height="207" alt="20-hashes-mariadb" src="https://github.com/user-attachments/assets/4aee7455-ea7c-48c2-9e7b-aadc10ea406b" /><br>

Se crackean los hashes encontrados y el de `nika` no se encuentra pero si el de `carline` pero no existe ese usuario y pienso que puede ser para engañar. Pruebo esa password con el usuario `nika` y consigo entrar.

> nika:lucymylove

---

## 🧨 8. Escalada a root

Lanzamos `sudo -l` y veo que se puede ejecutar un script de nika como root:

<img width="586" height="68" alt="22-sudo -l" src="https://github.com/user-attachments/assets/573852c9-0153-4318-b505-1f6daf020db9" /><br>

El script llama a find y chown sin ruta absoluta, así que hacemos un PATH hijack:

<img width="287" height="178" alt="23-find-malicioso-opt-porno" src="https://github.com/user-attachments/assets/6ad4a933-d8de-443a-8620-27853e64e489" /><br>

<img width="385" height="51" alt="24-escalada-root" src="https://github.com/user-attachments/assets/ad873d55-b08c-44a1-bec1-3cc577a2f0fb" />

---

## ✅ Conclusión

1. Enumeración por SMB
2. Cracking de ZIP e ID_RSA protegida
3. Acceso como `gurpreet` y luego `nika`
4. Enumeración en MariaDB
5. Escalada con script con permisos de `root`

Un reto bien diseñado para practicar enumeración de servicios de red, cracking, pivoting entre usuarios y escalada local en Linux.

