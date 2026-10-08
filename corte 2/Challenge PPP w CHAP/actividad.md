### Teoría de PPP y CHAP

- **PPP (Point-to-Point Protocol):** Es un protocolo del nivel de enlace de datos (Capa 2 del modelo OSI) diseñado para establecer una conexión directa entre dos nodos de red. Permite el transporte de múltiples protocolos de capa de red (como IPv4 o IPv6) mediante el uso de subprotocolos como **LCP** (_Link Control Protocol_), para establecer y probar la conexión, y **NCP** (_Network Control Protocol_), para negociar parámetros de capa 3.
    
- **CHAP (Challenge Handshake Authentication Protocol):** Es un método de autenticación seguro utilizado sobre PPP que evita el envío de contraseñas en texto plano a través de la red. Funciona mediante un proceso de "saludo en tres pasos" (_3-way handshake_):
    
    1. **Desafío (Challenge):** El router autenticador (por ejemplo, ISP) envía un mensaje de desafío al router cliente (R1). Este mensaje incluye un ID de sesión y una cadena de texto aleatoria.
        
    2. **Respuesta (Response):** El router cliente (R1) toma su contraseña compartida, el ID de sesión y el desafío recibido, genera un valor único utilizando el algoritmo de resumen **MD5**, y envía el resultado junto con su nombre de usuario (`R1`).
        
    3. **Aceptación o Rechazo (Success/Failure):** El autenticador (ISP) busca en su base local el usuario recibido (`R1`), toma la contraseña almacenada y realiza el mismo cálculo MD5. Si los resultados coinciden, valida la conexión; de lo contrario, la termina.
        
- Para este ejercicio aplicará los conocimientos adquiridos en las tareas anteriores
- La topología es la siguiente:

![[Pasted image 20261002091639.png]]

Utilizar la siguiente plantilla:

[PlantillaNubes821 (15).pka](https://umb.instructure.com/courses/81583/files/16332458?wrap=1 "PlantillaNubes821 (15).pka")[Download PlantillaNubes821 (15).pka](https://umb.instructure.com/courses/81583/files/16332458/download?download_frd=1)

Desde las redes se debe poder acceder al Web Server del ISP.

La topología debe ser documentada

Se validarán los protocolos implementados

Implementación del protocolo entre R1 y ISP (Forma de implementar el protocolo PPP y complementos)

**Configuración en R1**

**R1# configure terminal**

**R1(config)# hostname R1**

**R1(config)# username ISP password cisco123**

**R1(config)# interface Serial0/1/0**

**R1(config-if)# ip address 209.165.200.225 255.255.255.224**

**R1(config-if)# encapsulation ppp**

**R1(config-if)# ppp authentication chap**

**R1(config-if)# no shutdown**

**R1(config-if)# exit**

**# configure terminal**

**ISP(ISP config)# hostname ISP**

**ISP(config)# username R1 password cisco123**

**ISP(config)# interface Serial0/0/0**

**ISP(config-if)# ip address 209.165.200.226 255.255.255.224**

**ISP(config-if)# encapsulation ppp**

**ISP(config-if)# ppp authentication chap**

**ISP(config-if)# clock rate 64000**