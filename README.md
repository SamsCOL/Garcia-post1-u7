# Post-contenido — Unidad 7: Gestión de Tareas con Spring Boot

## Descripción
Repositorio del laboratorio de la Unidad 7 de Programación Web — Séptimo
Semestre. Un único proyecto Spring Boot con dos capas sobre el mismo
`TareaService`: una vista Thymeleaf (`@Controller`, parte 1) y una API REST
(`@RestController`, parte 2).

## Estructura del proyecto (plana)
Todos los archivos están en un mismo nivel, sin subcarpetas. Los paquetes
Java se conservan en la declaración `package` de cada clase y el `pom.xml`
indica a Maven dónde están las fuentes y dónde copiar los recursos.

| Archivo | Paquete / destino | Capa |
|---|---|---|
| `TareasApplication.java` | `com.universidad.tareas` | Arranque |
| `Prioridad.java`, `Tarea.java` | `com.universidad.tareas.model` | Modelo |
| `TareaService.java` | `com.universidad.tareas.service` | Servicio |
| `TareaController.java` | `com.universidad.tareas.controller` | Vista (Parte 1) |
| `TareaApiController.java`, `ApiErrorHandler.java` | `com.universidad.tareas.controller` | API (Parte 2) |
| `lista.html`, `formulario.html` | `classpath:/templates/tareas/` | Plantillas Thymeleaf |
| `application.properties` | `classpath:/` | Configuración |
| `pom.xml` | — | Build Maven |

## Prerrequisitos
- JDK 17 o superior.
- Apache Maven 3.8+.
- Postman o curl para probar la API.

## Parte 1 — Vista Thymeleaf con @Controller
`TareaController` expone `/tareas` con filtrado por `@RequestParam`
(prioridad, completada), formularios validados con `@Valid` + `BindingResult`,
y las acciones completar/eliminar implementadas como POST (no GET) para no
introducir efectos secundarios en peticiones de solo lectura.

## Parte 2 — API REST con @RestController
`TareaApiController` expone `/api/tareas` con los verbos GET, POST, PUT,
PATCH y DELETE, inyectando por constructor la MISMA instancia de
`TareaService` que usa la Parte 1. `ApiErrorHandler` traduce los errores de
`@Valid` en JSON estructurado (400 Bad Request), en lugar de la pantalla
de error HTML por defecto de Spring Boot.

### Endpoints
| Método | URL | Éxito | Error | Descripción |
|---|---|---|---|---|
| GET | `/api/tareas` | 200 OK | — | Lista en JSON, filtrable con `?prioridad=` y/o `?completada=` |
| GET | `/api/tareas/{id}` | 200 OK | 404 | Retorna la tarea con el ID indicado |
| POST | `/api/tareas` | 201 Created | 400 | Crea una tarea; 400 si falla la validación |
| PUT | `/api/tareas/{id}` | 200 OK | 404 / 400 | Reemplaza todos los campos de la tarea |
| PATCH | `/api/tareas/{id}/completar` | 200 OK | 404 | Marca únicamente `completada = true` |
| DELETE | `/api/tareas/{id}` | 204 No Content | 404 | Elimina la tarea |

### Ejemplos con curl
```bash
curl "http://localhost:8080/api/tareas?prioridad=ALTA&completada=false"

curl -X POST http://localhost:8080/api/tareas -H "Content-Type: application/json" \
  -d '{"titulo":"Documentar la API","descripcion":"Agregar ejemplos de uso","prioridad":"MEDIA","fechaLimite":"2026-12-10"}'

curl -X POST http://localhost:8080/api/tareas -H "Content-Type: application/json" \
  -d '{"titulo":"","prioridad":"ALTA","fechaLimite":"2026-12-10"}'

curl -X PATCH http://localhost:8080/api/tareas/4/completar

curl -i -X DELETE http://localhost:8080/api/tareas/4
```

## Decisiones de diseño
- Inyección por constructor (no `@Autowired` en campo) en ambos
  controladores: mejora la testabilidad y hace explícita la dependencia.
- `@FutureOrPresent` en lugar de `@Future` en `fechaLimite`: permite tareas
  con vencimiento el mismo día de su creación.
- POST (no GET) para completar/eliminar en `TareaController`: una petición
  GET debe ser segura y no debe modificar estado del servidor.
- PATCH (no PUT) para `/api/tareas/{id}/completar`: representa una
  actualización parcial de un único campo, no el reemplazo del recurso.
- Manejo de validación separado por capa: `BindingResult` para la vista
  HTML, `@RestControllerAdvice` para la API JSON — cada una responde en el
  formato que le corresponde.
- Persistencia en memoria (`Map` en `TareaService`) en lugar de JPA/Hibernate:
  la persistencia real se introduce formalmente en la Unidad 8.
- Estructura plana: todos los archivos en un mismo nivel; el `pom.xml`
  configura `sourceDirectory` y `resources` para que Spring encuentre las
  plantillas en `templates/tareas/` igual que en un proyecto estándar.

## Cómo compilar y ejecutar
1. Clonar el repositorio: `git clone [URL-del-repo]`
2. Abrir la carpeta como proyecto Maven en IntelliJ IDEA
3. Ejecutar `mvn spring-boot:run`
4. Vista web: http://localhost:8080/tareas
   API REST: http://localhost:8080/api/tareas (probar con Postman o curl)

## Capturas de pantalla
![Lista de tareas con filtros](lista-tareas.png)
![Formulario con error de validacion](formulario-error.png)
![POST 201 Created en Postman](postman-post-201.png)
![POST 400 Bad Request en Postman](postman-post-400.png)
