# MainScreenDS Monitor Suite

**🇪🇸 [Español](#español) · 🇺🇸 [English](#english)**

> Caso de estudio / Case study · .NET 8 · Windows Service · Código privado / Private source code

---

## Español

### Resumen

Watchdog para Windows desarrollado en **.NET 8** que vigila el software de las pizarras de resultados (MainScreenDS) en una red de **150 sucursales** de una empresa de banca de apuestas, y lo recupera automáticamente antes de que falle. Resolvió las caídas constantes de las pizarras sin que nadie tenga que intervenir en cada sucursal.

### El problema

- El programa de las pizarras consumía cada vez más memoria hasta volverse inestable y caerse.
- Cada caída requería atención manual y se notaba frente a los clientes en sala.
- Con 150 sucursales, resolverlo caso por caso no era sostenible.

### La solución

Un servicio de Windows que revisa el programa de forma periódica y actúa por niveles:

1. **Observa:** mide la memoria privada del proceso en cada ciclo y la registra en un log.
2. **Clasifica:** combina umbrales absolutos (detección, advertencia, crítico) con un análisis de **crecimiento sostenido** en una ventana de tiempo.
3. **Actúa solo cuando es crítico:** cierra el proceso y lo vuelve a lanzar en la sesión de consola activa, verificando después que arrancó bien.
4. **Se protege a sí mismo:** una protección *anti-loop* con ventana de cooldown evita reinicios en cadena cuando la recuperación falla; los reinicios exitosos no cuentan contra el límite.
5. **Sobrevive a errores:** un fallo en un ciclo se registra y se reintenta en el siguiente, y Windows reinicia el servicio si este se cae.

| Módulo | Función |
|---|---|
| **MonitorService** | Servicio en segundo plano (`BackgroundService` / `IHost`) con la lógica de monitoreo y recuperación. |
| **Setup** | Instalador: detiene la versión anterior, copia los archivos, registra el servicio con inicio automático y configura su recuperación ante caídas. |
| **Uninstall** | Desinstalador limpio que detiene y elimina el servicio, conservando los logs. |
| **MemoryMonitorTests** | 16 pruebas unitarias (xUnit) de la clasificación de memoria y la protección anti-loop. |

### Mi aporte

- Refactoricé `MonitorService` a un **Worker Service** con `BackgroundService` e `IHost`.
- Diseñé la clasificación de estados de memoria y la protección anti-loop, y escribí pruebas unitarias que reproducen el bloqueo que corregí.
- Construí el instalador y desinstalador, y la publicación *self-contained* x64 para instalar en cada PC sin requisitos previos.
- Desplegué la versión final en las sucursales y validé que todo funcionara correctamente.
- Resultado: **se resolvieron las caídas constantes de las pizarras** en las 150 sucursales.

### Tecnologías

`C#` · `.NET 8` · `Worker Service` · `BackgroundService` · `IHost` · `Windows Service` · `xUnit` · `PowerShell`

### Nota sobre el código

El código fuente es **privado** (software interno de la empresa). Este repositorio documenta el problema, la arquitectura y el resultado como caso de estudio.

### Contacto

Leandro Perdomo Soto · Estudiante de Ingeniería de Software · Portafolio Leo Dev

---

## English

### Overview

A Windows watchdog built with **.NET 8** that supervises the results-board software (MainScreenDS) across a network of **150 branches** of a sports-betting company, and recovers it automatically before it fails. It resolved the constant board crashes without anyone having to intervene at each branch.

### The problem

- The board application kept consuming more memory until it became unstable and crashed.
- Every crash needed manual attention and was visible to customers on the floor.
- With 150 branches, fixing it case by case was not sustainable.

### The solution

A Windows service that checks the application periodically and responds in tiers:

1. **Observe:** measures the process's private memory every cycle and logs it.
2. **Classify:** combines absolute thresholds (detected, warning, critical) with a **sustained-growth** analysis over a time window.
3. **Act only when critical:** terminates the process and relaunches it in the active console session, then verifies it came back correctly.
4. **Protect itself:** an *anti-loop* guard with a cooldown window prevents restart storms when recovery fails; successful restarts don't count against the limit.
5. **Survive errors:** a failure in one cycle is logged and retried on the next, and Windows restarts the service itself if it ever goes down.

| Module | Purpose |
|---|---|
| **MonitorService** | Background service (`BackgroundService` / `IHost`) with the monitoring and recovery logic. |
| **Setup** | Installer: stops the previous version, copies files, registers the service with automatic start, and configures its crash recovery. |
| **Uninstall** | Clean uninstaller that stops and removes the service, keeping the logs. |
| **MemoryMonitorTests** | 16 unit tests (xUnit) covering memory classification and the anti-loop guard. |

### My contribution

- Refactored `MonitorService` into a **Worker Service** using `BackgroundService` and `IHost`.
- Designed the memory-state classification and the anti-loop guard, and wrote unit tests that reproduce the lockout bug I fixed.
- Built the installer and uninstaller, plus a self-contained x64 publish so it installs on any PC with no prerequisites.
- Rolled out the final version to the branches and validated that everything worked correctly.
- Outcome: **the constant board crashes were resolved** across all 150 branches.

### Tech stack

`C#` · `.NET 8` · `Worker Service` · `BackgroundService` · `IHost` · `Windows Service` · `xUnit` · `PowerShell`

### Note on the source code

The source code is **private** (internal company software). This repository documents the problem, architecture and outcome as a case study.

### Contact

Leandro Perdomo Soto · Software Engineering student · Leo Dev portfolio
