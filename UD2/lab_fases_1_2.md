# Laboratorio fases 1 y 2 — Estudio Torrent

## A1. whois y nslookup

No es necesario.

## B. Antes y después

### 1. Objetivo y alcance

El objetivo es identificar los puertos y servicios accesibles de `Torrent-Vulnerable`, desactivar los servicios que no sean necesarios y comparar el resultado antes y después de aplicar las contramedidas.

**Alcance autorizado:** solo se escanea `Torrent-Vulnerable` desde `Torrent-Auditor`, dentro de la red interna aislada. `Torrent-Vulnerable` no debe tener adaptador NAT ni adaptador puente.

La política de este laboratorio indica que el servidor solo necesita ofrecer:

- **HTTP:** puerto TCP 80.
- **SSH:** puerto TCP 22.

No se deben apagar servicios sin identificarlos primero. Los resultados que dependen del laboratorio se dejan como campos para completar con las evidencias reales; no deben inventarse.

### 2. Preparación

1. Arrancar `Torrent-Auditor` y `Torrent-Vulnerable`.
2. Antes de modificar nada, crear una instantánea de `Torrent-Vulnerable` en VirtualBox: **Máquina → Tomar instantánea**.
3. En `Torrent-Vulnerable`, averiguar su dirección IP:

   ```bash
   ip a
   ```

4. Desde `Torrent-Auditor`, comprobar la conectividad:

   ```bash
   ping -c 3 <IP>
   ```

   - IP de `Torrent-Vulnerable`: `<IP_REAL>`
   - Resultado inicial del ping: `<RESPONDE / NO RESPONDE>`

### 3. Conexión por SSH

Desde una terminal de `Torrent-Auditor`, conectar con la IP de `Torrent-Vulnerable`:

```bash
ssh msfadmin@<IP>
```

Cuando pregunte si se confía en la clave del servidor, comprobar que se trata de la máquina del laboratorio antes de aceptarla. Después, introducir la contraseña del usuario:

```text
msfadmin
```

Al escribir la contraseña no se muestran caracteres en la terminal; es normal. Si la conexión funciona, aparecerá una terminal remota. Para cerrar la sesión:

```bash
exit
```

#### Alternativa para sistemas antiguos (por ejemplo, Metasploitable 2)

Las versiones antiguas de OpenSSH pueden ofrecer algoritmos de clave de servidor que los clientes actuales deshabilitan por seguridad. Si aparece un error parecido a `no matching host key type found` y el mensaje indica que el servidor ofrece `ssh-rsa`, prueba desde `Torrent-Auditor`:

```bash
ssh -oHostKeyAlgorithms=+ssh-rsa -oPubkeyAcceptedAlgorithms=+ssh-rsa msfadmin@<IP>
```

Si el error indica expresamente que el servidor solo ofrece `ssh-dss`, puedes probar esta alternativa más antigua:

```bash
ssh -oHostKeyAlgorithms=+ssh-dss msfadmin@<IP>
```

Usa estas opciones únicamente para la máquina antigua del laboratorio; no las apliques globalmente en la configuración de SSH ni las uses para conectarte a sistemas reales sin una razón justificada. Si aparece otro error, lee el mensaje completo antes de añadir más opciones: puede deberse a un algoritmo de intercambio de claves, a que SSH no esté activo o a un problema de red.

Si no conecta, comprobar que la IP es correcta, que ambas máquinas están en la misma red interna y que el servicio SSH está activo y escucha en el puerto 22. No se debe añadir NAT ni puente a `Torrent-Vulnerable` para resolver un problema de conectividad.

### 4. Medición inicial («antes»)

Ejecutar desde `Torrent-Auditor`:

```bash
sudo nmap -sV <IP> -oN antes.txt
```

`-sV` intenta identificar los servicios y sus versiones; `-oN` guarda el resultado en `antes.txt`.

Anotar los resultados reales:

- Número de puertos abiertos: `<NÚMERO>`
- Puertos, servicios y versiones detectados: `<RESULTADOS DE NMAP>`
- ¿Responde al ping?: `<SÍ / NO>`

Conservar `antes.txt` para compararlo al final.

### 5. Inventario local de servicios

En `Torrent-Vulnerable`, iniciar sesión con el usuario `msfadmin` (contraseña inicial del laboratorio: `msfadmin`) y ejecutar:

```bash
sudo netstat -tulpn
ps aux
```

`netstat` muestra puertos y sockets en escucha y, cuando es posible, el proceso asociado. Un puerto puede aparecer a nombre de `inetd` o `xinetd`: en ese caso, el servicio se inicia bajo demanda y puede que no aparezca como proceso permanente en `ps`.

#### Cómo identificar un servicio

**1. Consultar el servicio desde `Torrent-Auditor` (primera opción):**

```bash
sudo nmap -sV -p <PUERTO> <IP>
```

Nmap analiza la respuesta del puerto para intentar identificar el servicio. El número de puerto por sí solo no demuestra qué programa está detrás.

**2. Si el puerto pertenece a `inetd` o `xinetd`, consultar su configuración** en `Torrent-Vulnerable`:

```bash
grep -v '^#' /etc/inetd.conf
grep -H 'disable' /etc/xinetd.d/*
sudo grep -rn -w <PUERTO> /etc/inetd.conf /etc/xinetd.conf /etc/xinetd.d/ 2>/dev/null
```

En `inetd.conf`, revisad la última columna para localizar el programa ejecutado. En la configuración de `xinetd`, buscad las líneas `service`, `server` y `disable`. No todas las instalaciones tienen todos estos ficheros.

**3. Consultar `/etc/services` como referencia:**

```bash
grep -w '<PUERTO>/tcp' /etc/services
```

Este fichero asocia puertos con nombres convencionales, pero puede estar incompleto o no coincidir con el servicio real. Si no aparece una entrada, no significa que el puerto esté libre.

Una vez identificada la ruta del programa, se puede averiguar qué paquete lo instaló:

```bash
dpkg -S /ruta/al/programa
```

Sustituir `/ruta/al/programa` por la ruta real. Si no se encuentra el paquete, registrar el resultado en vez de asumir cuál es.

#### Tabla de inventario

Completar una fila por cada puerto/servicio observado. Los puertos 22 y 80 se mantienen si corresponden a SSH y HTTP, respectivamente; los demás se investigan antes de decidir.

| Puerto/protocolo | Servicio identificado | Programa o dueño del puerto | ¿Necesario? | Decisión y motivo |
|---|---|---|---|---|
| 22/tcp | `<RESULTADO REAL>` | `<PROGRAMA / PROCESO>` | Sí, si es SSH | Mantener si se confirma que es SSH |
| 80/tcp | `<RESULTADO REAL>` | `<PROGRAMA / PROCESO>` | Sí, si es HTTP | Mantener si se confirma que es HTTP |
| `<PUERTO>` | `<SERVICIO>` | `<PROGRAMA / inetd / xinetd>` | `<SÍ / NO>` | `<MANTENER / DESACTIVAR / INVESTIGAR>` |
| `<PUERTO>` | `<SERVICIO>` | `<PROGRAMA / inetd / xinetd>` | `<SÍ / NO>` | `<MANTENER / DESACTIVAR / INVESTIGAR>` |

### 6. Contramedida 1: desactivar servicios innecesarios

La forma correcta depende de quién tenga el puerto. La guía del laboratorio distingue dos casos.

#### Caso A: el propio programa escucha en el puerto (servicio permanente)

Ejemplo de la guía: `proftpd`. El puerto de ejemplo es 2121, pero en el laboratorio hay que usar el puerto y el nombre de servicio que se hayan identificado realmente.

1. Comprobar quién tiene el puerto:

   ```bash
   sudo netstat -tulpn | grep '<PUERTO>'
   ```

2. Buscar el script y la configuración de inicio:

   ```bash
   ls /etc/init.d/ | grep -i '<SERVICIO>'
   ls /etc/rc2.d/ | grep -i '<SERVICIO>'
   ```

3. Detener el servicio ahora, usando su nombre real:

   ```bash
   sudo /etc/init.d/<SERVICIO> stop
   ```

4. Evitar que arranque automáticamente en esta instalación antigua:

   ```bash
   sudo update-rc.d -f <SERVICIO> remove
   ```

5. Comprobar desde la máquina objetivo y desde `Torrent-Auditor` que el puerto ya no está abierto. Si se reinicia la máquina como parte de la comprobación, repetir las verificaciones después del reinicio.

No borrar el script de `/etc/init.d/` a ciegas. Si el servicio no coincide con el ejemplo o el comando no existe, detenerse y consultar al profesor.

#### Caso B: servicio gestionado por `inetd` o `xinetd` (servicio bajo demanda)

En este caso, el dueño del puerto puede ser `inetd` o `xinetd`, y el programa del servicio puede no aparecer en `ps` mientras nadie se conecta.

1. Buscar la definición por el nombre del servicio o por el número de puerto:

   ```bash
   grep -n -w '<SERVICIO>' /etc/inetd.conf /etc/xinetd.conf /etc/xinetd.d/* 2>/dev/null
   sudo grep -rn -w <PUERTO> /etc/inetd.conf /etc/xinetd.conf /etc/xinetd.d/ 2>/dev/null
   ```

2. Hacer una copia de seguridad del fichero concreto antes de editarlo. Por ejemplo, si la definición está en `/etc/inetd.conf`:

   ```bash
   sudo cp /etc/inetd.conf /root/inetd.conf.bak
   ```

   Si está en un fichero de `/etc/xinetd.d/`, copiar ese fichero y adaptar el nombre de la copia.

3. Desactivar **solo el servicio identificado**:
   - En `inetd.conf`, comentar la línea correspondiente añadiendo `#` al principio.
   - En `xinetd`, establecer `disable = yes` en el bloque del servicio correspondiente.

4. Recargar el portero si procede:

   ```bash
   sudo /etc/init.d/xinetd reload
   ```

   Si `reload` no aplica el cambio, la guía indica probar `restart`, siempre comprobando después que los demás servicios gestionados por `xinetd` siguen funcionando.

**Importante:** no detener `xinetd` entero para apagar un solo servicio, porque podrían dejar de funcionar otros servicios que sí se necesitan.

#### Comprobación tras desactivar

En `Torrent-Vulnerable`:

```bash
sudo netstat -tulpn
```

En `Torrent-Auditor`, comprobar el puerto concreto:

```bash
sudo nmap -sV -p <PUERTO> <IP>
```

Registrar el resultado. La guía recomienda verificar el estado dentro de la máquina, desde fuera con Nmap y, cuando corresponda, tras reiniciar. Un servicio no se considera desactivado de forma permanente hasta comprobar que no vuelve a arrancar.

### 7. Contramedida 2: bloquear el ping

En `Torrent-Vulnerable`, ejecutar:

```bash
sudo iptables -A INPUT -p icmp --icmp-type echo-request -j DROP
```

Comprobar la regla:

```bash
sudo iptables -L INPUT -n -v
```

Desde `Torrent-Auditor`, repetir:

```bash
ping -c 3 <IP>
```

Resultado posterior del ping: `<RESPONDE / NO RESPONDE>`.

La regla descarta las solicitudes ICMP de eco. No cierra puertos TCP y no impide necesariamente que Nmap detecte la máquina mediante los servicios que siguen accesibles. Según la configuración del cortafuegos, las reglas de `iptables` también pueden no persistir tras reiniciar.

### 8. Medición posterior («después»)

Desde `Torrent-Auditor`, repetir el escaneo con las mismas opciones:

```bash
sudo nmap -sV <IP> -oN despues.txt
diff -u antes.txt despues.txt
```

Comparar `antes.txt` y `despues.txt`, y completar la tabla con las observaciones reales.

| Aspecto | Antes | Contramedida aplicada | Después |
|---|---|---|---|
| Número de puertos abiertos | `<RESULTADO REAL>` | Desactivar servicios innecesarios | `<RESULTADO REAL>` |
| Servicios y versiones visibles | `<RESULTADO REAL>` | Reducir los servicios expuestos | `<RESULTADO REAL>` |
| Respuesta al ping | `<SÍ / NO>` | Regla ICMP `DROP` | `<SÍ / NO>` |
| Puerto 22/tcp (SSH) | `<ESTADO REAL>` | Mantener si se necesita SSH | `<ESTADO REAL>` |
| Puerto 80/tcp (HTTP) | `<ESTADO REAL>` | Mantener si se necesita HTTP | `<ESTADO REAL>` |

Si los resultados no cambian, no asumir que la contramedida ha funcionado: revisar el servicio, el puerto, la regla y la configuración, y repetir la comprobación.

### 9. Ampliación (opcional)

#### DROP frente a REJECT

Realizar esta prueba únicamente en `Torrent-Vulnerable` y desde `Torrent-Auditor`. El puerto 23 se usa como ejemplo; verificar antes que la prueba no interfiera con un servicio necesario.

Aplicar `DROP`:

```bash
sudo iptables -A INPUT -p tcp --dport 23 -j DROP
```

Desde `Torrent-Auditor`:

```bash
sudo nmap -sV -p 23 <IP>
```

Retirar la regla de prueba antes de cambiarla:

```bash
sudo iptables -D INPUT -p tcp --dport 23 -j DROP
```

Aplicar `REJECT`:

```bash
sudo iptables -A INPUT -p tcp --dport 23 -j REJECT --reject-with tcp-reset
```

Volver a probar desde `Torrent-Auditor`:

```bash
sudo nmap -sV -p 23 <IP>
```

Retirar la regla al terminar:

```bash
sudo iptables -D INPUT -p tcp --dport 23 -j REJECT --reject-with tcp-reset
```

`DROP` descarta el paquete sin responder; `REJECT` devuelve un rechazo. Por eso Nmap puede informar de estados diferentes. No dejar reglas temporales activas al finalizar.

#### Registrar posibles escaneos

En `Torrent-Vulnerable`:

```bash
sudo iptables -I INPUT 1 -p tcp --syn -m limit --limit 5/second -j LOG --log-prefix "POSIBLE-ESCANEO: "
```

Después de realizar el escaneo autorizado desde `Torrent-Auditor`, consultar:

```bash
dmesg | grep 'POSIBLE-ESCANEO' | tail
```

La regla registra paquetes TCP SYN que coincidan, pero no los bloquea. El tráfico legítimo también puede generar registros. Retirarla al acabar:

```bash
sudo iptables -D INPUT -p tcp --syn -m limit --limit 5/second -j LOG --log-prefix "POSIBLE-ESCANEO: "
```

#### Ocultar banners

Si el profesor indica cómo modificar el banner de un servicio, comparar la identificación antes y después:

```bash
sudo nmap -sV -p <PUERTO> <IP>
```

Ocultar un banner puede dificultar la identificación de la versión, pero no corrige por sí mismo las vulnerabilidades del programa.

## Reflexión

### 1. ¿Qué contramedida ha reducido más lo que ve el atacante? ¿Por qué?

En general, desactivar servicios innecesarios reduce de forma más directa la superficie de ataque, porque elimina servicios que estaban accesibles. Bloquear el ping dificulta el reconocimiento mediante ICMP, pero los servicios TCP que siguen abiertos pueden continuar detectándose con Nmap. La conclusión del laboratorio debe basarse en la comparación real entre `antes.txt` y `despues.txt`.

### 2. Si solo ocultamos un banner, pero el servicio sigue activo, ¿la vulnerabilidad sigue ahí?

Sí, puede seguir existiendo. Ocultar la versión no corrige los fallos del software; solo dificulta una parte de su identificación. El servicio puede seguir siendo vulnerable y puede reconocerse por otras respuestas o comportamientos. La solución debe incluir actualizar o sustituir el software, corregir su configuración y limitar su exposición.

### 3. ¿Puede una contramedida sustituir a actualizar un sistema operativo sin soporte? ¿Qué haríamos en una empresa real?

No. Las contramedidas pueden reducir el riesgo temporalmente, pero no sustituyen las actualizaciones de seguridad ni el soporte del sistema operativo. En una empresa, se debería inventariar el sistema y sus dependencias, evaluar el riesgo, planificar y probar la migración a una versión con soporte, migrar los servicios y datos, y verificar el funcionamiento. Mientras dure la transición, habría que aislar el sistema antiguo y restringir sus conexiones al mínimo necesario.

### 4. De las contramedidas de las partes A y B, ¿cuáles evitan que el atacante encuentre información y cuáles ayudan a detectar que lo está intentando?

| Técnica o contramedida | Tipo | Efecto |
|---|---|---|
| Desactivar servicios innecesarios | Prevención / reducción de exposición | Elimina servicios accesibles que no hacen falta. |
| Bloquear solicitudes de ping | Reducción de exposición | Dificulta el reconocimiento por ICMP, pero no oculta necesariamente los puertos TCP. |
| `DROP` | Filtrado | Descarta el tráfico sin responder explícitamente. |
| `REJECT` | Filtrado | Rechaza el tráfico y puede revelar que la conexión está siendo filtrada. |
| Ocultar banners | Reducción de información | Dificulta conocer la versión, pero no elimina la vulnerabilidad. |
| Registrar paquetes SYN | Detección | Deja registros que pueden ayudar a investigar posibles escaneos; no los bloquea. |
| `whois` y `nslookup` | Reconocimiento | Permiten recopilar información de dominios y resolución de nombres. |
| `nmap -sV` | Exploración | Identifica puertos accesibles e intenta determinar servicios y versiones. |
| Comparar los escaneos antes/después | Verificación | Permite comprobar el efecto de las contramedidas aplicadas. |

Las medidas preventivas intentan limitar la información o los servicios accesibles. Las medidas de registro y supervisión ayudan a detectar o investigar actividad, pero no la impiden por sí solas.

### 5. Conclusión

El escaneo inicial permite conocer la superficie de ataque observable desde `Torrent-Auditor`. El inventario local ayuda a relacionar cada puerto con su proceso y su configuración. Con esa información se decide qué servicios mantener y cuáles desactivar.

La desactivación debe hacerse según el tipo de servicio: un proceso permanente puede requerir detenerlo y quitar su inicio automático; un servicio gestionado por `inetd` o `xinetd` se desactiva en su configuración, sin detener el portero entero. Después hay que verificar los resultados desde dentro y desde fuera de la máquina.

Bloquear el ping, ocultar banners y registrar conexiones son medidas complementarias. La seguridad también requiere mantener el software actualizado, reducir los servicios expuestos y comprobar que los cambios funcionan.

