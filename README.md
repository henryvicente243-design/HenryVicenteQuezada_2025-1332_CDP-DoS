# Ataque DoS mediante el Protocolo CDP
Nombre: Henry Vicente Quezada | Matrícula: 2025-1332 | Fecha: 06 de Junio 2026

---

## 🎬 Video Demostrativo
 https://youtu.be/AZFYsvABjY4 
 
---

## 1. Objetivo del Laboratorio
Demostrar cómo el protocolo CDP (Cisco Discovery Protocol) puede ser explotado para realizar un ataque de Denegación de Servicio (DoS) contra switches Cisco, saturando su tabla de vecinos CDP con entradas falsas, consumiendo memoria y CPU del dispositivo, y aplicar la contramedida correspondiente.

---

## 2. Objetivo del Script
Generar una inundación masiva de paquetes CDP falsos con Device IDs y MACs aleatorias hacia el switch víctima, con el propósito de saturar su tabla de vecinos CDP.

### 2.1 Parámetros Usados
| Parámetro | Descripción | Valor por defecto |
|---|---|---|
| `interfaz` | Interfaz de red del atacante | Requerido (ej: `eth0`) |
| `cantidad` | Número de paquetes CDP a enviar | 1000 |

### 2.2 Requisitos
- Sistema operativo: **Kali Linux**
- Python 3.x
- Librería Scapy: `pip install scapy`
- Módulo CDP incluido en Scapy
- Permisos de root: `sudo`
- Conectividad con el switch objetivo

---

## 3. Funcionamiento del Script
1. Genera una MAC de origen aleatoria por cada paquete
2. Construye un paquete CDP válido con Device ID aleatorio
3. Encapsula en trama Ethernet con destino multicast CDP `01:00:0c:cc:cc:cc`
4. Envía el paquete por la interfaz especificada
5. Repite el proceso la cantidad de veces indicada
6. Muestra el progreso cada 100 paquetes enviados

```bash
sudo python3 cdp_dos.py <interfaz> [cantidad]
sudo python3 cdp_dos.py eth0 500
```

---

## 4. Documentación de la Red

### Topología
<img width="627" height="533" alt="image" src="https://github.com/user-attachments/assets/d860cb35-146a-4fd5-a443-11bf825aed55" />


### Tabla de Direccionamiento
| Dispositivo | Interfaz | VLAN | IP | Máscara | Rol |
|---|---|---|---|---|---|
| R1 | e0/0.10 | 10 | 10.13.32.1 | /24 | Gateway VLAN10 TI |
| R1 | e0/0.20 | 20 | 10.13.33.1 | /24 | Gateway VLAN20 Gerencia |
| R1 | e0/0.99 | 99 | 10.13.99.1 | /24 | Gateway Management |
| SW1 | vlan 99 | 99 | 10.13.99.2 | /24 | Gestión Switch |
| VPC10 | eth0 | 10 | 10.13.32.11 | /24 | Cliente VLAN10 |
| VPC20 | eth0 | 20 | 10.13.33.11 | /24 | Cliente VLAN20 |
| Kali | eth0 | 10 | 10.13.32.5 | /24 | **Atacante** |

### VLANs
| VLAN | Nombre | Descripción |
|---|---|---|
| 10 | TI | Red de usuarios TI |
| 20 | Gerencia | Red de Gerencia |
| 99 | Management | Red de gestión de dispositivos |

### Interfaces SW1
| Puerto | Modo | VLAN | Conectado a |
|---|---|---|---|
| e0/0 | Trunk 802.1q | 10,20,99 | R1 |
| e0/1 | Access | 10 | VPC10 |
| e0/2 | Access | 20 | VPC20 |
| e0/3 | Access | 10 | **Kali Linux** |

---

## 5. Capturas de Pantalla

### Antes del ataque
<img width="611" height="429" alt="image" src="https://github.com/user-attachments/assets/d224a82b-d4aa-4c69-a3ea-6bcf8451bd71" />

### Script en ejecución
<img width="628" height="426" alt="image" src="https://github.com/user-attachments/assets/8c0170c2-e9c7-4b2f-9a1b-f114bddbcbe6" />

### Durante el ataque
<img width="660" height="518" alt="image" src="https://github.com/user-attachments/assets/f3d5ecf4-9019-4bcf-8551-1501099cc50b" />

### Contramedida aplicada
<img width="639" height="450" alt="image" src="https://github.com/user-attachments/assets/a1d3a814-4465-4aba-8f2a-9b12de7878c0" />

---

## 6. Contramedidas

```
! Deshabilitar CDP globalmente
SW1(config)# no cdp run

! O por interfaz de acceso
SW1(config-if)# no cdp enable

! Verificación
SW1# show cdp neighbors
% CDP is not enabled
```

Al deshabilitar CDP el switch deja de procesar cualquier paquete CDP entrante, eliminando completamente la superficie de ataque. Se recomienda deshabilitar CDP en todos los puertos de acceso y mantenerlo solo en enlaces troncales de confianza entre dispositivos Cisco.
