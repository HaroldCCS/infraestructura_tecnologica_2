# RÁPIDO — Challenge PPP w/CHAP

Sigue de arriba abajo. Sin saltarte pasos. Cada bloque de comandos se pega **completo, de una sola vez**.

---

## 1. Arrastra esto al lienzo

| Cantidad | Qué                                      | Modelo             | Dónde                           |
| -------- | ---------------------------------------- | ------------------ | ------------------------------- |
| 4        | R1, R2, R3, ISP                          | **2811**           | Network Devices → Routers       |
| 4        | S1, S2, S3, ISP Switch                   | **2960**           | Network Devices → Switches      |
| 2        | Frame Relay, Internet                    | **Cloud-PT**       | Network Devices → WAN Emulation |
| 1        | Cable Modem                              | **Cable Modem-PT** | Network Devices → WAN Emulation |
| 1        | DSL Modem                                | **DSL Modem-PT**   | Network Devices → WAN Emulation |
| 5        | PC1, PC3, Outside Host, TW-Cable, TW-DSL | **PC**             | End Devices                     |
| 2        | Intranet, Web Server                     | **Server**         | End Devices                     |

Renombra cada equipo (clic → `Config` → `Display Name`) como en el dibujo.

### Módulos (router APAGADO → arrastrar módulo → ENCENDER)

| Router | Módulo → ranura | Para qué |
|---|---|---|
| R1 | **WIC-2T** → ranura `HWIC 0` y **WIC-2T** → ranura `HWIC 1` | `S0/0/0` (Frame Relay) y `S0/1/0` (PPP) |
| R2 | **WIC-2T** → `HWIC 0` | `S0/0/0` |
| R3 | **WIC-2T** → `HWIC 0` | `S0/0/0` |
| ISP | **WIC-2T** → `HWIC 0` y **NM-1FE-TX** → ranura grande `NM 1` | `S0/0/0` (PPP) y `Fa1/0` (red del Web Server) |

> ⚠️ El dibujo dice `Eth0/1/0` en el ISP. En un 2811 ese puerto no existe tal cual: usamos **Fa1/0** (módulo NM-1FE-TX). Hace lo mismo.

### Nube "Internet" — puertos Ethernet

La Cloud-PT de PT 8 ya trae un segundo puerto Ethernet: **FastEthernet9**. Ese es el que se usa para el DSL (no hay que tocar módulos).

---

## 2. Cablea

| Desde       | Puerto       | Hacia        | Puerto            | Cable                                |
| ----------- | ------------ | ------------ | ----------------- | ------------------------------------ |
| **ISP**     | **S0/0/0**   | R1           | **S0/1/0**        | **Serial DCE** (clic primero en ISP) |
| R1          | **S0/0/0**   | Frame Relay  | **Serial1**       | Serial DTE                           |
| R2          | **S0/0/0**   | Frame Relay  | **Serial0**       | Serial DTE                           |
| R3          | **S0/0/0**   | Frame Relay  | **Serial2**       | Serial DTE                           |
| R1          | **Fa0/1**    | S3           | **Fa0/1**         | Straight-Through                     |
| S3          | **Fa0/2**    | Intranet     | **Fa0**           | Straight-Through                     |
| R2          | **Fa0/1**    | S1           | **Fa0/1**         | Straight-Through                     |
| S1          | **Fa0/2**    | PC1          | **Fa0**           | Straight-Through                     |
| R3          | **Fa0/1**    | S2           | **Fa0/1**         | Straight-Through                     |
| S2          | **Fa0/2**    | PC3          | **Fa0**           | Straight-Through                     |
| ISP         | **Fa1/0**    | ISP Switch   | **Fa0/1**         | Straight-Through                     |
| ISP Switch  | **Fa0/2**    | Web Server   | **Fa0**           | Straight-Through                     |
| ISP Switch  | **Fa0/3**    | Outside Host | **Fa0**           | Straight-Through                     |
| ISP         | **Fa0/0**    | Internet     | **Ethernet6**     | Straight-Through                     |
| ISP         | **Fa0/1**    | Internet     | **FastEthernet9** | Straight-Through                     |
| Internet    | **Coaxial7** | Cable Modem  | **Port 0**        | **Coaxial**                          |
| Cable Modem | **Port 1**   | TW-Cable     | **Fa0**           | Straight-Through                     |
| Internet    | **Modem4**   | DSL Modem    | **Port 0**        | **Phone**                            |
| DSL Modem   | **Port 1**   | TW-DSL       | **Fa0**           | Straight-Through                     |

---

## 3. Nube Frame Relay (GUI, no CLI)

Clic en la nube **Frame Relay** → `Config`:

| Panel izquierdo | DLCI | Name | → |
|---|---|---|---|
| **Serial0** (R2) | `201` | `R2-R1` | **Add** |
| **Serial1** (R1) | `102` | `R1-R2` | **Add** |
| **Serial1** (R1) | `103` | `R1-R3` | **Add** |
| **Serial2** (R3) | `301` | `R3-R1` | **Add** |

Luego `Connections` → **Frame Relay** → agrega estas 2 parejas:

| Izquierda | | Derecha | |
|---|---|---|---|
| Serial1 · `R1-R2` | ↔ | Serial0 · `R2-R1` | **Add** |
| Serial1 · `R1-R3` | ↔ | Serial2 · `R3-R1` | **Add** |

## 4. Nube Internet (GUI)

Clic en la nube **Internet** → `Config` → en la lista de la izquierda (sección **INTERFACE**, baja con el scroll):

1. **Ethernet6** → en *Provider Network* elige **Cable**.
2. El otro puerto Ethernet (el que va a ISP Fa0/1, p. ej. **FastEthernet9**) → *Provider Network* **DSL**.

> Sin este paso, Ethernet6 **no aparece** en el desplegable de Cable/DSL.

Luego `Connections`:

- **Cable** → `Coaxial7` ↔ `Ethernet6` → **Add**
- **DSL** → `Modem4` ↔ `FastEthernet9` → **Add**

---

## 5. R1 — pegar en el CLI

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

> 💡 Si sale `% Invalid input` en `no ip split-horizon`, ignóralo: no rompe nada (ver "Si algo falla").

## 6. ISP — pegar en el CLI

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

## 7. R2 — pegar en el CLI

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

## 8. R3 — pegar en el CLI

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

**Los 4 switches NO se configuran.**

---

## 9. IPs de PCs y servidores

Clic → `Desktop` → `IP Configuration`.

| Equipo       | Modo     | IP                       | Máscara         | Gateway       | DNS Server     |
| ------------ | -------- | ------------------------ | --------------- | ------------- | -------------- |
| PC1          | Static   | 192.168.10.10            | 255.255.255.0   | 192.168.10.1  | 209.165.201.30 |
| PC3          | Static   | 192.168.30.10            | 255.255.255.0   | 192.168.30.1  | 209.165.201.30 |
| Intranet     | Static   | 192.168.20.254           | 255.255.255.0   | 192.168.20.1  | 209.165.201.30 |
| Web Server   | Static   | 209.165.201.30           | 255.255.255.224 | 209.165.201.1 | 209.165.201.30 |
| Outside Host | Static   | 209.165.201.2            | 255.255.255.224 | 209.165.201.1 | 209.165.201.30 |
| TW-Cable     | **DHCP** | (automática 192.168.2.x) |                 |               |                |
| TW-DSL       | **DHCP** | (automática 192.168.1.x) |                 |               |                |

> TW-Cable y TW-DSL: si se quedan en `169.254.x.x`, pulsa `Static` y otra vez `DHCP` cuando ya esté configurado el ISP.

## 10. Servicios

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

| Name           | Type     | Address          |         |
| -------------- | -------- | ---------------- | ------- |
| `www.isp.com`  | A Record | `209.165.201.30` | **Add** |
| `intranet.com` | A Record | `192.168.20.254` | **Add** |

**Intranet** → `Services` → **HTTP** → On (con la página por defecto basta).

---

## 11. Pruebas + capturas

| #   | Dónde             | Qué hacer                                                                              | Qué debe pasar                                                                                                   |
| --- | ----------------- | -------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| 1   | R1                | `show interfaces serial0/1/0`                                                          | `up, line protocol is up` · `Encapsulation PPP` · `LCP Open`                                                     |
| 2   | R1                | `ping 209.165.200.226`                                                                 | `!!!!!` (PPP+CHAP funciona)                                                                                      |
| 3   | R1                | `show frame-relay map`                                                                 | 2 mapas (102 y 103) `active`                                                                                     |
| 4   | R2 y R3           | `show ip route`                                                                        | Rutas `R` a las otras LAN + `R* 0.0.0.0/0`                                                                       |
| 5   | PC1               | `ping 192.168.30.10` y `ping 192.168.20.254`                                           | Responden (Frame Relay + RIP)                                                                                    |
| 6   | PC1               | `Web Browser` → `http://www.isp.com`                                                   | Abre la página azul del ISP                                                                                      |
| 7   | PC3               | `Web Browser` → `http://209.165.201.30`                                                | Abre la página del ISP                                                                                           |
| 8   | Intranet          | `Command Prompt` → `ping 209.165.201.30`                                               | Responde                                                                                                         |
| 9   | R1                | `show ip nat translations`                                                             | Traducciones 192.168.x.x → 209.165.200.225                                                                       |
| 10  | TW-Cable y TW-DSL | `ipconfig` y `Web Browser` → `http://www.isp.com`                                      | IP por DHCP y abre la página                                                                                     |
| 11  | Outside Host      | `Web Browser` → `http://209.165.201.30`                                                | Abre la página                                                                                                   |
| 12  | **Prueba CHAP**   | En ISP: `conf t` → `username R1 password malo` → en Se0/0/0 `shutdown` / `no shutdown` | El enlace queda `down` (CHAP rechaza). Devuélvelo con `username R1 password cisco123` y `shutdown`/`no shutdown` |

Captura las 12. Guarda el archivo como `Challenge-PPP-CHAP.pka` (o `.pkt`).

---

## Si algo falla

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
