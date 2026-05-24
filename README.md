# Network Diagram — Packet Analysis

Visualizador interactivo de tráfico de red basado en grafos utilizando D3.js.

---

# Descripción

**Network Diagram — Packet Analysis** es una aplicación web completamente cliente (frontend puro) diseñada para visualizar comunicaciones de red a partir de archivos JSON de capturas de paquetes.

La aplicación analiza automáticamente relaciones entre direcciones IP y genera un grafo dinámico e interactivo donde:

- cada nodo representa una IP,
- cada conexión representa tráfico entre dos hosts,
- el tamaño de los nodos representa actividad,
- el grosor de las conexiones representa volumen de comunicación,
- y los colores representan protocolos detectados.

Todo el procesamiento ocurre localmente en el navegador, sin servidores, sin backend y sin transmisión externa de datos.

---

# Arquitectura General

La aplicación sigue este flujo:

```text
Arrastrar archivos JSON
        ↓
Lectura mediante FileReader
        ↓
Parseo de paquetes
        ↓
Extracción de IPs y protocolos
        ↓
Construcción del grafo
        ↓
Simulación física con D3.js
        ↓
Renderizado interactivo
```

## Tecnologías utilizadas

- HTML5
- CSS3
- JavaScript Vanilla
- D3.js v7

---

# Características Principales

## Visualización interactiva

- Grafo dinámico en tiempo real
- Zoom y desplazamiento
- Nodos arrastrables
- Tooltips informativos
- Simulación física avanzada

---

## Filtros por protocolo

Protocolos soportados:

- ARP
- DHCP
- DNS
- HTTP
- ICMP
- MQTT
- QUIC
- TCP
- TLS

Cada protocolo tiene:
- color propio,
- filtrado independiente,
- estadísticas visuales.

---

## Configuración dinámica

La simulación puede ajustarse en tiempo real mediante sliders:

- Umbral mínimo para mostrar etiquetas
- Fuerza de repulsión
- Distancia entre nodos
- Tamaño de nodos
- Fuerza de enlaces

Cada cambio reconstruye completamente la simulación física.

---

# Estructura de los JSON

La aplicación es flexible con los nombres de campos.

## Campos soportados para IP origen/destino

Acepta cualquiera de estas variantes:

| Campo origen | Campo destino |
|---|---|
| `ip_origen` | `ip_destino` |
| `IP origen` | `IP destino` |

---

## JSON mínimo válido

El requisito real mínimo es:

```json
[
  {
    "ip_origen": "192.168.1.1",
    "ip_destino": "10.0.0.5"
  }
]
```

Si un paquete no contiene ambos campos IP válidos:
- el paquete se ignora,
- no rompe la aplicación,
- simplemente no se renderiza.

---

# Detección de protocolos

La aplicación usa un sistema de fallbacks para detectar protocolos.

## Orden de prioridad

### 1. layer_name

```json
{
  "layer_name": "tcp"
}
```

---

### 2. db_name

```json
{
  "db_name": "dns.log"
}
```

Se elimina automáticamente `.log`.

---

### 3. Nombre del archivo

Si el archivo comienza por:

```text
ARP
DHCP
DNS
HTTP
ICMP
MQTT
QUIC
TCP
TLS
```

el protocolo se detecta automáticamente.

Ejemplos válidos:

```text
TCP_capture.json
DNS_packets.json
DHCP_data.json
```

---

### 4. Fallback final

Si nada coincide:

```text
unknown
```

---

# Cómo funciona realmente el parser

La aplicación no usa el nombre real del archivo directamente.

Cuando haces drag & drop:

```js
const proto = file.name.match(/^(ARP|DHCP|DNS|HTTP|ICMP|MQTT|QUIC|TCP|TLS)/i);
```

Extrae el protocolo del nombre del archivo y lo usa como clave interna.

Por ejemplo:

```text
DHCP_capture.json
```

se convierte internamente en:

```text
DHCP
```

Luego `getProtocol()` reutiliza esa clave para clasificar paquetes incluso si:
- no existe `layer_name`,
- no existe `db_name`.

Por eso DHCP funciona correctamente aunque el JSON no tenga información explícita del protocolo.

---

# Construcción del Grafo

La función principal:

```js
processData(fileMap)
```

convierte paquetes en estructuras de nodos y conexiones.

---

# Cómo se crean las conexiones

Cada paquete genera o incrementa una arista.

Clave interna:

```js
srcIP|dstIP
```

o:

```js
srcIP->dstIP|protocol
```

Ejemplo:

```js
{
  source: "192.168.1.1",
  target: "10.0.0.5",
  count: 152,
  proto: "tcp"
}
```

Cada nuevo paquete entre las mismas IPs incrementa:

```js
count++
```

---

# Cómo se crean los nodos

Cada IP se convierte en un nodo.

La aplicación calcula:

- cantidad total de paquetes,
- protocolos asociados,
- densidad de conexiones.

---

# Tamaño de nodos

Los nodos más activos son más grandes.

Cálculo:

```js
radius = 6 + (packets / maxPackets) * nodeSize
```

---

# Grosor de conexiones

Las conexiones con más tráfico son más gruesas.

Cálculo:

```js
strokeWidth = 1 + (count / maxEdge) * 4
```

---

# Simulación física con D3.js

La aplicación utiliza 4 fuerzas simultáneas.

| Fuerza | Función |
|---|---|
| `forceLink` | Une nodos conectados |
| `forceManyBody` | Repulsión global |
| `forceCenter` | Centra el grafo |
| `forceCollide` | Evita solapamientos |

---

# Actualización en tiempo real

Cada modificación del usuario ejecuta:

```js
onConfigChange() → renderGraph()
```

Esto:
- destruye la simulación actual,
- recalcula fuerzas,
- reconstruye nodos,
- y vuelve a renderizar el grafo completo.

---

# Interfaz

## Drop Zone

Zona para arrastrar archivos JSON.

---

## Toolbar

Filtros de protocolos.

---

## Panel de configuración

Permite modificar:
- repulsión,
- distancia,
- tamaños,
- umbrales,
- fuerza de enlaces.

---

## Tooltip

Al pasar el ratón sobre un nodo muestra:
- IP,
- cantidad de paquetes,
- protocolos detectados.

---

## Estadísticas

Muestra:
- número de nodos,
- conexiones,
- protocolos activos.

---

# Rendimiento

El coste computacional aumenta rápidamente con datasets grandes porque:

- todos los nodos participan en simulación física,
- existe colisión continua,
- las fuerzas se recalculan constantemente.

## Recomendaciones

- prefiltrar capturas muy grandes,
- dividir datasets,
- evitar millones de paquetes en navegador.

---

# Seguridad y privacidad

La aplicación funciona completamente offline en el navegador.

No:
- sube datos,
- usa APIs,
- transmite tráfico,
- ni almacena información externa.

Todo el análisis ocurre localmente.

---










