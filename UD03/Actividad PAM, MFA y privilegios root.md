# UD05 · PAM, MFA y delegación controlada con sudo

**Alumno:** Daniel Piqueras Burriel  
**Módulo:** Seguridad y Alta Disponibilidad (SAD)  
**Curso:** 2º ASIR

---

# 1. Objetivo

El objetivo de la práctica es reforzar la seguridad del sistema Linux mediante:

- Políticas de contraseñas robustas.
- Caducidad de contraseñas.
- Autenticación multifactor (MFA).
- Delegación controlada de privilegios mediante `sudo`.

---

# 2. Política de contraseñas

## Instalación

Se instaló el módulo de complejidad de contraseñas:

```bash
sudo apt update
sudo apt install libpam-pwquality
```

---

## Configuración

Archivo modificado:

```bash
sudo nano /etc/security/pwquality.conf
```

Configuración aplicada:

```text
minlen = 12
minclass = 3
dcredit = -1
ucredit = -1
lcredit = -1
ocredit = -1
dictcheck = 1
maxrepeat = 3
enforce_for_root
```

---

## Explicación

- Longitud mínima de 12 caracteres.
- Obligación de utilizar mayúsculas.
- Obligación de utilizar minúsculas.
- Obligación de utilizar números.
- Obligación de utilizar símbolos.
- Aplicación de la política también al usuario root.

---

## Verificación

Se probó la contraseña:

```text
Asir2026!
```

Resultado:

```text
Rechazada
```

Motivo: no alcanza la longitud mínima de 12 caracteres.

Posteriormente se probó:

```text
Asir2026!123
```

Resultado:

```text
Aceptada
```

La contraseña cumple:

- Longitud mínima.
- Mayúsculas.
- Minúsculas.
- Números.
- Caracteres especiales.

### Conclusión

La política de contraseñas configurada mediante PAM funciona correctamente.

---

# 3. Caducidad de contraseñas

## Usuarios de ingeniería y supervisores

Configuración aplicada:

```bash
sudo chage -m 1 -M 60 -W 7 usuario
```

Parámetros:

- Mínimo: 1 día.
- Máximo: 60 días.
- Aviso: 7 días.

---

## Usuarios operarios

Configuración aplicada:

```bash
sudo chage -m 1 -M 90 -W 7 usuario
```

Parámetros:

- Mínimo: 1 día.
- Máximo: 90 días.
- Aviso: 7 días.

---

## Verificación

Comando utilizado:

```bash
chage -l usuario
```

La configuración quedó aplicada correctamente.

---

# 4. Implementación de MFA

## Instalación

Se instaló Google Authenticator:

```bash
sudo apt install libpam-google-authenticator
```

---

## Configuración del usuario

Como usuario normal se ejecutó:

```bash
google-authenticator
```

Durante el proceso:

- Se generó una clave secreta.
- Se generó un código QR.
- Se generaron códigos de recuperación.
- Se habilitó protección frente a reutilización de códigos.
- Se habilitó limitación de intentos.

---

## Configuración PAM

Archivo modificado:

```bash
sudo nano /etc/pam.d/sshd
```

Línea añadida:

```text
auth required pam_google_authenticator.so
```

---

## Configuración SSH

Archivo modificado:

```bash
sudo nano /etc/ssh/sshd_config
```

Opciones utilizadas:

```text
UsePAM yes
PasswordAuthentication yes
KbdInteractiveAuthentication yes
```

---

## Reinicio del servicio

```bash
sudo systemctl restart ssh
```

---

## Verificación

Conexión realizada:

```bash
ssh hugo@10.0.2.15
```

Durante el acceso se solicitaron dos factores:

```text
Password:
```

y

```text
Verification code:
```

Tras introducir la contraseña y el código generado por Google Authenticator se permitió el acceso al sistema.

### Conclusión

La autenticación multifactor quedó implementada correctamente.

---

# 5. Delegación de privilegios con sudo

## Creación de script de auditoría

Archivo creado:

```bash
sudo nano /usr/local/bin/auditoria_planta.sh
```

Contenido:

```bash
#!/bin/bash

echo "[$(date)] Auditoria de planta ejecutada por $(whoami)"
```

Permisos:

```bash
sudo chmod 755 /usr/local/bin/auditoria_planta.sh
```

---

## Configuración de sudoers

Archivo:

```bash
sudo visudo -f /etc/sudoers.d/torrent
```

Contenido:

```text
%ingenieria ALL=(root) /usr/bin/systemctl restart hmi_planta, /usr/bin/systemctl status hmi_planta

%supervisores ALL=(root) NOPASSWD: /usr/local/bin/auditoria_planta.sh
```

---

# 6. Verificación de permisos sudo

## Usuario de ingeniería

Comando:

```bash
sudo -l -U nestor
```

Resultado:

```text
(root) /usr/bin/systemctl restart hmi_planta
(root) /usr/bin/systemctl status hmi_planta
```

### Conclusión

Los miembros del grupo ingeniería pueden consultar el estado del servicio HMI y reiniciarlo.

---

## Usuario supervisor

Comando:

```bash
sudo -l -U carlos
```

Resultado:

```text
(root) NOPASSWD: /usr/local/bin/auditoria_planta.sh
```

### Conclusión

Los supervisores pueden ejecutar el script de auditoría sin necesidad de introducir contraseña.

---

## Usuario operario

Comando:

```bash
sudo -l -U alberto
```

Resultado:

```text
User alberto is not allowed to run sudo on SAD-Ubuntu.
```

### Conclusión

Los operarios no disponen de privilegios administrativos.

---

# 7. Principio de mínimo privilegio

La configuración aplicada sigue el principio de mínimo privilegio:

- Los operarios no tienen permisos sudo.
- Los supervisores sólo pueden ejecutar un script específico.
- El personal de ingeniería únicamente puede gestionar el servicio HMI.
- Ningún usuario recibe privilegios completos de administrador.

De esta forma se reduce la superficie de ataque y se limita el impacto de posibles errores o compromisos de cuentas.

---

# Conclusión

Durante esta práctica se reforzó la seguridad del sistema mediante la implementación de políticas de contraseñas robustas, caducidad de credenciales, autenticación multifactor y delegación controlada de privilegios. La combinación de estas medidas mejora significativamente la protección frente a accesos no autorizados, ataques de fuerza bruta y uso indebido de privilegios administrativos.
