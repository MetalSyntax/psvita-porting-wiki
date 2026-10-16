# 15. Referencia Rápida y Cheat Sheets

Esta sección compila tablas de consulta rápida para firmas JNI, códigos de error habituales de PlayStation Vita y equivalencias de llamadas del sistema de Android Bionic libc hacia VitaSDK.

---

## 🔡 Tabla Maestra de Firmas de Tipos JNI

| Tipo en Java | Símbolo de Firma JNI | Tipo en C / C++ | Tamaño en Bits |
|---|---|---|---|
| `boolean` | `Z` | `jboolean` | 8 bits (unsigned) |
| `byte` | `B` | `jbyte` | 8 bits (signed) |
| `char` | `C` | `jchar` | 16 bits (UTF-16 unsigned) |
| `short` | `S` | `jshort` | 16 bits (signed) |
| `int` | `I` | `jint` | 32 bits (signed) |
| `long` | `J` | `jlong` | 64 bits (signed) |
| `float` | `F` | `jfloat` | 32 bits (IEEE 754) |
| `double` | `D` | `jdouble` | 64 bits (IEEE 754) |
| `void` | `V` | `void` | 0 |
| `java.lang.String` | `Ljava/lang/String;` | `jstring` | Puntero |
| `java.lang.Class` | `Ljava/lang/Class;` | `jclass` | Puntero |
| `int[]` | `[I` | `jintArray` | Puntero |
| `byte[]` | `[B` | `jbyteArray` | Puntero |
| `String[]` | `[Ljava/lang/String;` | `jobjectArray` | Puntero |

---

## ⚠️ Códigos de Error Frecuentes de VitaOS

| Código Hexadecimal | Nombre del Error en SDK | Causa Habitual en Ports | Solución Recomendada |
|---|---|---|---|
| **`0x80010018`** | `SCE_KERNEL_ERROR_NO_MEMORY` | Se agotó la memoria en la partición solicitada (RAM o CDRAM). | Reducir la resolución de texturas a la mitad o mover texturas secundarias de CDRAM a la RAM principal. |
| **`0x8010113D`** | `SCE_SBL_ERROR_COMMON_INVALID_FORMAT` | Imágenes de LiveArea inválidas al instalar el VPK. | Asegurar que `icon0.png`, `bg0.png`, `startup.png` sean PNG indexados a 8 bits y no excedan el tamaño en KiB permitido. |
| **`0x800200CB`** | `SCE_KERNEL_ERROR_ILLEGAL_MEMBLOCK_TYPE` | Intento de asignar memoria ejecutable sin `kubridge`. | Instalar el plugin de kernel `kubridge.skprx` en `ur0:tai/config.txt`. |
| **`0x80020002`** | `SCE_ERROR_ERRNO_ENOENT` | Archivo o directorio no encontrado. | Revisar que la ruta comience por `ux0:data/<juego>/` y que los archivos existan en la tarjeta física. |
| **`0x800200D9`** | `SCE_KERNEL_ERROR_CAN_NOT_ALLOC` | Límite máximo de descriptores de bloques de memoria alcanzado por el OS. | Reutilizar pools de memoria en lugar de crear cientos de `sceKernelAllocMemBlock` individuales. |

---

## 🔄 Tabla de Correspondencia Bionic libc -> VitaSDK / Newlib

| Función Android (Bionic) | Comportamiento en Android | Equivalente en PS Vita (VitaSDK) |
|---|---|---|
| `__android_log_print` | Escribe en logcat de Android. | Reemplazar por función propia que escriba a archivo de texto con `fflush` y a `sceClibPrintf`. |
| `dlopen(path, flags)` | Carga una librería dinámica del sistema. | Mapear al cargador interno (`so_file_load`). |
| `dlsym(handle, name)` | Resuelve la dirección de una función. | Mapear a `so_resolve(&so_mod, name)`. |
| `pthread_create` | Crea un hilo nativo POSIX. | Soportado por VitaSDK POSIX threads o mapeado a `sceKernelCreateThread`. |
| `mprotect` | Cambia permisos de páginas de memoria. | Innecesario si el bloque inicial se reservó como `USER_RWX` con `kubridge`. |
| `gettimeofday` | Obtiene el tiempo UNIX actual. | Totalmente soportado por la Newlib de VitaSDK. |
| `epoll_create` / `inotify` | Eventos asíncronos de Linux. | No soportado directamente; simular mediante sondeo periódico en hilo worker o devolver descriptor dummy. |
