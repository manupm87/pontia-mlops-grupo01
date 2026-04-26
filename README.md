# MLOps - Modelo de Predicción de Ingresos (Adult Income)

## 👥 Integrantes del Grupo
* [TU NOMBRE Y APELLIDO]
* [NOMBRE DEL INTEGRANTE 2]
* [NOMBRE DEL INTEGRANTE 3]

## 📌 Enlaces del Proyecto
- **Render deployment**: [https://pontia-mlops-tutorial-manuel-perez.onrender.com/docs](https://pontia-mlops-tutorial-manuel-perez.onrender.com/docs)
- **Video Evidencia**: [AGREGAR LINK AL VIDEO AQUÍ]
- **Troubleshooting**: [Ver TROUBLESHOOTING.md](./TROUBLESHOOTING.md)

## 📁 Estructura del Repositorio

El repositorio está organizado de la siguiente manera:

* **`.github/workflows/`**: Contiene las pipelines de CI/CD (GitHub Actions) que se ejecutan automáticamente en cada PR y merge a `main`.
* **`src/`**: Contiene el código fuente para el script de entrenamiento del modelo RandomForest (`data_loader.py`, `model.py`, `evaluate.py`, `main.py`). Aquí se procesa el dataset "Adult Income" y se generan los artefactos.
* **`deployment/`**: Contiene el código de la API basada en FastAPI (`app/main.py`) para servir las predicciones del modelo. Esta carpeta actúa como Raíz para el servidor de producción.
* **`model_tests/`**: Pruebas automáticas y validaciones enfocadas sobre el comportamiento del modelo.
* **`unit_tests/`**: Pruebas unitarias parametrizadas (con `pytest`) para asegurar la lógica del código y transformaciones (`src` y `deployment`).
* **`render.yaml`**: Archivo Blueprint ("Infrastructure as Code") que define todos los parámetros del servicio, comandos de construcción y variables de entorno para automatizar completamente el servidor Render.

## 🚀 Funcionamiento

1. **Desarrollo y CI**: Al crear una Pull Request (con revisión o "Code Review" obligatorio), GitHub Actions lanza las 3 pipelines. Si las validaciones y los tests pasan en "verde", se aprueba y se integra ("merge").
2. **Entrenamiento Continuo (CD)**: Durante la acción sobre la rama main, se produce la generación y validación del modelo. Los artefactos (`model.pkl`, `scaler.pkl`, `encoders.pkl`) se exportan y guardan en el área de Releases de GitHub.
3. **Despliegue a Producción**: Render está monitoreando el repositorio y detecta el nuevo código. La API `app/main.py` usa requests HTTP (con un `GITHUB_TOKEN` para evitar caer en el rate-limit de GitHub) para descargar de manera dinámica la última versión de los modelos empaquetados y servirlos a los usuarios en un endpoint de predicción tipado y testeable desde la UI.

## 💻 Instrucciones para ejecución local

1. Clonar el repositorio.
2. Ir al directorio de despliegue: `cd deployment`
3. Crear un entorno e instalar dependencias usando `uv`:
   ```bash
   uv venv --python 3.10 .venv
   source .venv/bin/activate
   uv pip install -r requirements.txt
   ```
4. Ejecutar la API:
   ```bash
   uvicorn app.main:app --reload
   ```

## ⏪ Simulación de Rollback (Proceso documentado)

Si un despliegue reciente introduce un error crítico en producción (ej. API caída, degradación severa de las métricas), el proceso de Rollback en este proyecto se ejecuta de la siguiente manera:

1. **Hotfix Automático vía Git**: Se localiza el commit que causó el efecto adverso. Dentro de GitHub o la terminal local, ejecutamos una reversión en la rama principal (`git revert <commit-hash>`). Esto genera un commit limpio que elimina los cambios problemáticos.
2. **Pipeline de Integración Inmediata**: Al generarse un nuevo _push_ revirtiendo los cambios, GitHub Actions correrá nuestras pipelines en verde restableciendo de nuevo el ecosistema seguro anterior (nuevo release).
3. **Despliegue Continuo (Zero-Downtime)**: Render recibe la actualización automática, provisiona un nuevo contenedor con el código parcheado sano y cambia el tráfico web de forma transparente.
4. **Rollback Manual Rápido (Plataforma Render)**: Como medida de emergencia si el código no funciona "en absoluto", el administrador ingresa a Render > "Deploys", clica en el último _deploy_ verde (previo a la interrupción) y selecciona **"Rollback to this deploy"**. Render levantará la imagen previa de manera instántanea.
