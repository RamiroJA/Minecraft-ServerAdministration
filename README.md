# Minecraft-ServerAdministration
El repositorio incluye toda la informacion para levantar, configurar y administrar con tareas programadas un servidor de minecraft, utilizando una VPS y una notebook de bajos recursos.

## Pasos previos

En primer lugar hay que pensar que recursos tenemos para alojar el servidor de Minecraft, esto incluye el hardware de la PC que va a alojar el servidor, tambien hay que revisar si nuestro proveedor de internet nos permite abrir puertos para darle salida a internet al servidor y otro detalle a tener en cuenta es si nuestra PC se trata de una PC de escritorio o una notebook/laptop. 

Una vez hecho el analisis podemos definir que recursos vamos a necesitar para alojar nuestro servidor. 

En mi caso cuento con una notebook de bajos recursos y mi proveedor de internet no me permite abrir los puertos para darle salida al servidor, por lo que en este instructivo vamos a abarcar uno de los casos en los cuales hay que conseguir y configurar mas cosas para lograr levantar el servidor de Minecraft y mantenerlo estable en el tiempo.

## Mi setup

### Notebook Lenovo IdeaPad S145-15IGM
- Procesador: Intel(R) Celeron(R) N4000 CPU @ 1.10GHz x2
- GPU integrada: Mesa Intel UHD Graphics 600
- Memoria RAM: 8GB DDR4 a 2400MB/S
- Disco SSD: WDC PC SN530 de 256 GB
- S.O: Linux Mint 21.3

### VPS Hostinger tier KVM 1
- Nucleos de CPU: 1
- Memoria RAM: 4GB
- Espacio en disco: 50 GB
- S.O: Ubuntu 24.04 LTS


## Primeros pasos

En primer lugar tenemos que configurar tanto nuestra notebook (servidor) como el VPS para poder realizar un tunel SSH (proxy reverso), esto nos va a permitir habilitar un puerto para que el servidor de minecraft alojado en la notebook tenga salida a internet a traves del VPS.

En primer lugar debemos instalar la dependencia `ufw` tanto en el servidor como en el VPS, con el siguiente comando.

`sudo apt install ufw`

Supongamos que vamos a utilizar el puerto `19072`, entonces en el VPS deberiamos habilitar el puerto `19072` tanto en el firewall utilizando `ufw` como en `iptables` utilizando los siguientes comandos.

En el servidor y en el VPS:

1. `sudo iptables -A INPUT -p tcp --dport 19072 -j ACCEPT`

2. `iptables-save`

3. `sudo ufw enable`

4. `ufw allow 19072`

5. `ufw allow 19072/tcp`

6. `systemctl restart ssh`

Una vez configurado el puerto `19072` entonces podremos realizar el tunel SSH (proxy reverso) con el siguiente comando en la notebook servidor.

`ssh -N -R 19072:localhost:19072 [usuario-de-VPS]@[IP-de-VPS]`

Con esto vamos a lograr que el puerto `19072` de nuestra notebook servidor tenga salida a internet a traves del VPS.

## Descarga  y configuracion de servidor

Una vez que descarguemos el archivo .jar de servidor de Minecraft de la version que necesitamos debemos crear un script para inicializar el servidor, añadiendo parametros para ejecutarlo de forma mas eficiente y tambien ejecutarlo utilizando `screen`, esto nos permite crear una sesion de terminal particular para el servidor de Minecraft.

Tanto el script .sh como el archivo .jar deben ubicarse dentro de una misma carpeta.

### Script utilizado

### minecraft.sh
```
#!/bin/bash

SERVER_JAR="[ruta de archivo .jar de servidor]"

screen -S minecraft java -server -Xms5G -Xmx5G -XX:+UnlockExperimentalVMOptions -XX:+UnlockDiagnosticVMOptions -XX:+AlwaysActAsServerClassMachine -XX:+AlwaysPreTouch -XX:+DisableExplicitGC -XX:+UseNUMA -XX:NmethodSweepActivity=1 -XX:ReservedCodeCacheSize=400M -XX:NonNMethodCodeHeapSize=12M -XX:ProfiledCodeHeapSize=194M -XX:NonProfiledCodeHeapSize=194M -XX:-DontCompileHugeMethods -XX:MaxNodeLimit=240000 -XX:NodeLimitFudgeFactor=8000 -XX:+UseVectorCmov -XX:+PerfDisableSharedMem -XX:+UseFastUnorderedTimeStamps -XX:+UseCriticalJavaThreadPriority -XX:ThreadPriorityPolicy=1 -XX:AllocatePrefetchStyle=3 -XX:+UseG1GC -XX:MaxGCPauseMillis=37 -XX:G1HeapRegionSize=16M -XX:G1NewSizePercent=23 -XX:G1ReservePercent=20 -XX:SurvivorRatio=32 -XX:G1MixedGCCountTarget=3 -XX:G1HeapWastePercent=20 -XX:InitiatingHeapOccupancyPercent=10 -XX:G1RSetUpdatingPauseTimePercent=0 -XX:MaxTenuringThreshold=1 -XX:G1SATBBufferEnqueueingThresholdPercent=30 -XX:G1ConcMarkStepDurationMillis=5.0 -XX:G1ConcRSHotCardLimit=16 -XX:G1ConcRefinementServiceIntervalMillis=150 -XX:GCTimeRatio=99 -jar "$SERVER_JAR" nogui
```
Ejecutando el script **minecraft.sh** por primera vez observaremos que se van a crear nuevos archivos dentro de la carpeta del servidor, debemos detectar el archivo `eula.txt` y `server.properties` y editarlos.

### eula.txt
```
#By changing the setting below to TRUE you are indicating your agreement to our EULA (https://aka.ms/MinecraftEULA).
#DDD MMM AA HH:MM:SS GMT 202x
eula=false
```

Lo unico que debemos hacer es cambiar la clave eula=false por el valor `true`.

### server.properties (ejemplo)

```
#Minecraft server properties
#Fri May 02 07:05:31 ART 2025
accepts-transfers=false
allow-flight=false
allow-nether=true
broadcast-console-to-ops=true
broadcast-rcon-to-ops=true
bug-report-link=
difficulty=normal
enable-command-block=false
enable-jmx-monitoring=false
enable-query=false
enable-rcon=false
enable-status=true
enforce-secure-profile=false
enforce-whitelist=false
entity-broadcast-range-percentage=100
force-gamemode=false
function-permission-level=2
gamemode=survival
generate-structures=true
generator-settings={}
hardcore=false
hide-online-players=false
initial-disabled-packs=
initial-enabled-packs=vanilla
level-name=world
level-seed=
level-type=minecraft\:normal
log-ips=true
max-chained-neighbor-updates=1000000
max-players=20
max-tick-time=60000
max-world-size=29999984
motd=A minecraft server
network-compression-threshold=256
online-mode=false
op-permission-level=3
pause-when-empty-seconds=500
player-idle-timeout=0
prevent-proxy-connections=false
pvp=true
query.port=25565
rate-limit=0
rcon.password=
rcon.port=25575
region-file-compression=deflate
require-resource-pack=false
resource-pack=
resource-pack-id=
resource-pack-prompt=
resource-pack-sha1=
server-ip=
server-port=25565
simulation-distance=8
spawn-monsters=true
spawn-protection=16
sync-chunk-writes=true
text-filtering-config=
text-filtering-version=0
use-native-transport=true
view-distance=24
white-list=true
```

En este caso debemos cambiar dos claves: 

- online-mode=true por el valor `false` (Esto permitira a usuarios con minecraft pirata entrar al servidor).
- server-port=25565 por el valor `19072` (Esto cambia el puerto del servidor al 19072).


Una vez hecho los cambios en los archivos podemos lanzar el script **minecraft.sh** nuevamente y el servidor se inicializara, luego podemos ejecutar el comando `ssh -N -R 19072:localhost:19072 [usuario-de-VPS]@[IP-de-VPS]` y el servidor ya va a tener salida a internet, para conectarse al servidor se va a usar el siguiente formato `[IP-del-VPS]:19072`.

Hasta esta parte del manual ya logramos crear un servidor, con comandos JAVA optimizados y con salida a internet, desde aca ya se podria jugar. El problema radica en que la administracion del servidor se va a tener que hacer de forma manual **(reiniciar el servidor periodicamente, reiniciar la notebook de forma periodica, refrescar el tunel SSH y realizar backups periodicos).**

## Administracion del servidor

Como se menciono anteriormente vamos a necesitar reiniciar tanto la notebook como el servidor de minecraft de forma periodica, esto va a evitar bajones de performance, de igual manera con el tunel SSH, ademas necesitamos crear una estructura de backups periodicos. 

Para lograr esto vamos a utilizar tareas programadas con `cron`.

Esto lo podemos hacer con el comando `crontab -e`. Hay que tener en cuenta se pueden crear tareas de `cron` tanto para el usuario `root` como para el usuario regular del servidor, para por ejemplo reiniciar la PC necesitaremos permisos `root`, para hacerlo usaremos `sudo crontab -e`.

Usando `sudo crontab -e` añadiremos las siguientes lineas.
```
1 7 * * * /sbin/shutdown -r now

3 7 * * * cpufreq-set -c 0,1 -g performance
```

- La primera linea lo que va a hacer es reiniciar la PC a las 07:01 am todos los dias.

- La segunda linea va a setear los nucleos del cpu al governor de `performance` a las 07:03 am todos los dias, esto con el objetivo de que el CPU utilice toda su potencia sin restricciones mientras ejecute el servidor, resultando en una mejor performance.

Luego usando `crontab -e` añadiremos las siguientes lineas

```
0 9 * * * rm -rf /home/SERVER-BACKUP/diario/mañana/SERVER && cp -rf /home/[your-user]/Desktop/SERVER /home/[your-user]/Desktop/SERVER-BACKUP/diario/mañana

0 21 * * * rm -rf /home/SERVER-BACKUP/diario/noche/SERVER && cp -rf /home/[your-user]/Desktop/SERVER /home/[your-user]/Desktop/SERVER-BACKUP/diario/noche

0 3 * * * rm -rf /home/SERVER-BACKUP/semanal/$(date +\%A) && cp -rf /home/[your-user]/Desktop/SERVER /home/[your-user]/Desktop/SERVER-BACKUP/semanal/$(date +\%A)/

0 4 * * 3 rm -rf /home/[your-user]/.local/share/Trash/files/ && cd /home/[your-user]/.local/share/Trash/ && mkdir files && chmod 700 files

0 7 * * * screen -S minecraft -X stuff "stop$(printf '\r')"

4 7 * * * DISPLAY=:0 XAUTHORITY=/home/[your-user]/.Xauthority bash -c 'xfce4-terminal --title="TunelSSH" --geometry=80x5 --command="ssh -N -R 19071:localhost:19072 root@167.88.33.72" & sleep 5 && wmctrl -r TunelSSH -b add,hidden'

5 7 * * * cd /home/[your-user]/Desktop/SERVER && DISPLAY=:0 XAUTHORITY=/home/[your-user]/.Xauthority bash -c 'xfce4-terminal --hold --title="MinecraftServer" --command="sh minecraft.sh" & sleep 5 && wmctrl -r MinecraftServer -b add,maximized_vert,maximized_horz'

```

- La primer linea va a crear un backup del servidor a las 9 am todos los dias, el comando tambien borrara el backup anterior con el objetivo de no acumular backups innecesarios y utilizar demasiado espacio en disco.

- La segunda y tercer linea hacen lo mismo que la primera, pero a las 9 pm todos los dias y a las 3 am todos los dias respectivamente, la tercer linea guarda un backup diario, de lunes a domingo en carpetas separadas.

- La cuarta linea va a borrar el directorio que acumula los archivos borrados, simil a la papelera de reciclaje. Esto lo hara todos los dias miercoles a las 4 am.

- La quinta linea va a ingresar el texto "stop" a la sesion minecraft de `screen` a las 07:00 am todos los dias. Esto es especialmente util porque al ingresar el texto stop en la sesion **minecraft** de `screen` lo que se logra es parar el servidor de minecraft de manera segura, guardando el estado de ese momento.

- La sexta linea sirve para iniciar el tunel SSH de forma automatica en una ventana de terminal xfce-4, acto seguido la minimiza para no ocupar espacio en la pantalla de forma innecesaria, esto lo hace todos los dias a las 07:04 am.

- La septima linea lo que hara es correr el script `minecraft.sh` en una sesion de `screen` llamada **minecraft** (esto lo sabemos porque el script `minecraft.sh` comienza con `screen -S minecraft`). Esto permitira que el servidor se inicie de forma automatica todos los dias a las 07:05 am. 

**Revisando los puntos podemos ver que el servidor estara inactivo todos los dias entre las 07:00 am y las 07:05 am.** 

Hasta esta parte de la guia ya pudimos configurar el servidor notebook y el VPS para poder alojar el servidor de minecraft, pudimos configurar el servidor de minecraft para correr de forma eficiente y pudimos automatizar las tareas de backup y administracion del servidor.

## Configuracion de bateria (solo para usuarios de notebook).

**Esta configuracion sirve para mi modelo de notebook, aunque no deberia variar demasiado entre proveedores cada uno debera investigar dependiendo su modelo.**

Si estamos alojando el servidor en una notebook/laptop y necesitamos que el servidor este 24hs disponible entonces hay que asumir que la notebook estara enchufada a la corriente todo el dia.

Pensando en este escenario entonces es prudente limitar la carga de la bateria a un umbral entre 60% u 80%, esto va a servir para que la bateria no se degrade con el tiempo de forma tan rapida.

Para lograrlo hay que usar los siguientes comandos 

`sudo apt install tlp tlp-rdw`

`sudo apt install tp-smapi-dkms`

`sudo nano /etc/tlp.conf` Esto sirve para ajustar el valor STOP_CHARGE_THRESH_BAT0=60, ajustando el limite maximo de bateria al 60%, es decir cuando la bateria alcance este valor dejara de cargarse aunque permanezca enchufada.

`sudo tee /sys/bus/platform/drivers/ideapad_acpi/VPC2004\:00/conservation_mode <<< 1` Esto sirve para habilitar el modo conservacion de bateria y habilitar el limite maximo de 60% configurado en el paso anterior.

`sudo systemctl restart tlp`

Con esta configuracion la bateria se dejara de cargar cuando llegue al 60% logrando que la bateria no se degrade a pesar de estar conectada a la corriente 24hs.

Si queremos ver mas detalles de la bateria podemos correr `sudo tlp-stat -b` y `upower -i $(upower -e | grep battery)`.

## Mods utilizados

En mi caso utilizo la version de Minecraft 1.21.5 con la ultima version de `fabric`.

- Chestprotection (Sirve para proteger los cofres, solo lo puede abrir el dueño si lo configura).

- Chunky (Mod para precargar porciones grandes de mapa, sirve para aumentar la performance del servidor).

- Easyauth (Este mod sirve para que los jugadores al ingresar al servidor tengan que registrarse y utilizar una contraseña, es una buena medida de seguridad).

- Easywhitelist (El mod sirve para administrar la whitelist del servidor desde la consola con el servidor en vivo de manera facil, es otra medida de seguridad).

- FabricAPI (Contiene dependencias para que puedan funcionar algunos de los mods de fabric).

- FallingTree (Sirve para talar arboles de forma instantanea usando un hacha).

- JourneyMap (Es un mod para integrar un mapa en pantalla y un menu de mapa en la partida, este mod debe ser instalado tanto en el servidor como en el cliente para funcionar).

- Lithium (Mod para optimizacion de servidores, contiene configuraciones varias para aumentar la performance).

- SkinRestorer (El mod sirve para que los usuarios pirata puedan contar con skins cuando se conecten al servidor).

- Spark (Mod de administracion de networking del servidor, sirve por ejemplo para monitorear el ping del servidor y de los jugadores).

- UnlockAllRecipes (Mod para tener todas las recetas desbloqueadas en el menu de crafteo).

---
