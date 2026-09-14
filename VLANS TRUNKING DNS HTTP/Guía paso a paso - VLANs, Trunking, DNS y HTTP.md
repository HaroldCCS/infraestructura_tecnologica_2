---
tipo: laboratorio
practica: "Laboratorio — Creación de VLANs, Trunking, DNS y HTTP"
fuente: "LABORATORIO INFRA 2 Vlans Trunking DNS HHTP.docx"
software: Cisco Packet Tracer 7.x / 8.x
creado: 2026-08-21
tags: [redes, vlan, trunking, 802.1q, rip, dns, http, telnet, packet-tracer, cisco]
---

# Laboratorio — VLANs, Trunking, DNS y HTTP

> **Objetivo:** montar una topología con dos VLAN enrutadas por *router-on-a-stick*, unir dos sedes por un enlace serial con **RIP**, y publicar un **servidor DNS** y un **servidor HTTP** a los que se llegue por nombre desde cualquier VLAN.

Es la práctica más completa del vault: junta VLAN + troncal (P2), subinterfaces 802.1Q (P3), enrutamiento dinámico (Port Security/RIPv2) y le añade **servicios de aplicación** (DNS, HTTP) y **acceso remoto** (consola y Telnet).

## Contenido

| | Alcance | Secciones |
|---|---|---|
| **A. Montaje** | Arquitectura, componentes, cableado, direccionamiento | §1 – §4 |
| **B. Comandos** | Switch0, Router0 y Router1, de un solo pegue | §5 |
| **C. Servicios** | DNS, HTTP y acceso por consola/Telnet | §6 – §8 |
| **D. Cierre** | Pruebas, capturas, errores frecuentes, checklist | §9 – §12 |

---

# A. MONTAJE

## 1. Arquitectura

```
   219.17.100.0/24                                    223.8.151.0/24
   ┌──────────┐                                        ┌──────────┐
   │ Serv DNS │ .2                                  .2 │ Serv WEB │
   └────┬─────┘                                        └─────┬────┘
        │                                                    │
   ┌────┴─────┐        ┌────────────┐              ┌─────────┴──┐
   │ Switch2  ├────────┤            ├──────────────┤  Switch1   │
   └────┬─────┘  Fa0/0 │  Router1   │ Fa0/1        └─────────┬──┘
        │              └──────┬─────┘                       │
   ┌────┴─────┐            S0/0│ 201.100.11.1          ┌─────┴────┐
   │   PC2    │ .3             │                    .3 │   PC3    │
   └──────────┘                ║  serial (RIP)         └──────────┘
                               ║
                          S0/0 ║ 201.100.11.2  (DCE, clock rate)
                        ┌──────┴─────┐
                        │  Router0   │   Fa0/0.1 → 1.0.0.1   (VLAN 1, nativa)
                        └──────┬─────┘   Fa0/0.2 → 10.0.0.1  (VLAN 10)
                          Fa0/0│         Fa0/0.3 → 20.0.0.1  (VLAN 20)
                               │ ← troncal 802.1Q
                          Fa0/1│
                        ┌──────┴─────┐
                        │  Switch0   │  SVI VLAN 1 → 1.0.0.2
                        └──┬──────┬──┘
                     Fa0/2 │      │ Fa0/4
                      ┌────┴──┐ ┌─┴─────┐
                      │  PC0  │ │  PC1  │
                      └───┬───┘ └───────┘
                    VLAN 10│      VLAN 20
                    10.0.0.2      20.0.0.2
                          ╎
                          ╎ cable de CONSOLA (celeste)
                          └╌╌╌► puerto Console de Switch0
```

**Las tres piezas del laboratorio:**

1. **Capa 2 + capa 3 en la sede de abajo.** Switch0 separa PC0 y PC1 en dos VLAN. Router0 las vuelve a comunicar con tres subinterfaces sobre un solo cable (*router-on-a-stick*).
2. **La WAN.** Router0 y Router1 se unen por serial y se aprenden las redes con **RIP**.
3. **Los servicios.** Router1 cuelga dos redes: una con el servidor DNS y otra con el servidor WEB. Al final, cualquier PC debe abrir `www.teleinfo.com` **por nombre**.

### Redes

| Red | Dirección | Máscara | Dónde |
|---|---|---|---|
| VLAN 1 (administración) | `1.0.0.0` | `255.0.0.0` (/8) | SVI de Switch0 |
| VLAN 10 — INGENIERIA | `10.0.0.0` | `255.0.0.0` (/8) | PC0 |
| VLAN 20 — VENTAS | `20.0.0.0` | `255.0.0.0` (/8) | PC1 |
| WAN serial | `201.100.11.0` | `255.255.255.0` (/24) | Router0 ↔ Router1 |
| LAN del DNS | `219.17.100.0` | `255.255.255.0` (/24) | Switch2 |
| LAN del WEB | `223.8.151.0` | `255.255.255.0` (/24) | Switch1 |

> 💡 **Por qué esas máscaras.** `1.0.0.0`, `10.0.0.0` y `20.0.0.0` son redes **clase A** → máscara por defecto `255.0.0.0`. `201.100.11.0`, `219.17.100.0` y `223.8.151.0` son **clase C** → `255.255.255.0`. El enunciado deja las máscaras en blanco justamente para que las deduzcas.

---

## 2. Componentes y dónde encontrarlos

| Qué necesitas | Categoría (inglés) | Categoría (español) | Modelo |
|---|---|---|---|
| Switch0 (el de las VLAN) | **Network Devices → Switches** | Dispositivos de red → Conmutadores | **2950T-24** |
| Switch1 y Switch2 | **Network Devices → Switches** | Dispositivos de red → Conmutadores | **Switch-PT** (genérico) |
| Router0 | **Network Devices → Routers** | Dispositivos de red → Enrutadores | **2620XM** + `WIC-1T` |
| Router1 | **Network Devices → Routers** | Dispositivos de red → Enrutadores | **2621XM** + `WIC-1T` |
| PC0…PC3 | **End Devices → End Devices** | Dispositivos finales | **PC** (`PC-PT`) |
| Servidor DNS y Servidor WEB | **End Devices → End Devices** | Dispositivos finales | **Server** (`Server-PT`) |
| Cable directo | **Connections** | Conexiones | **Copper Straight-Through** |
| Cable serial | **Connections** | Conexiones | **Serial DCE** (el del reloj ⏱) |
| Cable de consola | **Connections** | Conexiones | **Console** (celeste) |

> ⚠️ **Router1 necesita DOS interfaces FastEthernet** (una hacia Switch2 y otra hacia Switch1), y el **2620XM solo trae una**. Dos salidas:
>
> | Opción | Cómo | Interfaces que quedan |
> |---|---|---|
> | **Recomendada** | Usar un **2621XM** | `Fa0/0` y `Fa0/1` de fábrica |
> | Fiel al enunciado | 2620XM + módulo **NM-2FE2W** | `Fa0/0` + `Fa1/0` y `Fa1/1` |
>
> Esta guía usa `Fa0/0` y `Fa0/1` (opción recomendada). Si eliges la segunda, cambia `Fa0/1` por `Fa1/0` en los comandos de §5.3.
>
> 📌 **Menciónalo en el informe:** la tabla de componentes del enunciado pide 2620XM para los dos routers, pero uno de ellos no tiene suficientes puertos para la topología que el mismo enunciado dibuja.

### Instalar los módulos (routers apagados)

En **Router0** y **Router1**:

1. Clic en el router → pestaña **`Physical`**.
2. **Apaga** el equipo con el interruptor del chasis.
3. Arrastra **`WIC-1T`** desde `MODULES` a una ranura libre.
4. **Enciéndelo**.
5. Verifica con `show ip interface brief`: debe aparecer `Serial0/0`.

> ⚠️ Otras dos cosas de la tabla de componentes del enunciado que no cuadran: dice **2 switches** cuando la topología dibuja **3**, y **3 cables directos** cuando hacen falta **9**. Lo correcto es lo que muestra la imagen.

---

## 3. Cableado

| # | Desde | Puerto | Hacia | Puerto | Cable |
|---|---|---|---|---|---|
| 1 | PC0 | `Fa0` | Switch0 | `Fa0/2` | Directo |
| 2 | PC1 | `Fa0` | Switch0 | `Fa0/4` | Directo |
| 3 | Switch0 | `Fa0/1` | Router0 | `Fa0/0` | Directo |
| 4 | **Router0** (DCE ⏱) | `Se0/0` | Router1 | `Se0/0` | **Serial DCE** |
| 5 | Router1 | `Fa0/0` | Switch2 | `Fa0/1` | Directo |
| 6 | Switch2 | `Fa0/2` | Servidor DNS | `Fa0` | Directo |
| 7 | Switch2 | `Fa0/3` | PC2 | `Fa0` | Directo |
| 8 | Router1 | `Fa0/1` | Switch1 | `Fa0/1` | Directo |
| 9 | Switch1 | `Fa0/2` | Servidor WEB | `Fa0` | Directo |
| 10 | Switch1 | `Fa0/3` | PC3 | `Fa0` | Directo |
| 11 | PC0 | `RS 232` | Switch0 | `Console` | **Console** (celeste) |

> ⏱ **El primer equipo donde haces clic con el cable serial es el DCE.** Empieza por **Router0**, que es el que lleva `clock rate`.
>
> 🔌 **El cable de consola no transporta datos de red.** Es solo para administrar Switch0 desde PC0 sin usar la red — el equivalente a sentarse frente al equipo. Se usa desde `Desktop → Terminal`.

---

## 4. Direccionamiento de los equipos finales

`Desktop → IP Configuration → Static`. **Rellena también el DNS Server en todos**: es lo que permite navegar por nombre.

| Equipo | Dirección IP | Máscara | Gateway | DNS Server |
|---|---|---|---|---|
| PC0 | `10.0.0.2` | `255.0.0.0` | `10.0.0.1` | `219.17.100.2` |
| PC1 | `20.0.0.2` | `255.0.0.0` | `20.0.0.1` | `219.17.100.2` |
| PC2 | `219.17.100.3` | `255.255.255.0` | `219.17.100.1` | `219.17.100.2` |
| PC3 | `223.8.151.3` | `255.255.255.0` | `223.8.151.1` | `219.17.100.2` |
| Servidor DNS | `219.17.100.2` | `255.255.255.0` | `219.17.100.1` | `219.17.100.2` |
| Servidor WEB | `223.8.151.2` | `255.255.255.0` | `223.8.151.1` | `219.17.100.2` |

> 🔑 **La columna de Gateway venía vacía en el enunciado**: es parte del ejercicio. La regla es siempre la misma — el gateway de un equipo es **la interfaz del router que está en su misma red**. PC0 está en la VLAN 10, y quien tiene una IP en `10.0.0.0` es la subinterfaz `Fa0/0.2` de Router0 → `10.0.0.1`.

---

# B. COMANDOS

## 5. Configuración de los equipos activos

Tres bloques. **Switch1 y Switch2 no se configuran**: son switches genéricos de capa 2 y funcionan tal cual salen de la caja.

### 5.1 Switch0 — VLAN, puertos de acceso y troncal

```cisco
enable
configure terminal
hostname Sw1
!
enable password cisco
!
line vty 0 4
 password cisco
 login
 exit
line console 0
 password cisco
 login
 logging synchronous
 exit
!
vlan 10
 name INGENIERIA
 exit
vlan 20
 name VENTAS
 exit
!
interface vlan 1
 ip address 1.0.0.2 255.0.0.0
 no shutdown
 exit
ip default-gateway 1.0.0.1
!
interface fastEthernet 0/2
 switchport mode access
 switchport access vlan 10
 exit
!
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

> 💡 **El enunciado crea las VLAN con `vlan database`** (`Sw1# vlan database` → `vlan 10 name INGENIERIA`). Es la sintaxis antigua de IOS 12.0; sigue funcionando en el 2950 pero está obsoleta y en varias versiones de Packet Tracer ni siquiera existe. Los comandos de arriba hacen exactamente lo mismo desde el modo de configuración global, que es la forma actual. Si tu docente pide la sintaxis literal, es:
>
> ```cisco
> vlan database
> vlan 10 name INGENIERIA
> vlan 20 name VENTAS
> exit
> ```
>
> 🔑 **`ip default-gateway 1.0.0.1` es imprescindible.** Un switch de capa 2 no enruta: sin gateway, su SVI solo respondería a equipos de la red `1.0.0.0`, y el Telnet desde PC1 (que está en `20.0.0.0`) fallaría.

### 5.2 Router0 — subinterfaces, serial y RIP

```cisco
enable
configure terminal
hostname Router0
!
enable password cisco
!
line vty 0 4
 password cisco
 login
 exit
line console 0
 password cisco
 login
 logging synchronous
 exit
!
interface fastEthernet 0/0
 no shutdown
 exit
!
interface fastEthernet 0/0.1
 encapsulation dot1Q 1 native
 ip address 1.0.0.1 255.0.0.0
 exit
!
interface fastEthernet 0/0.2
 encapsulation dot1Q 10
 ip address 10.0.0.1 255.0.0.0
 exit
!
interface fastEthernet 0/0.3
 encapsulation dot1Q 20
 ip address 20.0.0.1 255.0.0.0
 exit
!
interface serial 0/0
 ip address 201.100.11.2 255.255.255.0
 clock rate 128000
 no shutdown
 exit
!
router rip
 version 2
 no auto-summary
 network 1.0.0.0
 network 10.0.0.0
 network 20.0.0.0
 network 201.100.11.0
 exit
!
end
copy running-config startup-config
```

> 🔑 **`no shutdown` va en la interfaz FÍSICA `Fa0/0`.** Las subinterfaces heredan su estado: si la física está apagada, las tres quedan abajo por bien configuradas que estén. Es el error número uno de esta práctica.
>
> 🔑 **`clock rate 128000` solo aquí.** Router0 es el DCE. Si falta, el serial queda `up/down` (línea arriba, protocolo abajo) y nunca funciona.
>
> 🔑 **`encapsulation dot1Q 1 native`.** La VLAN 1 viaja **sin etiqueta** por el troncal. La palabra `native` le dice al router que espere tramas sin etiquetar en esa subinterfaz; sin ella, el Switch0 queda inalcanzable desde las otras redes.

### 5.3 Router1 — dos LAN, serial y RIP

```cisco
enable
configure terminal
hostname Router1
!
enable password cisco
!
line vty 0 4
 password cisco
 login
 exit
line console 0
 password cisco
 login
 logging synchronous
 exit
!
interface fastEthernet 0/0
 ip address 219.17.100.1 255.255.255.0
 no shutdown
 exit
!
interface fastEthernet 0/1
 ip address 223.8.151.1 255.255.255.0
 no shutdown
 exit
!
interface serial 0/0
 ip address 201.100.11.1 255.255.255.0
 no shutdown
 exit
!
router rip
 version 2
 no auto-summary
 network 201.100.11.0
 network 219.17.100.0
 network 223.8.151.0
 exit
!
end
copy running-config startup-config
```

> ℹ️ Si usaste un 2620XM con `NM-2FE2W`, la segunda interfaz se llama **`Fa1/0`** en vez de `Fa0/1`.
>
> ℹ️ **`version 2` y `no auto-summary` son opcionales aquí** porque todas las redes usan su máscara por defecto (clase A o clase C) y RIPv1 las resolvería igual. Los incluyo porque es la buena práctica y porque no cuesta nada; si tu docente pide la versión literal del enunciado, basta con quitar esas dos líneas.

### 5.4 Verificar el enrutamiento antes de seguir

```cisco
show ip route
show ip protocols
show ip interface brief
```

En **ambos routers**, `show ip route` debe mostrar las **6 redes**: las propias con `C` (conectadas) y las del otro extremo con `R` (aprendidas por RIP).

```
R    219.17.100.0/24 [120/1] via 201.100.11.1, 00:00:09, Serial0/0
R    223.8.151.0/24  [120/1] via 201.100.11.1, 00:00:09, Serial0/0
```

**Prueba de fuego:** `ping 219.17.100.2` desde PC0. Si esto falla, **no sigas con DNS ni HTTP** — arregla primero el enrutamiento o después no sabrás qué está fallando.

---

# C. SERVICIOS

## 6. Servidor WEB (HTTP)

Clic en **Servidor WEB** → pestaña **`Services`** → **`HTTP`**.

1. `HTTP` en **On** (`HTTPS` puedes dejarlo en Off).
2. En la lista de archivos, busca **`index.html`** y pulsa **`(edit)`**.
3. Borra el contenido y pega exactamente esto:

```html
<html>
<center><font size='+2' color='blue'>Teleinformática 3</font></center>
<hr>Servidor Teleinformática 3. Bienvenido
<p>Enlaces:
<br><a href='helloworld.html'>Página pequeña.</a>
<br><a href='copyrights.html'>Derechos de Autor</a>
</html>
```

4. **`Save`**.

> 💡 `helloworld.html` y `copyrights.html` ya vienen creados por defecto en el servidor de Packet Tracer, así que los dos enlaces funcionan sin hacer nada más. Compruébalo: son parte de la prueba.

## 7. Servidor DNS

Clic en **Servidor DNS** → pestaña **`Services`** → **`DNS`**.

1. `DNS Service` en **On**.
2. Añade el registro:

| Campo | Valor |
|---|---|
| Name | `www.teleinfo.com` |
| Type | `A Record` |
| Address | `223.8.151.2` |

3. **`Add`**.

> ⚠️ **Diferencia con el enunciado — léela.** El documento dice *"Configurar el DNS con la siguiente información: `www.teleinfo.com: 219.17.100.2`"*, pero **`219.17.100.2` es el propio servidor DNS**, no el servidor WEB. Si registras esa dirección, al abrir `www.teleinfo.com` no verás la página que te pidieron montar: llegarás al DNS.
>
> **Lo que hago:** apunto `www.teleinfo.com` al **servidor WEB (`223.8.151.2`)**, que es lo que hace funcionar la prueba, y añado un segundo registro con la dirección literal del enunciado para no perder nada:
>
> | Name | Type | Address |
> |---|---|---|
> | `www.teleinfo.com` | A Record | `223.8.151.2` |
> | `dns.teleinfo.com` | A Record | `219.17.100.2` |
>
> Si tu docente insiste en la versión literal, es cambiar una dirección y listo — pero deja la observación en el informe, porque detectar la inconsistencia vale más que copiarla.

## 8. Acceso al Switch0

### 8.1 Por consola (desde PC0)

El cable celeste ya está puesto. En **PC0** → `Desktop` → **`Terminal`** → deja los valores por defecto (9600, 8, N, 1, none) → **`OK`**.

Entras directo a la CLI de Switch0. Pedirá la contraseña de consola: `cisco`.

### 8.2 Por Telnet (desde PC1, el del puerto 4)

En **PC1** → `Desktop` → `Command Prompt`:

```
telnet 1.0.0.2
```

- Contraseña de vty: **`cisco`**
- Después `enable` → contraseña: **`cisco`**
- Y entonces:

```
show vlan
```

> 🔑 **Por qué este Telnet es la prueba más interesante del laboratorio.** PC1 está en la **VLAN 20** (`20.0.0.2`) y el switch responde en la **VLAN 1** (`1.0.0.2`). Son redes distintas, así que el paquete sale de PC1, sube por el troncal etiquetado como VLAN 20, llega a `Fa0/0.3` de Router0, este lo enruta hacia `Fa0/0.1`, y vuelve a bajar por el mismo cable físico como VLAN 1 hasta el switch. **Un solo cable, tres VLAN, y el tráfico dio la vuelta completa por el router.** Eso es router-on-a-stick funcionando.

---

# D. CIERRE

## 9. Pruebas

### 9.1 Ping de una VLAN a otra

Desde **PC0** (`Desktop → Command Prompt`):

```
ping 10.0.0.1        ← su gateway
ping 20.0.0.2        ← PC1, en la otra VLAN
```

Las dos deben responder. La segunda demuestra el enrutamiento entre VLAN.

### 9.2 Ping desde los PC de Switch0 hacia los servidores

Desde **PC0** y desde **PC1**:

```
ping 219.17.100.2    ← Servidor DNS
ping 223.8.151.2     ← Servidor WEB
```

Los cuatro pings deben responder. Si fallan, es RIP: revisa `show ip route`.

### 9.3 Resolución por nombre y navegación

Desde **PC0** → `Desktop` → **`Web Browser`**:

```
http://www.teleinfo.com
```

Debe cargar la página azul de *Teleinformática 3*, y los dos enlaces (*Página pequeña* y *Derechos de Autor*) deben abrir.

Y en el `Command Prompt`:

```
nslookup www.teleinfo.com
```

Debe devolver `223.8.151.2`.

> 💡 **Prueba que vale la pena mostrar:** navega primero a `http://223.8.151.2` (por IP) y después a `http://www.teleinfo.com` (por nombre). La misma página por dos caminos: la segunda solo funciona si el DNS está bien y si los PC tienen configurado el `DNS Server`.

### 9.4 Telnet

Desde **PC1**: `telnet 1.0.0.2` → `enable` → `show vlan` (ver §8.2).

### 9.5 Verificación global

```cisco
! Switch0
show vlan brief
show interfaces trunk
show ip interface brief

! Router0 y Router1
show ip route
show ip protocols
show ip interface brief
show controllers serial 0/0
```

## 10. Capturas para la entrega

| # | Captura |
|---|---|
| 1 | Topología completa con todos los enlaces en verde |
| 2 | `show vlan brief` de Switch0 (VLAN 10 en Fa0/2, VLAN 20 en Fa0/4) |
| 3 | `show interfaces trunk` de Switch0 |
| 4 | `show ip route` de Router0 con las 6 redes |
| 5 | `show ip route` de Router1 con las 6 redes |
| 6 | Ping PC0 → PC1 (de una VLAN a otra) |
| 7 | Ping PC0 → Servidor DNS y → Servidor WEB |
| 8 | `nslookup www.teleinfo.com` desde PC0 |
| 9 | La página web abierta por nombre en el navegador de PC0 |
| 10 | Terminal de consola desde PC0 hacia Switch0 |
| 11 | Telnet desde PC1 con la salida de `show vlan` |

## 11. Errores frecuentes

| Síntoma | Causa | Solución |
|---|---|---|
| El serial queda `up/down` | Falta `clock rate` en el DCE | `interface s0/0` → `clock rate 128000` en **Router0** · comprueba con `show controllers serial 0/0` |
| No aparece `Serial0/0` | Falta el módulo WIC-1T | §2 (con el router **apagado**) |
| Ninguna subinterfaz levanta | `Fa0/0` física apagada | `interface fa0/0` → `no shutdown` |
| PC0 no llega a su gateway | El puerto del PC no está en la VLAN correcta, o el `dot1Q` tiene otro número | Deben coincidir el ID de VLAN del switch y el del `encapsulation` |
| Los PC se ven pero el switch no responde | Falta `native` en `Fa0/0.1`, o falta `ip default-gateway` en el switch | §5.1 y §5.2 |
| Telnet desde PC1 no conecta | Falta `ip default-gateway 1.0.0.1`, o las vty sin `password`/`login` | §5.1 |
| Telnet conecta pero `enable` rebota | Falta `enable password cisco` | §5.1 |
| Bogotá… perdón, Router0 no ve las redes de Router1 | Falta algún `network` en RIP | §5.2 y §5.3 |
| Ping por IP funciona pero por nombre no | Los PC no tienen el `DNS Server` configurado | §4 · ponle `219.17.100.2` a todos |
| `nslookup` resuelve pero la página no abre | El registro apunta al servidor equivocado | §7 · debe ser `223.8.151.2` |
| La página abre pero los enlaces dan error | Borraste `helloworld.html` o `copyrights.html` del servidor | Vuelve a crearlos en `Services → HTTP` |
| `% Invalid input` en `encapsulation` | Lo escribiste en la interfaz física | Debe ser `interface fa0/0.2`, con punto |
| Todo se pierde al reiniciar | Falta guardar | `copy running-config startup-config` |

## 12. Checklist de entrega

**Montaje**

- [ ] 1 switch 2950T-24 + 2 switches genéricos + 2 routers + 4 PC + 2 servidores
- [ ] `WIC-1T` instalado en los dos routers (con el equipo apagado)
- [ ] Router1 con dos interfaces FastEthernet (2621XM, o 2620XM + NM-2FE2W)
- [ ] Cable **serial DCE** desde Router0 · cable de **consola** de PC0 a Switch0
- [ ] Los 11 enlaces de §3

**Capa 3**

- [ ] IP, máscara, gateway **y DNS Server** en los 6 equipos finales
- [ ] VLAN 10 `INGENIERIA` y 20 `VENTAS` creadas en Switch0
- [ ] Fa0/2 → VLAN 10 · Fa0/4 → VLAN 20 · Fa0/1 → troncal
- [ ] SVI `1.0.0.2` y `ip default-gateway 1.0.0.1` en Switch0
- [ ] Las 3 subinterfaces de Router0, con `native` en la de la VLAN 1
- [ ] `no shutdown` en la interfaz **física** `Fa0/0` de Router0
- [ ] `clock rate 128000` en el serial de Router0
- [ ] RIP en los dos routers con todas sus redes
- [ ] `show ip route` muestra las 6 redes en ambos routers

**Servicios**

- [ ] HTTP encendido en el servidor WEB con el HTML del enunciado
- [ ] DNS encendido con el registro `www.teleinfo.com`
- [ ] Contraseñas: consola `cisco`, vty `cisco`, enable `cisco`

**Pruebas**

- [ ] Ping de VLAN 10 a VLAN 20
- [ ] Ping de PC0 y PC1 a los dos servidores
- [ ] `http://www.teleinfo.com` abre desde PC0, con sus dos enlaces
- [ ] Consola desde PC0 hacia Switch0
- [ ] Telnet desde PC1 con `show vlan`

**Cierre**

- [ ] `copy running-config startup-config` en Switch0, Router0 y Router1
- [ ] Las 11 capturas de §10
- [ ] `entregable.md` con las dos preguntas respondidas
- [ ] Archivo guardado como `VLANs-Trunking-DNS-HTTP.pkt` en esta carpeta
- [ ] Mencionadas en el informe las tres inconsistencias del enunciado: el 2620XM sin puertos suficientes (§2), la tabla de componentes (§2) y el registro DNS apuntando al servidor equivocado (§7)
