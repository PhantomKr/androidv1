# Arquitectura modular del cliente MU Online Android

Este proyecto se mantiene sin la carpeta `Data` del usuario. Los recursos se leen desde el almacenamiento configurado en `client_runtime.ini`.

## Estructura objetivo

```text
android/app/src/main/cpp/
├── android_main.cpp                 # Punto de entrada Android, futuro archivo pequeño
├── android_bootstrap.cpp            # Código actual; se reduce gradualmente
├── CMakeLists.txt
├── core/
│   ├── GameApplication.cpp/.h       # Ciclo principal y cambio de escenas
│   ├── GameState.cpp/.h             # Estado global no gráfico
│   └── SceneId.h                    # Identificadores de escenas
├── platform/
│   ├── AndroidPlatform.cpp/.h       # Android, EGL, ciclo de vida
│   ├── AndroidInput.cpp/.h           # Eventos táctiles
│   └── AndroidStorage.cpp/.h         # Rutas y almacenamiento
├── renderer/
│   ├── RenderBackend.cpp/.h         # Interfaz gráfica
│   ├── OpenGLESRenderer.cpp/.h       # OpenGL ES 3
│   ├── ShaderProgram.cpp/.h
│   └── TextureRenderer.cpp/.h
├── resources/
│   ├── ResourceManager.cpp/.h       # Caché y ciclo de vida
│   ├── BmdLoader.cpp/.h             # Modelos BMD
│   ├── OzjLoader.cpp/.h              # Texturas OZJ
│   └── OztLoader.cpp/.h              # Texturas OZT
├── scenes/
│   ├── LoadingScene.cpp/.h
│   ├── LoginScene.cpp/.h
│   ├── ServerSelectScene.cpp/.h
│   ├── CharacterSelectScene.cpp/.h
│   └── GameScene.cpp/.h
├── game/
│   ├── Map.cpp/.h
│   ├── Character.cpp/.h
│   ├── Animation.cpp/.h
│   └── Player.cpp/.h
├── network/
│   ├── NetworkClient.cpp/.h
│   ├── PacketReader.cpp/.h
│   └── PacketWriter.cpp/.h
├── ui/
│   ├── Hud.cpp/.h
│   ├── Button.cpp/.h
│   └── TouchControls.cpp/.h
└── security/
    ├── SECURITY_POLICY.md
    └── README.md
```

## Estado actual

`android_bootstrap.cpp` todavía contiene el flujo funcional para poder probar el cliente. No se debe dividir con simples cortes de texto: cada extracción debe compilar y conservar el estado de escena.

| Código actual | Destino previsto |
|---|---|
| `BootstrapState` | `core/GameState` |
| `DrawLoadingIntro` | `scenes/LoadingScene` |
| `BootstrapInterfacePreview` | `resources/ResourceManager` + `scenes` |
| `DrawInterfacePreview` | `scenes/LoginScene` y `scenes/ServerSelectScene` |
| `DrawTexturedQuad*` | `renderer/TextureRenderer` |
| `PollPreviewConnectServer` | `network/NetworkClient` |
| `PollPreviewGameServer` | `network/NetworkClient` |
| `HandleInputEvent` | `platform/AndroidInput` |
| `android_main` | `core/GameApplication` |

## Orden de migración

1. **LoadingScene**: separar el loading original (`New_lo_back`, `MU_TITLE`, `lo_121518`, `lo_lo`) sin cambiar su comportamiento.
2. **TextureRenderer**: mover únicamente las funciones `DrawTexturedQuad*`.
3. **ResourceManager**: centralizar la carga y destrucción de `OZJ`, `OZT` y `BMD`.
4. **LoginScene/ServerSelectScene**: trasladar el preview actual y sus áreas táctiles.
5. **NetworkClient**: aislar la conexión y el lector/escritor de paquetes.
6. **CharacterSelectScene/GameScene**: integrar personajes, mapas, animaciones y HUD.
7. Reducir `android_bootstrap.cpp` hasta dejar solamente la entrada Android y el ciclo de aplicación.

## Reglas de integración

- Cada módulo debe compilar en **ARM64-v8a** y Android API 29+.
- No incluir la carpeta `Data` en el repositorio ni en el APK.
- Mantener exactamente las rutas y mayúsculas/minúsculas de los recursos.
- Las escenas no deben acceder directamente a APIs Android; deben usar `platform`.
- El renderer no debe abrir sockets ni leer paquetes.
- La red no debe dibujar ni cargar texturas.
- No copiar módulos `CBGAMEGUARD`, `APICB`, `Protect`, `NProtect`, `XShield` ni bibliotecas precompiladas no auditadas.
- Cada movimiento de código debe ir seguido por `git diff --check` y una compilación Debug en Android Studio.
