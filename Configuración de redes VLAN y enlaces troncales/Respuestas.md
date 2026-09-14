# Respuestas — Laboratorio 2 (VLAN y enlaces troncales)

Solo las preguntas que aplican a la topología implementada (S1/S2/S3, VLAN 10 Cuerpo_Docente, 20 Estudiante, 30 Invitado).

## Comandos y verificación

**¿Qué comando determina si un puerto está funcionando en modo access?**
`show interfaces switchport` (`sh int sw`). Muestra "Administrative Mode: static access".

**¿Cuál es la VLAN predeterminada?**
La VLAN 1 (`default`).

**¿Qué puertos se asignan a la VLAN predeterminada?**
Todos los que no se asignaron a otra VLAN. En nuestros switches, todo lo que no sea puerto de acceso o troncal configurado.

## Manejo de la base de datos de VLANs

**Si aplico `no switchport access vlan` a un puerto, ¿a qué VLAN queda?**
A la VLAN 1.

**Si asigno un puerto a una VLAN que no existe, ¿qué pasa?**
El switch la crea sola, con nombre por defecto `VLAN00XX` (ej. `VLAN0030`).

**Si elimino una VLAN con `no vlan 30`, ¿qué pasa con sus puertos?**
Siguen asignados a una VLAN que ya no existe, quedan inactivos y su tráfico se descarta. El equipo conectado pierde la conexión.

**¿Por qué hay que reasignar los puertos antes de borrar una VLAN?**
Porque si no, quedan huérfanos y dejan de reenviar tráfico sin ningún aviso. La falla es silenciosa y difícil de detectar.

**¿Qué comandos dejan el switch en configuración de fábrica?**
```
erase startup-config
delete vlan.dat
reload
```

## Enlaces troncales

**¿Por qué configurar el trunk manualmente (`switchport mode trunk`) en vez de dejar DTP?**
Por seguridad y control. DTP negocia solo, y un equipo conectado puede forzar un trunk y ver el tráfico de todas las VLANs (VLAN hopping). Además la negociación falla si el vecino no habla DTP. Manual queda fijo y predecible.

## Resultados esperados en nuestra topología

| Ping | Resultado | Por qué |
|---|---|---|
| PC1 → Serv. Docentes (172.17.10.10) | Sí | Misma VLAN 10, el trunk la lleva de S2 a S1 |
| PC1 → PC4 | Sí | Misma VLAN 10, en switches distintos, unidos por el trunk |
| PC2 → PC5 | Sí | Misma VLAN 20 |
| PC3 → PC6 | Sí | Misma VLAN 30 |
| PC2 → Serv. Docentes (172.17.10.10) | No | VLAN 20 contra VLAN 10, dominios de broadcast distintos |
| PC1 → PC2 | No | VLAN 10 contra VLAN 20 |

## Reflexión

**¿Qué se necesita para que la VLAN 10 se comunique con la VLAN 20?**
Un dispositivo de capa 3: un router con subinterfaces 802.1Q (router-on-a-stick) o un switch multicapa con SVIs e `ip routing`. Nuestra topología no lo tiene, por eso el aislamiento entre VLANs es total y es justamente lo que demuestra la práctica.

**¿Cuáles son los beneficios principales de usar VLANs?**

- Cada VLAN es su propio dominio de broadcast: menos ruido, mejor rendimiento.
- Seguridad: equipos de distintas VLANs no se ven salvo que un router lo permita.
- Agrupan por función o departamento, sin importar la ubicación física.
- Un solo switch hace de varias redes: menos hardware.
- Mover a alguien de red es cambiar un comando, no tender cable.
