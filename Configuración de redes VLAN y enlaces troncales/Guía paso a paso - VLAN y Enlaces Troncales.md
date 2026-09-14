---
tipo: laboratorio
practica: Configuración de redes VLAN y enlaces troncales
fuente: "05_infraestructura_tecnologica_2_laboratorio_2 261.docx" + foto de la topología entregada por el docente
software: Cisco Packet Tracer 7.x / 8.x
creado: 2026-09-02
tags: [redes, vlan, trunk, packet-tracer, cisco, dot1q, capa2]
---

# Laboratorio: VLANs y enlaces troncales (S1–S2–S3)

> **Objetivo:** armar únicamente la **infraestructura** que pidió el profesor en la foto — no hay que
> desarrollar el `.docx` completo (competencias, preguntas orientadoras, el video "Voky" de VTP, etc.). Esta
> guía cubre solo el montaje en Packet Tracer: 3 switches, 6 PCs y 3 servidores, 3 VLANs y los enlaces
> troncales entre switches.

Ver contexto general en [[00 - Contexto del Vault]].

## 0. Diferencia clave con la práctica de "VLAN Trunking" que ya hiciste

En la carpeta `trunk/` armaste un *router-on-a-stick*: las VLANs sí se comunicaban entre sí porque había un
router haciendo enrutamiento inter-VLAN. **Aquí no hay router.** Es una práctica de **Capa 2 pura**: el
objetivo es demostrar que

- dos hosts de la **misma VLAN** se ven entre sí aunque estén colgados de **switches distintos** (gracias al
  enlace troncal), y
- dos hosts de **VLANs distintas** **no** se ven entre sí, aunque estén en el mismo switch o conectados por
  el mismo cable troncal — porque nadie está enrutando entre ellas.

Por eso ningún equipo (PCs ni servidores) lleva *Default Gateway*: no hay a dónde apuntarlo.

---

## 1. Resumen de lo que vas a construir

```
                          Servidor        Servidor        Servidor
                          Docentes        Estudiantes     Invitados
                        172.17.10.10    172.17.20.10    172.17.30.10
                         (VLAN 10)       (VLAN 20)        (VLAN 30)
                               \             |              /
                            Fa0/10        Fa0/20        Fa0/3
                                 \           |           /
                                  ┌──────────┴──────────┐
                                  │   S1 · 2960-24TT     │
                                  └───┬──────────────┬───┘
                             Fa0/1    │  TRUNK  TRUNK │    Fa0/2
                            (802.1Q)  │               │  (802.1Q)
                                Fa0/1 │               │ Fa0/1
                          ┌───────────┴───┐     ┌─────┴───────────┐
                          │ S2 · 2960-24TT│     │ S3 · 2960-24TT  │
                          └─┬──────┬─────┬┘     └┬──────┬────────┬┘
                       Fa0/2│ Fa0/3│Fa0/4│  Fa0/2 │ Fa0/3│  Fa0/4 │
                           PC1    PC2   PC3      PC4    PC5      PC6
                         VLAN10 VLAN20 VLAN30  VLAN10  VLAN20   VLAN30
                        .10.21  .20.22 .30.23  .10.24  .20.25   .30.26
```

> Este diagrama ya trae aplicadas las dos correcciones a la foto del profesor (puerto `Fa0/3` en vez del
> inexistente `Fa0/30`, y PC6 con `.26` en vez de repetir la IP de PC3). Detalle en la sección 6.

### Tabla de direccionamiento

| Equipo | Rol | VLAN | Nombre de VLAN | IP | Máscara | Gateway |
|---|---|---|---|---|---|---|
| PC1 | Cuerpo Docente | 10 | Cuerpo_Docente | 172.17.10.21 | 255.255.255.0 | — (no aplica) |
| PC2 | Estudiante | 20 | Estudiante | 172.17.20.22 | 255.255.255.0 | — |
| PC3 | Invitado | 30 | Invitado | 172.17.30.23 | 255.255.255.0 | — |
| PC4 | Cuerpo Docente | 10 | Cuerpo_Docente | 172.17.10.24 | 255.255.255.0 | — |
| PC5 | Estudiante | 20 | Estudiante | 172.17.20.25 | 255.255.255.0 | — |
| PC6 | Invitado | 30 | Invitado | **172.17.30.26** | 255.255.255.0 | — |
| Servidor Docentes | Servidor | 10 | Cuerpo_Docente | 172.17.10.10 | 255.255.255.0 | — |
| Servidor Estudiantes | Servidor | 20 | Estudiante | 172.17.20.10 | 255.255.255.0 | — |
| Servidor Invitados | Servidor | 30 | Invitado | 172.17.30.10 | 255.255.255.0 | — |

> Ninguno de los tres servidores necesita gateway: la foto del profesor lo aclara explícitamente ("No
> requieren Default Gateway al ser una red de Capa 2 puramente local").

### Componentes

| Componente | Cantidad | Modelo | Notas |
|---|---|---|---|
| Switch | 3 | **2960-24TT** | S1 (distribución/central), S2 y S3 (acceso) |
| PC | 6 | PC-PT | PC1…PC6 |
| Servidor | 3 | Server-PT | Solo necesitan IP estática, no hace falta activar ningún servicio (HTTP/DHCP) |
| Cable cruzado | 2 | **Copper Cross-Over** | Enlaces troncales S1↔S2 y S1↔S3 (switch a switch) |
| Cable directo | 9 | Copper Straight-Through | 6 PCs + 3 servidores hacia su switch |

---

## 2. Armar la topología en Packet Tracer

### 2.1 Dónde encontrar cada objeto

| Qué necesitas | Categoría (inglés) | Categoría (español) | Modelo a arrastrar |
|---|---|---|---|
| Switches | **Network Devices → Switches** | Dispositivos de red → Conmutadores | **2960-24TT** (×3) |
| PCs | **End Devices → End Devices** | Dispositivos finales | **PC** (icono `PC-PT`) ×6 |
| Servidores | **End Devices → End Devices** | Dispositivos finales | **Server** (icono `Server-PT`) ×3 |
| Cable a PCs y servidores | **Connections → Connections** | Conexiones | **Copper Straight-Through** (negro continuo) |
| Cable entre switches | **Connections → Connections** | Conexiones | **Copper Cross-Over** (negro punteado) |

### 2.2 Colocar los dispositivos

1. Arrastra **3 × 2960-24TT** → se llamarán `Switch0`, `Switch1`, `Switch2`; los vas a renombrar a `S1`
   (centro), `S2` (izquierda) y `S3` (derecha) por comando.
2. Arrastra **6 × PC** → `PC1`…`PC6`. Colócalos tres a la izquierda de S2 y tres a la derecha de S3, como en
   el diagrama.
3. Arrastra **3 × Server** → colócalos arriba de S1: Servidor Docentes, Servidor Estudiantes, Servidor
   Invitados.

### 2.3 Cablear

| # | Desde | Puerto | Hacia | Puerto | Rol |
|---|---|---|---|---|---|
| 1 | S1 | `Fa0/1` | S2 | `Fa0/1` | **Trunk** — cable **CRUZADO** |
| 2 | S1 | `Fa0/2` | S3 | `Fa0/1` | **Trunk** — cable **CRUZADO** |
| 3 | S1 | `Fa0/10` | Servidor Docentes | `FastEthernet0` | Acceso VLAN 10 |
| 4 | S1 | `Fa0/20` | Servidor Estudiantes | `FastEthernet0` | Acceso VLAN 20 |
| 5 | S1 | `Fa0/3` | Servidor Invitados | `FastEthernet0` | Acceso VLAN 30 |
| 6 | S2 | `Fa0/2` | PC1 | `FastEthernet0` | Acceso VLAN 10 |
| 7 | S2 | `Fa0/3` | PC2 | `FastEthernet0` | Acceso VLAN 20 |
| 8 | S2 | `Fa0/4` | PC3 | `FastEthernet0` | Acceso VLAN 30 |
| 9 | S3 | `Fa0/2` | PC4 | `FastEthernet0` | Acceso VLAN 10 |
| 10 | S3 | `Fa0/3` | PC5 | `FastEthernet0` | Acceso VLAN 20 |
| 11 | S3 | `Fa0/4` | PC6 | `FastEthernet0` | Acceso VLAN 30 |

> ⚠️ **Los cables 1 y 2 (switch↔switch) van con cable CRUZADO (Copper Cross-Over), no directo.** Packet Tracer
> aplica la regla de cableado por tipo de dispositivo: con cable directo entre dos switches el enlace queda en
> **rojo** y nunca levanta. Todos los demás cables (a PCs y servidores) sí son directos.

> ℹ️ **Los números de puerto de los cables 1, 2 y 6–11 no aparecen en la foto del profesor** (la topología solo
> muestra líneas de conexión, sin etiqueta de puerto). Los de arriba son una propuesta razonable y consistente;
> puedes usar cualquier puerto libre del switch, siempre que anotes el que realmente usaste porque los
> comandos de las secciones 3 y 4 dependen de él.

---

## 3. Configuración de S1 (switch central)

> Los bloques de esta guía llevan el prompt delante (`S1(config)#`) para que veas en qué modo estás; **no se
> pegan tal cual**. Para copiar y pegar todo de una vez usa el archivo **`Infra y Comandos.md`**.

### 3.1 Nombre del equipo

```cisco
Switch>enable
Switch#configure terminal
Switch(config)#hostname S1
S1(config)#
```

### 3.2 Crear las VLANs

Esto es lo único que la foto del profesor muestra completo y sin ambigüedad:

```cisco
S1#configure terminal
S1(config)#vlan 10
S1(config-vlan)#name Cuerpo_Docente
S1(config-vlan)#vlan 20
S1(config-vlan)#name Estudiante
S1(config-vlan)#vlan 30
S1(config-vlan)#name Invitado
S1(config-vlan)#exit
```

### 3.3 Puertos de acceso hacia los servidores

```cisco
! Servidor Docentes → VLAN 10
S1(config)#interface fastEthernet 0/10
S1(config-if)#switchport mode access
S1(config-if)#switchport access vlan 10
S1(config-if)#exit

! Servidor Estudiantes → VLAN 20
S1(config)#interface fastEthernet 0/20
S1(config-if)#switchport mode access
S1(config-if)#switchport access vlan 20
S1(config-if)#exit

! Servidor Invitados → VLAN 30
S1(config)#interface fastEthernet 0/3
S1(config-if)#switchport mode access
S1(config-if)#switchport access vlan 30
S1(config-if)#exit
```

### 3.4 Puertos troncales hacia S2 y S3

La foto no incluye este bloque (se cortó de la captura), pero es indispensable — sin esto las VLANs no
cruzan entre switches y el nombre mismo de la práctica ("enlaces troncales") no se cumple:

```cisco
S1(config)#interface fastEthernet 0/1
S1(config-if)#switchport mode trunk
S1(config-if)#exit

S1(config)#interface fastEthernet 0/2
S1(config-if)#switchport mode trunk
S1(config-if)#exit
S1(config)#exit
S1#copy running-config startup-config
```

> El 2960-24TT solo soporta 802.1Q, así que no hace falta (ni acepta) el comando
> `switchport trunk encapsulation dot1q`.

---

## 4. Configuración de S2 y S3 (switches de acceso)

Son idénticos entre sí salvo el hostname. Cada uno necesita **las mismas VLANs creadas localmente** (aunque
solo tengan puertos de acceso, no de servidor) y **un puerto en modo trunk hacia S1**.

### 4.1 S2

```cisco
Switch>enable
Switch#configure terminal
Switch(config)#hostname S2
S2(config)#vlan 10
S2(config-vlan)#name Cuerpo_Docente
S2(config-vlan)#vlan 20
S2(config-vlan)#name Estudiante
S2(config-vlan)#vlan 30
S2(config-vlan)#name Invitado
S2(config-vlan)#exit

S2(config)#interface fastEthernet 0/2
S2(config-if)#switchport mode access
S2(config-if)#switchport access vlan 10
S2(config-if)#exit

S2(config)#interface fastEthernet 0/3
S2(config-if)#switchport mode access
S2(config-if)#switchport access vlan 20
S2(config-if)#exit

S2(config)#interface fastEthernet 0/4
S2(config-if)#switchport mode access
S2(config-if)#switchport access vlan 30
S2(config-if)#exit

S2(config)#interface fastEthernet 0/1
S2(config-if)#switchport mode trunk
S2(config-if)#exit
S2(config)#exit
S2#copy running-config startup-config
```

### 4.2 S3

```cisco
Switch>enable
Switch#configure terminal
Switch(config)#hostname S3
S3(config)#vlan 10
S3(config-vlan)#name Cuerpo_Docente
S3(config-vlan)#vlan 20
S3(config-vlan)#name Estudiante
S3(config-vlan)#vlan 30
S3(config-vlan)#name Invitado
S3(config-vlan)#exit

S3(config)#interface fastEthernet 0/2
S3(config-if)#switchport mode access
S3(config-if)#switchport access vlan 10
S3(config-if)#exit

S3(config)#interface fastEthernet 0/3
S3(config-if)#switchport mode access
S3(config-if)#switchport access vlan 20
S3(config-if)#exit

S3(config)#interface fastEthernet 0/4
S3(config-if)#switchport mode access
S3(config-if)#switchport access vlan 30
S3(config-if)#exit

S3(config)#interface fastEthernet 0/1
S3(config-if)#switchport mode trunk
S3(config-if)#exit
S3(config)#exit
S3#copy running-config startup-config
```

> 💡 **¿Por qué crear las VLANs también en S2 y S3, si la foto solo lo hace en S1?** Cada switch mantiene su
> **propia base de datos de VLANs** (no hay VTP configurado en esta práctica). Un puerto de acceso solo puede
> asignarse a una VLAN que exista *localmente* en ese switch. Si no la creas antes, Packet Tracer la crea
> igual al ejecutar `switchport access vlan 20`, pero queda sin nombre (`VLAN0020`); crearla a mano con su
> nombre correcto deja la configuración prolija y es lo que pediría cualquier profesor al revisar
> `show vlan brief`.

---

## 5. Configuración de PCs y servidores

Ningún equipo lleva CLI para esto — todo por interfaz gráfica.

**Clic en el equipo → pestaña `Desktop` → `IP Configuration` → `Static`.**

| Equipo | IP Address | Subnet Mask | Default Gateway |
|---|---|---|---|
| PC1 | 172.17.10.21 | 255.255.255.0 | *(vacío)* |
| PC2 | 172.17.20.22 | 255.255.255.0 | *(vacío)* |
| PC3 | 172.17.30.23 | 255.255.255.0 | *(vacío)* |
| PC4 | 172.17.10.24 | 255.255.255.0 | *(vacío)* |
| PC5 | 172.17.20.25 | 255.255.255.0 | *(vacío)* |
| PC6 | 172.17.30.26 | 255.255.255.0 | *(vacío)* |
| Servidor Docentes | 172.17.10.10 | 255.255.255.0 | *(vacío)* |
| Servidor Estudiantes | 172.17.20.10 | 255.255.255.0 | *(vacío)* |
| Servidor Invitados | 172.17.30.10 | 255.255.255.0 | *(vacío)* |

> Deja el campo *Default Gateway* completamente vacío en los 9 equipos. No es un olvido — es correcto para
> esta práctica (ver sección 0).

---

## 6. Cosas de la foto del profesor que hay que resolver antes de armar la red

### 6.1 El puerto de la foto (`Fa0/30`) no existe — ya está reemplazado por `Fa0/3`

La foto dice `interface FastEthernet 0/30` para el servidor de Invitados, pero el **2960-24TT** solo llega a
`Fa0/24`. Esta guía ya usa **`Fa0/3`** en su lugar, en el diagrama, el cableado y los comandos. No hay nada
que decidir: cablea y configura `Fa0/3`.

### 6.2 PC3 y PC6 aparecen con la misma IP (172.17.30.23)

En la foto, la etiqueta "Invitado / VLAN 30 / 172.17.30.23" se repite igual para PC3 (lado S2) **y** para PC6
(lado S3). Como ambos quedan en la **misma VLAN 30**, que además viaja por el enlace troncal entre S1, S2 y
S3, PC3 y PC6 terminan en el **mismo dominio de broadcast** — con la misma IP eso es un conflicto real (ARP
duplicado, uno de los dos equipos deja de responder confiablemente). Siguiendo el patrón numérico de los
otros pares (Docentes: `.21`/`.24`, Estudiantes: `.22`/`.25`), lo más consistente es que PC6 debería llevar
`172.17.30.26`. Coméntalo con el profesor o, si no puedes preguntarle a tiempo, usa `.26` y anótalo como
corrección en tu informe — igual que se hizo con el error de VLAN 10/20 repetida en la guía de VLAN Trunking.

### 6.3 Puertos de los enlaces (trunk y accesos de S2/S3) no vienen en la foto

Como se explicó en §2.3, la topología de la foto muestra las conexiones pero no los números de puerto
(salvo los tres accesos de S1 a los servidores). Los números usados en esta guía (`Fa0/1`/`Fa0/2` para los
troncales, `Fa0/2`-`Fa0/3`-`Fa0/4` para los accesos de S2 y S3) son una propuesta coherente, no un dato del
profesor. Usa los que quieras, pero sé consistente entre el cableado real en Packet Tracer y los comandos que
escribas.

---

## 7. Verificación

### 7.1 Misma VLAN, distinto switch → debe funcionar

```
Desde PC1 (172.17.10.21):  ping 172.17.10.24   ← PC4, otro switch, misma VLAN 10 → éxito (vía trunk)
Desde PC1 (172.17.10.21):  ping 172.17.10.10   ← Servidor Docentes → éxito
Desde PC2 (172.17.20.22):  ping 172.17.20.25   ← PC5, misma VLAN 20 → éxito
Desde PC2 (172.17.20.22):  ping 172.17.20.10   ← Servidor Estudiantes → éxito
Desde PC3 (172.17.30.23):  ping 172.17.30.10   ← Servidor Invitados → éxito
```

### 7.2 Distinta VLAN → debe fallar siempre

```
Desde PC2 (172.17.20.22):  ping 172.17.10.10   ← Servidor Docentes, VLAN distinta → falla (Request timed out)
Desde PC1 (172.17.10.21):  ping 172.17.20.22   ← PC2, VLAN distinta → falla
Desde PC3 (172.17.30.23):  ping 172.17.10.21   ← PC1, VLAN distinta → falla
```

> Esto es exactamente lo que dice la foto del profesor: *"Desde PC2 (172.17.20.22): un ping 172.17.10.10
> fallará automáticamente por estar en dominios de broadcast/VLAN distintos."* Si en tu montaje SÍ hace ping
> entre VLANs, algo está mal cableado o configurado (revisa que ningún puerto haya quedado en la VLAN
> equivocada).

### 7.3 Comandos de verificación en cada switch

```cisco
S1#show vlan brief              ! VLANs 10/20/30 con sus puertos de servidor asignados
S1#show interfaces trunk        ! Fa0/1 y Fa0/2 en modo trunk, dot1q
S2#show vlan brief               ! VLANs 10/20/30 con fa0/2, fa0/3, fa0/4
S2#show interfaces trunk        ! Fa0/1 en modo trunk hacia S1
S3#show vlan brief
S3#show interfaces trunk
```

Salida esperada de `show vlan brief` en S1:

```
VLAN Name                             Status    Ports
---- -------------------------------- --------- ---------------
1    default                          active    Fa0/3-9, Fa0/11-19, Fa0/21-24 (lo no usado)
10   Cuerpo_Docente                   active    Fa0/10
20   Estudiante                       active    Fa0/20
30   Invitado                         active    Fa0/3
```

---

## 8. Checklist de entrega

- [ ] 3 switches **2960-24TT** (S1, S2, S3), 6 PCs, 3 servidores colocados
- [ ] Enlace troncal S1↔S2 y S1↔S3 en `switchport mode trunk`
- [ ] VLANs 10 (Cuerpo_Docente), 20 (Estudiante), 30 (Invitado) creadas en **los tres switches**
- [ ] Puertos de acceso de S2 y S3 asignados a la VLAN correcta (¡revisar con `show vlan brief`!)
- [ ] Puertos de acceso de S1 hacia los servidores: Fa0/10, Fa0/20, Fa0/3
- [ ] PC6 con IP 172.17.30.26
- [ ] Los 9 equipos con IP estática y **sin** gateway
- [ ] Ping entre PCs/servidores de la misma VLAN, aun en switches distintos → éxito (captura)
- [ ] Ping entre VLANs distintas → falla (captura, sirve como evidencia de segmentación)
- [ ] `show vlan brief` y `show interfaces trunk` en los tres switches (captura)
- [ ] Archivo guardado como `VLAN-Enlaces-Troncales.pkt`
