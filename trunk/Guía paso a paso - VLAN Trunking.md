---
tipo: laboratorio
practica: VLAN Trunking (Parte 1 + Parte 2)
fuente: "VLAN Trunking.docx"
software: Cisco Packet Tracer 7.x / 8.x
creado: 2026-08-14
tags: [redes, vlan, trunk, packet-tracer, cisco, dot1q, http, consola]
---

# Laboratorio: Creación de VLANs con Trunking

> **Objetivo:** configurar una topología con redes virtuales (VLANs) que se comunican entre sí a través de un enlace **trunk** hacia un router (*router-on-a-stick*, encapsulación **802.1Q**).

Ver contexto general en [[00 - Contexto del Vault]].

## Contenido

| | Alcance | Secciones |
|---|---|---|
| **Parte 1** | VLANs, trunk 802.1Q y enrutamiento inter-VLAN con router-on-a-stick | §0 – §10 |
| **Parte 2** | Dos servidores web en un segundo segmento + acceso al router por cable de consola | §11 – §22 |

Explicación conceptual interactiva: **`Conceptos - VLAN Trunking.html`** (en esta misma carpeta).

## Novedades y correcciones del docente

| Fecha | Novedad | Estado en esta guía |
|---|---|---|
| 14/08/2026 | La VLAN de **VENTAS es la 20**, no la 10. El `.docx` repetía `vlan 10` por error de tipeo. | ✅ Aplicado (§2.4, §2.5, §4, §9) |
| 14/08/2026 | **Segunda parte en clase:** agregar 2 servidores web colgados de una nueva interfaz del router, y una laptop conectada por consola para configurarlo. | ✅ Desarrollado en la **Parte 2** (§11 – §22) |
| 14/08/2026 | **Hallazgo al armarlo:** el puerto `Fa4/1` del `Switch-PT` es de **fibra** y no acepta cable de cobre. Se usa **`Fa2/1`**. | ✅ Corregido en §11, §13.1, §13.2 y §19 |

> ℹ️ **Direccionamiento de la Parte 2.** La red del segmento de servidores no se alcanzaba a ver en la foto de
> clase; se usa **192.168.1.0/24**. Si el profesor indica otra, solo hay que cambiar las IPs de §11, §15 y §17.

---

## 0. Resumen de lo que vas a construir

```
                        ┌──────────────┐
                        │  Router0     │  2620XM
                        │  fa0/0       │  ← subinterfaces .1 .2 .3
                        └──────┬───────┘
                               │  TRUNK (802.1Q)
                        ┌──────┴───────┐
                        │   Sw1        │  2950T-24   fa0/1 = trunk
                        │              │  IP mgmt 1.0.0.2
                        └──┬────────┬──┘
                   fa0/2 ──┘        └── fa0/4
                  (VLAN 10)          (VLAN 20)
                     │                   │
                  ┌──┴──┐             ┌──┴──┐
                  │ PC0 │             │ PC1 │
                  └─────┘             └─────┘
                 10.0.0.2            20.0.0.2
```

### Tabla de direccionamiento

| Red / VLAN | Nombre | Dirección de red | Máscara | Gateway (subinterfaz router) | Host |
|---|---|---|---|---|---|
| VLAN 1 (nativa / gestión) | default | 1.0.0.0 | 255.0.0.0 | 1.0.0.1 (fa0/0.1) | Sw1 → 1.0.0.2 |
| VLAN 10 | INGENIERIA | 10.0.0.0 | 255.0.0.0 | 10.0.0.1 (fa0/0.2) | PC0 → 10.0.0.2 |
| VLAN 20 | VENTAS | 20.0.0.0 | 255.0.0.0 | 20.0.0.1 (fa0/0.3) | PC1 → 20.0.0.2 |

> Máscara `255.0.0.0` = `/8`. Es la que pide la guía original (redes clase A completas, muy poco eficiente pero funcional para el laboratorio).

### Componentes

| Componente | Cantidad | Modelo | Notas |
|---|---|---|---|
| Computadores | 2 | PC-PT | PC0 y PC1 |
| Switch | 1 | **2950T-24** | 24 puertos FastEthernet |
| Router | 1 | **2620XM** | 1 puerto FastEthernet0/0 |
| Cable | 3 | Directo (uno a uno) | Copper Straight-Through |

---

## 1. Armar la topología en Packet Tracer

### 1.1 Dónde encontrar cada objeto

Todo el catálogo está en la **esquina inferior izquierda** de Packet Tracer. Funciona en dos niveles:

- **Fila de arriba** = categoría principal (*Device-Type Selection Box* / «Cuadro de selección de tipo de dispositivo»).
- **Panel de la derecha** = modelos concretos (*Device-Specific Selection Box* / «Cuadro de selección de dispositivo específico»).

| Qué necesitas | Categoría (inglés) | Categoría (español) | Subcategoría | Modelo a arrastrar |
|---|---|---|---|---|
| Router | **Network Devices → Routers** | Dispositivos de red → Enrutadores | — | **2620XM** |
| Switch | **Network Devices → Switches** | Dispositivos de red → Conmutadores | — | **2950T-24** |
| PCs | **End Devices → End Devices** | Dispositivos finales → Dispositivos finales | — | **PC** (icono `PC-PT`) |
| Cables | **Connections → Connections** | Conexiones → Conexiones | — | **Copper Straight-Through** / «Cobre directo» (icono de línea negra continua) |

> 💡 Los iconos de cable, de izquierda a derecha: *Automatically Choose Connection Type* (rayo dorado ⚡), *Console* (celeste), **Copper Straight-Through** (negro continuo), *Copper Cross-Over* (negro punteado), *Fiber*, *Phone*, *Coaxial*, *Serial DCE*, *Serial DTE*.
>
> ⚡ El rayo dorado («elegir automáticamente») sirve para salir del paso, pero **para la entrega usa el cable directo explícito**: el profesor suele revisar que el tipo de cable sea el correcto.

### 1.2 Colocar los dispositivos

1. Arrastra **1 × 2620XM** al lienzo → se llamará `Router0`.
2. Arrastra **1 × 2950T-24** → se llamará `Switch0` (después lo renombramos a `sw1` por comando).
3. Arrastra **2 × PC** → `PC0` y `PC1`.

Colócalos como en el diagrama: router arriba, switch al centro, PC0 a la izquierda y PC1 a la derecha.

### 1.3 Cablear (3 cables directos)

Selecciona **Copper Straight-Through**, haz clic en el dispositivo origen, elige el puerto, y clic en el destino:

| # | Desde | Puerto | Hacia | Puerto | Rol |
|---|---|---|---|---|---|
| 1 | Router0 | `FastEthernet0/0` | Sw1 | `FastEthernet0/1` | **Trunk** |
| 2 | PC0 | `FastEthernet0` | Sw1 | `FastEthernet0/2` | Acceso VLAN 10 |
| 3 | PC1 | `FastEthernet0` | Sw1 | `FastEthernet0/4` | Acceso VLAN 20 |

> ⚠️ **Respeta los números de puerto** (fa0/1, fa0/2, fa0/4). Los comandos de más abajo dependen de ellos.
>
> ⚠️ Al inicio los enlaces se ven con **puntos rojos** (router apagado o puerto en `shutdown`) o **ámbar** (STP convergiendo). Se ponen **verdes** solos tras aplicar la configuración y esperar ~30 segundos.

### 1.4 Entrar a la consola de un dispositivo

Clic en el dispositivo → pestaña **CLI** (*Command Line Interface* / «Interfaz de línea de comandos»).
En los PCs no hay CLI: se usa la pestaña **Desktop** («Escritorio»).

---

## 2. Configuración del Switch (Sw1)

> Notación: `!` son comentarios, no se escriben. Puedes pegar bloque por bloque.

### 2.1 Nombre del equipo

```cisco
Switch>enable
Switch#configure terminal
Switch(config)#hostname sw1
sw1(config)#
```

> ⚠️ **Corrección de la guía original.** El `.docx` escribe `Switch(Config)# configure terminal`. Está mal: `configure terminal` se ejecuta desde el modo **privilegiado** (`Switch#`), no desde el de configuración global. La secuencia correcta es la de arriba.
>
> **Los tres modos que vas a usar todo el tiempo:**
> | Prompt | Modo | Cómo entrar | Cómo salir |
> |---|---|---|---|
> | `sw1>` | Usuario (*user EXEC*) | inicio | `exit` |
> | `sw1#` | Privilegiado (*privileged EXEC*) | `enable` | `disable` |
> | `sw1(config)#` | Configuración global | `configure terminal` | `exit` o `end` |
> | `sw1(config-if)#` | Configuración de interfaz | `interface fa0/x` | `exit` |

### 2.2 Acceso remoto (Telnet) y contraseña de enable

```cisco
sw1(config)#line vty 0 4
sw1(config-line)#password cisco
sw1(config-line)#login
sw1(config-line)#exit
sw1(config)#enable password cisco
```

- `line vty 0 4` → habilita 5 sesiones remotas simultáneas (0,1,2,3,4).
- `login` → obliga a pedir la contraseña. **Sin este comando el telnet falla** con *"password required, but none set"*.
- `enable password cisco` → contraseña para pasar a modo privilegiado por telnet.

### 2.3 IP de gestión y gateway del switch

```cisco
sw1(config)#interface vlan 1
sw1(config-if)#ip address 1.0.0.2 255.0.0.0
sw1(config-if)#no shutdown
sw1(config-if)#exit
sw1(config)#ip default-gateway 1.0.0.1
sw1(config)#exit
sw1#
```

> El switch es capa 2: la IP no se pone en un puerto físico sino en la **interfaz virtual VLAN 1**, y solo sirve para administrarlo (telnet/ping).

### 2.4 Crear las VLANs

```cisco
sw1#configure terminal
sw1(config)#vlan 10
sw1(config-vlan)#name INGENIERIA
sw1(config-vlan)#exit
sw1(config)#vlan 20
sw1(config-vlan)#name VENTAS
sw1(config-vlan)#exit
```

> ⚠️ **Dos correcciones importantes a la guía original:**
>
> 1. ✅ **Confirmado por el profesor (14/08/2026).** El `.docx` dice `vlan 10 name INGENIERIA` y luego **`vlan 10 name VENTAS`** — repite el 10. El docente aclaró que fue un error suyo y que **VENTAS es la VLAN 20**. Por eso el comando correcto es **`vlan 20` → `name VENTAS`**; con el texto original la VLAN 20 nunca se crea y PC1 queda aislado. La guía ya está escrita con la versión corregida.
> 2. El `.docx` usa el modo `vlan database`. Ese modo está **obsoleto y no funciona en Packet Tracer** (responde `% Unrecognized command` o avisa de deprecación). Usa el método de arriba, desde `configure terminal`.

### 2.5 Puertos de acceso

**VLAN 10 por el puerto fa0/2 (PC0):**

```cisco
sw1(config)#interface fastEthernet 0/2
sw1(config-if)#switchport mode access
sw1(config-if)#switchport access vlan 10
sw1(config-if)#exit
```

**VLAN 20 por el puerto fa0/4 (PC1):**

```cisco
sw1(config)#interface fastEthernet 0/4
sw1(config-if)#switchport mode access
sw1(config-if)#switchport access vlan 20
sw1(config-if)#exit
```

- `switchport mode access` → el puerto pertenece a **una sola** VLAN (para equipos finales).
- `switchport access vlan N` → cuál.

### 2.6 Puerto trunk hacia el router

```cisco
sw1(config)#interface fastEthernet 0/1
sw1(config-if)#switchport mode trunk
sw1(config-if)#exit
sw1(config)#exit
sw1#copy running-config startup-config
```

- `switchport mode trunk` → el puerto transporta **todas** las VLANs, etiquetando cada trama con su VLAN ID (802.1Q).
- El 2950T-24 solo soporta 802.1Q, así que **no** hace falta `switchport trunk encapsulation dot1q` (si lo escribes, dará error — es normal).
- `copy running-config startup-config` (o `write memory`) → guarda para que sobreviva a un reinicio.

---

## 3. Configuración del Router (Router0)

Aquí ocurre el **enrutamiento inter-VLAN**: una sola interfaz física dividida en **subinterfaces**, una por VLAN. Esto es lo que se llama *router-on-a-stick* («router en un palito»).

### 3.1 Levantar la interfaz física

```cisco
Router>enable
Router#configure terminal
Router(config)#hostname Router0
Router0(config)#interface fastEthernet 0/0
Router0(config-if)#no ip address
Router0(config-if)#no shutdown
Router0(config-if)#exit
```

> ⚠️ La interfaz física **no lleva IP** (las llevan las subinterfaces), pero **sí debe estar `no shutdown`**. En un router Cisco las interfaces vienen apagadas de fábrica: si olvidas este comando, nada funciona y el enlace queda en rojo.
>
> ⚠️ La guía original escribe `Router0(config)# conf term` justo después de `enable`. Es un error de transcripción: `configure terminal` se ejecuta desde `Router#`.

### 3.2 Subinterfaz para VLAN 1 (nativa)

```cisco
Router0(config)#interface fastEthernet 0/0.1
Router0(config-subif)#encapsulation dot1Q 1 native
Router0(config-subif)#ip address 1.0.0.1 255.0.0.0
Router0(config-subif)#exit
```

### 3.3 Subinterfaz para VLAN 10

```cisco
Router0(config)#interface fastEthernet 0/0.2
Router0(config-subif)#encapsulation dot1Q 10
Router0(config-subif)#ip address 10.0.0.1 255.0.0.0
Router0(config-subif)#exit
```

### 3.4 Subinterfaz para VLAN 20

```cisco
Router0(config)#interface fastEthernet 0/0.3
Router0(config-subif)#encapsulation dot1Q 20
Router0(config-subif)#ip address 20.0.0.1 255.0.0.0
Router0(config-subif)#exit
Router0(config)#exit
Router0#copy running-config startup-config
```

**Cómo leerlo:**

- `fa0/0.1`, `.2`, `.3` → el número después del punto es solo una **etiqueta local**; no tiene que coincidir con el número de VLAN (aunque muchos lo hacen coincidir por claridad: `fa0/0.10` para VLAN 10).
- `encapsulation dot1Q <vlan-id>` → **esto sí** define a qué VLAN pertenece la subinterfaz. Es el número que importa.
- `native` en la VLAN 1 → indica que esa VLAN viaja **sin etiqueta** por el trunk, igual que en el switch.
- La IP de cada subinterfaz es el **default gateway** de esa VLAN.

---

## 4. Configuración de los PCs

No hay CLI. Se hace por interfaz gráfica:

**Clic en el PC → pestaña `Desktop` («Escritorio») → `IP Configuration` («Configuración IP») → seleccionar `Static` («Estática»).**

| Equipo | IP Address / Dirección IP | Subnet Mask / Máscara | Default Gateway / Puerta de enlace |
|---|---|---|---|
| **PC0** (VLAN 10) | `10.0.0.2` | `255.0.0.0` | `10.0.0.1` |
| **PC1** (VLAN 20) | `20.0.0.2` | `255.0.0.0` | `20.0.0.1` |

> ⚠️ La tabla del `.docx` deja la columna de **Gateway vacía**. Sin gateway, el ping entre VLANs **falla siempre** — es el error nº 1 de este laboratorio. Rellénala con los valores de arriba.
>
> El campo *DNS Server* se deja en blanco.

---

## 5. Pruebas a realizar

### 5.1 Ping de una VLAN a otra

Desde **PC0 → Desktop → Command Prompt** («Símbolo del sistema»):

```
ping 10.0.0.1     ← su propio gateway (debe responder primero)
ping 20.0.0.2     ← PC1, en la otra VLAN
```

Desde **PC1**:

```
ping 20.0.0.1
ping 10.0.0.2
```

> El **primer paquete puede fallar** (`Request timed out`) por el ARP inicial. Es normal; repite el ping. Si fallan los 4, revisa la sección de errores.

### 5.2 Telnet al switch y ver VLANs

Desde cualquier PC → **Desktop → Command Prompt**:

```
telnet 1.0.0.2
```

Pide contraseña → `cisco`

```cisco
sw1>enable
Password: cisco
sw1#show vlan
sw1#show vlan brief
```

> ⚠️ Para que el telnet funcione, el PC debe poder llegar a `1.0.0.2`. Como los PCs están en 10.0.0.0 y 20.0.0.0, el tráfico pasa por el router, que sí tiene la subinterfaz `1.0.0.1`. Por eso el `ip default-gateway 1.0.0.1` del switch es obligatorio: sin él, el switch recibe el telnet pero **no sabe devolver la respuesta**.

### 5.3 Verificación completa

**En el switch:**

```cisco
sw1#show vlan brief              ! VLANs 10 y 20 con fa0/2 y fa0/4 asignados
sw1#show interfaces trunk        ! fa0/1 en modo trunk, dot1q, native vlan 1
sw1#show ip interface brief      ! Vlan1 con 1.0.0.2, up/up
sw1#show running-config
```

**En el router:**

```cisco
Router0#show ip interface brief  ! fa0/0 y las 3 subinterfaces en up/up
Router0#show ip route            ! 3 redes conectadas: 1.0.0.0, 10.0.0.0, 20.0.0.0
Router0#show vlans               ! subinterfaces y su VLAN dot1q
Router0#show running-config
```

Salida esperada de `show vlan brief`:

```
VLAN Name                             Status    Ports
---- -------------------------------- --------- ---------------
1    default                          active    Fa0/3, Fa0/5, ... Fa0/24
10   INGENIERIA                       active    Fa0/2
20   VENTAS                           active    Fa0/4
```

Salida esperada de `show ip route` en el router:

```
C    1.0.0.0/8 is directly connected, FastEthernet0/0.1
C    10.0.0.0/8 is directly connected, FastEthernet0/0.2
C    20.0.0.0/8 is directly connected, FastEthernet0/0.3
```

---

## 6. Errores frecuentes (checklist si no funciona)

| Síntoma | Causa probable | Solución |
|---|---|---|
| El ping entre PCs falla siempre | PCs sin **default gateway** | Ponerlos: 10.0.0.1 y 20.0.0.1 |
| Enlace router–switch en **rojo** | Falta `no shutdown` en fa0/0 | `interface fa0/0` → `no shutdown` |
| Enlace en **ámbar** que no cambia | STP convergiendo | Esperar 30 s |
| `show vlan brief` no muestra la VLAN 20 | Se creó dos veces la VLAN 10 (errata del `.docx`, corregida por el profesor) | `vlan 20` → `name VENTAS` |
| PC llega a su gateway pero no a la otra VLAN | El trunk no está activo | `show interfaces trunk` en fa0/1 |
| PC0 y PC1 se ven pero no debían… | Ambos puertos quedaron en la misma VLAN | Revisar `switchport access vlan` en fa0/2 y fa0/4 |
| Telnet: *"password required, but none set"* | Falta `login` o `password` en `line vty` | Reaplicar 2.2 |
| Telnet conecta pero `enable` rechaza | Falta `enable password` | `enable password cisco` |
| `vlan database` da error | Modo obsoleto | Crear VLANs desde `configure terminal` |
| Todo se pierde al reiniciar | No se guardó | `copy running-config startup-config` |

---

## 7. Respuestas a las preguntas de la guía

### ¿Qué es trunking?

**Trunking** («enlace troncal») es la técnica que permite que **un solo enlace físico transporte el tráfico de varias VLANs al mismo tiempo**.

Normalmente un puerto de switch pertenece a una única VLAN (puerto de *acceso*). Si hubiera que conectar dos switches con 20 VLANs, harían falta 20 cables. El trunk resuelve eso: por un único cable viaja el tráfico de todas las VLANs, y para no mezclarlas el switch **etiqueta** cada trama con un identificador de VLAN (*tag*) antes de enviarla; el equipo del otro extremo lee la etiqueta y la devuelve a su VLAN correspondiente.

- El estándar de etiquetado es **IEEE 802.1Q** (`dot1Q` en los comandos), que inserta 4 bytes en la cabecera Ethernet con el VLAN ID (12 bits → 4094 VLANs posibles).
- La **VLAN nativa** (por defecto la 1) es la excepción: viaja **sin etiqueta** por el trunk, por compatibilidad con equipos antiguos.
- Los trunks se usan entre **switch–switch** o **switch–router**, no hacia PCs.
- En este laboratorio el trunk `fa0/1 ↔ fa0/0` es lo que permite que **un router con una sola tarjeta de red** enrute entre las tres VLANs (*router-on-a-stick*).

### ¿Para qué se hacen las VLANs?

Una **VLAN** (*Virtual LAN*, red de área local virtual) divide un switch físico en **varias redes lógicas independientes**, sin necesidad de comprar más switches ni recablear.

Razones principales:

1. **Segmentación lógica sin hardware extra** — Ingeniería y Ventas comparten el mismo switch pero son redes separadas; el criterio es la función o el departamento, no la ubicación física.
2. **Seguridad** — dispositivos de distintas VLANs **no se comunican directamente**; el tráfico está obligado a pasar por un router o switch de capa 3, donde se pueden aplicar ACLs y filtros.
3. **Control del tráfico broadcast** — cada VLAN es un **dominio de broadcast** propio. Un ARP o un broadcast de la VLAN 10 no molesta a la VLAN 20, lo que reduce el ruido y mejora el rendimiento.
4. **Flexibilidad y administración** — mover a un empleado de departamento es cambiar un comando (`switchport access vlan`), no tender cable nuevo.
5. **Independencia de la ubicación física** — usuarios en pisos o edificios distintos pueden estar en la misma VLAN.
6. **Ahorro de costos** — se aprovecha el mismo equipo para varias redes.

**Contrapartida:** al aislar las VLANs, para que se hablen entre sí hace falta enrutamiento (*inter-VLAN routing*) — exactamente lo que resuelve el router con subinterfaces de esta práctica.

---

## 8. Glosario

**Encapsulación (*Encapsulation*).** Proceso de envolver un paquete de datos dentro de otro protocolo para transportarlo. En este laboratorio, `encapsulation dot1Q` significa que las tramas Ethernet se envuelven con la etiqueta 802.1Q que identifica su VLAN. En un sentido más amplio (p. ej. GRE), es un protocolo que encapsula distintos tipos de tráfico dentro de túneles IP, creando una red punto a punto entre dos máquinas — uso principal: túneles VPN.
*Fuente: http://es.wikipedia.org/wiki/Generic_Routing_Encapsulation*

**Trunk (Troncal).** En el contexto de las VLANs, designa una conexión de red que transporta múltiples VLANs identificadas por etiquetas (*tags*) insertadas en sus paquetes. Los trunks operan entre *tagged ports* («puertos etiquetados») de dispositivos con soporte de VLANs, por lo que suelen ser enlaces conmutador-a-conmutador o conmutador-a-enrutador, más que enlaces a nodos finales. (Cisco también usa «trunk» para lo que denomina «canales»; véase *agregado de enlaces*.) Un enrutador —o conmutador de nivel 3— funciona como columna vertebral del tráfico transmitido entre distintas VLANs.
*Fuente: http://es.wikipedia.org/wiki/VLAN*

**VTP (*VLAN Trunking Protocol*).** Protocolo propietario de Cisco que permite definir dominios de VLAN y propagar automáticamente su configuración entre switches, facilitando la administración. También permite *pruning* («podar»): dirigir el tráfico de una VLAN solo a los switches que tienen puertos en esa VLAN.

**VLAN nativa (*Native VLAN*).** La única VLAN cuyas tramas viajan **sin etiquetar** por un enlace trunk. Por defecto es la VLAN 1. Debe coincidir en ambos extremos del trunk.

**Router-on-a-stick.** Diseño en el que un router enruta entre varias VLANs usando **una sola interfaz física** dividida en subinterfaces lógicas, conectada al switch mediante un trunk. Es exactamente la topología de esta práctica.

**Puerto de acceso (*Access port*).** Puerto de switch que pertenece a una única VLAN y transporta tráfico **sin etiquetar**. Es el tipo de puerto al que se conectan PCs, impresoras y servidores.

---

## 9. Configuración completa (para copiar de una vez)

<details>
<summary><b>Switch — Sw1</b></summary>

```cisco
enable
configure terminal
hostname sw1
!
line vty 0 4
 password cisco
 login
 exit
enable password cisco
!
interface vlan 1
 ip address 1.0.0.2 255.0.0.0
 no shutdown
 exit
ip default-gateway 1.0.0.1
!
vlan 10
 name INGENIERIA
 exit
vlan 20
 name VENTAS
 exit
!
interface fastEthernet 0/2
 switchport mode access
 switchport access vlan 10
 exit
interface fastEthernet 0/4
 switchport mode access
 switchport access vlan 20
 exit
!
interface fastEthernet 0/1
 switchport mode trunk
 exit
!
end
copy running-config startup-config
```
</details>

<details>
<summary><b>Router — Router0</b></summary>

```cisco
enable
configure terminal
hostname Router0
!
interface fastEthernet 0/0
 no ip address
 no shutdown
 exit
!
interface fastEthernet 0/0.1
 encapsulation dot1Q 1 native
 ip address 1.0.0.1 255.0.0.0
 exit
interface fastEthernet 0/0.2
 encapsulation dot1Q 10
 ip address 10.0.0.1 255.0.0.0
 exit
interface fastEthernet 0/0.3
 encapsulation dot1Q 20
 ip address 20.0.0.1 255.0.0.0
 exit
!
end
copy running-config startup-config
```
</details>

---

## 10. Checklist de entrega — Parte 1

- [ ] Topología armada con 2620XM, 2950T-24 y 2 PC-PT
- [ ] 3 cables directos en fa0/1, fa0/2, fa0/4
- [ ] Switch renombrado a `sw1`, con IP 1.0.0.2 y gateway 1.0.0.1
- [ ] VLAN 10 = INGENIERIA, VLAN 20 = VENTAS (¡verificar con `show vlan brief`!)
- [ ] fa0/2 en VLAN 10, fa0/4 en VLAN 20, fa0/1 en trunk
- [ ] Router con 3 subinterfaces y `no shutdown` en fa0/0
- [ ] PCs con IP **y gateway**
- [ ] Ping PC0 → PC1 exitoso (captura de pantalla)
- [ ] Telnet a 1.0.0.2 + `show vlan` exitoso (captura de pantalla)
- [ ] Preguntas respondidas (sección 7)
- [ ] Archivo guardado como `VLAN-Trunking.pkt`

---
---

# PARTE 2 — Servidores web y acceso por consola

> **Continuidad.** Todo lo de la Parte 1 se conserva tal cual: el trunk, las VLANs 10 y 20, los dos PCs y las
> tres subinterfaces del router. Aquí **solo se agrega**. Si la Parte 1 ya te funciona, no toques nada de ella.

## 11. Qué se agrega y por qué

En la Parte 1 lograste que dos VLANs se hablaran entre sí. Ahora vas a darles **algo a lo que conectarse**: dos
servidores web en su propio segmento de red, colgados de una **segunda interfaz física** del router. Y como
extra, vas a configurar el router de la forma en que se hace en la vida real: por **cable de consola** desde un
portátil.

### Topología completa (Parte 1 + Parte 2)

```
              Server1                            Server0
           192.168.1.3                        192.168.1.2
                 │ Fa0                              │ Fa0
                 └──────────┐              ┌────────┘
                      Fa1/1 │              │ Fa0/1
                       ┌────┴──────────────┴────┐
                       │  Switch1 · Switch-PT   │   ← LAN normal, SIN VLANs
                       └───────────┬────────────┘
                                   │ Fa2/1
                                   │
                                   │ Fa1/0 → 192.168.1.1   (módulo NM-1FE-TX)
  Laptop0 ····consola····┌─────────┴──────────┐
                         │  Router1 · 2620XM  │
                         └─────────┬──────────┘
                                   │ Fa0/0 → subinterfaces .1 .2 .3
                                   │ TRUNK 802.1Q
                                   │ Fa0/1
                       ┌───────────┴────────────┐
                       │    sw1 · 2950T-24      │
                       └──┬──────────────────┬──┘
                    Fa0/2 │                  │ Fa0/4
                        PC0                 PC1
                     10.0.0.2            20.0.0.2
                     VLAN 10             VLAN 20
```

### Direccionamiento nuevo

| Equipo | Interfaz | Dirección IP | Máscara | Gateway |
|---|---|---|---|---|
| **Router1** | Fa1/0 | 192.168.1.1 | 255.255.255.0 | — |
| **Server0** | Fa0 | 192.168.1.2 | 255.255.255.0 | 192.168.1.1 |
| **Server1** | Fa0 | 192.168.1.3 | 255.255.255.0 | 192.168.1.1 |
| **Laptop0** | consola | — (no lleva IP) | — | — |

> ⚠️ **Ojo con la máscara.** Este segmento usa `/24` (255.255.255.0), **no** el `/8` de la Parte 1. Es una red
> distinta y no pasa nada por mezclar máscaras: cada red tiene la suya.

### Componentes nuevos

| Componente | Cantidad | Modelo | Dónde va |
|---|---|---|---|
| Servidor | 2 | Server-PT | Server0 y Server1 |
| Switch de servidores | 1 | Switch-PT | Switch1 |
| Portátil | 1 | Laptop-PT | Laptop0, solo para consola |
| **Módulo de red** | 1 | **NM-1FE-TX** | Dentro del Router1 |
| Cable directo | 3 | Copper Straight-Through | Servidores↔Switch1, Switch1↔Router1 |
| **Cable de consola** | 1 | **Console** (celeste) | Laptop0 ↔ Router1 |

---

## 12. PRIMERO: instalar el módulo NM-1FE-TX en el router

> 🛑 **Este es el paso que atasca a todo el mundo.** El **2620XM trae una sola tarjeta de red**
> (`FastEthernet0/0`), que ya está ocupada por el trunk de la Parte 1. La interfaz `Fa1/0` que ves en la
> topología del profesor **no existe de fábrica: hay que instalarla**. Si intentas escribir
> `interface fa1/0` sin hacer esto, el router responderá `% Invalid input detected`.

### Pasos

1. Clic en **Router1** → pestaña **`Physical`** («Físico»), la primera de todas.
2. Localiza el **interruptor de encendido** (*power switch*): es el botoncito redondo en el chasis, a la
   derecha. **Apaga el router** haciendo clic en él. El dibujo debe quedar sin luces.
   > ⚠️ Packet Tracer **no deja instalar módulos con el equipo encendido**, igual que un servidor real. Si
   > arrastras el módulo sin apagar, no pasa nada y creerás que el simulador está fallando.
3. En la lista **`MODULES`** («Módulos») de la izquierda, busca **`NM-1FE-TX`**.
   - `NM` = *Network Module* (módulo de red) · `1FE` = 1 FastEthernet · `TX` = cobre.
4. **Arrástralo** desde la lista hasta la **ranura vacía grande** del chasis (la del centro-derecha).
5. **Vuelve a encender** el router con el mismo interruptor.
6. Verifica: pestaña **CLI** → `show ip interface brief`. Debe aparecer una línea nueva `FastEthernet1/0`.

> 💡 **¿Por qué se llama `Fa1/0`?** Porque el módulo va en la **ranura 1** y es su **puerto 0**. La interfaz
> integrada de fábrica es la ranura 0 → `Fa0/0`.
>
> 🔄 **Alternativa:** si tu versión de Packet Tracer no lista `NM-1FE-TX`, usa **`NM-2FE2W`**, que da dos
> puertos (`Fa1/0` y `Fa1/1`). Usa el `Fa1/0` y deja el otro libre.

---

## 13. Agregar los objetos nuevos en Packet Tracer

### 13.1 Dónde encontrar cada uno

| Qué necesitas | Categoría (inglés) | Categoría (español) | Modelo a arrastrar |
|---|---|---|---|
| Servidores | **End Devices → End Devices** | Dispositivos finales | **Server** (icono `Server-PT`, torre azul) |
| Switch de servidores | **Network Devices → Switches** | Dispositivos de red → Conmutadores | **Switch-PT** (el genérico, primero de la lista) |
| Portátil | **End Devices → End Devices** | Dispositivos finales | **Laptop** (icono `Laptop-PT`) |
| Cable de consola | **Connections → Connections** | Conexiones | **Console** («Consola») — el **celeste claro**, segundo icono |
| Cable directo | **Connections → Connections** | Conexiones | **Copper Straight-Through** — el negro continuo |

> 💡 **Switch-PT vs 2950T-24.** El profesor usó el `Switch-PT` genérico. Sirve igual un **2950T-24**; solo
> cambian los nombres de puerto. El `Switch-PT` numera sus puertos como **`Fa<ranura>/<puerto>`** porque es
> modular — por eso aparecen números salteados como `Fa0/1`, `Fa1/1`, `Fa2/1`.

> 🛑 **¡OJO! No todas las ranuras del Switch-PT son de cobre.** Esto es lo que más desconcierta de este equipo.
>
> El `Switch-PT` viene con **módulos distintos según la ranura**, y no todos aceptan el mismo cable:
>
> | Módulo | Tipo de puerto | Cable que acepta |
> |---|---|---|
> | `PT-SWITCH-NM-1CFE` | **C**opper FastEthernet (cobre) | Copper Straight-Through / Cross-Over |
> | `PT-SWITCH-NM-1FFE` | **F**iber FastEthernet (fibra) | **Fiber** (naranja) — el cable de cobre **no conecta** |
>
> La letra del medio lo dice todo: **C** = cobre, **F** = fibra. Si intentas soltar un cable de cobre sobre un
> puerto de fibra, Packet Tracer simplemente **no te deja hacer clic**: el puerto aparece deshabilitado en la
> lista, o el cable no engancha. No es un fallo del programa ni tuyo.
>
> **Cómo saber qué tiene cada ranura:** clic en Switch1 → pestaña **`Physical`**. Ahí ves el chasis con los
> módulos instalados y el nombre de cada uno. También puedes soltar el cable sobre el switch y mirar la lista:
> los puertos que puedes usar con ese cable son los que aparecen **seleccionables**.
>
> **Solución:** usa cualquier ranura con módulo de **cobre** (`1CFE`). En esta guía se usa **`Fa2/1`**.
> Si en tu versión de Packet Tracer las ranuras de cobre son otras, **usa las que tengas** — el número de
> puerto es indiferente, porque este switch no se configura (§16). Lo único que importa es que los tres cables
> lleguen a puertos de cobre.
>
> 🔄 **Alternativa a prueba de balas:** si te cansa adivinar ranuras, cambia el `Switch-PT` por un
> **2950T-24**, donde los 24 puertos son de cobre y se llaman `Fa0/1` … `Fa0/24`. La práctica funciona
> exactamente igual.

### 13.2 Cablear (3 directos + 1 de consola)

| # | Desde | Puerto | Hacia | Puerto | Cable |
|---|---|---|---|---|---|
| 4 | Server1 (izq.) | `FastEthernet0` | Switch1 | `FastEthernet1/1` | Directo |
| 5 | Server0 (der.) | `FastEthernet0` | Switch1 | `FastEthernet0/1` | Directo |
| 6 | Switch1 | `FastEthernet2/1` ⚠️ | Router1 | `FastEthernet1/0` | Directo |
| 7 | **Laptop0** | **`RS 232`** | **Router1** | **`Console`** | **Consola (celeste)** |

> ⚠️ **Sobre el puerto `Fa2/1` del cable 6.** En la topología de clase este cable salía de `Fa4/1`, pero esa
> ranura del `Switch-PT` **es de fibra** y no acepta cable de cobre. Se usa `Fa2/1`, que sí es de cobre.
> Cualquier puerto de cobre libre sirve igual — ver el recuadro de §13.1.
>
> ⚠️ **En el cable 7 los puertos son distintos en cada extremo.** En la laptop eliges `RS 232` (el puerto
> serie), en el router eliges `Console`. Si buscas un `Fa0` en el router para el cable de consola, no lo
> encontrarás: la consola es un puerto aparte, el circulito del costado.
>
> ⚠️ **El enlace 6 se verá ROJO** hasta que hagas el `no shutdown` del paso 15 — igual que en la foto de la
> clase. Es normal, no es un error de cableado.

---

## 14. Configurar el router DESDE la laptop (cable de consola)

Esta es la parte "extra", y es la más parecida a la realidad: así se configura un equipo Cisco **recién sacado
de la caja**, cuando todavía no tiene ninguna IP y no puedes llegar por red.

1. Clic en **Laptop0** → pestaña **`Desktop`** («Escritorio»).
2. Abre **`Terminal`** (icono de ventana negra). *No* es el `Command Prompt`.
3. Aparece la ventana **Terminal Configuration** («Configuración de terminal»). **Déjala tal cual**, son los
   valores estándar de Cisco:

   | Campo (inglés) | Campo (español) | Valor |
   |---|---|---|
   | Bits Per Second | Bits por segundo | **9600** |
   | Data Bits | Bits de datos | **8** |
   | Parity | Paridad | **None** / Ninguna |
   | Stop Bits | Bits de parada | **1** |
   | Flow Control | Control de flujo | **None** / Ninguno |

4. Clic en **`OK`**. Aparece el prompt del router: `Router1>`.
5. **Desde aquí escribes todos los comandos del paso 15**, en vez de usar la pestaña CLI del router.

> 💡 **Por qué esto importa (concepto nuevo).** Hay dos formas de administrar un equipo de red:
>
> | | **Por consola** (*out-of-band*) | **Por telnet/SSH** (*in-band*) |
> |---|---|---|
> | Necesita red funcionando | ❌ No | ✅ Sí |
> | Necesita que el equipo tenga IP | ❌ No | ✅ Sí |
> | Requiere estar físicamente al lado | ✅ Sí | ❌ No |
> | Se usa para | Configuración inicial, rescate | Administración del día a día |
>
> El telnet que hiciste al switch en la Parte 1 (§5.2) es *in-band*: solo funcionó **porque la red ya
> andaba**. Si te equivocas y dejas un equipo incomunicado, la consola es la única salida. Por eso siempre hay
> un cable azul celeste en el rack.

---

## 15. Configurar la nueva interfaz del router (Fa1/0)

```cisco
Router1>enable
Router1#configure terminal
Router1(config)#interface fastEthernet 1/0
Router1(config-if)#ip address 192.168.1.1 255.255.255.0
Router1(config-if)#no shutdown
Router1(config-if)#description Segmento de servidores
Router1(config-if)#exit
Router1(config)#exit
Router1#copy running-config startup-config
```

Al hacer `no shutdown` verás en la consola:

```
%LINK-5-CHANGED: Interface FastEthernet1/0, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet1/0, changed state to up
```

…y el enlace rojo del paso 13 **se pondrá verde**.

> 🔑 **El concepto clave de la Parte 2 — y la pregunta que probablemente caiga en el examen:**
>
> **¿Por qué `Fa1/0` NO lleva subinterfaces ni `encapsulation dot1Q`, si `Fa0/0` sí?**
>
> Porque son dos situaciones distintas:
>
> | | `Fa0/0` (Parte 1) | `Fa1/0` (Parte 2) |
> |---|---|---|
> | Cuántas redes pasan por el cable | **3** (VLAN 1, 10 y 20) | **1** (192.168.1.0/24) |
> | ¿Hay que distinguirlas? | Sí → hacen falta **etiquetas** | No → no hay nada que distinguir |
> | Tipo de enlace | **Trunk** | Enlace enrutado normal |
> | Configuración | Subinterfaces + `dot1Q` | IP directa en la interfaz física |
>
> Las subinterfaces y el etiquetado 802.1Q existen **solo** para resolver el problema de "varias redes, un
> cable". Aquí ese problema no existe: los servidores tienen su propio cable y su propia red, así que la
> interfaz se configura como cualquier interfaz de router de toda la vida.

---

## 16. Configurar el switch de servidores (Switch1)

**No requiere ninguna configuración.** Todos sus puertos vienen de fábrica en la VLAN 1 y en modo acceso, que
es exactamente lo que necesitas: los dos servidores y el router están en la misma red, sin VLANs de por medio.

Si quieres dejarlo prolijo para la entrega:

```cisco
Switch>enable
Switch#configure terminal
Switch(config)#hostname Switch1
Switch1(config)#exit
```

> 💡 **Contrasta con `sw1`.** El switch de la Parte 1 necesitó VLANs, puertos de acceso, un trunk, IP de
> gestión y telnet. Este no necesita nada. La diferencia no es el modelo del switch — es **qué le estás
> pidiendo que haga**.

---

## 17. Configurar los servidores

Los servidores en Packet Tracer se configuran solo por interfaz gráfica. Repite esto en **cada uno**.

### 17.1 Dirección IP

**Clic en el servidor → pestaña `Desktop` → `IP Configuration` → marcar `Static`:**

| Servidor                | IP Address    | Subnet Mask     | Default Gateway |
| ----------------------- | ------------- | --------------- | --------------- |
| **Server0** (derecha)   | `192.168.1.2` | `255.255.255.0` | `192.168.1.1`   |
| **Server1** (izquierda) | `192.168.1.3` | `255.255.255.0` | `192.168.1.1`   |

> ⚠️ **El gateway también aquí.** Sin él, el servidor responde a quien esté en su misma red, pero **no puede
> contestarle a PC0 ni a PC1**, que están en 10.0.0.0 y 20.0.0.0. El ping saldría del PC, llegaría al
> servidor… y la respuesta se perdería. Es el mismo error de la Parte 1, repetido.

### 17.2 Encender el servicio web

**Clic en el servidor → pestaña `Services` («Servicios») → `HTTP` en el menú lateral:**

1. **`HTTP`** → marcar **`On`** («Activado»). Suele venir encendido por defecto — verifícalo igual.
2. `HTTPS` puede quedar en `Off`, no lo necesitas.
3. Abajo verás la lista de archivos con `index.html`. Clic en **`(edit)`** a su derecha.
4. Cambia el contenido para poder distinguir un servidor del otro. Por ejemplo, en Server0:

```html
<html>
  <center><font size='+2' color='blue'>
    <b>SERVIDOR 0 - 192.168.1.2</b>
  </font></center>
  <hr>
  <p>Servidor web del laboratorio de VLAN Trunking.</p>
  <p>Si ves esta pagina desde PC0 o PC1, el enrutamiento funciona.</p>
</html>
```

5. Clic en **`Save`** («Guardar»).
6. Repite en Server1 cambiando el texto a `SERVIDOR 1 - 192.168.1.3`.

> 💡 Distinguirlos no es un capricho: es la única forma de comprobar que estás llegando **al servidor que
> crees** y no siempre al mismo.

---

## 18. Pruebas de la Parte 2

### 18.1 Conectividad básica

Desde **PC0 → Desktop → Command Prompt**:

```
ping 192.168.1.1     ← la interfaz Fa1/0 del router
ping 192.168.1.2     ← Server0
ping 192.168.1.3     ← Server1
```

Repite lo mismo desde **PC1**. Los dos PCs, aunque estén en VLANs distintas, deben alcanzar ambos servidores.

### 18.2 La prueba que importa: navegar

Desde **PC0 → Desktop → `Web Browser` («Navegador web»)**:

```
http://192.168.1.2
```

Debe cargar la página de **SERVIDOR 0**. Prueba también `http://192.168.1.3` y repite desde **PC1**.

> ⚠️ **Escribe la IP, no un nombre.** `http://www.servidor.com` daría *"Host Name Unresolved"* porque no hay
> servidor DNS en la topología. Con la IP funciona perfectamente y es lo que pide la práctica.

### 18.3 Verificación en el router

```cisco
Router1#show ip interface brief
```

Deben aparecer **up/up**: `FastEthernet0/0`, sus tres subinterfaces y `FastEthernet1/0`.

```cisco
Router1#show ip route
```

Ahora deben verse **cuatro** redes conectadas:

```
C    1.0.0.0/8 is directly connected, FastEthernet0/0.1
C    10.0.0.0/8 is directly connected, FastEthernet0/0.2
C    20.0.0.0/8 is directly connected, FastEthernet0/0.3
C    192.168.1.0/24 is directly connected, FastEthernet1/0
```

> 💡 **Fíjate en que no configuraste ninguna ruta.** El router sabe llegar a las cuatro redes porque están
> *directamente conectadas* (la `C` del margen). Solo harían falta rutas estáticas o un protocolo de
> enrutamiento si hubiera un segundo router de por medio.

### 18.4 Que la consola sirvió de algo

Comprueba que realmente configuraste por consola: en la **Terminal de Laptop0** escribe

```cisco
Router1#show running-config
```

y verifica que aparece el bloque de `interface FastEthernet1/0` que escribiste desde ahí.

---

## 19. Errores frecuentes de la Parte 2

| Síntoma | Causa probable | Solución |
|---|---|---|
| `% Invalid input` al escribir `interface fa1/0` | No instalaste el módulo NM-1FE-TX | Paso 12 |
| El módulo no se deja arrastrar | El router está encendido | Apágalo en `Physical` primero |
| **El cable no engancha en un puerto del Switch-PT** | Esa ranura tiene módulo de **fibra** (`1FFE`), no de cobre | Usa una ranura de cobre (`1CFE`), p. ej. `Fa2/1` · §13.1 |
| El puerto aparece gris o no seleccionable al soltar el cable | Tipo de cable ≠ tipo de puerto | Verifica en `Physical` qué módulo tiene esa ranura |
| Enlace Switch1–Router1 en **rojo** | Falta `no shutdown` en Fa1/0 | Paso 15 |
| Ping a 192.168.1.1 ✅ pero a 192.168.1.2 ❌ | Servidor sin IP o sin gateway | Paso 17.1 |
| Ping al servidor ✅ pero la web no carga | Servicio HTTP apagado | `Services → HTTP → On` |
| El navegador dice *Host Name Unresolved* | Escribiste un nombre, no una IP | Usa `http://192.168.1.2` |
| El navegador dice *Request Timeout* | Es problema de red, no del servidor | Revisa gateway del servidor |
| La Terminal de la laptop no muestra nada | Cable en el puerto equivocado, o velocidad ≠ 9600 | Debe ir `RS 232` ↔ `Console`, a 9600 |
| No encuentro el puerto `Console` en el router | Estás buscando entre los FastEthernet | Es un puerto aparte, aparece en la lista al soltar el cable celeste |
| Los servidores se ven entre sí pero no a los PCs | Falta el gateway en los servidores | Paso 17.1 |
| Se rompió la Parte 1 al hacer la Parte 2 | Tocaste `Fa0/0` por error | `Fa0/0` no se toca; lo nuevo va todo en `Fa1/0` |

---

## 20. Conceptos nuevos de esta parte

**Interfaz enrutada vs enlace troncal.** Una interfaz de router con IP propia atiende **una** red. Un trunk con
subinterfaces atiende **varias**. Elegir uno u otro depende de cuántas redes deban compartir el cable, no de
ninguna preferencia técnica.

**Módulo de red (*Network Module*, NM).** Tarjeta de expansión que agrega puertos a un router. Los routers
reales son modulares justo por esto: compras el chasis y le pones las interfaces que necesites. Se instalan
**con el equipo apagado**.

**Gestión out-of-band vs in-band.** *Out-of-band* (consola) no depende de la red y sirve para la configuración
inicial o para rescatar un equipo. *In-band* (telnet/SSH) es cómoda pero se cae cuando la red se cae.

**Servidor HTTP.** Un equipo que escucha peticiones en el puerto 80 y devuelve páginas. Que el ping funcione
solo prueba que hay **conectividad de red**; que cargue la página prueba además que hay un **servicio
escuchando**. Son dos cosas distintas y se diagnostican por separado.

**Por qué el segmento de servidores no lleva VLAN.** Porque una VLAN resuelve el problema de "quiero separar
equipos que comparten switch". Aquí los servidores ya están separados **físicamente**: tienen su propio switch
y su propio cable al router. Añadir una VLAN no aportaría nada.

---

## 21. Configuración completa de la Parte 2 (para copiar)

<details>
<summary><b>Router1 — solo lo nuevo (desde la Terminal de Laptop0)</b></summary>

```cisco
enable
configure terminal
!
interface fastEthernet 1/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown
 description Segmento de servidores
 exit
!
end
copy running-config startup-config
```
</details>

<details>
<summary><b>Switch1 — opcional</b></summary>

```cisco
enable
configure terminal
hostname Switch1
end
copy running-config startup-config
```
</details>

<details>
<summary><b>Servidores — por interfaz gráfica, no hay CLI</b></summary>

```
Server0 → Desktop → IP Configuration → Static
   IP 192.168.1.2 · Máscara 255.255.255.0 · Gateway 192.168.1.1
Server0 → Services → HTTP → On → editar index.html

Server1 → Desktop → IP Configuration → Static
   IP 192.168.1.3 · Máscara 255.255.255.0 · Gateway 192.168.1.1
Server1 → Services → HTTP → On → editar index.html
```
</details>

---

## 22. Checklist de entrega — Parte 2

- [ ] Módulo **NM-1FE-TX** instalado en Router1 (router apagado durante la instalación)
- [ ] `show ip interface brief` muestra `FastEthernet1/0`
- [ ] Switch-PT (Switch1) y 2 Server-PT colocados
- [ ] Laptop-PT colocada
- [ ] 3 cables directos: servidores↔Switch1 y Switch1↔Router1
- [ ] 1 **cable de consola** Laptop0 `RS 232` ↔ Router1 `Console`
- [ ] Fa1/0 con IP 192.168.1.1/24 y `no shutdown` → enlace **verde**
- [ ] Configuración de Fa1/0 hecha **desde la Terminal de la laptop** (no desde la pestaña CLI)
- [ ] Server0 = 192.168.1.2 · Server1 = 192.168.1.3 · **ambos con gateway**
- [ ] Servicio **HTTP encendido** en los dos, con `index.html` distinto en cada uno
- [ ] Ping desde PC0 y desde PC1 a ambos servidores (captura)
- [ ] `http://192.168.1.2` y `http://192.168.1.3` cargando desde PC0 y PC1 (captura)
- [ ] `show ip route` muestra las **4** redes conectadas (captura)
- [ ] **La Parte 1 sigue funcionando**: ping PC0 ↔ PC1 y telnet a 1.0.0.2
- [ ] Archivo guardado como `VLAN-Trunking.pkt`
