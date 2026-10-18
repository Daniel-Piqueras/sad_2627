| # |	Incidente en Torrent| Tipo | ¿Se replica solo? | ¿Necesita al usuario? |
|---|---------------------|------|-------------------|-----------------------|
| 1 | Un socio abre factura_marzo.pdf.exe; poco después todo el disco compartido está cifrado y aparece un mensaje pidiendo 800 € | Ransomware | No | Si |
| 2 | Un "optimizador gratuito" de PC que, además, envía todo lo que se teclea a un servidor externo | trollano con key logger | No | Si |
| 3 | Sin que nadie haga nada, un programa salta de portátil en portátil a través de la wifi del estudio | Whorm | Si | No |
| 4 | Al abrir ciertos ejecutables, éstos infectan a su vez a otros ficheros del equipo | Virus | Si | Si |
| 5 | Tras instalar un editor de PDF gratuito, aparecen ventanas de publicidad constantemente | Adware | No | Si |
| 6 | Un antiguo colaborador dejó un código que borrará los planos el 31 de diciembre | bomba logica | No | No |
| 7 | El ordenador del estudio está enviando spam sin que nadie lo sepa, bajo las órdenes de un servidor externo | zombie | No | No |
| 8 | Un conjunto de programas modificados que esconde procesos y ficheros del intruso dentro del propio sistema | Rootkit | No | Si |
| 9 | Un servicio escuchando en un puerto que permite entrar al equipo sin contraseña | bakdoor | No | No |
| 10 | Correo: "Tu banco necesita que confirmes tus datos en este enlace" | Phising | No | Si1 |
| 11 | Mensaje: "Reenvía esto a 10 contactos o tu cuenta se cerrará" | Bulo | No | Si |
| 12 | Una web idéntica a la del ERP a la que se llega porque el DNS ha sido manipulado | pharming | No | No |
| 13 | Un programa que registra sin permiso los hábitos de navegación y los vende | spyware | No | Si |

¿Cuáles de estos incidentes no son "malware" propiamente dicho, sino fraude o ingeniería social? ¿Por qué un antivirus no los detendría?
| Nº | Incidente | Tipo | ¿Por qué no es malware propiamente dicho? | ¿Por qué un antivirus podría no detenerlo? |
|---:|---|---|---|---|
| 10 | Correo que pide confirmar los datos bancarios mediante un enlace | Phishing | Es una técnica de fraude que engaña al usuario para que revele información confidencial. | El correo y el enlace pueden no contener código malicioso. El usuario puede introducir sus datos voluntariamente en una web falsa. |
| 11 | Mensaje que pide reenviarlo a 10 contactos para evitar el cierre de una cuenta | Bulo (hoax) | Es un engaño basado en información falsa y en la manipulación del comportamiento del usuario. | No tiene por qué incluir archivos ni código malicioso. Un antivirus no suele detectar ni corregir la información falsa de un mensaje. |
| 12 | Web idéntica a la del ERP a la que se llega porque el DNS ha sido manipulado | Pharming | Es una técnica de fraude que redirige al usuario a una web falsa mediante la manipulación de la resolución de nombres. | La web falsa puede no contener malware. El antivirus podría no detectar la manipulación del DNS ni bloquear una página fraudulenta desconocida. |
