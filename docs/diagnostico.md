# Diagnostico — Menu Club YAM

**Fecha:** 2026-10-12
**Especialista:** Jorge Esteban Anaya
**Generacion:** Temprana

## Que es esta app

Sistema de pedidos de comida para los empleados de la oficina PROESA en Hermosillo.
Los empleados entran desde su celular, buscan su numero de empleado, eligen los dias
que van a comer y guardan su pedido. La administradora (Vivi/Paola) gestiona el menu
semanal, ve los pedidos y comparte el resumen a la cocinera por WhatsApp.

## Comparacion contra el estandar

| Revision | Estandar | Estado | Nota |
|---|---|---|---|
| Repo | En Grupo-PROESA | No cumple | Vive en cuenta personal Artikke/Menu-club-YAM |
| Backend | FastAPI + uv + Python 3.12 | No cumple | No hay backend. Todo es un index.html con JS que habla directo a Firestore REST API |
| Base de datos | PostgreSQL propia | No cumple | Usa Firestore (proyecto menu-proesa-yama) |
| Migraciones | Alembic | No cumple | No hay migraciones. Los datos se crean/modifican desde el JS del navegador |
| Autenticacion | Portal Gateway con appCode | No cumple | Busqueda por numero de empleado, sin password. Admin con PIN hardcodeado |
| Integraciones | Por API con token | No aplica | No se integra con otras apps |
| Design system | @proesa/design | No cumple | CSS propio inline, usa Montserrat e Inter de Google Fonts |
| Ayuda embebida | Recorrido, ayuda contextual, /ayuda | No cumple | No tiene ayuda |
| CI | ci.yml en PRs | No cumple | No hay CI |
| CD | deploy.yml con migraciones y prueba de humo | No cumple | Deploy por GitHub Pages automatico al hacer push a main |
| Health | /api/health con estado de la base | No aplica | No hay API propia |
| Documentacion | CLAUDE.md, README.md, .env.example | No cumple | No tenia ninguno (se crean en este PR) |

**Cumple: 0 de 11 puntos aplicables.**

## Arquitectura actual

```
Empleado (celular)
    |
    v
index.html (GitHub Pages)
    |
    v
Firestore REST API (menu-proesa-yama)
    |
    +-- config/menu          (menu semanal)
    +-- config/catalogo      (lista de empleados)
    +-- config/cortesias      (invitados/cortesias)
    +-- config/comentarios    (comentarios de empleados)
    +-- orders/{numEmpleado}  (pedido activo de cada persona)
    +-- historial/{quincena}  (archivo historico por quincena)
```

Un solo archivo HTML de ~136 KB con todo: CSS, JS, logica de negocio,
llamadas a Firestore, interfaz de empleado e interfaz de administrador.

## Deuda priorizada por riesgo

### Riesgo alto (seguridad y datos)

1. **Secretos en el codigo.** El proyecto Firebase y la estructura de Firestore estan
   expuestos en el HTML publico. Cualquiera puede leer/escribir la base si conoce la URL.
   No hay reglas de seguridad en Firestore.
2. **PINs hardcodeados.** ADMIN_PIN (2835) y SUPER_PIN (7741) estan en el codigo fuente
   visible desde el navegador.
3. **Sin autenticacion real.** Un empleado puede pedir como otro si conoce su numero.
4. **Repo en cuenta personal.** Si la cuenta personal se desactiva, se pierde el sistema.

### Riesgo medio (operacion)

5. **Sin CI.** Cualquier push a main va directo a produccion sin verificacion.
6. **Sin health check.** No hay forma automatica de saber si el sistema esta funcionando.
7. **Sin respaldos formales.** Los datos en Firestore no tienen politica de respaldo.
8. **Un solo archivo.** Cualquier cambio puede romper todo. No hay separacion de capas.

### Riesgo bajo (evolucion)

9. **Sin design system.** La interfaz no usa @proesa/design.
10. **Sin ayuda embebida.** Los empleados dependen de que alguien les explique.
11. **Sin migraciones.** Los cambios de estructura de datos se hacen manualmente.
12. **Sin documentacion.** No habia README, CLAUDE.md ni docs/.

## Que funciona bien

- Los empleados lo usan y hacen sus pedidos sin problemas.
- La interfaz es clara y rapida en celular.
- El flujo de pedido es simple: buscar, elegir dias, guardar.
- La administradora puede gestionar menu, ver pedidos y compartir por WhatsApp.
- El historial por quincena permite ver consumos pasados.
- Las cortesias permiten registrar invitados.

## Camino de puesta al dia

La migracion completa al estandar (FastAPI + React + PostgreSQL + Portal) es un
proyecto grande. Se recomienda hacerlo por fases:

### Fase 0: Documentacion (este PR)
- Crear CLAUDE.md, README.md, docs/diagnostico.md, docs/contrato.md

### Fase 1: Transferir el repo a Grupo-PROESA
- Requiere: que Rodrigo apruebe y que TI lo ejecute
- El redirect de GitHub mantiene funcionando los links actuales

### Fase 2: Crear la microapp formal
- Usar /crear-microapp con la plantilla
- Migrar la logica a FastAPI + React + PostgreSQL
- Implementar autenticacion por Portal Gateway
- Migrar los datos de Firestore a PostgreSQL

### Fase 3: Seguridad y operacion
- CI/CD, health check, ayuda embebida
- Reglas de Firestore mientras se migra (o deshabilitar acceso directo)
