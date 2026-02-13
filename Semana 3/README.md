# Semana 3

## Configuraciones Basicas

## 1. Descripción de la Topología

La práctica se desarrolló con dos segmentos de red:

### 1.1 Departamento de Ingeniería

* 1 Hub
* 3 dispositivos finales (PC1, PC2, Laptop)
* Conexión hacia un switch central

Todos los dispositivos conectados comparten el mismo dominio de colisión.
El tráfico recibido en un puerto es replicado hacia todos los demás puertos.

---

### 1.2 Departamento de Auditoría

* 1 Switch
* 1 PC
* 2 Servidores
* Conexión hacia un switch central

Cada puerto representa un dominio de colisión independiente.
El Switch utiliza una tabla MAC para reenviar el tráfico únicamente al puerto correspondiente al destino.

---

## 2. Configuración Realizada en los Switches

La configuración aplicada fue básica e incluyó:

* Acceso al modo privilegiado
* Acceso al modo de configuración global
* Cambio de hostname
* Configuración de contraseña segura

---

## 3. Comandos Utilizados

### 3.1 Acceso al modo privilegiado

```bash
enable
```

Permite cambiar del modo usuario (`>`) al modo privilegiado (`#`).

---

### 3.2 Acceso al modo de configuración global

```bash
configure terminal
```

Permite ingresar al modo de configuración para modificar parámetros del dispositivo.

---

### 3.3 Configuración del hostname

Para el switch del departamento de Ingeniería:

```bash
hostname ingenieria
```

Para el switch del departamento de Auditoría:

```bash
hostname auditoria
```

Este comando cambia el nombre del dispositivo en la línea de comandos.

---

### 3.4 Configuración de contraseña privilegiada

```bash
enable secret 123
```

El comando `enable secret` establece una contraseña cifrada para el acceso al modo privilegiado.

---

### 3.5 Verificación de la configuración

```bash
show run
```

Muestra la configuración actual almacenada en la memoria RAM del dispositivo.

Ejemplo observado:

```
hostname ingenieria
spanning-tree mode pvst
```

---

## 4. Configuración de Direccionamiento IP

Se asignaron direcciones IP manualmente a los dispositivos finales.

### Red utilizada

* Dirección de red: 192.168.1.0/24
* Máscara de subred: 255.255.255.0

### Direcciones asignadas

| Dispositivo | Dirección IP |
| ----------- | ------------ |
| PC1         | 192.168.1.2  |
| PC2         | 192.168.1.3  |
| PC3         | 192.168.1.4  |
| PC4         | 192.168.1.5  |
| Server1     | 192.168.1.6  |
| Server2     | 192.168.1.7  |

Todos los dispositivos pertenecen al mismo segmento de red.

---

## 5. Diferencias Técnicas Observadas

| Característica      | Hub                 | Switch          |
| ------------------- | ------------------- | --------------- |
| Dominio de colisión | Uno compartido      | Uno por puerto  |
| Reenvío de tráfico  | A todos los puertos | Solo al destino |
| Inteligencia        | No                  | Sí (tabla MAC)  |
