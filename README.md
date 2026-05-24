# Network Diagram — Packet Analysis

Interactive network graph visualization tool for packet analysis using JSON capture files.

---

## Overview

This project generates an interactive network topology graph from packet capture JSON files.

It visualizes:
- IP communications
- Protocol relationships
- Packet density
- Network interaction patterns

The graph is rendered dynamically using D3.js force simulation.

---

## Features

### Interactive Graph Visualization
- Dynamic force-directed graph
- Zoom and pan support
- Draggable nodes
- Real-time rendering

### Protocol Filtering
Supports filtering by protocol:
- ARP
- DHCP
- DNS
- HTTP
- ICMP
- MQTT
- QUIC
- TCP
- TLS

### Visual Analytics
- Node size based on packet count
- Connection thickness based on traffic volume
- Protocol color mapping
- Hover tooltips with packet statistics

### Customizable Simulation
Adjustable parameters:
- Label threshold
- Repulsion force
- Link distance
- Node size multiplier
- Link strength

### Drag & Drop Interface
Simply drag JSON packet files into the browser window.

---

## Technologies Used

- HTML5
- CSS3
- JavaScript (Vanilla)
- D3.js v7

---

## Requirements

This application runs entirely in the browser.

### Minimum Requirements
- Modern web browser
  - Chrome
  - Firefox
  - Edge
  - Brave

### No Backend Required
The application is fully client-side.

No:
- server
- database
- API
- installation

required.

---

## Running the Project

### Option 1 — Open Directly

Simply open:

```bash
network-graph.html
