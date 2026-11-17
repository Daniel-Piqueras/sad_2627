# UD03 · Certificados y HTTPS
# Parte 1 · Nuestra propia autoridad de certificación

**Alumno:** Daniel Piqueras Burriel

---

# Paso 0 - Instalación de Apache

Instalación de paquetes necesarios:

```bash
sudo apt update
sudo apt install -y apache2 openssl curl cron
```

Comprobación:

```bash
systemctl status apache2 --no-pager
```

```bash
sudo ss -tlnp | grep ':80'
```

Resultado:

- Apache se encontraba en estado `active (running)`.
- El puerto 80 estaba abierto y escuchando conexiones.

Desde el navegador se accedió a:

```text
http://192.168.57.2
```

mostrándose correctamente la página:

```text
Apache2 Default Page
```

### ¿Qué indica el navegador?

El navegador indica que la conexión no es segura porque se está utilizando HTTP y no HTTPS. La información viaja sin cifrar y podría ser interceptada o modificada por un atacante situado en la red.

---

# Paso 1 - Creación de la estructura de la CA

Creación de directorios:

```bash
mkdir -p ~/ca/private ~/ca/certs

chmod 700 ~/ca/private
```

Comprobación:

```bash
ls -ld private
```

Resultado:

```text
drwx------
```

Esto garantiza que únicamente el usuario propietario puede acceder a las claves privadas.

---

# Paso 2 - Generación de la clave privada de la CA

Comando utilizado:

```bash
openssl genrsa -aes256 -out private/ca.key 4096
```

Resultado:

```text
ca.key
```

### Reflexión

La clave privada de una CA debe protegerse con una contraseña incluso aunque esté almacenada en un directorio con permisos restrictivos. Si un atacante consiguiera copiar el archivo, seguiría necesitando la contraseña para poder usarla.

---

# Paso 3 - Creación del certificado raíz

Comando utilizado:

```bash
openssl req -x509 -new -key private/ca.key -sha256 -days 3650 \
-out certs/ca.crt \
-subj "/C=ES/O=Torrent S.L./CN=Torrent S.L. CA Raiz" \
-addext "basicConstraints=critical,CA:TRUE" \
-addext "keyUsage=critical,keyCertSign,cRLSign"
```

Resultado:

Se generó el certificado:

```text
certs/ca.crt
```

---

# Paso 4 - Inspección del certificado raíz

Comandos utilizados:

```bash
openssl x509 -in certs/ca.crt -noout -subject -issuer -dates
```

```bash
openssl x509 -in certs/ca.crt -noout -text | grep -A1 "Basic Constraints"
```

Resultado:

```text
subject=C = ES, O = Torrent S.L., CN = Torrent S.L. CA Raiz
issuer=C = ES, O = Torrent S.L., CN = Torrent S.L. CA Raiz
```

```text
CA:TRUE
```

### ¿Por qué coinciden Subject e Issuer?

Porque se trata de un certificado autofirmado. La propia Autoridad Certificadora firma su propio certificado.

### Fecha de validez

```text
notAfter=Nov 13 22:17:26 2036 GMT
```

La CA será válida hasta el 13 de noviembre de 2036.

---

# Paso 5 - Clave privada y CSR del servidor

Generación de la clave privada:

```bash
openssl genrsa -out private/srv.key 2048
```

Generación de la CSR:

```bash
openssl req -new -key private/srv.key -out srv.csr \
-subj "/C=ES/O=Torrent S.L./CN=hmi.torrent.local"
```

Archivos generados:

```text
srv.key
srv.csr
```

---

# Paso 6 - Configuración del SAN

Archivo creado:

```bash
nano srv.ext
```

Contenido:

```text
subjectAltName = DNS:hmi.torrent.local, IP:192.168.57.2
basicConstraints = CA:FALSE
keyUsage = digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth
```

---

# Paso 7 - Firma del certificado del servidor

Comando utilizado:

```bash
openssl x509 -req -in srv.csr \
-CA certs/ca.crt -CAkey private/ca.key -CAcreateserial \
-out certs/srv.crt -days 365 -sha256 -extfile srv.ext
```

Resultado:

Se obtuvo:

```text
certs/srv.crt
```

### ¿Por qué el certificado del servidor dura 365 días?

Porque los certificados de servidor se renuevan con frecuencia para reducir el impacto de una posible filtración de la clave privada, mientras que las CA raíz suelen tener una vida útil mucho más larga.

---

# Paso 8 - Verificación de la cadena de confianza

Comandos:

```bash
openssl verify -CAfile certs/ca.crt certs/srv.crt
```

Resultado:

```text
certs/srv.crt: OK
```

Inspección:

```bash
openssl x509 -in certs/srv.crt -noout -subject -issuer -ext subjectAltName
```

Resultado:

```text
subject=C = ES, O = Torrent S.L., CN = hmi.torrent.local
```

```text
issuer=C = ES, O = Torrent S.L., CN = Torrent S.L. CA Raiz
```

```text
DNS:hmi.torrent.local, IP Address:192.168.57.2
```

---

# Paso 9 - Instalación del certificado en Apache

Copiado de certificados:

```bash
sudo cp ~/ca/certs/srv.crt /etc/ssl/certs/torrent-srv.crt

sudo cp ~/ca/private/srv.key /etc/ssl/private/torrent-srv.key

sudo chmod 600 /etc/ssl/private/torrent-srv.key
```

Activación de SSL:

```bash
sudo a2enmod ssl

sudo a2ensite default-ssl
```

Comprobación:

```bash
sudo apache2ctl configtest
```

Resultado:

```text
Syntax OK
```

Apache quedó escuchando en HTTPS:

```bash
sudo ss -tlnp | grep ':443'
```

Resultado:

```text
*:443
```

---

# Paso 10 - Primera prueba HTTPS

Comando:

```bash
curl -I https://192.168.57.2
```

Resultado:

```text
SSL certificate problem:
unable to get local issuer certificate
```

Posteriormente:

```bash
curl -I --cacert ~/ca/certs/ca.crt https://192.168.57.2
```

Resultado:

```text
HTTP/1.1 200 OK
```

### Explicación

El primero comando falla porque el sistema no conoce la Autoridad Certificadora de Torrent S.L.

El segundo funciona porque se indica explícitamente a curl que debe confiar en dicha CA.

---

# Paso 11 - Importación de la CA

Se publicó el certificado:
```bash
sudo cp ~/ca/certs/ca.crt /var/www/html/torrent-ca.crt
```

Obtención de la huella digital:

```bash
openssl x509 -in ~/ca/certs/ca.crt -noout -fingerprint -sha256
```

Resultado:

```text
3D:D0:F0:E4:B2:EA:AF:4F:15:1E:90:1D:FB:30:C1:F6:56:5D:E1:D3:0D:30:52:B9:5E:6F:67:3A:EB:0D:3D:B7
```

La CA fue importada en Firefox tras comprobar que la huella coincidía exactamente.

Tras la importación, el navegador mostró conexión segura mediante HTTPS.

### ¿Por qué es peligroso descargar la CA sin verificar la huella?

Porque un atacante podría sustituir el certificado por otro malicioso. Si el usuario lo importara sin comprobar la huella, estaría confiando en una CA falsa.

---

# Paso 12 - Inspección de la conexión TLS

Comando:

```bash
openssl s_client -connect 192.168.57.2:443 \
-CAfile ~/ca/certs/ca.crt \
</dev/null 2>/dev/null | grep -E 's:|i:|Verify return code|Cipher'
```

Resultado:

```text
s:C = ES, O = Torrent S.L., CN = hmi.torrent.local
```

```text
i:C = ES, O = Torrent S.L., CN = Torrent S.L. CA Raiz
```

```text
TLSv1.3
```

```text
TLS_AES_256_GCM_SHA384
```

```text
Verify return code: 0 (ok)
```

### Investigación

Sin utilizar `-CAfile`:

```bash
openssl s_client -connect 192.168.57.2:443 \
</dev/null 2>/dev/null | grep -E 'Verify return code'
```

Resultado:

```text
Verify return code: 21 (unable to verify the first certificate)
```

Esto ocurre porque la CA de Torrent S.L. no forma parte de las autoridades de confianza instaladas en el sistema.

---

# Paso 13 - Copia de seguridad

Comando:
```bash
tar czf ~/ca-torrent-backup.tar.gz -C ~ ca
```

Resultado:

```text
ca-torrent-backup.tar.gz
```

La copia de seguridad se guardará para futuras prácticas relacionadas con VPN y certificados.
