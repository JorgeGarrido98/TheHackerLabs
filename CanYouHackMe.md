# 🧠 Resolución del CTF "CanYouHackMe" | TheHackersLabs

**Plataforma:** TheHackersLabs  
**Nombre de la máquina:** CanYouHackMe  
**Dificultad:** Fácil  
**Enfoque:** Nmap, Hydra, Análisis de .bash_history y Docker

---

## 🔍 1. Escaneo de puertos

```bash
nmap -p- -- open -sS -sC -sV -- min-rate 5000 -n -vvV -Pn 172.20.10.2 -oN escaneo. txt
```

<img width="819" height="507" alt="1-nmap" src="https://github.com/user-attachments/assets/50957668-eddf-40d3-876a-51e707425d2f" /><br>

Puertos abiertos:

- 22 → SSH
- 80 → Servicio HTTP

---

## 🔍 2. Enumeración Web

En el puerto 80, se muestra una web con estética Matrix:

<img width="889" height="425" alt="2-puerto80" src="https://github.com/user-attachments/assets/0644ff62-c885-4661-8db2-eec442e04b57" /><br>

Revisando el código fuente del HTML, encontramos un comentario sospechoso:

```javascript
/* Hola juan, te he dejado un correo importante, cuando puedas, leelo */
```

Esto sugiere que:

- Existe un usuario llamado juan.

- Hay un archivo o recurso tipo “correo”.<br>

Sin embargo, no se descubrieron rutas accesibles con gobuster ni recursos adicionales en la red.

---

## 🚪 3. Ataque de fuerza bruta con Hydra

Dado que se sospechaba que existía el usuario juan, se intentó fuerza bruta por SSH:

```bash
hydra -l juan -P /usr/share/wordlists/rockyou.txt ssh://172.20.10.2 -f
```

Credenciales válidas encontradas:  
- Usuario: `juan`  
- Contraseña: `matrix`

---

## 📬 4. Búsqueda del "correo"

Una vez dentro como juan, se exploraron los buzones comunes:

```bash
cd /var/mail
cd /var/spool/mail
```

Ambos estaban vacíos.

Al revisar el .bash_history del usuario:

```bash
cat ~/.bash_history
```

<img width="269" height="152" alt="5-bash_history" src="https://github.com/user-attachments/assets/477d6ebe-3577-431e-8a3d-4f07e217c308" /><br>

Se encontró:

```bash
cat para\ juan.txt
cat para\ root.txt
docker run -it -v /:/mnt alpine
su
```

Esto indica:

- Existieron archivos para juan.txt y para root.txt.

- Se usó Docker con acceso al sistema host.

- Se intentó escalar privilegios con su.

## 🐳 4. Escalada de privilegios con Docker

El historial reveló el uso del siguiente comando:

```bash
docker run -it -v /:/mnt alpine
```

<img width="412" height="296" alt="6-contenedor-docker" src="https://github.com/user-attachments/assets/bc56ef31-7b55-4c5e-81fd-bb0a72f6e241" /><br>

Esto monta todo el sistema del host dentro del contenedor en /mnt, una técnica clásica de escape cuando el usuario tiene acceso sin restricciones a docker.<br>

Se accedió al contenedor, y luego se ejecutó:

```bash
chroot /mnt
```

<img width="353" height="58" alt="7-escalada-root" src="https://github.com/user-attachments/assets/b389b8c1-f465-44a2-b04b-9f0da3c16c94" />

### 🔍 ¿Cómo lo supe?

#### 1. Pistas en el .bash_history

```bash
docker run -it -v /:/mnt alpine
```

Este comando monta todo el sistema anfitrión (/) dentro de /mnt del contenedor. Es decir, estás viendo el sistema real desde dentro del contenedor.

#### 2. Uso típico del chroot

El paso habitual cuando haces eso es:

```bash
chroot /mnt
```

Esto cambia el "root del sistema" al nuevo directorio montado (/mnt) y, si ese sistema tiene un entorno real (como /bin/bash, /etc/passwd, etc.), te conviertes en root sin necesidad de credenciales, porque estás ejecutando comandos como si estuvieras logueado directamente en la máquina como root.

### 💡 Por qué funciona

✅ Requisitos para este tipo de escalada:

- El usuario puede ejecutar docker.

- Puede montar el sistema real dentro de un contenedor (-v /:/mnt).

- El contenedor tiene chroot disponible.

- El sistema anfitrión tiene bash, sh o similar en /mnt/bin.

Cuando haces:

```bash
chroot /mnt
```

Estás diciendo: "quiero que este contenedor ahora se comporte como si su raíz fuera el sistema real, y como estoy dentro del contenedor como `root`, soy root también en el host."

### 🔥 Esta técnica se llama `"Docker Host Breakout via Volume Mount + chroot"`

Es un fallo común de configuración cuando los usuarios tienen acceso a docker sin restricciones, ya que el grupo docker equivale prácticamente a root.<br>

✅ Esto cambia el entorno raíz al del sistema real y, dado que estamos en el contenedor como root, se obtiene acceso como root en el sistema host.

---

## 🛠️ Técnicas utilizadas

- Enumeración de código fuente web

- Fuerza bruta de SSH con Hydra

- Análisis de .bash_history

- Escape de Docker vía volumen montado

- `chroot` para escalada de privilegios

---

## 🏆 Conclusión

Este CTF demuestra cómo un acceso aparentemente limitado a un usuario sin privilegios puede escalar hasta root si el usuario tiene acceso sin restricciones al comando docker.<br>

🔓 Si un usuario pertenece al grupo docker, tiene virtualmente permisos de root.
