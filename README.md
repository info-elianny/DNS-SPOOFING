# 🛡️ Ataque DNS Spoofing / DNS Poisoning

📹 [Video demostración](https://youtu.be/eYPZtDWVI0U)

---

## 📌 Objetivo del Laboratorio

Demostrar cómo un atacante puede interceptar y manipular las consultas DNS de una víctima en la red local, redirigiendo el tráfico hacia un servidor web falso controlado por el atacante, con el fin de observar el impacto que este tipo de ataque tiene sobre la integridad y confidencialidad de las comunicaciones.

### Topología de Red
![Topología de Red](https://i.postimg.cc/DZFwpWvG/DNS.png)

---

## 🎯 Objetivo del Script

Combinar un ataque de ARP Poisoning con DNS Spoofing para posicionarse como intermediario entre la víctima y el gateway, interceptar las consultas DNS y redirigirlas hacia una página web falsa alojada en Kali Linux, simulando un portal legítimo del ITLA para capturar credenciales.

---

## ⚙️ Requisitos

### Hardware y Software

- Un equipo atacante con *Kali Linux*
- Un equipo víctima con *Windows 7*
- Ambos equipos conectados a la *misma red local*
- Un *router o gateway* accesible desde ambos dispositivos que sirva también como DNS
- *Python 3* instalado
- Biblioteca *Scapy* instalada
- Permisos de *administrador (root)* para ejecutar el script

### Instalación de dependencias

```bash
pip install scapy
```

---

## 🔧 Parámetros Utilizados

| Parámetro | Descripción |
|-----------|-------------|
| Interfaz de red | Adaptador desde el cual se realizará el ataque |
| IP de la víctima | Dirección IP del equipo a atacar (Windows 7) |
| IP del gateway | Dirección IP del router o puerta de enlace de la red |
| IP de Kali (servidor falso) | Dirección IP donde se alojará la página web falsa |
| Dominios a redirigir | Opción para interceptar solo itla.edu.do o todos los dominios |
| MAC de la víctima | Obtenida automáticamente mediante solicitudes ARP |
| MAC del gateway | Obtenida automáticamente mediante solicitudes ARP |
| IP Forwarding | Habilitado automáticamente para mantener la conectividad de la víctima |

---

## 🚀 Cómo Ejecutar el Script

```bash
sudo python3 dns_spoofing.py
```

> ⚠️ *Debe ejecutarse con permisos root (sudo)*

---

## 📋 Funcionamiento del Script

**Paso 1:** El script inicia comprobando que el usuario tenga permisos de administrador (root), ya que la captura y el envío de tramas de red requieren privilegios elevados.

**Paso 2:** Se solicita la interfaz de red, la IP de la víctima, la IP del gateway, la IP de Kali y los dominios a interceptar (solo itla.edu.do o todos).

**Paso 3:** El script obtiene automáticamente las direcciones MAC de la víctima y del gateway mediante solicitudes ARP, y habilita el IP Forwarding para mantener la conectividad de la víctima.

**Paso 4:** Se inicia en segundo plano un servidor web falso en el puerto 80 que sirve una página que simula el portal del ITLA, lista para capturar credenciales.

**Paso 5:** Se lanza el ARP Poisoning en segundo plano, enviando respuestas ARP falsas cada 2 segundos tanto a la víctima como al gateway para mantenerse como intermediario.

**Paso 6:** El script comienza a interceptar consultas DNS (puerto 53 UDP). Cuando detecta una consulta al dominio configurado, responde con la IP de Kali, redirigiendo a la víctima hacia la página falsa sin que lo note.

---

## 📸 Capturas de Pantalla

### Consulta DNS hacia itla.edu.do antes del ataque
![Consulta DNS antes del ataque](https://i.postimg.cc/kG6zxc0t/DNS-CAP-1.png)

---

### Búsqueda en Firefox antes del ataque
![Firefox antes del ataque](https://i.postimg.cc/Wpd1WPj5/DNS-CAP-2.png)

---

### Ataque corriendo
![Ataque corriendo](https://i.postimg.cc/zXWXNSVs/DNS-CAP-3.png)
![Ataque corriendo](https://i.postimg.cc/hG4SVkpt/DNS-CAP-4.png)

---

### Búsqueda DNS con el ataque corriendo — Kali responde en lugar del servidor DNS legítimo
![DNS con ataque corriendo](https://i.postimg.cc/hjfc5Vkk/DNS-CAP-5.png)

---

### Búsqueda del dominio en Firefox con el ataque activo
![Firefox con ataque](https://i.postimg.cc/SQYmL77d/DNS-CAP-6.png)

---

## 🌐 Documentación de la Red

| Dispositivo | Interfaz | Dirección IP | Máscara de Red |
|-------------|----------|-------------|----------------|
| R-1 | e0/0 | 6.6.1.1 | 255.255.255.0 |
| SW-1 | — | — | — |
| SW-2 | — | — | — |
| Kali Linux (Atacante) | e0/2 | 6.6.1.14 | 255.255.255.0 |
| Windows 7 (Víctima) | e0/1 | 6.6.1.12 | 255.255.255.0 |
| Cloud | net | 192.168.206.140 | 255.255.255.0 |

---

## 🛡️ Contramedidas para Mitigar el Ataque

### 1. Dynamic ARP Inspection (DAI)
Bloquea el ARP Poisoning que el script necesita para posicionarse como intermediario. Sin ARP Poisoning, el DNS Spoofing no funciona.

```
SW1(config)# ip dhcp snooping
SW1(config)# ip dhcp snooping vlan 1
SW1(config)# no ip dhcp snooping information option
SW1(config)# ip arp inspection vlan 1
SW1(config)# interface e0/0
SW1(config-if)# ip dhcp snooping trust
SW1(config-if)# ip arp inspection trust
SW1(config-if)# exit
```

![DAI resultado](https://i.postimg.cc/TwLv9fmM/DNS-M-1.png)

### 2. DNSSEC (DNS Security Extensions)
Agrega firmas digitales a las respuestas DNS para que el cliente pueda verificar su autenticidad. Si la respuesta ha sido manipulada, el cliente la descarta.

![DNSSEC resultado](insertar-link-de-imagen-aqui)

---

> ⚠️ *Este script es únicamente con fines educativos y de investigación en entornos controlados.*  
> ⚠️ *BY: Elianny*
