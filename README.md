# Topologia2_IKEv1_RouteBased

https://www.youtube.com/watch?v=VMDXZFgK2FY

Instituto Tecnologico
Documentacion Tecnica - Configuracion de VPN
VPN Site-to-Site IPSec IKEv1
Basada en Enrutamiento (Route-Based / VTI)
Estudiante: Darwing Manuel Pena Reyes
Matricula: 2024-2690
Asignatura: Ethical Hacking 2
Fecha: Julio 2026
 1. Objetivo de la VPN
Establecer una conexion VPN segura punto a punto entre dos sedes utilizando una interfaz de tunel virtual (VTI), donde el trafico a cifrar se determina por la tabla de enrutamiento en vez de una lista de acceso. Este metodo es mas flexible que el policy-based porque permite ejecutar protocolos de enrutamiento dinamico sobre el tunel, siendo la base tecnica de tecnologias como DMVPN.
2. Topologia de red
Misma topologia fisica que la VPN policy-based (R2 y R3 como peers, LAN y switch en cada extremo, router ISP intermedio), pero en este caso se crea una interfaz Tunnel30 sobre cada router, protegida con un IPSec profile, y se enruta el trafico hacia la red remota mediante una ruta estatica que apunta a dicha interfaz.
2.1 Interfaces utilizadas
Dispositivo / Interfaz	Rol
R2 - F0/0	WAN, tunnel source hacia ISP
R2 - G2/0	LAN, hacia switch y PC1
R2 - Tunnel30	Interfaz virtual IPSec (VTI)
R3 - F1/0	WAN, tunnel source hacia ISP
R3 - G2/0	LAN, hacia switch y PC2
R3 - Tunnel30	Interfaz virtual IPSec (VTI)
<img width="748" height="486" alt="image" src="https://github.com/user-attachments/assets/bfae321b-9ea1-4f5d-ba52-85dd9068dd45" />



2.2 Direccionamiento IP
Segmento	Red / IP
LAN R2	20.24.30.0/24 - Gateway .90 - PC1 .91
LAN R3	20.24.31.0/24 - Gateway .90 - PC2 .91
Enlace ISP-R2	172.16.30.0/30 (R2 .1 / ISP .2)
Enlace ISP-R3	172.16.31.0/30 (ISP .1 / R3 .2)
Tunnel30 (VTI)	10.30.30.0/30 (R2 .1 / R3 .2)


Explicacion: Captura general de la topologia armada en GNS3, mostrando todos los dispositivos, sus interfaces conectadas, y el recuadro con nombre y matricula del estudiante.
3. Parametros utilizados
Parametro	Valor configurado
Version IKE	IKEv1 (ISAKMP)
Metodo de autenticacion	Pre-shared key (CiscoVPN123)
Cifrado Fase 1	AES 256
Hash Fase 1	SHA256
Grupo Diffie-Hellman	Grupo 14
Transform-set (Fase 2)	esp-aes 256 + esp-sha256-hmac
Modo del tunel	tunnel mode ipsec ipv4
Mecanismo de seleccion de trafico	Tabla de enrutamiento (route-based)

4. Configuracion y explicacion (capturas de pantalla)
Interfaz Tunnel30 (VTI) en R2
interface Tunnel30
 ip address 10.30.30.1 255.255.255.252
 tunnel source f0/0
 tunnel destination 172.16.31.2
 tunnel mode ipsec ipv4
 tunnel protection ipsec profile PROFILE-VTI

Explicacion: Se crea una interfaz de tunel virtual con IP propia (10.30.30.1), asociada a un origen y destino fisico, protegida por un IPSec profile. A diferencia del crypto map, aqui no existe ninguna ACL: todo lo que la tabla de rutas envie por esta interfaz se cifra automaticamente.
Ruta estatica hacia la red remota
ip route 20.24.31.0 255.255.255.0 Tunnel30

Explicacion: Esta ruta estatica indica que el trafico destinado a la LAN remota (20.24.31.0/24) debe enviarse a traves de la interfaz Tunnel30, lo que provoca su cifrado automatico. Este es el mecanismo central de las VPN route-based.
5. Evidencia de funcionamiento
Estado de la interfaz Tunnel30
Tunnel30 is up, line protocol is up
Tunnel protocol/transport IPSEC/IP
Tunnel protection via IPSec (profile "PROFILE-VTI")
La interfaz esta operativa (up/up) y confirma que el transporte es IPSEC/IP, con la proteccion del profile correctamente asociada.
Traceroute desde PC1 (diferencia clave vs policy-based)
trace 20.24.31.91
 1   20.24.30.90
 2   10.30.30.2
 3   *20.24.31.91 (Destination port unreachable)
A diferencia de la VPN policy-based, aqui el segundo salto SI es visible y corresponde a la IP del tunel en R3 (10.30.30.2), porque es una interfaz de red real y enrutable, no una simple regla de cifrado sobre una interfaz fisica.
6. Conclusion
La VPN IPSec IKEv1 route-based (VTI) quedo funcional y verificada. El contraste con la topologia policy-based (interior del tunel visible en el traceroute) demuestra la diferencia conceptual clave entre ambos metodos, y confirma que el enfoque route-based es el fundamento tecnico necesario para escalar hacia escenarios de enrutamiento dinamico como DMVPN.
