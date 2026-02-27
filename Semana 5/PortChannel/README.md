# Configuración de Port-Channel (EtherChannel)

## Descripción

Port-Channel (EtherChannel) permite agrupar varios enlaces físicos en **un solo enlace lógico**, incrementando ancho de banda y redundancia. Para STP, el Port-Channel se ve como **un único enlace**, reduciendo bloqueos innecesarios.

**VLANs de ejemplo**

* VLAN 10 → Ventas
* VLAN 20 → Compras

**Escenario**

* Enlace entre dos switches: **SW1 ↔ SW2**
* Puertos físicos a agrupar: ejemplo **Fa0/1 y Fa0/2**
* Port-Channel: **Po1**

---

## Reglas para que levante (muy importante)

* Los puertos deben tener **misma velocidad/duplex** y ser del **mismo tipo** (no mezclar Fa con Gi).
* La configuración debe ser **igual en ambos lados**:
  * trunk/access
  * allowed VLANs
* En Packet Tracer, si algo se “suspende”, resetea interfaces y vuelve a crear el grupo.

---

# Opción A — LACP (IEEE, recomendado)

## SW1 (LACP Active)

```bash
enable
conf t

interface range g0/1 - 2
 channel-protocol lacp
 channel-group 1 mode active
 no shutdown
exit

interface port-channel 1
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 no shutdown
end
wr
```

## SW2 (LACP Passive)

```bash
enable
conf t

interface range g0/1 - 2
 channel-protocol lacp
 channel-group 1 mode passive
 no shutdown
exit

interface port-channel 1
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 no shutdown
end
wr
```

**Combinación válida:** `active + passive` (o `active + active`)
**No forma:** `passive + passive`

---

# Opción B — PAgP (Cisco)

## SW1 (PAgP Desirable)

```bash
enable
conf t

interface range g0/1 - 2
 channel-protocol pagp
 channel-group 1 mode desirable
 no shutdown
exit

interface port-channel 1
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 no shutdown
end
wr
```

## SW2 (PAgP Auto)

```bash
enable
conf t

interface range g0/1 - 2
 channel-protocol pagp
 channel-group 1 mode auto
 no shutdown
exit

interface port-channel 1
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 no shutdown
end
wr
```

**Combinación válida:** `desirable + auto` (o `desirable + desirable`)
**No forma:** `auto + auto`

---

# Verificación (para ambos)

## 1. Ver estado del EtherChannel

```bash
show etherchannel summary
```

## 2. Ver que el Port-Channel está en trunk

```bash
show interfaces port-channel 1 trunk
```

## 3. Ver vecinos LACP (solo si usaste LACP)

```bash
show lacp neighbor
```

## 4. Ver STP sobre el Port-Channel

```bash
show spanning-tree
```

STP debería ver **Po1** como un solo enlace lógico (y no cada cable por separado).

---

# Troubleshooting rápido (si no levanta)

## Reset limpio (recomendado)

En ambos switches:

```bash
conf t
default interface range g0/1 - 2
no interface port-channel 1
end
wr
```

Luego repite la configuración.

---
