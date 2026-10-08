# Challenge PPP w/CHAP — Entrega

**Estudiante:** Harold Camargo · **Asignatura:** Infraestructura Tecnológica 2 · **Fecha:** 02/10/2026

**Resumen:** R1 se conecta al ISP por un enlace serial con **PPP y autenticación CHAP** (usuario/clave compartida `cisco123`). R1, R2 y R3 se unen por **Frame Relay** (10.1.1.0/29) con **RIPv2**. R1 hace **NAT con sobrecarga** hacia el ISP, y el ISP entrega IP por **DHCP** a los clientes de cable y DSL. El Web Server (209.165.201.30) también hace de **DNS** (`www.isp.com`).

---

### 1. Enlace PPP con CHAP y Frame Relay en R1

![[Captura de pantalla 2026-10-02 a la(s) 11.03.06 a.m..png]]

`Serial0/1/0` está *up/up* con **Encapsulation PPP**, **LCP Open** e **IPCP** abierto, así que CHAP autenticó bien. El ping al ISP (209.165.200.226) responde al 100 %. `show frame-relay map` muestra los DLCI 102 (R2) y 103 (R3) activos.

### 2. Tablas de enrutamiento de R2 y R3 (RIPv2)

![[Captura de pantalla 2026-10-02 a la(s) 11.04.17 a.m..png]]

Cada router aprende por RIP (`R`) las LAN de los otros y la ruta por defecto `R* 0.0.0.0/0` vía R1 (10.1.1.1), que R1 propaga con `default-information originate`.

### 3. Conectividad interna por Frame Relay

![[Captura de pantalla 2026-10-02 a la(s) 11.05.29 a.m..png]]

PC1 (192.168.10.10) llega a PC3 (192.168.30.10) y al servidor Intranet (192.168.20.254). El primer paquete se pierde mientras se resuelve ARP, que es normal.

### 4. PC1 accede al Web Server por nombre

![[Captura de pantalla 2026-10-02 a la(s) 11.05.50 a.m..png]]

PC1 abre `http://www.isp.com`, lo que valida a la vez el DNS, el NAT y el enlace PPP.

### 5. PC3 accede al Web Server por IP

![[Captura de pantalla 2026-10-02 a la(s) 11.06.14 a.m..png]]

PC3 abre `http://209.165.201.30` desde la red de R3, pasando por Frame Relay y después por PPP.

### 6. Servidor Intranet llega al Web Server

![[Captura de pantalla 2026-10-02 a la(s) 11.07.03 a.m..png]]

Ping exitoso (4/4) desde la red 192.168.20.0/24 de R1 hacia el Web Server del ISP.

### 7. Traducciones NAT en R1

![[Captura de pantalla 2026-10-02 a la(s) 11.07.43 a.m..png]]

`show ip nat translations` muestra que las IP privadas (192.168.10.10, 192.168.20.254, 192.168.30.10) salen con la IP pública **209.165.200.225**, diferenciadas por puerto (PAT). Se ve el tráfico ICMP, el DNS (UDP 53) y el HTTP (TCP 80).

### 8. Clientes de cable y DSL del ISP

![[Captura de pantalla 2026-10-02 a la(s) 11.08.31 a.m..png]]

TW-Cable y TW-DSL obtienen IP por DHCP del ISP a través de la nube Internet (cable módem y módem DSL) y abren `http://www.isp.com`.

### 9. Outside Host

![[Captura de pantalla 2026-10-02 a la(s) 11.09.00 a.m..png]]

El host de la red del ISP (209.165.201.0/27) abre el Web Server directamente.

---

**Conclusión:** todas las redes acceden al Web Server del ISP, y quedan validados PPP con CHAP, Frame Relay, RIPv2, NAT/PAT, DHCP, DNS y HTTP.
