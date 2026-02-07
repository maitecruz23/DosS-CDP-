# 🔴 Ataque DoS mediante Protocolo CDP

## Laboratorio de Seguridad de Redes - Cisco Discovery Protocol Flooding

**Autor:** Maitte Rodriguez 
**Matrícula:** 20241165
**Institución:** ITLA
**Asignatura:** Seguridad de Redes  
**Fecha:** Febrero 2026

---

## ⚠️ ADVERTENCIA LEGAL

> **ESTE PROYECTO ES EXCLUSIVAMENTE PARA FINES EDUCATIVOS**

Este código está diseñado para ser utilizado **únicamente en entornos de laboratorio controlados** con el propósito de comprender y aprender sobre vulnerabilidades de seguridad en redes. 

**El uso no autorizado de estas técnicas constituye un delito grave:**
- ❌ NO utilizar en redes de producción
- ❌ NO utilizar sin autorización escrita explícita
- ❌ NO utilizar en infraestructura crítica
- ❌ NO utilizar fuera de tu laboratorio personal

**Al usar este código, aceptas total responsabilidad por tus acciones.**

---

## 📋 Descripción

Este laboratorio implementa un **ataque de Denegación de Servicio (DoS)** mediante el protocolo **Cisco Discovery Protocol (CDP)**. El objetivo es demostrar cómo un atacante puede explotar CDP para sobrecargar switches y routers Cisco, causando:

- 🔴 **Saturación de la tabla de vecinos CDP**
- 🔴 **Alto consumo de CPU** en el dispositivo objetivo
- 🔴 **Degradación del rendimiento de red**
- 🔴 **Potencial caída del servicio** (en casos extremos)

### ¿Qué es CDP?

**Cisco Discovery Protocol** es un protocolo propietario de Cisco Systems que opera en la **Capa 2** del modelo OSI. CDP permite que dispositivos Cisco descubran otros dispositivos Cisco directamente conectados.

**Características clave:**
- ✅ Habilitado por defecto en dispositivos Cisco
- ✅ Dirección MAC destino: `01:00:0c:cc:cc:cc`
- ❌ **NO tiene mecanismo de autenticación**
- ❌ **NO valida la identidad del emisor**

---

## 🎯 Objetivos de Aprendizaje

1. Comprender el protocolo CDP y su funcionamiento
2. Identificar vulnerabilidades en protocolos de capa 2
3. Implementar técnicas de flooding con Scapy
4. Analizar tráfico de red con Wireshark
5. Aplicar medidas de mitigación efectivas

---

## 🏗️ Topología de Red

```
<img width="754" height="893" alt="image" src="https://github.com/user-attachments/assets/fb538e87-2367-4c2a-b5b5-70a675455ace" />

              
```

### Configuración de Dispositivos

**Router vIOS (R1):**
- IP: 11.6.5.1/24
- Interfaz: GigabitEthernet0/0/12
- CDP: Habilitado

**Switch:**
- Gi0/0: Trunk al Router
- Gi0/2: Access port → Kali Linux

**Kali Linux:**
- IP: 11.6.5.10/24
- Gateway: 11.6.5.1
- Interfaz: eth0

---

## 📦 Instalación

### 1. Clonar el Repositorio

```bash
git clone https://github.com/[tu-usuario]/cdp-dos-attack.git
cd cdp-dos-attack
```

### 2. Instalar Dependencias

```bash
# Actualizar sistema
sudo apt update && sudo apt upgrade -y

# Instalar Python y pip
sudo apt install python3 python3-pip -y

# Instalar Scapy
sudo pip3 install scapy --break-system-packages

# Instalar Wireshark (opcional)
sudo apt install wireshark -y

# Verificar instalación
python3 -c "from scapy.all import *; from scapy.contrib.cdp import *; print('✓ Scapy OK')"
```

### 3. Dar Permisos

```bash
chmod +x cdp_dos_attack.py
```

---

## ⚙️ Configuración

Edita `cdp_dos_attack.py` para ajustar los parámetros:

```python
INTERFACE = "eth0"              # Tu interfaz de red
TARGET_SWITCH = "11.6.5.1"      # IP del switch/router
PACKET_COUNT = 5000             # Número de paquetes
DELAY = 0.001                   # Retardo entre paquetes (1ms)
```

---

## 🚀 Ejecución

```bash
# Ejecutar el ataque (requiere root)
sudo python3 cdp_dos_attack.py
```

### Salida Esperada

```
=== ATAQUE DoS MEDIANTE CDP ===
[*] Iniciando ataque DoS CDP contra 11.6.5.1
[*] Usando interfaz: eth0
[*] Enviando 5000 paquetes...

[+] Enviados 100/5000 paquetes CDP... (100.0 pps)
[+] Enviados 200/5000 paquetes CDP... (200.0 pps)
...
[✓] Ataque completado. Total paquetes enviados: 5000
[✓] Tiempo total: 30.00s
[✓] Tasa promedio: 166.7 pps
```

---

## 📊 Análisis con Wireshark

### Captura de Tráfico

```bash
# Terminal 1: Ejecutar ataque
sudo python3 cdp_dos_attack.py

# Terminal 2: Capturar tráfico
sudo wireshark -i eth0 -k -f "ether dst 01:00:0c:cc:cc:cc"
```

### Filtros Útiles

| Filtro | Propósito |
|--------|-----------|
| `cdp` | Mostrar solo paquetes CDP |
| `cdp.deviceid == "ATTACKER-DOS"` | Ver paquetes del atacante |
| `frame.time_delta > 0.1` | Paquetes con >100ms |
| `eth.dst == 01:00:0c:cc:cc:cc` | Tráfico CDP por MAC |

---

## 🔍 Verificación en el Switch

```cisco
! Ver vecinos CDP (durante el ataque)
Router# show cdp neighbors

! Ver estadísticas
Router# show cdp traffic

! Ver uso de CPU
Router# show processes cpu sorted

! Ver logs
Router# show logging | include CDP
```

### Comportamiento Durante el Ataque

```cisco
Router# show cdp neighbors

Device ID        Local Intrfce   Holdtme    Capability  Platform
ATTACKER-DOS     Gig 0/0/12      180        H           Linux x86
ATTACKER-DOS     Gig 0/0/12      180        H           Linux x86
ATTACKER-DOS     Gig 0/0/12      180        H           Linux x86
...
[Múltiples entradas duplicadas]
```

```cisco
Router# show cdp traffic

Total packets output: 45, Input: 5247  ← Alto número!
```

---

## 🛡️ Medidas de Mitigación

### 1. Deshabilitar CDP en Interfaces No Necesarias

```cisco
interface GigabitEthernet0/1
 no cdp enable
 exit
```

### 2. Deshabilitar CDP Globalmente

```cisco
no cdp run
```

### 3. Port Security

```cisco
interface GigabitEthernet0/1
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
 switchport port-security mac-address sticky
 exit
```

### 4. Implementar 802.1X

```cisco
aaa new-model
aaa authentication dot1x default group radius
dot1x system-auth-control

interface GigabitEthernet0/1
 authentication port-control auto
 dot1x pae authenticator
 exit
```

### 5. Rate Limiting

```cisco
class-map match-all CDP-TRAFFIC
 match protocol cdp
 exit

policy-map CDP-RATE-LIMIT
 class CDP-TRAFFIC
  police 8000 conform-action transmit exceed-action drop
 exit

interface GigabitEthernet0/1
 service-policy input CDP-RATE-LIMIT
 exit
```

---

## 🧪 Conceptos Técnicos

### Estructura del Paquete CDP

```
┌──────────────────────────────────────┐
│   ETHERNET HEADER                     │
├──────────────────────────────────────┤
│ Dst MAC: 01:00:0c:cc:cc:cc (CDP)    │
│ Src MAC: [MAC del atacante]          │
└──────────────────────────────────────┘
           ↓
┌──────────────────────────────────────┐
│   LLC HEADER                         │
├──────────────────────────────────────┤
│ DSAP: 0xAA, SSAP: 0xAA              │
└──────────────────────────────────────┘
           ↓
┌──────────────────────────────────────┐
│   SNAP HEADER                        │
├──────────────────────────────────────┤
│ OUI: 0x00000C, Protocol: 0x2000     │
└──────────────────────────────────────┘
           ↓
┌──────────────────────────────────────┐
│   CDP HEADER                         │
├──────────────────────────────────────┤
│ Version: 2, TTL: 180                 │
└──────────────────────────────────────┘
           ↓
┌──────────────────────────────────────┐
│   CDP TLVs                           │
├──────────────────────────────────────┤
│ Device ID: "ATTACKER-DOS"            │
│ Port ID: "Ethernet0"                 │
│ Capabilities: 0x00000028             │
│ Platform: "Linux x86_64"             │
└──────────────────────────────────────┘
```

### ¿Por qué funciona?

1. **Sin autenticación** - CDP no valida emisores
2. **Procesamiento obligatorio** - El switch debe procesar cada paquete
3. **Recursos limitados** - Tabla CDP y CPU tienen capacidad limitada
4. **Sin rate limiting** - Acepta todos los paquetes recibidos
5. **Habilitado por defecto** - En todos los puertos

---

## 📈 Resultados Esperados

| Métrica | Antes | Durante | Después |
|---------|-------|---------|---------|
| Paquetes CDP/seg | 0-2 | 100-500 | 0-2 |
| CPU Usage | 5-15% | 60-90% | 5-15% |
| Vecinos CDP | 1-5 | >100 | 1-5 |
| Latencia | <5ms | 20-100ms | <5ms |

---

## 🐛 Troubleshooting

**"Permission denied"**
```bash
sudo python3 cdp_dos_attack.py
```

**"No module named 'scapy'"**
```bash
sudo pip3 install scapy --break-system-packages
```

**"Interface does not exist"**
```bash
# Listar interfaces
ip link show

# Editar script con interfaz correcta
```

**No se ve impacto**
```bash
# Aumentar paquetes
PACKET_COUNT = 10000

# Reducir delay
DELAY = 0.0001
```

---

## 📚 Referencias

- [Cisco CDP Configuration Guide](https://www.cisco.com/c/en/us/support/docs/switches/catalyst-6500-series-switches/24048-148.html)
- [Scapy Documentation](https://scapy.readthedocs.io/)
- [Wireshark User Guide](https://www.wireshark.org/docs/)
- [CDP Security Best Practices](https://www.cisco.com/c/en/us/about/security-center/cdp-best-practices.html)

---

## 📄 Licencia

MIT License - Ver archivo [LICENSE](LICENSE)

**DISCLAIMER:** El uso no autorizado es ilegal. El autor no se hace responsable del mal uso de esta herramienta.

---

## 👨‍💻 Autor

**[Tu Nombre]**
- 🎓 [Tu Universidad]
- 📧 [tu-email@universidad.edu]
- 🔗 GitHub: [@tu-usuario](https://github.com/tu-usuario)

---

## ✅ Checklist del Laboratorio

- [ ] Topología implementada
- [ ] Script ejecutándose correctamente
- [ ] Capturas de Wireshark
- [ ] Screenshots de comandos show
- [ ] Video de demostración (max 8 min)
- [ ] Documentación completa
- [ ] Medidas de mitigación probadas
- [ ] Repositorio en GitHub

---

<div align="center">

**¡Desarrollado con fines educativos!** 🎓

**Usa responsablemente. La seguridad es responsabilidad de todos.** 🛡️

</div>
