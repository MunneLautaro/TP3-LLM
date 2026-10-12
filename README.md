# Trabajo Práctico: Módulo 3 – Ingeniería de Prompts e Interacción Avanzada

Sistema backend automatizado de **triaje y resolución inteligente de reclamos** para una plataforma de comercio electrónico. El motor procesa mensajes de usuarios no estructurados, aplica razonamiento guiado (*Chain-of-Thought*), clasifica la urgencia y categoría mediante *Few-Shot Prompting*, y garantiza una salida en **formato JSON estrictamente validada con Pydantic** para su inserción directa en bases de datos transaccionales.

---

## Estructura del Repositorio

```text
TP3-LLM/
│
├── README.md                 # Documentación e instrucciones de ejecución
│
└── TP3/
    ├── tp3.ipynb             # Jupyter Notebook ejecutable con todo el flujo
    ├── requirements.txt      # Dependencias del proyecto
    ├── .env.example          # Plantilla para variables de entorno
    ├── .gitignore            # Archivos ignorados por Git (.env)
    └── enunciado.md          # Consigna oficial y rúbrica de evaluación
```

---

## 🛠️ Tecnologías y Requisitos

- **Python:** 3.10 o superior (recomendado 3.13)
- **Proveedor LLM:** Ollama Cloud (`https://ollama.com/v1`)
- **Modelo por defecto:** `gpt-oss:20b` (compatible también con `gemma4:31b`, `nemotron-3-nano:30b`, etc.)
- **Librerías principales:**
  - `openai`: Cliente para interactuar con la API compatible de Ollama.
  - `pydantic`: Validación estricta del esquema de datos.
  - `python-dotenv`: Carga de credenciales y variables de entorno desde `.env`.

---

## Guía de Instalación y Ejecución

### 1. Clonar el repositorio
```bash
git clone https://github.com/MunneLautaro/TP3-LLM.git
cd TP3-LLM
```

### 2. Crear y activar entorno virtual (Recomendado)
- **En Windows:**
  ```bash
  python -m venv .venv
  .venv\Scripts\activate
  ```
- **En Linux / macOS:**
  ```bash
  python3 -m venv .venv
  source .venv/bin/activate
  ```

### 3. Instalar las dependencias
```bash
pip install -r TP3/requirements.txt
```

### 4. Configurar la API Key
1. Dirígete a la carpeta `TP3`.
2. Crea una copia del archivo `.env.example` y nómbralo `.env`:
   ```bash
   cp TP3/.env.example TP3/.env
   ```
3. Genera una API Key gratuita en [Ollama Cloud Keys](https://ollama.com/settings/keys) y pégala en `TP3/.env`:
   ```env
   API_KEY=tu_clave_de_ollama_aqui
   ```

### 5. Ejecutar el Notebook
1. Abri VS Code o tu entorno preferido de Jupyter:
   ```bash
   code .
   ```
2. Abre el archivo `TP3/tp3.ipynb`.
3. Selecciona el **Kernel de Python** correspondiente (donde se instalaron las librerías).
4. Ejecuta las celdas secuencialmente o presiona **"Run All" / "Ejecutar todo"**.

---

## Arquitectura de la Solución

El proyecto implementa los tres requerimientos fundamentales de ingeniería de prompts:

1. **In-Context Learning & Clasificación (Few-Shot):**
   - El *System Prompt* incorpora **12 ejemplos complejos** de entrenamiento en contexto, cubriendo casos límite: cobros duplicados, productos defectuosos dentro y fuera de ventana de devolución, entregas falsas y reclamos combinados.
2. **Razonamiento Guiado (Chain-of-Thought):**
   - Se fuerza al LLM a razonar en el campo `cadena_de_pensamiento` antes de emitir la respuesta en 3 fases:
     1. Identificación del problema real (descartando sesgos emocionales).
     2. Aplicación de políticas empresariales (P1 a P5) y plazos.
     3. Selección del código de acción óptimo (`codigo_accion_sistema`).
3. **Salida Estructurada y Resiliencia con Pydantic:**
   - La respuesta del modelo se parsea en modo JSON y se valida contra modelos Pydantic estrictos (`RespuestaSoporte`).
   - Cuenta con normalizadores (`@field_validator`) para garantizar que la categoría sea unívoca incluso en mensajes con múltiples reclamos simultáneos.
4. **Seguridad contra Prompt Injections:**
   - El mensaje del cliente está aislado mediante delimitadores XML `<mensaje_cliente>`.
   - Se instruye al modelo a tratar el texto del usuario estrictamente como **datos pasivos** y nunca como directivas de control.

---

## Resumen del Análisis de Temperatura (Punto 5)

Durante los experimentos ejecutando el mismo reclamo complejo 3 veces a **Temperatura 0.0** y 3 veces a **Temperatura 1.0**:
- **Consistencia Estructural:** En ambas temperaturas se obtuvo un 100% de éxito en la validación JSON y estabilidad en decisiones críticas (`necesita_operador_humano = True`, `ACC_ESCALAR_HUMANO`).
- **Riesgos con Temperatura Alta ($T=1.0$):**
  - **Alucinación de SLAs:** El modelo inventó compromisos operativos no contemplados en las políticas (*"en las próximas 24 horas"*).
  - **Fuga de metadatos:** Mencionó explícitamente *"debido a su amenaza legal"* en el mensaje hacia el cliente.
- **Recomendación Operativa:** Para sistemas transaccionales y de atención al cliente en producción, se debe utilizar **Temperatura 0.0** para asegurar predictibilidad, rigor legal y evitar alucinaciones.
