**Proyecto Construye tu propio Sistema Operativo Linux Parte 2: Espacio
de usuario y dispositivos**

**Kevin Andres Usama Trespalacios**

**Código estudiantil: 240220241001**

**Sistemas Operativos**

**Grupo A**

**11/11/2026**

**NÚMERO ASIGNADO: 01**

Inicialmente, se valida que esten activas las 3 opciones en defconfig
para la realización de la actividad usando el comando

cd \~/milinux grep -E \'\^CONFIG\_(DEVTMPFS\|BINFMT_SCRIPT\|PROC_FS)=\'
build-default/.config

![](media/image6.png){width="6.267716535433071in"
height="1.9861111111111112in"}

**Actividad 1.**

**Compilar BusyBox**

**1.1 Descargar y configurar:**

Respuesta de configurar BusyBox (se usa version 1.37.0):

![](media/image26.png){width="6.267716535433071in"
height="2.638888888888889in"}

**1.2 Compilar e instalar en rootfs:**

![](media/image1.png){width="6.819160104986877in"
height="1.4385936132983377in"}

**1.3 Observar el resultado:**

![](media/image13.png){width="5.510416666666667in" height="4.1875in"}

**P1. ¿Cuánto pesa el archivo busybox y cuántos enlaces simbólicos creó
la instalación? Compare con el tamaño de su init de la Parte 1.**

El ejecutable estático busybox tiene un tamaño exacto de 2,526,552 bytes
o 2.5 MB. Durante el proceso de instalación, la herramienta creó un
total de 402 enlaces simbólicos distribuidos dentro de la estructura de
directorios de rootfs. En comparación con el programa init desarrollado
en la Parte 1 cuyo binario estático pesaba 803 KB, busybox es
aproximadamente tres veces más pesado, sin embargo, empaqueta cientos de
utilidades esenciales del sistema de archivos dentro de un único
ejecutable en lugar de limitarse a una rutina básica de arranque.

**P2. ¿A dónde apunta rootfs/bin/ls ? Explique cómo un solo programa
puede comportarse como ls , cp y mount .**

El archivo rootfs/bin/ls apunta directamente mediante un enlace
simbólico al binario central busybox. Este comportamiento se debe a que
Busybox está diseñado como un binario multipropósito. Al ejecutarse un
comando, el programa inspecciona el primer argumento del sistema que
contiene su propio nombre de invocación (argv\[0\]); de esta forma, si
es llamado como ls ejecuta la rutina para listar archivos, mientras que
si es invocado como cp o mount, activa la lógica del applet
correspondiente conservando un uso mínimo de espacio.

**Actividad 2. Armar la jerarquía y escribir /init**

**2.1 Crear las carpetas estándar y la consola.**

Carpetas estándar de rootfs:

![](media/image4.png){width="4.833333333333333in"
height="0.8645833333333334in"}

**2.2 Escribir el script de arranque. Cree \~/milinux/rootfs/init con
este contenido. Reemplace 03 por su número asignado (el mismo de la
Parte 1):**

**NÚMERO ASIGNADO: 01**

![](media/image23.png){width="6.267716535433071in"
height="2.8194444444444446in"}

**2.3 Dar permiso de ejecución y empaquetar. Igual que en la Parte 1,
pero desde rootfs :**

![](media/image32.png){width="6.267716535433071in"
height="1.4722222222222223in"}

**P3. Explique para qué sirve cada carpeta creada en 2.1. ¿Cuáles de
ellas estarán vacías en el archivo rootfs.cpio.gz y se llenarán solo al
arrancar?**

**/dev:** Almacena los archivos o nodos de dispositivos necesarios para
interactuar con el hardware o controladores virtuales.

**/proc:** Punto de montaje para el sistema de archivos virtual proc, el
cual expone información del estado del kernel y los procesos en tiempo
real.

**/sys:** Punto de montaje para sysfs, que expone el modelo unificado de
dispositivos, buses y controladores gestionados por el kernel.

**/etc:** Aloja archivos de configuración globales del sistema.

**/tmp:** Directorio destinado al almacenamiento de archivos temporales
de ejecuciones.

**/root:** Directorio de trabajo personal del usuario administrador
(root).

Las carpetas /proc, /sys y /dev permanecen totalmente vacías dentro del
paquete comprimido rootfs.cpio.gz (salvo por el nodo inicial
dev/console). Su contenido no ocupa espacio físico en disco ni en el
initramfs, el kernel de Linux las completa de manera dinámica en memoria
RAM únicamente al momento del arranque cuando el script /init ejecuta
las instrucciones mount.

**P4. ¿Por qué el script usa exec en la última línea? Prediga qué PID
tendrá la terminal.**

El script utiliza la llamada de sistema exec (exec setsid cttyhack sh)
para reemplazar en memoria la imagen del proceso actual del script por
el intérprete de comandos de BusyBox, manteniendo exactamente el mismo
PID. Como el kernel ejecutó el script /init como el primer programa de
usuario del sistema, al aplicar exec, la terminal hereda esta posición
primordial y asumirá el rol de PID 1.

**Actividad 3. Arrancar y comprobar quién es PID 1**

![](media/image11.png){width="4.84375in" height="1.9166666666666667in"}

**Comandos:**

![](media/image15.png){width="5.177083333333333in" height="9.09375in"}

![](media/image10.png){width="6.267716535433071in"
height="2.763888888888889in"}

**Comando exit:**

![](media/image8.png){width="6.267716535433071in"
height="1.7916666666666667in"}

**P5. ¿Qué PID muestra echo \$\$ ? ¿Coincide con su predicción de P4?
¿Qué procesos aparecen en ps y cuáles tienen el nombre entre
corchetes?**

El comando echo \$\$ devuelve el valor 1, lo que confirma la predicción
hecha, la terminal hereda la posición de PID 1 gracias al reemplazo de
memoria realizado por exec en el script /init. En la salida de ps se
observan en el espacio de usuario únicamente la shell interactiva sh
(PID 1) y el comando ejecutable ps (PID 61). El resto de las entidades
corresponden a procesos cuyos nombres aparecen encerrados entre
corchetes (como \[kthreadd\], \[kworker/\...\], \[ksoftirqd/0\] o
\[kdevtmpfs\]), los cuales representan hilos de ejecución del núcleo que
corren internamente en el espacio del kernel para gestionar tareas del
sistema y no ejecutables del espacio de usuario.

**P6. ¿Qué mensaje apareció al escribir exit ? Relácionelo con la
Actividad 5 de la Parte 1.**

Al ingresar la instrucción exit, el proceso PID 1 (sh) finalizó su
ejecución, provocando de inmediato un fallo crítico en el sistema con el
mensaje:

Kernel panic - not syncing: Attempted to kill init! exitcode=0x00000000.

Este comportamiento concuerda con el de la actividad 5 de la parte 1, el
kernel de Linux requiere la permanencia continua de un proceso
primordial con PID 1 para orquestar el espacio de usuario. Si dicho
proceso finaliza, el núcleo es incapaz de continuar operando y fuerza la
detención segura del sistema operativo mediante un Kernel Panic.

**P7. Copie la salida de mount e identifique los tres sistemas de
archivos que montó su script.**

**Salida de mount:**

rootfs on / type rootfs (rw,size=106984k,nr_inodes=26746)

proc on /proc type proc (rw,relatime)

sysfs on /sys type sysfs (rw,relatime)

devtmpfs on /dev type devtmpfs
(rw,relatime,size=106984k,nr_inodes=26746,mode=755)

Los tres sistemas de archivos virtuales montados dinámicamente por el
script /init son:

**/proc:** Expone la estructura de procesos activos, estado del núcleo y
métricas de rendimiento en tiempo real.

**sysfs en /sys:** Ofrece una representación jerárquica del modelo de
dispositivos, buses y controladores detectados por el sistema.

**devtmpfs en /dev:** Mantiene y gestiona dinámicamente en la memoria
RAM los nodos de dispositivos del sistema.

**Actividad 4. Explorar /proc y /sys**

**4.1 ps depende de /proc . Dentro de su sistema:**

**Comandos:**

![](media/image29.png){width="5.447916666666667in"
height="9.791666666666666in"}

![](media/image25.png){width="4.78125in" height="8.989583333333334in"}

![](media/image34.png){width="4.364583333333333in" height="8.75in"}

**4.2 Leer el estado del sistema. Cada archivo se lee con cat , y el
kernel genera su contenido en ese momento:**

**cat /proc/1/status \| head -12**

![](media/image22.png){width="3.625in" height="3.3645833333333335in"}

**cat /proc/cmdline**

![](media/image19.png){width="2.7708333333333335in"
height="0.7916666666666666in"}

**head -5 /proc/meminfo**

![](media/image12.png){width="3.5416666666666665in"
height="2.2291666666666665in"}

**grep -c processor /proc/cpuinfo**

![](media/image7.png){width="3.4895833333333335in" height="0.46875in"}

**cat /proc/uptime**

![](media/image30.png){width="2.03125in" height="0.8333333333333334in"}

**cat /proc/filesystems**

![](media/image21.png){width="2.8958333333333335in"
height="6.739583333333333in"}

**ls -l /proc/meminfo**

![](media/image9.png){width="6.267716535433071in"
height="0.8611111111111112in"}

**4.3 Recorrer el modelo de dispositivos.**

**ls /sys/class**

![](media/image36.png){width="6.267716535433071in"
height="3.4583333333333335in"}

**ls /sys/class/net**

![](media/image35.png){width="3.0833333333333335in" height="0.9375in"}

**ls /sys/block**

![](media/image18.png){width="5.895833333333333in"
height="0.9583333333333334in"}

**cat /sys/class/net/eth0/address**

![](media/image2.png){width="3.6666666666666665in" height="1.0in"}

**ls /sys/devices/system/cpu**

![](media/image5.png){width="6.267716535433071in"
height="1.5138888888888888in"}

**P8. ¿Qué mostró ps después de umount /proc ? Explique de dónde obtiene
ps la información de los procesos.**

Después de ejecutar umount /proc, el comando ps no mostró ningún proceso
(solo la cabecera con los títulos de las columnas). Esto sucede porque
ps no mantiene una base de datos propia ni consulta directamente a la
CPU, su funcionamiento depende exclusivamente de leer los subdirectorios
numéricos asociados a cada PID dentro del sistema de archivos virtual
/proc. Al desmontar /proc, la interfaz entre el espacio de usuario y el
kernel se deshabilita, impidiendo que ps pueda extraer la lista e
información de los procesos en ejecución.

**P9. En /proc/1/status , identifique Name , State , PPid y VmRSS . ¿Qué
programa es el PID 1 ahora y cuánta memoria física usa?**

Name: sh

State: R (running)

PPid: 0

El programa que se ejecuta actualmente como PID 1 es el shell
interactivo sh de BusyBox.

**P10. ls -l /proc/meminfo muestra tamaño 0, pero cat muestra contenido.
Explique por qué.**

ls -l /proc/meminfo reporta un tamaño de 0 bytes porque los archivos
dentro de /proc son archivos sintéticos o virtuales. No existen
físicamente almacenados en ningún bloque de disco duro ni ocupan
almacenamiento en el initramfs. Cuando se ejecuta cat /proc/meminfo, el
kernel intercepta la llamada al sistema de lectura y una función interna
genera dinámicamente la información del estado de la memoria RAM en ese
preciso instante para entregarla en texto plano al usuario.

**P11. Compare MemTotal de /proc/meminfo con los 256 MB que le dio a
QEMU. ¿Por qué no son iguales?**

/proc/meminfo reporta un MemTotal de 220,072 kB (aproximadamente 214.9
MB), lo cual es menor a los 256 MB (262,144 kB) asignados a la máquina
virtual de QEMU. Esta diferencia ocurre porque durante la etapa temprana
de arranque, el kernel de Linux reserva de forma exclusiva una porción
de la memoria RAM para almacenar su propia imagen ejecutable (bzImage),
sus tablas de páginas, estructuras de datos internas y los búferes
asignados a los controladores de hardware. El valor MemTotal refleja
únicamente la memoria física remanente que queda disponible para la
gestión del sistema operativo y las aplicaciones de usuario.

**P12. ¿Qué interfaces de red y qué dispositivos de bloque aparecen en
/sys ? ¿Quién los creó, si usted no instaló nada?**

Aparecen las siguientes interfaces de red y dispositivos de bloque en
/sys:

**Interfaces de red (ls /sys/class/net):** Aparecen las interfaces eth0
(la tarjeta de red emulada por QEMU), lo (la interfaz de bucle local o
loopback) y sit0 (túnel de IPv6 sobre IPv4).

**Dispositivos de bloque:** Se encuentra el subsistema block en
/sys/class/block o /sys/block.

Fueron creados y mapeados automáticamente en la RAM por el subsistema
sysfs del kernel durante el proceso de detección e inicialización de
hardware (driver model).

**Actividad 5. Archivos de dispositivo: mknod frente a devtmpfs**

**5.1 Ver los dispositivos que creó el kernel.**

**ls -l /dev \| head -20**

![](media/image27.png){width="6.267716535433071in"
height="4.291666666666667in"}

**ls /dev \| wc -l**

![](media/image28.png){width="2.4270833333333335in"
height="0.5833333333333334in"}

**cat /proc/devices**

![](media/image31.png){width="3.625in" height="6.3125in"}

![](media/image17.png){width="2.78125in" height="5.5625in"}

**5.2 Quitar devtmpfs y crear dispositivos a mano. Se usa umount -l
porque la terminal tiene abierto /dev/ttyS0 :**

![](media/image3.png){width="6.267716535433071in"
height="3.4305555555555554in"}

**5.3 Volver a montar devtmpfs .**

![](media/image24.png){width="3.6354166666666665in" height="0.96875in"}

**P13. ¿Cuántos archivos había en /dev con devtmpfs y cuántos quedaron
al desmontarlo? ¿De dónde salió el que quedó?**

Con el sistema de archivos devtmpfs montado en /dev, el kernel
gestionaba automáticamente 111 archivos de dispositivos. Al desmontarlo
mediante la orden umount -l /dev, la lista se redujo a 1 solo archivo.
Este nodo remanente proviene del archivo físico de dispositivo que se
creó manualmente con la orden sudo mknod -m 600 dev/console c 5 1 dentro
de la estructura estática del rootfs guardada en el initramfs.

**P14. /dev/mi_consola no se llama console , pero el texto apareció en
la pantalla. Explique qué identifica realmente a un dispositivo.**

El nombre asignado al archivo en la ruta /dev/mi_consola es simplemente
una etiqueta o alias dentro del sistema de archivos para facilitar la
interacción con el usuario. Lo que identifica e instruye realmente al
kernel sobre cómo gestionar un dispositivo es su tipo (c para carácter o
b para bloque) y su pareja de números mayor (major) y menor (minor). Al
asignarle al archivo /dev/mi_consola los números mayor 5 y menor 1, el
kernel asoció inmediatamente las operaciones de entrada/salida al
controlador de la consola del sistema, imprimiendo la salida en pantalla
sin importar el nombre del archivo.

**P15. Compare crear dispositivos con mknod y usar devtmpfs . ¿Qué
problema tendría un sistema con cientos de dispositivos si los creara a
mano?**

Crear nodos de dispositivo manualmente con mknod exige conocer con
antelación el tipo y la tupla exacta de números mayor/menor de cada
controlador, además de mantener archivos estáticos en disco que ocupan
espacio inútil cuando el hardware no está presente. En un sistema
moderno con cientos de dispositivos o periféricos dinámicos, crearlos a
mano resulta ineficiente, rígido y propenso a errores. En contraste,
devtmpfs delega esta tarea al kernel, permitiendo que cree y destruya
automáticamente los nodos de dispositivo en la memoria RAM a medida que
el hardware es detectado en tiempo real.

**Actividad 6. Entrada/salida e interrupciones**

**6.1 Contar interrupciones del puerto serie. Ejecute el comando,
presione varias teclas sin ejecutar nada y bórrelas, y vuelva a
ejecutarlo:**

**Comando antes y luego comando después de escribir teclas:**

![](media/image16.png){width="3.7708333333333335in"
height="1.7083333333333333in"}

**6.2 Medir dispositivos de carácter.**

![](media/image20.png){width="6.267716535433071in"
height="4.347222222222222in"}

**Cierre. Apague su sistema desde dentro; QEMU termina solo:**

![](media/image14.png){width="3.4270833333333335in"
height="1.1666666666666667in"}

**P16. ¿Cuánto aumentó el contador de ttyS0 al presionar teclas? ¿En qué
línea IRQ está? Explique qué hace el kernel en cada interrupción.**

El contador de la consola serie ttyS0 se encuentra en la linea IRQ 4. Al
presionar teclas en la terminal, el contador aumentó en 334
interrupciones (pasó de 4946 a 5280). En cada interrupción de hardware,
la CPU suspende temporalmente la ejecución del proceso actual y cede el
control a la rutina de atención de interrupciones (Interrupt Service
Routine - ISR) dentro del kernel. Esta rutina lee los datos transmitidos
por el puerto serie, deposita el carácter en el búfer de entrada del
driver TTY para que el shell (sh) pueda procesarlo, incrementa el
contador registrado en /proc/interrupts y restaura el contexto del
procesador para continuar con la ejecución del programa de usuario.

**P17. ¿Qué velocidad reportó el primer dd ? ¿Por qué es tan alta, si no
hay disco involucrado?**

El primer comando reportó una velocidad de 2.0 GB/s. Esta velocidad es
tan alta porque no hay operaciones de entrada/salida físicas sobre
ningún almacenamiento magnético o estado sólido. El dispositivo fuente
/dev/zero genera bloques de ceros directamente desde la memoria RAM del
sistema, mientras que el dispositivo destino /dev/null descarta
inmediatamente todos los bytes que se le escriben. Todo el proceso
ocurre como una transferencia rápida de búferes de memoria gestionada
por la CPU y la RAM virtual.

**P18. /tmp no tiene disco detrás. ¿Dónde se guardaron los 20 MB del
segundo dd ? Use MemFree para justificarlo.**

Los 20 MB del segundo comando se guardaron directamente en la memoria
RAM, ya que el directorio /tmp opera sobre un sistema de archivos en
memoria de tipo tmpfs / ramfs. Justificación con MemFree: Antes de crear
el archivo de 20 MB, el comando head -5 /proc/meminfo reportaba un valor
de MemFree: 205900 kB (205.9 MB). Tras la ejecución del segundo dd, al
consultar nuevamente meminfo se observa que MemFree descendió a 185332
kB (185.3 MB). La diferencia exacta entre ambos valores es de 20,568 kB
(aproximadamente 20.0 MB), lo que demuestra matemáticamente que la
creación del archivo consumió espacio de la memoria RAM disponible.

**Trabajo autónomo:**

1.  **Un árbol de directorios de su rootfs (puede generarlo con find
    rootfs -**

**maxdepth 2 -type d en la VM), con una línea que explique cada
carpeta.**

![](media/image33.png){width="6.267716535433071in"
height="3.1944444444444446in"}

**rootfs/:** Directorio raíz principal que alberga la jerarquía del
sistema de archivos (root filesystem).

**rootfs/root:** Directorio personal de trabajo exclusivo para el
usuario administrador (root).

**rootfs/etc:** Almacena los archivos de configuración globales del
sistema.

**rootfs/sys:** Punto de montaje reservado para sysfs, exponiendo el
modelo de dispositivos, buses y controladores en RAM.

**rootfs/tmp:** Directorio reservado para el almacenamiento de archivos
y búferes temporales en memoria RAM.

**rootfs/usr:** Jerarquía secundaria que contiene subdirectorios de
utilidades y aplicaciones (usr/bin, usr/sbin).

**rootfs/usr/bin:** Contiene ejecutables de usuario no esenciales
instalados por el sistema de empaquetado.

**rootfs/usr/sbin:** Aloja binarios de administración del sistema no
esenciales.

**rootfs/bin:** Almacena los binarios esenciales del espacio de usuario
provistos por BusyBox (como ls, sh, cat).

**rootfs/proc:** Punto de montaje reservado para el sistema de archivos
virtual proc, utilizado por el kernel para exponer la información de
procesos, memoria e interrupciones.

**rootfs/sbin:** Contiene los binarios esenciales para la administración
del sistema y tareas de arranque.

**rootfs/dev:** Punto de montaje reservado para devtmpfs, destinado a la
creación y gestión dinámica de archivos de dispositivo en RAM (alberga
el nodo inicial estático dev/console).

### 

2.  **Una tabla con los tres montajes de /init : qué sistema de archivos
    es, qué información expone y qué comando de la clase dejó de
    funcionar o funcionó gracias a él.**

  ------------------------------------------------------------------------
  Punto de    Sistema de   Información que expone   Impacto en comandos
  montaje     archivos                              del laboratorio
  ----------- ------------ ------------------------ ----------------------
  /proc       proc         Expone información       Permitió el
                           dinámica de los procesos funcionamiento del
                           activos, métricas de     comando ps. Sin /proc
                           memoria RAM              montado, ps no puede
                           (/proc/meminfo),         listar la tabla de
                           parámetros de arranque   procesos del sistema.
                           (/proc/cmdline) y        
                           contadores de            
                           interriupciones          
                           (/proc/interrupts).      

  /sys        sysfs        Expone la jerarquía del  Permitió consultar el
                           modelo de dispositivos,  estado del hardware y
                           buses, controladores     la interfaz de red
                           cargados, interfaces de  emulada con ls
                           red (net) y dispositivos /sys/class/net y
                           de almacenamiento de     obtener la dirección
                           bloque (block).          MAC.

  /dev        devtmpfs     Gestiona en tiempo real  Permitió el
                           los nodos y archivos de  funcionamiento de dd
                           dispositivo (de carácter mediante la
                           c y bloque b) creados    lectura/escritura en
                           dinámicamente por el     nodos sintéticos como
                           kernel.                  /dev/zero y /dev/null
                                                    sin tener que crearlos
                                                    a mano.
  ------------------------------------------------------------------------

3.  **Comentarios dentro de su script init , una línea por instrucción,
    con sus propias palabras.**

**\**
#!/bin/sh

\# Especifica que el intérprete de comandos \'sh\' provisto por BusyBox
ejecutará este script.

export PATH=/sbin:/usr/sbin:/bin:/usr/bin

\# Define las rutas donde el sistema buscará los programas ejecutables
al escribir comandos.

mount -t proc proc /proc

\# Monta en /proc el sistema virtual que expone la información de
procesos y memoria del kernel

mount -t sysfs sysfs /sys

\# Monta en /sys el sistema virtual que mapea la jerarquía de
dispositivos y controladores

mount -t devtmpfs devtmpfs /dev

\# Monta devtmpfs en /dev para que el kernel gestione dinámicamente los
nodos de dispositivos en la RAM

hostname milinux-01

\# Asigna el nombre \'milinux-01\' como el nombre de host de este
sistema emulado.

echo

\# Imprime una línea en blanco en la consola para separar los mensajes
del arranque.

echo \"=== Mi Linux 01: espacio de usuario con BusyBox ===\"

\# Muestra en la pantalla el banner principal que identifica tu espacio
de usuario.

echo \"Kernel: \$(uname -r) \| Procesos: \$(ls -d /proc/\[0-9\]\* \| wc
-l)\"

\# Lee la versión del kernel compilado y cuenta las carpetas numéricas
en /proc para reportar los procesos activos

echo

\# Imprime otra línea vacía para dejar limpio el formato del texto.

exec setsid cttyhack sh

\# Reemplaza este script por la shell interactiva con terminal asignada,
asumiendo el PID 1 definitivo.
