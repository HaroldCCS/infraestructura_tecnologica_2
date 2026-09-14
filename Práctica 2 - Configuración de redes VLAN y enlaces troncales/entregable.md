---
tipo: entregable
practica: "Práctica 2 — Configuración de redes VLAN y enlaces troncales"
estudiante: Harold Camargo
archivo: P2-VLAN-Trunking.pkt
fecha: 2026-08-18
---

# Entregable — Práctica 2 (VLAN y enlaces troncales)

**Estudiante:** Harold Camargo · **Archivo adjunto:** `P2-VLAN-Trunking.pkt`

---

## Preguntas orientadoras

**1. ¿Qué comando puede ingresar para determinar si un puerto está funcionando en modo access?**
`show interfaces switchport` (opción **`sh int sw`**). Muestra el `Administrative Mode` y el `Operational Mode` de cada puerto.

**2. Según la salida de `show vlan`, ¿cuántas VLAN configuró el administrador de TI?**
**3**: la 10 (Student), la 20 (Faculty) y la 99 (Management). La VLAN 1 y las 1002-1005 vienen de fábrica.

**3. ¿Cuántas interfaces tiene asignadas a la VLAN de estudiantes?**
**Ninguna (0).** La VLAN 10 aparece activa pero sin puertos en la columna *Ports*.

---

## Parte 1 — Conectividad antes de crear las VLAN

| Ping | Resultado |
|---|---|
| PC-A → PC-B | **Sí** |
| PC-A → PC-C | No |
| PC-A → S1 | No |
| PC-B → PC-C | No |
| PC-B → S2 | No |
| PC-C → S2 | No |
| S1 → S2 | **Sí** |

**¿Por qué fallaron esos pings?**
Porque están en subredes distintas y no hay ningún dispositivo de capa 3 que enrute entre ellas. PC-A y PC-B comparten la red `192.168.10.0/24`, por eso se ven; PC-C está en `192.168.20.0/24` y los switches en `192.168.1.0/24`.

---

## Parte 2 — Crear VLAN y asignar puertos

**¿Cuál es la VLAN predeterminada?**
La **VLAN 1** (`default`).

**¿Qué puertos se asignan a la VLAN predeterminada?**
**Todos**: `Fa0/1` a `Fa0/24` más `Gig0/1` y `Gig0/2`.

**Tras mover la IP a la VLAN 99, ¿cuál es el estado de la VLAN 99 y por qué?**
**`down/down`.** Una SVI solo levanta si hay al menos un puerto activo asignado a esa VLAN, o un enlace troncal que la transporte. En este momento la VLAN 99 no tiene ninguno.

**¿Es posible hacer ping de la PC-A a la PC-B? ¿Por qué?**
**No.** Ahora están en la VLAN 10 en switches distintos, pero el enlace F0/1 sigue siendo un puerto de acceso en la VLAN 1: el tráfico de la VLAN 10 no lo cruza.

**¿Es posible hacer ping del S1 al S2? ¿Por qué?**
**No.** Sus direcciones se movieron a la VLAN 99 y esa VLAN tampoco atraviesa el enlace de acceso.

---

## Parte 3 — Mantenimiento de VLAN

**Tras `no switchport access vlan` en F0/24, ¿a qué VLAN queda asociado?**
A la **VLAN 1**.

**¿Cuál es el nombre predeterminado de la VLAN 30?**
**`VLAN0030`.**

**Al eliminar la VLAN 30, ¿a qué VLAN queda asignado F0/24 y qué pasa con su tráfico?**
Queda asignado a la **VLAN 30, que ya no existe**. El puerto pasa a estado inactivo y **todo el tráfico del host conectado se descarta**.

**Después de `no switchport access vlan` en F0/24, ¿a qué VLAN se asignó?**
A la **VLAN 1**.

**¿Por qué se debe reasignar un puerto antes de eliminar su VLAN?**
Porque si no queda huérfano en una VLAN inexistente y deja de pasar tráfico **sin dar ningún aviso**. El puerto se ve conectado pero el host queda incomunicado, y es una falla difícil de diagnosticar.

---

## Parte 4 — Enlace troncal 802.1Q

| Ping (con el troncal activo) | Resultado |
|---|---|
| S1 → S2 | **Sí** |
| PC-A → PC-B | **Sí** |
| PC-A → PC-C | No |
| PC-B → PC-C | No |
| PC-A → S1 | No |
| PC-B → S2 | No |
| PC-C → S2 | No |

**Justificación de los que fallan:**
PC-C está en la VLAN 20 y en otra subred; sin un router no hay forma de llegar a ella desde la VLAN 10. Los pings PC → switch fallan porque los PCs están en `192.168.10.0/24` y `192.168.20.0/24`, mientras que las SVI de los switches están en `192.168.1.0/24`.

**¿Por qué configurar un troncal manualmente en lugar de usar DTP?**
Por seguridad y por control. DTP negocia automáticamente y un atacante puede aprovecharlo para hacer *switch spoofing* y convertir su puerto en troncal (ataque de salto de VLAN). Además la negociación tarda y puede caerse a modo acceso sin avisar. `switchport mode trunk` es determinista: el puerto es troncal siempre. En producción se acompaña de `switchport nonegotiate`.

---

## Parte 5 — Eliminar la base de datos de VLAN

**Para inicializar un switch por completo, ¿qué otros comandos se necesitan?**
`erase startup-config` y después `reload`. El `delete vlan.dat` por sí solo borra las VLAN, pero no la configuración; y `erase startup-config` por sí solo no borra las VLAN, porque viven en un archivo aparte de la flash.

---

## Reflexión

**1. ¿Qué se necesita para que los hosts de la VLAN 10 se comuniquen con los de la VLAN 20?**
Un dispositivo de **capa 3** que enrute entre ellas. Hay dos formas: un **router con subinterfaces 802.1Q** sobre un enlace troncal (*router-on-a-stick*) o un **switch multicapa** con una SVI por VLAN. Es exactamente el tema de la Práctica 3.

**2. ¿Cuáles son los beneficios principales de usar VLAN?**

- **Menos difusión:** divide un dominio de broadcast grande en varios pequeños, y eso mejora el rendimiento.
- **Seguridad:** los equipos de una VLAN no ven el tráfico de otra, así que se puede aislar información sensible.
- **Flexibilidad:** el agrupamiento es lógico, no físico. Un usuario puede cambiar de oficina sin cambiar de VLAN.
- **Ahorro:** se segmenta con los switches que ya se tienen, sin comprar equipos ni cablear de nuevo.
- **Administración más simple:** los cambios se hacen por configuración y no moviendo cables.

---

## Ejercicio de Spanning Tree (ilustraciones 1, 2 y 3)

**¿Cómo funciona STP y cómo encuentra sus puertos?**
STP evita bucles de capa 2 dejando una sola ruta activa y bloqueando las redundantes. El orden es siempre el mismo:

1. **Root switch:** el de menor Bridge ID (prioridad + MAC). Si nadie tocó la prioridad, gana la MAC más baja.
2. **Root port:** uno por switch no raíz, el de menor coste acumulado hacia la raíz.
3. **Designated port:** uno por segmento, el del lado con menor coste. Todos los puertos de la raíz lo son.
4. **Non-designated (bloqueado):** el que sobra. Packet Tracer lo pinta en ámbar.

**Costes:** FastEthernet = **19**, GigabitEthernet = **4**. Por eso en las tres ilustraciones los enlaces bloqueados son los FastEthernet: STP prefiere rodear por gigabit antes que usar un enlace de 100 Mbps.

| | Puerto bloqueado |
|---|---|
| Ilustración 1 (triángulo, 2 Gig + 1 Fa) | `Fa0/1` del switch izquierdo |
| Ilustración 2 (rombo, todo Gig) | `Gig0/2` del switch inferior |
| Ilustración 3 (malla, anillo Gig + Fa0/24 cruzados) | `Gig0/1` y `Fa0/24` del switch izquierdo |

En la ilustración 2 todos los enlaces cuestan lo mismo, así que el desempate lo deciden el Bridge ID del vecino y luego el número de puerto.

**Comandos usados para revisar STP:**

```
show spanning-tree
show spanning-tree vlan 1
show spanning-tree root
show spanning-tree summary
show spanning-tree interface gigabitEthernet 0/1
```

---

## Ejercicio "Password UMB"

Topología de 5 switches en cadena con 20 PCs, resuelta con **VTP** (dominio `UMB`, contraseña `UMB`): el switch rotulado como VTP SERVER trabaja en modo servidor y los otros cuatro en modo cliente, así que las VLAN se crean una sola vez y se propagan solas.

| VLAN | Nombre | Red |
|---|---|---|
| 10 | gerencia | 172.16.10.0/24 |
| 20 | presidencia | 172.16.20.0/24 |
| 30 | rh | 172.16.30.0/24 |
| 40 | porterias | 172.16.40.0/24 |
| 50 | secretarias | 172.16.50.0/24 |
| 99 | admin | 172.16.99.0/24 |

**Acceso del equipo del programador:** se creó la VLAN 99 de administración, cada switch recibió una SVI (`172.16.99.1` a `.5`) y las líneas vty quedaron con contraseña. El equipo del programador, puesto en la VLAN 99 con IP `172.16.99.100`, entra por Telnet a los cinco switches.

> **Supuesto declarado.** La topología del enunciado no incluye ningún router, así que la comunicación entre las cinco VLAN no es posible solo con switches de capa 2. Para cumplir el punto del enrutamiento se agregó un router 1941 con subinterfaces 802.1Q (*router-on-a-stick*), que es la técnica de la Práctica 3.

---

## Conclusiones

Las VLAN resuelven en configuración un problema que antes se resolvía con hardware: segmentar una red sin comprar switches ni volver a cablear. El aislamiento que producen es real —se comprobó que dos PCs en la misma subred IP no se ven si están en VLAN distintas— y eso las vuelve tanto una herramienta de rendimiento como de seguridad.

El enlace troncal es la pieza que hace que esa segmentación escale más allá de un switch: con un solo cable y el etiquetado 802.1Q, la VLAN 10 sigue siendo la VLAN 10 al otro lado. La prueba más clara de la práctica fue el par de pings PC-A → PC-B antes y después de configurar el troncal.

Quedó también claro el límite de la capa 2: por más VLAN y troncales que se configuren, dos VLAN nunca se comunican entre sí. Para eso hace falta subir a capa 3, que es donde continúa la Práctica 3.
