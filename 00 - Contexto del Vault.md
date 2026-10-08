---
tipo: contexto
creado: 2026-08-14
tags: [meta, contexto, redes, universidad]
---

# 00 — Contexto del Vault

> **Esta nota es genérica y vale para todo el vault.**
> Nada específico de una práctica concreta va aquí: cada actividad se documenta **dentro de su propia carpeta**.

## Propósito

Este vault de Obsidian existe para **entender y desarrollar las guías de laboratorio de la universidad** (asignatura de Infraestructura / Redes de Computadores).

Cada carpeta corresponde a **una guía o práctica entregada por el docente**, normalmente en formato `.docx`, que hay que:

1. **Leer y entender** — extraer objetivo, topología, tabla de direccionamiento y componentes.
2. **Desarrollar** — reconstruir la práctica en **Cisco Packet Tracer**, corrigiendo los errores tipográficos que suelen traer las guías originales.
3. **Documentar** — dejar en la **misma carpeta** una nota `.md` con el paso a paso reproducible: qué objetos arrastrar en Packet Tracer, dónde encontrarlos, cómo cablear y qué comandos ejecutar en cada dispositivo.
4. **Responder** — las preguntas de control y el glosario que la guía incluye al final.

## Regla de organización (importante)

**Todo lo específico de una actividad vive dentro de la carpeta de esa actividad.** La raíz del vault solo contiene notas transversales (esta) y el índice de prácticas.

```
infra 2/
├── 00 - Contexto del Vault.md        ← esta nota (genérica, transversal)
├── <tema-1>/                         ← una carpeta por práctica
│   ├── <Guía original>.docx          ← fuente del docente, intacta
│   ├── Guía paso a paso - <tema>.md  ← desarrollo
│   └── <tema>.pkt                    ← archivo de Packet Tracer
├── <tema-2>/
│   └── ...
└── ...
```

Motivo: a futuro habrá varias prácticas en carpetas distintas. Si los comandos, topologías y direccionamientos de una actividad se mezclan en la raíz, dejan de ser localizables y se pisan entre sí.

## Convención de trabajo

| Elemento | Convención |
|---|---|
| Una carpeta = una práctica | `trunk/`, `nat/`, `ospf/`, … (nombre corto, minúscula) |
| Fuente original | El `.docx` del docente se deja **intacto** dentro de la carpeta |
| Desarrollo | `Guía paso a paso - <tema>.md`, **siempre junto al `.docx`**, nunca en la raíz |
| Simulación | El `.pkt` se guarda en la misma carpeta |
| Nomenclatura de dispositivos | Se respeta la de la guía original (`Sw1`, `Router0`, `PC0`, …) |
| Idioma | Explicaciones en español; **nombres de objetos de Packet Tracer en inglés y español**, porque la interfaz puede estar en cualquiera de los dos |
| Correcciones | Cuando la guía original tiene un error, se marca con ⚠️ y se explica la corrección en vez de copiarlo tal cual |
| Estructura de cada nota | Objetivo → topología → componentes y dónde hallarlos en PT → cableado → comandos por dispositivo → pruebas → errores frecuentes → preguntas → glosario → checklist |

## Índice de prácticas

| Carpeta | Práctica | Nota de desarrollo |
|---|---|---|
| `trunk/` | **P1:** VLANs y enrutamiento inter-VLAN (802.1Q) · **P2:** servidores web + consola | [[Guía paso a paso - VLAN Trunking]] · [explicador interactivo](trunk/Conceptos%20-%20VLAN%20Trunking.html) |
| `security port/` | Port Security (control de acceso por MAC) + RIPv2 en topología multisede Bogotá–Cali–Cúcuta | [[Guía paso a paso - Port Security y RIPv2]] |
| `corte 2/Challenge PPP w CHAP/` | **Challenge:** PPP con CHAP + Frame Relay/RIP + NAT + DHCP/DNS/HTTP | [[Guía paso a paso - PPP con CHAP]] · [[RAPIDO]] |

<!-- Añadir una fila por cada práctica nueva. Solo el enlace: los detalles quedan en la carpeta. -->

## Herramientas

- **Cisco Packet Tracer** (7.x / 8.x) — simulador donde se arma cada topología.
- **Obsidian** — este vault, para la documentación.
- Los comandos usados son **Cisco IOS**; sirven igual en equipo real de esa familia.

## Notas transversales de Packet Tracer

Válidas para cualquier práctica, por eso viven aquí y no se repiten en cada carpeta:

- El catálogo de dispositivos está **abajo a la izquierda**: primero la **categoría** (*Device-Type Selection Box*, fila superior) y luego el **modelo** (*Device-Specific Selection Box*, panel derecho).
- Los dispositivos tardan ~30 s en arrancar; los enlaces pasan de **ámbar** a **verde** cuando el puerto converge (STP).
- Un enlace en **rojo** casi siempre significa interfaz en `shutdown` o tipo de cable equivocado.
- Los routers Cisco traen todas las interfaces apagadas de fábrica: `no shutdown` es obligatorio.
- Guardar siempre con `copy running-config startup-config` antes de cerrar, y el archivo como `.pkt` con el nombre de la práctica.
- Comandos de verificación que sirven en casi todo laboratorio:
  - `show running-config` — configuración activa
  - `show ip interface brief` — resumen de interfaces e IPs
  - `show vlan brief` — VLANs y puertos asignados (switch)
  - `show interfaces trunk` — enlaces troncales (switch)
  - `show ip route` — tabla de enrutamiento (router)

## Modos de la CLI de Cisco (referencia rápida)

| Prompt | Modo | Cómo entrar | Cómo salir |
|---|---|---|---|
| `equipo>` | Usuario (*user EXEC*) | inicio de sesión | `exit` |
| `equipo#` | Privilegiado (*privileged EXEC*) | `enable` | `disable` |
| `equipo(config)#` | Configuración global | `configure terminal` | `exit` / `end` |
| `equipo(config-if)#` | Interfaz | `interface <tipo> <num>` | `exit` |
| `equipo(config-subif)#` | Subinterfaz | `interface <tipo> <num>.<sub>` | `exit` |
| `equipo(config-vlan)#` | VLAN | `vlan <id>` | `exit` |
| `equipo(config-line)#` | Línea (consola / vty) | `line vty 0 4` | `exit` |

> ⚠️ Error recurrente en las guías del docente: escriben `configure terminal` desde el prompt `(Config)#`. Ese comando se ejecuta desde el modo **privilegiado** (`equipo#`).
