---
tipo: guía-paso-a-paso
creado: 2026-09-11
tags: [frame-relay, nat, wan, packet-tracer, infraestructura-2]
---

# Guía paso a paso — Frame Relay y NAT

> **Objetivo:** Configurar una topología WAN con Frame Relay entre un router Gateway y un ISP, implementar NAT dinámico para traducir direcciones privadas (10.10.10.0/24) a un pool público (199.99.9.40–199.99.9.62), y verificar la conectividad extremo a extremo.

---

## 1 · Topología

```
   [5 PCs]       [5 PCs]
      |             |
   [Server]--[Sw1]---------[Sw2]                            [Loopback 0]
10.10.10.20   | Fa0         S0           S0                  172.16.1.1/32
              +---[Gateway]---[Cloud FR]---[ISP]--------------+
                10.10.10.1    200.2.2.18   200.2.2.17         | Fa0
                  /24           /30          /30              |
                                                        [Server ISP]
                                                       209.165.200.2/24
```

| Dispositivo | Interfaz | IP | Máscara |
|---|---|---|---|
| Gateway | FastEthernet 0/0 | 10.10.10.1 | 255.255.255.0 |
| Gateway | Serial 0/0/0 | 200.2.2.18 | 255.255.255.252 |
| ISP | Serial 0/0/0 | 200.2.2.17 | 255.255.255.252 |
| ISP | FastEthernet 0/0 | 209.165.200.1 | 255.255.255.0 |
| ISP | Loopback 0 | 172.16.1.1 | 255.255.255.255 |
| PC0 - PC9 | NIC | 10.10.10.10 - 10.10.10.19 | 255.255.255.0 (GW: 10.10.10.1) |
| Web Server| NIC | 10.10.10.20 | 255.255.255.0 (GW: 10.10.10.1) |
| Server ISP| NIC | 209.165.200.2 | 255.255.255.0 (GW: 209.165.200.1) |

---

## 2 · Componentes en Packet Tracer y dónde encontrarlos

| Qué necesitas | Categoría (barra inferior) | Modelo sugerido |
|---|---|---|
| 2 Routers | **Routers** | Router genérico (**Router-PT**) o **1841** |
| 2 Switches | **Switches** | **2960** o Switch-PT |
| 1 Nube Frame Relay | **WAN Emulation** | **Cloud-PT** |
| 10 PCs | **End Devices** | **PC-PT** (5 por cada switch) |
| 2 Servidores | **End Devices** | **Server-PT** (1 Interno, 1 en el ISP) |

### Cableado

| De | A | Cable |
|---|---|---|
| PCs 0-4 (FastEthernet) | Sw1 (Fa0/1 a Fa0/5) | Straight-through |
| PCs 5-9 (FastEthernet) | Sw2 (Fa0/1 a Fa0/5) | Straight-through |
| Web Server (FastEthernet)| Sw1 (Fa0/6) | Straight-through |
| Sw2 (Gig0/1 o Fa0/24) | Sw1 (Gig0/1 o Fa0/24) | Crossover (o cruzado) |
| Sw1 (Fa0/7) | Gateway Fa0/0 | Straight-through |
| Gateway S0/0 (DCE o DTE) | Cloud-PT Serial0 | Serial DCE / DTE |
| Cloud-PT Serial1 | ISP S0/0 (DCE) | Serial DCE / DTE |
| Server ISP (FastEthernet)| ISP Fa0/0 | Crossover (o cruzado) |

> ⚠️ **El clock rate va en el lado DCE.** En esta práctica se pone en el ISP (S0/0). Si Packet Tracer te obliga a usar un lado específico, ajusta según el cable que conectaste.

---

## 3 · Configuración de la Nube Frame Relay (Cloud-PT)

Esto se hace **desde la GUI** de Packet Tracer (clic en la nube → pestaña Config o CLI no aplica aquí):

### Pestaña "Config" → Frame Relay

1. **Serial0** → Agregar DLCI:
   - DLCI: `101`
   - Nombre: `Gateway` (o cualquier nombre descriptivo)
   - Clic en **Add**

2. **Serial1** → Agregar DLCI:
   - DLCI: `102`
   - Nombre: `ISP` (o cualquier nombre descriptivo)
   - Clic en **Add**

3. **Frame Relay** (pestaña de mapeo):
   - Serial0 DLCI 101 ↔ Serial1 DLCI 102
   - Clic en **Add**

> ⚠️ Si no ves la pestaña "Frame Relay", asegúrate de haber seleccionado la nube **Cloud-PT** (WAN Emulation), no un router.

---

## 4 · Comandos del Router Gateway — COPIAR Y PEGAR

> **Instrucción:** Abre la CLI del router Gateway. Si ves `Router>`, escribe `enable` y presiona Enter. Luego pega **todo el bloque** de una sola vez.

```
enable
configure terminal
hostname Gateway
!
interface FastEthernet 0/0
ip address 10.10.10.1 255.255.255.0
ip nat inside
no shutdown
exit
!
interface Serial 0/0/0
ip address 200.2.2.18 255.255.255.252
encapsulation frame-relay
frame-relay interface-dlci 101
ip nat outside
no shutdown
exit
!
ip route 0.0.0.0 0.0.0.0 200.2.2.17
!
ip nat pool public-access 199.99.9.40 199.99.9.62 netmask 255.255.255.224
access-list 1 permit 10.10.10.0 0.0.0.255
ip nat inside source list 1 pool public-access
ip nat inside source static 10.10.10.20 199.99.9.33
!
end
copy running-config startup-config

```

> 💡 Cuando pregunte `[startup-config]?` solo presiona **Enter**.

---

## 5 · Comandos del Router ISP — COPIAR Y PEGAR

> **Instrucción:** Abre la CLI del router ISP. Si ves `Router>`, escribe `enable` y presiona Enter. Luego pega **todo el bloque** de una sola vez.

```
enable
configure terminal
hostname ISP
!
interface Loopback 0
ip address 172.16.1.1 255.255.255.255
no shutdown
exit
!
interface FastEthernet 0/0
ip address 209.165.200.1 255.255.255.0
no shutdown
exit
!
interface Serial 0/0/0
ip address 200.2.2.17 255.255.255.252
clock rate 64000
encapsulation frame-relay
frame-relay interface-dlci 102
no shutdown
exit
!
ip route 199.99.9.32 255.255.255.224 200.2.2.18
!
end
copy running-config startup-config

```

> 💡 Cuando pregunte `[startup-config]?` solo presiona **Enter**.

---

## 6 · Configuración de los PCs y el Servidor Web

**1. Para los PCs:** (Desktop → **IP Configuration**)
Asigna a cada PC una IP única dentro del rango. Ejemplo para el **PC0**:

| Campo           | Valor                                           |
| --------------- | ----------------------------------------------- |
| IP Address      | `10.10.10.10` (luego `.11`, `.12`, hasta `.19`) |
| Subnet Mask     | `255.255.255.0`                                 |
| Default Gateway | `10.10.10.1`                                    |

**2. Para el Servidor Web (FARMACIA):**
- Ve a Desktop → **IP Configuration**:
  - IP Address: `10.10.10.20`
  - Subnet Mask: `255.255.255.0`
  - Default Gateway: `10.10.10.1`
- Ve a Services → **HTTP**: Asegúrate de que HTTP y HTTPS estén en **ON**.

**3. Para el Servidor Externo (ISP):**
- Ve a Desktop → **IP Configuration**:
  - IP Address: `209.165.200.2`
  - Subnet Mask: `255.255.255.0`
  - Default Gateway: `209.165.200.1`
- Ve a Services → **HTTP**: Asegúrate de que HTTP y HTTPS estén en **ON**.

---

## 7 · Verificación y pruebas

### 7.1 Desde el ISP — verificar enlace Frame Relay

```
ping 200.2.2.18
```

✅ Debe responder con `!!!!` (éxito).

### 7.2 Desde PC0 — verificar NAT Dinámico hacia Internet

Abre el **Web Browser** del PC0 (o cualquier otro PC interno) y entra a la IP del servidor externo: `http://209.165.200.2`.

✅ Debe responder con éxito cargando la página web. Esto confirma que el NAT Dinámico está traduciendo la IP privada de tu PC (`10.10.10.X`) a una de las IPs públicas del pool, permitiendo la salida a "Internet".

### 7.3 Prueba del Servidor Web (NAT Estático)

- Desde cualquier PC interno (ej. PC0), abre el **Web Browser** y entra a `http://10.10.10.20`. Debe mostrar la página web de bienvenida.
- El servidor quedó mapeado de forma estática a la IP pública `199.99.9.33`. En un entorno real, los usuarios externos entrarían usando `http://199.99.9.33`. (En Packet Tracer, si conectas un PC o laptop temporalmente a la red del ISP con IP 200.2.2.X, podrás entrar con esta IP).

### 7.4 Desde Gateway — ver traducciones NAT

```
enable
show ip nat translations
```

Deberías ver una entrada como:

```
Pro  Inside global     Inside local      Outside local     Outside global
---  199.99.9.40       10.10.10.10       172.16.1.1        172.16.1.1
```

### 7.4 Otros comandos de verificación útiles

```
show ip route
show ip interface brief
show frame-relay map
show frame-relay pvc
show ip nat statistics
```

---

## 8 · Actividades adicionales

### 8.1 ¿De qué lado debe estar el servidor DNS?

El servidor DNS debe estar **del lado del ISP** (red externa/pública), ya que simula un servicio de Internet. En un escenario real, el DNS público se encuentra fuera de la red de la empresa. Sin embargo, si la empresa tiene un DNS interno para resolución local, puede estar en la red 10.10.10.0/24.

---

## 9 · Resumen de direccionamiento NAT

| Tipo | Rango | Uso |
|---|---|---|
| **Estáticas** | 199.99.9.33 – 199.99.9.39 | Servidores internos mapeados 1:1 |
| **Dinámicas** (pool) | 199.99.9.40 – 199.99.9.62 | PCs y otros hosts internos |
| **Red privada** | 10.10.10.0/24 | LAN interna de la empresa |
| **Enlace WAN** | 200.2.2.16/30 | Enlace serial Gateway ↔ ISP |

---

## 10 · Errores frecuentes

| Problema | Causa probable | Solución |
|---|---|---|
| Interfaces serial en rojo | `encapsulation frame-relay` falta o DLCI mal configurado en la nube | Revisa la nube FR y los DLCIs |
| Ping de PC a 172.16.1.1 falla | Ruta estática o ruta por defecto incorrecta | Verifica `show ip route` en ambos routers |
| NAT no traduce | Falta `ip nat inside` o `ip nat outside` en las interfaces | Revisa con `show ip nat statistics` |
| `Serial0/0 changed state to down` | Cable serial mal conectado o clock rate faltante | Verifica cableado y `clock rate 64000` en lado DCE (ISP) |
| `copy running-config startup-config` no funciona | Estás en modo config | Escribe `end` primero para volver a modo privilegiado |

---

## 11 · Glosario

| Término | Definición |
|---|---|
| **Frame Relay** | Protocolo WAN de capa 2 que usa circuitos virtuales (PVC) para conectar sitios remotos |
| **DLCI** | Data Link Connection Identifier — número que identifica el circuito virtual en Frame Relay (valor local) |
| **NAT** | Network Address Translation — traduce IPs privadas a públicas para acceso a Internet |
| **NAT estático** | Mapeo fijo 1:1 de una IP privada a una IP pública |
| **NAT dinámico** | Asigna IPs públicas de un pool de forma temporal a hosts internos |
| **CIDR** | Classless Inter-Domain Routing — notación de red sin clases (ej: /27) |
| **Loopback** | Interfaz virtual que siempre está activa; simula una red o servicio |
| **Pool NAT** | Conjunto de IPs públicas disponibles para traducción dinámica |
| **ACL** | Access Control List — filtra tráfico; aquí define qué IPs internas se traducen |

---

## Checklist final

- [ ] Nube Frame Relay configurada con DLCIs 101 y 102 mapeados
- [ ] Gateway configurado (Fa0/0, S0/0, Frame Relay, NAT, ruta por defecto)
- [ ] ISP configurado (Loopback, S0/0, Frame Relay, clock rate, ruta estática)
- [ ] PCs configurados con IPs 10.10.10.X, máscara /24, gateway 10.10.10.1
- [ ] Ping exitoso de ISP a 200.2.2.18 (enlace WAN)
- [ ] Ping exitoso de PC0 a 172.16.1.1 (NAT funcionando)
- [ ] `show ip nat translations` muestra la traducción
- [ ] (Adicional) Servidor web agregado y accesible
- [ ] Configuración guardada con `copy running-config startup-config`
