---
tipo: entregable
practica: "Práctica 3 — Configuración de routing entre VLAN basado en enlaces troncales"
estudiante: Harold Camargo
archivo: P3-InterVLAN.pkt
fecha: 2026-08-18
---

# Entregable — Práctica 3 (routing entre VLAN / router-on-a-stick)

**Estudiante:** Harold Camargo · **Archivo adjunto:** `P3-InterVLAN.pkt`

---

## Preguntas orientadoras

**1. Which of the following encrypts the traffic on a leased line?**
**D. vpn.** Una VPN cifra el tráfico que viaja por el enlace. SSH cifra únicamente una sesión de administración, no el tráfico de la línea.

**2. What does split horizon prevent?**
**B. routing loops, distance vector.** Split horizon impide que un router anuncie una ruta por la misma interfaz por la que la aprendió; es un mecanismo antibucle de los protocolos de vector distancia como RIP.

**3. Which two protocols can detect native VLAN mismatch errors?**
**A. STP** y **C. Cisco Discovery Protocol.** Ambos generan un mensaje de error cuando los dos extremos de un troncal declaran VLAN nativas distintas.

---

## Parte 2 — Comandos utilizados en los switches

**Crear las VLAN en S1:**

```
S1(config)# vlan 10
S1(config-vlan)# name Students
S1(config-vlan)# vlan 20
S1(config-vlan)# name Faculty
S1(config-vlan)# exit
```

**Configurar como troncales las interfaces hacia R1 y hacia S2:**

```
S1(config)# interface f0/1
S1(config-if)# switchport mode trunk
S1(config-if)# interface f0/5
S1(config-if)# switchport mode trunk
```

**Asignar el puerto de acceso de PC-A a la VLAN 10:**

```
S1(config)# interface f0/6
S1(config-if)# switchport mode access
S1(config-if)# switchport access vlan 10
```

**Verificar que los nombres y números de VLAN de S2 coinciden con los de S1:**

```
S2# show vlan brief
```

**En S2, puerto de acceso de PC-B y troncal hacia S1:**

```
S2(config)# interface f0/18
S2(config-if)# switchport mode access
S2(config-if)# switchport access vlan 20
S2(config-if)# interface f0/1
S2(config-if)# switchport mode trunk
```

---

## Parte 3 — Comandos utilizados en el router

**Crear la subinterfaz de la VLAN 1:**

```
R1(config)# interface g0/1.1
```

**Hacer que opere en la VLAN 1:**

```
R1(config-subif)# encapsulation dot1Q 1 native
```

**Asignarle la dirección IP:**

```
R1(config-subif)# ip address 192.168.1.1 255.255.255.0
```

**VLAN 10 y VLAN 20 (mismo patrón):**

```
R1(config)# interface g0/1.10
R1(config-subif)# encapsulation dot1Q 10
R1(config-subif)# ip address 192.168.10.1 255.255.255.0
R1(config)# interface g0/1.20
R1(config-subif)# encapsulation dot1Q 20
R1(config-subif)# ip address 192.168.20.1 255.255.255.0
```

**Habilitar la interfaz G0/1:**

```
R1(config)# interface g0/1
R1(config-if)# no shutdown
```

---

## Verificación de la conectividad

**Comando para ver la tabla de routing:** `show ip route`

**¿Qué redes se enumeran?** Cuatro, todas **directamente conectadas** (no hay rutas estáticas ni protocolo de enrutamiento, porque el router es el gateway de todas):

| Red | Interfaz |
|---|---|
| `192.168.1.0/24` | G0/1.1 |
| `192.168.10.0/24` | G0/1.10 |
| `192.168.20.0/24` | G0/1.20 |
| `209.165.200.224/27` | Loopback0 |

| Ping | Resultado |
|---|---|
| PC-A → gateway de la VLAN 10 (`192.168.10.1`) | **Sí** |
| PC-A → PC-B (`192.168.20.3`) | **Sí** |
| PC-A → Lo0 (`209.165.200.225`) | **Sí** |
| PC-A → S2 (`192.168.1.12`) | **Sí** |

Los cuatro responden. El de PC-A a PC-B es el resultado importante: son dos VLAN distintas comunicándose a través de las subinterfaces del router.

---

## Reflexión

**¿Cuáles son las ventajas del routing entre VLAN basado en enlaces troncales?**

> *Nota: el enunciado compara "routing basado en enlaces troncales" contra "router-on-a-stick", pero son el mismo método. La comparación de la guía original de Cisco es contra el **routing entre VLAN tradicional (legacy)**, que usa una interfaz física del router por cada VLAN. Respondo sobre esa comparación.*

- **Usa una sola interfaz física.** El método tradicional necesita un puerto del router y un puerto del switch por cada VLAN; aquí basta un cable para todas.
- **Es más barato.** No hay que comprar routers con muchas interfaces ni módulos adicionales.
- **Escala mejor.** Agregar una VLAN es crear una subinterfaz por software, no tender un cable nuevo ni quedarse sin puertos.
- **Simplifica el cableado.** Un enlace troncal en vez de N enlaces reduce puntos de falla y desorden físico.
- **Administración centralizada.** Todo el direccionamiento entre VLAN queda en un solo lugar de la configuración del router.

**Desventaja que conviene mencionar:** ese único enlace es un cuello de botella y un punto único de falla — todo el tráfico entre VLAN pasa por él. Cuando el volumen lo justifica, se reemplaza por un switch multicapa con SVI, que enruta en hardware.

---

## Observaciones sobre el enunciado

1. **Tabla de puertos incompleta.** La tabla *"Especificaciones de la asignación de puertos de switch"* del documento solo trae la fila de `S1 F0/1`. Las cuatro filas restantes se reconstruyeron a partir de la topología y del Apéndice A del mismo documento: `S1 F0/5` troncal, `S1 F0/6` VLAN 10 (`192.168.10.0/24`), `S2 F0/1` troncal y `S2 F0/18` VLAN 20 (`192.168.20.0/24`).

2. **Falta `native` en el Apéndice A.** El apéndice indica `encapsulation dot1Q 1` para la subinterfaz de la VLAN 1. Como la VLAN 1 es la nativa del troncal, su tráfico viaja **sin etiqueta**, así que la subinterfaz debe declararse `encapsulation dot1Q 1 native`. Sin esa palabra, el router descarta las tramas sin etiquetar y los switches quedan inalcanzables desde las otras VLAN.

---

## Conclusiones

Router-on-a-stick resuelve con software un problema que antes exigía hardware: en lugar de una interfaz física por VLAN, se crean interfaces lógicas sobre un mismo cable, cada una asociada a una etiqueta 802.1Q. La subinterfaz es simultáneamente el gateway de su VLAN y el punto donde el router lee la etiqueta para saber de dónde viene el tráfico y hacia dónde devolverlo.

La práctica cierra lo que quedó abierto en la Práctica 2: allí se comprobó que dos VLAN no se comunican por más troncales que se configuren, porque el problema no era de capa 2. Aquí, con un solo router y tres subinterfaces, la VLAN 10 y la VLAN 20 quedaron comunicadas sin tocar la configuración de los switches más allá de los troncales.

El detalle que más fácil se pasa por alto es el `no shutdown` de la interfaz física: las subinterfaces heredan su estado, y tres subinterfaces perfectamente configuradas quedan inservibles si `G0/1` sigue apagada.
