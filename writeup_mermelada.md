# Write-up: Mermelada (Pentesting)

Este documento detalla el proceso de intrusión y escalada de privilegios en la máquina "Mermelada".

## 1. Resumen Ejecutivo
Se ha logrado el compromiso total del sistema, obteniendo acceso de superusuario (root). La intrusión fue posible gracias a la presencia de webshells activas, credenciales débiles en la base de datos y una configuración de sudo insegura.

## 2. Fase de Reconocimiento
El escaneo inicial con Nmap reveló los siguientes servicios:
*   **22/tcp**: SSH (OpenSSH 9.2p1)
*   **80/tcp**: HTTP (Apache 2.4.62)
*   **3306/tcp**: MySQL (MariaDB)

La enumeración web identificó una instalación de WordPress y varios archivos sensibles en el directorio de subidas (`/uploads/`).

## 3. Acceso Inicial (Vulnerabilidad: Webshell Expuesta)
Durante la enumeración de directorios, se localizaron varios archivos PHP sospechosos en la ruta:
`/wordpress/wp-content/uploads/2026/01/`

Uno de estos archivos (`macoduweklgkmvp-1767607866.7342.php`) permitía la ejecución remota de comandos (RCE). Utilizando este vector, se estableció una reverse shell como el usuario `www-data`.

## 4. Escalada de Privilegios Local

### De www-data a mermeladita
1.  **Extracción de Credenciales**: En el archivo `wp-config.php`, se hallaron las credenciales de la base de datos (`root` / `12345`).
2.  **Enumeración de DB**: Al explorar la base de datos `mermelada`, se encontró una referencia al usuario `mermeladita` y la pista "un piquito?".
3.  **Password Cracking/Discovery**: Se localizó la contraseña en texto plano `pepitU` asociada al usuario del sistema.
4.  **Movimiento Horizontal**: Tras estabilizar la TTY, se utilizó el comando `su mermeladita` para cambiar de usuario.

**Banda de Usuario (User Flag)**: `KNBPPGDVSADNQDGBDADMKDQLADADASDP`

### De mermeladita a root (Vulnerabilidad: Sudo Misconfiguration)
Al revisar los privilegios de sudo (`sudo -l`), se descubrió que `mermeladita` podía ejecutar el binario `/usr/bin/find` como root sin necesidad de contraseña:

```bash
(ALL : ALL) NOPASSWD: /usr/bin/find
```

Se utilizó la técnica de **GTFOBins** para spawnear una shell de root:
```bash
sudo find . -exec /bin/bash -p \; -quit
```

**Bandera de Root (Root Flag)**: `MDHSABDASKDASDAPÑHOTBBQSAMCFNAMGTPEXAFGH`

## 5. Conclusiones y Recomendaciones
1.  **Eliminar archivos de depuración**: Las webshells en directorios de subidas deben ser eliminadas inmediatamente.
2.  **Fortalecer contraseñas**: Utilizar contraseñas robustas para bases de datos y usuarios del sistema.
3.  **Principio de mínimo privilegio**: No permitir que binarios como `find`, que permiten ejecución de comandos, se ejecuten con SUDO sin restricciones.

---
*¡Máquina completada con éxito!*
