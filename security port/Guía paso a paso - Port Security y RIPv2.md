---
tipo: laboratorio
practica: Port Security + RIPv2 (topología multisede)
fuente: "Imagen1.png (enunciado del docente)"
software: Cisco Packet Tracer 7.x / 8.x
creado: 2026-08-17
tags: [redes, port-security, rip, ripv2, packet-tracer, cisco, capa2, mac, seguridad]
---

# Laboratorio: Port Security y enrutamiento RIPv2

> **Objetivo:** asegurar la infraestructura de **capa 2** con **Port Security** (control de acceso por dirección
> MAC) y garantizar la interconexión de **capa 3** entre las tres sedes —Bogotá, Cali y Cúcuta— mediante
> **RIP versión 2**.

Ver contexto general en [[00 - Contexto del Vault]].

## Contenido

| | Alcance | Secciones |
|---|---|---|
| **A. Montaje** | Armar la topología, direccionar y levantar RIPv2 | §0 – §5 |
| **B. Seguridad** | Los 7 requerimientos de Port Security, uno por uno | §6 – §9 |
| **C. Cierre** | Pruebas, errores, conceptos, configuración completa y checklist | §10 – §14 |

## Notas importantes antes de empezar

> ⚠️ **Sobre los otros archivos de esta carpeta.** `Challenge02.html` y
> `Solución Packet Tracer Port Security RIPv2.pdf` son una **solución generada con IA** que contiene errores y
> huecos sin rellenar (`<MAC_DEL_CCTV>`, `<RED_LAN_BOGOTA>`…). Concretamente:
>
> | Lo que dice ese material | Lo que muestra la topología real |
> |---|---|
> | El ServerDNS está en `Fa0/2` de Switch21 | Está en **`Gig0/2`** de Switch21. `Fa0/2` es **PC16** |
> | `interface range Fa0/1 - 23` con `maximum 1` | Rompe los requisitos **3** y **6** (ver §6) |
> | No contempla el requerimiento 6 | El **Portátil IT** obliga a subir el `maximum` en **todos** los puertos |
>
> Esta guía está hecha **desde el enunciado y la topología originales** (`Imagen1.png`), no desde ese material.

> ℹ️ **Supuestos marcados.** El enunciado no da direccionamiento IP y RIPv2 lo necesita, así que en §2 se
> propone un esquema limpio. Si el profesor entrega otro, solo hay que cambiar §2 y los `network` de §5.

## Registro de avance

| Fecha | Estado |
|---|---|
| 17/08/2026 | ✅ **Topología física terminada** — 3×1941, 6×2960-24TT, 12 equipos finales, cables correctos (cruzado entre switches, serial en la WAN). Revisada contra el enunciado. |
| 17/08/2026 | 🔧 Ajuste: en Router6 los seriales quedaron invertidos respecto a la primera versión de esta guía. **`Se0/0/1` → Bogotá, `Se0/0/0` → Cúcuta.** IPs de §2.3 y §13 corregidas. |
| 17/08/2026 | 🔧 Decisión: el enlace Switch18 ↔ Switch22(2) se mueve de `Fa0/24` a **`Fa0/23`** en **ambos** extremos, para dejar `Fa0/24` libre en los 6 switches y poder cumplir el requisito 5 al 100 %. Propagado a §0, §1.3, §4.5, §6.0, §6.4, §7 y §13. |
| 17/08/2026 | 🔴 **Pendiente:** configurar los 3 routers (§2.3). Todas las interfaces siguen en `shutdown`, por eso los enlaces a routers se ven en rojo. Después: §5 RIPv2 y §6 Port Security. |

---

# A. MONTAJE

## 0. Resumen de lo que vas a construir

### Topología general

```
        ┌───────────── BOGOTÁ ─────────────┐   ┌── CALI ──┐   ┌──────── CÚCUTA ────────┐

              Secretaria   Gerencia                                   PC15
                  │Fa0/1     │Fa0/2                                    │Fa0/2
              ┌───┴──────────┴───┐                                ┌────┴─────┐
              │    Switch22      │                                │ Switch20 │
              └────────┬─────────┘                                └────┬─────┘
                       │Gig0/2                                         │Gig0/2
                       │                                               │ (cruzado)
              ┌────────┴─────────┐                                ┌────┴─────┐
   Router5 ───┤    Switch18      │                                │ Switch21 │──── PC16
     Gig0/1   └────────┬─────────┘                                └──┬────┬──┘  Fa0/2
                       │Fa0/23  (cruzado)                     Gig0/2 │    └─ Gig0/1 ↑
                       │                                             │
              ┌────────┴─────────┐                                ServerDNS
              │   Switch22(2)    │
              └──┬─────┬──────┬──┘
           Fa0/1 │Fa0/2│ Fa0/3│
        Gerencia │ CCTV│      └─ Secretaria administrativa
   Administrativa│

   Router5 ══Se0/0/0══╗                          ╔══Se0/0/1══ Router5(1)
                  10.0.0.1 ─ .2 (Se0/0/1)   (Se0/0/0) 10.0.0.5 ─ .6
                      ║                          ║              Gig0/0 → Switch20 Gig0/1
                      ╚═ Router6 (CALI) ═════════╝
                            Gig0/1
                              │
                       ┌──────┴──────┐
                       │  SwitchCali │
                       └──┬───────┬──┘
                    Fa0/1 │       │ Fa0/2
                  Contaduría   Tesorería

   SUELTOS (no cableados): Portátil IT · Máquina empleado externo 1 · Máquina empleado externo 2
```

### Inventario de equipos

| Sede | Equipo | Modelo sugerido | Nombre en la topología |
|---|---|---|---|
| Bogotá | Router | **1941** (+ HWIC-2T) | `Router5` |
| Bogotá | Switch núcleo | **2960-24TT** | `Switch18` |
| Bogotá | Switch | 2960-24TT | `Switch22` |
| Bogotá | Switch | 2960-24TT | `Switch22(2)` |
| Bogotá | PCs | PC-PT ×4 | Secretaria · Gerencia · Gerencia Administrativa · Secretaria administrativa |
| Bogotá | Cámara | PC-PT | `CCTV` |
| Cali | Router | 1941 (+ HWIC-2T) | `Router6` |
| Cali | Switch | 2960-24TT | `Switch22(2)(2)` → aquí lo llamo **SwitchCali** |
| Cali | PCs | PC-PT ×2 | Contaduría · Tesorería |
| Cúcuta | Router | 1941 (+ HWIC-2T) | `Router5(1)` |
| Cúcuta | Switches | 2960-24TT ×2 | `Switch20` · `Switch21` |
| Cúcuta | PCs | PC-PT ×2 | PC15 · PC16 |
| Cúcuta | Servidor | Server-PT | `ServerDNS` |
| — | Portátil | Laptop-PT | `Portatil IT` (sin cablear) |
| — | Portátiles | Laptop-PT ×2 | `Maquina de empleado externo 1` y `2` (sin cablear) |

> 💡 **Por qué el 2960-24TT.** Es el único switch de Packet Tracer cuyos puertos se llaman exactamente como en
> la topología del profesor: **`Fa0/1` … `Fa0/24`** más **`Gig0/1`** y **`Gig0/2`**. El 2950T-24 de la práctica
> anterior no sirve aquí porque sus puertos gigabit se llaman distinto.
>
> 💡 **Por qué el 1941.** Tiene `Gig0/0` y `Gig0/1` de fábrica, y con un módulo **HWIC-2T** aparecen
> `Serial0/0/0` y `Serial0/0/1`, que es justo lo que pide la topología. El **2911** también sirve.

### Requerimientos del enunciado

| # | Requerimiento (texto del profesor) | Mecanismo |
|---|---|---|
| 1 | Un equipo externo conectado a **cualquier switch** no debe dar conexión de red, así tenga la IP bien configurada | Port Security en todos los puertos de acceso + apagar puertos libres |
| 2 | Conectar otro equipo en el **puerto del servidor de Cúcuta** bloquea el puerto inmediatamente | `maximum 1` + MAC estática del ServerDNS + `violation shutdown` |
| 3 | Los equipos de **gerencia** se pueden intercambiar entre las sedes de Bogotá **sin afectar el servicio** | Ambas MAC de gerencia autorizadas en ambos puertos + `violation restrict` |
| 4 | Los equipos de **secretaría** **no** se pueden intercambiar; el servicio caería inmediatamente | 1 MAC propia por puerto + `violation shutdown` |
| 5 | El **CCTV** se podrá conectar en cualquier **Fa0/24** de cualquier switch, si tiene la IP de esa red | MAC del CCTV autorizada en `Fa0/24` de todos los switches |
| 6 | El **Portátil IT** se podrá conectar en **cualquier puerto** | MAC del portátil autorizada en todos los puertos de acceso (`maximum` +1) |
| 7 | En **Cali** solo se permiten los equipos de la topología, respetando las reglas anteriores | Port Security en Fa0/1 y Fa0/2 + resto de puertos apagados |

---

## 1. Armar la topología en Packet Tracer

### 1.1 Dónde encontrar cada objeto

| Qué necesitas | Categoría (inglés) | Categoría (español) | Modelo a arrastrar |
|---|---|---|---|
| Routers | **Network Devices → Routers** | Dispositivos de red → Enrutadores | **1941** |
| Switches | **Network Devices → Switches** | Dispositivos de red → Conmutadores | **2960-24TT** |
| PCs y CCTV | **End Devices → End Devices** | Dispositivos finales | **PC** (`PC-PT`) |
| Portátiles | **End Devices → End Devices** | Dispositivos finales | **Laptop** (`Laptop-PT`) |
| Servidor | **End Devices → End Devices** | Dispositivos finales | **Server** (`Server-PT`) |
| Cable directo | **Connections → Connections** | Conexiones | **Copper Straight-Through** (negro continuo) |
| Cable cruzado | **Connections → Connections** | Conexiones | **Copper Cross-Over** (negro **punteado**) |
| Cable serial | **Connections → Connections** | Conexiones | **Serial DCE** (el del **reloj** ⏱) |

> 💡 **Directo vs cruzado, la regla de siempre.** Directo para **equipos distintos** (PC↔switch, router↔switch);
> cruzado para **equipos iguales** (switch↔switch). En la imagen del profesor los enlaces switch↔switch se ven
> **punteados**: eso es exactamente el cable cruzado.
>
> ⚡ Si te lías, el rayo dorado *Automatically Choose Connection Type* elige por ti — pero para la entrega usa
> el cable explícito, que suele revisarse.

### 1.2 Instalar el módulo serial en los 3 routers

El 1941 **no trae puertos seriales de fábrica**. En cada router:

1. Clic en el router → pestaña **`Physical`** («Físico»).
2. **Apaga el equipo** con el interruptor del chasis. *(Packet Tracer no deja instalar módulos encendido.)*
3. Arrastra **`HWIC-2T`** desde la lista `MODULES` hasta una ranura libre.
4. **Enciéndelo** de nuevo.
5. Verifica con `show ip interface brief`: deben aparecer `Serial0/0/0` y `Serial0/0/1`.

> Repite en **Router5**, **Router6** y **Router5(1)**. Router6 usa los dos seriales; los otros dos, uno cada uno.

### 1.3 Cableado — Bogotá

| # | Desde | Puerto | Hacia | Puerto | Cable |
|---|---|---|---|---|---|
| 1 | Router5 | `Gig0/1` | Switch18 | `Gig0/1` | Directo |
| 2 | Switch18 | `Gig0/2` | Switch22 | `Gig0/2` | **Cruzado** |
| 3 | Switch18 | **`Fa0/23`** | Switch22(2) | **`Fa0/23`** | **Cruzado** |
| 4 | Secretaria | `Fa0` | Switch22 | `Fa0/1` | Directo |
| 5 | Gerencia | `Fa0` | Switch22 | `Fa0/2` | Directo |
| 6 | Gerencia Administrativa | `Fa0` | Switch22(2) | `Fa0/1` | Directo |
| 7 | CCTV | `Fa0` | Switch22(2) | `Fa0/2` | Directo |
| 8 | Secretaria administrativa | `Fa0` | Switch22(2) | `Fa0/3` | Directo |

> 🔧 **Enlace 3: por qué va en `Fa0/23` y no en `Fa0/24`** (decisión tomada el 17/08).
>
> La topología del profesor pone ese cable en `Fa0/24` ↔ `Fa0/24`. **El problema:** el **requerimiento 5** dice
> que el CCTV debe poder conectarse en el `Fa0/24` de **cualquier** switch. Si el `Fa0/24` de Switch18 y de
> Switch22(2) está ocupado por este cable, el requisito solo se podría cumplir en 4 de los 6 switches.
>
> **La solución es mover los dos extremos del cable un puerto hacia abajo:**
>
> | | Antes (imagen del profesor) | Ahora (esta guía) |
> |---|---|---|
> | Switch18 | `Fa0/24` | **`Fa0/23`** |
> | Switch22(2) | `Fa0/24` | **`Fa0/23`** |
>
> Es **un solo cable** y **ningún otro enlace cambia**. Con esto el `Fa0/24` queda libre en los **6** switches
> y el requisito 5 se cumple al 100 %. Ese puerto no se usaba para nada más, así que no se pierde nada.
>
> 📌 **Menciónalo en el informe**: detectar esta contradicción y resolverla es exactamente lo que se busca que
> haga el estudiante, y suma más que copiar la topología a ciegas.

### 1.4 Cableado — Cali

| # | Desde | Puerto | Hacia | Puerto | Cable |
|---|---|---|---|---|---|
| 9 | Router6 | `Gig0/1` | SwitchCali | `Gig0/1` | Directo |
| 10 | Contaduría | `Fa0` | SwitchCali | `Fa0/1` | Directo |
| 11 | Tesorería | `Fa0` | SwitchCali | `Fa0/2` | Directo |

### 1.5 Cableado — Cúcuta

| # | Desde | Puerto | Hacia | Puerto | Cable |
|---|---|---|---|---|---|
| 12 | Router5(1) | `Gig0/0` | Switch20 | `Gig0/1` | Directo |
| 13 | Switch20 | `Gig0/2` | Switch21 | `Gig0/1` | **Cruzado** |
| 14 | PC15 | `Fa0` | Switch20 | `Fa0/2` | Directo |
| 15 | PC16 | `Fa0` | Switch21 | `Fa0/2` | Directo |
| 16 | ServerDNS | `Fa0` | Switch21 | **`Gig0/2`** ⚠️ | Directo |

> ⚠️ **El ServerDNS va en `Gig0/2`, no en `Fa0/2`.** `Fa0/2` es PC16. Es raro colgar un servidor de un puerto
> gigabit, pero es lo que muestra la topología del profesor — y es el error que trae el PDF de la carpeta.

### 1.6 Cableado — enlaces seriales WAN

Selecciona **Serial DCE**. **El primer equipo donde haces clic es el DCE** (el que marca el reloj):

| # | Desde (DCE ⏱) | Puerto | Hacia (DTE) | Puerto |
|---|---|---|---|---|
| 17 | **Router5** (Bogotá) | `Se0/0/0` | Router6 (Cali) | **`Se0/0/1`** |
| 18 | **Router6** (Cali) | **`Se0/0/0`** | Router5(1) (Cúcuta) | `Se0/0/1` |

> ✅ **Actualizado el 17/08 según la topología ya armada.** En Router6 los dos seriales quedaron al revés de lo
> que decía la versión anterior de esta guía: **`Se0/0/1` mira a Bogotá** y **`Se0/0/0` mira a Cúcuta**.
> Funcionalmente da exactamente igual — pero las IPs de §2.3 tienen que seguir el cable, y ya están ajustadas.

> 💡 **DCE / DTE.** En un enlace serial alguien tiene que marcar el ritmo del reloj. El extremo **DCE** lleva
> `clock rate`; el **DTE** no configura nada. En la imagen del profesor el icono de reloj ⏱ aparece junto a
> `Se0/0/0` de Router5 y a `Se0/0/1` de Router6 — por eso esos son los DCE. Si dudas:
> `show controllers serial 0/0/0` te dice `DCE` o `DTE`.

### 1.7 Equipos sueltos (no se cablean todavía)

Coloca en el lienzo, **sin conectar**:

- **Portatil IT** (Laptop-PT) — se usará en §10 para probar el requerimiento 6.
- **Maquina de empleado externo 1** y **2** (Laptop-PT) — se usarán para probar los requerimientos 1 y 7.

---

## 2. Direccionamiento IP

> ℹ️ Esquema propuesto (el enunciado no lo da). Coherente y fácil de cambiar.

### 2.1 Redes

| Red | Segmento | Máscara | Gateway | Interfaz del router |
|---|---|---|---|---|
| LAN Bogotá | `192.168.10.0` | `255.255.255.0` (/24) | `192.168.10.1` | Router5 `Gig0/1` |
| LAN Cali | `192.168.20.0` | `255.255.255.0` (/24) | `192.168.20.1` | Router6 `Gig0/1` |
| LAN Cúcuta | `192.168.30.0` | `255.255.255.0` (/24) | `192.168.30.1` | Router5(1) `Gig0/0` |
| WAN Bogotá–Cali | `10.0.0.0` | `255.255.255.252` (/30) | — | Router5 `Se0/0/0` ↔ Router6 `Se0/0/1` |
| WAN Cali–Cúcuta | `10.0.0.4` | `255.255.255.252` (/30) | — | Router6 `Se0/0/0` ↔ Router5(1) `Se0/0/1` |

| Interfaz | IP | Enlace |
|---|---|---|
| Router5 `Se0/0/0` | `10.0.0.1` /30 | → Cali |
| Router6 `Se0/0/1` | `10.0.0.2` /30 | → Bogotá |
| Router6 `Se0/0/0` | `10.0.0.5` /30 | → Cúcuta |
| Router5(1) `Se0/0/1` | `10.0.0.6` /30 | → Cali |

> 💡 **¿Por qué /30 en los seriales?** Un enlace punto a punto solo necesita **2 direcciones útiles**. Un /30
> da exactamente 2. Usar /24 ahí desperdiciaría 252 direcciones.

### 2.2 Equipos finales

| Equipo                    | Sede   | IP                                | Máscara       | Gateway         |
| ------------------------- | ------ | --------------------------------- | ------------- | --------------- |
| Secretaria                | Bogotá | `192.168.10.11`                   | 255.255.255.0 | 192.168.10.1    |
| Gerencia                  | Bogotá | `192.168.10.12`                   | 255.255.255.0 | 192.168.10.1    |
| Gerencia Administrativa   | Bogotá | `192.168.10.13`                   | 255.255.255.0 | 192.168.10.1    |
| Secretaria administrativa | Bogotá | `192.168.10.14`                   | 255.255.255.0 | 192.168.10.1    |
| CCTV                      | Bogotá | `192.168.10.20`                   | 255.255.255.0 | 192.168.10.1    |
| Contaduría                | Cali   | `192.168.20.11`                   | 255.255.255.0 | 192.168.20.1    |
| Tesorería                 | Cali   | `192.168.20.12`                   | 255.255.255.0 | 192.168.20.1    |
| ServerDNS                 | Cúcuta | `192.168.30.10`                   | 255.255.255.0 | 192.168.30.1    |
| PC15                      | Cúcuta | `192.168.30.11`                   | 255.255.255.0 | 192.168.30.1    |
| PC16                      | Cúcuta | `192.168.30.12`                   | 255.255.255.0 | 192.168.30.1    |
| **Portátil IT**           | móvil  | `.100` de la LAN donde se conecte | 255.255.255.0 | `.1` de esa LAN |
| **CCTV** (al moverlo)     | móvil  | `.20` de la LAN donde se conecte  | 255.255.255.0 | `.1` de esa LAN |
| Empleado externo 1 y 2    | —      | cualquiera                        | —             | —               |

> 💡 El requerimiento 5 dice *"siempre y cuando tenga configurado el direccionamiento ip de la red"*. Por eso
> el CCTV y el Portátil IT **cambian de IP** según la sede donde los enchufes. Si los mueves y no cambias la
> IP, el puerto los deja pasar (Port Security es capa 2) pero **no habrá comunicación** (capa 3).

**Cómo se configuran:** clic en el equipo → pestaña **`Desktop`** → **`IP Configuration`** → **`Static`**.
En el servidor es idéntico.

### 2.3 Configuración de las interfaces de los routers

# Router 5
enable
configure terminal
hostname Router5
!
interface gigabitEthernet 0/1
 ip address 192.168.10.1 255.255.255.0
 no shutdown
 exit
!
interface serial 0/0/0
 ip address 10.0.0.1 255.255.255.252
 clock rate 64000
 no shutdown
 exit
!
end
```
</details>

<details>
<summary><b>Router6 — Cali</b></summary>

```cisco
enable
configure terminal
hostname Router6
!
interface gigabitEthernet 0/1
 ip address 192.168.20.1 255.255.255.0
 no shutdown
 exit
!
interface serial 0/0/1          ! ← mira a BOGOTA
 ip address 10.0.0.2 255.255.255.252
 no shutdown
 exit
!
interface serial 0/0/0          ! ← mira a CUCUTA
 ip address 10.0.0.5 255.255.255.252
 clock rate 64000
 no shutdown
 exit
!
end
```


<details>
<summary><b>Router5(1) — Cúcuta</b></summary>

```cisco
enable
configure terminal
hostname Router5-1
!
interface gigabitEthernet 0/0
 ip address 192.168.30.1 255.255.255.0
 no shutdown
 exit
!
interface serial 0/0/1
 ip address 10.0.0.6 255.255.255.252
 no shutdown
 exit
!
end
```
</details>

> ⚠️ **`clock rate` solo en el DCE.** Si lo escribes en el DTE, el IOS lo acepta pero lo ignora. Si **falta**
> en el DCE, el enlace queda `up/down` (línea arriba, protocolo abajo) y nunca funciona. Es el error nº 1 de
> los enlaces seriales.
>
> ⚠️ Los paréntesis no son válidos en un `hostname` de Cisco: `Router5(1)` → usa **`Router5-1`**.

---

## 3. Recolectar las direcciones MAC

**Este es el paso que hace posible todo el resto.** Port Security trabaja con MAC, así que necesitas
tenerlas anotadas antes de configurar nada.

### 3.1 Cómo leer la MAC de un equipo

**Opción A — la rápida:** clic en el equipo → pestaña **`Config`** → en el panel izquierdo elige
**`FastEthernet0`** → el campo **`MAC Address`** muestra algo como `0001.C4A1.B2C3`.

**Opción B — desde el sistema:** clic en el equipo → **`Desktop`** → **`Command Prompt`** →

```
ipconfig /all
```

y busca la línea **`Physical Address`**.

> 💡 **Formato Cisco.** El IOS espera la MAC en **tres grupos de 4 dígitos separados por puntos**:
> `0001.C4A1.B2C3`. Si Packet Tracer te la muestra como `0001.C4A1.B2C3` ya está lista; si la ves con guiones
> (`00-01-C4-A1-B2-C3`), reagrúpala.

### 3.2 Tabla para rellenar

> 📋 Copia esta tabla y **complétala con las MAC de tu propio archivo**. Son distintas en cada Packet Tracer:
> se generan al azar al crear el equipo. **No copies MAC de ejemplo de internet ni del PDF de la carpeta.**

| Equipo | Sede | Dirección MAC | ¿Dónde se autoriza? |
|---|---|---|---|
| Secretaria | Bogotá | `____.____.____` | Switch22 Fa0/1 |
| Gerencia | Bogotá | `____.____.____` | Switch22 Fa0/2 **y** Switch22(2) Fa0/1 |
| Gerencia Administrativa | Bogotá | `____.____.____` | Switch22(2) Fa0/1 **y** Switch22 Fa0/2 |
| Secretaria administrativa | Bogotá | `____.____.____` | Switch22(2) Fa0/3 |
| **CCTV** | móvil | `____.____.____` | **`Fa0/24` de los 6 switches** (todos libres) + Switch22(2) Fa0/2 |
| Contaduría | Cali | `____.____.____` | SwitchCali Fa0/1 |
| Tesorería | Cali | `____.____.____` | SwitchCali Fa0/2 |
| PC15 | Cúcuta | `____.____.____` | Switch20 Fa0/2 |
| PC16 | Cúcuta | `____.____.____` | Switch21 Fa0/2 |
| ServerDNS | Cúcuta | `____.____.____` | Switch21 Gig0/2 |
| **Portátil IT** | móvil | `____.____.____` | **todos los puertos de acceso** (menos el del ServerDNS) |
| Empleado externo 1 | — | `____.____.____` | **ninguno** (debe quedar bloqueado) |
| Empleado externo 2 | — | `____.____.____` | **ninguno** (debe quedar bloqueado) |

> 💡 Anota también las MAC de los **externos**: te sirven para comprobar en `show port-security address` que
> efectivamente son las que dispararon la violación.

---

## 4. Conceptos: cómo funciona Port Security

> Léelo antes de escribir comandos. Si entiendes estas cuatro ideas, los 7 requerimientos salen solos.

### 4.1 Qué hace exactamente

Un switch normal aprende la MAC de cualquiera que se conecte y le da servicio. **Port Security** le pone
condiciones a un puerto: *"por aquí solo pueden hablar estas MAC, y como máximo tantas a la vez"*. Si aparece
otra, el puerto reacciona.

> 🔑 **Por eso el requerimiento 1 insiste en *"así el equipo tenga configurada la dirección ip correctamente"*.**
> Port Security actúa en **capa 2**, antes de que la IP importe. La trama del intruso ni siquiera llega a
> mirarse: se descarta o el puerto se apaga. Tener la IP correcta no sirve de nada.

### 4.2 Las tres piezas de la configuración

| Comando | Qué controla |
|---|---|
| `switchport port-security maximum <n>` | **Cuántas** MAC distintas se admiten en el puerto |
| `switchport port-security mac-address <MAC>` | **Cuál** MAC se admite (la escribes tú, *estática*) |
| `switchport port-security mac-address sticky` | El switch **aprende** la MAC conectada y la guarda sola |
| `switchport port-security violation <modo>` | **Qué pasa** cuando aparece una MAC no autorizada |

### 4.3 Sticky vs estática — cuándo usar cada una

| | `sticky` | estática (`mac-address 0001.…`) |
|---|---|---|
| Cómo se llena | El switch aprende lo que esté conectado **en ese momento** | La escribes tú a mano |
| Ventaja | Rapidísimo, no hay que buscar MAC | Funciona con equipos que **no** están conectados aún |
| Riesgo | Si al activarlo hay conectado el equipo equivocado, **memoriza el equivocado** | Tedioso, hay que teclear bien |
| Se usa aquí para | Requisitos **1, 2, 4, 7** (el equipo ya está en su puerto) | Requisitos **3, 5, 6** (el equipo aún no está ahí) |

> ⚠️ **Sticky no guarda solo.** Aprende la MAC y la escribe en la `running-config`. Si no haces
> `copy running-config startup-config`, al reiniciar se pierde.

### 4.4 Los tres modos de violación — la clave de los requisitos 3 y 4

| Modo | ¿Descarta la trama? | ¿Avisa? | ¿Apaga el puerto? | Se usa aquí en… |
|---|---|---|---|---|
| `protect` | ✅ | ❌ silencioso | ❌ | — |
| `restrict` | ✅ | ✅ log + contador | ❌ **sigue dando servicio** | **Requisito 3** (gerencia) |
| `shutdown` *(por defecto)* | ✅ | ✅ | ✅ pasa a **`err-disabled`** | **Requisitos 1, 2, 4, 5, 7** |

> 🔑 **Aquí está la diferencia entre el requisito 3 y el 4.** El enunciado dice que gerencia se puede
> intercambiar *"y el servicio no se debe ver afectado"* → **`restrict`**, que bloquea al intruso pero deja el
> puerto vivo. Y dice que secretaría no se puede intercambiar *"pues el servicio caería inmediatamente"* →
> **`shutdown`**, que tumba el puerto. El profesor te está pidiendo literalmente los dos modos.

### 4.5 Regla de oro: nunca en los uplinks

**Port Security va solo en puertos de acceso** (donde se conecta **un** equipo final).

Un enlace switch↔switch o switch↔router transporta las MAC de **todos** los equipos que hay detrás. Si le
pones `maximum 2`, lo tumbas en cuanto pase el tercer equipo. En esta topología **quedan sin Port Security**:

- Switch18: `Gig0/1` (al router), `Gig0/2` (a Switch22), **`Fa0/23`** (a Switch22(2))
- Switch22: `Gig0/2` · Switch22(2): **`Fa0/23`**
- SwitchCali: `Gig0/1` · Switch20: `Gig0/1`, `Gig0/2` · Switch21: `Gig0/1`

---

## 5. Enrutamiento RIPv2

Sin esto, cada sede se ve a sí misma pero no a las otras.

### 5.1 Los comandos

<details open>
<summary><b>Router5 — Bogotá</b></summary>

```cisco
enable
configure terminal
router rip
 version 2
 no auto-summary
 network 192.168.10.0
 network 10.0.0.0
 passive-interface gigabitEthernet 0/1
 exit
end
copy running-config startup-config
```
</details>

<details>
<summary><b>Router6 — Cali</b></summary>

```cisco
enable
configure terminal
router rip
 version 2
 no auto-summary
 network 192.168.20.0
 network 10.0.0.0
 passive-interface gigabitEthernet 0/1
 exit
end
copy running-config startup-config
```
</details>

<details>
<summary><b>Router5(1) — Cúcuta</b></summary>

```cisco
enable
configure terminal
router rip
 version 2
 no auto-summary
 network 192.168.30.0
 network 10.0.0.0
 passive-interface gigabitEthernet 0/0
 exit
end
copy running-config startup-config
```
</details>

### 5.2 Qué hace cada línea

| Comando | Traducción |
|---|---|
| `router rip` | Entra a configurar el protocolo RIP |
| `version 2` | Usa **RIPv2**. Sin esto queda en v1, que **no envía la máscara** y rompería los /30 |
| `no auto-summary` | No resumas las redes a su clase por defecto. **Obligatorio** con /30 y subredes |
| `network 192.168.10.0` | "Anuncia esta red y activa RIP en las interfaces que estén en ella" |
| `network 10.0.0.0` | Cubre **ambos** enlaces seriales de una sola vez (RIP toma la red *classful*) |
| `passive-interface Gig0/x` | Opcional pero recomendado: no mandes anuncios RIP hacia las PCs. Solo por los seriales |

> 🔑 **`version 2` y `no auto-summary` son los dos comandos que más se olvidan.** Sin ellos, RIPv1 resumiría
> `10.0.0.0/30` a `10.0.0.0/8` y las rutas quedarían mal. Verifica siempre con `show ip protocols`.

### 5.3 Verificación

```cisco
Router5#show ip route rip
Router5#show ip protocols
Router5#show ip route
```

En `show ip route` de **cualquier** router deben aparecer **las tres LAN y los dos enlaces WAN**: las propias
con `C` (conectadas) y las ajenas con `R` (aprendidas por RIP).

```
R    192.168.20.0/24 [120/1] via 10.0.0.2, 00:00:12, Serial0/0/0
R    192.168.30.0/24 [120/2] via 10.0.0.2, 00:00:12, Serial0/0/0
```

> 💡 `[120/1]` = distancia administrativa 120 (la de RIP) y **1 salto**. Desde Bogotá, Cúcuta está a 2 saltos.
> RIP tiene un máximo de **15 saltos**; a 16 considera la red inalcanzable.

**Prueba de fuego antes de seguir:** `ping` desde una PC de Bogotá a una de Cúcuta. Si esto no funciona, **no
sigas con Port Security** — arregla primero el enrutamiento, o después no sabrás qué está fallando.

---

# B. SEGURIDAD — Los 7 requerimientos

## 6. Configuración de Port Security

### 6.0 Tabla maestra — qué lleva cada puerto

> 📌 **Esta tabla es el corazón de la práctica.** Todo lo de abajo sale de aquí.

| Switch | Puerto | Equipo | `maximum` | MAC autorizadas | `violation` | Requisito |
|---|---|---|---|---|---|---|
| **Switch22** | Fa0/1 | Secretaria | **2** | Secretaria + Portátil IT | `shutdown` | 4, 6 |
| Switch22 | Fa0/2 | Gerencia | **3** | Gerencia + Gerencia Adm + Portátil IT | `restrict` | 3, 6 |
| Switch22 | Fa0/24 | *(CCTV móvil)* | **2** | CCTV + Portátil IT | `shutdown` | 5, 6 |
| **Switch22(2)** | Fa0/1 | Gerencia Administrativa | **3** | Gerencia + Gerencia Adm + Portátil IT | `restrict` | 3, 6 |
| Switch22(2) | Fa0/2 | CCTV | **2** | CCTV + Portátil IT | `shutdown` | 5, 6 |
| Switch22(2) | Fa0/3 | Secretaria administrativa | **2** | Secr. adm + Portátil IT | `shutdown` | 4, 6 |
| Switch22(2) | Fa0/24 | *(CCTV móvil)* | **2** | CCTV + Portátil IT | `shutdown` | 5, 6 |
| **Switch18** | Fa0/23 | enlace a Switch22(2) | — | — | **sin Port Security** (uplink) | — |
| Switch18 | Fa0/24 | *(CCTV móvil)* | **2** | CCTV + Portátil IT | `shutdown` | 5, 6 |
| Switch22(2) | Fa0/23 | enlace a Switch18 | — | — | **sin Port Security** (uplink) | — |
| **SwitchCali** | Fa0/1 | Contaduría | **2** | Contaduría + Portátil IT | `shutdown` | 7, 6 |
| SwitchCali | Fa0/2 | Tesorería | **2** | Tesorería + Portátil IT | `shutdown` | 7, 6 |
| SwitchCali | Fa0/24 | *(CCTV móvil)* | **2** | CCTV + Portátil IT | `shutdown` | 5, 6 |
| **Switch20** | Fa0/2 | PC15 | **2** | PC15 + Portátil IT | `shutdown` | 1, 6 |
| Switch20 | Fa0/24 | *(CCTV móvil)* | **2** | CCTV + Portátil IT | `shutdown` | 5, 6 |
| **Switch21** | Fa0/2 | PC16 | **2** | PC16 + Portátil IT | `shutdown` | 1, 6 |
| Switch21 | **Gig0/2** | **ServerDNS** | **1** | **solo ServerDNS** | `shutdown` | **2** |
| Switch21 | Fa0/24 | *(CCTV móvil)* | **2** | CCTV + Portátil IT | `shutdown` | 5, 6 |

> 🔑 **Por qué casi todo lleva `maximum 2` y no 1.** Por el **requerimiento 6**: el Portátil IT debe poder
> conectarse en cualquier puerto. Como su MAC va autorizada en todos, cada puerto necesita hueco para **su
> equipo habitual + el portátil**. Si dejaras `maximum 1` (como propone el PDF de la carpeta), el portátil
> tumbaría todos los puertos donde se conectara.
>
> 🔑 **La única excepción es el puerto del ServerDNS.** El requerimiento 2 dice *"si se intenta conectar
> **cualquier otro equipo** … el puerto se bloquea inmediatamente"*. "Cualquier otro" incluye al portátil IT.
> Por eso ese puerto va con `maximum 1` y sin la MAC del portátil. **Menciónalo en la entrega**: es una
> contradicción aparente entre los requisitos 2 y 6, y el 2 gana por ser más específico.

### 6.1 Requisitos 1 y 4 — protección general y secretaría

Sobre los puertos donde el equipo **ya está conectado y no debe moverse**.

**Switch22 — puerto de Secretaria:**

```cisco
enable
configure terminal
hostname Switch22
!
interface fastEthernet 0/1
 switchport mode access
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security mac-address AAAA.AAAA.AAAA   ! ← MAC del Portátil IT
 switchport port-security violation shutdown
 exit
```

**Switch22(2) — puerto de Secretaria administrativa:**

```cisco
interface fastEthernet 0/3
 switchport mode access
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security mac-address AAAA.AAAA.AAAA   ! ← MAC del Portátil IT
 switchport port-security violation shutdown
 exit
```

> 💡 **Combinación sticky + estática.** El `sticky` aprende sola la MAC de la secretaria que está conectada; la
> línea estática añade el portátil IT, que no está ahí. Las dos conviven sin problema.
>
> ⚠️ Sustituye `AAAA.AAAA.AAAA` por la MAC real de tu Portátil IT (§3.2). El comando la acepta aunque sea
> inventada — y entonces no funcionará y no sabrás por qué.

**Resultado:** si desconectas la Secretaria de Bogotá y enchufas ahí la Secretaria administrativa, su MAC no
está autorizada → el puerto pasa a `err-disabled` y **el servicio cae inmediatamente**. Justo el requisito 4.

### 6.2 Requisito 2 — puerto del ServerDNS (Cúcuta)

```cisco
enable
configure terminal
hostname Switch21
!
interface gigabitEthernet 0/2
 switchport mode access
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
 exit
```

> ⚠️ **`Gig0/2`, no `Fa0/2`.** Ver §1.5.
>
> ⚠️ **`maximum 1` y sin el portátil IT**, a propósito. Ver la nota de §6.0.
>
> ⚠️ **Conecta el ServerDNS ANTES de escribir esto.** `sticky` aprende lo que esté conectado en ese momento;
> si el puerto está vacío no aprende nada, y si hay otra cosa memoriza esa otra cosa.

### 6.3 Requisito 3 — equipos de gerencia intercambiables

Los dos puertos de gerencia deben aceptar **las dos** MAC, y **no caerse** ante un intruso.

**Switch22 — Fa0/2 (Gerencia):**

```cisco
interface fastEthernet 0/2
 switchport mode access
 switchport port-security
 switchport port-security maximum 3
 switchport port-security mac-address 1111.1111.1111   ! ← MAC de Gerencia
 switchport port-security mac-address 2222.2222.2222   ! ← MAC de Gerencia Administrativa
 switchport port-security mac-address AAAA.AAAA.AAAA   ! ← MAC del Portátil IT
 switchport port-security violation restrict
 exit
```

**Switch22(2) — Fa0/1 (Gerencia Administrativa):** exactamente lo mismo, cambiando el número de interfaz.

```cisco
interface fastEthernet 0/1
 switchport mode access
 switchport port-security
 switchport port-security maximum 3
 switchport port-security mac-address 1111.1111.1111   ! ← MAC de Gerencia
 switchport port-security mac-address 2222.2222.2222   ! ← MAC de Gerencia Administrativa
 switchport port-security mac-address AAAA.AAAA.AAAA   ! ← MAC del Portátil IT
 switchport port-security violation restrict
 exit
```

> 🔑 **Aquí NO se usa `sticky`, se usan MAC estáticas.** `sticky` solo aprendería la que está conectada ahora;
> necesitas que el puerto acepte también la del **otro** equipo de gerencia, que está en el otro switch. Por
> eso se escriben las dos a mano.
>
> 🔑 **`restrict` en vez de `shutdown`** para que *"el servicio no se vea afectado"*: si llega un intruso, se
> le descarta el tráfico y se cuenta la violación, pero el puerto **sigue funcionando** para los autorizados.

### 6.4 Requisito 5 — puerto del CCTV (`Fa0/24` de todos los switches)

Se repite **idéntico en los 6 switches** — gracias a haber movido el uplink a `Fa0/23` (§1.3), el `Fa0/24`
está libre en todos:

```cisco
interface fastEthernet 0/24
 switchport mode access
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address 3333.3333.3333   ! ← MAC del CCTV
 switchport port-security mac-address AAAA.AAAA.AAAA   ! ← MAC del Portátil IT
 switchport port-security violation shutdown
 no shutdown
 exit
```

Y en **Switch22(2) Fa0/2**, donde el CCTV está físicamente conectado según la topología:

```cisco
interface fastEthernet 0/2
 switchport mode access
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address 3333.3333.3333   ! ← MAC del CCTV
 switchport port-security mac-address AAAA.AAAA.AAAA   ! ← MAC del Portátil IT
 switchport port-security violation shutdown
 exit
```

> 💡 **Fíjate en el `no shutdown` de los `Fa0/24`.** En §7 vas a apagar todos los puertos sin usar. Los
> `Fa0/24` están sin usar pero **deben quedar encendidos**, porque el CCTV tiene que poder llegar y conectarse.
> Es la excepción a la regla.
>
> 💡 **Recuerda la condición del enunciado:** el CCTV solo dará red *"siempre y cuando tenga configurado el
> direccionamiento ip de la red"*. Port Security lo deja pasar (capa 2), pero si su IP es de Bogotá y lo
> enchufas en Cúcuta, no habrá comunicación. Son dos capas distintas.

### 6.5 Requisito 6 — Portátil IT en cualquier puerto

**No tiene comandos propios.** Se cumple porque su MAC está incluida en **todos** los bloques anteriores y
porque cada `maximum` tiene un hueco reservado para él.

Repasa que la línea

```cisco
 switchport port-security mac-address AAAA.AAAA.AAAA
```

esté presente en **todos** los puertos de acceso de los 6 switches, **excepto** en `Gig0/2` de Switch21
(el ServerDNS, ver §6.0).

> 💡 **Atajo para no repetir 20 veces lo mismo.** Cuando varios puertos llevan configuración idéntica:
>
> ```cisco
> interface range fastEthernet 0/5 - 23
>  switchport mode access
>  switchport port-security
>  switchport port-security maximum 2
>  switchport port-security mac-address AAAA.AAAA.AAAA
>  switchport port-security violation shutdown
>  exit
> ```
>
> ⚠️ Pero **no uses `interface range` a lo bruto sobre `Fa0/1 - 23`** como propone el PDF de la carpeta: te
> pisaría la configuración especial de los puertos de gerencia y secretaría.

### 6.6 Requisito 7 — sede Cali

```cisco
enable
configure terminal
hostname SwitchCali
!
interface fastEthernet 0/1
 switchport mode access
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security mac-address AAAA.AAAA.AAAA   ! ← Portátil IT
 switchport port-security violation shutdown
 exit
!
interface fastEthernet 0/2
 switchport mode access
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security mac-address AAAA.AAAA.AAAA   ! ← Portátil IT
 switchport port-security violation shutdown
 exit
```

Más el `Fa0/24` del CCTV (§6.4) y el apagado de puertos libres (§7).

---

## 7. Apagar los puertos sin usar

**Sin este paso el requisito 1 no se cumple de verdad.** Port Security protege los puertos que configuraste,
pero un equipo externo enchufado en un `Fa0/15` vacío entraría sin problema.

En **cada uno de los 6 switches**, apaga los puertos que no uses. Ejemplo para Switch22, donde solo se usan
`Fa0/1`, `Fa0/2`, `Fa0/24` y `Gig0/2`:

```cisco
interface range fastEthernet 0/3 - 23
 shutdown
 exit
!
interface gigabitEthernet 0/1
 shutdown
 exit
```

| Switch | Puertos en uso | Rango a apagar |
|---|---|---|
| Switch18 | Gig0/1, Gig0/2, **Fa0/23**, Fa0/24 | `Fa0/1 - 22` |
| Switch22 | Fa0/1, Fa0/2, Fa0/24, Gig0/2 | `Fa0/3 - 23` y `Gig0/1` |
| Switch22(2) | Fa0/1, Fa0/2, Fa0/3, **Fa0/23**, Fa0/24 | `Fa0/4 - 22` y `Gig0/1 - 2` |
| SwitchCali | Fa0/1, Fa0/2, Fa0/24, Gig0/1 | `Fa0/3 - 23` y `Gig0/2` |
| Switch20 | Fa0/2, Fa0/24, Gig0/1, Gig0/2 | `Fa0/1`, `Fa0/3 - 23` |
| Switch21 | Fa0/2, Fa0/24, Gig0/1, Gig0/2 | `Fa0/1`, `Fa0/3 - 23` |

> ⚠️ **Fíjate en Switch18 y Switch22(2):** el rango termina en **`Fa0/22`**, no en 23. `Fa0/23` es el uplink
> entre ellos y `Fa0/24` es el del CCTV — ninguno de los dos se apaga.
>
> 💡 En el mundo real esto se llama *port hardening* y es de las primeras cosas que audita un pentester: un
> puerto de red vivo y sin vigilancia en una sala de reuniones es una puerta abierta.

---

## 8. Guardar la configuración

En **los 6 switches y los 3 routers**:

```cisco
end
copy running-config startup-config
```

> ⚠️ **Especialmente importante con `sticky`.** Las MAC aprendidas viven en la `running-config`. Si no las
> guardas, al reiniciar el equipo desaparecen y toca volver a empezar.

---

# C. CIERRE

## 9. Pruebas a realizar

> 📸 Toma captura de cada una: son la evidencia de la entrega.

### 9.1 Conectividad base (antes de probar seguridad)

Desde **Secretaria (Bogotá)**, en `Desktop → Command Prompt`:

```
ping 192.168.10.1      ← su gateway
ping 192.168.20.11     ← Contaduría, en Cali
ping 192.168.30.10     ← ServerDNS, en Cúcuta
```

Las tres deben responder. Si falla la última, revisa RIP (§5.3).

### 9.2 Requisito 1 y 7 — el equipo externo no entra

1. Configura en **Maquina de empleado externo 1** una IP **correcta** de la LAN de Cali
   (`192.168.20.50 / 255.255.255.0 / gw 192.168.20.1`). *Esto es a propósito: el enunciado insiste en que ni
   así debe funcionar.*
2. Desconecta la Tesorería de `SwitchCali Fa0/2` y **conecta ahí el equipo externo**.
3. Desde el externo: `ping 192.168.20.1` → **debe fallar**.
4. En el switch:

```cisco
SwitchCali#show interfaces fastEthernet 0/2 status
```
```
Port      Name    Status        Vlan   Duplex  Speed  Type
Fa0/2             err-disabled  1      auto    auto   10/100BaseTX
```

```cisco
SwitchCali#show port-security interface fastEthernet 0/2
```
```
Port Security              : Enabled
Port Status                : Secure-shutdown
Violation Mode             : Shutdown
Last Source Address:Vlan   : 00E0.F7A1.B2C3:1     ← la MAC del intruso
Security Violation Count   : 1
```

> ✅ **Esto es el requisito 1 demostrado.** Tenía la IP perfecta y aun así no pasó, porque el bloqueo ocurre
> en capa 2. Prueba lo mismo en un puerto de Bogotá y en uno de Cúcuta.

### 9.3 Requisito 2 — puerto del ServerDNS

Desconecta el ServerDNS de `Switch21 Gig0/2` y conecta ahí **cualquier** otro equipo (incluso el Portátil IT).

```cisco
Switch21#show port-security interface gigabitEthernet 0/2
```

`Port Status: Secure-shutdown` → el puerto se bloqueó inmediatamente. ✅

### 9.4 Requisito 3 — intercambiar los equipos de gerencia

1. Desconecta **Gerencia** de `Switch22 Fa0/2` y **Gerencia Administrativa** de `Switch22(2) Fa0/1`.
2. **Crúzalos**: Gerencia Administrativa a `Switch22 Fa0/2`, Gerencia a `Switch22(2) Fa0/1`.
3. Haz ping desde cada uno a su gateway → **deben funcionar los dos**.
4. Verifica que ningún puerto cayó:

```cisco
Switch22#show interfaces fastEthernet 0/2 status     ! debe decir "connected"
```

> ✅ Requisito 3 cumplido: se intercambiaron **y el servicio no se vio afectado**.

### 9.5 Requisito 4 — intercambiar los equipos de secretaría

Haz lo mismo con **Secretaria** y **Secretaria administrativa**.

```cisco
Switch22#show interfaces fastEthernet 0/1 status     ! debe decir "err-disabled"
```

> ✅ Requisito 4 cumplido: **el servicio cae inmediatamente**, que es exactamente lo que pide el enunciado.
>
> 💡 **Esta es la prueba que mejor demuestra que entendiste `restrict` vs `shutdown`.** Ponla junto a la de
> §9.4 en la entrega, una al lado de la otra.

### 9.6 Requisito 5 — el CCTV viaja

1. Desconecta el CCTV de Bogotá.
2. Conéctalo al **`Fa0/24` de Switch20** (Cúcuta).
3. Cámbiale la IP a `192.168.30.20 / 255.255.255.0 / gw 192.168.30.1`.
4. `ping 192.168.30.1` → **funciona**. ✅
5. **Prueba adicional interesante:** vuelve a ponerle la IP de Bogotá (`192.168.10.20`) sin moverlo de sitio.
   El puerto **sigue arriba** (la MAC está autorizada) pero **el ping falla**. Eso demuestra que Port Security
   y direccionamiento IP son **dos capas independientes** — y es justo la condición que pone el enunciado.

### 9.7 Requisito 6 — el Portátil IT

Conéctalo, uno por uno, a varios puertos de distintas sedes (dale la IP `.100` de cada LAN). En todos debe
tener red, **sin tumbar el puerto**.

Luego pruébalo en `Switch21 Gig0/2` (ServerDNS): ahí **sí** debe bloquearse. ✅ Requisitos 6 y 2 conviviendo.

### 9.8 Verificación global

En cada switch:

```cisco
show port-security
show port-security address
show port-security interface fastEthernet 0/1
show interfaces status
show running-config
```

En cada router:

```cisco
show ip route
show ip route rip
show ip protocols
show ip interface brief
```

Salida esperada de `show port-security`:

```
Secure Port  MaxSecureAddr  CurrentAddr  SecurityViolation  Security Action
                (Count)       (Count)          (Count)
------------------------------------------------------------------------
   Fa0/1            2            1                0             Shutdown
   Fa0/2            3            1                0             Restrict
   Fa0/24           2            0                0             Shutdown
```

---

## 10. Cómo recuperar un puerto bloqueado

Cuando un puerto entra en `err-disabled` **no se recupera solo**. Después de cada prueba:

```cisco
Switch22#configure terminal
Switch22(config)#interface fastEthernet 0/1
Switch22(config-if)#shutdown
Switch22(config-if)#no shutdown
Switch22(config-if)#end
```

> 🔑 **Hay que hacer `shutdown` y luego `no shutdown`.** Solo con `no shutdown` no basta: el puerto sigue en
> `err-disabled` porque nunca se apagó de verdad.

Si además la MAC equivocada quedó registrada:

```cisco
Switch22#clear port-security sticky interface fastEthernet 0/1
```

Recuperación automática (opcional, muy útil mientras pruebas):

```cisco
Switch22(config)#errdisable recovery cause psecure-violation
Switch22(config)#errdisable recovery interval 60
```

> Así el puerto se reactiva solo a los 60 segundos. En producción se deja **desactivado** a propósito, para
> que un humano revise qué pasó.

---

## 11. Errores frecuentes

| Síntoma | Causa probable | Solución |
|---|---|---|
| El enlace serial queda `up/down` | Falta `clock rate` en el **DCE** | §2.3 · comprueba con `show controllers serial 0/0/0` |
| No aparecen `Serial0/0/x` en el router | Falta el módulo HWIC-2T | §1.2 (con el router **apagado**) |
| Bogotá no ve a Cúcuta | Falta `network 10.0.0.0` en algún router | §5.1 |
| Las rutas RIP salen con máscara rara | Falta `version 2` o `no auto-summary` | §5.2 · verifica con `show ip protocols` |
| Un puerto se cae en cuanto conecto algo | `maximum` demasiado bajo (olvidaste el Portátil IT) | Sube a 2 · §6.0 |
| El **uplink** entre switches se cae | Pusiste Port Security en un puerto troncal | Quítalo: `no switchport port-security` · §4.5 |
| Sticky memorizó la MAC equivocada | El equipo correcto no estaba conectado al activarlo | `clear port-security sticky interface …` y reconecta |
| El puerto sigue `err-disabled` tras arreglarlo | Solo hiciste `no shutdown` | Hay que hacer `shutdown` **y** `no shutdown` · §10 |
| Gerencia se intercambia pero el puerto cae | Pusiste `shutdown` en vez de `restrict` | §6.3 |
| Secretaría se intercambia y **no** cae | Pusiste `restrict` en vez de `shutdown`, o falta una MAC | §6.1 |
| El CCTV no da red en otra sede | No le cambiaste la IP a la de esa LAN | §2.2 · es capa 3, no Port Security |
| El equipo externo **sí** entra | Lo conectaste a un puerto libre sin configurar | Apaga los puertos sin usar · §7 |
| Todo se pierde al reiniciar | No guardaste | `copy running-config startup-config` · §8 |
| `% Invalid input` en `switchport port-security` | El puerto no está en modo acceso | Escribe antes `switchport mode access` |

---

## 12. Conceptos y glosario

**Port Security.** Función de capa 2 que limita **qué** direcciones MAC y **cuántas** pueden usar un puerto de
switch. Es la defensa básica contra que alguien enchufe su equipo a una toma de red libre.

**Dirección MAC.** Identificador único de 48 bits grabado en cada tarjeta de red (`0001.C4A1.B2C3`). A
diferencia de la IP —que se configura y se cambia— la MAC identifica al **hardware**. Por eso el control por
MAC funciona aunque el intruso ponga la IP correcta.

**MAC sticky.** MAC aprendida automáticamente por el puerto y guardada en la configuración. Combina la
comodidad del aprendizaje dinámico con la permanencia del registro estático.

**err-disabled.** Estado en el que el IOS deja un puerto tras una violación grave. No transmite nada y solo se
recupera manualmente (`shutdown` + `no shutdown`) o con `errdisable recovery`.

**Modos de violación.** `protect` descarta en silencio · `restrict` descarta y avisa · `shutdown` apaga el
puerto. El enunciado exige distinguir entre `restrict` (gerencia, requisito 3) y `shutdown` (secretaría,
requisito 4).

**Puerto de acceso vs uplink.** El de acceso lleva **un** equipo final y es donde va Port Security. El uplink
(switch↔switch o switch↔router) transporta el tráfico de muchos equipos y **nunca** debe llevarlo.

**RIP (*Routing Information Protocol*).** Protocolo de enrutamiento dinámico por **vector distancia**: cada
router anuncia a sus vecinos las redes que conoce y a cuántos **saltos** están. Máximo 15 saltos.

**RIPv2 vs RIPv1.** v2 envía la **máscara** junto con la red (soporta VLSM/CIDR), usa multicast `224.0.0.9` y
admite autenticación. v1 no envía máscara y por eso no sirve con subredes como los `/30` de esta práctica.

**`no auto-summary`.** Desactiva el resumen automático a la red *classful*. Sin él, `10.0.0.0/30` se anunciaría
como `10.0.0.0/8` y el enrutamiento fallaría.

**Distancia administrativa.** Grado de confianza en el origen de una ruta. RIP = **120**; una ruta estática =
1; una red conectada = 0. Menor número gana.

**`passive-interface`.** Le dice al router que no envíe anuncios de enrutamiento por esa interfaz. Se pone en
las interfaces LAN: los PCs no necesitan oír RIP, y evitarlo reduce tráfico y superficie de ataque.

**Capa 2 vs capa 3 — la idea que atraviesa toda la práctica.** Port Security decide **quién puede hablar**
(MAC, capa 2). El direccionamiento IP y RIP deciden **hacia dónde va lo que se habla** (capa 3). Son
independientes: un equipo puede pasar el filtro de capa 2 y aun así no comunicarse por tener mal la IP —y al
revés, tener la IP perfecta y no pasar del capa 2.

---

## 13. Configuración completa por equipo

> Reemplaza las MAC de ejemplo por las de tu archivo (§3.2):
> `1111.1111.1111` = Gerencia · `2222.2222.2222` = Gerencia Administrativa ·
> `3333.3333.3333` = CCTV · `AAAA.AAAA.AAAA` = Portátil IT

<details>
<summary><b>Switch22 — Bogotá (Secretaria + Gerencia)</b></summary>

```cisco
enable
configure terminal
hostname Switch22
!
interface fastEthernet 0/1
 switchport mode access
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security mac-address AAAA.AAAA.AAAA
 switchport port-security violation shutdown
 exit
!
interface fastEthernet 0/2
 switchport mode access
 switchport port-security
 switchport port-security maximum 3
 switchport port-security mac-address 1111.1111.1111
 switchport port-security mac-address 2222.2222.2222
 switchport port-security mac-address AAAA.AAAA.AAAA
 switchport port-security violation restrict
 exit
!
interface fastEthernet 0/24
 switchport mode access
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address 3333.3333.3333
 switchport port-security mac-address AAAA.AAAA.AAAA
 switchport port-security violation shutdown
 no shutdown
 exit
!
interface range fastEthernet 0/3 - 23
 shutdown
 exit
interface gigabitEthernet 0/1
 shutdown
 exit
!
end
copy running-config startup-config
```
</details>

<details>
<summary><b>Switch22(2) — Bogotá (Gerencia Adm + CCTV + Secretaria adm)</b></summary>

```cisco
enable
configure terminal
hostname Switch22-2
!
interface fastEthernet 0/1
 switchport mode access
 switchport port-security
 switchport port-security maximum 3
 switchport port-security mac-address 1111.1111.1111
 switchport port-security mac-address 2222.2222.2222
 switchport port-security mac-address AAAA.AAAA.AAAA
 switchport port-security violation restrict
 exit
!
interface fastEthernet 0/2
 switchport mode access
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address 3333.3333.3333
 switchport port-security mac-address AAAA.AAAA.AAAA
 switchport port-security violation shutdown
 exit
!
interface fastEthernet 0/3
 switchport mode access
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security mac-address AAAA.AAAA.AAAA
 switchport port-security violation shutdown
 exit
!
! Fa0/23 es el UPLINK a Switch18: no lleva port-security ni se apaga.
!
interface fastEthernet 0/24
 switchport mode access
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address 3333.3333.3333
 switchport port-security mac-address AAAA.AAAA.AAAA
 switchport port-security violation shutdown
 no shutdown
 exit
!
interface range fastEthernet 0/4 - 22
 shutdown
 exit
interface range gigabitEthernet 0/1 - 2
 shutdown
 exit
!
end
copy running-config startup-config
```
</details>

<details>
<summary><b>Switch18 — Bogotá (núcleo, solo uplinks)</b></summary>

```cisco
enable
configure terminal
hostname Switch18
!
! Gig0/1 (router), Gig0/2 (Switch22) y Fa0/23 (Switch22-2) son UPLINKS:
! NO llevan port-security.
!
! Fa0/24 SI lleva port-security: es el puerto movil del CCTV (requisito 5).
interface fastEthernet 0/24
 switchport mode access
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address 3333.3333.3333
 switchport port-security mac-address AAAA.AAAA.AAAA
 switchport port-security violation shutdown
 no shutdown
 exit
!
interface range fastEthernet 0/1 - 22
 shutdown
 exit
!
end
copy running-config startup-config
```
</details>

<details>
<summary><b>SwitchCali</b></summary>

```cisco
enable
configure terminal
hostname SwitchCali
!
interface range fastEthernet 0/1 - 2
 switchport mode access
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security mac-address AAAA.AAAA.AAAA
 switchport port-security violation shutdown
 exit
!
interface fastEthernet 0/24
 switchport mode access
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address 3333.3333.3333
 switchport port-security mac-address AAAA.AAAA.AAAA
 switchport port-security violation shutdown
 no shutdown
 exit
!
interface range fastEthernet 0/3 - 23
 shutdown
 exit
interface gigabitEthernet 0/2
 shutdown
 exit
!
end
copy running-config startup-config
```
</details>

<details>
<summary><b>Switch20 — Cúcuta (PC15)</b></summary>

```cisco
enable
configure terminal
hostname Switch20
!
interface fastEthernet 0/2
 switchport mode access
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security mac-address AAAA.AAAA.AAAA
 switchport port-security violation shutdown
 exit
!
interface fastEthernet 0/24
 switchport mode access
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address 3333.3333.3333
 switchport port-security mac-address AAAA.AAAA.AAAA
 switchport port-security violation shutdown
 no shutdown
 exit
!
interface fastEthernet 0/1
 shutdown
 exit
interface range fastEthernet 0/3 - 23
 shutdown
 exit
!
end
copy running-config startup-config
```
</details>

<details>
<summary><b>Switch21 — Cúcuta (PC16 + ServerDNS)</b></summary>

```cisco
enable
configure terminal
hostname Switch21
!
interface fastEthernet 0/2
 switchport mode access
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security mac-address AAAA.AAAA.AAAA
 switchport port-security violation shutdown
 exit
!
! *** PUERTO DEL SERVIDOR DNS - Requisito 2 ***
! maximum 1 y SIN la MAC del portatil IT, a proposito.
interface gigabitEthernet 0/2
 switchport mode access
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
 exit
!
interface fastEthernet 0/24
 switchport mode access
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address 3333.3333.3333
 switchport port-security mac-address AAAA.AAAA.AAAA
 switchport port-security violation shutdown
 no shutdown
 exit
!
interface fastEthernet 0/1
 shutdown
 exit
interface range fastEthernet 0/3 - 23
 shutdown
 exit
!
end
copy running-config startup-config
```
</details>

<details>
<summary><b>Los 3 routers (interfaces + RIPv2)</b></summary>

```cisco
! ===== Router5 - BOGOTA =====
enable
configure terminal
hostname Router5
interface gigabitEthernet 0/1
 ip address 192.168.10.1 255.255.255.0
 no shutdown
 exit
interface serial 0/0/0
 ip address 10.0.0.1 255.255.255.252
 clock rate 64000
 no shutdown
 exit
router rip
 version 2
 no auto-summary
 network 192.168.10.0
 network 10.0.0.0
 passive-interface gigabitEthernet 0/1
 exit
end
copy running-config startup-config

! ===== Router6 - CALI =====
enable
configure terminal
hostname Router6
interface gigabitEthernet 0/1
 ip address 192.168.20.1 255.255.255.0
 no shutdown
 exit
interface serial 0/0/1
 ip address 10.0.0.2 255.255.255.252
 no shutdown
 exit
interface serial 0/0/0
 ip address 10.0.0.5 255.255.255.252
 clock rate 64000
 no shutdown
 exit
router rip
 version 2
 no auto-summary
 network 192.168.20.0
 network 10.0.0.0
 passive-interface gigabitEthernet 0/1
 exit
end
copy running-config startup-config

! ===== Router5(1) - CUCUTA =====
enable
configure terminal
hostname Router5-1
interface gigabitEthernet 0/0
 ip address 192.168.30.1 255.255.255.0
 no shutdown
 exit
interface serial 0/0/1
 ip address 10.0.0.6 255.255.255.252
 no shutdown
 exit
router rip
 version 2
 no auto-summary
 network 192.168.30.0
 network 10.0.0.0
 passive-interface gigabitEthernet 0/0
 exit
end
copy running-config startup-config
```
</details>

---

## 14. Checklist de entrega

**Montaje**

- [ ] 3 routers 1941 con módulo **HWIC-2T** instalado (con el equipo apagado)
- [ ] 6 switches 2960-24TT
- [ ] 9 equipos finales cableados + 3 sueltos (Portátil IT y los 2 externos)
- [ ] Cables **cruzados** en los enlaces switch↔switch
- [ ] Cables **seriales DCE** en los 2 enlaces WAN, con el reloj en el extremo correcto
- [ ] Enlace Switch18 ↔ Switch22(2) movido a **`Fa0/23`** en ambos extremos (§1.3)

**Capa 3**

- [ ] IPs y gateways en los 10 equipos finales
- [ ] Interfaces de los 3 routers con IP y `no shutdown`
- [ ] `clock rate` en los 2 DCE
- [ ] RIPv2 con `version 2` y `no auto-summary` en los 3 routers
- [ ] `show ip route` muestra las 3 LAN y las 2 WAN en cada router
- [ ] Ping Bogotá → Cali → Cúcuta funcionando **antes** de tocar Port Security

**Capa 2 — los 7 requerimientos**

- [ ] **R1** · Port Security en todos los puertos de acceso + puertos libres apagados
- [ ] **R2** · `Switch21 Gig0/2` con `maximum 1` y solo el ServerDNS
- [ ] **R3** · Gerencia: ambas MAC en ambos puertos + `violation restrict`
- [ ] **R4** · Secretaría: 1 MAC propia + `violation shutdown`
- [ ] **R5** · MAC del CCTV en `Fa0/24` de todos los switches (y esos puertos **encendidos**)
- [ ] **R6** · MAC del Portátil IT en todos los puertos de acceso, con `maximum` +1
- [ ] **R7** · Cali con Port Security en Fa0/1 y Fa0/2 + resto apagado
- [ ] Ningún uplink lleva Port Security

**Evidencias (capturas)**

- [ ] `show ip route` de los 3 routers
- [ ] Externo con IP correcta → puerto en `err-disabled` (R1)
- [ ] Puerto del ServerDNS bloqueado al conectar otro equipo (R2)
- [ ] Gerencia intercambiada → **sigue funcionando** (R3)
- [ ] Secretaría intercambiada → **puerto caído** (R4)
- [ ] CCTV funcionando en el `Fa0/24` de otra sede (R5)
- [ ] Portátil IT funcionando en 3 puertos distintos (R6)
- [ ] `show port-security` y `show port-security address` de cada switch

**Cierre**

- [ ] `copy running-config startup-config` en los 9 equipos
- [ ] Archivo guardado como `Port-Security-RIPv2.pkt`
- [ ] Mencionadas en el informe las dos contradicciones detectadas y cómo se resolvieron: `Fa0/24` → `Fa0/23` (§1.3) y R2 vs R6 (§6.0)
