# Laboratorio Virtual de Ciberseguridad — Simulación de Ataque, Detección y Respuesta

Laboratorio de ciberseguridad construido desde cero para practicar en un entorno controlado: segmentación de red con firewall perimetral, monitoreo centralizado con SIEM, y un ciclo completo de ataque simulado → detección → contención.

 Proyecto de portafolio — Estudiante de Ingeniería Civil Informática, aspirante a prácticas/trabajo en ciberseguridad o redes (Chile).

---

## Infraestructura

Firewall perimetral (**pfSense**) que separa una "red externa" (WAN) de una LAN interna donde residen un servidor **SIEM (Wazuh)** y un **endpoint objetivo** (Ubuntu Desktop) con un servicio FTP deliberadamente mal configurado. La estación atacante (**Kali Linux**) opera desde un equipo físico independiente, conectado a la misma red externa.

| Dispositivo | Rol | IP | Asignación |
|---|---|---|---|
| pfSense (WAN) | Firewall perimetral — interfaz externa | `192.168.1.110` | DHCP (red doméstica) |
| pfSense (LAN) | Firewall perimetral — gateway interno | `192.168.10.1` | Estática |
| Wazuh Server | SIEM — manager, indexer y dashboard | `192.168.10.2` | DHCP reservado, hostname `siem` |
| Ubuntu Endpoint | Objetivo del ataque — FTP vulnerable | `192.168.10.13` | DHCP reservado, hostname `endpoint1` |
| Kali Linux | Estación atacante (equipo físico) | `192.168.1.21` | DHCP (red doméstica) |

Reglas clave en la interfaz **WAN** de pfSense: permiso de NAT hacia el puerto FTP del endpoint (origen restringido a la IP de Kali) y una regla de bloqueo total hacia esa misma IP, aplicada como medida de respuesta y ubicada al inicio del ruleset para que tenga prioridad de evaluación.

## Stack tecnológico

| Categoría | Herramienta |
|---|---|
| Virtualización | VirtualBox |
| Firewall / Router | pfSense 2.8.1 |
| SIEM | Wazuh (manager + indexer + dashboard) |
| Sistema objetivo | Ubuntu Desktop 24.04 + vsftpd |
| Atacante | Kali Linux (nmap, Hydra) |
| Servidor SIEM | Ubuntu Server 24.04 |

## Metodología del ataque

**1. Reconocimiento**
```bash
nmap -sS -Pn -sCV -T5 -p21 -n 192.168.1.110
```

**2. Explotación — fuerza bruta sobre FTP**
```bash
hydra -l endpoint1 -P rockyou.txt -t 1 -f -V ftp://192.168.1.110
```

**Resultado:** múltiples intentos fallidos seguidos de un inicio de sesión exitoso con las credenciales del sistema operativo, detectado por Wazuh mediante una regla de correlación por umbral (nivel 10, "Multiple failed logins in a small period of time"), confirmando tanto el ataque como el compromiso efectivo de la cuenta.

## Mapeo MITRE ATT&CK

| Fase | Técnica | ID |
|---|---|---|
| Reconocimiento | Escaneo activo de puertos y servicios | `T1595` |
| Acceso inicial | Fuerza bruta de contraseñas (FTP) | `T1110.001` |
| Acceso inicial | Uso de credenciales válidas | `T1078` |

## Consideraciones éticas

Todo el ejercicio se desarrolló en un entorno aislado y controlado, de propiedad del autor, con fines exclusivamente educativos. Ningún ataque se dirigió hacia sistemas, redes o servicios de terceros.

## Informe completo

El informe técnico detallado (arquitectura completa, configuración paso a paso, evidencia y conclusiones) está disponible en [`/docs/Informe_Laboratorio.pdf`](docs/Informe_Laboratorio.pdf).

---

**Autor:** Franco · Estudiante de Ingeniería Civil Informática
📍 Talca, Chile
