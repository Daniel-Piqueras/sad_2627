# UD03 · Práctica ACL en Linux
# Gestión de permisos mediante ACL

**Alumno:** Daniel Piqueras Burriel  
**Módulo:** ASO / SAD  
**Curso:** 2º ASIR

---

# 1. Creación de grupos

Se crearon los grupos necesarios para representar los distintos perfiles de la empresa:

```bash
sudo groupadd operarios
sudo groupadd ingenieria
sudo groupadd supervisores
```

Comprobación:

```bash
cat /etc/group | grep -E "operarios|ingenieria|supervisores"
```

---

# 2. Creación de usuarios

## Operarios

```bash
sudo useradd -m -g operarios alberto
sudo useradd -m -g operarios oscar
sudo useradd -m -g operarios hugo
```

## Ingeniería

```bash
sudo useradd -m -g ingenieria nestor
sudo useradd -m -g ingenieria victor
sudo useradd -m -g ingenieria lucia
```

## Supervisores

```bash
sudo useradd -m -g supervisores carlos
sudo useradd -m -g supervisores mary
sudo useradd -m -g supervisores john
```

Comprobación:

```bash
cat /etc/passwd | grep -E "alberto|oscar|hugo|nestor|victor|lucia|carlos|mary|john"
```

---

# 3. Creación de la estructura

Se creó la estructura indicada en el enunciado:

```text
acl_torrent/
├── hmi_planta
└── planos/
    ├── autoclave.dwg
    ├── caldera.dwg
    └── destilacion.dwg
```

Comandos utilizados:

```bash
mkdir acl_torrent
cd acl_torrent

touch hmi_planta

mkdir planos

touch planos/autoclave.dwg
touch planos/caldera.dwg
touch planos/destilacion.dwg
```

---

# 4. Asignación de propietario

Se seleccionó al usuario `nestor` como propietario de los recursos.

```bash
sudo chown nestor:ingenieria hmi_planta

sudo chown -R nestor:ingenieria planos
```

---

# 5. Permisos base

Se configuraron permisos restrictivos para que el acceso se controle mediante ACL.

```bash
chmod 700 hmi_planta

chmod 700 planos

chmod 600 planos/*.dwg
```

---

# 6. Configuración ACL

## Recurso hmi_planta

Según el diagrama:

| Grupo | Permisos |
|---------|---------|
| Operarios | Ejecutar |
| Ingeniería | Leer y escribir |

Configuración:

```bash
sudo setfacl -m g:operarios:x hmi_planta

sudo setfacl -m g:ingenieria:rw hmi_planta
```

Comprobación:

```bash
getfacl hmi_planta
```

Resultado:

```text
group:operarios:--x
group:ingenieria:rw-
```

---

## Recurso planos

Según el diagrama:

| Grupo | Permisos |
|---------|---------|
| Operarios | Lectura |
| Ingeniería | Lectura y escritura |
| Supervisores | Lectura y ejecución |

Configuración:

```bash
sudo setfacl -R -m g:operarios:r planos

sudo setfacl -R -m g:ingenieria:rw planos

sudo setfacl -R -m g:supervisores:rx planos
```

---

## ACL por defecto

Para que los archivos nuevos hereden automáticamente los permisos:

```bash
sudo setfacl -d -m g:operarios:r planos

sudo setfacl -d -m g:ingenieria:rw planos

sudo setfacl -d -m g:supervisores:rx planos
```

---

# 7. Verificación de ACL

## ACL de hmi_planta

```bash
getfacl hmi_planta
```

Resultado:

```text
group:operarios:--x
group:ingenieria:rw-
```

---

## ACL de planos

```bash
getfacl planos
```

Resultado:

```text
group:operarios:r--
group:ingenieria:rw-
group:supervisores:r-x
```

ACL por defecto:

```text
default:group:operarios:r--
default:group:ingenieria:rw-
default:group:supervisores:r-x
```

---

# 8. Pruebas de acceso

## Operario (alberto)

Inicio de sesión:

```bash
su - alberto
```

Lectura de planos:

```bash
cat /srv/acl_torrent/planos/autoclave.dwg
```

Resultado:

```text
Permitido
```

Intento de escritura:

```bash
echo prueba >> /srv/acl_torrent/planos/autoclave.dwg
```

Resultado:

```text
Permission denied
```

### Conclusión

El usuario operario puede leer la documentación pero no modificarla.

---

## Ingeniería (nestor)

Inicio de sesión:

```bash
su - nestor
```

Escritura sobre planos:

```bash
echo prueba >> /srv/acl_torrent/planos/autoclave.dwg
```

Resultado:

```text
Permitido
```

### Conclusión

El personal de ingeniería puede consultar y modificar los planos.

---

## Supervisor (carlos)

Inicio de sesión:

```bash
su - carlos
```

Lectura:

```bash
cat /srv/acl_torrent/planos/autoclave.dwg
```

Resultado:

```text
Permitido
```

Escritura:

```bash
echo prueba >> /srv/acl_torrent/planos/autoclave.dwg
```

Resultado:

```text
Permission denied
```

### Conclusión

Los supervisores pueden consultar la documentación técnica, pero no modificarla.

---

# 9. Problemas encontrados

Durante las pruebas se detectó que los usuarios no podían acceder a la práctica cuando ésta se encontraba dentro del directorio personal:

```text
/home/linux/acl_torrent
```

El acceso era bloqueado por los permisos del directorio personal.

### Solución aplicada

Se trasladó la práctica a:

```text
/srv/acl_torrent
```

y se volvieron a aplicar las ACL correspondientes.

También fue necesario otorgar permiso de ejecución sobre el directorio para permitir que los usuarios pudieran atravesarlo y acceder a los archivos contenidos.

---

# 10. Preguntas de reflexión

## ¿Por qué no bastan los permisos tradicionales de Unix?

Los permisos tradicionales sólo permiten definir permisos para:

- Propietario
- Grupo
- Otros

En este escenario varios grupos necesitan permisos distintos sobre los mismos recursos. Las ACL permiten asignar permisos específicos a múltiples grupos y usuarios de forma mucho más flexible.

---

## ¿Qué es la máscara (mask)?

La máscara ACL define el límite máximo de permisos efectivos que pueden tener los usuarios y grupos definidos mediante ACL.

Ejemplo:

```text
group:ingenieria:rwx
mask:r--
```

Aunque el grupo tenga asignado `rwx`, únicamente dispondrá efectivamente de:

```text
r--
```

---

## Diferencia entre ACL de acceso y ACL por defecto

### ACL de acceso

Se aplican a archivos y directorios existentes.

Ejemplo:

```bash
setfacl -m g:ingenieria:rw planos
```

### ACL por defecto

Se aplican únicamente a directorios y determinan los permisos que heredarán los nuevos archivos y subdirectorios.

Ejemplo:

```bash
setfacl -d -m g:ingenieria:rw planos
```

---

## Incorporación de un nuevo operario

Bastaría con crear el usuario y añadirlo al grupo correspondiente:

```bash
sudo useradd -m pepe

sudo usermod -aG operarios pepe
```

No sería necesario modificar las ACL, ya que los permisos están asignados al grupo `operarios`.

---

# Conclusión

Las ACL permiten implementar modelos de seguridad más flexibles que los permisos tradicionales de Linux. Gracias a ellas se han podido asignar permisos específicos a los grupos de operarios, ingeniería y supervisores, garantizando que cada perfil sólo pueda realizar las acciones autorizadas sobre los recursos de la empresa.
