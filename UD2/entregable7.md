# UD02 · Práctica 7
# Malware en un entorno controlado, y cómo detectarlo y eliminarlo

**Alumno:** Daniel Piqueras Burriel  
**Módulo:** Seguridad y Alta Disponibilidad (SAD)  
**Curso:** 2º ASIR

---

# Parte B · Preparar un entorno controlado

## Medidas de aislamiento

| Medida de aislamiento | ¿Hecho? | ¿Qué riesgo evita? |
|----------|----------|----------|
| Sin carpetas compartidas ni portapapeles | Sí | Evita que archivos sospechosos pasen de la máquina virtual al sistema anfitrión. |
| Snapshot previo (`limpio-antes-malware`) | Sí | Permite restaurar la máquina a un estado conocido y limpio. |
| Sin red o red aislada durante la ejecución | Sí | Impide comunicaciones externas y posibles propagaciones. |
| Máquina sin datos personales ni credenciales | Sí | Protege información sensible frente a posibles fugas o accesos no autorizados. |

### Reflexión

Un entorno controlado o sandbox permite observar el comportamiento de programas sospechosos sin comprometer otros sistemas. Además de evitar que una amenaza salga de la máquina virtual, también evita que factores externos alteren los resultados del análisis.

---

# Parte C · Fichero EICAR

## Creación del fichero de prueba

Se creó un archivo llamado:

```bash
~/eicar.txt
```

conteniendo la cadena oficial de prueba EICAR.

## Hash SHA-256

Comando utilizado:

```bash
sha256sum ~/eicar.txt
```

El hash se consultó posteriormente en VirusTotal para comprobar su nivel de detección.

### Observación

EICAR no es malware real. Se utiliza exclusivamente para comprobar si las soluciones antivirus están funcionando correctamente.

---

# Parte D · ClamAV

## D1. Instalación

### Comandos ejecutados

```bash
sudo apt update
sudo apt install -y clamav
```

La instalación se completó correctamente.

---

## D2. Actualización de firmas

### Comandos ejecutados

```bash
sudo systemctl stop clamav-freshclam
sudo freshclam
sudo systemctl start clamav-freshclam
```

### Resultado

```text
daily.cvd database is up-to-date
main.cvd database is up-to-date
bytecode.cvd database is up-to-date
```

### Fecha de la base de datos

```text
ClamAV 1.5.4
daily.cvd version 28162
main.cvd version 63
bytecode.cvd version 339
```

La base de datos de firmas estaba actualizada correctamente.

---

## D3. Escaneo del fichero EICAR

### Comando ejecutado

```bash
clamscan ~/eicar.txt
```

### Resultado

```text
/home/linux/eicar.txt: Eicar-Signature FOUND

SCAN SUMMARY

Known viruses: 3628191
Engine version: 1.5.4
Scanned files: 1
Infected files: 1
```

### Respuesta

ClamAV detectó correctamente el fichero EICAR.

**Nombre de la amenaza detectada:**

```text
Eicar-Signature
```

**Número de archivos infectados:**

```text
1
```

---

## D4. Escaneo recursivo del directorio personal

### Comando ejecutado

```bash
clamscan -r -i -v ~
```

### Resultado

```text
Scanned directories: 447
Scanned files: 3754
Infected files: 2
```

### Interpretación

ClamAV analizó todo el directorio personal y detectó dos archivos marcados como amenaza de prueba EICAR.

---

## D5. Uso de cuarentena

### Comandos ejecutados

```bash
mkdir -p ~/cuarentena

clamscan -r -i -v --move=$HOME/cuarentena ~
```

### Resultado

```text
Infected files: 3
```

### Contenido de la cuarentena

```text
eicar.com.txt.001
eicar.txt
```

### Pregunta D

Mover los archivos infectados a cuarentena suele ser mejor que eliminarlos directamente porque permite recuperarlos si se trata de un falso positivo.

Si ClamAV clasificara erróneamente un fichero importante del ERP de Torrent como malware y lo eliminara automáticamente, podría provocar interrupciones del servicio o pérdida de información. La cuarentena permite revisar el archivo antes de tomar una decisión definitiva.

---

## D6. RKHunter

### Comandos ejecutados

```bash
sudo rkhunter --propupd

sudo rkhunter --check --sk --rwo
```

### Avisos encontrados

```text
Warning: The command '/usr/bin/lwp-request' has been replaced by a script
```

```text
Warning: User 'postfix' has been added to the passwd file.
```

```text
Warning: Group 'postfix' has been added to the group file.
```

```text
Warning: Changes found in the group file for group 'postdrop'
```

```text
Warning: Hidden file found:
/etc/.resolv.conf.systemd-resolved.bak
```

```text
Warning: Hidden file found:
/etc/.updated
```

### Valoración

Los avisos observados no parecen indicar una infección real.

- `postfix` es un servicio legítimo de correo.
- `postdrop` es un grupo asociado a Postfix.
- Los archivos ocultos encontrados son ficheros de configuración o copias de seguridad.
- El aviso relacionado con `lwp-request` suele aparecer habitualmente en distribuciones Linux modernas.

Por tanto, estos resultados parecen corresponder a falsos positivos o cambios legítimos del sistema.

---

# Parte E · Análisis estático

## Comando utilizado

```bash
cat sospechoso.sh
```

## ¿Qué hace cada línea?

### Define un directorio oculto

```bash
DIR=/tmp/.cache_sistema
```

### Crea la carpeta

```bash
mkdir -p "$DIR"
```

### Registra actividad

```bash
echo "$(date) - activo" >> "$DIR/estado.log"
```

### Se copia a sí mismo

```bash
cp "$0" "$DIR/actualizador.sh"
```

### Comprueba si el puerto 8081 está ocupado

```bash
ss -tln | grep -q ':8081 '
```

### Abre un servidor HTTP local

```bash
nohup python3 -m http.server 8081 --bind 127.0.0.1 --directory "$DIR"
```

### Crea persistencia

```bash
@reboot /bin/bash /tmp/.cache_sistema/actualizador.sh
```

añadiendo una tarea programada al crontab.

---

## Predicción antes de ejecutarlo

El script:

- Crea una carpeta oculta.
- Genera archivos de configuración y registro.
- Se copia a sí mismo.
- Abre un puerto local.
- Intenta persistir tras reiniciar el sistema.

No parece borrar archivos ni realizar conexiones externas.

---

## ¿Qué habría que hacer si fuera un .exe?

Al tratarse de un binario no podría leerse con `cat`.

Sería necesario utilizar herramientas como:

```bash
strings archivo.exe
```

o herramientas de ingeniería inversa:

- Ghidra
- IDA Free
- Radare2
- objdump

---

# Parte F · Análisis dinámico

## Estado inicial

Antes de ejecutar el programa se recopiló información del sistema:

```bash
ps aux > ~/antes_procesos.txt

ss -tulpn > ~/antes_puertos.txt

crontab -l > ~/antes_cron.txt 2>&1

ls -la /tmp > ~/antes_tmp.txt
```

---

## Ejecución del programa

Primero fue necesario añadir permisos de ejecución:

```bash
chmod u+x /home/linux/Downloads/sospechoso.sh
```

Posteriormente se ejecutó:

```bash
/ home/linux/Downloads/sospechoso.sh
```

---

## Comparación de puertos

### Comando

```bash
diff ~/antes_puertos.txt ~/despues_puertos.txt
```

### Resultado relevante

```text
tcp LISTEN 0 5 127.0.0.1:8081

users:(("python3",pid=176719,fd=3))
```

### Conclusión

El programa abrió un servidor web local en:

```text
127.0.0.1:8081
```

utilizando Python.

---

## Comparación del crontab

### Comando

```bash
diff ~/antes_cron.txt ~/despues_cron.txt
```

### Resultado

```text
@reboot /bin/bash /tmp/.cache_sistema/actualizador.sh
```

### Conclusión

El programa añadió una tarea programada para ejecutarse automáticamente al reiniciar el sistema.

---

## Comparación del directorio temporal

### Comando

```bash
diff ~/antes_tmp.txt ~/despues_tmp.txt
```

### Resultado

```text
drwxrwxr-x 2 linux linux 4096 Oct 23 15:22 .cache_sistema
```

### Conclusión

Se creó la carpeta:

```text
/tmp/.cache_sistema
```

---

## Informe de comportamiento

| Qué hace | Evidencia (comando y salida) | ¿Qué fase del ataque recuerda? |
|-----------|-----------|-----------|
| Crea carpeta oculta | `/tmp/.cache_sistema` | Instalación |
| Crea ficheros | `estado.log` y `actualizador.sh` | Instalación |
| Abre un puerto | `127.0.0.1:8081` con Python | Persistencia / Control |
| Se copia a sí mismo | `actualizador.sh` | Persistencia |
| Añade ejecución automática | Entrada `@reboot` en cron | Persistencia |

---

## Comprobación de persistencia

La entrada:

```text
@reboot /bin/bash /tmp/.cache_sistema/actualizador.sh
```

permite que el script vuelva a ejecutarse después de reiniciar el sistema.

---

## Erradicación

### Detener el servidor

```bash
pkill -f "python3 -m http.server 8081"
```

### Eliminar la persistencia

```bash
crontab -e
```

Eliminar la línea:

```text
@reboot /bin/bash /tmp/.cache_sistema/actualizador.sh
```

### Eliminar los archivos creados

```bash
rm -rf /tmp/.cache_sistema
```

### Verificación

```bash
ss -tulpn

crontab -l

ls -la /tmp
```

---

## Pregunta F

Si nadie hubiera realizado una fotografía inicial del sistema, sería mucho más complicado detectar modificaciones provocadas por una amenaza.

En un entorno real se utilizan inventarios de servicios, puertos abiertos, tareas programadas, procesos y configuraciones para disponer de una línea base de referencia. Comparando el estado actual con esa línea base es posible detectar comportamientos anómalos o indicios de una intrusión.

Esta práctica está directamente relacionada con el inventario realizado en prácticas anteriores, ya que dicho inventario permite conocer qué elementos son legítimos y cuáles aparecieron posteriormente.

---

# Parte G · Preventivo o paliativo

| Medida | Preventiva | Paliativa |
|----------|:---:|:---:|
| Mantener el sistema actualizado | Si | |
| ClamAV escaneando a demanda un disco | | Si |
| Descargar software solo de repositorios oficiales | Si | |
| Copia de seguridad semanal desconectada de la red | | Si |
| Cuenta de usuario sin privilegios de administrador | Si | |
| rkhunter tras sospechar una intrusión | | Si |
| Formación del personal contra phishing | Si | |

---

# Conclusión

Durante la práctica se utilizaron herramientas de detección y análisis en un entorno controlado para estudiar el comportamiento de software potencialmente sospechoso.

ClamAV detectó correctamente el fichero de prueba EICAR y permitió poner en cuarentena los archivos detectados. Por su parte, RKHunter mostró varios avisos que, tras ser analizados, parecen corresponder a configuraciones legítimas del sistema.

El análisis estático permitió predecir el comportamiento del script antes de ejecutarlo y el análisis dinámico confirmó la creación de archivos, la apertura de un puerto local y la persistencia mediante cron. Todo ello demuestra la importancia de trabajar en entornos aislados y mantener un inventario actualizado para detectar comportamientos anómalos.
