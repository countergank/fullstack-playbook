# 11 — Arquitectura de Software: Orden de Ejecución

> ⚠️ **El orden importa.** Este tópico es mayormente conceptual; la única guía práctica es la estructura del proyecto, que asume un backend ya funcionando.

## Prerequisito

Antes de empezar, asegurate de tener un proyecto backend Express + TypeScript armado (guía `04-backend`):

```bash
ls src/   # debe existir un proyecto con TypeScript configurado
```

## Paso a paso

1. **[setup-estructura-proyecto.md](setup-estructura-proyecto.md)** — Organizar el proyecto en capas (domain / application / infrastructure) siguiendo Clean Architecture y Hexagonal, respetando la regla de dependencia.

---

## Antes de la guía (contexto conceptual)

La guía asume que ya leíste el concept `concepts/11-arquitectura-software.md`, en particular las secciones 11.1 (Clean Architecture), 11.2 (Hexagonal) y 11.5 (Separación de Concerns). Sin ese contexto, la estructura de carpetas parece burocracia; con él, entendés que cada capa es un límite que protege tu negocio de los detalles técnicos.

---

## Verificación final

```bash
grep -rn "from '.*infrastructure" src/domain/   # sin resultados
grep -rn "from 'express" src/domain/ src/application/   # sin resultados
npx tsc --noEmit   # compila sin errores
```

**Si la regla de dependencia se cumple y compila → estructura lista. ✅**
