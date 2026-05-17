# Chatbot Beca 18 — Sistema RAG con PRONABEC

## Propósito y documento fuente

Este proyecto implementa un sistema de **Generación Aumentada por Recuperación (RAG)** para responder preguntas sobre el reglamento oficial de Beca 18 de PRONABEC. El documento fuente es la **Resolución Directoral Ejecutiva N.° 033-2026-MINEDU/VMGI-PRONABEC** (138 páginas), que contiene las Bases del Concurso Beca 18 y Becas Especiales — Convocatoria 2026.

El sistema responde **exclusivamente** a partir del texto del documento oficial, sin usar conocimiento externo del modelo, y rechaza preguntas fuera de tema.

---

## Resumen del proceso

El pipeline extrae el texto del PDF página a página (con marcadores `[PAGE N]`), lo divide en fragmentos de 400 caracteres con 60 de solapamiento usando LangChain, genera embeddings con `gemini-embedding-001` (768 dimensiones) y los indexa en una base de datos vectorial ChromaDB con distancia coseno. Ante cada pregunta, la consulta se incrusta con tarea `RETRIEVAL_QUERY`, se recuperan los *k* fragmentos más similares y se envían como contexto al modelo `gemini-2.5-flash`, que genera una respuesta citando el número de página correspondiente. Una interfaz ipywidgets permite interactuar con el sistema de forma interactiva dentro del notebook.

---

## Instalación y configuración

### 1. Clona el repositorio

```bash
git clone <url-repo>
cd beca18-rag-chatbot
```

### 2. Configura la clave API de Gemini

```bash
cp .env.example .env
# Edita .env y reemplaza your_key_here con tu clave de Google AI Studio
```

Obtén tu clave gratuita en: https://aistudio.google.com/app/apikey

> **Importante:** Nunca subas el archivo `.env` al repositorio. Está excluido en `.gitignore`.

### 3. Instala las dependencias

```bash
pip install -r requirements.txt
```

### 4. Coloca el PDF en la carpeta `data/`

Descarga el documento desde:  
https://www.gob.pe/institucion/pronabec/normas-legales/7778068-033-2026-minedu-vmgi-pronabec

Y guárdalo como:
```
data/beca18_reglamento.pdf
```

---

## Cómo ejecutar el notebook

```bash
cd notebooks
jupyter notebook beca18_rag_chatbot.ipynb
```

Ejecuta las celdas **en orden** de arriba a abajo. La primera ejecución tardará varios minutos en el paso de indexación (generación de embeddings para todos los chunks). Las siguientes ejecuciones cargarán la colección ChromaDB existente automáticamente.

En **Google Colab**: sube el PDF a `data/beca18_reglamento.pdf` y crea el archivo `.env` con tu API key antes de ejecutar.

---

## Cómo usar la interfaz de chat

Una vez ejecutado el Paso 7, aparecerá la interfaz en la salida de la celda:

1. **Escribe tu pregunta** en el cuadro de texto.
2. **Ajusta el slider `k`** para controlar cuántos fragmentos se recuperan (1–10; mayor k = más contexto, respuesta más completa pero más lenta).
3. **Haz clic en "🔍 Preguntar"** — la respuesta aparece en verde con las páginas citadas.
4. **Expande el acordeón** "📎 Ver fragmentos fuente recuperados" para ver los chunks exactos usados.
5. **"🗑 Borrar"** limpia la interfaz para una nueva consulta.

### Ejemplos de preguntas

- *¿Cuáles son los requisitos económicos para postular?*
- *¿Qué cubre el financiamiento de la beca?*
- *¿Cuál es el monto del estipendio mensual?*
- *¿En qué casos se puede perder la beca?*
- *¿Qué obligaciones tiene el becario durante sus estudios?*

---

## Estructura del repositorio

```
beca18-rag-chatbot/
├── data/
│   └── beca18_reglamento.pdf          # PDF fuente (no incluido en el repo)
├── notebooks/
│   └── beca18_rag_chatbot.ipynb       # Notebook principal
├── .env.example                       # Plantilla de configuración de API key
├── .gitignore                         # Excluye .env y chroma_db_*/
├── requirements.txt                   # Dependencias con versiones fijadas
└── README.md
```

La carpeta `chroma_db_beca18/` se genera automáticamente al ejecutar el notebook y está excluida del repositorio por `.gitignore`.

---

## Modelos utilizados

| Componente | Modelo |
|---|---|
| Embeddings (indexación) | `gemini-embedding-001` — tarea `RETRIEVAL_DOCUMENT` |
| Embeddings (consulta) | `gemini-embedding-001` — tarea `RETRIEVAL_QUERY` |
| Generación de respuestas | `gemini-2.5-flash` |
| Dimensiones de vector | 768 |
| Distancia vectorial | Coseno (ChromaDB HNSW) |
