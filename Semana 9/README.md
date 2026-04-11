# Documentación de comandos de red

## Resumen de la topología trabajada

Se configuraron los siguientes equipos y esquemas de enrutamiento:

* **Router1**: Router-on-a-Stick para VLAN 10 y VLAN 20, con OSPF.
* **Router0**: OSPF para enlaces seriales.
* **Router2**: OSPF para enlaces seriales y enlace hacia MS2.
* **MS1**: EIGRP.
* **MS2**: Redistribución entre OSPF y EIGRP.
* **Router3**: Router-on-a-Stick para VLAN 10 y VLAN 20, con ruteo estático.
* **MS1**: rutas estáticas de retorno hacia las redes de Router3.

---

# 1. Router1

## Redes conectadas

* `10.0.0.0/30`
* `10.0.0.8/30`
* `192.168.1.0/27` → VLAN 10 Estudiantes
* `192.168.1.32/28` → VLAN 20 Docentes

## Configuración Router-on-a-Stick

```bash
enable
configure terminal

interface GigabitEthernet0/0
 no shutdown

interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.1.1 255.255.255.224

interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.1.33 255.255.255.240

end
write memory
```

## OSPF en Router1

```bash
enable
configure terminal

router ospf 1
 router-id 1.1.1.1
 network 10.0.0.0 0.0.0.3 area 0
 network 10.0.0.8 0.0.0.3 area 0
 network 192.168.1.0 0.0.0.31 area 0
 network 192.168.1.32 0.0.0.15 area 0

end
write memory
```

---

# 2. Router0

## Redes conectadas

* `10.0.0.0/30`
* `10.0.0.4/30`

## OSPF en Router0

```bash
enable
configure terminal

router ospf 1
 router-id 2.2.2.2
 network 10.0.0.0 0.0.0.3 area 0
 network 10.0.0.4 0.0.0.3 area 0

end
write memory
```

---

# 3. Router2

## Redes conectadas

* `10.0.0.4/30`
* `10.0.0.8/30`
* `10.0.0.12/30`

## OSPF en Router2

```bash
enable
configure terminal

router ospf 1
 router-id 3.3.3.3
 network 10.0.0.4 0.0.0.3 area 0
 network 10.0.0.8 0.0.0.3 area 0
 network 10.0.0.12 0.0.0.3 area 0

end
write memory
```

---

# 4. MS1

## Redes conectadas

* `10.0.0.16/30` → Port-channel1
* `10.0.0.24/30` → FastEthernet0/5

## EIGRP en MS1

> Se utilizó el AS **100**.

```bash
enable
configure terminal

router eigrp 100
 no auto-summary
 network 10.0.0.16 0.0.0.3
 network 10.0.0.24 0.0.0.3

end
write memory
```

## Rutas estáticas de retorno hacia Router3

Estas rutas se agregaron porque las PCs de Router3 podían llegar a Router3, pero no podían completar el ping hacia MS1. El problema era el camino de regreso desde MS1 hacia las redes de Router3.

```bash
enable
configure terminal

ip route 192.168.2.0 255.255.255.128 10.0.0.26
ip route 192.168.2.128 255.255.255.224 10.0.0.26

end
write memory
```

---

# 5. MS2

## Redes conectadas

* `10.0.0.12/30` → lado OSPF
* `10.0.0.16/30` → lado EIGRP

## Función

MS2 se configuró para realizar **redistribución entre OSPF y EIGRP**.

## Configuración completa en MS2

```bash
enable
configure terminal

router ospf 1
 router-id 4.4.4.4
 network 10.0.0.12 0.0.0.3 area 0
 redistribute eigrp 100 subnets

router eigrp 100
 no auto-summary
 network 10.0.0.16 0.0.0.3
 redistribute ospf 1 metric 100000 100 255 1 1500

end
write memory
```

## Explicación breve

* La red `10.0.0.12/30` participa en **OSPF**.
* La red `10.0.0.16/30` participa en **EIGRP**.
* MS2 redistribuye rutas aprendidas de un protocolo hacia el otro.

---

# 6. MS3 (antes Router3 - ahora Multicapa)

## Cambio realizado

Se reemplazó Router3 por un **switch multicapa (Layer 3)** para manejar VLANs mediante **SVI (interface vlan)** en lugar de subinterfaces.

## Redes conectadas

* `10.0.0.24/30`
* `192.168.2.0/25` → VLAN 10 Estudiantes
* `192.168.2.128/27` → VLAN 20 Docentes

## Configuración en Multicapa (SVI)

```bash
enable
configure terminal

ip routing

vlan 10
 name ESTUDIANTES

vlan 20
 name DOCENTES

interface vlan 10
 ip address 192.168.2.1 255.255.255.128
 no shutdown

interface vlan 20
 ip address 192.168.2.129 255.255.255.224
 no shutdown

interface GigabitEthernet0/0
 no switchport
 ip address 10.0.0.26 255.255.255.252
 no shutdown

end
write memory
```

## Rutas estáticas en MS3

> Se utilizó como siguiente salto `10.0.0.25`.

```bash
enable
configure terminal

ip route 10.0.0.0 255.255.255.252 10.0.0.25
ip route 10.0.0.4 255.255.255.252 10.0.0.25
ip route 10.0.0.8 255.255.255.252 10.0.0.25
ip route 10.0.0.12 255.255.255.252 10.0.0.25
ip route 10.0.0.16 255.255.255.252 10.0.0.25
ip route 192.168.1.0 255.255.255.224 10.0.0.25
ip route 192.168.1.32 255.255.255.240 10.0.0.25

end
write memory
```

## Nota

* Se reemplazó Router-on-a-Stick por **SVI**, lo cual es más eficiente.
* El **ruteo estático se mantiene exactamente igual**.
* Se debe habilitar `ip routing` para que el switch multicapa enrute.

---

# 7. Switch conectado a Router1

## VLANs y trunk

```bash
enable
configure terminal

vlan 10
 name ESTUDIANTES

vlan 20
 name DOCENTES

interface fa0/1
 switchport mode access
 switchport access vlan 10

interface fa0/2
 switchport mode access
 switchport access vlan 20

interface fa0/3
 switchport mode trunk
 switchport trunk allowed vlan 10,20

end
write memory
```

---

# 8. Gateways de las PCs

## PCs en Router1

### VLAN 10 - Estudiantes

* Red: `192.168.1.0/27`
* Gateway: `192.168.1.1`
* Máscara: `255.255.255.224`

### VLAN 20 - Docentes

* Red: `192.168.1.32/28`
* Gateway: `192.168.1.33`
* Máscara: `255.255.255.240`

## PCs en Router3

### VLAN 10 - Estudiantes

* Red: `192.168.2.0/25`
* Gateway: `192.168.2.1`
* Máscara: `255.255.255.128`

### VLAN 20 - Docentes

* Red: `192.168.2.128/27`
* Gateway: `192.168.2.129`
* Máscara: `255.255.255.224`

---

# 9. Comandos de verificación

## OSPF

```bash
show ip ospf neighbor
show ip route
show ip protocols
```

## EIGRP

```bash
show ip eigrp neighbors
show ip route
show ip protocols
```

## Interfaces

```bash
show ip interface brief
```

## Pruebas de conectividad

```bash
ping 192.168.1.1
ping 192.168.1.33
ping 192.168.2.1
ping 192.168.2.129
ping 10.0.0.25
ping 10.0.0.26
```

---

# 10. Conexiones punto a punto (P2P)

Se utilizaron enlaces /30 para interconectar routers y switches capa 3.

## Direccionamiento P2P

* R1 ↔ R0 → `10.0.0.0/30`
* R0 ↔ R2 → `10.0.0.4/30`
* R1 ↔ R2 → `10.0.0.8/30`
* R2 ↔ MS2 → `10.0.0.12/30`
* MS2 ↔ MS1 → `10.0.0.16/30`
* MS1 ↔ MS3 → `10.0.0.24/30`

## Ejemplo configuración interfaz (Router)

```bash
interface Serial0/1/0
 ip address 10.0.0.1 255.255.255.252
 no shutdown

interface Serial0/1/1
 ip address 10.0.0.9 255.255.255.252
 no shutdown
```

## Ejemplo configuración interfaz (Switch L3)

```bash
interface GigabitEthernet0/0
 no switchport
 ip address 10.0.0.26 255.255.255.252
 no shutdown
```

---

# 11. VLANs y VTP

## Creación de VLANs

```bash
enable
configure terminal

vlan 10
 name ESTUDIANTES

vlan 20
 name DOCENTES

end
write memory
```

---

## Configuración VTP (ejemplo)

### Switch VTP Server (MS1)

```bash
configure terminal
vtp mode server
vtp domain RED-GT
vtp password cisco
```

### Switch VTP Client (MS2, MS3)

```bash
configure terminal
vtp mode client
vtp domain RED-GT
vtp password cisco
```

---

## Configuración de Trunks

```bash
interface fa0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20
```

---

## Configuración de EtherChannel (LACP)

Se configuró un EtherChannel entre switches utilizando **LACP (Link Aggregation Control Protocol)** para aumentar ancho de banda y redundancia.

### Configuración en ambos switches

```bash
enable
configure terminal

# Router MS1

interface range fa0/1 - 2
 channel-protocol lacp
 channel-group 1 mode active

interface port-channel 1
 no switchport
 ip address 10.0.0.17 255.255.255.252
 no shutdown

# Router MS2

interface range fa0/1 - 2
 channel-protocol lacp
 channel-group 1 mode active

interface port-channel 1
 no switchport
 ip address 10.0.0.18 255.255.255.252
 no shutdown

end
write memory
```

### Explicación

* `channel-protocol lacp` → Define el uso de LACP
* `channel-group 1 mode active` → Activa la negociación LACP
* `Port-channel1` → Interfaz lógica resultante
* Se configura como trunk para transportar VLANs

---

## Configuración de puertos de acceso

```bash
interface fa0/2
 switchport mode access
 switchport access vlan 10

interface fa0/3
 switchport mode access
 switchport access vlan 20
```

---

