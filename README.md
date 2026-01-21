# 🌐 Packet Tracer: Configuración de NAT Estática - Laboratorio Práctico

<div align="center">

**Laboratorio CISCO - Network Address Translation Estática para Servidores**

[![Cisco Packet Tracer](https://img.shields.io/badge/Cisco-Packet_Tracer-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)](https://www.netacad.com)
[![NAT Protocol](https://img.shields.io/badge/Protocol-NAT_Estático-00A86B?style=for-the-badge)](https://www.cisco.com/)
[![CCNA](https://img.shields.io/badge/Certification-CCNA-blue?style=for-the-badge)](https://www.cisco.com/c/en/us/training-events/training-certifications/certifications/associate/ccna.html)
[![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)](LICENSE)

[🎯 Objetivos](#-objetivos) • 
[📋 Situación](#️-situación) • 
[⚙️ Configuración](#️-configuración-paso-a-paso) • 
[🔍 Verificación](#️-verificación-y-resultados) • 
[👨‍💻 Autor](#️-autor)

</div>

---

## 📋 Descripción del Proyecto
Este laboratorio de Cisco Packet Tracer implementa **NAT Estática** para permitir acceso externo a un servidor interno. A diferencia de NAT dinámica o PAT, la NAT estática crea un mapeo permanente 1:1 entre una dirección IP privada y una pública, ideal para servidores que necesitan ser accesibles desde Internet.

### 🎯 Objetivos
**Parte 1:** Probar el acceso sin NAT (conexión fallida)  
**Parte 2:** Configurar NAT estática para el servidor  
**Parte 3:** Probar el acceso con NAT (conexión exitosa)  

### 🎯 Situación del Laboratorio
En redes IPv4, los servidores internos usan direcciones privadas. Para que dispositivos externos puedan acceder a estos servidores, es necesario configurar NAT estática que asigne una dirección pública fija a cada servidor, permitiendo el acceso controlado desde Internet.

---

## 🛠️ Topología y Escenario

### 🔧 Dispositivos y Direccionamiento
| Dispositivo | Dirección IP | Tipo | Función | Accesibilidad |
|-------------|--------------|------|---------|---------------|
| **Server1** | 172.16.16.1 | Privada | Servidor Web Interno | Solo red local |
| **Server1** | 64.100.50.1 | Pública | Traducción NAT | Acceso desde Internet |
| **R1** | 209.165.201.2 | Pública | Interface S0/0/0 | Gateway a Internet |
| **PC1** | 172.16.x.x | Privada | Cliente Interno | Acceso a Server1 |
| **L1** | 172.16.x.x | Privada | Cliente Interno | Acceso a Server1 |

### 🌐 Flujo de Tráfico
```
INTERNET → 64.100.50.1 (NAT) → 172.16.16.1 (Server1)
Server1 → 172.16.16.1 (NAT) → 64.100.50.1 → INTERNET
```

---

## ⚙️ Configuración Paso a Paso

### Parte 1: Probar el Acceso SIN NAT

#### Paso 1: Intentar conexión vía Modo Simulación
1. **Cambiar a Modo Simulación** en Packet Tracer
2. Desde PC1 o L1, intentar acceder a Server1 (172.16.16.1)
3. Usar botón **Capture Forward** para observar tráfico
4. **Observación:** Los paquetes nunca salen de la nube de Internet
5. **Resultado:** Conexión fallida

#### Paso 2: Verificar configuraciones en R1
```cisco
! Verificar que no hay configuración NAT
R1# show run | include nat
! No debería mostrar ningún comando NAT

! Verificar tabla de routing
R1# show ip route
! No debería haber rutas específicas para las IPs internas

! Verificar traducciones NAT (debería estar vacía)
R1# show ip nat translations
! No debería mostrar traducciones
```

#### Paso 3: Probar conectividad básica
```cisco
! Desde PC1, probar ping a interface externa de R1
PC1> ping 209.165.201.2
! El ping DEBERÍA ser exitoso (conectividad básica OK)
```

### Parte 2: Configurar NAT Estática

#### Paso 1: Configurar mapeo NAT estático
```cisco
! Crear traducción NAT estática 1:1
R1(config)# ip nat inside source static 172.16.16.1 64.100.50.1
! 172.16.16.1 = Dirección privada del servidor
! 64.100.50.1 = Dirección pública asignada
```

#### Paso 2: Configurar interfaces NAT
```cisco
! Configurar interfaz interna (hacia red LAN)
R1(config)# interface g0/0
R1(config-if)# ip nat inside
! Esta interfaz conecta con Server1 y clientes internos

! Configurar interfaz externa (hacia Internet)
R1(config)# interface s0/0/0
R1(config-if)# ip nat outside
! Esta interfaz conecta con Internet/ISP
```

**Diagrama de configuración:**
```
[Server1:172.16.16.1] ← (inside) [R1] (outside) → [Internet:64.100.50.1]
      ↓                                      ↑
    NAT Estático: 172.16.16.1 ↔ 64.100.50.1
```

### Parte 3: Probar el Acceso CON NAT

#### Paso 1: Verificar conectividad desde clientes internos
```cisco
! Desde PC1 o L1, hacer ping a la dirección pública
PC1> ping 64.100.50.1
! Resultado: Pings exitosos ✓

! Acceder vía navegador web
http://64.100.50.1
! Resultado: Página web del Server1 cargada exitosamente ✓
```

#### Paso 2: Verificar configuraciones NAT en R1
```cisco
! Ver configuración completa
R1# show running-config
! Debería mostrar:
! ip nat inside source static 172.16.16.1 64.100.50.1
! interface g0/0 → ip nat inside
! interface s0/0/0 → ip nat outside

! Ver traducciones NAT activas
R1# show ip nat translations
! Debería mostrar:
! Pro Inside global      Inside local       Outside local      Outside global
! --- 64.100.50.1        172.16.16.1        ---                ---

! Ver estadísticas NAT
R1# show ip nat statistics
! Debería mostrar:
! Total active translations: 1 (1 static, 0 dynamic)
! Hits: [número] Misses: 0
```

---

## 📊 Resultados y Análisis

### 🔍 Comparación ANTES vs DESPUÉS de NAT Estática

| Aspecto | SIN NAT Estática | CON NAT Estática |
|---------|------------------|------------------|
| **Acceso desde Internet** | ✗ Imposible | ✓ Posible |
| **Dirección visible externamente** | N/A | 64.100.50.1 |
| **Traducciones NAT** | 0 | 1 (estática) |
| **Acceso web a Server1** | ✗ Fallido | ✓ Exitoso |
| **Ping a IP pública** | ✗ No responde | ✓ Responde |

### 🎯 Puntos Clave Demostrados

1. **Aislamiento por defecto:** Sin NAT, servidores internos son invisibles para Internet
2. **Traducción bidireccional:** NAT estática permite tráfico en ambos sentidos
3. **Mapeo permanente:** La traducción 172.16.16.1 ↔ 64.100.50.1 es fija
4. **Seguridad implícita:** Solo el servidor mapeado es accesible desde fuera

### 📈 Estadísticas de Configuración
- **Tipo de NAT:** Estática (1:1 permanente)
- **Direcciones involucradas:** 2 (1 privada + 1 pública)
- **Interfaces configuradas:** 2 (1 inside + 1 outside)
- **Traducciones activas:** 1 siempre presente
- **Overhead:** Mínimo (no hay negociación dinámica)

---

## 💡 Conceptos Fundamentales Aprendidos

### 🎯 NAT Estática vs Dinámica vs PAT

| Característica | NAT Estática | NAT Dinámica | PAT (NAT Overload) |
|----------------|--------------|--------------|-------------------|
| **Mapeo** | 1:1 permanente | 1:1 temporal | Muchos:1 |
| **Direcciones IP** | Fijas | Del pool | Compartidas |
| **Uso típico** | Servidores | Clientes salientes | Clientes salientes |
| **Configuración** | Manual por cada host | Automática del pool | Automática por puertos |
| **Tabla NAT** | Entradas estáticas | Entradas dinámicas | Entradas dinámicas por puerto |

### 🔧 Comandos Clave de NAT Estática

```cisco
! Comando principal
ip nat inside source static [local-ip] [global-ip]

! Verificaciones
show ip nat translations        # Ver mapeos activos
show ip nat statistics          # Ver estadísticas
show run | include nat          # Ver configuración NAT
debug ip nat                    # Depurar en tiempo real

! Configuración de interfaces
interface [interfaz]
 ip nat inside                  # Para redes internas
 ip nat outside                 # Para Internet
```

### 🌐 Arquitectura de Red con NAT Estática

```
                [Internet]
                    |
             64.100.50.1 (NAT)
                    |
            +-------+-------+
            |       R1      |
            |  NAT Router   |
            +-------+-------+
                    |
             172.16.16.0/24
                    |
            +-------+-------+
            |               |
        [PC1, L1]      [Server1]
                      172.16.16.1
```

---

## 🚀 Aplicaciones Prácticas en el Mundo Real

### Casos de Uso Comunes
1. **Servidores Web Corporativos**
   ```cisco
   ip nat inside source static 192.168.1.100 203.0.113.10
   ```

2. **Servidores de Correo**
   ```cisco
   ip nat inside source static 192.168.1.101 203.0.113.11
   ```

3. **Servidores VPN**
   ```cisco
   ip nat inside source static 192.168.1.102 203.0.113.12
   ```

4. **Cámaras de Seguridad IP**
   ```cisco
   ip nat inside source static 192.168.1.200 203.0.113.20
   ```

### 🔒 Consideraciones de Seguridad

**Ventajas:**
- ✅ Acceso controlado y predecible
- ✅ Solo servicios explícitamente mapeados son accesibles
- ✅ Registro claro de accesos externos

**Precauciones:**
- ⚠️ Cada mapeo estático expone un servicio a Internet
- ⚠️ Necesita firewall adicional para protección
- ⚠️ Mantener inventario actualizado de mapeos

### 📝 Mejores Prácticas
1. **Documentación:** Mantener lista de todos los mapeos estáticos
2. **Segmentación:** Usar VLANs separadas para servidores mapeados
3. **Monitoreo:** Implementar logging de accesos NAT
4. **Backup:** Guardar configuración NAT en repositorio de configuraciones

---

## 🔍 Solución de Problemas NAT Estática

### Síntomas Comunes y Soluciones

#### ❌ No hay conectividad desde Internet
```cisco
! Verificar configuración NAT
show ip nat translations
show running-config | section nat

! Verificar rutas
show ip route
show ip interface brief

! Probar conectividad paso a paso
ping 64.100.50.1 source 172.16.16.1
traceroute 64.100.50.1
```

#### ❌ Conectividad unidireccional
```cisco
! Verificar reglas de firewall
show access-lists
show running-config | include access-group

! Verificar estado de interfaces
show interfaces [interfaz] status
show interfaces [interfaz] counters
```

#### ❌ Traducciones NAT no aparecen
```cisco
! Forzar tráfico para generar traducción
ping 64.100.50.1 from internal host

! Verificar con debug
debug ip nat
debug ip packet

! Verificar ACLs implícitas
show ip access-lists
```

### 📋 Checklist de Verificación NAT Estática

- [ ] Comando NAT estático configurado correctamente
- [ ] Interface inside configurada en interfaz LAN
- [ ] Interface outside configurada en interfaz WAN
- [ ] Rutas configuradas hacia ambas direcciones
- [ ] No hay ACLs bloqueando el tráfico
- [ ] Servidor interno responde a pings localmente
- [ ] Tabla NAT muestra la traducción estática

---

## 📚 Recursos Adicionales

### Documentación Oficial Cisco
- [Cisco NAT Static Configuration Guide](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/ipaddr_nat/configuration/15-mt/nat-15-mt-book.html)
- [Static NAT Best Practices](https://www.cisco.com/c/en/us/support/docs/ip/network-address-translation-nat/13772-12.html)
- [NAT for Server Access](https://www.cisco.com/c/en/us/td/docs/security/asa/asa82/configuration/guide/config/nat_static.html)

### Libros Recomendados
- "CCNA 200-301 Official Cert Guide" - NAT Chapter
- "Network Address Translation" - K. Holdaway
- "Cisco ASA Configuration Guide" - NAT Section

### Laboratorios Relacionados
- **NAT Dinámica:** Traducciones temporales para clientes
- **PAT (NAT Overload):** Múltiples hosts con una IP
- **NAT64:** Traducción entre IPv6 e IPv4
- **Twice NAT:** Traducción en origen y destino

### 🎓 Preguntas de Práctica CCNA
1. ¿Cuál es la diferencia entre NAT estática y dinámica?
2. ¿Cómo afecta NAT estática a la seguridad de red?
3. ¿Qué comando muestra todas las traducciones NAT?
4. ¿Cómo se configura NAT para permitir acceso SSH a un servidor interno?

---

## 👨‍💻 Autor

<div align="center">

**Darwin Manuel Ovalles Cesar**

<p align="center">
<a href="https://www.linkedin.com/in/darwin-manuel-ovalles-cesar-dev" target="_blank">
<img align="center" src="https://raw.githubusercontent.com/rahuldkjain/github-profile-readme-generator/master/src/images/icons/Social/linked-in-alt.svg" alt="LinkedIn - Darwin Ovalles" height="40" width="50" />
</a>
<a href="https://github.com/dovalless" target="_blank">
<img align="center" src="https://raw.githubusercontent.com/rahuldkjain/github-profile-readme-generator/master/src/images/icons/Social/github.svg" alt="GitHub - Darwin Ovalles" height="40" width="50" />
</a>
</p>

💼 **LinkedIn**: [darwin-manuel-ovalles-cesar-dev](https://www.linkedin.com/in/darwin-manuel-ovalles-cesar-dev/)  
🌐 **GitHub**: [@dovalless](https://github.com/dovalless)  
🎓 **Certificaciones**: CCNA, Network+, A+  

*"La NAT estática es el puente seguro entre los servicios internos y el mundo exterior. Este laboratorio demuestra cómo exponer servicios de manera controlada mientras se mantiene la integridad de la red interna."*

**#Cisco #PacketTracer #NAT #NATEstática #CCNA #Networking #ServerAccess**

</div>

---

## 📄 Licencia

Este proyecto está bajo la Licencia MIT. Consulta el archivo [LICENSE](LICENSE) para más detalles.

```
MIT License
Copyright (c) 2024 Darwin Manuel Ovalles Cesar
```

---

## 🙏 Agradecimientos

- **Cisco Networking Academy** - Por Packet Tracer y recursos educativos
- **Comunidad de Networking** - Por compartir conocimiento y experiencias
- **Profesionales de Seguridad** - Por destacar la importancia del acceso controlado

<div align="center">

### ⭐ Si este laboratorio te ayudó a entender NAT estática, compártelo ⭐

### 🔄 **Reflexión Final:**
*"La NAT estática es como tener un número de teléfono empresarial específico para cada departamento: predecible, controlado y profesional. Cada llamada (paquete) sabe exactamente a dónde ir."*

**Desarrollado con 💙 para la próxima generación de ingenieros de red**

---
*Laboratorio completado: Packet Tracer - Configurar NAT estática*  
*Habilidades demostradas: NAT Estática, Configuración de Servidores, Troubleshooting, Seguridad Perimetral*

</div>
```
