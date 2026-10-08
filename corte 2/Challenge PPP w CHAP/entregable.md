---
tipo: entregable
practica: "Challenge PPP w/CHAP"
estudiante: Harold Camargo
archivo: Challenge-PPP-CHAP.pka
fecha: 2026-10-02
---

# Entregable — Challenge PPP w/CHAP

**Estudiante:** Harold Camargo · **Archivo adjunto:** `Challenge-PPP-CHAP.pka`

---

## 1. Documentación de la topología

```
                                   209.165.201.0/27
                         [Web Server .30]   [Outside Host .2]
                                  \             /
                                  [ ISP Switch ]
                                        | Fa1/0 .1
 192.168.20.0/24        209.165.200.224/27   |                         192.168.2.0/24
[Intranet .254]--[S3]--Fa0/1[ R1 ]S0/1/0====PPP/CHAP====S0/0/0[ ISP ]Fa0/0--Eth6[          ]Coax7--[Cable Modem]--[TW-Cable]
                          .1   |  .225                     .226 DCE  Fa0/1--FE9[ Internet ]Modem4--[DSL Modem]---[TW-DSL]
                               | S0/0/0 .1                                                                 192.168.1.0/24
                               | (Serial1)
                     [ Frame Relay 10.1.1.0/29 · RIPv2 ]
                      (Serial0)/              \(Serial2)
                     S0/0/0 .2               S0/0/0 .3
                       [ R2 ]                   [ R3 ]
                    Fa0/1 .1                    Fa0/1 .1
                       [S1]                     [S2]
                       PC1 .10                  PC3 .10
                 192.168.10.0/24          192.168.30.0/24
```

### Redes

| Red | Subred | Máscara | Uso |
|---|---|---|---|
| LAN R2 | 192.168.10.0 | /24 — 255.255.255.0 | PC1 |
| LAN R1 | 192.168.20.0 | /24 — 255.255.255.0 | Servidor Intranet |
| LAN R3 | 192.168.30.0 | /24 — 255.255.255.0 | PC3 |
| Frame Relay | 10.1.1.0 | /29 — 255.255.255.248 | Hub-and-spoke R1 (hub) – R2 – R3 · RIPv2 |
| WAN R1–ISP | 209.165.200.224 | /27 — 255.255.255.224 | Enlace serial **PPP con CHAP** |
| Red del ISP | 209.165.201.0 | /27 — 255.255.255.224 | Web Server y Outside Host |
| Clientes cable | 192.168.2.0 | /24 | TW-Cable (DHCP del ISP) |
| Clientes DSL | 192.168.1.0 | /24 | TW-DSL (DHCP del ISP) |

### Tabla de direccionamiento

| Dispositivo | Interfaz | IP | Máscara | Gateway |
|---|---|---|---|---|
| R1 | Fa0/1 | 192.168.20.1 | 255.255.255.0 | — |
| R1 | S0/0/0 | 10.1.1.1 | 255.255.255.248 | — |
| R1 | S0/1/0 | 209.165.200.225 | 255.255.255.224 | — |
| R2 | S0/0/0 | 10.1.1.2 | 255.255.255.248 | — |
| R2 | Fa0/1 | 192.168.10.1 | 255.255.255.0 | — |
| R3 | S0/0/0 | 10.1.1.3 | 255.255.255.248 | — |
| R3 | Fa0/1 | 192.168.30.1 | 255.255.255.0 | — |
| ISP | S0/0/0 (DCE) | 209.165.200.226 | 255.255.255.224 | — |
| ISP | Fa1/0 | 209.165.201.1 | 255.255.255.224 | — |
| ISP | Fa0/0 | 192.168.2.1 | 255.255.255.0 | — |
| ISP | Fa0/1 | 192.168.1.1 | 255.255.255.0 | — |
| Intranet | NIC | 192.168.20.254 | 255.255.255.0 | 192.168.20.1 |
| PC1 | NIC | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 |
| PC3 | NIC | 192.168.30.10 | 255.255.255.0 | 192.168.30.1 |
| Web Server | NIC | 209.165.201.30 | 255.255.255.224 | 209.165.201.1 |
| Outside Host | NIC | 209.165.201.2 | 255.255.255.224 | 209.165.201.1 |
| TW-Cable | NIC | DHCP (192.168.2.11+) | 255.255.255.0 | 192.168.2.1 |
| TW-DSL | NIC | DHCP (192.168.1.11+) | 255.255.255.0 | 192.168.1.1 |

### Circuitos Frame Relay (DLCI)

| Router | Puerto de la nube | DLCI local | Llega a |
|---|---|---|---|
| R1 | Serial1 | 102 | R2 (DLCI 201) |
| R1 | Serial1 | 103 | R3 (DLCI 301) |
| R2 | Serial0 | 201 | R1 (y a R3 pasando por R1) |
| R3 | Serial2 | 301 | R1 (y a R2 pasando por R1) |

---

## 2. Protocolos implementados

| Protocolo | Dónde | Para qué |
|---|---|---|
| **PPP** | R1 S0/1/0 ↔ ISP S0/0/0 | Encapsulación de capa 2 del enlace WAN con el ISP (LCP abre y prueba el enlace; IPCP, un NCP, negocia IPv4) |
| **CHAP** | Mismo enlace, en **ambos sentidos** | Autenticación sin enviar la contraseña: desafío → respuesta MD5 → aceptación. Usuario/clave: `R1`/`cisco123` en el ISP y `ISP`/`cisco123` en R1 |
| **Frame Relay** | R1, R2, R3 | WAN de la empresa; topología hub-and-spoke con mapas estáticos `frame-relay map ... broadcast` |
| **RIPv2** | R1, R2, R3 (red 10.1.1.0/29 y LAN) | Enrutamiento dinámico interno; R1 inyecta la ruta por defecto (`default-information originate`) |
| **Ruta estática por defecto** | R1 → 209.165.200.226 | Salida a Internet |
| **NAT con sobrecarga (PAT)** | R1 (inside: Fa0/1, S0/0/0 · outside: S0/1/0) | Las redes privadas salen al ISP con la IP 209.165.200.225; el ISP no necesita rutas hacia 192.168.x.x |
| **DHCP** | ISP | Direcciona a los clientes de cable y DSL |
| **DNS + HTTP** | Web Server | `www.isp.com` → 209.165.201.30 y página web |
| **HTTP** | Intranet | Servidor web interno de la empresa |

### Cómo se configuró PPP con CHAP (comandos clave)

```cisco
! R1
username ISP password cisco123
interface Serial0/1/0
 ip address 209.165.200.225 255.255.255.224
 encapsulation ppp
 ppp authentication chap
 no shutdown

! ISP
username R1 password cisco123
interface Serial0/0/0
 ip address 209.165.200.226 255.255.255.224
 encapsulation ppp
 ppp authentication chap
 clock rate 64000
 no shutdown
```

**Lógica del `username`:** cada router crea un usuario con el **hostname del vecino** y la **contraseña compartida**. Cuando el ISP desafía, R1 responde con su nombre (`R1`) y el hash MD5; el ISP busca `username R1`, calcula el mismo hash con `cisco123` y compara. Como los dos lados tienen `ppp authentication chap`, la autenticación ocurre en ambos sentidos.

---

## 3. Validación

| Prueba | Resultado esperado | ✓ |
|---|---|---|
| `show interfaces serial0/1/0` en R1 | `line protocol is up`, `Encapsulation PPP`, `LCP Open`, `Open: IPCP` | |
| `ping 209.165.200.226` desde R1 | Éxito | |
| Clave CHAP errónea en ISP + `shutdown`/`no shutdown` | Enlace `up/down` → CHAP está validando | |
| `show frame-relay map` en R1 | DLCI 102 y 103 activos | |
| `show ip route` en R2/R3 | Rutas `R` a las demás LAN y `R* 0.0.0.0/0` | |
| PC1, PC3 e Intranet → `http://www.isp.com` | Carga la página del Web Server | |
| `show ip nat translations` en R1 | Traducciones a 209.165.200.225 | |
| TW-Cable / TW-DSL → `http://www.isp.com` | Carga la página | |

*(Pegar debajo de cada prueba la captura correspondiente.)*

---

## 4. Correcciones al enunciado

- ⚠️ El enunciado escribe `ISP(ISP config)# hostname ISP` y `# configure terminal`. Lo correcto es `Router# configure terminal` y luego `Router(config)# hostname ISP`.
- ⚠️ Al bloque del ISP le falta `no shutdown`: sin él la interfaz serial queda apagada y PPP nunca negocia.
- ⚠️ `clock rate 64000` solo funciona en el extremo **DCE** del cable; por eso el cable serial se tendió empezando en el ISP.
- ⚠️ El dibujo muestra `Eth0/1/0` en el ISP; en el router 2811 de Packet Tracer se usó un módulo **NM-1FE-TX** (`FastEthernet1/0`), que cumple la misma función.
- El enunciado no da direcciones para la red Frame Relay ni para los equipos; se asignaron dentro de las subredes del dibujo (tabla de la sección 1).
