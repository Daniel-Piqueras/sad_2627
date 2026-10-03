# Informe de auditoría de vulnerabilidades

## 1. Contexto y alcance

Se ha realizado una auditoría de seguridad sobre una máquina vulnerable mediante la herramienta Nessus. El objetivo del análisis es identificar vulnerabilidades presentes en 
los servicios expuestos y evaluar el riesgo que representan para la confidencialidad, integridad y disponibilidad de la información.

## 2. Metodología

La auditoría se ha realizado utilizando Nessus Essentials mediante un escaneo de vulnerabilidades remoto sobre el sistema objetivo.

El procedimiento seguido ha consistido en:

1. Configuración del objetivo.
2. Ejecución del escaneo de vulnerabilidades.
3. Revisión de los hallazgos encontrados.
4. Clasificación de las vulnerabilidades según su origen.
5. Análisis de una vulnerabilidad crítica.

## 3. Resumen de resultados

Se han analizado las siguientes vulnerabilidades:

| Severidad | Cantidad |
|------------|----------|
| Crítica | 8 |
| Alta | 1 |
| Media | 6 |
| Baja | 4 |
| Informativa | 23 |

### Vulnerabilidad con mayor puntuación CVSS

| Vulnerabilidad | CVSS |
|---------------|------|
| VNC Server 'password' Password | 10.0 |

## 4. Vulnerabilidades clasificadas

| Vulnerabilidad | Severidad / CVSS | Origen | Breve descripción |
|---------------|------------------|---------|-------------------|
| VNC Server 'password' Password | Crítica / 10.0 | Uso | El servidor VNC utiliza la contraseña por defecto "password", permitiendo el acceso remoto no autorizado. |
| NFS Shares World Readable | Media / 5.0 | Uso | Los recursos NFS están compartidos sin restricciones de acceso adecuadas. |
| SSL/TLS EXPORT_RSA <= 512-bit Cipher Suites Supported (FREAK) | Media / 4.3 | Diseño | El servicio admite algoritmos criptográficos débiles basados en claves RSA de 512 bits. |

## 5. Análisis en profundidad

### Vulnerabilidad: VNC Server 'password' Password

#### Qué es

Esta vulnerabilidad indica que el servicio VNC utiliza una contraseña extremadamente débil. Un atacante podría autenticarse fácilmente y obtener acceso remoto al sistema.

#### Cómo se explota

El atacante detecta el servicio VNC mediante un escaneo de red y prueba credenciales comunes o por defecto. Al utilizar la contraseña "password", consigue acceder directamente al sistema.

#### Cómo se mitiga

- Cambiar la contraseña actual por una contraseña robusta.
- Configurar listas de control de acceso.
- Restringir el acceso mediante firewall.
- Utilizar una VPN para las conexiones remotas.
- Deshabilitar el servicio si no es necesario.

#### Referencia

Plugin Nessus 61708 – VNC Server 'password' Password.

## 6. Recomendaciones

### Prioridad 1

Cambiar inmediatamente la contraseña del servicio VNC por una contraseña compleja y restringir el acceso únicamente a usuarios autorizados.

### Prioridad 2

Configurar correctamente los recursos NFS limitando el acceso por dirección IP o rango de red autorizado.

### Prioridad 3

Eliminar el soporte de suites criptográficas EXPORT_RSA de 512 bits y actualizar la configuración TLS del servidor para utilizar algoritmos modernos y seguros.

## 7. Conclusión

La auditoría ha identificado una vulnerabilidad crítica y varias vulnerabilidades de severidad media. Especialmente preocupante es el acceso remoto mediante una contraseña trivial en el servicio VNC, ya que permite el control completo del sistema.

No firmaría un contrato de mantenimiento sin que previamente se corrigiera la vulnerabilidad crítica detectada y se revisaran las configuraciones inseguras de NFS y TLS. Una vez aplicadas estas medidas, el nivel de riesgo disminuiría considerablemente.
