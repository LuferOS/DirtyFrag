# DirtyFrag (CVE-2026-43284)

DirtyFrag es una bifurcación reforzada (hardened fork) de [diabl0w/DFRoot](https://github.com/diabl0w/DFRoot). Explota CVE-2026-43284 (DirtyFrag), una primitiva de escritura en la caché de páginas del kernel, para cargar un módulo del kernel sin necesidad de un bootloader desbloqueado y sin un kernel personalizado. La bifurcación está orientada a teléfonos Samsung (OneUI, endurecimiento RKP y KDP) y mantiene el diseño agnóstico respecto al gestor SU del proyecto original (upstream).

> [!IMPORTANT]
> El root es **efímero**. El módulo del kernel, `su` y cualquier módulo cargado desaparecen en el momento en que el dispositivo se reinicia. Un reinicio siempre devuelve el teléfono a un estado limpio y sin root, lo cual también es la vía de recuperación si una ejecución sale mal.

## Instalación

1. Instala el APK de DirtyFrag desde [Releases](https://github.com/mitschud/DirtyFrag/releases/latest).
2. Instala un gestor SU que **incluya `libksud.so`**:
   - Samsung - **requerido**:
     - [KernelSU de diabl0w](https://github.com/diabl0w/KernelSU/releases)
   - Otros:
     - [KernelSU](https://github.com/tiann/KernelSU/releases/latest)
     - [KernelSU-Next](https://github.com/KernelSU-Next/KernelSU-Next/releases/latest)
     - [KowSU](https://github.com/KOWX712/KernelSU/releases/latest)
     - [BakaSU](https://github.com/Baka-SU/BakaSU/releases/latest) (anteriormente ReSukiSU)
3. Abre DirtyFrag, elige tu gestor en la tarjeta **SU Manager** y luego presiona **Run exploit**.

> [!WARNING]
> Desde la versión 1.10 (3.2 en upstream), DirtyFrag ya no incluye `ksud`; este se extrae de la propia biblioteca `libksud.so` del gestor seleccionado. Un gestor que no incluya esa biblioteca no puede otorgarte root. El antiguo consejo de instalar una compilación de CI / Actions del gestor ya no es válido.

Una instalación nueva de DirtyFrag obtiene un nuevo UID de Android, por lo que el permiso en el gestor debe otorgarse nuevamente después de cada reinstalación.

<img width="540" height="1170" alt="Screenshot_20261015_220718" src="https://github.com/user-attachments/assets/bd727962-a520-4820-a91a-90147aa84faa" />

## Características

- Elige tu gestor SU dentro de la aplicación
- **Autorun** (inicio automático), oculto tras el modo Experto (Expert mode), con un bloqueo de seguridad que se desactiva a sí mismo después de un intento fallido para que una mala ejecución no te deje en un bucle de reinicios (bootloop)
- **Auto reboot** (reinicio suave) - desde 1.11 (4.1 en upstream) esto funciona con cualquier derivado de KernelSU: `dfroot.ko` se descarga después de la carga tardía y se ejecuta el `soft-reboot` del propio ksud
- **KernelSU Modules** activado/desactivado - funciona sin root, aplicado por el módulo del kernel antes de que inicie ksud
- Protección de particiones RO (Solo Lectura)
- Registro (log) avanzado, con un toque para guardarlo en Descargas
- Píldora de **Odin** en la barra de herramientas: su punto indica si este firmware permitirá que el dispositivo entre en modo descarga (download mode), y al tocarla se muestran los tres factores con los que se toma esa decisión
- Cuando se publica una versión más reciente, el titular mismo se convierte en una píldora verde lima de **Update** (Actualizar) - a un toque del lanzamiento
- No se necesita Shizuku

## Preguntas Frecuentes (FAQ)

**P: El registro dice que la aplicación del parche falló y que mi dispositivo no es vulnerable**

R: Lo sentimos, no hay solución. O bien:
  - el kernel de tu dispositivo es demasiado nuevo
  - el fabricante de tu dispositivo implementó la mitigación oficial (backport)
  - [mitigación accidental](https://github.com/V4bel/dirtyfrag/issues/23#issuecomment-4405314290) (típicamente visto en el kernel 6.1)

**P: El registro dice "SUCCESS" (ÉXITO) o "ksud exited with error" (ksud salió con error), pero no tengo root**

R: Apaga el **Auto reboot** en la tarjeta Autorun si está habilitado - un reinicio suave que el gestor no puede completar deja el módulo cargado nuevamente descargado. De lo contrario, consulta la [discusión 69 en upstream](https://github.com/diabl0w/DFRoot/discussions/69).

**P: Instalé un módulo y ahora no puedo lanzar el root sin que la aplicación se cierre (crash)**

R: Configura la tarjeta **KernelSU Modules** en Disabled (Deshabilitado) y vuelve a ejecutar el root. El interruptor funciona sin root, que es justamente su propósito: es la vía de escape cuando un módulo es lo que está provocando el fallo.

**P: Módulo + Autorun y ahora estoy en un bucle de reinicios (bootloop)**

R: Android generalmente detecta el bootloop y ofrece el Modo Seguro (Safe Mode); de lo contrario, mantén presionado **Bajar Volumen** durante el arranque. Desde el Modo Seguro, desactiva Autorun y configura KernelSU Modules en Disabled, o simplemente desinstala DirtyFrag hasta que estabilices el sistema. Si no puedes avanzar lo suficiente en el sistema, `adb uninstall df.root` desde cualquier PC funciona en compilaciones de usuario (user builds) sin root.

**P: La barra de progreso se queda en 0 por ciento**

R: Ese es el mapeo del marcador de registro de la aplicación, no del exploit. La ventana de ejecución es de 70 segundos - no vuelvas a iniciar una ejecución "lenta", el demonio aún podría estar trabajando.

**P: ¿Cómo elimino el root de forma limpia?**

R: Antes de desrootear, usa un administrador de archivos con root y elimina /data/adb/KernelSU y los residuos de KSUD (NO LA CARPETA ADB EN SÍ).

## Compilación

``sh
JAVA_HOME="<Android Studio jbr>" ./gradlew assembleRelease

El APK se escribe en app/build/outputs/apk/release/dirtyfrag.apk.
Las compilaciones de lanzamiento (Release) se firman con app/keystore.jks, el cual está ignorado en git (la configuración de firma espera el alias dirtyfrag y dirtyfrag para ambas contraseñas). Genera el tuyo antes de compilar:
keytool -genkeypair -v -keystore app/keystore.jks -alias dirtyfrag \
  -keyalg RSA -keysize 2048 -validity 10000 \
  -storepass dirtyfrag -keypass dirtyfrag

Los ocho módulos de kernel precompilados están confirmados en app/src/main/jni/ko/. El repositorio original los ignora en git y los compila por KMI en CI; esta bifurcación los incluye para que un clon simple se pueda compilar sin una cadena de herramientas DDK. Estos son construidos por el flujo de trabajo Build de este repositorio (los trabajos LKMs / <kmi>) a partir del lkm/dfroot.c actual, por lo que deben recompilarse cada vez que ese archivo cambie: ejecuta el flujo de trabajo, descarga los artefactos df-lkm-<kmi> y colócalos en este directorio. Para compilarlos tú mismo, usa el flujo DDK de make / podman del repositorio original.
Cómo funciona
El kernel de Android descifra los paquetes ESP AES-CBC directamente en la caché de páginas de los archivos abiertos para splice(). Al crear IV = AES_ECB_DEC(key, current_content) XOR desired_content, cualquier bloque alineado a 16 bytes en una biblioteca compartida mapeada se puede sobrescribir sin permiso de escritura y sin copy-on-write (copia en escritura).
El exploit utiliza esta primitiva para parchear un shellcode en libc++.so en la caché de páginas del kernel. La siguiente llamada con privilegios a la función interceptada (hooked) ejecuta el shellcode, que carga el módulo del kernel a través de insmod.
Cadena del exploit
 * Transformación IpSec - la aplicación asigna un UdpEncapsulationSocket más un SPI y construye una transformación ESP AES-CBC/HMAC-SHA256 a través de IpSecManager.
 * splicehelper -> crash_dump64 - splicehelper se empalma (spliced) en crash_dump64 usando la primitiva CBC. crash_dump64 se ejecuta en el dominio de SELinux crash_dump, el cual puede abrir archivos etiquetados como vendor_file que untrusted_app no puede leer - el puente que necesitamos.
 * dfroot.ko -> vendor_file - el módulo del kernel se escribe a través de ese puente en un archivo etiquetado como vendor_file.
 * Hook en libc++ (se activa en init, uid=0, tid=1) - el shellcode se parchea en std::ostream::sentry::sentry(). Comprueba el uid y el tid, crea /dev/df como un mutex de un solo uso para que no pueda volver a entrar, y luego ejecuta insmod en el dominio vendor_modprobe.
 * dfroot.ko init - establece SELinux en permisivo, omite DEFEX a través de kprobes donde esté presente y luego usa call_usermodehelper para ejecutar el bootstrap.
 * bootstrap - lee la configuración protegida por el dispositivo desde la aplicación, adopta el entorno de zygote, configura las particiones como de solo lectura, deshabilita opcionalmente todos los módulos y, finalmente, lanza el demonio SU del gestor SU que tengas instalado.
Créditos
 * Proyecto original y cadena del exploit: https://github.com/diabl0w/DFRoot
 * PoC original y código diverso: https://github.com/lsposed/lspromise
 * Módulos de kernel permisivos de SELinux y código diverso: https://github.com/polygraphene/DFReroot
 * Método de socket XFRM sin privilegios: https://github.com/combeng6th/DirtyInit
 * Luis Fernando Guzmán Niño (LuferOS) - Cambios de interfaz y adiciones para esta bifurcación.

