---
tipo: laboratorio
practica: "Práctica 3 — Configuración de routing entre VLAN basado en enlaces troncales"
fuente: "05_infraestructura_tecnologica_2_laboratorio_3 261.docx (CCNA lab 5.1.3.7)"
software: Cisco Packet Tracer 7.x / 8.x
creado: 2026-08-18
tags: [redes, vlan, intervlan, router-on-a-stick, 802.1q, subinterfaces, packet-tracer, cisco]
---

# Práctica 3 — Routing entre VLAN (router-on-a-stick)

> **Objetivo:** comunicar VLAN distintas usando **un solo enlace físico** entre el router y el switch. La interfaz `G0/1` del router se divide en **subinterfaces**, una por VLAN, cada una con su propia IP y su etiqueta 802.1Q.

**Entrega que pide el docente** (textual): *"Formato de pdf con la resolución de la actividad utilizando normas APA. Pkt de las actividades solicitadas en packet tracert."*
→ Las respuestas están en [[entregable]]. El archivo de simulación se guarda como `P3-InterVLAN.pkt` en esta misma carpeta.

## Contenido

| | Alcance | Secciones |
|---|---|---|
| **A. Montaje** | Arquitectura, componentes, cableado, IPs | §1 – §4 |
| **B. Todo en uno** | Config completa de R1, S1 y S2 de un solo pegue | §5 |
| **C. Por partes** | El recorrido de la guía, parte por parte | §6 – §8 |
| **D. Cierre** | Pruebas, capturas, errores frecuentes, checklist | §9 – §12 |

---

# A. MONTAJE

## 1. Arquitectura

```
                        ┌──────────┐
                        │   Lo0    │  209.165.200.225/27
                        └────┬─────┘
                        ┌────┴─────┐
                        │    R1    │   G0/1.1  → 192.168.1.1
                        └────┬─────┘   G0/1.10 → 192.168.10.1
                       G0/1  │         G0/1.20 → 192.168.20.1
                             │  ← troncal 802.1Q
                        F0/5 │
        ┌────────────────────┴──┐                  ┌───────────┐
        │          S1           │ F0/1 ══════ F0/1 │    S2     │
        └──┬────────────────────┘   troncal        └────────┬──┘
     F0/6  │                          802.1Q                │ F0/18
           │                                                │
       ┌───┴──┐                                         ┌───┴──┐
       │ PC-A │  192.168.10.3                           │ PC-B │  192.168.20.3
       └──────┘  VLAN 10                                └──────┘  VLAN 20
```

**La idea en una frase:** un cable, tres redes. El router no tiene una interfaz física por VLAN; tiene **una interfaz lógica (subinterfaz) por VLAN** sobre el mismo cable físico, y cada subinterfaz sabe con qué etiqueta 802.1Q hablar.

### Tabla de direccionamiento

| Dispositivo | Interfaz | Dirección IP | Máscara | Gateway |
|---|---|---|---|---|
| R1 | `G0/1.1` | 192.168.1.1 | 255.255.255.0 | N/A |
| R1 | `G0/1.10` | 192.168.10.1 | 255.255.255.0 | N/A |
| R1 | `G0/1.20` | 192.168.20.1 | 255.255.255.0 | N/A |
| R1 | `Lo0` | 209.165.200.225 | **255.255.255.224** (/27) | N/A |
| S1 | VLAN 1 | 192.168.1.11 | 255.255.255.0 | 192.168.1.1 |
| S2 | VLAN 1 | 192.168.1.12 | 255.255.255.0 | 192.168.1.1 |
| PC-A | NIC | 192.168.10.3 | 255.255.255.0 | 192.168.10.1 |
| PC-B | NIC | 192.168.20.3 | 255.255.255.0 | 192.168.20.1 |

> 🔑 **Diferencia clave con la Práctica 2.** Aquí los gateways de los PCs **sí existen**: son las subinterfaces del router. Por eso PC-A y PC-B, que están en VLAN distintas, ahora sí se comunican.
>
> 🔑 **La VLAN 1 también necesita su subinterfaz** (`G0/1.1`). Es la VLAN de administración de los switches; sin ella, los switches quedarían sin gateway.

### Asignación de puertos

| Switch | Puerto | Asignación | Red |
|---|---|---|---|
| S1 | `F0/1` | Enlace troncal 802.1Q (hacia S2) | N/A |
| S1 | `F0/5` | Enlace troncal 802.1Q (hacia R1) | N/A |
| S1 | `F0/6` | VLAN 10 — `Students` | 192.168.10.0/24 |
| S2 | `F0/1` | Enlace troncal 802.1Q (hacia S1) | N/A |
| S2 | `F0/18` | VLAN 20 — `Faculty` | 192.168.20.0/24 |

> ⚠️ **Corrección al enunciado.** La tabla *"Especificaciones de la asignación de puertos de switch"* del `.docx` quedó **truncada**: solo trae la fila de `S1 F0/1` y le faltan las otras cuatro. Las reconstruí a partir de la topología y del Apéndice A del mismo documento, que sí menciona `f0/5`, `f0/6`, `f0/18` y los nombres `Students` y `Faculty`. **Vale la pena mencionarlo en el informe.**

---

## 2. Componentes y dónde encontrarlos

| Qué necesitas | Categoría (inglés) | Categoría (español) | Modelo |
|---|---|---|---|
| 1 router | **Network Devices → Routers** | Dispositivos de red → Enrutadores | **2901** (o **1941**) |
| 2 switches | **Network Devices → Switches** | Dispositivos de red → Conmutadores | **2960-24TT** |
| 2 PCs | **End Devices → End Devices** | Dispositivos finales | **PC** (`PC-PT`) |
| Cable router↔switch | **Connections** | Conexiones | **Copper Straight-Through** |
| Cable PC↔switch | **Connections** | Conexiones | **Copper Straight-Through** |
| Cable switch↔switch | **Connections** | Conexiones | **Copper Cross-Over** (punteado) |

> 💡 **2901 o 1941, da igual.** La guía pide un *Cisco 2901 con IOS 15.2(4)M3*. Los dos traen `Gig0/0` y `Gig0/1` de fábrica, que es todo lo que hace falta. **No** se necesita ningún módulo adicional en esta práctica.
>
> 💡 **El Lo0 no se cablea.** Una interfaz *loopback* es virtual: se crea con un comando y siempre está arriba. Aquí simula una red externa a la que llegar.

---

## 3. Cableado

| # | Desde | Puerto | Hacia | Puerto | Cable |
|---|---|---|---|---|---|
| 1 | R1 | `Gig0/1` | S1 | `Fa0/5` | Directo |
| 2 | S1 | `Fa0/1` | S2 | `Fa0/1` | **Cruzado** |
| 3 | PC-A | `Fa0` | S1 | `Fa0/6` | Directo |
| 4 | PC-B | `Fa0` | S2 | `Fa0/18` | Directo |

---

## 4. Configuración de los PCs

`Desktop → IP Configuration → Static`:

| PC   | IP             | Máscara         | Gateway        |
| ---- | -------------- | --------------- | -------------- |
| PC-A | `192.168.10.3` | `255.255.255.0` | `192.168.10.1` |
| PC-B | `192.168.20.3` | `255.255.255.0` | `192.168.20.1` |

---

# B. TODO EN UNO

## 5. Configuración completa — copiar y pegar

Tres bloques, en este orden. Al terminar, la práctica está lista.

### R1 — router

```cisco
enable
configure terminal
no ip domain-lookup
hostname R1
enable secret class
banner motd #Acceso no autorizado prohibido. Uso exclusivo personal autorizado UMB.#
!
line console 0
 password cisco
 login
 logging synchronous
 exit
line vty 0 4
 password cisco
 login
 exit
!
interface loopback 0
 ip address 209.165.200.225 255.255.255.224
 exit
!
interface gigabitEthernet 0/1.1
 encapsulation dot1Q 1 native
 ip address 192.168.1.1 255.255.255.0
 exit
!
interface gigabitEthernet 0/1.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 exit
!
interface gigabitEthernet 0/1.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 exit
!
interface gigabitEthernet 0/1
 no shutdown
 exit
!
end
copy running-config startup-config
```

> 🔑 **El `no shutdown` va en la interfaz FÍSICA `G0/1`, no en las subinterfaces.** Si la física está apagada, las tres subinterfaces quedan abajo por más bien configuradas que estén. Es el error número uno de esta práctica.
>
> 🔑 **`encapsulation dot1Q 1 native`.** La VLAN 1 es la nativa del troncal y su tráfico viaja **sin etiqueta**. La palabra `native` le dice al router que espere tramas sin etiquetar en esa subinterfaz. Sin ella, la comunicación con las SVI de los switches falla. *(El Apéndice A del enunciado la omite; es una corrección necesaria.)*

### S1 — switch

```cisco
enable
configure terminal
no ip domain-lookup
hostname S1
enable secret class
banner motd #Acceso no autorizado prohibido. Uso exclusivo personal autorizado UMB.#
!
line console 0
 password cisco
 login
 logging synchronous
 exit
line vty 0 15
 password cisco
 login
 exit
!
vlan 10
 name Students
 exit
vlan 20
 name Faculty
 exit
!
interface vlan 1
 ip address 192.168.1.11 255.255.255.0
 no shutdown
 exit
ip default-gateway 192.168.1.1
!
interface fastEthernet 0/1
 switchport mode trunk
 exit
interface fastEthernet 0/5
 switchport mode trunk
 exit
!
interface fastEthernet 0/6
 switchport mode access
 switchport access vlan 10
 exit
!
interface range fastEthernet 0/2 - 4
 shutdown
 exit
interface range fastEthernet 0/7 - 24
 shutdown
 exit
interface range gigabitEthernet 0/1 - 2
 shutdown
 exit
!
end
copy running-config startup-config
```

### S2 — switch

```cisco
enable
configure terminal
no ip domain-lookup
hostname S2
enable secret class
banner motd #Acceso no autorizado prohibido. Uso exclusivo personal autorizado UMB.#
!
line console 0
 password cisco
 login
 logging synchronous
 exit
line vty 0 15
 password cisco
 login
 exit
!
vlan 10
 name Students
 exit
vlan 20
 name Faculty
 exit
!
interface vlan 1
 ip address 192.168.1.12 255.255.255.0
 no shutdown
 exit
ip default-gateway 192.168.1.1
!
interface fastEthernet 0/1
 switchport mode trunk
 exit
!
interface fastEthernet 0/18
 switchport mode access
 switchport access vlan 20
 exit
!
interface range fastEthernet 0/2 - 17
 shutdown
 exit
interface range fastEthernet 0/19 - 24
 shutdown
 exit
interface range gigabitEthernet 0/1 - 2
 shutdown
 exit
!
end
copy running-config startup-config
```

> ℹ️ **La VLAN 10 se crea también en S2 aunque ahí no haya ningún PC en ella.** El enunciado lo pide y tiene sentido: el troncal solo transporta VLAN que existen en la base de datos del switch. Si S2 no conoce la VLAN 10, descarta ese tráfico.

---

# C. PARTE POR PARTE (como lo pide la guía)

## 6. Parte 1 — parámetros básicos

Es el primer bloque de cada dispositivo en §5: `no ip domain-lookup`, `hostname`, `enable secret class`, consola y vty con `cisco`, `logging synchronous`, banner MOTD, SVI de la VLAN 1, `ip default-gateway` en los switches y el `Lo0` en el router.

> La guía dice explícitamente: *"No configure las subinterfaces en esta instancia; esto lo hará en la parte 3."*

📸 **Captura 1:** `show ip interface brief` de R1 con `Lo0` arriba y `G0/1` todavía en `administratively down`.

## 7. Parte 2 — VLAN y troncales en los switches

Los comandos que pide escribir la guía en los espacios en blanco:

**Crear las VLAN en S1:**

```cisco
vlan 10
 name Students
 exit
vlan 20
 name Faculty
 exit
```

**Configurar los troncales en S1:**

```cisco
interface fastEthernet 0/1
 switchport mode trunk
 exit
interface fastEthernet 0/5
 switchport mode trunk
 exit
```

**Asignar el puerto de acceso de PC-A en S1:**

```cisco
interface fastEthernet 0/6
 switchport mode access
 switchport access vlan 10
 exit
```

**Verificar que las VLAN de S2 coinciden con las de S1:**

```cisco
show vlan brief
```

**En S2, puerto de acceso de PC-B y troncal:**

```cisco
interface fastEthernet 0/18
 switchport mode access
 switchport access vlan 20
 exit
interface fastEthernet 0/1
 switchport mode trunk
 exit
```

📸 **Captura 2:** `show vlan brief` de S1 y de S2 uno al lado del otro.
📸 **Captura 3:** `show interfaces trunk` de S1 mostrando **dos** troncales (`Fa0/1` y `Fa0/5`).

## 8. Parte 3 — routing entre VLAN

> **Todos los comandos de esta parte van en el ROUTER `R1`.** Los switches ya quedaron listos en la Parte 2; aquí solo se toca el router. Entra por `Router → pestaña CLI`.

**0) Punto de partida (router recién encendido).** Si arranca con el diálogo de configuración inicial, respóndelo con `no` y sitúate en modo de configuración global:

```cisco
! --- En R1 ---
Would you like to enter the initial configuration dialog? [yes/no]: no

enable
configure terminal
```

> 💡 Si vienes de hacer la Parte 1 en este mismo router, ya tendrá `hostname R1` y el prompt se verá como `R1#`; en ese caso basta con `configure terminal`. Si el prompt se ve como `Router>`, escribe primero `enable`.

**1) Subinterfaz para la VLAN 1 (administración de los switches) — en R1:**

```cisco
! --- En R1 ---
interface gigabitEthernet 0/1.1
 encapsulation dot1Q 1 native
 ip address 192.168.1.1 255.255.255.0
 exit
```

**2) Subinterfaz para la VLAN 10 (gateway de PC-A) — en R1:**

```cisco
! --- En R1 ---
interface gigabitEthernet 0/1.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 exit
```

**3) Subinterfaz para la VLAN 20 (gateway de PC-B) — en R1:**

```cisco
! --- En R1 ---
interface gigabitEthernet 0/1.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 exit
```

**4) Habilitar la interfaz FÍSICA y guardar — en R1:**

```cisco
! --- En R1 ---
interface gigabitEthernet 0/1
 no shutdown
 exit
!
end
copy running-config startup-config
```

> 🔑 **El `no shutdown` va en `G0/1` (la interfaz física), no en las subinterfaces.** Recién encendido el router, `G0/1` está en `administratively down`; si no la levantas, las tres subinterfaces quedan caídas por bien configuradas que estén. Es el error número uno de esta práctica.
>
> 🔑 **¿Por qué `G0/1.1` y no da error?** El número después del punto es solo la etiqueta de la subinterfaz; lo que de verdad importa es que `encapsulation dot1Q 1 native` coincida con la VLAN nativa del troncal. La VLAN 1 viaja **sin etiqueta**, y la palabra `native` es lo que le dice al router que la espere así. Sin `native`, esta subinterfaz no se comunica con las SVI de administración de los switches. *(El Apéndice A del enunciado omite `native`; es una corrección necesaria.)*

**5) Verificar — en R1:**

```cisco
! --- En R1 ---
show ip route
show ip interface brief
```

En `show ip interface brief`, las cuatro interfaces `G0/1`, `G0/1.1`, `G0/1.10` y `G0/1.20` deben aparecer en **up / up**. Si `G0/1` sale como `administratively down`, te faltó el `no shutdown` del paso 4.

> ℹ️ **Primer ping.** Al probar la comunicación (§9), el primer intento suele salir `Request timed out` mientras se resuelve el ARP; repite el ping y ya deben responder los cuatro paquetes.

📸 **Captura 4:** `show ip route` de R1 con las cuatro redes conectadas.

---

# D. CIERRE

## 9. Pruebas

Desde `Desktop → Command Prompt` de PC-A:

```
ping 192.168.10.1      ← su gateway (subinterfaz G0/1.10)
ping 192.168.20.3      ← PC-B, en la VLAN 20
ping 209.165.200.225   ← el loopback del router
ping 192.168.1.12      ← S2
```

**Las cuatro deben responder.** Si alguna falla, ve a §11.

📸 **Captura 5:** los cuatro pings de PC-A en una sola pantalla. Es la evidencia central de la práctica: un PC de la VLAN 10 alcanzando la VLAN 20, la red del router y la VLAN de administración.

### Qué debe mostrar `show ip route` en R1

| Red | Origen |
|---|---|
| `192.168.1.0/24` | C — conectada por `G0/1.1` |
| `192.168.10.0/24` | C — conectada por `G0/1.10` |
| `192.168.20.0/24` | C — conectada por `G0/1.20` |
| `209.165.200.224/27` | C — conectada por `Lo0` |

**No hay rutas estáticas ni protocolo de enrutamiento.** Las cuatro redes están directamente conectadas al router; ese es el punto de router-on-a-stick.

### Verificación completa

```cisco
! En R1
show ip route
show ip interface brief
show interfaces gigabitEthernet 0/1.10
show vlans

! En S1 y S2
show vlan brief
show interfaces trunk
show ip interface brief
show running-config
```

## 10. Capturas para la entrega

| # | Captura | Sección |
|---|---|---|
| 1 | `show ip interface brief` de R1 antes de las subinterfaces | §6 |
| 2 | `show vlan brief` de S1 y S2 | §7 |
| 3 | `show interfaces trunk` de S1 (dos troncales) | §7 |
| 4 | `show ip route` de R1 con las 4 redes | §8 |
| 5 | Los 4 pings de PC-A funcionando | §9 |
| 6 | `show vlans` de R1 (tráfico por subinterfaz) | §9 |
| 7 | Topología completa con todos los enlaces en verde | — |

## 11. Errores frecuentes

| Síntoma | Causa | Solución |
|---|---|---|
| Ninguna subinterfaz levanta | La interfaz física `G0/1` está apagada | `interface g0/1` → `no shutdown` |
| PC-A no llega a su gateway | El puerto del PC no está en la VLAN correcta, o el `encapsulation dot1Q` tiene otro número | Deben coincidir el ID de VLAN del switch y el del `encapsulation` |
| Los PCs se ven pero los switches no | Falta `native` en `G0/1.1`, o falta `ip default-gateway` en los switches | §5 |
| `Native VLAN mismatch` en la consola | La VLAN nativa no coincide entre los dos extremos del troncal | Déjala en 1 en todos, o cámbiala en todos |
| El troncal hacia el router no funciona | `F0/5` quedó en modo acceso | `switchport mode trunk` |
| PC-A llega al router pero no a PC-B | La VLAN 20 no existe en S1, o el troncal S1↔S2 no está | Crea la VLAN 20 en **ambos** switches |
| `% Invalid input` al escribir `encapsulation` | Lo intentaste en la interfaz física en vez de la subinterfaz | Debe ser `interface g0/1.10`, con punto |
| El ping al `Lo0` falla | El `Lo0` no se creó o tiene mala máscara | Debe ser `/27` = `255.255.255.224` |
| Todo se pierde al reiniciar | Falta guardar | `copy running-config startup-config` |

## 12. Checklist de entrega

- [ ] 1 router 2901/1941 + 2 switches 2960-24TT + 2 PCs, cableado según §3
- [ ] IPs de PC-A y PC-B con sus gateways
- [ ] Configuración básica en los 3 dispositivos: hostname, `enable secret class`, vty y consola con `cisco`, `logging synchronous`, banner, `no ip domain-lookup`
- [ ] `Lo0` en R1 con `209.165.200.225/27`
- [ ] Las 3 subinterfaces en `G0/1`, con `native` en la de la VLAN 1
- [ ] `no shutdown` en la interfaz **física** `G0/1`
- [ ] VLAN 10 `Students` y 20 `Faculty` creadas en **ambos** switches
- [ ] SVI de la VLAN 1 y `ip default-gateway 192.168.1.1` en ambos switches
- [ ] Troncales: S1 `F0/1` y `F0/5` · S2 `F0/1`
- [ ] PC-A en `F0/6` → VLAN 10 · PC-B en `F0/18` → VLAN 20
- [ ] Puertos sin usar apagados
- [ ] Los 4 pings de PC-A respondiendo
- [ ] `copy running-config startup-config` en los 3 dispositivos
- [ ] Las 7 capturas de §10
- [ ] `entregable.md` con las preguntas respondidas → pasado a PDF con normas APA
- [ ] Archivo `P3-InterVLAN.pkt` guardado en esta carpeta
- [ ] Mencionada en el informe la tabla truncada del enunciado (§1) y el `native` faltante en el Apéndice A (§5)
