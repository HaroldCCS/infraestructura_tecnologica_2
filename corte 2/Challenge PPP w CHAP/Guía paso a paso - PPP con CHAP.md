---
tipo: guía-paso-a-paso
creado: 2026-10-02
tags: [ppp, chap, frame-relay, rip, nat, dhcp, wan, packet-tracer, infraestructura-2]
---

# Guía paso a paso — Challenge PPP w/CHAP

> **Objetivo:** montar la topología completa del challenge (empresa con Frame Relay + RIP, enlace WAN al ISP con **PPP y autenticación CHAP**, clientes de cable y DSL del ISP) y lograr que **todas las redes accedan al Web Server del ISP**.
>
> **Modo afán:** si solo quieres ejecutar, usa [[RAPIDO]] de esta misma carpeta. Esta guía tiene lo mismo, más el porqué.

---

# A. MONTAJE

## 1. Topología

![[Pasted image 20261002091639.png]]

| Red | Subred | Gateway | Equipos |
|---|---|---|---|
| LAN R2 | 192.168.10.0/24 | 192.168.10.1 (R2 Fa0/1) | PC1 |
| LAN R1 | 192.168.20.0/24 | 192.168.20.1 (R1 Fa0/1) | Intranet 192.168.20.254 |
| LAN R3 | 192.168.30.0/24 | 192.168.30.1 (R3 Fa0/1) | PC3 |
| Frame Relay | 10.1.1.0/29 | — | R1 .1 · R2 .2 · R3 .3 |
| PPP R1–ISP | 209.165.200.224/27 | — | R1 .225 · ISP .226 |
| Red ISP | 209.165.201.0/27 | 209.165.201.1 (ISP Fa1/0) | Web Server .30 · Outside Host .2 |
| Cable | 192.168.2.0/24 | 192.168.2.1 (ISP Fa0/0) | TW-Cable (DHCP) |
| DSL | 192.168.1.0/24 | 192.168.1.1 (ISP Fa0/1) | TW-DSL (DHCP) |

**Decisiones de diseño (lo que el enunciado no dice):**

- **Frame Relay hub-and-spoke:** R1 es el hub (dos PVC: DLCI 102 → R2, 103 → R3). R2 y R3 solo tienen un PVC hacia R1, así que para hablarse entre ellos mapean la IP del otro spoke al mismo DLCI que va a R1.
- **RIPv2** entre R1, R2 y R3. En R1 se desactiva *split horizon* en la serial Frame Relay: sin eso, R1 no le reenviaría a R3 las rutas que aprendió de R2 por la misma interfaz.
- **NAT con sobrecarga (PAT) en R1:** las redes 192.168.x.x de la empresa salen al ISP con la IP 209.165.200.225. Así el ISP no necesita conocer rutas privadas de la empresa (es lo que pasa en la vida real).
- **Ruta por defecto** en R1 hacia el ISP, repartida a R2 y R3 por RIP con `default-information originate`.
- **ISP** da IP por **DHCP** a los clientes de cable y DSL, y el Web Server hace también de **DNS** (`www.isp.com`).

## 2. Componentes y dónde encontrarlos

| Cant. | Equipo | Modelo | Categoría (barra inferior) |
|---|---|---|---|
| 4 | R1, R2, R3, ISP | **2811** | Network Devices → Routers |
| 4 | S1, S2, S3, ISP Switch | **2960** | Network Devices → Switches |
| 2 | Frame Relay, Internet | **Cloud-PT** | Network Devices → WAN Emulation |
| 1 | Cable Modem | **Cable Modem-PT** | Network Devices → WAN Emulation |
| 1 | DSL Modem | **DSL Modem-PT** | Network Devices → WAN Emulation |
| 5 | PC1, PC3, Outside Host, TW-Cable, TW-DSL | **PC-PT** | End Devices |
| 2 | Intranet, Web Server | **Server-PT** | End Devices |

> La plantilla `PlantillaNubes821 (15).pka` viene **vacía** (no trae equipos): todo se arma desde cero.

### Módulos (Physical → apagar → arrastrar → encender)

| Router | Módulo | Ranura | Interfaces que aparecen |
|---|---|---|---|
| R1 | WIC-2T | HWIC 0 | S0/0/0, S0/0/1 |
| R1 | WIC-2T | HWIC 1 | **S0/1/0**, S0/1/1 |
| R2, R3 | WIC-2T | HWIC 0 | S0/0/0 |
| ISP | WIC-2T | HWIC 0 | S0/0/0 |
| ISP | NM-1FE-TX | NM 1 (ranura grande) | **Fa1/0** |

> ⚠️ El dibujo muestra `Eth0/1/0` en el ISP hacia el ISP Switch. En el 2811 de PT usamos **Fa1/0** (NM-1FE-TX): mismo papel, nombre distinto.

**Nube Internet:** necesita dos puertos Ethernet (uno para cable y otro para DSL). La Cloud-PT de PT 8 ya trae **Ethernet6** y **FastEthernet9**, así que no hay que cambiar módulos.

## 3. Cableado

| Desde | Puerto | Hacia | Puerto | Cable |
|---|---|---|---|---|
| ISP | S0/0/0 | R1 | S0/1/0 | **Serial DCE** — clic primero en el ISP (lleva el reloj) |
| R1 | S0/0/0 | Frame Relay | Serial1 | Serial DTE |
| R2 | S0/0/0 | Frame Relay | Serial0 | Serial DTE |
| R3 | S0/0/0 | Frame Relay | Serial2 | Serial DTE |
| R1 / R2 / R3 | Fa0/1 | S3 / S1 / S2 | Fa0/1 | Straight-Through |
| S3 / S1 / S2 | Fa0/2 | Intranet / PC1 / PC3 | Fa0 | Straight-Through |
| ISP | Fa1/0 | ISP Switch | Fa0/1 | Straight-Through |
| ISP Switch | Fa0/2 · Fa0/3 | Web Server · Outside Host | Fa0 | Straight-Through |
| ISP | Fa0/0 · Fa0/1 | Internet | Ethernet6 · FastEthernet9 | Straight-Through |
| Internet | Coaxial7 | Cable Modem | Port 0 | **Coaxial** |
| Internet | Modem4 | DSL Modem | Port 0 | **Phone** |
| Cable Modem / DSL Modem | Port 1 | TW-Cable / TW-DSL | Fa0 | Straight-Through |

## 4. Configuración de las nubes (GUI)

### 4.1 Frame Relay

`Config` → elige el puerto en el panel izquierdo → escribe DLCI y nombre → **Add**:

| Puerto | DLCI | Nombre |
|---|---|---|
| Serial0 (R2) | 201 | R2-R1 |
| Serial1 (R1) | 102 | R1-R2 |
| Serial1 (R1) | 103 | R1-R3 |
| Serial2 (R3) | 301 | R3-R1 |

`Connections` → **Frame Relay** → `Serial1 R1-R2 ↔ Serial0 R2-R1` **Add** · `Serial1 R1-R3 ↔ Serial2 R3-R1` **Add**.

> El DLCI tiene **significado local**: R1 llama 102 al circuito que R2 llama 201. La nube es la que une los dos extremos.

### 4.2 Internet

`Config` → en la lista de la izquierda (sección **INTERFACE**, baja con el scroll):

1. **Ethernet6** → en *Provider Network* elige **Cable**.
2. El otro puerto Ethernet (el que va a ISP Fa0/1, p. ej. **FastEthernet9**) → *Provider Network* **DSL**.

> Sin este paso, Ethernet6 **no aparece** en el desplegable de Cable/DSL.

Luego `Connections`:
- **Cable:** `Coaxial7 ↔ Ethernet6` → **Add**
- **DSL:** `Modem4 ↔ FastEthernet9` → **Add**

---

# B. COMANDOS

> Abre la CLI de cada router, responde `no` si pregunta por el *initial configuration dialog*, y pega **todo el bloque** de una vez. Cuando pida `Destination filename [startup-config]?` presiona **Enter**.

## 5. R1 — PPP/CHAP, Frame Relay hub, RIP y NAT

```cisco
enable
configure terminal
hostname R1
username ISP password cisco123
interface Serial0/1/0
 description PPP-CHAP hacia ISP
 ip address 209.165.200.225 255.255.255.224
 encapsulation ppp
 ppp authentication chap
 ip nat outside
 no shutdown
 exit
interface Serial0/0/0
 description Frame Relay hub
 ip address 10.1.1.1 255.255.255.248
 encapsulation frame-relay
 frame-relay map ip 10.1.1.2 102 broadcast
 frame-relay map ip 10.1.1.3 103 broadcast
 no ip split-horizon
 ip nat inside
 no shutdown
 exit
interface FastEthernet0/1
 description LAN Intranet
 ip address 192.168.20.1 255.255.255.0
 ip nat inside
 no shutdown
 exit
ip route 0.0.0.0 0.0.0.0 209.165.200.226
router rip
 version 2
 no auto-summary
 network 10.0.0.0
 network 192.168.20.0
 passive-interface FastEthernet0/1
 default-information originate
 exit
access-list 1 permit 192.168.10.0 0.0.0.255
access-list 1 permit 192.168.20.0 0.0.0.255
access-list 1 permit 192.168.30.0 0.0.0.255
ip nat inside source list 1 interface Serial0/1/0 overload
end
copy running-config startup-config

```

Qué hace cada parte:
- `username ISP password cisco123` → credencial que R1 usa para validar al ISP (nombre del **vecino** + clave compartida).
- `encapsulation ppp` + `ppp authentication chap` → cambia la serial de HDLC (por defecto en Cisco) a PPP y exige CHAP.
- `frame-relay map ip ... broadcast` → asocia cada IP vecina a su DLCI; `broadcast` deja pasar las actualizaciones de RIP.
- `no ip split-horizon` → R1 puede reenviar por la misma serial las rutas de un spoke al otro.
- `passive-interface Fa0/1` → no manda RIP hacia la LAN (no hay routers allí).
- `ip nat inside source list 1 interface Serial0/1/0 overload` → PAT: todas las redes internas salen con la IP de la serial.

## 6. ISP — PPP/CHAP, redes del ISP y DHCP

```cisco
enable
configure terminal
hostname ISP
username R1 password cisco123
interface Serial0/0/0
 description PPP-CHAP hacia R1
 ip address 209.165.200.226 255.255.255.224
 encapsulation ppp
 ppp authentication chap
 clock rate 64000
 no shutdown
 exit
interface FastEthernet1/0
 description Red Web Server
 ip address 209.165.201.1 255.255.255.224
 no shutdown
 exit
interface FastEthernet0/0
 description Clientes Cable
 ip address 192.168.2.1 255.255.255.0
 no shutdown
 exit
interface FastEthernet0/1
 description Clientes DSL
 ip address 192.168.1.1 255.255.255.0
 no shutdown
 exit
ip dhcp excluded-address 192.168.2.1 192.168.2.10
ip dhcp excluded-address 192.168.1.1 192.168.1.10
ip dhcp pool TW-CABLE
 network 192.168.2.0 255.255.255.0
 default-router 192.168.2.1
 dns-server 209.165.201.30
 exit
ip dhcp pool TW-DSL
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1
 dns-server 209.165.201.30
 exit
end
copy running-config startup-config

```

> ⚠️ **Correcciones al enunciado:** escribe `ISP(ISP config)# hostname ISP` (el prompt correcto es `Router(config)#`), y al bloque del ISP le falta `no shutdown`, sin el cual el enlace nunca sube.
>
> El ISP **no necesita rutas** hacia la empresa: gracias al NAT de R1 todo le llega desde 209.165.200.225, que es una red directamente conectada.

## 7. R2 — spoke Frame Relay y RIP

```cisco
enable
configure terminal
hostname R2
interface Serial0/0/0
 ip address 10.1.1.2 255.255.255.248
 encapsulation frame-relay
 frame-relay map ip 10.1.1.1 201 broadcast
 frame-relay map ip 10.1.1.3 201 broadcast
 no shutdown
 exit
interface FastEthernet0/1
 ip address 192.168.10.1 255.255.255.0
 no shutdown
 exit
router rip
 version 2
 no auto-summary
 network 10.0.0.0
 network 192.168.10.0
 passive-interface FastEthernet0/1
 exit
end
copy running-config startup-config

```

## 8. R3 — spoke Frame Relay y RIP

```cisco
enable
configure terminal
hostname R3
interface Serial0/0/0
 ip address 10.1.1.3 255.255.255.248
 encapsulation frame-relay
 frame-relay map ip 10.1.1.1 301 broadcast
 frame-relay map ip 10.1.1.2 301 broadcast
 no shutdown
 exit
interface FastEthernet0/1
 ip address 192.168.30.1 255.255.255.0
 no shutdown
 exit
router rip
 version 2
 no auto-summary
 network 10.0.0.0
 network 192.168.30.0
 passive-interface FastEthernet0/1
 exit
end
copy running-config startup-config

```

**Switches:** no se configuran (todo en VLAN 1).

## 9. Direccionamiento de los equipos finales

Clic → `Desktop` → `IP Configuration`.

| Equipo | Modo | IP | Máscara | Gateway | DNS Server |
|---|---|---|---|---|---|
| PC1 | Static | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 | 209.165.201.30 |
| PC3 | Static | 192.168.30.10 | 255.255.255.0 | 192.168.30.1 | 209.165.201.30 |
| Intranet | Static | 192.168.20.254 | 255.255.255.0 | 192.168.20.1 | 209.165.201.30 |
| Web Server | Static | 209.165.201.30 | 255.255.255.224 | 209.165.201.1 | 209.165.201.30 |
| Outside Host | Static | 209.165.201.2 | 255.255.255.224 | 209.165.201.1 | 209.165.201.30 |
| TW-Cable | **DHCP** | (automática 192.168.2.x) | | | |
| TW-DSL | **DHCP** | (automática 192.168.1.x) | | | |

> TW-Cable y TW-DSL: si se quedan en `169.254.x.x`, pulsa `Static` y otra vez `DHCP` cuando ya esté configurado el ISP.

---

# C. SERVICIOS

## 10. Web Server, DNS e Intranet

**Web Server** → `Services`:
- **HTTP** → On → `index.html` → **(edit)** → borra todo y pega → **Save**:

```html
<html>
<center><font size='+2' color='blue'>Web Server del ISP</font></center>
<hr>Challenge PPP w/CHAP — Infraestructura 2
<p>Si ves esta pagina, tu red llega al ISP a traves de PPP con CHAP.
</html>
```

- **DNS** → On → agrega:

| Name | Type | Address | |
|---|---|---|---|
| `www.isp.com` | A Record | `209.165.201.30` | **Add** |
| `intranet.com` | A Record | `192.168.20.254` | **Add** |

**Intranet** → `Services` → **HTTP** → On (con la página por defecto basta).

---

# D. CIERRE

## 11. Pruebas

| # | Dónde | Qué hacer | Qué debe pasar |
|---|---|---|---|
| 1 | R1 | `show interfaces serial0/1/0` | `up, line protocol is up` · `Encapsulation PPP` · `LCP Open` |
| 2 | R1 | `ping 209.165.200.226` | `!!!!!` (PPP+CHAP funciona) |
| 3 | R1 | `show frame-relay map` | 2 mapas (102 y 103) `active` |
| 4 | R2 y R3 | `show ip route` | Rutas `R` a las otras LAN + `R* 0.0.0.0/0` |
| 5 | PC1 | `ping 192.168.30.10` y `ping 192.168.20.254` | Responden (Frame Relay + RIP) |
| 6 | PC1 | `Web Browser` → `http://www.isp.com` | Abre la página azul del ISP |
| 7 | PC3 | `Web Browser` → `http://209.165.201.30` | Abre la página del ISP |
| 8 | Intranet | `Command Prompt` → `ping 209.165.201.30` | Responde |
| 9 | R1 | `show ip nat translations` | Traducciones 192.168.x.x → 209.165.200.225 |
| 10 | TW-Cable y TW-DSL | `ipconfig` y `Web Browser` → `http://www.isp.com` | IP por DHCP y abre la página |
| 11 | Outside Host | `Web Browser` → `http://209.165.201.30` | Abre la página |
| 12 | **Prueba CHAP** | En ISP: `conf t` → `username R1 password malo` → en Se0/0/0 `shutdown` / `no shutdown` | El enlace queda `down` (CHAP rechaza). Devuélvelo con `username R1 password cisco123` y `shutdown`/`no shutdown` |

Captura las 12 y pégalas en [[entregable]].

### Comandos de verificación útiles

```cisco
show interfaces serial0/1/0     ! Encapsulation PPP, LCP Open, Open: IPCP
debug ppp authentication        ! ver el CHALLENGE / RESPONSE / SUCCESS al hacer shut/no shut
undebug all
show frame-relay map
show frame-relay pvc
show ip route
show ip protocols
show ip nat translations
show ip nat statistics
show ip dhcp binding            ! en el ISP
```

## 12. Errores frecuentes

| Problema | Arreglo |
|---|---|
| R1–ISP en rojo | Falta `no shutdown`, o falta `clock rate 64000` en el lado DCE (el reloj va donde está el extremo DCE del cable) |
| Serial `up/down` entre R1 e ISP | Contraseña CHAP distinta, o el `hostname` no coincide con el `username` del otro lado |
| No aparece `Serial0/1/0` en R1 | Falta el WIC-2T en la ranura `HWIC 1` |
| No aparece `FastEthernet1/0` en ISP | Falta el NM-1FE-TX (router apagado para ponerlo) |
| R2 no ve la LAN de R3 | Falta `no ip split-horizon` en R1 S0/0/0, o el mapa `10.1.1.3 201` en R2 |
| `no ip split-horizon` da `% Invalid input` | Esa versión de PT no lo trae; si R2 no ve 192.168.30.0, pon en R2 `ip route 192.168.30.0 255.255.255.0 10.1.1.3` y en R3 `ip route 192.168.10.0 255.255.255.0 10.1.1.2` |
| Frame Relay en rojo | DLCIs de la nube mal escritos o sin la pareja en `Frame Relay` |
| PCs no llegan al Web Server pero R1 sí | Falta NAT (`ip nat inside/outside`) o la ACL 1 |
| R2/R3 sin `R* 0.0.0.0` | Falta `default-information originate` en R1 |
| TW-Cable/TW-DSL sin IP | Revisa los dos mapeos de la nube Internet (Cable y DSL) y vuelve a pedir DHCP |
| La nube no trae FastEthernet9 | `Physical` → quita el módulo Serial3 → pon **PT-CLOUD-NM-1CFE** en ese hueco y usa el puerto nuevo |

## 13. Teoría breve

**PPP** (capa 2) establece enlaces punto a punto y transporta varios protocolos de capa 3. Usa **LCP** para abrir, configurar y probar el enlace, y un **NCP** por cada protocolo de red (IPCP para IPv4). A diferencia de HDLC de Cisco, es estándar (RFC 1661) y admite autenticación.

**CHAP** (RFC 1994) autentica con un *3-way handshake*: el autenticador envía un **desafío** (ID + número aleatorio), el otro responde con un **hash MD5** de ID + contraseña + desafío, y el autenticador compara con su propio cálculo → **success/failure**. La contraseña nunca viaja por el cable, y el desafío se repite periódicamente durante la sesión. **PAP**, en cambio, envía usuario y contraseña en texto plano y solo al inicio.

## 14. Glosario

| Término | Definición |
|---|---|
| **PPP** | Point-to-Point Protocol — encapsulación WAN de capa 2 para enlaces punto a punto |
| **LCP** | Link Control Protocol — establece, prueba y termina el enlace PPP |
| **NCP / IPCP** | Network Control Protocol — negocia el protocolo de capa 3 (IPCP para IPv4) |
| **CHAP** | Challenge Handshake Authentication Protocol — autenticación por desafío y hash MD5 |
| **PAP** | Password Authentication Protocol — autenticación en texto plano (menos segura) |
| **MD5** | Función hash usada por CHAP para no enviar la contraseña |
| **DCE / DTE** | Extremo que da el reloj (DCE, `clock rate`) / extremo que lo recibe (DTE) |
| **Frame Relay** | WAN de capa 2 con circuitos virtuales permanentes (PVC) |
| **DLCI** | Identificador local del PVC en Frame Relay |
| **Split horizon** | Regla anti-bucles: no anunciar una ruta por la interfaz por la que se aprendió |
| **PAT / NAT overload** | Muchas IP privadas comparten una IP pública distinguiéndose por puerto |

## 15. Checklist

- [ ] Módulos WIC-2T (R1 ×2, R2, R3, ISP) y NM-1FE-TX (ISP) instalados
- [ ] Nube Frame Relay con 4 DLCI y 2 conexiones
- [ ] Nube Internet con mapeos Cable y DSL
- [ ] R1, ISP, R2, R3 configurados y guardados
- [ ] Enlace R1–ISP `up/up` con `Encapsulation PPP` y `LCP Open`
- [ ] Prueba negativa de CHAP hecha (clave errónea → enlace caído)
- [ ] PC1, PC3, Intranet, TW-Cable, TW-DSL y Outside Host abren `www.isp.com`
- [ ] `show ip nat translations` con entradas
- [ ] Capturas pegadas en [[entregable]]
- [ ] Archivo guardado como `Challenge-PPP-CHAP.pka`
