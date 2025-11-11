# Sistema RAG con LangChain y Pinecone

**Autor:** Andrés Serrato Camero  
**Curso:** AREP - Arquitecturas Empresariales  
**Universidad:** Escuela Colombiana de Ingeniería Julio Garavito

## 📋 Descripción

Este proyecto implementa un sistema de **Retrieval-Augmented Generation (RAG)** que permite realizar consultas inteligentes sobre documentos web. El sistema combina la potencia de los Large Language Models (LLMs) con búsqueda semántica en bases de datos vectoriales para proporcionar respuestas precisas y contextualizadas.

### ¿Qué es RAG?

RAG (Retrieval-Augmented Generation) es una técnica que mejora las capacidades de los modelos de lenguaje al permitirles acceder a información externa relevante antes de generar una respuesta. Esto reduce las alucinaciones del modelo y permite trabajar con información actualizada sin necesidad de reentrenar el modelo.

## 🛠️ Tecnologías Utilizadas

- **[LangChain](https://www.langchain.com/)**: Framework para desarrollo de aplicaciones con LLMs
- **[OpenAI GPT-4](https://openai.com/)**: Modelo de lenguaje para generación de respuestas
- **[Pinecone](https://www.pinecone.io/)**: Base de datos vectorial serverless para búsqueda semántica
- **[BeautifulSoup4](https://www.crummy.com/software/BeautifulSoup/)**: Librería para parsing de contenido HTML
- **Python 3.12+**: Lenguaje de programación principal

## 🏗️ Arquitectura del Sistema

```
┌─────────────────┐
│  Documento Web  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  WebBaseLoader  │ ──► Extrae contenido HTML
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Text Splitter   │ ──► Divide en chunks de 1000 caracteres
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ OpenAI Embeddings│ ──► Convierte a vectores (1024 dims)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Pinecone Index  │ ──► Almacena vectores
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  RAG Agent      │ ──► Consulta + Búsqueda + Generación
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   Respuesta     │
└─────────────────┘
```

## 📦 Instalación

### 1. Clonar el repositorio

```bash
git clone <url-del-repositorio>
cd taller9-arep
```

### 2. Crear entorno virtual

```bash
python -m venv .venv
source .venv/bin/activate  # En Windows: .venv\Scripts\activate
```

### 3. Instalar dependencias

```bash
pip install openai python-dotenv
pip install langchain langchain-text-splitters langchain-community bs4
pip install -U "langchain-openai"
pip install -qU langchain-pinecone
```

### 4. Configurar variables de entorno

Crea un archivo `.env` en la raíz del proyecto con el siguiente contenido:

```env
# OpenAI API Configuration
OPENAI_API_KEY=tu-api-key-de-openai

# Pinecone Configuration
PINECONE_API_KEY=tu-api-key-de-pinecone
PINECONE_INDEX_NAME=quickstart

# LangChain Configuration (Optional)
LANGCHAIN_API_KEY=tu-api-key-de-langchain
LANGCHAIN_TRACING=true
```

#### Obtener las API Keys:

- **OpenAI API Key**: https://platform.openai.com/api-keys
- **Pinecone API Key**: https://app.pinecone.io/
- **LangChain API Key**  https://smith.langchain.com/

## 🚀 Uso

### Ejecutar el Notebook

1. Abre Jupyter Notebook o VS Code con la extensión de Jupyter
2. Abre el archivo `taller9.ipynb`
3. Ejecuta las celdas en orden secuencial

### Estructura del Notebook

El notebook está organizado en 5 secciones principales:

#### 1. Instalación de Dependencias
Instala todas las librerías necesarias para el proyecto.

#### 2. Configuración Inicial
- **2.1 Variables de Entorno**: Carga las credenciales desde el archivo `.env`
- **2.2 Inicialización de Modelos**: Configura GPT-4 y el modelo de embeddings
- **2.3 Configuración de Pinecone**: Conecta con la base de datos vectorial

#### 3. Carga y Procesamiento de Documentos
- **3.1 Carga de Documento Web**: Extrae contenido de un blog post
- **3.2 Visualización del Contenido**: Muestra una vista previa del documento
- **3.3 División en Chunks**: Fragmenta el documento en piezas manejables
- **3.4 Indexación en Pinecone**: Almacena los embeddings en la base de datos

#### 4. Creación del Agente RAG
- **4.1 Definición de Herramienta**: Crea la función de recuperación de contexto
- **4.2 Configuración del Agente**: Inicializa el agente con GPT-4 y las herramientas

#### 5. Ejecución de Consultas
- **5.1 Consulta de Ejemplo**: Ejecuta una consulta compleja con streaming de respuestas

## 💡 Ejemplo de Uso

```python
# Realizar una consulta al sistema RAG
query = "What is the standard method for Task Decomposition?"

for event in agent.stream(
    {"messages": [{"role": "user", "content": query}]},
    stream_mode="values",
):
    event["messages"][-1].pretty_print()
```

El agente:
1. Recibe la pregunta del usuario
2. Usa la herramienta `retrieve_context` para buscar información relevante en Pinecone
3. Combina el contexto recuperado con la pregunta
4. Genera una respuesta coherente usando GPT-4
5. Retorna la respuesta en tiempo real (streaming)

## 🔧 Configuración Avanzada

### Personalizar el Chunk Size

```python
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1500,      # Aumentar tamaño de fragmentos
    chunk_overlap=300,    # Aumentar superposición
    add_start_index=True,
)
```

### Cambiar el número de documentos recuperados

```python
@tool(response_format="content_and_artifact")
def retrieve_context(query: str):
    """Retrieve information to help answer a query."""
    retrieved_docs = vector_store.similarity_search(query, k=5)  # Cambiar k=2 a k=5
    # ... resto del código
```

### Usar un modelo diferente

```python
model = init_chat_model("gpt-4-turbo")  # o "gpt-3.5-turbo"
```

## 📊 Características del Sistema

- ✅ **Búsqueda Semántica**: Encuentra información relevante basándose en el significado, no solo palabras clave
- ✅ **Streaming de Respuestas**: Muestra las respuestas en tiempo real a medida que se generan
- ✅ **Contexto Preservado**: Mantiene la coherencia entre múltiples consultas
- ✅ **Escalable**: Pinecone permite escalar a millones de vectores
- ✅ **Modular**: Fácil de extender con nuevas herramientas y fuentes de datos

## 📝 Notas Importantes

### Sobre Pinecone

- El índice debe tener **1024 dimensiones** para coincidir con el modelo de embeddings `text-embedding-3-large`
- Usa la región **us-east-1** (AWS) para mejor rendimiento
- La métrica de similitud es **cosine**

### Sobre los Costos

- **OpenAI**: Cobra por tokens de entrada y salida
  - GPT-4: ~$0.03 por 1K tokens de entrada, ~$0.06 por 1K tokens de salida
  - Embeddings: ~$0.0001 por 1K tokens
- **Pinecone**: Plan gratuito disponible con 1 índice serverless

### Límites y Consideraciones

- El modelo `text-embedding-3-large` tiene un límite de ~8,191 tokens por entrada
- Los chunks deben ser lo suficientemente pequeños para caber en este límite
- GPT-4 tiene un contexto de ~128K tokens

## 🐛 Solución de Problemas

### Error: "Invalid API Key"
- Verifica que tu API key de OpenAI sea válida y esté activa
- Asegúrate de que tu cuenta tenga créditos disponibles
- Recarga las variables de entorno ejecutando la celda 2.1

### Error: "Index not found"
- Verifica que el nombre del índice en `.env` coincida con el índice en Pinecone
- Asegúrate de que el índice esté creado con las especificaciones correctas

### Error: "Dimension mismatch"
- El índice debe tener exactamente 1024 dimensiones
- Recrea el índice si es necesario

## 📚 Recursos Adicionales

- [Documentación de LangChain](https://python.langchain.com/)
- [Documentación de Pinecone](https://docs.pinecone.io/)
- [OpenAI API Reference](https://platform.openai.com/docs/api-reference)
- [Tutorial de RAG](https://python.langchain.com/docs/tutorials/rag/)

## 👨‍💻 Autor

**Andrés Serrato Camero**  

