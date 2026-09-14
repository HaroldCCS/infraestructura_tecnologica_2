# Infraestructura y comandos — VLAN y Enlaces Troncales

## Topología

```
        Serv.Docentes   Serv.Estudiantes   Serv.Invitados
        172.17.10.10      172.17.20.10       172.17.30.10
             \                 |                  /
           Fa0/10            Fa0/20            Fa0/3
              \                |                /
               ┌───────────────┴───────────────┐
               │          S1 · 2960-24TT        │
               └────┬───────────────────────┬───┘
              Fa0/1 │        TRUNK          │ Fa0/2
                    │                       │
              Fa0/1 │                       │ Fa0/1
              ┌─────┴──────┐         ┌──────┴─────┐
              │S2·2960-24TT│         │S3·2960-24TT│
              └┬────┬─────┬┘         └┬────┬─────┬┘
          Fa0/2│Fa0/3│Fa0/4│     Fa0/2│Fa0/3│Fa0/4│
              PC1   PC2   PC3        PC4   PC5   PC6
            V10    V20   V30        V10   V20   V30
```

## Direccionamiento

| Equipo            | VLAN | IP           | Máscara       | Gateway |
| ----------------- | ---- | ------------ | ------------- | ------- |
| PC1               | 10   | 172.17.10.21 | 255.255.255.0 | vacío   |
| PC2               | 20   | 172.17.20.22 | 255.255.255.0 | vacío   |
| PC3               | 30   | 172.17.30.23 | 255.255.255.0 | vacío   |
| PC4               | 10   | 172.17.10.24 | 255.255.255.0 | vacío   |
| PC5               | 20   | 172.17.20.25 | 255.255.255.0 | vacío   |
| PC6               | 30   | 172.17.30.26 | 255.255.255.0 | vacío   |
| Serv. Docentes    | 10   | 172.17.10.10 | 255.255.255.0 | vacío   |
| Serv. Estudiantes | 20   | 172.17.20.10 | 255.255.255.0 | vacío   |
| Serv. Invitados   | 30   | 172.17.30.10 | 255.255.255.0 | vacío   |

> PC6: la foto del profesor repite 172.17.30.23 (misma IP que PC3, quedan en la misma VLAN 30 = conflicto). Aqui va .26.
> Serv. Invitados en Fa0/3 porque la foto dice Fa0/30 y el 2960-24TT solo tiene 24 puertos.

## Cableado

> Switch a switch = cable **CRUZADO** (Copper Cross-Over). Todo lo demas = cable **directo** (Copper Straight-Through).
> Con cable directo entre switches el enlace queda ROJO y no levanta.

| Desde | Puerto | Hacia | Puerto | Cable |
|---|---|---|---|---|
| S1 | Fa0/1 | S2 | Fa0/1 | **CRUZADO** |
| S1 | Fa0/2 | S3 | Fa0/1 | **CRUZADO** |
| S1 | Fa0/10 | Serv. Docentes | Fa0 | Directo |
| S1 | Fa0/20 | Serv. Estudiantes | Fa0 | Directo |
| S1 | Fa0/3 | Serv. Invitados | Fa0 | Directo |
| S2 | Fa0/2 | PC1 | Fa0 | Directo |
| S2 | Fa0/3 | PC2 | Fa0 | Directo |
| S2 | Fa0/4 | PC3 | Fa0 | Directo |
| S3 | Fa0/2 | PC4 | Fa0 | Directo |
| S3 | Fa0/3 | PC5 | Fa0 | Directo |
| S3 | Fa0/4 | PC6 | Fa0 | Directo |

## Comandos — S1

Entra a S1 → pestaña CLI → pega TODO este bloque de una vez:

```
enable
configure terminal
hostname S1
vlan 10
name Cuerpo_Docente
vlan 20
name Estudiante
vlan 30
name Invitado
exit
interface fastEthernet 0/10
switchport mode access
switchport access vlan 10
exit
interface fastEthernet 0/20
switchport mode access
switchport access vlan 20
exit
interface fastEthernet 0/3
switchport mode access
switchport access vlan 30
exit
interface fastEthernet 0/1
switchport mode trunk
exit
interface fastEthernet 0/2
switchport mode trunk
exit
end
write memory
```

## Comandos — S2

```
enable
configure terminal
hostname S2
vlan 10
name Cuerpo_Docente
vlan 20
name Estudiante
vlan 30
name Invitado
exit
interface fastEthernet 0/2
switchport mode access
switchport access vlan 10
exit
interface fastEthernet 0/3
switchport mode access
switchport access vlan 20
exit
interface fastEthernet 0/4
switchport mode access
switchport access vlan 30
exit
interface fastEthernet 0/1
switchport mode trunk
exit
end
write memory
```

## Comandos — S3

```
enable
configure terminal
hostname S3
vlan 10
name Cuerpo_Docente
vlan 20
name Estudiante
vlan 30
name Invitado
exit
interface fastEthernet 0/2
switchport mode access
switchport access vlan 10
exit
interface fastEthernet 0/3
switchport mode access
switchport access vlan 20
exit
interface fastEthernet 0/4
switchport mode access
switchport access vlan 30
exit
interface fastEthernet 0/1
switchport mode trunk
exit
end
write memory
```

## PCs y servidores (los 9 equipos)

Desktop → IP Configuration → Static → IP y máscara de la tabla → Gateway vacío.

## Verificación

```cisco
S1#show vlan brief
S1#show interfaces trunk
S2#show vlan brief
S2#show interfaces trunk
S3#show vlan brief
S3#show interfaces trunk
```

```
ping entre misma VLAN (aunque switch distinto) → éxito
ping entre VLAN distinta → falla
```
