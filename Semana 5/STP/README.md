# Configuración de Spanning Tree Protocol (STP)

## Descripción

Este documento describe la configuración de STP en una topología con tres switches (Server Switch0, Cliente Switch1 y Cliente Switch2) utilizando VLANs y Rapid-PVST para prevenir bucles de Capa 2.

La implementación garantiza:

* Prevención de loops
* Elección controlada del Root Bridge
* Convergencia rápida
* Separación por VLAN

---

## Topología

* **Server Switch0** → Root Bridge principal
* **Cliente Switch1** → Switch de acceso
* **Cliente Switch2** → Switch de acceso
* VLANs utilizadas:

  * VLAN 10 → Ventas
  * VLAN 20 → Compras

---

# Paso 1 — Configuración global de STP

> Ejecutar en TODOS los switches

```bash
enable
conf t
spanning-tree mode rapid-pvst
end
wr
```

- Habilita Rapid-PVST
- Una instancia STP por VLAN
- Convergencia rápida

---

# Paso 2 — Creación de VLANs

> Ejecutar en todos los switches

```bash
conf t
vlan 10
 name VENTAS
vlan 20
 name COMPRAS
```

---

# Paso 3 — Configuración de puertos access

## Ejemplo VLAN 10 (Ventas)

```bash
conf t
interface fa0/1
 switchport mode access
 switchport access vlan 10
exit
```

## Ejemplo VLAN 20 (Compras)

```bash
conf t
interface fa0/2
 switchport mode access
 switchport access vlan 20
exit
wr
```

- PortFast acelera el estado forwarding en hosts finales.

---

# Paso 4 — Configuración de enlaces troncales

> Entre switches

```bash
conf t
interface g0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20
exit

interface g0/2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20
```

---

# Paso 5 — Forzar Root Bridge

> Ejecutar SOLO en **Server Switch0**

```bash
conf t
spanning-tree vlan 10 root primary
spanning-tree vlan 20 root primary
end
wr
```

- Define el switch como root para ambas VLANs
- Reduce la prioridad automáticamente

---

# Paso 6 — Verificación

## Ver quién es el root

```bash
show spanning-tree vlan 10
show spanning-tree vlan 20
```

Debe aparecer:

```
This bridge is the root
```

---

## Ver estado general

```bash
show spanning-tree summary
```

Debe indicar:

```
Spanning tree enabled protocol rstp
```

---

## Ver puertos bloqueados

```bash
show spanning-tree blockedports
```

Debe existir al menos un puerto en blocking debido al enlace redundante.

---

# Prueba de convergencia

1. Identificar puerto bloqueado
2. Apagar enlace activo:

```bash
conf t
interface g0/x
 shutdown
```

3. Observar que el puerto alterno pasa a **forwarding**

---

# Buenas prácticas aplicadas

* Rapid-PVST habilitado
* Root Bridge controlado
* PortFast en puertos de acceso
* Troncales correctamente definidos
* Prevención de loops validada

---

# Comandos útiles de diagnóstico

```bash
show spanning-tree
show spanning-tree vlan 10
show spanning-tree vlan 20
show spanning-tree blockedports
show spanning-tree summary
```

---

# Resultado esperado

* Server Switch0 actúa como Root Bridge
* STP elimina bucles
* Cada VLAN tiene su propio árbol
* La red converge correctamente ante fallos

