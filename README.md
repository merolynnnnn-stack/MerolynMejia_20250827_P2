# MerolynMejia_20250827_P2

## Infraestructura 1 - VPN IPsec Site-to-Site entre dos FortiGate

**Estudiante:** Merolyn Mejía  
**Matrícula:** 2025-0827  

---

## 🎥 Video demostrativo

**Enlace del video:**  
https://youtu.be/VYO3u3gqH28

---

## 📑 Índice

1. [Propósito del laboratorio](#propósito-del-laboratorio)
2. [Topología](#topología)
3. [Direccionamiento IP](#direccionamiento-ip)
4. [Configuración del FGT-USUARIOS](#configuración-del-fgt-usuarios)
5. [Configuración del FGT-SERVIDOR](#configuración-del-fgt-servidor)
6. [Enrutamiento](#enrutamiento)
7. [Políticas de firewall y NAT](#políticas-de-firewall-y-nat)
8. [VPN IPsec Site-to-Site](#vpn-ipsec-site-to-site)
9. [Pruebas de funcionamiento](#pruebas-de-funcionamiento)
10. [Conclusión](#conclusión)
11. [Archivos del repositorio](#archivos-del-repositorio)

---

## Propósito del laboratorio

El propósito de este laboratorio es implementar una VPN IPsec Site-to-Site entre dos firewalls FortiGate en GNS3, permitiendo la comunicación segura entre una red de usuarios y una red de servidores.

La infraestructura fue configurada con direccionamiento basado en la matrícula **2025-0827**. También se configuraron interfaces LAN y WAN, DHCP, rutas, políticas de firewall, NAT y el túnel VPN IPsec.

Una parte importante de la práctica consiste en demostrar que los equipos de ambas redes pueden comunicarse mediante las políticas y el túnel VPN configurado.

---

## Topología

La infraestructura está formada por:

- 2 FortiGate.
- 1 VPCS para representar al usuario.
- 1 servidor.
- 1 switch para la red WAN.
- 1 conexión NAT/ISP.
- 1 túnel VPN IPsec Site-to-Site.

![Topología de la infraestructura](Imagenes/image01.png)

La topología está dividida en dos redes internas que se comunican mediante una VPN IPsec Site-to-Site. En el primer extremo se encuentra el FGT-USUARIOS, encargado de administrar la red `10.8.27.0/25`, donde se encuentra el equipo que representa al usuario.

En el segundo extremo se encuentra el FGT-SERVIDOR, encargado de administrar la red `172.8.27.0/28`, donde se encuentra el WEB-SERVER.

Ambos FortiGate están conectados mediante un segmento WAN que representa el ISP dentro del entorno de GNS3. Sobre esta conexión se establece el túnel IPsec que permite transportar el tráfico entre las dos redes internas.

De esta manera, las redes privadas no necesitan estar conectadas directamente, sino que utilizan los dos FortiGate como extremos de la comunicación segura.

---
---

## Direccionamiento IP

| Dispositivo | Interfaz/Función | Dirección IP |
|---|---|---|
| NAT/ISP | Gateway WAN | 192.168.42.1 |
| FGT-USUARIOS | port1 / WAN | 192.168.42.233/24 |
| FGT-USUARIOS | USUARIOS-VLAN10 / LAN | 10.8.27.1/25 |
| USUARIOS-VLAN10 | Cliente | 10.8.27.2/25 |
| FGT-SERVIDOR | port1 / WAN | 192.168.42.37/24 |
| FGT-SERVIDOR | SERVIDORES / LAN | 172.8.27.1/28 |
| WEB-SERVER | Servidor | 172.8.27.2/28 |

El direccionamiento fue diseñado utilizando como referencia los últimos cuatro dígitos de la matrícula `2025-0827`, por lo que se utilizaron los identificadores `8.27` en las redes internas.

La red de usuarios utiliza `10.8.27.0/25`, proporcionando espacio suficiente para los dispositivos pertenecientes a este segmento. El FGT-USUARIOS utiliza `10.8.27.1` como gateway y proporciona direccionamiento mediante DHCP.

La red de servidores utiliza `172.8.27.0/28`. En este segmento, el FGT-SERVIDOR utiliza `172.8.27.1` como gateway y el WEB-SERVER utiliza `172.8.27.2`.

Para la comunicación entre ambos FortiGate se utiliza el segmento `192.168.42.0/24`, que representa la red WAN/ISP dentro del laboratorio virtualizado en GNS3.

---

# Configuración del FGT-USUARIOS

## Interfaz WAN

La interfaz `port1` del FGT-USUARIOS obtiene mediante DHCP la dirección:

**192.168.42.233/24**

El gateway utilizado es:

**192.168.42.1**

![WAN FGT-USUARIOS](Imagenes/image02.png)

### Función del FGT-USUARIOS
El FGT-USUARIOS representa el extremo de la infraestructura donde se encuentra la red de clientes. Su función principal es proporcionar conectividad a los equipos de usuarios, administrar el direccionamiento de esta red y controlar el tráfico que sale hacia Internet o que se dirige hacia la red remota de servidores.

Además, este FortiGate participa como uno de los extremos de la VPN IPsec Site-to-Site. Por medio de la VPN, el tráfico destinado a la red `172.8.27.0/28` puede ser enviado hacia el FGT-SERVIDOR.

Las configuraciones realizadas en este dispositivo incluyen la interfaz WAN, la interfaz LAN de usuarios, DHCP, rutas, políticas de firewall, NAT y la configuración del túnel IPsec.

---
---

## Interfaz LAN de usuarios

La interfaz correspondiente a la red de usuarios fue configurada con:

- **Nombre:** USUARIOS-VLAN10
- **Interfaz:** port3
- **Dirección:** 10.8.27.1/25
- **Rol:** LAN
- **PING:** habilitado

![LAN FGT-USUARIOS](Imagenes/image03.png)

### Función de la interfaz WAN

La interfaz `port1` es utilizada como conexión externa del FGT-USUARIOS. Esta interfaz permite la comunicación con el segmento WAN/ISP y sirve como punto de salida hacia el otro extremo de la infraestructura.

La dirección `192.168.42.233/24` identifica al FGT-USUARIOS dentro de esta red. El gateway `192.168.42.1` corresponde al dispositivo que representa la salida hacia el segmento ISP.

Esta interfaz también es utilizada como interfaz de transporte para establecer la comunicación IPsec con el FGT-SERVIDOR.

---
---

## DHCP para usuarios
### Funcionamiento del DHCP

El servicio DHCP fue habilitado en el FGT-USUARIOS para evitar que las direcciones IP de los clientes tuvieran que configurarse manualmente.

El rango disponible comprende desde `10.8.27.2` hasta `10.8.27.126`, utilizando la máscara `255.255.255.128`.

El gateway entregado a los clientes es `10.8.27.1`, correspondiente a la interfaz LAN del FortiGate.

De esta manera, cuando el equipo de usuario se conecta a la red, puede obtener automáticamente los parámetros necesarios para comunicarse con el resto de la infraestructura.

Se habilitó DHCP en la red de usuarios para asignar automáticamente las direcciones IP a los clientes.

**Rango DHCP:**
10.8.27.2 - 10.8.27.126

**Máscara:**
255.255.255.128

**Gateway:**
10.8.27.1

![DHCP Usuarios](Imagenes/image04.png)

---
---

## Dirección obtenida por el usuario

El equipo `USUARIOS-VLAN10` obtiene correctamente una dirección IP perteneciente a la red de usuarios.

![IP del usuario](Imagenes/image05.png)

---
---

# Configuración del FGT-SERVIDOR

## Interfaz WAN

La interfaz `port1` del FGT-SERVIDOR obtiene:

**192.168.42.37/24**

Gateway:

**192.168.42.1**

![WAN FGT-SERVIDOR](Imagenes/image06.png)

---
---

## Interfaz LAN de servidores

La interfaz de servidores fue configurada con:

- **Nombre:** SERVIDORES
- **Interfaz:** port3
- **Dirección:** 172.8.27.1/28
- **Rol:** LAN
- **PING:** habilitado

![LAN FGT-SERVIDOR](Imagenes/image07.png)

---
---

## WEB-SERVER

El servidor utiliza la dirección:

172.8.27.2/28


![Configuración WEB-SERVER](Imagenes/image08.png)

La ruta por defecto del servidor apunta hacia el FortiGate:

default via 172.8.27.1


![Ruta WEB-SERVER](Imagenes/image09.png)

---
---

# Enrutamiento

## Importancia del enrutamiento

El enrutamiento permite que cada FortiGate conozca cómo alcanzar las redes que se encuentran fuera de sus segmentos directamente conectados.

En esta infraestructura, el FGT-USUARIOS necesita conocer el camino hacia `172.8.27.0/28`, mientras que el FGT-SERVIDOR necesita conocer el camino de regreso hacia `10.8.27.0/25`.

Estas rutas son necesarias para que exista comunicación bidireccional. No basta con establecer el túnel VPN; ambos extremos deben saber hacia dónde enviar el tráfico y cómo regresar las respuestas.

## Rutas del FGT-USUARIOS

El FGT-USUARIOS posee una ruta hacia la red remota de servidores mediante el túnel `VPN-USER-SERVER`.

Las redes principales son:

0.0.0.0/0
10.8.27.0/25
172.8.27.0/28
192.168.42.0/24


La red remota:

172.8.27.0/28

es alcanzada mediante la VPN.

![Rutas FGT-USUARIOS](Imagenes/image10.png)

---
---

## Rutas del FGT-SERVIDOR

El FGT-SERVIDOR posee una ruta hacia:

10.8.27.0/25

mediante el túnel `VPN-SERVER-USER`.

La red:

172.8.27.0/28

se encuentra directamente conectada al FortiGate.

![Rutas FGT-SERVIDOR](Imagenes/image11.png)

---
---

# Políticas de firewall y NAT

## FGT-USUARIOS

Se configuraron políticas para permitir la comunicación entre la red de usuarios, Internet y el túnel VPN.

La política:

USUARIOS-VLAN10 → port1

utiliza **NAT Enabled** para el acceso a Internet.

La política:

USUARIOS-VLAN10 → VPN-USER-SERVER


utiliza **NAT Disabled**, permitiendo conservar las direcciones originales durante la comunicación entre las redes privadas.

También existe la política de retorno:

VPN-USER-SERVER → USUARIOS-VLAN10

![Políticas FGT-USUARIOS](Imagenes/image12.png)

---
---

## FGT-SERVIDOR

En el segundo FortiGate se configuraron las políticas correspondientes a la red de servidores.

La política hacia Internet utiliza NAT, mientras que la comunicación a través de la VPN se mantiene sin NAT.

También se configuró la política de retorno desde el túnel hacia la red de servidores.

![Políticas FGT-SERVIDOR](Imagenes/image13.png)

---
---

# VPN IPsec Site-to-Site

## Funcionamiento de la VPN

La VPN IPsec Site-to-Site permite conectar de forma lógica las dos redes privadas aunque físicamente se encuentren separadas por el segmento WAN/ISP.

El FGT-USUARIOS establece el túnel `VPN-USER-SERVER` con el FGT-SERVIDOR, mientras que en el extremo contrario se encuentra la VPN `VPN-SERVER-USER`.

La comunicación utiliza el segmento WAN únicamente como medio de transporte entre ambos extremos. Una vez establecido el túnel, el tráfico destinado a las redes remotas puede ser procesado mediante IPsec.

Los selectores de la VPN identifican las redes internas que participan en la comunicación:

- Red de usuarios: `10.8.27.0/25`
- Red de servidores: `172.8.27.0/28`

El objetivo es que el tráfico entre ambas redes utilice el túnel en lugar de atravesar la red externa como tráfico interno sin protección.


Se configuró una VPN **IPsec Site-to-Site** entre los dos FortiGate.

### Lado Usuarios

VPN-USER-SERVER

La VPN utiliza `port1` como interfaz WAN y se encuentra en estado **Up**.

![VPN FGT-USUARIOS](Imagenes/image14.png)

### Lado Servidor

VPN-SERVER-USER

El túnel también se encuentra en estado **Up**.

![VPN FGT-SERVIDOR](Imagenes/image15.png)

Esto confirma que ambos extremos del túnel IPsec fueron establecidos correctamente.

### Validación del túnel

La visualización del estado `Up` en ambos FortiGate confirma que los dos extremos de la VPN lograron establecer correctamente la conexión IPsec.

Sin embargo, el estado `Up` por sí solo no demuestra que las políticas de firewall permitan el tráfico. Por esta razón, posteriormente se realizaron pruebas de comunicación, deshabilitando y habilitando nuevamente la política correspondiente.

De esta manera se pudo comprobar tanto el establecimiento de la VPN como el efecto de las políticas de seguridad sobre el tráfico que atraviesa el túnel.

---
---

# Pruebas de funcionamiento

## Objetivo de las pruebas

Las pruebas fueron realizadas para comprobar que la infraestructura no solamente tiene configurados los dispositivos y el túnel VPN, sino que la comunicación funciona de acuerdo con las políticas establecidas.

Se realizaron tres escenarios:

1. Comunicación con la VPN y las políticas habilitadas.
2. Bloqueo de la comunicación mediante la deshabilitación de la política correspondiente.
3. Restauración de la política y comprobación nuevamente de la comunicación.

Este procedimiento permite comparar el comportamiento de la red cuando el tráfico está permitido y cuando el firewall lo bloquea.

## Prueba 1 - Comunicación con la VPN operativa

Desde `USUARIOS-VLAN10` se realizó:

ping 172.8.27.2

El WEB-SERVER respondió correctamente.

En la misma evidencia se observan los túneles IPsec en estado **Up**.

![VPN funcionando](Imagenes/image16.png)

### Análisis del resultado

El resultado exitoso del ping demuestra que existe conectividad entre las dos redes privadas.

El equipo de usuarios, ubicado en `10.8.27.0/25`, puede alcanzar el WEB-SERVER ubicado en `172.8.27.0/28`.

Además, el estado `Up` de los túneles confirma que la VPN IPsec se encuentra establecida durante la prueba.

Por lo tanto, en este escenario se comprueba tanto la conectividad de extremo a extremo como el funcionamiento del túnel.

---
---

## Prueba 2 - Bloqueo del tráfico

Para comprobar el control del tráfico, se deshabilitó temporalmente la política:

USUARIOS-VLAN10 → VPN-USER-SERVER

Al ejecutar nuevamente:

ping 172.8.27.2

se obtuvieron respuestas:

timeout

![VPN bloqueada](Imagenes/image17.png)

### Análisis del bloqueo

Al deshabilitar la política que permite el tráfico desde `USUARIOS-VLAN10` hacia `VPN-USER-SERVER`, el FortiGate deja de permitir ese flujo.

Aunque el túnel IPsec continúe configurado, la política de firewall determina si el tráfico puede atravesar el dispositivo.

El resultado de `timeout` demuestra que el tráfico fue bloqueado y que la política de seguridad está teniendo efecto sobre la comunicación.

---
---

## Prueba 3 - Restauración de la comunicación

Se volvió a habilitar la política de firewall y se realizó nuevamente:

ping 172.8.27.2

La comunicación fue restaurada correctamente.

![VPN restaurada](Imagenes/image18.png)

### Análisis de la restauración

Después de habilitar nuevamente la política `USUARIOS-VLAN10 → VPN-USER-SERVER`, se realizó nuevamente la prueba de conectividad hacia `172.8.27.2`.

El servidor volvió a responder correctamente.

Esto demuestra que el bloqueo observado en la prueba anterior estaba relacionado con la política de firewall y que, al restaurar la autorización del tráfico, la comunicación entre ambas redes vuelve a funcionar.

Por lo tanto, las tres pruebas permiten comprobar el comportamiento de la infraestructura en los estados permitido, bloqueado y restaurado.

---
---

# Conclusión

En esta práctica se implementó una infraestructura de red compuesta por dos FortiGate conectados mediante una VPN IPsec Site-to-Site. Se configuraron las redes de usuarios y servidores, el direccionamiento IP, DHCP, rutas, políticas de firewall y NAT.

Durante la implementación se comprobó que el FGT-USUARIOS puede comunicarse con el FGT-SERVIDOR mediante el túnel IPsec y que el equipo de usuarios puede alcanzar el WEB-SERVER ubicado en la red remota.

Las pruebas realizadas también permitieron comprobar la función de las políticas de firewall. Con la política correspondiente habilitada, la comunicación entre ambas redes fue exitosa. Al deshabilitarla, el tráfico fue bloqueado y se obtuvo un `timeout`. Finalmente, al habilitar nuevamente la política, la comunicación fue restaurada.

Esto permitió comprobar de manera práctica que la VPN establece el canal de comunicación entre las dos redes, mientras que las políticas de firewall determinan qué tráfico está permitido atravesar dicho canal.

La práctica permitió reforzar los conceptos de segmentación de redes, direccionamiento, DHCP, NAT, enrutamiento, políticas de firewall y VPN IPsec Site-to-Site dentro de una infraestructura de Seguridad de Redes.

---
---

# Archivos del repositorio

La carpeta **Imagenes** contiene las evidencias visuales utilizadas en esta documentación.

La carpeta **Running-Configs** contiene las configuraciones exportadas de los dispositivos FortiGate.
