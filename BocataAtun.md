
# 🧠 Bocata de Atún | TheHackersLabs

## 📍 Objetivo
Obtener dos flags a través de técnicas de enumeración, esteganografía, OSINT y análisis de metadatos.

---

## 🔎 Paso 1: Escaneo de la máquina

Una vez identificada la IP de la máquina (`192.168.1.96`), realizamos un escaneo de puertos:

```bash
nmap -p- --open -sS -sC -sV --min-rate=5000 -n -vvv -Pn -oN escaneo.txt 192.168.1.96
```
<img width="563" height="147" alt="image" src="https://github.com/user-attachments/assets/fbfce9ce-5a1d-4275-a3ab-e261e7d75e34" />

### ✅ Servicios detectados:
- `22/tcp` – SSH (OpenSSH 9.6p1)
- `80/tcp` – HTTP (Apache 2.4.58)
- `3306/tcp` – MySQL/MariaDB


## 🌐 Paso 2: Análisis del servicio web (puerto 80)

Accedemos al sitio web y detectamos que presenta una galería de imágenes simple.

Utilizando `gobuster` encontramos dos rutas relevantes:

<img width="560" height="221" alt="image" src="https://github.com/user-attachments/assets/402a4b80-3446-4b33-ba5f-2dbbc60453f0" />


```
/IMAGES
/test.php
```

En `/IMAGES`, encontramos varias imágenes. Una destaca por su nombre codificado en hexadecimal:

```
horse_426f7272617220646573707565730a.jpg
```

<img width="227" height="28" alt="image" src="https://github.com/user-attachments/assets/2301ab55-6e82-4e45-b556-45e08d86c4bc" />

### ✅ El nombre se traduce como:
**"Borrar después"** → lo que indica que puede contener algo oculto.


## 🕵️ Paso 3: Análisis de la imagen

Verificamos el tipo del archivo:

```bash
file horse_426f7272617220646573707565730a.jpg
```

Usamos `stegseek`, una herramienta de fuerza bruta para extraer archivos ocultos en imágenes protegidas con contraseña:

```bash
stegseek horse_426f7272617220646573707565730a.jpg /usr/share/wordlists/rockyou.txt
```

<img width="475" height="108" alt="image" src="https://github.com/user-attachments/assets/997b0e6e-a7b8-4670-aa85-9aa1604378c8" />

### 🎯 Éxito:
`stegseek` encuentra la contraseña válida: `123mango` y extrae un archivo oculto que contiene la ruta:

```
/sup3r_secret_d00r.php
```

## 🔐 Paso 4: Acceso al panel oculto

Accedemos a `sup3r_secret_d00r.php` y encontramos un formulario que solicita el nombre de un agente.

Probamos con `Melissa` (extraído del mensaje oculto) y accedemos a sus conversaciones.

<img width="190" height="139" alt="image" src="https://github.com/user-attachments/assets/b658045c-729b-4b94-97bd-b0a73a2c0fab" /><br>

<img width="451" height="367" alt="image" src="https://github.com/user-attachments/assets/e88926d0-def0-4b49-a355-08aad83217d8" />


Descubrimos que el parámetro `name` en la URL es vulnerable a enumeración:

```
secr3t_message_app.php?name=christian
```

<img width="449" height="430" alt="image" src="https://github.com/user-attachments/assets/ad88ae80-b4b6-43c5-a2c5-210768dfe9cf" />

Esto nos permite leer conversaciones de múltiples agentes.


## 🧠 Paso 5: Deducción por OSINT

Identificamos los siguientes nombres en las conversaciones:
- Melissa
- Christian
- Seth
- Lauren
- Josh

Comprobamos que todos coinciden con actores de *The Walking Dead*.

➡️ Esto sugiere que los nombres reales pueden ser clave para avanzar o incluso usuarios del sistema.

Probamos con más nombres reales y con `Norman` (Daryl en la serie) conseguimos leer una conversación oculta.


## 📥 Paso 6: Conversaciones de Norman

Al acceder como `Norman`, encontramos una conversación con Ross:

> “Toda la información está en la ruta `m1si0n.php`. La contraseña es el mismo lugar donde nos vimos en la primera misión.”  
> “He dejado una foto en `http://juanlopez.thl/78074567.png`”

<img width="557" height="103" alt="image" src="https://github.com/user-attachments/assets/b2c1cc48-13b9-4545-b117-f897b316361b" />


## 🖼️ Paso 7: Análisis de imagen `78074567.png`

Descargamos la imagen y analizamos los metadatos:

```bash
exiftool 78074567.png
```

<img width="568" height="243" alt="image" src="https://github.com/user-attachments/assets/6f9e7f54-7867-4e2e-9307-b7bdbca63690" />

### ✅ Resultados clave:
- **Modelo de cámara**: `flag(66 6c 61 67 31)` → corresponde a `flag1`
- **Comentario**: contiene una URL de stream MJPG


## 🌍 Paso 8: OSINT + Google Dorks

Usamos Google Dorks para localizar el origen del stream:

```text
inurl:"axis-cgi/mjpg/video.cgi" inurl:"camera=1" inurl:"resolution=1280x960" inurl:"compression=50" inurl:"videocodec=h264"
```

### 🔍 Resultado:
Una cámara pública ubicada en **Almería**:
```
https://wcam.apalmeria.com:10880/axis-cgi/mjpg/video.cgi?camera=1...
```

<img width="467" height="131" alt="image" src="https://github.com/user-attachments/assets/a8cf891c-de08-4dc9-b6cd-51afe0f2bf41" />


## 🔓 Paso 9: Acceso a `m1si0n.php`

Con la pista de “el lugar donde nos vimos en la primera misión” y el origen de la webcam, deducimos la contraseña:

➡️ **Almería**

Ingresamos `Almería` en el formulario y obtenemos el mensaje final.

<img width="254" height="112" alt="image" src="https://github.com/user-attachments/assets/59f14ffe-7755-4f3e-936a-7627205025aa" /><br>

<img width="299" height="180" alt="image" src="https://github.com/user-attachments/assets/ea511028-76a3-4fcb-a8cf-d35d5de580ad" />


## 🏁 Flags obtenidas

- → de los metadatos de la imagen PNG
- → obtenida tras acceder a `m1si0n.php`

---

## ✅ Conclusión

Este reto combinó múltiples técnicas:

- Enumeración de usuarios basada en patrones narrativos
- Esteganografía con `stegseek`
- Recolección OSINT mediante Google Dorks
- Análisis forense de metadatos
- Decodificación de información oculta
