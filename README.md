# 🔓 Network Security Lab — Plaintext vs Ciphertext (SWITCH)

Hands-on network security laboratory conducted on **real physical equipment**
at the Universidad Nacional de Colombia. An advanced attack scenario was
executed where a hacker connected to an L2 switch performs an **ARP Spoofing**
attack to intercept traffic that would normally be protected by switch
segmentation.

> **Course:** Introduction to Cryptography and Information Security
> **University:** Universidad Nacional de Colombia — Bogotá Campus
> **Professor:** Jesús Guillermo Tovar Rache
> **Team:** Diego Andres Alvarez Gonzalez, Andres Camilo Orduz Lunar, Adrian Ramirez Gonzalez

---

## 🖥️ Physical Environment

Unlike the HUB lab, this scenario uses a **real L2 switch**, which requires
an active attack to intercept traffic. Everything was executed on real physical
infrastructure, not simulators:

- Physical PCs across multiple VLANs interconnected through real switches
- Physical Cisco router with dot1Q subinterfaces (router-on-a-stick)
- FTP server running FileZilla Server
- Laptop running **Kali Linux** as the attacker machine
- Wireshark for intercepted traffic analysis

---

## 🗺️ Network Diagram
```
VLAN 10 — 10.0.0.0/24     VLAN 20 — 20.0.0.0/24    VLAN 30 — 30.0.0.0/24
   PC_A                        PC_B                     PC_C / PC_D
     |                           |                           |
  [Switch L2-A] ————————— [Switch L2-B] ————————————— [Switch L3]
     |                                                       |
PC_Hacker (Kali)                                      FTP Server (PC_C)
                         [Router L3]
                    40.0.0.0/24 — PC_D (Server)
```

The attacker connects to switch L2-A. Unlike a HUB, the switch only forwards
traffic to the destination port — therefore **ARP Spoofing** is required to
intercept communications between other devices.

---

## ⚔️ Attacks Executed

### 1. ARP Spoofing + Telnet Session Interception

Since the switch does not forward traffic to non-destination ports, the
attacker executes ARP Spoofing to position as a Man-in-the-Middle between
PC_A and the router:
```bash
# Trick PC_A into believing the attacker is the router
sudo arpspoof -i eth0 -t 10.0.0.10 10.0.0.1

# Trick the router into believing the attacker is PC_A
sudo arpspoof -i eth0 -t 10.0.0.1 10.0.0.10

# Enable IP forwarding to relay packets and avoid dropping the connection
sudo sysctl -w net.ipv4.ip_forward=1
```

With the attack running, the Telnet session is captured with Wireshark and
the `show running-config` is fully reconstructed, obtaining:

- Enable password: `cisco123`
- User `admin` with plaintext password visible
- User `admin2` with encrypted password (type 5 — MD5)
- Complete interface, VLAN, and VTY line configuration

**Result:** the attacker authenticates with the captured credentials and
creates user `hacker` with password `hacker123` directly on the router.

### 2. ARP Spoofing on Server VLAN + FTP Interception

The attacker relocates to the same VLAN as the FTP server and repeats the
ARP Spoofing attack targeting new IPs:
```bash
sudo arpspoof -i eth0 -t 30.0.0.1 30.0.0.10
sudo arpspoof -i eth0 -t 30.0.0.10 30.0.0.1
sudo sysctl -w net.ipv4.ip_forward=1
```

The FTP transfer is intercepted, obtaining:

- Username: `usuariopcD` / Password: `clavesegura123`
- Full contents of `credenciales.txt` containing system users

**Result:** the attacker connects to the FTP server via FileZilla using the
captured credentials, modifies the file by injecting `hacker clave_hacker123`,
and overwrites the original on the server.

---

## 🛡️ Documented Mitigations

| Vulnerability | Mitigation |
|---|---|
| ARP Spoofing on switch | Dynamic ARP Inspection (DAI) on managed switches |
| ARP Spoofing | Port security + DHCP snooping |
| Telnet in plaintext | Migrate to SSH v2 with RSA 2048-bit keys |
| Unencrypted FTP | Configure FTPS (Explicit FTP over TLS) |
| Plaintext passwords on router | `service password-encryption` + `enable secret` |

### Plaintext vs encrypted password — captured from running-config
```
username admin password 0 cisco123          ← Plaintext, readable
username admin2 secret 5 $1$Msl4$nzZX...   ← MD5 hash, not reversible
```

This demonstrates the importance of using `enable secret` instead of
`enable password` on Cisco devices.

---

## 🧰 Tools Used

- **Kali Linux** — attack platform
- **arpspoof** (dsniff) — ARP Spoofing attack execution
- **Wireshark** — packet capture and traffic analysis
- **FileZilla Client** — connection to the compromised FTP server
- **Cisco IOS 12.4** — physical router with dot1Q subinterface configuration
- **Cisco L2 Switches** — real network infrastructure

---

## 📚 Concepts Applied

- ARP Spoofing / ARP Poisoning on switch-based networks
- Active Man-in-the-Middle (MITM) attack
- IP Forwarding for transparent attacks
- Practical difference between HUB and switch under sniffing attacks
- Analysis of insecure protocols: Telnet and FTP
- Comparison between plaintext and encrypted passwords on Cisco IOS
- Dynamic ARP Inspection as a countermeasure

---

## 🔄 Key Difference vs HUB Lab

| Aspect | HUB | Switch |
|---|---|---|
| Traffic capture | Passive — just connect | Active — requires ARP Spoofing |
| Attack complexity | Low | Medium-High |
| Detection | Difficult | Detectable with ARPWatch or DAI |
| Realism | Legacy networks | Current corporate networks |

---

## ⚠️ Disclaimer

This laboratory was conducted in a controlled academic environment for
educational purposes only. Its purpose is to demonstrate the risks of
poorly configured networks and the importance of implementing countermeasures
such as encryption, strong authentication, and network controls.
It must not be replicated outside authorized environments.

-------------------------------------------------------------------

# 🔓 Laboratorio de Seguridad en Redes — Texto Claro vs Texto Cifrado (SWITCH)

Laboratorio práctico de seguridad en redes realizado en **equipos físicos reales**
en el laboratorio de la Universidad Nacional de Colombia. Se ejecutó un escenario
de ataque avanzado donde un hacker conectado a un switch L2 realiza un ataque
de **ARP Spoofing** para interceptar tráfico que normalmente estaría protegido
por la segmentación del switch.

> **Curso:** Introducción a la Criptografía y a la Seguridad de la Información
> **Universidad:** Universidad Nacional de Colombia — Sede Bogotá
> **Profesor:** Jesús Guillermo Tovar Rache
> **Equipo:** Diego Andres Alvarez Gonzalez, Andres Camilo Orduz Lunar, Adrian Ramirez Gonzalez

---

## 🖥️ Entorno físico

A diferencia del laboratorio con HUB, este escenario usa un **switch L2 real**,
lo que requiere un ataque activo para interceptar el tráfico. Todo fue ejecutado
sobre infraestructura física real, no en simuladores:

- PCs físicas en múltiples VLANs interconectadas mediante switches reales
- Router Cisco físico con subinterfaces dot1Q (router-on-a-stick)
- Servidor FTP corriendo FileZilla Server
- Laptop con **Kali Linux** como máquina atacante
- Wireshark para análisis del tráfico interceptado

---

## 🗺️ Esquema de red
```
VLAN 10 — 10.0.0.0/24      VLAN 20 — 20.0.0.0/24    VLAN 30 — 30.0.0.0/24
   PC_A                         PC_B                     PC_C / PC_D
     |                            |                           |
  [Switch L2-A] —————————— [Switch L2-B] ————————————— [Switch L3]
     |                                                        |
PC_Hacker (Kali)                                       Servidor FTP (PC_C)
                          [Router L3]
                     40.0.0.0/24 — PC_D (Servidor)
```

El hacker se conecta al switch L2-A. A diferencia del HUB, el switch solo
envía el tráfico al puerto destino, por lo que se requiere **ARP Spoofing**
para interceptar comunicaciones entre otros dispositivos.

---

## ⚔️ Ataques ejecutados

### 1. ARP Spoofing + Interceptación de sesión Telnet

El switch no reenvía tráfico a puertos que no son el destino, por lo que
el hacker ejecuta ARP Spoofing para posicionarse como Man-in-the-Middle
entre PC_A y el router:
```bash
# Engañar al PC_A haciéndole creer que el atacante es el router
sudo arpspoof -i eth0 -t 10.0.0.10 10.0.0.1

# Engañar al router haciéndole creer que el atacante es PC_A
sudo arpspoof -i eth0 -t 10.0.0.1 10.0.0.10

# Habilitar IP forwarding para reenviar paquetes y no cortar la comunicación
sudo sysctl -w net.ipv4.ip_forward=1
```

Con el ataque activo, se captura la sesión Telnet con Wireshark y se
reconstruye el `show running-config`, obteniendo:

- Contraseña de enable: `cisco123`
- Usuario `admin` con contraseña plana visible
- Usuario `admin2` con contraseña cifrada (type 5 — MD5)
- Configuración completa de interfaces, VLANs y líneas VTY

**Resultado:** el hacker se autentica con las credenciales capturadas y
crea el usuario `hacker` con contraseña `hacker123` directamente en el router.

### 2. ARP Spoofing en VLAN del servidor + Intercepción FTP

El hacker se reubica en la misma VLAN que el servidor FTP y repite el
ataque ARP Spoofing apuntando a nuevas IPs:
```bash
sudo arpspoof -i eth0 -t 30.0.0.1 30.0.0.10
sudo arpspoof -i eth0 -t 30.0.0.10 30.0.0.1
sudo sysctl -w net.ipv4.ip_forward=1
```

Se intercepta la transferencia FTP y se obtiene:

- Usuario: `usuariopcD` / Contraseña: `clavesegura123`
- Contenido del archivo `credenciales.txt` con usuarios del sistema

**Resultado:** el hacker se conecta al servidor FTP con FileZilla usando
las credenciales capturadas, modifica el archivo agregando `hacker clave_hacker123`
y sobreescribe el original.

---

## 🛡️ Mitigaciones documentadas

| Vulnerabilidad | Mitigación |
|---|---|
| ARP Spoofing en switch | Dynamic ARP Inspection (DAI) en switches gestionados |
| ARP Spoofing | Port security + DHCP snooping |
| Telnet en texto claro | Migrar a SSH v2 con RSA 2048 bits |
| FTP sin cifrado | Configurar FTPS (Explicit FTP over TLS) |
| Contraseñas planas en router | `service password-encryption` + `enable secret` |

### Diferencia entre contraseña plana y cifrada capturada

En el `show running-config` interceptado se pudo observar la diferencia:
```
username admin password 0 cisco123          ← Texto claro, legible
username admin2 secret 5 $1$Msl4$nzZX...   ← Hash MD5, no reversible
```

Esto demuestra la importancia de usar `enable secret` en lugar de
`enable password` en dispositivos Cisco.

---

## 🧰 Herramientas utilizadas

- **Kali Linux** — plataforma de ataque
- **arpspoof** (dsniff) — ejecución del ataque ARP Spoofing
- **Wireshark** — captura y análisis de paquetes
- **FileZilla Client** — conexión al servidor FTP comprometido
- **Cisco IOS 12.4** — router físico con configuración de subinterfaces dot1Q
- **Switches Cisco L2** — infraestructura real de red

---

## 📚 Conceptos aplicados

- ARP Spoofing / ARP Poisoning en redes con switch
- Ataque Man-in-the-Middle (MITM) activo
- IP Forwarding para ataques transparentes
- Diferencia práctica entre HUB y switch ante ataques de sniffing
- Análisis de protocolos inseguros: Telnet y FTP
- Comparación entre contraseñas planas y cifradas en Cisco IOS
- Dynamic ARP Inspection como contramedida

---

## 🔄 Diferencia clave vs laboratorio con HUB

| Aspecto | HUB | Switch |
|---|---|---|
| Captura de tráfico | Pasiva — solo conectarse | Activa — requiere ARP Spoofing |
| Complejidad del ataque | Baja | Media-Alta |
| Detección | Difícil | Detectable con ARPWatch o DAI |
| Realismo | Redes legacy | Redes corporativas actuales |

---

## ⚠️ Aviso

Este laboratorio fue realizado en un entorno académico controlado con fines
exclusivamente educativos. Su propósito es demostrar los riesgos de redes
mal configuradas y la importancia de implementar contramedidas como cifrado,
autenticación robusta y controles de red. No debe replicarse fuera de entornos
autorizados.
