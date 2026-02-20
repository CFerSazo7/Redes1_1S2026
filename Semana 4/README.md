# Configuración de VLANs + VTP (Semana 4) + Trunks/Access (Cisco IOS)

> Objetivo: Crear 3 VLANs (Ventas=10, Admin=20, Soporte=30), configurar **1 switch Server VTP** y **3 switches Client VTP** en el dominio **semana4**, **VTP v2**, contraseña **123**, y dejar ejemplos de **puertos trunk** (Fa0/2) y **puertos access**.

---

## VLANs a crear
- **VLAN 10** → `VENTAS`
- **VLAN 20** → `ADMIN`
- **VLAN 30** → `SOPORTE`

---

## Switch SERVER (VTP Server)

### Configurar VTP (Server)
```bash
enable
configure terminal

vtp version 2
vtp domain semana4
vtp password 123
vtp mode server
```

### Crear VLANS
```bash
enable
configure terminal

vlan 10
 name VENTAS
exit

vlan 20
 name ADMIN
exit

vlan 30
 name SOPORTE
exit
```

### Ejemplo puerto TRUNK
```bash
enable
configure terminal

interface fa0/2
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
 no shutdown
exit
```

### Configurar VTP (Client)
```bash
enable
configure terminal

vtp domain semana4
vtp version 2
vtp mode client
vtp password 123
```

### Ejemplo puerto ACCESS
```bash
enable
configure terminal

interface fa0/3
 description PC Ventas
 switchport mode access
 switchport access vlan 10
 no shutdown
exit
```

## Comandos de Verificacion
```bash
show vtp status
show vlan brief
show interfaces trunk
```