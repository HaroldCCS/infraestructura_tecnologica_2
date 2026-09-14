---
tipo: laboratorio
practica: "Práctica 2 — Configuración de redes VLAN y enlaces troncales"
fuente: "05_infraestructura_tecnologica_2_laboratorio_2 261.docx (CCNA lab 3.2.2.5)"
software: Cisco Packet Tracer 7.x / 8.x
creado: 2026-08-18
tags: [redes, vlan, trunking, 802.1q, stp, vtp, packet-tracer, cisco]
---

# Práctica 2 — VLAN y enlaces troncales

> **Objetivo:** crear VLAN en dos switches, asignarlas a puertos de acceso, verificar el aislamiento entre ellas y después unirlas con un enlace troncal 802.1Q para que los hosts de la misma VLAN se comuniquen aunque estén en switches distintos.

**Entrega que pide el docente** (textual): *"Formato de pdf con la resolución de la actividad utilizando normas APA. Pkt de las actividades solicitadas en packet tracert."*
→ Las respuestas están en [[entregable]]. El archivo de simulación se guarda como `P2-VLAN-Trunking.pkt` en esta misma carpeta.

## Contenido

| | Alcance | Secciones |
|---|---|---|
| **A. Montaje** | Arquitectura, componentes, cableado, IPs | §1 – §4 |
| **B. Ruta rápida** | Config final de S1 y S2 de un solo pegue | §5 |
| **C. Sesiones** | El recorrido de la guía, sesión por sesión, con las pruebas | §6 – §9 |
| **D. Extras** | STP (ilustraciones 1-3) y ejercicio "Password UMB" | §10 – §11 |
| **E. Cierre** | Capturas, errores frecuentes, checklist | §12 – §14 |

---

# A. MONTAJE

## 1. Arquitectura

```
                    Enlace troncal 802.1Q
                    ┌────────────────────┐
        ┌───────────┴──┐              ┌──┴───────────┐
        │      S1      │ F0/1 —— F0/1 │      S2      │
        └──┬───────────┘              └──┬────────┬──┘
     F0/6  │                       F0/11 │        │ F0/18
           │                             │        │
        ┌──┴───┐                     ┌───┴──┐  ┌──┴───┐
        │ PC-A │                     │ PC-B │  │ PC-C │
        └──────┘                     └──────┘  └──────┘
        VLAN 10                      VLAN 10   VLAN 20
       (Estudiante)                (Estudiante) (Cuerpo docente)
```

**No hay router en esta práctica.** Eso es a propósito: la VLAN 10 y la VLAN 20 quedan incomunicadas, y ese aislamiento es justamente lo que hay que demostrar. Comunicarlas es el tema de la Práctica 3.

### Tabla de direccionamiento

| Dispositivo | Interfaz | Dirección IP | Máscara | Gateway |
|---|---|---|---|---|
| S1 | VLAN 1 → luego **VLAN 99** | 192.168.1.11 | 255.255.255.0 | N/A |
| S2 | VLAN 1 → luego **VLAN 99** | 192.168.1.12 | 255.255.255.0 | N/A |
| PC-A | NIC | 192.168.10.3 | 255.255.255.0 | 192.168.10.1 |
| PC-B | NIC | 192.168.10.4 | 255.255.255.0 | 192.168.10.1 |
| PC-C | NIC | 192.168.20.3 | 255.255.255.0 | 192.168.20.1 |

> ℹ️ Los gateways `192.168.10.1` y `192.168.20.1` **no existen** en esta topología (no hay router). Se escriben igual porque la tabla del docente los pide y porque en la Práctica 3 sí existirán.

### VLAN a crear (en los dos switches)

| ID | Nombre | Uso |
|---|---|---|
| 10 | `Student` | PC-A y PC-B |
| 20 | `Faculty` | PC-C |
| 99 | `Management` | SVI de administración de S1 y S2 |

### Asignación de puertos

| Switch | Puerto | Asignación |
|---|---|---|
| S1 | F0/1 | Enlace troncal 802.1Q hacia S2 |
| S1 | F0/6 | VLAN 10 — PC-A |
| S2 | F0/1 | Enlace troncal 802.1Q hacia S1 |
| S2 | F0/11 | VLAN 10 — PC-B |
| S2 | F0/18 | VLAN 20 — PC-C |

---

## 2. Componentes y dónde encontrarlos

| Qué necesitas | Categoría (inglés) | Categoría (español) | Modelo |
|---|---|---|---|
| 2 switches | **Network Devices → Switches** | Dispositivos de red → Conmutadores | **2960-24TT** |
| 3 PCs | **End Devices → End Devices** | Dispositivos finales | **PC** (`PC-PT`) |
| Cable PC↔switch | **Connections** | Conexiones | **Copper Straight-Through** (línea continua) |
| Cable switch↔switch | **Connections** | Conexiones | **Copper Cross-Over** (línea punteada) |

> 💡 **Por qué el 2960-24TT.** La guía pide un *Cisco 2960 con IOS 15.0(2) lanbasek9*. El 2960-24TT de Packet Tracer es el equivalente y sus puertos se llaman igual que en la guía: `Fa0/1`…`Fa0/24` más `Gig0/1` y `Gig0/2`.
>
> 💡 **Directo vs cruzado.** Directo para equipos distintos (PC↔switch); cruzado para equipos iguales (switch↔switch). Si te lías, el rayo dorado *Automatically Choose Connection Type* elige por ti, pero para la entrega usa el cable explícito.

---

## 3. Cableado

| # | Desde | Puerto | Hacia | Puerto | Cable |
|---|---|---|---|---|---|
| 1 | S1 | `Fa0/1` | S2 | `Fa0/1` | **Cruzado** |
| 2 | PC-A | `Fa0` | S1 | `Fa0/6` | Directo |
| 3 | PC-B | `Fa0` | S2 | `Fa0/11` | Directo |
| 4 | PC-C | `Fa0` | S2 | `Fa0/18` | Directo |

---

## 4. Configuración de los PCs

Clic en el equipo → pestaña **`Desktop`** → **`IP Configuration`** → **`Static`**.

| PC | IP | Máscara | Gateway |
|---|---|---|---|
| PC-A | `192.168.10.3` | `255.255.255.0` | `192.168.10.1` |
| PC-B | `192.168.10.4` | `255.255.255.0` | `192.168.10.1` |
| PC-C | `192.168.20.3` | `255.255.255.0` | `192.168.20.1` |

---

# B. RUTA RÁPIDA

## 5. Configuración final completa — copiar y pegar

> ⚠️ **Léelo antes de usarlo.** Estos dos bloques dejan la práctica **terminada**. Si los pegas de una, te saltas los estados intermedios que la guía pide capturar (VLAN 99 caída, ping fallando antes del trunk, etc.). Si necesitas esas capturas para la entrega, ve a §6 y sigue el orden de las sesiones.

### S1 — completo

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
 name Student
 exit
vlan 20
 name Faculty
 exit
vlan 99
 name Management
 exit
!
interface vlan 1
 no ip address
 shutdown
 exit
interface vlan 99
 ip address 192.168.1.11 255.255.255.0
 no shutdown
 exit
!
interface fastEthernet 0/1
 switchport mode trunk
 switchport trunk native vlan 99
 exit
!
interface fastEthernet 0/6
 switchport mode access
 switchport access vlan 10
 exit
!
interface range fastEthernet 0/2 - 5
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

### S2 — completo

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
 name Student
 exit
vlan 20
 name Faculty
 exit
vlan 99
 name Management
 exit
!
interface vlan 1
 no ip address
 shutdown
 exit
interface vlan 99
 ip address 192.168.1.12 255.255.255.0
 no shutdown
 exit
!
interface fastEthernet 0/1
 switchport mode trunk
 switchport trunk native vlan 99
 exit
!
interface fastEthernet 0/11
 switchport mode access
 switchport access vlan 10
 exit
!
interface fastEthernet 0/18
 switchport mode access
 switchport access vlan 20
 exit
!
interface range fastEthernet 0/2 - 10
 shutdown
 exit
interface range fastEthernet 0/12 - 17
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

> 🔑 **`switchport trunk native vlan 99` en los DOS extremos.** Si la pones en uno solo, el IOS reporta *native VLAN mismatch* y CDP/STP se quejan. O la pones en ambos, o la omites en ambos (queda la VLAN 1 nativa por defecto).

### Verificación rápida

```cisco
show vlan brief
show interfaces trunk
show ip interface brief
show interfaces fastEthernet 0/6 switchport
```

**Resultado esperado:** ping S1↔S2 ✅ · PC-A↔PC-B ✅ · PC-A↔PC-C ❌ (VLAN distintas, sin router).

---

# C. LAS SESIONES DE LA GUÍA

## 6. Sesión 1 — parámetros básicos y creación de VLAN

### 6.1 Configuración básica (antes de crear VLAN)

**S1:**

```cisco
enable
configure terminal
no ip domain-lookup
hostname S1
enable secret class
banner motd #Acceso no autorizado prohibido. Uso exclusivo personal autorizado UMB.#
line console 0
 password cisco
 login
 logging synchronous
 exit
line vty 0 15
 password cisco
 login
 exit
interface vlan 1
 ip address 192.168.1.11 255.255.255.0
 no shutdown
 exit
end
copy running-config startup-config
```

**S2:** idéntico, cambiando `hostname S2` y la IP a `192.168.1.12 255.255.255.0`.

> ℹ️ **Deja los puertos sin usar encendidos por ahora.** La guía pide apagarlos en este punto, pero en la Sesión 2 vas a asignar F0/11–24 a VLAN 10 en S1 y necesitas verlos activos. Apágalos al final (§5 ya lo incluye).

### 6.2 Prueba de conectividad (Parte 1)

Desde `Desktop → Command Prompt` de cada PC:

| Ping | Resultado | Por qué |
|---|---|---|
| PC-A → PC-B | ✅ | Misma subred `192.168.10.x`, todo está en VLAN 1 |
| PC-A → PC-C | ❌ | Subredes distintas (`.10.x` vs `.20.x`) y no hay router |
| PC-A → S1 | ❌ | S1 está en `192.168.1.11`, otra subred |
| PC-B → PC-C | ❌ | Subredes distintas |
| PC-B → S2 | ❌ | Subredes distintas |
| PC-C → S2 | ❌ | Subredes distintas |
| S1 → S2 | ✅ | Ambos en `192.168.1.x` y en VLAN 1 |

📸 **Captura 1:** los pings de PC-A (uno que funciona y uno que falla).

### 6.3 Crear las VLAN (Parte 2)

**En S1 y en S2 (los dos, el mismo bloque):**

```cisco
enable
configure terminal
vlan 10
 name Student
 exit
vlan 20
 name Faculty
 exit
vlan 99
 name Management
 exit
end
show vlan
```

📸 **Captura 2:** `show vlan` de S1 con las tres VLAN creadas.

### 6.4 Asignar puertos y mover la IP a la VLAN 99

**S1:**

```cisco
configure terminal
interface fastEthernet 0/6
 switchport mode access
 switchport access vlan 10
 exit
interface vlan 1
 no ip address
 exit
interface vlan 99
 ip address 192.168.1.11 255.255.255.0
 no shutdown
 exit
end
show vlan brief
show ip interface brief
```

**S2:**

```cisco
configure terminal
interface fastEthernet 0/11
 switchport mode access
 switchport access vlan 10
 exit
interface fastEthernet 0/18
 switchport mode access
 switchport access vlan 20
 exit
interface vlan 1
 no ip address
 exit
interface vlan 99
 ip address 192.168.1.12 255.255.255.0
 no shutdown
 exit
end
show vlan brief
```

📸 **Captura 3:** `show ip interface brief` con **VLAN 99 en `down/down`**. Es el estado esperado y una de las preguntas de la guía: la SVI no levanta porque ningún puerto activo pertenece todavía a la VLAN 99.

📸 **Captura 4:** ping PC-A → PC-B **fallando** (ahora están en VLAN 10 pero el enlace F0/1 sigue siendo de acceso en VLAN 1, así que el tráfico de VLAN 10 no cruza).

---

## 7. Sesión 2 — mantenimiento de VLAN

Todo esto se hace en **S1**. Es un ejercicio de manipulación; al final se revierte.

```cisco
enable
configure terminal
! (1) Asignar un rango completo a la VLAN 10
interface range fastEthernet 0/11 - 24
 switchport mode access
 switchport access vlan 10
 exit
end
show vlan brief
```

```cisco
configure terminal
! (2) Reasignar F0/11 y F0/21 a la VLAN 20
interface range fastEthernet 0/11 , fastEthernet 0/21
 switchport access vlan 20
 exit
end
show vlan brief
```

```cisco
configure terminal
! (3) Quitarle la asignación a F0/24 -> vuelve a la VLAN 1
interface fastEthernet 0/24
 no switchport access vlan
 exit
end
show vlan brief
```

```cisco
configure terminal
! (4) Asignar una VLAN inexistente -> el IOS la crea sola
interface fastEthernet 0/24
 switchport access vlan 30
 exit
end
show vlan brief
```

Salida esperada: `% Access VLAN does not exist. Creating vlan 30` y en la tabla aparece `30  VLAN0030  active  Fa0/24`.

```cisco
configure terminal
! (5) Borrar la VLAN 30 dejando el puerto huérfano
no vlan 30
end
show vlan brief
```

📸 **Captura 5:** F0/24 **desaparece de la tabla**. El puerto sigue asignado a una VLAN que ya no existe, queda inactivo y descarta todo el tráfico del host conectado.

```cisco
configure terminal
! (6) Rescatar el puerto
interface fastEthernet 0/24
 no switchport access vlan
 exit
end
show vlan brief
```

Ahora F0/24 vuelve a la VLAN 1.

> 🔑 **Moraleja del ejercicio** (y respuesta a la pregunta de la guía): antes de borrar una VLAN hay que reasignar sus puertos, porque si no quedan huérfanos en una VLAN inexistente y dejan de pasar tráfico en silencio.

---

## 8. Sesión 3 — enlace troncal 802.1Q

### 8.1 Primero con DTP (negociado)

**Solo en S1:**

```cisco
enable
configure terminal
interface fastEthernet 0/1
 switchport mode dynamic desirable
 exit
end
show interfaces trunk
```

S2 está en `dynamic auto` por defecto, así que acepta y el enlace se convierte en troncal solo.

Salida esperada:

```
Port    Mode        Encapsulation  Status     Native vlan
Fa0/1   desirable   802.1q         trunking   1
```

En S2 el mismo comando muestra `Mode: auto`.

📸 **Captura 6:** `show interfaces trunk` en S1 y en S2 (modos `desirable` y `auto`).

**Pruebas con el trunk arriba:**

| Ping | Resultado | Por qué |
|---|---|---|
| S1 → S2 | ✅ | La VLAN 99 ya cruza el troncal |
| PC-A → PC-B | ✅ | Ambos en VLAN 10, el troncal transporta la VLAN 10 |
| PC-A → PC-C | ❌ | VLAN y subredes distintas, no hay router |
| PC-B → PC-C | ❌ | Igual que el anterior |
| PC-A → S1 | ❌ | PC en `192.168.10.x`, switch en `192.168.1.x` |
| PC-B → S2 | ❌ | Subredes distintas |
| PC-C → S2 | ❌ | Subredes distintas |

📸 **Captura 7:** ping PC-A → PC-B **ahora funcionando** (compárala con la Captura 4: es la evidencia de que el troncal resolvió el problema).

### 8.2 Después, troncal manual

**En S1 y en S2 (los dos):**

```cisco
configure terminal
interface fastEthernet 0/1
 switchport mode trunk
 switchport trunk native vlan 99
 exit
end
show interfaces trunk
```

Ahora el modo cambia de `desirable` / `auto` a **`on`**.

📸 **Captura 8:** `show interfaces trunk` con `Mode: on`.

### 8.3 Guardar

```cisco
end
copy running-config startup-config
```

---

## 9. Sesión 3b — eliminar la base de datos de VLAN

Solo para demostrar cómo se deja un switch en fábrica. **Hazlo al final y sobre una copia**, o tendrás que reconfigurar todo.

```cisco
show flash
delete vlan.dat
show flash
```

Confirma con **Enter** las dos veces que pregunta.

Para dejar el switch **realmente** en configuración de fábrica hacen falta además:

```cisco
erase startup-config
reload
```

---

# D. EJERCICIOS EXTRA DE LA GUÍA

## 10. Spanning Tree — ilustraciones 1, 2 y 3

La guía pide armar las tres topologías, decir cómo trabaja STP y encontrar el **root switch**, los **root ports**, los **designated ports** y los **non-designated (bloqueados)**, a mano y en Packet Tracer.

### Cómo se resuelve, en orden

1. **Root bridge.** Gana el switch con el **Bridge ID** más bajo. El Bridge ID = *prioridad* (32768 + VLAN por defecto) + *MAC*. Si nadie cambió la prioridad, gana **la MAC más baja**.
2. **Root port** — uno por cada switch **no raíz**: el puerto con el **menor coste acumulado** hacia la raíz.
3. **Designated port** — uno por cada segmento: el puerto del lado con menor coste hacia la raíz. **Todos** los puertos de la raíz son designated.
4. **Non-designated / blocking** — el que sobra. Es el puerto que Packet Tracer pinta en **ámbar**.

### Costes que necesitas (STP clásico, 802.1D)

| Velocidad | Coste |
|---|---|
| 10 Mbps | 100 |
| **100 Mbps (FastEthernet)** | **19** |
| **1 Gbps (GigabitEthernet)** | **4** |

> 🔑 **Ese cuadro explica solo las tres ilustraciones.** En todas ellas los enlaces ámbar (bloqueados) son los **FastEthernet**: con coste 19 frente a 4 del gigabit, STP siempre prefiere rodear por gigabit antes que usar un enlace de 100 Mbps.

### Lo que se ve en cada ilustración

| | Topología | Puerto bloqueado en la imagen |
|---|---|---|
| **Ilustración 1** | Triángulo de 3 switches: dos enlaces Gig y uno Fa | El **`Fa0/1`** del switch izquierdo |
| **Ilustración 2** | Rombo de 4 switches, todos los enlaces Gig | El **`Gig0/2`** del switch inferior |
| **Ilustración 3** | Malla de 4 switches: anillo Gig + enlaces `Fa0/24` cruzados | El **`Gig0/1`** y el **`Fa0/24`** del switch izquierdo |

En la ilustración 2 todos los enlaces cuestan lo mismo (4), así que el desempate no lo decide el coste sino, en orden: **menor Bridge ID del vecino** y luego **menor número de puerto**. Por eso ahí el bloqueo cae en el switch más alejado de la raíz.

### Comandos para verificarlo en Packet Tracer

```cisco
show spanning-tree
show spanning-tree vlan 1
show spanning-tree root
show spanning-tree summary
show spanning-tree interface gigabitEthernet 0/1
```

`show spanning-tree` te dice en dos líneas si ese switch es la raíz (*"This bridge is the root"*) y lista cada puerto con su **Role** (`Root` / `Desg` / `Altn`) y su **Sts** (`FWD` / `BLK`).

> 📌 **Para el informe:** las MAC de tu archivo son distintas a las de cualquier otro, así que el root switch de tus tres topologías no tiene por qué coincidir con el de un compañero. Toma la captura de `show spanning-tree` de cada switch y compara el resultado con el que calculaste a mano — esa comparación es lo que se está evaluando.

📸 **Capturas 9, 10 y 11:** `show spanning-tree` de cada ilustración.

---

## 11. Ejercicio "Password UMB" — VTP con 5 switches

La guía inserta una segunda topología con **5 switches 2960-24TT en cadena** y **20 PCs**, rotulada *Password UMB*, y pide: configurar las VLAN, asignar los puertos, hacer el enrutamiento necesario y que *"el equipo del programador pueda ingresar a cualquiera de los dispositivos activos"*.

> ⚠️ **Interpretación.** El rótulo *Password UMB* y el switch marcado como **VTP SERVER** indican que el ejercicio es de **VTP con dominio y contraseña `UMB`**. Y como en la imagen **no hay ningún router**, el "enrutamiento" solo puede lograrse añadiendo uno (router-on-a-stick, que es el tema de la Práctica 3) — lo dejo abajo como bloque opcional y lo marco como supuesto en el informe.

### VLAN del ejercicio

| ID | Nombre | Red |
|---|---|---|
| 10 | `gerencia` | 172.16.10.0/24 |
| 20 | `presidencia` | 172.16.20.0/24 |
| 30 | `rh` | 172.16.30.0/24 |
| 40 | `porterias` | 172.16.40.0/24 |
| 50 | `secretarias` | 172.16.50.0/24 |
| 99 | `admin` | 172.16.99.0/24 *(añadida para la gestión)* |

### Reparto de PCs por switch (leído de los colores de la imagen)

| Switch | PCs | VLAN de cada uno |
|---|---|---|
| **S4** (arriba) | PC1, PC11, PC12 | 10, 20, 40 |
| **S3** | PC2, PC3, PC4, PC13, PC14 | 20, 40, 50, 30, 30 |
| **S2** | PC5, PC6, PC15, PC16 | 50, 10, 40, 40 |
| **S1** | PC7, PC8, PC17, PC18 | 40, 30, 40, 50 |
| **VTPSERVER** (abajo) | PC9, PC10, PC19, PC20 | 30, 30, 50, 40 |

Conéctalos en orden a `Fa0/1`, `Fa0/2`, `Fa0/3`… de cada switch. La cadena vertical entre switches usa los puertos gigabit: `Gig0/2` del de arriba ↔ `Gig0/1` del de abajo.

### VTPSERVER — el único donde se crean las VLAN

```cisco
enable
configure terminal
hostname VTPSERVER
vtp domain UMB
vtp password UMB
vtp mode server
vtp version 2
!
vlan 10
 name gerencia
 exit
vlan 20
 name presidencia
 exit
vlan 30
 name rh
 exit
vlan 40
 name porterias
 exit
vlan 50
 name secretarias
 exit
vlan 99
 name admin
 exit
!
interface gigabitEthernet 0/1
 switchport mode trunk
 exit
!
interface fastEthernet 0/1
 switchport mode access
 switchport access vlan 30
 exit
interface fastEthernet 0/2
 switchport mode access
 switchport access vlan 30
 exit
interface fastEthernet 0/3
 switchport mode access
 switchport access vlan 50
 exit
interface fastEthernet 0/4
 switchport mode access
 switchport access vlan 40
 exit
!
interface vlan 99
 ip address 172.16.99.5 255.255.255.0
 no shutdown
 exit
ip default-gateway 172.16.99.1
!
line vty 0 15
 password cisco
 login
 exit
enable secret class
!
end
copy running-config startup-config
```

### Los otros 4 switches — clientes VTP

Pega esta base en **S1, S2, S3 y S4**, cambiando el `hostname` y la IP de la VLAN 99. **Las VLAN no se crean aquí: llegan solas por VTP.**

```cisco
enable
configure terminal
hostname S1
vtp domain UMB
vtp password UMB
vtp mode client
!
interface range gigabitEthernet 0/1 - 2
 switchport mode trunk
 exit
!
interface vlan 99
 ip address 172.16.99.1 255.255.255.0
 no shutdown
 exit
ip default-gateway 172.16.99.1
!
line vty 0 15
 password cisco
 login
 exit
enable secret class
!
end
copy running-config startup-config
```

| Switch | IP de la VLAN 99 | Trunks |
|---|---|---|
| S1 | `172.16.99.1` | Gig0/1 y Gig0/2 |
| S2 | `172.16.99.2` | Gig0/1 y Gig0/2 |
| S3 | `172.16.99.3` | Gig0/1 y Gig0/2 |
| S4 | `172.16.99.4` | **solo Gig0/2** (es el de arriba) |
| VTPSERVER | `172.16.99.5` | **solo Gig0/1** (es el de abajo) |

### Puertos de acceso por switch

```cisco
! ===== S4 =====
interface fastEthernet 0/1
 switchport mode access
 switchport access vlan 10
 exit
interface fastEthernet 0/2
 switchport mode access
 switchport access vlan 20
 exit
interface fastEthernet 0/3
 switchport mode access
 switchport access vlan 40
 exit

! ===== S3 =====
interface fastEthernet 0/1
 switchport mode access
 switchport access vlan 20
 exit
interface fastEthernet 0/2
 switchport mode access
 switchport access vlan 40
 exit
interface fastEthernet 0/3
 switchport mode access
 switchport access vlan 50
 exit
interface range fastEthernet 0/4 - 5
 switchport mode access
 switchport access vlan 30
 exit

! ===== S2 =====
interface fastEthernet 0/1
 switchport mode access
 switchport access vlan 50
 exit
interface fastEthernet 0/2
 switchport mode access
 switchport access vlan 10
 exit
interface range fastEthernet 0/3 - 4
 switchport mode access
 switchport access vlan 40
 exit

! ===== S1 =====
interface fastEthernet 0/1
 switchport mode access
 switchport access vlan 40
 exit
interface fastEthernet 0/2
 switchport mode access
 switchport access vlan 30
 exit
interface fastEthernet 0/3
 switchport mode access
 switchport access vlan 40
 exit
interface fastEthernet 0/4
 switchport mode access
 switchport access vlan 50
 exit
```

### El "equipo del programador"

Elige un PC libre (o reutiliza uno), conéctalo a un puerto cualquiera y ponlo en la **VLAN 99**:

```cisco
interface fastEthernet 0/10
 switchport mode access
 switchport access vlan 99
 exit
```

IP del equipo: `172.16.99.100 / 255.255.255.0 / gw 172.16.99.1`.

Desde su `Command Prompt` debe poder entrar a los 5 switches:

```
telnet 172.16.99.1
telnet 172.16.99.2
telnet 172.16.99.3
telnet 172.16.99.4
telnet 172.16.99.5
```

Contraseña de vty: `cisco` · contraseña de enable: `class`.

### (Opcional) El router para el "enrutamiento"

Si tu docente sí espera comunicación entre las 5 VLAN, agrega un **1941** colgado del VTPSERVER por un troncal y configúralo así:

```cisco
enable
configure terminal
hostname R-UMB
interface gigabitEthernet 0/0
 no shutdown
 exit
interface gigabitEthernet 0/0.10
 encapsulation dot1Q 10
 ip address 172.16.10.1 255.255.255.0
 exit
interface gigabitEthernet 0/0.20
 encapsulation dot1Q 20
 ip address 172.16.20.1 255.255.255.0
 exit
interface gigabitEthernet 0/0.30
 encapsulation dot1Q 30
 ip address 172.16.30.1 255.255.255.0
 exit
interface gigabitEthernet 0/0.40
 encapsulation dot1Q 40
 ip address 172.16.40.1 255.255.255.0
 exit
interface gigabitEthernet 0/0.50
 encapsulation dot1Q 50
 ip address 172.16.50.1 255.255.255.0
 exit
interface gigabitEthernet 0/0.99
 encapsulation dot1Q 99
 ip address 172.16.99.1 255.255.255.0
 exit
end
copy running-config startup-config
```

Y en el VTPSERVER, el puerto hacia el router en modo troncal:

```cisco
interface gigabitEthernet 0/2
 switchport mode trunk
 exit
```

### Verificación del ejercicio

```cisco
show vtp status
show vlan brief
show interfaces trunk
```

En los clientes, `show vtp status` debe mostrar **`VTP Operating Mode: Client`** y el mismo **`Configuration Revision`** que el servidor. Si las VLAN no aparecen en los clientes: el dominio o la contraseña no coinciden, o el enlace entre switches no quedó en modo troncal.

---

# E. CIERRE

## 12. Capturas para la entrega

| # | Captura | Sección |
|---|---|---|
| 1 | Pings iniciales de PC-A (uno OK, uno fallido) | §6.2 |
| 2 | `show vlan` con las 3 VLAN creadas | §6.3 |
| 3 | `show ip interface brief` con VLAN 99 en `down/down` | §6.4 |
| 4 | Ping PC-A → PC-B **fallando** (antes del trunk) | §6.4 |
| 5 | `show vlan brief` con F0/24 huérfano tras borrar la VLAN 30 | §7 |
| 6 | `show interfaces trunk` con modos `desirable` / `auto` | §8.1 |
| 7 | Ping PC-A → PC-B **funcionando** (después del trunk) | §8.1 |
| 8 | `show interfaces trunk` con modo `on` | §8.2 |
| 9-11 | `show spanning-tree` de las ilustraciones 1, 2 y 3 | §10 |
| 12 | `show vtp status` del ejercicio Password UMB | §11 |

> 💡 Las capturas 4 y 7 son el par más importante de toda la práctica: son el antes y el después del enlace troncal.

## 13. Errores frecuentes

| Síntoma | Solución |
|---|---|
| La SVI de la VLAN 99 no levanta | Necesita al menos un puerto **activo** en la VLAN 99, o un troncal que la transporte |
| `% Invalid input` en `switchport access vlan` | Falta `switchport mode access` antes |
| PC-A no ve a PC-B con las VLAN creadas | El enlace F0/1 sigue en modo acceso: falta el troncal |
| `Native VLAN mismatch` en la consola | Pusiste `native vlan 99` en un solo extremo. Ponla en ambos o en ninguno |
| El troncal no se forma con DTP | Los dos extremos quedaron en `dynamic auto`. Al menos uno debe ser `desirable` o `trunk` |
| Un puerto desaparece de `show vlan brief` | Su VLAN fue borrada. `no switchport access vlan` lo devuelve a la VLAN 1 |
| Las VLAN no llegan a los clientes VTP | Dominio o contraseña distintos, o el enlace no es troncal |
| Todo se pierde al reiniciar | Falta `copy running-config startup-config` (y el `vlan.dat` también se guarda aparte) |

## 14. Checklist de entrega

- [ ] 2 switches 2960-24TT + 3 PCs, cableado según §3 (cruzado entre switches)
- [ ] IPs de los 3 PCs configuradas
- [ ] Configuración básica en ambos switches: hostname, `enable secret class`, vty y consola con `cisco`, `logging synchronous`, banner MOTD, `no ip domain-lookup`
- [ ] VLAN 10 `Student`, 20 `Faculty` y 99 `Management` creadas en **ambos** switches
- [ ] SVI de administración movida de la VLAN 1 a la **VLAN 99** en ambos
- [ ] Puertos asignados: S1 F0/6 → v10 · S2 F0/11 → v10 · S2 F0/18 → v20
- [ ] Enlace troncal 802.1Q en F0/1, primero por DTP y después forzado con `switchport mode trunk`
- [ ] Puertos sin usar apagados en ambos switches
- [ ] `copy running-config startup-config` en los 2 switches
- [ ] Las 12 capturas de §12
- [ ] Ejercicio STP (3 ilustraciones) con el cálculo a mano y la comparación en Packet Tracer
- [ ] Ejercicio Password UMB con VTP funcionando
- [ ] `entregable.md` con las preguntas respondidas → pasado a PDF con normas APA
- [ ] Archivo `P2-VLAN-Trunking.pkt` guardado en esta carpeta
