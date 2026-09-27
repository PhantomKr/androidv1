# Política de seguridad del cliente Android

## Permitido

- OpenGL ES y EGL.
- Lectura de recursos desde la carpeta `Data` elegida por el usuario.
- Sockets TCP/UDP hacia los servidores configurados por el usuario.
- Carga de modelos `BMD`, texturas `OZJ/OZT` y archivos de configuración.
- Almacenamiento privado de Android necesario para caché local.

## Excluido

No se debe portar ni enlazar ningún componente de estas familias:

```text
CBGAMEGUARD
APICB
Protect
ProtectSysKey
NProtect
XShield
```

Tampoco se deben incluir bibliotecas nativas o archivos `.so`/`.aar` de origen desconocido.

## Revisión obligatoria de código importado

Antes de integrar código del cliente Android externo, buscar:

```text
Runtime.exec
ProcessBuilder
su
chmod
rm -rf
/system
/data/adb
ptrace
kill
fork
execve
dlopen
```

También revisar todas las URLs, dominios, permisos del `AndroidManifest.xml`, tareas en segundo plano y descargas automáticas.

## Regla de integración

El código importado debe entrar primero en una rama o copia separada, compilarse sin la carpeta `Data` y revisarse antes de conectarlo al servidor real.
