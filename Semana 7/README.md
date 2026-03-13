# Semana 7 - Ruteo estático

## ¿Qué es el ruteo estático?

El ruteo estático es cuando el administrador le indica manualmente a cada router cómo llegar a redes remotas. El router no aprende solo — se le programa el camino.

### ¿Cuándo usarlo?
- Redes pequeñas y estables
- Cuando se necesita control absoluto sobre el tráfico
- En enlaces específicos donde no se quiere overhead de protocolos dinámicos

---

## Conceptos clave

### Tabla de ruteo
Cada router solo conoce las redes **directamente conectadas**. Sin rutas estáticas, no sabe que existen otras redes más allá de sus interfaces.

### Next-hop
El router no necesita conocer el camino completo al destino. Solo necesita saber a quién pasarle el paquete — su vecino inmediato (next-hop) se encarga del resto.

### Simetría
La comunicación requiere ida **y** vuelta. Si solo se configura una dirección, el paquete llega pero la respuesta se pierde. Todos los routers en el camino deben tener rutas en ambos sentidos.

### Default Gateway en PCs
Los dispositivos finales (PCs) también necesitan saber a quién entregar los paquetes fuera de su red. La default gateway es la IP del router en **la misma red que la PC**.

> La PC detecta si el destino está en su red o no. Si está fuera, lo entrega al gateway y el router resuelve el resto.

---

## Configuración de interfaces

Antes de configurar rutas, cada interfaz del router debe tener su IP asignada.

```
Router# enable
Router# configure terminal

Router(config)# interface gigabitEthernet 0/0
Router(config-if)# ip address 192.168.1.1 255.255.255.0
Router(config-if)# no shutdown
Router(config-if)# exit

Router(config)# interface gigabitEthernet 0/1
Router(config-if)# ip address 11.0.0.2 255.255.255.252
Router(config-if)# no shutdown
Router(config-if)# end
```

> **Nota:** Usa `gigabitEthernet` o `fastEthernet` según el modelo del router. El comando `no shutdown` activa la interfaz — por defecto están apagadas.

---

## Rutas estáticas

### Sintaxis del comando

```
ip route [red_destino] [mascara] [next-hop]
```

| Parámetro | Descripción |
|---|---|
| `red_destino` | La red a la que se quiere llegar |
| `mascara` | Máscara de subred de esa red destino |
| `next-hop` | IP del vecino inmediato que conoce el camino |

### Configuración Router0

```
Router0(config)# ip route 10.0.0.0 255.255.255.0 11.0.0.1
Router0(config)# ip route 192.167.1.0 255.255.255.252 11.0.0.1
```

> Router0 no conoce la LAN del otro lado ni el enlace WAN entre Router2 y Router1. Se le indica que para llegar a ambas, el next-hop es `11.0.0.1` (Router2).

### Configuración Router2

```
Router2(config)# ip route 192.168.1.0 255.255.255.0 11.0.0.2
Router2(config)# ip route 10.0.0.0 255.255.255.0 192.167.1.2
```

> Router2 está en el centro — necesita rutas hacia ambas LANs, cada una con su next-hop correspondiente.

### Configuración Router1

```
Router1(config)# ip route 192.168.1.0 255.255.255.0 192.167.1.1
Router1(config)# ip route 11.0.0.0 255.255.255.252 192.167.1.1
```

> Router1 necesita conocer la LAN de Router0 y también el enlace WAN entre Router0 y Router2 para que el tráfico de retorno funcione completo.

---

## Asignación de IPs en la topología

| Dispositivo | Interfaz | IP | Red |
|---|---|---|---|
| Router0 | Gi0/0 | 192.168.1.1/24 | LAN izquierda |
| Router0 | Gi0/1 | 11.0.0.2/30 | WAN → Router2 |
| Router2 | Gi0/0 | 11.0.0.1/30 | WAN → Router0 |
| Router2 | Gi0/1 | 192.167.1.1/30 | WAN → Router1 |
| Router1 | Gi0/0 | 192.167.1.2/30 | WAN → Router2 |
| Router1 | Gi0/1 | 10.0.0.1/24 | LAN derecha |

### Configuración de PCs

| PC | IP | Máscara | Default Gateway |
|---|---|---|---|
| PC0 | 192.168.1.10 | 255.255.255.0 | 192.168.1.1 |
| PC1 | 192.168.1.11 | 255.255.255.0 | 192.168.1.1 |
| PC2 | 10.0.0.10 | 255.255.255.0 | 10.0.0.1 |
| PC3 | 10.0.0.11 | 255.255.255.0 | 10.0.0.1 |

---

## Verificación

### Ver la tabla de ruteo

```
Router# show ip route
```

Las entradas se identifican con una letra al inicio:

| Código | Significado |
|---|---|
| `C` | Connected — red directamente conectada |
| `L` | Local — IP de la propia interfaz |
| `S` | Static — ruta estática configurada manualmente |

---

## Aclaraciones importantes

### Una interfaz = una red
Un router no puede tener dos interfaces en la misma red IP. Cada interfaz física o lógica debe pertenecer a una red diferente.

- Para conectar más dispositivos en la **misma red** → usar un **switch**
- Para conectar dispositivos en **redes diferentes** → usar una nueva **interfaz del router**

### Por qué falla el ping aunque "parezca" que la ruta existe

```
PC0 ──────────────────────→ Router2   ✅ llega
PC0 ←──────────────────────  Router2  ❌ se pierde (no hay ruta de regreso)
```

Un ping exitoso requiere que **ambos sentidos** estén configurados en todos los routers del camino.

### Default gateway vs ruta estática

| | ¿Quién lo usa? | ¿Para qué? |
|---|---|---|
| Default gateway | La PC | Entregar paquetes fuera de su red |
| Ruta estática | El router | Reenviar paquetes entre redes |

La default gateway es esencialmente la "ruta estática del dispositivo final" — todo lo que no sea su propia red, se lo entrega al gateway.

---

## Resumen de comandos

| Comando | Descripción |
|---|---|
| `enable` | Entrar a modo privilegiado |
| `configure terminal` | Entrar a modo de configuración global |
| `interface gi0/0` | Seleccionar una interfaz |
| `ip address X.X.X.X M.M.M.M` | Asignar IP y máscara |
| `no shutdown` | Activar la interfaz |
| `ip route RED MASCARA NEXTHOP` | Agregar ruta estática |
| `show ip route` | Ver tabla de ruteo |
| `show ip interface brief` | Ver estado de interfaces |
| `ping X.X.X.X` | Probar conectividad |
| `write memory` | Guardar configuración |