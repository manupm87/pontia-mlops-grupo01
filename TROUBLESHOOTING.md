# Troubleshooting

Documentación de los principales problemas técnicos encontrados durante el desarrollo y sus respectivas soluciones.

### 1. Incompatibilidad de versión de Python en local y cloud
**Síntoma:** Fallo transaccional al instalar dependencias de despliegue rápido (`fastapi==0.104.1`), retornando error fatal nativo por incompatibilidad estricta de base (3.7.8 < 3.8).
**Causa:** El ecosistema de intérprete subyacente (tanto en la máquina de desarrollo como en los "builders" originales de Render) apuntaba a `Python 3.7.8`, versión obsoleta sin soporte para la capa asíncrona moderna de dependencias.
**Solución:** Se ejecutó una estrategia de _Pinning_ de infraestructura. A nivel de Integración en el repositorio, se versionó estáticamente el archivo raíz `.python-version` forzando la directiva `3.10`. De forma inherente, esto forzó a los motores de aprovisionamiento del servidor remoto de Render a instanciar contenedores de dicha versión moderna, mitigando cualquier colapso al cargar y evaluar el `requirements.txt`.

### 2. Error 403 (Rate Limit) de la API de GitHub en Render
**Síntoma:** Caída de la aplicación en Render durante el arranque con `requests.exceptions.HTTPError: 403 Client Error: rate limit exceeded` intentando comunicarse con `api.github.com/repos/.../releases/latest`.
**Causa:** La aplicación efectuaba la llamada a la red localmente de manera anónima. Render opera con IPs compartidas; consecuentemente, el límite de 60 request/hora unificado por IP se consumía rápidamente debido a la latencia de peticiones en la red del servicio PaaS.
**Solución:** Se procedió a aprovisionar un Personal Access Token de GitHub, inyectándose paramétricamente a nivel global en Render e interceptándose en el script vía `os.getenv("GITHUB_TOKEN")` anexándolo al HTTP Header `Authorization`. Esto amplió el límite operacional a 5.000 requests.

### 3. Swagger UI sin generación de FormData (/docs)
**Síntoma:** La interfaz gráfica autogenerada en el endpoint reservado `/docs` omitía parcial o totalmente la redondancia de Payload properties para probar `POST /predict`.
**Causa:** El enrutador de FastAPI manejaba las cargas bajo el tipo genérico nativo `request: Request`, impidiendo a OpenAPI inferir programáticamente las tuplas requeridas para esquematizar un JSON validado.
**Solución:** Se definió no penalizar la flexibilidad estricta de la aplicación insertando la lógica de validación extra de `pydantic`. La omisión se salvaguardó migrando la dinámica de testing interactivo a herramientas exclusivas REST como `curl` y Postman usando los Payloads crudos pertinentes.

### 4. Code 404 (Not Found) emitido en el CI de validación (GitHub Actions)
**Síntoma:** Al lanzar la pipeline `.github/workflows/integration.yml` bajo la condición en demanda manual (`workflow_dispatch`), los unit tests prosperaban pero la propagación de comentarios retornaba un Fatal `HttpError: Not Found` sobre la ruta unida  `issues//comments`.
**Causa:** El script nativo invocaba la propiedad inherente `context.issue.number` para orquestar dónde propagar el log de cobertura. Al ser invocado manualmente, la Action carece de un contexto asociado originario de *Pull Request*, insertando de manera subyacente un escalar indefinido.
**Solución:** Para preservar eficiencia de cómputo, se omitió un "workaround" directo alterando el script subyacente, y se bloqueó sistemáticamente el paso acoplando la restricción binaria estricta de tipo en el macro del _step_: `if: always() && github.event_name == 'pull_request'`.

### 5. Inyección ineficiente del GITHUB_TOKEN en `deploy.yml`
**Síntoma:** Existencia de líneas `env: GITHUB_TOKEN...` insertadas debajo del paso del webhook manual dentro del archivo de GitHub Actions.
**Causa:** Presunción errónea del alcance local. Empacar las variables de entorno dentro del block level de Actions asigna dicho secreto al _shell local_, sin embargo el comando delegado (`curl -X POST`) es ajeno a este Scope en el Request saliente si no se implementa activamente dentro de Headers explícitos.
**Solución:** Eliminación del bloque ineficiente. La responsabilidad de los credenciales de aprovisionación estricta hacia Render se encapsularon unitariamente dentro del `render.yaml` o directamente en el _Dashboard_.
