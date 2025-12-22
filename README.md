# Walkthroughs — Mariano Marina

Repositorio personal de writeups, walkthroughs y notas técnicas sobre plataformas de pentesting y laboratorios de ciberseguridad.

---

## Tabla de contenidos

* [Propósito del repositorio](#propósito-del-repositorio)
* [Aviso legal / Ética](#aviso-legal--ética)
* [Metodología utilizada](#metodología-utilizada)
* [Estructura del repositorio](#estructura-del-repositorio)
* [Plantilla rápida para cada walkthrough](#plantilla-rápida-para-cada-walkthrough)
* [Contacto](#contacto)
* [Etiquetas](#etiquetas)

---

## Propósito del repositorio
Este repositorio reúne mis walkthroughs y análisis técnicos realizados en:
- TryHackMe
- HackMyVM
- CTFs independientes
- Laboratorios privados
- Ejercicios de práctica personal
Mi objetivo es documentar mi proceso de aprendizaje en seguridad ofensiva, mejorar mi metodología de pentesting y mantener un registro organizado de técnicas, herramientas y vulnerabilidades.


---

## Aviso legal / Ética

- Solo publico walkthroughs de máquinas retiradas, permitidas o creadas para práctica.
- No incluyo flags, contraseñas reales ni claves privadas.
- No publico writeups de máquinas activas de plataformas como HTB o THM.
- Todo el contenido es únicamente para fines educativos y éticos
s.

---

## Metodología utilizada
Mis walkthroughs siguen una metodología ofensiva alineada con estándares como PTES y enfoques OSCP-like:
1. Reconocimiento
- Escaneos con Nmap
- Identificación de servicios, versiones y vectores iniciales
2. Enumeración
- Fuzzing con ffuf
- Análisis web con Burp Suite
- Enumeración SMB, FTP, SSH, etc.
- Recolección de credenciales y rutas vulnerables
3. Explotación
- Uso de exploits manuales o PoCs
- Metasploit cuando corresponde
- Inyección, RCE, LFI/RFI, fuerza bruta, etc.
4. Escalada de privilegios
- Análisis de permisos
- SUID, sudoers
- Servicios vulnerables
- Credenciales internas
5. Documentación
- Cada writeup incluye pasos claros, comandos utilizados y conclusiones técnicas.

---

## Estructura del repositorio

```
Walkthroughs-CTF-s/
│
├── TryHackMe/
│   ├── Easy/
│   │   ├── Vulnversity-THM.md
│   │   ├── Blue-THM.md
│   │   ├── BasicPentesting-THM.md
│   │   └── ColddBox-THM.md
│   └── (más niveles próximamente)
│
├── HackMyVM/
│   └── (walkthroughs futuros)
│
└── Plantillas y notas
```

---

## Plantilla rápida para cada walkthrough

```markdown
# Nombre de la máquina
## 1. Reconocimiento
## 2. Enumeración
## 3. Explotación
## 4. Escalada de privilegios
## 5. Lecciones aprendidas
## 6. Herramientas utilizadas
```

---

## Contacto

**Mariano Marina** — repositorio para uso personal, destinado únicamente a agregar mis walkthroughs y scripts de estudio.

---

## Etiquetas
#pentesting #tryhackme #ctf #writeups #cybersecurity  
#ethicalhacking #oscp #redteam #enumeration #privesc  
#infosec #walkthroughs #hackmyvm #offensivesecurity
