# RIP_NG

### RIPng para redes basadas en IPv6

Configuración básica del protocolo RIPng como solución de enrutamiento para
redes basadas en el protocolo IPv6.

Introducción

En IPv6 el enrutamiento puede realizarse de manera estática o dinámica. En el caso
estático la configuración debe realizarse de manera manual.

En cuanto al enrutamiento dinámico existen varios tipos de protocolos entre ellos:
* IGP: Protocolo de pasarela interna o RIPv6 (RFC2008) o OSPFv3 (RFC 2740) o IS-IS
for IPv6 o EIGRP for IPv6.
* EGP: Protocolo de pasarela externa o MP-BGP4 (RFC 2545/ 2858)

El protocolo RIPv6 es un sucesor de los protocolos RIPv1 y RIPv2. Existen varias
implementaciones como por ejemplo GateD, MRTd, Kame route6d, Zebra, CISCO.

Este protocolo está basado en RIPv2 por lo cual posee las siguientes características:

* Tiene una distancia administrativa de 120
* Está basado en la técnica de vector distancia por lo cual también adolece de problemas en la convergencia.
* La métrica es el número de saltos
* La distancia máxima es de 15 saltos (más de 15 saltos es destino inalcanzable)
* Utiliza técnicas de horizonte dividido y envenenamiento en reversa.
* Utiliza temporizadores para evitar bucles de encaminamiento
* Por defecto envía actualizaciones cada 30 segundos
* Admite resumen de rutas y actualizaciones rápidas
* Tiene dos tipos de mensajes solicitud y respuesta, los cuales son encapsulados en UDP.

También posee ciertas diferencias respecto a RIPv2:
* El formato de mensajes
* Intercambio de información entre redes IPv6 (utiliza IPv6 para el transporte)
* Utiliza el grupo multicast FF02::9 como dirección destino para las actualizaciones de RIPng.
* Envía actualizaciones por el puerto UDP 521

Tabla mensajes de actualizaciones enviados por RIPng.

| 				RIPng						| 
| ----------------------------------------- | 
| * Los mensajes RIPng se encapsulan en	datagramas UDP dirigidos al puerto 521	y se difunden a la dirección IPv6		multicast FF02::9|
| * RIPng (RIP new generation) es la adaptación del protocolo RIP-2 para soportar la compatibilidad con IPv6|		
| * Los vectores de distancia contenidos en los mensajes de tipo RESPONSE, en		 lugar de direcciones de red IPv4, anuncian prefijos de red IPv6|			
| * La información de ruta contenida en un vector de distancia no incluye el campo “Next Hop”.|								
| * No utiliza información de autentificación como en RIP-2. En lugar de ello, RIPng utiliza los mecanismos de cifrado y autenticación disponibles en IPv6|	
| * Convergencia rápida: se basa en el algoritmo Diffusing Update Algorithm	(dual), permitiendo ofrecer tiempos de convergencia rápidos|
|* Utilizacion reducida de ancho de banda,no actualiza periódicamente, actualiza parcialmente cuando cambia la métrica a un destino, actualizando el enlace, sin actualizar la tabla de enrrutamiento|

*	Horizonte divido
El protocolo de vector de distancias emplea la regla de horizonte dividido
que prohibe a un router publicar una ruta por la misma interfaz por la que
se aprendió en primer lugar. El horizonte dividido es uno de los métodos
usados para prevenir el problema de ciclos de enrutamiento o "cuenta
hasta el infinito" debido a los altos tiempos de convergencia del protocolo
de vector de distancias.

*	Envenenamiento en reversa
El concepto de horizonte dividido con envenenamiento en reversa se basa en el
hecho de que es mejor comunicar explícitamente a un router que ignore una ruta
en lugar de no informarle nada al respeto en primer lugar.

*	Actualizaciones rápidas
RIPng envía actualizaciones parciales no periódicas. Esto significa que cuando hay
un cambio se envía la actualización con únicamente la información que ha sido
modificada.

*	Resumen de rutas
El resumen de ruta (agregación de ruta o supernetting) reduce la cantidad de rutas
que un router debe mantener en sus tablas anunciando y manteniendo una sola
dirección que contengas a las demás.

* Multicast
Las direcciones IPv6 son identificadores de 128bits de longitud, identifican
interfaces de red. A una misma interfaces de un nodo se le puede asignar
multiples direcciones IPv6. Dichas direcciones se clasifican en tres tipos: Unicast,
Anycast, Multicast.
Una dirección multicast IPv6 es un identificador para un grupo de interfaces
(normalmente en diferentes nodos). Una interface puede pertenecer a cualquier
número de grupos multicast.

###	Enrutamiento dinámico con RIPng

### Topología de red

![alt text](https://github.com/brahianf/RIP_NG/blob/master/TOPOLOGIARED.PNG)

### Tabla de direccionamiento

![alt text](https://github.com/brahianf/RIP_NG/blob/master/TABLADIRECIONAMIENTO.PNG)

### Comandos CLI

	* R0
	
	```
	Router>enable
	Router#configure terminal
	Router(config)#hostname R0
	R0(config)#ipv6 unicast-routing 
	R0(config)#ipv6 router rip LANNG
	R0(config)#ipv6 route 0::0/0 Serial0/0/1
	
	R0(config)#interface fastEthernet 0/0
	R0(config-if)ipv6 address 2001:db8:10::1/64
	R0(config-if)#ipv6 enable 
	R0(config-if)#ipv6 rip LANNG enable 
	R0(config-if)#no shutdown 
	R0(config-if)#exit
	
	R0(config)#interface fastEthernet 0/1
	R0(config-if)#ipv6 address 2001:db8:172:16::1/64
	R0(config-if)#ipv6 enable 
	R0(config-if)#ipv6 rip LANNG enable
	R0(config-if)#ipv6 rip LANNG default-information originate 
	R0(config-if)#no shutdown 
	R0(config-if)#exit
	
	R0(config)#interface serial 0/0/0
	R0(config-if)#ipv6 address 2001:db8:172:17::1/64
	R0(config-if)#ipv6 enable 
	R0(config-if)#ipv6 rip LANNG enable 
	R0(config-if)#ipv6 rip LANNG default-information originate 
	R0(config-if)#clock rate 64000
	R0(config-if)#no shutdown 
	R0(config-if)#exit
	
	R0(config)#interface serial 0/0/1
	R0(config-if)#ipv6 address 2001:DB8:200::2/64
	R0(config-if)#ipv6 rip LANNG default-information originate 
	R0(config-if)#no shutdown 
	R0(config-if)#exit
	R0(config)#exit
	R0#copy running-config startup-config 
	```

		* R1
	
	```
	Router>enable 
	Router#configure terminal 
	Router(config)#hostname R1
	R1(config)#ipv6 unicast-routing 
	R1(config)#ipv6 router rip LANNG 
	R1(config-rtr)#exit

	R1(config)#interface fastEthernet 0/0
	R1(config-if)#ipv6 address 2001:db8:bb:fea::1/64
	R1(config-if)#ipv6 enable 
	R1(config-if)#ipv6 rip LANNG enable 
	R1(config-if)#no shutdown 
	R1(config-if)#exit
	
	R1(config)#interface fastEthernet 0/1
	R1(config-if)#ipv6 address 2001:db8:172:16::2/64
	R1(config-if)#ipv6 enable 
	R1(config-if)#ipv6 rip LANNG enable 
	R1(config-if)#no shutdown 
	R1(config-if)#exit

	R1(config)#interface serial 0/0/0
	R1(config-if)#ipv6 address 001:db8:172:18::1/64
	R1(config-if)#clock rate 64000
	R1(config-if)#ipv6 enable 
	R1(config-if)#ipv6 rip LANNG enable 
	R1(config-if)#no shutdown 
	R1(config-if)#exit
	
	R1(config)#exit
	R1#copy running-config startup-config 
	```
	
		* R2
	
	```
	Router>enable 
	Router#configure terminal 
	Router(config)#hostname R2
	R2(config)#ipv6 unicast-routing 
	R2(config)#ipv6 router rip LANNG
	R2(config-rtr)#exit
	
	R2(config)#interface fastEthernet 0/0
	R2(config-if)#ipv6 address 2001:db8:bb:cafe::1/64
	R2(config-if)#ipv6 enable 
	R2(config-if)#ipv6 rip LANNG enable
	R2(config-if)#no shutdown 
	R2(config-if)#exit
	
	R2(config)#interface fastEthernet 0/1
	R2(config-if)#ipv6 address 2001:db8:172:19::2/64
	R2(config-if)#ipv6 enable 
	R2(config-if)#ipv6 rip LANNG enable
	R2(config-if)#no shutdown 
	R2(config-if)#exit
	
	R2(config)#interface serial 0/0/0
	R2(config-if)#ipv6 address 001:db8:172:18::2/64
	R2(config-if)#ipv6 enable
	R2(config-if)#ipv6 rip LANNG enable 
	R2(config-if)#no shutdown 
	R2(config-if)#exit
	R2(config)#exit
	R2#copy running-config startup-config 
	```
	
		* R3
	
	```
	Router>enable 
	Router#configure terminal 
	Router(config)#hostname R3
	R3(config)#ipv6 unicast-routing 
	R3(config)#ipv6 router rip LANNG 
	R3(config-rtr)#exit
	
	R3(config)#interface fastEthernet 0/0
	R3(config-if)#ipv6 address 2001:db8:20::1/64
	R3(config-if)#ipv6 enable 
	R3(config-if)#ipv6 rip LANNG enable 
	R3(config-if)#no shutdown 
	R3(config-if)#exit
	
	R3(config)#interface fastEthernet 0/1
	R3(config-if)#ipv6 address 2001:db8:172:19::1/64
	R3(config-if)#ipv6 enable 
	R3(config-if)#ipv6 rip LANNG enable 
	R3(config-if)#no shutdown 
	R3(config-if)#exit
	
	R3(config)#interface serial 0/0/0
	R3(config-if)#ipv6 address 2001:db8:172:17::2/64
	R3(config-if)#ipv6 enable 
	R3(config-if)#ipv6 rip LANNG enable 
	R3(config-if)#no shutdown 
	R3(config-if)#exit
	R3(config)#exit
	R3#copy running-config startup-config 
	```

### RIP_NG.pkt
	
https://github.com/brahianf/RIP_NG/blob/master/RIP_NG.pkt?raw=true
