# Trabajo4-RedesNeuronales

# EduAgent: Asistente Inteligente para Generación de Contenido Educativo

Este proyecto implementa un agente inteligente basado en Grandes Modelos de Lenguaje (LLMs) para la generación automatizada de materiales educativos a partir de programas de curso universitarios, diseñado para ejecutarse en Google Colab.

## Características

- **Análisis de programas de curso** en formatos PDF, DOCX y TXT
- **Generación de cinco tipos de materiales didácticos**:
  - Notas detalladas de clase
  - Problemas de práctica con soluciones
  - Preguntas para discusión
  - Objetivos de aprendizaje
  - Lecturas y recursos recomendados
- **Sistema de evaluación** automatizada de la calidad del contenido
- **Exportación de materiales** en formatos Markdown y PDF
- **Seguimiento de uso** y estimación de costos de API

## Requisitos

- Cuenta de Google (para acceder a Google Colab)
- Clave de API de Google Gemini
- Conexión a internet

## Configuración en Google Colab

1. Abre el notebook `EduAgent.ipynb` en Google Colab
   - Opción 1: Haz clic en este [enlace directo a Colab](https://colab.research.google.com/) y sube el archivo
   - Opción 2: Desde GitHub, abre el notebook y haz clic en "Open in Colab"

2. Instala las dependencias ejecutando la primera celda:
   ```python
   # Install dependencies
   !apt-get update 
   !apt-get install -y pandoc texlive-xetex
   !pip install markdown2 pypandoc google-generativeai
   ```

3. Configura tu clave API de Google Gemini:
   ```python
   # Replace with your actual API key
   api_key = "TU_CLAVE_API_AQUÍ"
   ```

## Uso del Notebook

### Carga de un programa de curso

Tienes dos opciones para cargar el programa de curso:

**Opción 1: Texto directo**
```python
# Define el contenido del curso directamente
course_content = """
Contenido del curso Cálculo diferencial. 
Unidades: 
1. Introducción a la derivada 
2. Derivada de la función compuesta 
3. Aplicaciones de la derivada en problemas de razón de cambio

Autor: Juan David Ospina Arango
Profesor Asociado
Departamento de Ciencias de la Computación y de la Decisión
Universidad Nacional de Colombia
"""
```

**Opción 2: Subir un archivo**
```python
# Sube y procesa un archivo de programa de curso
from google.colab import files
uploaded = files.upload()  # Se abrirá un diálogo para seleccionar el archivo

# Procesar el primer archivo subido
file_name = list(uploaded.keys())[0]
if file_name.endswith('.pdf'):
    # Código para procesar PDF
elif file_name.endswith('.docx'):
    # Código para procesar DOCX
else:
    # Código para procesar como texto
    with open(file_name, 'r', encoding='utf-8') as f:
        course_content = f.read()
```

### Generación de materiales educativos

Para generar los materiales educativos completos para el curso:

```python
# Generar y evaluar todos los materiales
all_materials, all_evaluations, usage_stats = generate_and_evaluate_complete_materials(
    course_content=course_content,
    output_dir="output_completo",
    model="gemini-2.0-flash-001",
    output_format="markdown",
    api_key=api_key
)

# Mostrar resultados
if all_materials and all_evaluations:
    print("\n¡ÉXITO! Materiales generados y evaluados correctamente")
    print(f"Total de tipos de materiales: {len(all_materials)}")
    print(f"Total de unidades procesadas: {len(next(iter(all_materials.values())))}")
    print(f"Costo total: ${usage_stats['total_cost_usd']:.4f}")
```

### Descarga de los materiales generados

Después de generar los materiales, puedes descargarlos desde Colab:

```python
# Comprimir todos los archivos generados
!zip -r output_completo.zip output_completo

# Descargar el archivo ZIP
from google.colab import files
files.download('output_completo.zip')
```

## Estructura de los archivos generados

Los materiales generados se organizan de la siguiente manera:

```
output_completo/
├── class_notes/
│   ├── [timestamp]/
│   │   ├── unit_1_class_notes.md
│   │   ├── unit_2_class_notes.md
│   │   └── course_name_class_notes.md
├── exercises/
│   ├── [timestamp]/
│   │   ├── unit_1_exercises.md
│   │   ├── unit_2_exercises.md
│   │   └── course_name_exercises.md
├── discussion_questions/
│   └── ...
├── learning_objectives/
│   └── ...
├── suggested_readings/
│   └── ...
└── evaluations/
    ├── class_notes_evaluations.md
    ├── exercises_evaluations.md
    └── summary_report.md
```

## Personalización en el Notebook

### Modificación de plantillas

Puedes personalizar las plantillas para cada tipo de material añadiendo una celda con:

```python
# Personalizar plantilla para notas de clase
additional_templates = {
    "class_notes": """
    [CONTEXTO DEL CURSO]
    Título del curso: {title}
    ... tu plantilla personalizada aquí ...
    """
}

# Actualizar el generador con tus plantillas personalizadas
generator = CourseContentGenerator(llm_client=gemini_client)
generator.templates.update(additional_templates)
```

### Ajuste de parámetros de generación

Puedes modificar los parámetros de generación:

```python
# Configurar el cliente con parámetros personalizados
gemini_client = GeminiClient(
    api_key=api_key,
    model="gemini-1.5-pro-001",  # modelo alternativo
    temperature=0.7,             # mayor creatividad (0.0 a 1.0)
    max_tokens=12000             # respuestas más largas
)
```

## Limitaciones en el entorno Colab

- **Tiempo de ejecución limitado**: Las sesiones de Colab tienen un límite de tiempo, especialmente en cuentas gratuitas.
- **Memoria limitada**: Para cursos muy extensos, puede haber limitaciones de memoria.
- **Almacenamiento temporal**: Los archivos generados se pierden al cerrar la sesión si no los descargas.
- **Dependencias preinstaladas**: Algunas dependencias pueden no estar disponibles y requerir instalación manual.

## Consejos para el uso en Colab

1. **Guarda regularmente**: Usa `Archivo > Guardar una copia en Drive` para no perder tu trabajo.
2. **Monitorea los recursos**: Revisa `Entorno de ejecución > Detalles del entorno de ejecución` para ver el uso de memoria/CPU.
3. **Descarga los resultados**: Siempre descarga los materiales generados antes de cerrar la sesión.
4. **Usa Google Drive**: Para cursos extensos, considera montar Google Drive para almacenar resultados:
   ```python
   from google.colab import drive
   drive.mount('/content/drive')
   ```

## Solución de problemas comunes en Colab

### Desconexión del entorno de ejecución
Si Colab se desconecta durante la generación, reduce el tamaño del curso o disminuye la complejidad de los materiales a generar.

### Error "API key not valid"
Verifica que tu clave API sea correcta y esté activa. También asegúrate de tener acceso al modelo Gemini especificado.

### Memoria insuficiente
Ejecuta `!nvidia-smi` para verificar el uso de memoria. Si es alto, prueba a limpiar la memoria:
```python
import gc
gc.collect()
```
