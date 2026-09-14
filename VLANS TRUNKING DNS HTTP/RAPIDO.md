# RÁPIDO — VLANs, Trunking, DNS y HTTP

Sigue de arriba abajo. Sin saltarte pasos.

---

## 1. Arrastra esto al lienzo

| Cantidad | Qué | Modelo | Dónde |
|---|---|---|---|
| 1 | Switch0 | **2950T-24** | Network Devices → Switches |
| 2 | Switch1, Switch2 | **Switch-PT** (o **2960**) | Network Devices → Switches |
| 1 | Router0 | **2620XM** | Network Devices → Routers |
| 1 | Router1 | **2621XM** | Network Devices → Routers |
| 4 | PC0, PC1, PC2, PC3 | **PC** | End Devices |
| 2 | Servidor DNS, Servidor WEB | **Server** | End Devices |

**A los 2 routers ponles el módulo serial:** clic → `Physical` → **apagar** el interruptor → arrastrar **`WIC-1T`** a una ranura → **encender**.

---

## 2. Cablea

Todo con **Copper Straight-Through** menos donde diga otra cosa.

| Desde       | Puerto    | Hacia        | Puerto      |                                           |
| ----------- | --------- | ------------ | ----------- | ----------------------------------------- |
| PC0         | Fa0       | Switch0      | **Fa0/2**   |                                           |
| PC1         | Fa0       | Switch0      | **Fa0/4**   |                                           |
| Switch0     | **Fa0/1** | Router0      | Fa0/0       |                                           |
| **Router0** | Se0/0     | Router1      | Se0/0       | ← **Serial DCE**, clic primero en Router0 |
| Router1     | Fa0/0     | Switch2      | **Fa1/1**   |                                           |
| Switch2     | **Fa2/1** | Servidor DNS | Fa0         |                                           |
| Switch2     | **Fa3/1** | PC2          | Fa0         |                                           |
| Router1     | Fa0/1     | Switch1      | **Fa1/1**   |                                           |
| Switch1     | **Fa2/1** | Servidor WEB | Fa0         |                                           |
| Switch1     | **Fa3/1** | PC3          | Fa0         |                                           |
| PC0         | RS 232    | Switch0      | **Console** | ← cable **Console** (celeste)             |

> ⚠️ **Switch1 y Switch2 son Switch-PT (genéricos) y sus puertos se llaman `FastEthernet1/1`, `2/1`, `3/1`…**, no `Fa0/1`. Es solo un nombre distinto: se cablea igual y no se configuran.
>
> Si prefieres que se llamen `Fa0/1`, `Fa0/2`, `Fa0/3` como el resto, borra los dos Switch-PT y pon **2960** en su lugar. Da exactamente lo mismo para esta práctica.

---

## 3. IPs de PCs y servidores

Clic en el equipo → `Desktop` → `IP Configuration` → **Static**. Llena los 4 campos.

| Equipo       | IP           | Máscara       | Gateway      | DNS Server   |
| ------------ | ------------ | ------------- | ------------ | ------------ |
| PC0          | 10.0.0.2     | 255.0.0.0     | 10.0.0.1     | 219.17.100.2 |
| PC1          | 20.0.0.2     | 255.0.0.0     | 20.0.0.1     | 219.17.100.2 |
| PC2          | 219.17.100.3 | 255.255.255.0 | 219.17.100.1 | 219.17.100.2 |
| PC3          | 223.8.151.3  | 255.255.255.0 | 223.8.151.1  | 219.17.100.2 |
| Servidor DNS | 219.17.100.2 | 255.255.255.0 | 219.17.100.1 | 219.17.100.2 |
| Servidor WEB | 223.8.151.2  | 255.255.255.0 | 223.8.151.1  | 219.17.100.2 |

---

## 4. Switch0 — pegar en el CLI

```cisco
enable
configure terminal
hostname Sw1
enable password cisco
line vty 0 4
 password cisco
 login
 exit
line console 0
 password cisco
 login
 exit
vlan 10
 name INGENIERIA
 exit
vlan 20
 name VENTAS
 exit
interface vlan 1
 ip address 1.0.0.2 255.0.0.0
 no shutdown
 exit
ip default-gateway 1.0.0.1
interface fastEthernet 0/2
 switchport mode access
 switchport access vlan 10
 exit
interface fastEthernet 0/4
 switchport mode access
 switchport access vlan 20
 exit
interface fastEthernet 0/1
 switchport mode trunk
 exit
end
copy running-config startup-config
```

---

## 5. Router0 — pegar en el CLI

```cisco
enable
configure terminal
hostname Router0
enable password cisco
line vty 0 4
 password cisco
 login
 exit
interface fastEthernet 0/0
 no shutdown
 exit
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
interface serial 0/0
 ip address 201.100.11.2 255.255.255.0
 clock rate 128000
 no shutdown
 exit
router rip
 version 2
 no auto-summary
 network 1.0.0.0
 network 10.0.0.0
 network 20.0.0.0
 network 201.100.11.0
 exit
end
copy running-config startup-config
```

---

## 6. Router1 — pegar en el CLI

```cisco
enable
configure terminal
hostname Router1
enable password cisco
line vty 0 4
 password cisco
 login
 exit
interface fastEthernet 0/0
 ip address 219.17.100.1 255.255.255.0
 no shutdown
 exit
interface fastEthernet 0/1
 ip address 223.8.151.1 255.255.255.0
 no shutdown
 exit
interface serial 0/0
 ip address 201.100.11.1 255.255.255.0
 no shutdown
 exit
router rip
 version 2
 no auto-summary
 network 201.100.11.0
 network 219.17.100.0
 network 223.8.151.0
 exit
end
copy running-config startup-config
```

**Switch1 y Switch2 NO se configuran.**

### Antes de seguir

Desde PC0 → `Desktop` → `Command Prompt`:

```
ping 20.0.0.2
ping 223.8.151.2
```

Si no responden, no sigas: revisa `show ip route` en los dos routers.

---

## 7. Servidor WEB

Clic en **Servidor WEB** → `Services` → **HTTP** → **On** → busca `index.html` → **(edit)** → borra todo y pega:

```html
<html>
<center><font size='+2' color='blue'>Teleinformática 3</font></center>
<hr>Servidor Teleinformática 3. Bienvenido
<p>Enlaces:
<br><a href='helloworld.html'>Página pequeña.</a>
<br><a href='copyrights.html'>Derechos de Autor</a>
</html>
```

→ **Save**.

---

## 8. Servidor DNS

Clic en **Servidor DNS** → `Services` → **DNS** → **On** → agrega:

| Name               | Type     | Address        |           |
| ------------------ | -------- | -------------- | --------- |
| `www.teleinfo.com` | A Record | `223.8.151.2`  | → **Add** |
| `dns.teleinfo.com` | A Record | `219.17.100.2` | → **Add** |

---

## 9. Pruebas + capturas

| # | Qué hacer | Qué debe pasar |
|---|---|---|
| 1 | PC0 → `ping 20.0.0.2` | Responde (VLAN 10 → VLAN 20) |
| 2 | PC0 → `ping 219.17.100.2` y `ping 223.8.151.2` | Responden |
| 3 | PC1 → `ping 219.17.100.2` y `ping 223.8.151.2` | Responden |
| 4 | PC0 → `nslookup www.teleinfo.com` | Devuelve `223.8.151.2` |
| 5 | PC0 → `Desktop` → `Web Browser` → `http://www.teleinfo.com` | Abre la página azul, y sus 2 enlaces funcionan |
| 6 | PC0 → `Desktop` → `Terminal` → OK | Entra a la consola de Sw1 (pass `cisco`) |
| 7 | PC1 → `telnet 1.0.0.2` → `cisco` → `enable` → `cisco` → `show vlan` | Muestra las VLAN |
| 8 | Router0 y Router1 → `show ip route` | Salen las 6 redes (C y R) |
| 9 | Switch0 → `show vlan brief` y `show interfaces trunk` | VLAN 10 en Fa0/2, VLAN 20 en Fa0/4, trunk en Fa0/1 |

Captura las 9. Guarda el archivo como `VLANs-Trunking-DNS-HTTP.pkt`.

---

## 10. Las 2 preguntas

**¿Qué es trunking?**
Una conexión que lleva varias VLAN por un solo cable. Cada trama va marcada con una etiqueta 802.1Q que dice a qué VLAN pertenece, y el equipo del otro lado la lee y la entrega donde toca. Aquí el cable Switch0 Fa0/1 ↔ Router0 Fa0/0 transporta las VLAN 1, 10 y 20 al mismo tiempo.

**¿Para qué se hacen las VLAN?**
Para dividir un switch físico en varias redes lógicas separadas. Sirve para: reducir el broadcast (cada VLAN es su propio dominio), aislar por seguridad (una VLAN no ve el tráfico de otra), agrupar por área en vez de por ubicación física, y ahorrar equipos porque no hay que comprar un switch por red.

---

## Si algo falla

| Problema | Arreglo |
|---|---|
| Serial en rojo o `up/down` | Falta `clock rate 128000` en Router0 |
| No aparece `Serial0/0` | Falta el módulo WIC-1T (router apagado para ponerlo) |
| Las subinterfaces no levantan | Falta `no shutdown` en `fa0/0` (la física) |
| Un router no ve las redes del otro | Falta un `network` en RIP |
| Ping por IP sí, por nombre no | A los PC les falta el campo **DNS Server** |
| Telnet no conecta | Falta `ip default-gateway 1.0.0.1` en Switch0 |
| Telnet conecta pero `enable` rebota | Falta `enable password cisco` |
| `% Invalid input` en `encapsulation` | Lo escribiste en `fa0/0` en vez de `fa0/0.2` |
| Se pierde todo al reiniciar | Falta `copy running-config startup-config` |
