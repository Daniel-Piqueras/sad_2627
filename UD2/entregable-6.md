# UD02 · Práctica 6
# ¿Puedo fiarme de este software y está al día mi sistema?

**Alumno:** Daniel Piqueras Burriel

---

# Parte A · ¿Está actualizado el sistema?

## A1. Versión del sistema operativo y kernel

Comandos ejecutados:

```bash
lsb_release -a
uname -r
```

Resultado:

```text
Distributor ID: Ubuntu
Description: Ubuntu 24.04.3 LTS
Release: 24.04
Codename: noble

7.0.0-31-generic
```

### Respuesta

- Sistema operativo: Ubuntu 24.04.3 LTS
- Nombre en clave: noble
- Kernel: 7.0.0-31-generic

---

## A2. Comprobación de actualizaciones

Comandos ejecutados:

```bash
sudo apt update
apt list --upgradable
```

Resultado:

Se encontraron múltiples paquetes pendientes de actualización, entre ellos:

- apparmor
- apport
- cloud-init
- gdm3
- gnome-shell
- gtk
- iproute2
- initramfs-tools
- kerberos

Estos paquetes incluyen correcciones de errores, mejoras de estabilidad y actualizaciones de seguridad proporcionadas por Ubuntu.

### Respuesta

El sistema dispone de numerosas actualizaciones pendientes, por lo que no se encuentra completamente actualizado.

---

## A3. Simulación de actualización

Comando ejecutado:

```bash
sudo apt-get -s upgrade | head -30
```

### Respuesta

La opción `-s` permite simular una actualización sin realizar cambios reales en el sistema.

Esta simulación muestra qué paquetes serían actualizados y las dependencias implicadas, permitiendo revisar los cambios antes de aplicarlos.

---

## A4. Fin de soporte (EOL)

| Sistema | Fecha fin de soporte | ¿Recibe parches actualmente? |
|----------|----------|----------|
| Ubuntu 24.04.3 LTS | Sí (actualmente soportado) | Sí |
| Ubuntu 8.04 (Hardy Heron) | 09/05/2013 | No |

### Pregunta A

En la oferta de empleo de Torrent se indicaba que trabajaban con Ubuntu 8.04.

Aunque se utilizaran contraseñas seguras, el sistema seguiría siendo vulnerable porque ya no recibe actualizaciones de seguridad. Las vulnerabilidades descubiertas después de finalizar el soporte permanecerían sin corregir y podrían ser aprovechadas por atacantes.

---

# Parte B · ¿De dónde vienen los programas instalados?

## B1. Repositorios configurados

Comandos ejecutados:

```bash
cat /etc/apt/sources.list
ls /etc/apt/sources.list.d/
```

### Respuesta

`/etc/apt/sources.list` es el archivo principal donde se configuran los repositorios oficiales utilizados por APT para descargar software y actualizaciones.

`/etc/apt/sources.list.d/` es un directorio que contiene archivos adicionales de configuración de repositorios, normalmente añadidos por software de terceros.

Las entradas suelen contener:

```text
deb URL distribución componentes
```

Significado:

- `deb`: repositorio de paquetes binarios.
- `URL`: servidor desde donde se descargan los paquetes.
- Distribución: versión de Ubuntu utilizada.
- Componentes:
  - **main:** software oficial soportado por Canonical.
  - **restricted:** software propietario soportado.
  - **universe:** software mantenido por la comunidad.
  - **multiverse:** software con restricciones de licencia o legales.

---

## B2. Claves de confianza

Comando ejecutado:

```bash
ls /etc/apt/trusted.gpg.d/ /usr/share/keyrings/ 2>/dev/null
```

### Respuesta

APT utiliza las claves almacenadas en estos directorios para verificar la autenticidad de los repositorios y de los paquetes descargados.

### Reflexión

Si un atacante introduce una clave falsa en `/usr/share/keyrings/`, esta no será utilizada automáticamente. Para que tenga efecto también sería necesario modificar los repositorios para que confíen en dicha clave.

Si un atacante consiguiera sustituir la clave oficial `ubuntu-archive-keyring.gpg`, podría romper la cadena de confianza del sistema. En combinación con un repositorio malicioso o una interceptación de red, el sistema podría aceptar actualizaciones manipuladas como legítimas.

---

## B3. Información de un paquete

Comando ejecutado:

```bash
apt policy mysql-server
```

### Resultado

Versión instalada:

```text
8.4.11-0ubuntu0.26.04.1
```

Versión candidata:

```text
8.4.11-0ubuntu0.26.04.1
```

Repositorio de origen:

```text
http://es.archive.ubuntu.com/ubuntu
```

### Respuesta

La tabla de versiones indica:

- La versión instalada actualmente.
- La versión candidata que se instalaría en una actualización.
- Los repositorios desde los cuales se puede obtener cada versión.
- La prioridad asignada a cada repositorio para decidir qué versión instalar.

---

## B4. Paquetes instalados manualmente

Comando ejecutado:

```bash
apt list --installed 2>/dev/null | grep -i local
```

Resultado:

```text
nessus/now 10.12.4 amd64 [installed,local]
```

### Respuesta

Nessus no procede de los repositorios oficiales de Ubuntu.

Fue descargado desde la página oficial de Tenable e instalado manualmente mediante un paquete `.deb`.

APT no verificó la autenticidad mediante repositorios oficiales, por lo que la responsabilidad de verificar la procedencia y autenticidad del paquete recayó directamente sobre el administrador que lo instaló.

---

## B5. Verificación de integridad de paquetes

Comando ejecutado:

```bash
sudo dpkg -V
```

Resultado:

```text
??5?????? /opt/nessus/etc/nessus/fipsmodule.cnf
missing /opt/nessus/var/nessus/tmp/nessusd
missing /opt/nessus/var/nessus/tmp/nessusd.service
??5?????? /opt/nessus/lib/nessus/plugins/known_CA.inc
??5?????? /usr/lib/systemd/system/cloud-init.service
```

### Respuesta

El resultado indica que existen archivos modificados o ausentes respecto al estado registrado por el gestor de paquetes.

Significado:

- `5` indica que el hash MD5 del archivo es diferente.
- `missing` indica que el archivo no se encuentra en la ubicación esperada.

Algunas modificaciones pueden ser normales en aplicaciones como Nessus, pero cambios en archivos del sistema deben ser revisados para comprobar que sean legítimos.

### Pregunta B

No necesariamente.

`dpkg -V` compara los archivos instalados con la base de datos local de paquetes. 
Si un atacante modifica tanto los archivos como la propia base de datos de dpkg, la herramienta podría no detectar la manipulación.

Por ello es recomendable disponer de copias de referencia externas o utilizar sistemas independientes de verificación de integridad.

---

# Parte C · Verificar una descarga: hash y firma

## C1. Descarga de los archivos

Comandos ejecutados:

```bash
mkdir -p ~/verif
cd ~/verif

wget https://ftp.gnu.org/gnu/hello/hello-2.12.1.tar.gz
wget https://ftp.gnu.org/gnu/hello/hello-2.12.1.tar.gz.sig
wget https://ftp.gnu.org/gnu/gnu-keyring.gpg
```

---

## C2. Cálculo del hash

Comando ejecutado:

```bash
sha256sum hello-2.12.1.tar.gz
```

Resultado:

```text
8d99142afd92576f30b0cd7cb42a8dc6809998bc5d607d88761f512e26c7db20
```

### Respuesta

El hash identifica de forma única el fichero descargado.

Si otro compañero descarga exactamente el mismo archivo, debería obtener el mismo valor.

---

## C3. Verificación de la firma digital

Comando ejecutado:

```bash
gpg --no-default-keyring --keyring ./gnu-keyring.gpg --verify hello-2.12.1.tar.gz.sig hello-2.12.1.tar.gz
```

Resultado:

```text
Good signature from "Reuben Thomas <rrt@sc3d.org>"
```

### Datos obtenidos

- Firmante: Reuben Thomas
- Fecha de firma: 29 de mayo de 2022
- Identificador de clave:
  - 24093F016FFE8602EF449BB84C8EF3DA3FD37230

### Respuesta

La firma es válida, demostrando que el fichero ha sido publicado por el autor correspondiente y no ha sido modificado desde que fue firmado.

El mensaje:

```text
WARNING: This key is not certified with a trusted signature
```

es normal y significa únicamente que la clave aún no ha sido marcada manualmente como confiable en el sistema.

---

## C4. Manipulación del fichero

Comandos ejecutados:

```bash
cp hello-2.12.1.tar.gz hello-manipulado.tar.gz

echo "x" >> hello-manipulado.tar.gz
```

Posteriormente:

```bash
sha256sum hello-2.12.1.tar.gz hello-manipulado.tar.gz
```

y

```bash
gpg --verify hello-manipulado.tar.gz.sig hello-manipulado.tar.gz
```

### Respuesta

Al añadir un único carácter al archivo:

- El hash cambia completamente.
- La firma deja de ser válida.

GPG muestra:

```text
BAD signature
```

Esto demuestra que cualquier modificación, incluso mínima, es detectada por los mecanismos de integridad y autenticidad.

---

## Pregunta C

| Situación | ¿Basta con el hash? | ¿Hace falta la firma? | Motivo |
|------------|------------|------------|------------|
| El fichero se corrompe durante la descarga | Sí | No | El hash detecta que el contenido ha cambiado. |
| Un atacante modifica el fichero y también el hash publicado en la web | No | Sí | El atacante puede modificar ambos elementos. |
| Descarga desde un mirror no fiable | No | Sí | La firma permite verificar la autenticidad del autor. |

---

# Parte D · Política de actualizaciones para Torrent

La gestión de actualizaciones será responsabilidad del administrador de sistemas y se revisará semanalmente. 
Las actualizaciones críticas de seguridad se aplicarán lo antes posible. Los servidores que ejecuten servicios importantes, 
como el ERP o el servidor de impresión, se probarán previamente en un entorno de pruebas antes de actualizar producción. 
Solo se permitirá instalar software desde repositorios oficiales o proveedores autorizados. 
Toda instalación externa requerirá autorización previa del responsable de sistemas. Los equipos que hayan alcanzado el estado EOL deberán actualizarse o sustituirse por versiones soportadas.

## Debate

### Actualizaciones automáticas

Ventajas:

- Corrigen vulnerabilidades rápidamente.
- Reducen el riesgo de olvido.

Inconvenientes:

- Pueden provocar incompatibilidades.
- Pueden afectar a servicios críticos sin previo aviso.

### Actualizaciones tras pruebas

Ventajas:

- Mayor control sobre los cambios.
- Menor riesgo de interrumpir servicios.

Inconvenientes:

- Las vulnerabilidades pueden permanecer más tiempo sin corregirse.

# Conclusión

La práctica demuestra la importancia de mantener los sistemas actualizados, utilizar repositorios confiables y verificar tanto la integridad como la autenticidad del software antes de instalarlo. Los hashes permiten detectar cambios en los archivos, mientras que las firmas digitales permiten verificar además quién es el autor legítimo del software.
