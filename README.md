# MLOps - Modelo de Predicción de Ingresos (Adult Income)

[![Build](https://github.com/manupm87/pontia-mlops-grupo01/actions/workflows/build.yml/badge.svg)](https://github.com/manupm87/pontia-mlops-grupo01/actions/workflows/build.yml)
[![Integration](https://github.com/manupm87/pontia-mlops-grupo01/actions/workflows/integration.yml/badge.svg)](https://github.com/manupm87/pontia-mlops-grupo01/actions/workflows/integration.yml)
[![Deploy Model](https://github.com/manupm87/pontia-mlops-grupo01/actions/workflows/deploy.yml/badge.svg)](https://github.com/manupm87/pontia-mlops-grupo01/actions/workflows/deploy.yml)

## 👥 Integrantes del Grupo
* Manuel Pérez Martínez
* [NOMBRE DEL INTEGRANTE 2]
* [NOMBRE DEL INTEGRANTE 3]
* [NOMBRE DEL INTEGRANTE 4]
* Enmanuel Alejandro De Oleo Ubiera

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
* **`.python-version`**: Archivo de especificación global que fija la versión del intérprete de Python (3.10) para garantizar compatibilidad multiplataforma y forzar al _builder_ nativo de Render a utilizar utilidades modernas.
* **`render.yaml`**: Archivo Blueprint ("Infrastructure as Code") que define todos los parámetros del servicio, comandos de construcción y variables de entorno para automatizar completamente el servidor Render.

## 🚀 Funcionamiento

1. **Desarrollo y CI**: Al crear una Pull Request (con revisión o "Code Review" obligatorio), GitHub Actions lanza las 3 pipelines. Si las validaciones y los tests pasan en "verde", se aprueba y se integra ("merge").
2. **Entrenamiento Continuo (CD)**: Durante la acción sobre la rama main, se produce la generación y validación del modelo. Los artefactos (`model.pkl`, `scaler.pkl`, `encoders.pkl`) se exportan y guardan en el área de Releases de GitHub.
3. **Despliegue a Producción**: Render está monitoreando el repositorio y detecta el nuevo código. La API `app/main.py` usa requests HTTP (con un `GITHUB_TOKEN` para evitar caer en el rate-limit de GitHub) para descargar de manera dinámica la última versión de los modelos empaquetados y servirlos a los usuarios en un endpoint de predicción tipado y testeable desde la UI.

## 💻 Instrucciones para ejecución local

1. Clonar el repositorio.
2. Ir al directorio de despliegue: `cd deployment`
3. Crear un entorno virtual e instalar las dependencias con `pip` y `python` nativo:
   ```bash
   python3 -m venv .venv
   source .venv/bin/activate
   pip install -r requirements.txt
   ```
4. Ejecutar la API:
   ```bash
   uvicorn app.main:app --reload
   ```

## 🧪 Ejemplos de Inferencia (Testeo)

Puedes probar la API desde cualquier terminal (o apuntando localmente a tu URL de Render) enviando estas consultas para simular ambos escenarios posibles soportados por el modelo RandomForest:

**Escenario 1: Predicción de Ingreso <= 50K (Retorna `[0]`)**
```bash
curl -X POST https://pontia-mlops-tutorial-manuel-perez.onrender.com/predict \
-H "Content-Type: application/json" \
-d '{
  "age": 39,
  "workclass": "State-gov",
  "fnlwgt": 77516,
  "education": "Bachelors",
  "education-num": 13,
  "marital-status": "Never-married",
  "occupation": "Adm-clerical",
  "relationship": "Not-in-family",
  "race": "White",
  "sex": "Male",
  "capital-gain": 2174,
  "capital-loss": 0,
  "hours-per-week": 40,
  "native-country": "United-States"
}'
```

**Escenario 2: Predicción de Ingreso > 50K (Retorna `[1]`)**
```bash
curl -X POST https://pontia-mlops-tutorial-manuel-perez.onrender.com/predict \
-H "Content-Type: application/json" \
-d '{
  "age": 52,
  "workclass": "Private",
  "fnlwgt": 209642,
  "education": "Masters",
  "education-num": 14,
  "marital-status": "Married-civ-spouse",
  "occupation": "Exec-managerial",
  "relationship": "Husband",
  "race": "White",
  "sex": "Male",
  "capital-gain": 15024,
  "capital-loss": 0,
  "hours-per-week": 50,
  "native-country": "United-States"
}'
```
*(Nota: Si ejecutas la instancia en local, simplemente remplaza la URL de Render por `http://127.0.0.1:8000/predict`)*

## ⏪ Simulación de Rollback (Proceso documentado)

Si un despliegue reciente introduce un error crítico en producción (ej. API caída, degradación severa de las métricas), el proceso de Rollback en este proyecto se ejecuta de la siguiente manera:

1. **Hotfix Automático vía Git**: Se localiza el commit que causó el efecto adverso. Dentro de GitHub o la terminal local, ejecutamos una reversión en la rama principal (`git revert <commit-hash>`). Esto genera un commit limpio que elimina los cambios problemáticos.
2. **Pipeline de Integración Inmediata**: Al generarse un nuevo _push_ revirtiendo los cambios, GitHub Actions correrá nuestras pipelines en verde restableciendo de nuevo el ecosistema seguro anterior (nuevo release).
3. **Despliegue Continuo (Zero-Downtime)**: Render recibe la actualización automática, provisiona un nuevo contenedor con el código parcheado sano y cambia el tráfico web de forma transparente.
4. **Rollback Manual Rápido (Plataforma Render)**: Como medida de emergencia si el código no funciona "en absoluto", el administrador ingresa a Render > "Deploys", clica en el último _deploy_ verde (previo a la interrupción) y selecciona **"Rollback to this deploy"**. Render levantará la imagen previa de manera instántanea.
