# 🚀 Implementación de VPN Site-to-Site (GRE sobre IPsec)

## 📖 Resumen Ejecutivo
Este proyecto se desarrolló para explicar a detalle cómo implementar y combinar dos tecnologías que permiten configurar, escalar, diseñar y desplegar topologías corporativas a gran escala basadas en IPsec y GRE. La arquitectura se rige bajo estrictos criterios de tunelización, integridad, confidencialidad y autenticación, garantizando la seguridad absoluta del tráfico de extremo a extremo.

---

## 🛠️ Tecnologías y Protocolos Utilizados
* **Dispositivos:** 2 Cisco Routers ISR 2900 Series (Routers locales en México y Alemania) / 2 Cisco Routers C2900 Series (Simulando el ISP).
* **Enrutamiento (Underlay):** IP Static Routing.
* **Túneles (Overlay):** Interfaces Tunnel (Generic Routing Encapsulation - GRE).
* **Seguridad y Criptografía:** IPsec, Encryption AES, Pre-Shared key, ISAKMP IKEv1, SHA-HMAC, ACLs Host-to-Host.

---

## 📐 Arquitectura de Red (Topología)
> *Topología lógica y física implementada para el despliegue del túnel seguro a través de una red pública.*

![ Topología de Red ](ruta/a/tu/imagen_topologia.png)
*(Nota: Reemplaza "ruta/a/tu/imagen_topologia.png" con la ubicación real de tu imagen dentro de la carpeta "img")*

**Detalles de Direccionamiento:**
* **Redes Físicas (WAN):** 10.1.1.0/24 y 192.168.1.0/24
* **Red Virtual (Túnel):** 192.168.0.0/30
* **Redes LAN/Loopbacks:** 1.1.1.1/32, 2.2.2.2/32, 3.3.3.3/32 y 4.4.4.4/32

---

## ⚙️ Fases de Implementación

### Fase 1: Conectividad Base (Underlay)
Se configuró el direccionamiento IP de clase A y C (10.0.0.0 - 10.255.255.255 y 192.168.0.0 - 192.168.255.255) con un prefijo /24 para tener un buen margen de rango de hosts. Luego, en los routers intermedios (ISP), se configuró el enrutamiento y se definieron rutas estáticas por defecto desde los routers de México y Alemania hacia las puertas de enlace del ISP.

### Fase 2: Construcción del Túnel (Overlay)
Se configuraron las direcciones Loopback en cada uno de los routers asignando IPs para un mejor entendimiento e identificación de los nodos. En el caso de la red virtual, el protocolo GRE se levantó gracias a la dirección `192.168.0.0 /30`, dejando únicamente espacio para dos hosts (extremo a extremo) y al habilitar el enrutamiento estático hacia la interfaz, el protocolo cambió de estado dejando un exitoso `up/up`.

### Fase 3: Políticas de Seguridad (IPsec)
Considerando la importancia de IPsec como protocolo de seguridad, se asume que Internet es inseguro. Bajo esta premisa, se implementó una protección que garantiza la seguridad del túnel de extremo a extremo. Para proveer confidencialidad, integridad y autenticación, se realizó un despliegue que incluye:
* Encriptación AES y autenticación mediante Pre-Shared Key.
* Definición del grupo criptográfico Diffie-Hellman 2.
* Configuración del Transform-set y aplicación del Crypto Map atado a las ACLs que permiten el tráfico cifrado desde GRE.

---

## 🕵️‍♂️ Troubleshooting y Retos Superados
> *Incidentes críticos identificados y resueltos mediante diagnóstico de sistema operativo durante el despliegue.*

* **Incidencia:** El túnel IPsec se quedaba en estado `ACTIVE (deleted)` y los pings arrojaban *Timeout* o *Unreachable* (`U.U.U`).
* **Diagnóstico:** Mediante el comando `debug crypto isakmp` y rastreo de ICMP, se detectó que los routers del ISP descartaban los paquetes de Fase 1 al carecer de rutas hacia las redes físicas de los extremos.
* **Solución:** Se aislaron las capas separando el ruteo físico del virtual. Se inyectaron rutas estáticas en los routers intermedios (ISP) y se ajustó la ACL criptográfica a `permit ip host host`, forzando el inicio de la Fase 2 inyectando tráfico desde la Loopback.

---

## ✅ Validación y Resultados (Proof of Concept)
Se anexa la evidencia en consola (CLI) validando la encapsulación, encriptación y el correcto establecimiento de las adyacencias.

**1. Tráfico Interesante (Detonador del Túnel):**
```bash
R1# ping 4.4.4.4 source loopback 0

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 4.4.4.4, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/2/4 ms

R1# show crypto isakmp sa
IPv4 Crypto ISAKMP SA
dst             src             state          conn-id slot status
192.168.1.2     10.1.1.2        QM_IDLE           1075    0 ACTIVE

Verificación de Fase 2 (Data Tunnel y Encriptación):

R1# show crypto ipsec sa

interface: FastEthernet0/0
    Crypto map tag: VPN, local addr 10.1.1.2

    protected vrf: (none)
    local  ident (addr/mask/prot/port): (10.1.1.2/255.255.255.255/0/0)
    remote ident (addr/mask/prot/port): (192.168.1.2/255.255.255.255/0/0)
    current_peer 192.168.1.2 port 500
      PERMIT, flags={origin_is_acl,}
    #pkts encaps: 19, #pkts encrypt: 19, #pkts digest: 0
    #pkts decaps: 19, #pkts decrypt: 19, #pkts verify: 0


Conclusión: El tráfico viaja exitosamente a través del ISP y está siendo encriptado por el Crypto Map antes de salir por la interfaz física.




