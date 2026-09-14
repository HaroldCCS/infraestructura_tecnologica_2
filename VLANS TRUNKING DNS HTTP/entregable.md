---
tipo: entregable
practica: "Laboratorio — Creación de VLANs, Trunking, DNS y HTTP"
estudiante: Harold Camargo
archivo: VLANs-Trunking-DNS-HTTP.pkt
fecha: 2026-08-21
---

# Entregable — VLANs, Trunking, DNS y HTTP

**Estudiante:** Harold Camargo · **Archivo adjunto:** `VLANs-Trunking-DNS-HTTP.pkt`

---

## Preguntas

**1. ¿Qué es trunking?**

Es una conexión entre dispositivos que transporta **el tráfico de varias VLAN por un solo cable**. Para no mezclarlas, cada trama se marca con una etiqueta que dice a qué VLAN pertenece (estándar **802.1Q**); el equipo del otro extremo lee la etiqueta y entrega la trama a la VLAN correcta.

Se usa entre switch y switch, o entre switch y router. Sin trunking haría falta un cable por cada VLAN, y en esta práctica es lo que permite que un único enlace `Switch0 Fa0/1 ↔ Router0 Fa0/0` lleve al mismo tiempo la VLAN 1, la 10 y la 20.

**2. ¿Para qué se hacen las VLAN?**

Para **dividir un switch físico en varias redes lógicas independientes**. Los motivos:

- **Rendimiento:** cada VLAN es su propio dominio de difusión, así que el broadcast de una no molesta a las demás.
- **Seguridad:** los equipos de una VLAN no ven el tráfico de otra. Para comunicarlas hay que pasar por un router, y ahí se pueden aplicar controles.
- **Flexibilidad:** el agrupamiento es lógico, no físico. Ingeniería y Ventas pueden estar en el mismo switch, o repartidos por todo el edificio, sin que cambie nada.
- **Ahorro:** se segmenta con los switches que ya se tienen, sin comprar equipos nuevos ni volver a cablear.
- **Administración:** mover a alguien de un departamento a otro es cambiar una línea de configuración, no un cable.

---

## Tabla de direccionamiento completada

*(el enunciado dejaba en blanco las máscaras y toda la columna de gateway)*

| Equipo | Dirección IP | Máscara | Gateway | DNS Server |
|---|---|---|---|---|
| PC0 | 10.0.0.2 | 255.0.0.0 | 10.0.0.1 | 219.17.100.2 |
| PC1 | 20.0.0.2 | 255.0.0.0 | 20.0.0.1 | 219.17.100.2 |
| PC2 | 219.17.100.3 | 255.255.255.0 | 219.17.100.1 | 219.17.100.2 |
| PC3 | 223.8.151.3 | 255.255.255.0 | 223.8.151.1 | 219.17.100.2 |
| Servidor DNS | 219.17.100.2 | 255.255.255.0 | 219.17.100.1 | 219.17.100.2 |
| Servidor WEB | 223.8.151.2 | 255.255.255.0 | 223.8.151.1 | 219.17.100.2 |

**Criterio usado.** Las máscaras salen de la clase de cada red: `10.0.0.0` y `20.0.0.0` son clase A → `255.0.0.0`; `219.17.100.0` y `223.8.151.0` son clase C → `255.255.255.0`. El gateway de cada equipo es la interfaz del router que está en su misma red.

### Interfaces de los routers

| Interfaz | Dirección IP | Función |
|---|---|---|
| Router0 `Fa0/0.1` | 1.0.0.1 /8 | VLAN 1 (nativa) — gateway de Switch0 |
| Router0 `Fa0/0.2` | 10.0.0.1 /8 | VLAN 10 INGENIERIA — gateway de PC0 |
| Router0 `Fa0/0.3` | 20.0.0.1 /8 | VLAN 20 VENTAS — gateway de PC1 |
| Router0 `Se0/0` | 201.100.11.2 /24 | WAN — **DCE**, `clock rate 128000` |
| Router1 `Se0/0` | 201.100.11.1 /24 | WAN — DTE |
| Router1 `Fa0/0` | 219.17.100.1 /24 | LAN del servidor DNS |
| Router1 `Fa0/1` | 223.8.151.1 /24 | LAN del servidor WEB |
| Switch0 `VLAN 1` | 1.0.0.2 /8 | Administración (Telnet) |

---

## Configuración del Router1

El enunciado pedía *"Configurar el Router 1 para que la topología funcione"* sin dar los comandos. Lo que hice:

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

Router1 no lleva `clock rate` porque es el extremo **DTE** del enlace serial; el reloj lo marca Router0.

---

## Servicios configurados

**Servidor WEB** — `Services → HTTP`, encendido, con el `index.html` del enunciado (la página azul de *Teleinformática 3* con los enlaces a `helloworld.html` y `copyrights.html`).

**Servidor DNS** — `Services → DNS`, encendido, con estos registros:

| Name | Type | Address |
|---|---|---|
| `www.teleinfo.com` | A Record | 223.8.151.2 |
| `dns.teleinfo.com` | A Record | 219.17.100.2 |

> **Observación sobre el enunciado.** El documento pide registrar `www.teleinfo.com: 219.17.100.2`, pero esa dirección es la del **propio servidor DNS**, no la del servidor WEB. Con ese registro, abrir `www.teleinfo.com` en el navegador no lleva a la página que el mismo enunciado manda montar. Apunté el registro al servidor WEB (`223.8.151.2`), que es lo que hace funcionar la prueba, y conservé la dirección literal en un segundo registro.

---

## Pruebas realizadas

**Ping de una VLAN a otra.** Desde PC0 (VLAN 10, `10.0.0.2`) a PC1 (VLAN 20, `20.0.0.2`): responde. El tráfico sube etiquetado por el troncal hasta Router0, que lo enruta entre sus subinterfaces `Fa0/0.2` y `Fa0/0.3` y lo devuelve por el mismo cable físico.

**Ping de los PC de Switch0 a los servidores.** PC0 y PC1 alcanzan tanto el servidor DNS (`219.17.100.2`) como el servidor WEB (`223.8.151.2`), atravesando el enlace serial. Es la comprobación de que RIP propagó las seis redes: `show ip route` las muestra en ambos routers, las propias con `C` y las remotas con `R`.

**Resolución por nombre.** `nslookup www.teleinfo.com` desde PC0 devuelve `223.8.151.2`, y el navegador abre la página con sus dos enlaces funcionando. Navegar primero por IP y después por nombre deja claro que el DNS es lo único que separa una cosa de la otra.

**Consola.** Desde `Desktop → Terminal` de PC0, por el cable celeste, se entra a la CLI de Switch0 sin usar la red.

**Telnet.** Desde PC1 (puerto 4, VLAN 20): `telnet 1.0.0.2` → contraseña `cisco` → `enable` → `show vlan`. Funciona porque Switch0 tiene `ip default-gateway 1.0.0.1`; sin esa línea el switch no sabría cómo responderle a un equipo de otra red.

---

## Observaciones sobre el enunciado

1. **El 2620XM no alcanza para Router1.** La tabla de componentes pide 2620XM para los dos routers, pero Router1 necesita **dos** interfaces FastEthernet (una hacia Switch2 y otra hacia Switch1) y ese modelo trae una sola. Usé un **2621XM**, que tiene `Fa0/0` y `Fa0/1` de fábrica. La alternativa es un 2620XM con módulo `NM-2FE2W`.

2. **La tabla de componentes no coincide con la topología.** Declara 2 switches cuando el diagrama muestra 3 (Switch0, Switch1 y Switch2) y 3 cables directos cuando hacen falta 9.

3. **El registro DNS apunta al servidor equivocado** (explicado arriba).

4. **`vlan database` está obsoleto.** El enunciado crea las VLAN con la sintaxis de IOS 12.0. Usé la equivalente desde el modo de configuración global (`vlan 10` → `name INGENIERIA`), que es la vigente y la que aceptan todas las versiones de Packet Tracer.

---

## Conclusiones

El laboratorio encadena tres capas que normalmente se estudian por separado. En capa 2, las VLAN parten un switch en dos redes que no se ven entre sí, y el troncal 802.1Q permite que ese aislamiento sobreviva al salir del switch: un solo cable transporta tres VLAN sin mezclarlas porque cada trama lleva su etiqueta.

En capa 3, las subinterfaces del router hacen el trabajo inverso: vuelven a comunicar lo que las VLAN separaron, y RIP se encarga de que las redes de una sede aparezcan en la tabla de la otra sin escribir una sola ruta a mano.

En capa de aplicación, el DNS es lo que convierte todo eso en algo usable. La prueba más ilustrativa fue abrir la misma página primero por IP y después por nombre: la infraestructura ya funcionaba en los dos casos, pero solo el segundo se parece a cómo la gente usa realmente una red.

Y el detalle que amarra todo es el más pequeño: sin `ip default-gateway` en el switch, el Telnet desde otra VLAN falla. Un switch de capa 2 no enruta, y esa línea es lo único que le permite contestarle a alguien que no está en su propia red.
