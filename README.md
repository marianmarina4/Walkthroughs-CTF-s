# Walkthroughs — Mariano Marina

Repositorio personal donde guardo mis writeups y walkthroughs de plataformas de pentesting (Hack The Box, HackMyVM, CTFs, laboratorios privados, etc.) para estudio y referencia personal.

---

## Tabla de contenidos

* [Propósito del repo](#propósito-del-repo)
* [Aviso legal / Ética](#aviso-legal--ética)
* [Estructura del repo](#estructura-del-repo)
* [Plantilla rápida para cada walkthrough](#plantilla-rápida-para-cada-walkthrough)
* [Contacto](#contacto)

---

## Propósito del repo

Guardar y organizar mis walkthroughs personales para estudiar, repasar técnicas y mantener material de referencia seguro.

---

## Aviso legal / Ética

* Solo agregar máquinas **retired** o entornos donde tengo permiso.
* No subir flags, contraseñas ni claves privadas en texto o imágenes.
* Writeups de máquinas activas deben mantenerse **privados** o **encriptados**.

---

## Estructura del repo

```
walkthroughs/
  hackthebox/
    <machine-name>-<YYYY>-<difficulty>/
      README.md        # walkthrough específico
      screenshots/
      exploits/
      notes.md
  hackmyvm/
README.md
.gitignore
LICENSE.md
generate_index.sh (opcional)
.github/workflows/ (opcional)
```

---

## Plantilla rápida para cada walkthrough

```markdown
# <Machine-Name> — <Plataforma> — <Dificultad>

**Fecha:** YYYY-MM-DD  
**Plataforma:** Hack The Box / HackMyVM / CTF / Lab  
**Estado:** Retired / Private / Active  
**Autor:** Mariano Marina

## Resumen (TL;DR)
- Objetivos: user + root
- Cadena rápida (1–3 líneas)

## Enumeración
- Comandos y resultados importantes

## Acceso inicial (foothold)
- Pasos y outputs relevantes

## Escalada a root
- Herramientas y razonamiento

## PoC / scripts
- Scripts documentados

## Conclusión / Aprendizajes
- Qué aprendí

## Referencias
- Links a documentación y CVEs
```

---

## Contacto

**Mariano Marina** — repositorio para uso personal, destinado únicamente a agregar mis walkthroughs y scripts de estudio.
