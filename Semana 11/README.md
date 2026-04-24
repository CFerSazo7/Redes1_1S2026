# Manual de Configuración — Semana 11
## Topología: EIGRP + OSPF + VLANs + ACL

---

## 1. Descripción General de la Topología

La red está dividida en dos zonas de enrutamiento interconectadas:

- **Zona EIGRP** — Manejada por MS1, MS2 y MS3 (switches multicapa)
- **Zona OSPF** — Manejada por R1 y R2 (routers 2901)

La redistribución entre ambos protocolos se realiza en **MS3**, que actúa como punto frontera. Adicionalmente se configuró **HSRP** en R1 y R2 para alta disponibilidad en VLAN 30, y una **ACL extendida** en MS3 para bloquear tráfico de VLAN 10 hacia VLAN 30.

### Tabla de redes

| Red | Propósito |
|-----|-----------|
| 10.0.0.0/30 | Enlace MS2 ↔ MS1 |
| 10.0.0.4/30 | Enlace MS1 ↔ MS3 |
| 10.0.0.8/30 | Enlace MS2 ↔ MS3 |
| 10.0.0.12/30 | Enlace MS3 ↔ R1 |
| 10.0.0.16/30 | Enlace MS3 ↔ R2 |
| 192.168.10.0/24 | VLAN 10 — PC0 |
| 192.168.20.0/24 | VLAN 20 — PC2 |
| 192.168.30.0/24 | VLAN 30 — PC1 |

---

## 2. Configuración VTP y VLANs

Los tres switches de acceso (Switch0, Switch2, Switch3) se configuraron con VTP para sincronizar las VLANs automáticamente.

### Switch0, Switch2, Switch3 — Servidor VTP

```bash
vtp domain final
vtp version 2
vtp password 123
vtp mode server

vlan 10
 name VLAN10
vlan 20
 name VLAN20
vlan 30
 name VLAN30
```

### MS1 y MS2 — Clientes VTP

MS1 y MS2 se configuraron como clientes VTP. El trunkeo se habilitó con encapsulación dot1Q desde estas interfaces hacia los switches de acceso.

```bash
vtp domain final
vtp version 2
vtp password 123
vtp mode client
```

### Configuración de trunk hacia switches de acceso

```bash
! Ejemplo en MS2 — puerto hacia Switch0 (VLAN 10)
interface FastEthernet0/3
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 10
 switchport mode trunk

! Ejemplo en MS1 — puerto hacia Switch2 (VLAN 20)
interface FastEthernet0/3
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 20
 switchport mode trunk
```

---

## 3. Configuración MS2

MS2 actúa como switch multicapa de la zona EIGRP. Maneja VLAN 10 y conecta con MS1 y MS3.

```bash
ip routing

! Enlace hacia MS1
interface FastEthernet0/1
 no switchport
 ip address 10.0.0.1 255.255.255.252

! Enlace hacia MS3
interface FastEthernet0/2
 no switchport
 ip address 10.0.0.9 255.255.255.252

! Trunk hacia Switch0 (VLAN 10)
interface FastEthernet0/3
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 10
 switchport mode trunk

! SVI VLAN 10 — Gateway para PC0
interface Vlan10
 ip address 192.168.10.1 255.255.255.0

! EIGRP
router eigrp 100
 network 10.0.0.0 0.0.0.3
 network 10.0.0.8 0.0.0.3
 network 192.168.10.0
 auto-summary
```

---

## 4. Configuración MS1

MS1 actúa como switch multicapa de la zona EIGRP. Maneja VLAN 20 y conecta con MS2 y MS3.

```bash
ip routing

! Enlace hacia MS2
interface FastEthernet0/1
 no switchport
 ip address 10.0.0.2 255.255.255.252

! Enlace hacia MS3
interface FastEthernet0/2
 no switchport
 ip address 10.0.0.6 255.255.255.252

! Trunk hacia Switch2 (VLAN 20)
interface FastEthernet0/3
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 20
 switchport mode trunk

! SVI VLAN 20 — Gateway para PC2
interface Vlan20
 ip address 192.168.20.1 255.255.255.0

! EIGRP
router eigrp 100
 network 10.0.0.0 0.0.0.3
 network 10.0.0.4 0.0.0.3
 network 192.168.20.0
 auto-summary
```

---

## 5. Configuración MS3

MS3 es el punto frontera entre EIGRP y OSPF. Conecta con MS1, MS2, R1 y R2. Aquí también se aplica la ACL.

### Interfaces

```bash
ip routing

! Enlace hacia MS1
interface FastEthernet0/1
 no switchport
 ip address 10.0.0.5 255.255.255.252

! Enlace hacia MS2
interface FastEthernet0/2
 no switchport
 ip address 10.0.0.10 255.255.255.252

! Enlace hacia R1
interface FastEthernet0/3
 no switchport
 ip address 10.0.0.13 255.255.255.252

! Enlace hacia R2
interface FastEthernet0/4
 no switchport
 ip address 10.0.0.17 255.255.255.252
```

### OSPF

```bash
router ospf 1
 network 10.0.0.12 0.0.0.3 area 0
 network 10.0.0.16 0.0.0.3 area 0
```

### EIGRP

```bash
router eigrp 100
 network 10.0.0.4 0.0.0.3
 network 10.0.0.8 0.0.0.3
```

### Redistribución entre EIGRP y OSPF

```bash
! Redistribuir rutas EIGRP hacia OSPF
router ospf 1
 redistribute eigrp 100 subnets

! Redistribuir rutas OSPF hacia EIGRP
router eigrp 100
 redistribute ospf 1 metric 10000 100 255 1 1500
```

> **Nota:** Los parámetros del metric en EIGRP representan: bandwidth, delay, reliability, load, MTU.

---

## 6. Configuración R1

R1 conecta la zona OSPF con VLAN 30 usando una subinterfaz dot1Q. También implementa HSRP como router activo.

> **Importante:** La subinterfaz debe tener configurado `encapsulation dot1Q` **antes** de asignar la IP, de lo contrario Cisco rechaza el comando.

```bash
! Enlace hacia MS3
interface GigabitEthernet0/0
 ip address 10.0.0.14 255.255.255.252
 no shutdown

! Subinterfaz para VLAN 30
interface GigabitEthernet0/1
 no shutdown

interface GigabitEthernet0/1.30
 encapsulation dot1Q 30
 ip address 192.168.30.4 255.255.255.0

! HSRP — R1 será el router activo
 standby 10 ip 192.168.30.1

! OSPF
router ospf 1
 network 10.0.0.12 0.0.0.3 area 0
 network 192.168.30.0 0.0.0.255 area 0
```

### Verificación R1

```bash
R1# show ip route
R1# show standby brief
```

---

## 7. Configuración R2

R2 tiene la misma configuración que R1, usando la red `10.0.0.16/30` como enlace hacia MS3 y actuando como router en **espera** en HSRP.

```bash
! Enlace hacia MS3
interface GigabitEthernet0/0
 ip address 10.0.0.18 255.255.255.252
 no shutdown

! Subinterfaz para VLAN 30
interface GigabitEthernet0/1
 no shutdown

interface GigabitEthernet0/1.30
 encapsulation dot1Q 30
 ip address 192.168.30.5 255.255.255.0

! HSRP — R2 será el router en espera
 standby 10 ip 192.168.30.1

! OSPF
router ospf 1
 network 10.0.0.16 0.0.0.3 area 0
 network 192.168.30.0 0.0.0.255 area 0
```

> **Resultado HSRP:** La IP virtual `192.168.30.1` es el gateway de PC1. Si R1 falla, R2 asume automáticamente el rol activo sin interrupción para el usuario.

---

## 8. ACL Extendida — Bloqueo VLAN 10 → VLAN 30

La ACL se configura en **MS3** para bloquear el tráfico originado en VLAN 10 con destino a VLAN 30, permitiendo el resto del tráfico normalmente.

### ¿Por qué en MS3?

MS3 es el único punto por donde el tráfico de la zona EIGRP puede salir hacia la zona OSPF (VLAN 30). Aplicar la ACL aquí garantiza que **cualquier camino** hacia VLAN 30 quede controlado.

### Configuración

```bash
! Crear la ACL extendida nombrada
ip access-list extended BLOQUEO_VLAN10
 deny ip 192.168.10.0 0.0.0.255 192.168.30.0 0.0.0.255
 permit ip any any

! Aplicar en las interfaces de salida hacia la zona OSPF
interface FastEthernet0/3
 ip access-group BLOQUEO_VLAN10 out

interface FastEthernet0/4
 ip access-group BLOQUEO_VLAN10 out
```

### ¿Por qué `out` en Fa0/3 y Fa0/4?

| Interfaz | Conecta con | Dirección ACL |
|----------|-------------|---------------|
| Fa0/3 | R1 | out |
| Fa0/4 | R2 | out |

Aplicar `out` significa que la ACL revisa el tráfico **justo antes de salir** por esa interfaz hacia la zona OSPF. Si solo se aplicara en Fa0/3, el tráfico podría salir por Fa0/4 evadiendo el filtro.

### Verificación

```bash
MS3# show access-lists

! Resultado esperado:
! Extended IP access list BLOQUEO_VLAN10
!     10 deny ip 192.168.10.0 0.0.0.255 192.168.30.0 0.0.0.255
!     20 permit ip any any
```

---

## 9. Tabla de pruebas

| # | Origen | Destino | Resultado esperado |
|---|--------|---------|-------------------|
| 1 | PC0 — 192.168.10.x | PC1 — 192.168.30.x | ❌ Bloqueado por ACL |
| 2 | PC2 — 192.168.20.x | PC1 — 192.168.30.x | ✅ Permitido |
| 3 | PC0 — 192.168.10.x | PC2 — 192.168.20.x | ✅ Permitido (no afectado) |
| 4 | Apagar R1 → ping PC1 | Cualquier PC | ✅ R2 asume rol activo HSRP |

---

## 10. Errores comunes documentados

| Error | Causa | Solución |
|-------|-------|----------|
| `Invalid input` al poner IP en subinterfaz | Faltaba `encapsulation dot1Q` | Configurar encapsulation antes de la IP |
| ACL no filtra todo el tráfico | Solo aplicada en Fa0/3 | Aplicar también en Fa0/4 |
| `deny any` bloquea todo | Falta `permit ip any any` al final | Siempre agregar permit al final |
| EIGRP no aprende rutas OSPF | Falta redistribución bidireccional | Configurar redistribute en ambos protocolos |
