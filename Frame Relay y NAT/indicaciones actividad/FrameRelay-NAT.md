## PROBLEMA DE ASIGNACION DE IPs.

Un ISP(Internet Service Provider- Proveedor de Servicios de Internet) ha asignado a una

empresa la dirección IP pública de enrutamiento entre dominios sin clase (CIDR) 199.99.9.32/27. Esto es equivalente a 30 direcciones IP públicas.

Como la empresa tiene la necesidad interna de más de 30 direcciones, el gerente de TI ha decidido implementar NAT. Las direcciones 199.99.9.33 – 199.99.9.39 para la asignación estática y 199.99.9.40 – 199.99.9.62 para la asignación dinámica. El enrutamiento se realizará entre el ISP y el router del gateway de la empresa. Se utilizará una ruta estática entre el ISP y los routers de gateway y una ruta por defecto entre el router de gateway y el ISP. La conexión del ISP a la Internet se representará con una dirección de loopback en el router del ISP.


| DIRECCIONES DADAS POR EL ISP MÁSCARA DE SUBRED Número de Direcciones |
| --- |
| 199.99.9.32/27 255.255.255.224 30 |

| PLANEACIÓN DEL GERENTE DE TI |   |
| --- | --- |
| RANGO DE DIRECCIONES ESTÁTICAS(FIJAS) | RANGO DE DIRECCIONES DINÁMICAS |
| 199.99.9.33 – 199.99.9.39 | 199.99.9.40 – 199.99.9.62 |

| NOMBRE DEL ROUTER | DIRECCIÓN FASTHETHERNET/ MÁSCARA DE SUBRED | DIRECCIÓN SERIAL / MÁSCARA DE SUBRED | DRECCIÓN LOOPBACK |
| --- | --- | --- | --- |
| Gateway | 10.10.10.1/24 | 200.2.2.18/30 | No Aplica |
| ISP | No Aplica | 200.2.2.17/32 | 172.16.1.1/32 |

| IPs red privada | 10.10.10.0 Máscara de subred 24 |
| --- | --- |
| GATEWAY | 10.10.10.1 |

## PASOS A SEGUIR

- Crear la Topología aproximada (Packet Tracer).

- Configurar el Frame Relay.

- Configurar los routers.

- Configurar los hosts (PCs), con la dirección IP, máscara de subred y gateway por defecto.

- Verificar que la red esté funcionando.

- Crear una ruta estática.

- Crear una ruta por defecto.

- Definir el conjunto de Ips públicas utilizables.

- Definir una lista de acceso que coincida con las direcciones IP privada internas.

- Definir la traducción NAT de la lista interna al conjunto externo.

- Especificar las interfaces.

Probar la configuración.


INFRAESTRUCTURA 2

Ing. Robert Osorio Torres

## CONFIGURACIÓN DEL FRAME RELAY

En packet trace seleccionar WAN Emulation Cloud-PT

## Frame Relay


## INFRAESTRUCTURA 2

Ing. Robert Osorio Torres

- 1) Configurar el DLCI que va a contener el Serial 0 de la nube frame relay.

- 2) Se ingresa el DLCI y un nombre para poder identificarlo.

- 3) Hacer clic en el botón agregar

- 1) Hacemos clic en la opción Frame Relay

- 2) Elegimos de la Serial0 – Conector1 hacia la Seria1 – Conector2

- 3) Hacemos clic en el botón Agregar

## Gateway

Router#configure terminal Router(config)#hostname Gateway Gateway(config)#fastethernet 0 Gateway(config-if)#ip add 10.10.10.1 255.255.255.0 Gateway(config-if)#no shutdown Gateway(config-if)#exit Gateway(config)#interface serial 0 Gateway(config-if)#ip add 200.2.2.18 255.255.255.252

Gateway(config-if)#encapsulation frame-relay

Gateway(config-if)#frame-relay interface-dlci 101 Gateway(config-if)#no shutdown

Gateway(config)#ip route 0.0.0.0 0.0.0.0 200.2.2.17


## INFRAESTRUCTURA 2

Ing. Robert Osorio Torres

## ISP

Router#configure terminal

Router(config)#hostname ISP

ISP(config)#interface loopback 0

ISP(config-if)#ip add 172.16.1.1 255.255.255.255

ISP(config-if)#no shutdown

ISP(config-if)#exit

ISP(config)#interface serial 0

ISP(config-if)#ip add 200.2.2.17 255.255.255.252

ISP(config-if)#clock rate 64000

ISP (config-if)#encapsulation frame-relay

ISP (config-if)#frame-relay interface-dlci 102

ISP (config-if)#no sh

ISP(config)#ip route 199.99.9.32 255.255.255.224 200.2.2.18

ISP(config)#end

Probar las interfaces

ISP#ping 200.2.2.18

## Definir el conjunto de direcciones IP públicas utilizables

Para definir el conjunto de direcciones públicas, utilice el comando ip nat pool:

Gateway(config)#ip nat pool public-access 199.99.9.40 199.99.9.62

netmask 255.255.255.224

## Definir una lista de acceso que coincida con las direcciones IP privadas internas

Para definir la lista de acceso de modo que coincida con las direcciones privadas internas, use el

comando access list:

Gateway(config)#access-list 1 permit 10.10.10.0 0.0.0.255

## Definir la traducción NAT de la lista interna al conjunto externo

Para definir la traducción NAT, utilice el comando ip nat inside source:

Gateway(config)#ip nat inside source list 1 pool public-access

## Especificar las interfaces

Las interfaces activas en el router necesitan especificarse como interfaces internas o externas con respecto al NAT. Para hacer esto, use el comando ip nat inside o ip nat outside:

Gateway(config)#interface fastethernet 0

Gateway(config-if)#ip nat inside

Gateway(config-if)#interface serial 0

Gateway(config-if)#ip nat outside

## Probar la configuración

a. Configure una de las PC de la LAN con la dirección IP 10.10.10.10/24 y la dirección del gateway por defecto de 10.10.10.1. Desde el PC, haga ping a 172.16.1.1. Si tiene éxito, vea la traducción NAT en el router de Gateway, mediante el comando

show ip nat translations.


INFRAESTRUCTURA 2

Ing. Robert Osorio Torres

ADICIONALES.

- 1. ¿De qué lado debe estar el servidor DNS?

- 2. Adicione a la topología un servidor Web que se accesible desde la red de la empresa FARMACIA. Realice las pruebas para acceder a la página creada y complete las configuraciones necesarias.


Data Link Connection Identifier (DLCI) es el identificador de canal del circuito establecido en Frame Relay. Este identificador se aloja en la trama e indica el camino a seguir por los datos, es decir, el circuito virtual establecido. [URL 🔗](http://es.wikipedia.org/wiki/Frame_Relay)

El DLCI puede valer normalmente entre 0 y 1023 (10 bits), los valores del 0 al 15 y del 992 en adelante están reservador para funciones especiales

El DLCI tiene significado local, es decir, en el circuito virtual cada extremo puede tener un identificador de circuito diferente para identificar el mismo circuito.

Classless Inter-Domain Routing (CIDR Encaminamiento Inter-Dominios sin Clases) se introdujo en 1993 y representa la última mejora en el modo como se interpretan las direcciones IP. Su introducción permitió una mayor flexibilidad al dividir rangos de direcciones IP en redes separadas. De esta manera permitió:

- Un uso más eficiente de las cada vez más escasas direcciones IPv4.

- Un mayor uso de la jerarquía de direcciones ('agregación de prefijos de red'), disminuyendo la sobrecarga de los enrutadores principales de Internet para realizar el encaminamiento.

## DEFINICIÓN DE LOOPBACK

Una interfaz loopback es aquella interfaz virtual mas no fisica......sirve para tener latente el protocolo de ruteo como OSPF....el cual al no detectar interfaces en el dispositivo despues de cierto tiempo, lo descarta de la topologia de red........una interfaz loopback nunca se caera......

All TCP/IP implementations support a loopback device, which is a virtual network interface implemented in software only and not connected to any hardware, but which is fully integrated into the computer system's internal network infrastructure. Any traffic that a computer program sends to the loopback interface is immediately received on the same interface.
