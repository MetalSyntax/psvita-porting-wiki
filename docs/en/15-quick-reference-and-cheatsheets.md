# 15. Quick Reference & Cheat Sheets

Fast reference lookup tables for JNI type signatures, common VitaOS kernel errors, and Bionic-to-Newlib translation mappings.

---

## 🔡 Master JNI Type Signature Table

| Java Type | Signature Symbol | C / C++ Type | Bit Width |
|---|---|---|---|
| `boolean` | `Z` | `jboolean` | 8-bit unsigned |
| `byte` | `B` | `jbyte` | 8-bit signed |
| `char` | `C` | `jchar` | 16-bit UTF-16 unsigned |
| `short` | `S` | `jshort` | 16-bit signed |
| `int` | `I` | `jint` | 32-bit signed |
| `long` | `J` | `jlong` | 64-bit signed |
| `float` | `F` | `jfloat` | 32-bit IEEE 754 |
| `double` | `D` | `jdouble` | 64-bit IEEE 754 |
| `void` | `V` | `void` | 0 |
| `java.lang.String` | `Ljava/lang/String;` | `jstring` | Pointer |
| `java.lang.Class` | `Ljava/lang/Class;` | `jclass` | Pointer |
| `int[]` | `[I` | `jintArray` | Pointer |
| `byte[]` | `[B` | `jbyteArray` | Pointer |
| `String[]` | `[Ljava/lang/String;` | `jobjectArray` | Pointer |

---

## ⚠️ Common VitaOS Error Codes

| Hex Code | SDK Error Constant | Typical Cause in Ports | Recommended Fix |
|---|---|---|---|
| **`0x80010018`** | `SCE_KERNEL_ERROR_NO_MEMORY` | Exhausted RAM or CDRAM partition. | Halve texture resolutions or move secondary textures from CDRAM to main RAM. |
| **`0x8010113D`** | `SCE_SBL_ERROR_COMMON_INVALID_FORMAT` | Invalid LiveArea images during VPK installation. | Ensure `icon0.png`, `bg0.png`, `startup.png` are 8-bit indexed PNGs under allowed file sizes. |
| **`0x800200CB`** | `SCE_KERNEL_ERROR_ILLEGAL_MEMBLOCK_TYPE` | Attempted executable memory allocation without `kubridge`. | Install `kubridge.skprx` in `ur0:tai/config.txt`. |
| **`0x80020002`** | `SCE_ERROR_ERRNO_ENOENT` | File or directory not found. | Check path sanitization; confirm files reside in `ux0:data/<game>/`. |
| **`0x800200D9`** | `SCE_KERNEL_ERROR_CAN_NOT_ALLOC` | Kernel memory block descriptor exhaustion. | Use reusable memory pools instead of hundreds of individual `sceKernelAllocMemBlock` calls. |

---

## 🔄 Bionic libc -> VitaSDK / Newlib Translation Table

| Android Function (Bionic) | Role in Android | PS Vita Handling (VitaSDK) |
|---|---|---|
| `__android_log_print` | Writes to logcat. | Redirect to custom logger writing to `game.log` (with `fflush`) and `sceClibPrintf`. |
| `dlopen(path, flags)` | Loads a dynamic shared library. | Route to internal loader (`so_file_load`). |
| `dlsym(handle, name)` | Resolves function pointer. | Route to `so_resolve(&so_mod, name)`. |
| `pthread_create` | Spawns a POSIX thread. | Supported by VitaSDK POSIX threads or mapped to `sceKernelCreateThread`. |
| `mprotect` | Alters memory page protections. | Unnecessary if memory was allocated as `USER_RWX` via `kubridge`. |
| `gettimeofday` | Queries UNIX timestamp. | Supported by VitaSDK Newlib. |
| `epoll_create` / `inotify` | Linux async events. | Not directly supported; replace with polling worker thread or dummy descriptor. |
