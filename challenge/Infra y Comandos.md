# Challenge — Infraestructura y comandos (4 redes + DHCP)

## Topología

```
   RED 3 / 30.0.0.0                              RED 2 / 10.0.0.0
   (WiFi · VLAN 40)                              (WiFi · VLAN 30)
      laptops                                       laptops
         )))                                          (((
        [AP3]                                        [AP2]
           \  Fa0/2                          Fa0/3  /
            \                                      /
             ┌────────────────────────────────────┐
             │      SW-ARRIBA · 2960-24TT         │
             └─────────────────┬──────────────────┘
                         Fa0/1 │ TRUNK 802.1Q
                               │ Fa0/0  (subif .30 y .40)
                        ┌──────┴───────┐
                        │ R-CENTRAL    │  2811
                        │  DHCP server │
                        └──────┬───────┘
                               │ Fa0/1  (subif .10 y .20)
                         Fa0/1 │ TRUNK 802.1Q
             ┌─────────────────┴──────────────────┐
             │      SW-ABAJO · 2960-24TT          │
             └──┬──────────────┬─────┬─────┬──────┘
          Fa0/2 │        Fa0/3 │Fa0/4│Fa0/5│
              [AP0]           DNS   Web1  Web2
               )))         20.0.0.2 .3     .4
             laptops        RED 1 / 20.0.0.0 (VLAN 20)
       RED 0 / 192.168.1.0
        (WiFi · VLAN 10)
```

## Mapa de redes

| Red del dibujo     | VLAN | Subred      | Máscara       | Gateway (subinterfaz)  | IPs            |
| ------------------ | ---- | ----------- | ------------- | ---------------------- | -------------- |
| RED 0 (WiFi)       | 10   | 192.168.1.0 | 255.255.255.0 | 192.168.1.1 → Fa0/1.10 | DHCP desde .11 |
| RED 1 (servidores) | 20   | 20.0.0.0    | 255.0.0.0     | 20.0.0.1 → Fa0/1.20    | Estáticas      |
| RED 2 (WiFi)       | 30   | 10.0.0.0    | 255.0.0.0     | 10.0.0.1 → Fa0/0.30    | DHCP desde .11 |
| RED 3 (WiFi)       | 40   | 30.0.0.0    | 255.0.0.0     | 30.0.0.1 → Fa0/0.40    | DHCP desde .11 |

Servidores (IP fija, gateway 20.0.0.1, máscara 255.0.0.0):

| Servidor | IP       |
| -------- | -------- |
| DNS      | 20.0.0.2 |
| Web1     | 20.0.0.3 |
| Web2     | 20.0.0.4 |

## Equipos

| Equipo | Cant. | Modelo |
|---|---|---|
| Router central | 1 | **2811** (trae Fa0/0 y Fa0/1) |
| Switch | 2 | 2960-24TT |
| Punto de acceso | 3 | **Access Point-PT** (Wireless Devices) |
| Servidor | 3 | Server-PT |
| Laptop | 3 por red WiFi | Laptop-PT |

## Cableado — todo con cable DIRECTO (Copper Straight-Through)

| Desde | Puerto | Hacia | Puerto |
|---|---|---|---|
| R-CENTRAL | Fa0/0 | SW-ARRIBA | Fa0/1 |
| R-CENTRAL | Fa0/1 | SW-ABAJO | Fa0/1 |
| SW-ARRIBA | Fa0/2 | AP3 (RED 3) | Port 0 |
| SW-ARRIBA | Fa0/3 | AP2 (RED 2) | Port 0 |
| SW-ABAJO | Fa0/2 | AP0 (RED 0) | Port 0 |
| SW-ABAJO | Fa0/3 | Servidor DNS | Fa0 |
| SW-ABAJO | Fa0/4 | Servidor Web1 | Fa0 |
| SW-ABAJO | Fa0/5 | Servidor Web2 | Fa0 |

No hay enlaces switch↔switch, así que aquí no se usa cable cruzado.

## Comandos — R-CENTRAL

Pega TODO el bloque de una vez:

```
enable
configure terminal
hostname R-CENTRAL
ip dhcp excluded-address 192.168.1.1 192.168.1.10
ip dhcp excluded-address 10.0.0.1 10.0.0.10
ip dhcp excluded-address 30.0.0.1 30.0.0.10
ip dhcp pool RED0
network 192.168.1.0 255.255.255.0
default-router 192.168.1.1
dns-server 20.0.0.2
exit
ip dhcp pool RED2
network 10.0.0.0 255.0.0.0
default-router 10.0.0.1
dns-server 20.0.0.2
exit
ip dhcp pool RED3
network 30.0.0.0 255.0.0.0
default-router 30.0.0.1
dns-server 20.0.0.2
exit
interface fastEthernet 0/0
no ip address
no shutdown
exit
interface fastEthernet 0/0.30
encapsulation dot1Q 30
ip address 10.0.0.1 255.0.0.0
exit
interface fastEthernet 0/0.40
encapsulation dot1Q 40
ip address 30.0.0.1 255.0.0.0
exit
interface fastEthernet 0/1
no ip address
no shutdown
exit
interface fastEthernet 0/1.10
encapsulation dot1Q 10
ip address 192.168.1.1 255.255.255.0
exit
interface fastEthernet 0/1.20
encapsulation dot1Q 20
ip address 20.0.0.1 255.0.0.0
exit
end
write memory
```

## Comandos — SW-ABAJO

```
enable
configure terminal
hostname SW-ABAJO
vlan 10
name RED0_WIFI
vlan 20
name RED1_SERVIDORES
exit
interface fastEthernet 0/1
switchport mode trunk
exit
interface fastEthernet 0/2
switchport mode access
switchport access vlan 10
exit
interface range fastEthernet 0/3-5
switchport mode access
switchport access vlan 20
exit
end
write memory
```

## Comandos — SW-ARRIBA

```
enable
configure terminal
hostname SW-ARRIBA
vlan 30
name RED2_WIFI
vlan 40
name RED3_WIFI
exit
interface fastEthernet 0/1
switchport mode trunk
exit
interface fastEthernet 0/2
switchport mode access
switchport access vlan 40
exit
interface fastEthernet 0/3
switchport mode access
switchport access vlan 30
exit
end
write memory
```

## Puntos de acceso — configuración de SSID

No tienen CLI, todo es por interfaz gráfica.

**Paso 1 — nombre del equipo.** Clic en el AP → pestaña **Config** → **GLOBAL / Settings** → campo **Display Name** → escribe `AP0`, `AP2` o `AP3`.

**Paso 2 — SSID y radio.** En el panel izquierdo, **INTERFACE → Port 1**. Ese es el puerto inalámbrico; Port 0 es el de cobre que va al switch.

| Campo | AP0 (RED 0) | AP2 (RED 2) | AP3 (RED 3) |
|---|---|---|---|
| SSID | `RED0` | `RED2` | `RED3` |
| 2.4 GHz Channel | 1 | 6 | 11 |
| Authentication | Disabled | Disabled | Disabled |
| Coverage Range (m) | 60 | 60 | 60 |
| Port Status | On | On | On |

Tres cosas que importan:

- **SSID distinto en cada AP.** Es lo único que decide a qué red se asocia cada laptop, y por lo tanto en qué VLAN cae y qué IP recibe. Si dos AP comparten SSID, las laptops se van a la de mejor señal y terminan con IP de la red equivocada.
- **Canales 1, 6 y 11.** Son los tres únicos que no se solapan en 2.4 GHz. AP2 y AP3 quedan cerca, así que esto evita interferencia.
- **Coverage Range 60.** Por defecto son 140 m y las coberturas se pisan entre sí. Bájalo en los tres y separa bien los grupos de laptops en el lienzo. Si tu versión de Packet Tracer no muestra ese campo, basta con separarlos más.

Referencia completa SSID → red → VLAN:

| AP | SSID | Red | VLAN | Puerto del switch | IP que reparte |
|---|---|---|---|---|---|
| AP0 | RED0 | RED 0 · 192.168.1.0 | 10 | SW-ABAJO Fa0/2 | 192.168.1.11 en adelante |
| AP2 | RED2 | RED 2 · 10.0.0.0 | 30 | SW-ARRIBA Fa0/3 | 10.0.0.11 en adelante |
| AP3 | RED3 | RED 3 · 30.0.0.0 | 40 | SW-ARRIBA Fa0/2 | 30.0.0.11 en adelante |

**Verificar el AP:** el enlace punteado entre cada laptop y su AP debe quedar verde, y en `show ip dhcp binding` del router deben salir las IPs agrupadas por red.

**Si el profesor pide seguridad inalámbrica:** en Port 1 de cada AP pon Authentication en **WPA2-PSK**, Encryption Type **AES** y PSK Pass Phrase `challenge2026` (mínimo 8 caracteres). Después, en cada laptop, PC Wireless te pedirá esa misma frase al conectarse.

## Laptops

Cada laptop trae tarjeta de cobre de fábrica y hay que cambiarla por la inalámbrica:

1. Clic en la laptop → pestaña **Physical** → **apaga** el equipo (botón del costado).
2. Arrastra fuera el módulo de cobre que trae puesto.
3. Arrastra el módulo **WPC300N** a la ranura.
4. **Enciende** el equipo. Se conecta sola al SSID más cercano.
5. Pestaña **Desktop** → **PC Wireless** → pestaña **Connect** → selecciona el SSID de su grupo (`RED0`, `RED2` o `RED3`) → **Connect**.
6. Pestaña **Desktop** → **IP Configuration** → marca **DHCP**.

Debe aparecer una IP del rango correcto y el gateway automáticamente. Eso es todo lo que se configura por laptop.

## Servidores (interfaz gráfica)

**IP fija** — Desktop → IP Configuration → Static, con los valores de la tabla de arriba.

**Servidor DNS** — Services → DNS → **On** → agrega registros tipo A:

| Name               | Address  |
| ------------------ | -------- |
| www.challenge.com  | 20.0.0.3 |
| web2.challenge.com | 20.0.0.4 |

**Web1 y Web2** — Services → HTTP → **On** → edita `index.html` con un texto distinto en cada uno para diferenciarlos.

## Verificación

En el router:
```
show ip interface brief
show ip dhcp binding
show ip dhcp pool
show ip route
```

`show ip route` debe mostrar las 4 redes como directamente conectadas (`C`). No hace falta configurar rutas ni protocolo de enrutamiento: el router toca las cuatro redes.

Desde una laptop de RED 3:
```
ping 30.0.0.1          gateway propio
ping 20.0.0.3          Servidor Web1, otra red
```
Navegador web → `http://20.0.0.3` y `http://www.challenge.com` (esto último prueba el DNS).

## Notas del diagrama

- El dibujo etiqueta **dos servidores como "Servidor Web1"**. Aquí el segundo es **Web2** (20.0.0.4).
- Los números de VLAN (10, 20, 30, 40) y los puertos son propuesta mía: el dibujo no los trae. Lo que sí viene del dibujo son las 4 redes y sus direcciones.
- No se necesita `ip helper-address`: el router es el gateway de las cuatro redes, así que recibe las peticiones DHCP directamente.
