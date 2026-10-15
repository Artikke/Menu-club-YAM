# Menu Club YAM

## Que es

Sistema de pedidos de comida para los empleados de la oficina PROESA en Hermosillo.
Permite a los empleados elegir los dias que van a comer en la semana y a la
administradora gestionar el menu, ver pedidos y coordinar con la cocinera.

## Proceso que cubre

Pedido semanal de comida del comedor de oficina PROESA.

## Dueño

- **Area:** Talento Humano
- **Dueño del proceso:** Vivi (Mariana Jiménez, mjimenez@grupoproesa.mx)
- **Quien opera dia a dia:** Vivi / Paola
- **Especialista responsable:** Jorge Esteban Anaya

## Roles

| Rol | Que puede hacer |
|---|---|
| Empleado | Ver el menu, hacer su pedido semanal, agregar dias, dejar comentarios |
| Administrador (PIN 2835) | Configurar menu semanal, ver/borrar pedidos, gestionar cortesias, compartir resumen por WhatsApp, archivar quincena |
| Super admin (PIN 7741) | Todo lo anterior + gestionar catalogo de empleados |

## Vocabulario del area

- **Pedido:** los dias de la semana que un empleado elige para comer
- **Cortesia:** comida para un invitado que no es empleado
- **Quincena:** periodo de 15 dias que se usa para archivar el historial de pedidos
- **Catalogo:** lista de empleados con su numero, nombre y RFC

## Arquitectura

App de una sola pagina (index.html, ~136 KB) desplegada en GitHub Pages.
No hay backend propio. Los datos se guardan en Firestore (proyecto menu-proesa-yama).

### Colecciones en Firestore

| Coleccion | Documento | Que guarda |
|---|---|---|
| config | menu | Menu semanal (platillos por dia) |
| config | catalogo | Lista de empleados |
| config | cortesias | Invitados registrados |
| config | comentarios | Comentarios de empleados |
| orders | {numEmpleado} | Pedido activo de cada empleado |
| historial | {quincenaKey} | Archivo de pedidos por quincena |

### Acceso

- URL: https://artikke.github.io/Menu-club-YAM/
- Firestore: proyecto menu-proesa-yama, base (default)

## Generacion

Temprana. No cumple ningun punto del estandar (ver docs/diagnostico.md).

## Deuda conocida

- Secretos y PINs en el codigo fuente
- Sin autenticacion real (solo numero de empleado)
- Sin CI/CD (push directo a main = produccion)
- Sin separacion de capas (todo en un archivo)
- Sin ayuda embebida
- Repo en cuenta personal, no en Grupo-PROESA

## Reglas de negocio

- Un empleado puede pedir para los dias restantes de la semana
- Si ya tiene pedido, puede agregar dias nuevos sin perder los anteriores
- La quincena se archiva manualmente desde el panel de admin
- Las cortesias se registran aparte y se archivan al eliminarlas
- El menu lo configura la administradora cada semana
