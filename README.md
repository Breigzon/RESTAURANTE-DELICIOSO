# 🤖 Alura Agente

Agente de inteligencia artificial que responde preguntas en lenguaje natural
sobre un documento interno (PDF), usando una arquitectura **RAG
(Retrieval-Augmented Generation)**. Proyecto desarrollado como Challenge
Final del programa AlurAgente.

## 🧠 Arquitectura

El agente funciona en dos fases:

**1. Ingesta (`src/ingest.py`)** — se ejecuta una sola vez por documento:
1. `PyPDFLoader` carga el PDF y extrae el texto de cada página.
2. `RecursiveCharacterTextSplitter` divide el texto en fragmentos (~1000
   caracteres, con solapamiento de 150) para preservar el contexto.
3. Cada fragmento se convierte en un vector numérico (*embedding*) con
   `GoogleGenerativeAIEmbeddings` (modelo `embedding-001` de Gemini).
4. Los vectores se guardan en una base de datos vectorial local con
   **Chroma**.

**2. Consulta (`src/agent.py` + `app.py`)** — en cada pregunta:
1. La pregunta del usuario se convierte también en un vector.
2. Chroma busca los 4 fragmentos del documento más similares (más
   relevantes) a esa pregunta.
3. Esos fragmentos se insertan como contexto en un prompt.
4. **Gemini** (`gemini-1.5-flash`) redacta la respuesta usando *solo* la
   información de ese contexto — si no encuentra la respuesta en el
   documento, lo indica en vez de inventar datos.
5. La interfaz web (**Streamlit**) muestra la conversación en formato chat.

```
PDF ──▶ ingest.py ──▶ Chroma (base vectorial local)
                              │
Usuario ──▶ app.py ──▶ agent.py ──▶ busca contexto en Chroma ──▶ Gemini ──▶ Respuesta
```

## 🛠️ Tecnologías utilizadas

| Componente        | Tecnología                              |
|--------------------|------------------------------------------|
| Lenguaje           | Python                                   |
| Orquestación       | LangChain                                |
| Lectura de PDF     | PyPDF                                    |
| Embeddings + LLM   | Google Gemini (`embedding-001`, `gemini-1.5-flash`) |
| Base vectorial     | Chroma                                   |
| Interfaz           | Streamlit                                |
| Deploy             | Streamlit Community Cloud                |

## 📁 Documentos fuente incluidos

El agente responde sobre 5 documentos internos de **Restaurante Delicioso**, ya
incluidos en `data/` (todos en PDF, listos para la ingesta):

- `manual_proveedores_delicioso.pdf` — política de compras y proveedores
- `politica_atencion_cliente_delicioso.pdf` — atención al cliente, reservas y reembolsos
- `reglamento_interno_delicioso.pdf` — reglamento interno del personal
- `preguntas_frecuentes_delicioso.pdf` — FAQ del restaurante
- `carta_criolla_delicioso.pdf` — carta completa (Platos Criollos, A la Carta, Vegetarianos) con precios y alérgenos

(Las versiones originales en Word/Excel, por si necesitas editarlas, están en
`docs_fuente_word/` — no se usan en la ingesta, que solo lee PDFs.)

## 💬 Ejemplos de preguntas y respuestas

| Pregunta | Respuesta esperada del agente |
|---|---|
| "¿Cuál es el horario de atención de Delicioso?" | Todos los días de 12:00 p. m. a 10:00 p. m., para local y delivery. |
| "¿Qué documentos debe presentar un nuevo proveedor?" | RUC vigente, registro sanitario o autorización DIGESA (para alimentos), certificado de manipulación de alimentos, datos bancarios y al menos 2 referencias comerciales. |
| "¿Cuánto cuesta el Lomo Saltado y qué alérgenos tiene?" | S/. 32.00; contiene soya y sulfitos. |
| "¿Qué opciones vegetarianas de fondo tienen y cuál es la más barata?" | Tacu Tacu Vegetariano (S/. 24), Ají de Habas (S/. 22), Quinotto de Champiñones (S/. 28), Lomo Saltado de Champiñones (S/. 26) y Milanesa de Berenjena (S/. 24). La más barata es el Ají de Habas, a S/. 22.00. |
| "¿Cuál es la tolerancia de tardanza para el personal?" | 10 minutos sobre la hora de ingreso; la tercera tardanza en 30 días se considera falta injustificada. |

## 🚀 Cómo ejecutar el proyecto localmente

1. Clona el repositorio:
   ```bash
   git clone https://github.com/Breigzon/RESTAURANTE-DELICIOSO.git
   cd alura-agente
   ```

2. Crea un entorno virtual e instala las dependencias:
   ```bash
   python -m venv venv
   source venv/bin/activate  # En Windows: venv\Scripts\activate
   pip install -r requirements.txt
   ```

3. Configura tu API key gratuita de Google Gemini:
   - Consíguela en https://aistudio.google.com/apikey
   - Copia `.env.example` a `.env` y pega tu clave:
     ```
     GOOGLE_API_KEY=tu_clave_aqui
     ```

4. Genera la base vectorial a partir de los PDFs ya incluidos en `data/`
   (o coloca los tuyos ahí primero):
   ```bash
   python src/ingest.py --pdf-dir data/
   ```
   También puedes ingerir un único PDF con `python src/ingest.py --pdf data/archivo.pdf`.

5. Ejecuta la aplicación:
   ```bash
   streamlit run app.py
   ```

6. Abre `http://localhost:8501` y empieza a preguntar.

## ☁️ Deploy

La aplicación está desplegada públicamente en Streamlit Community Cloud:

**🔗 URL: `https://restaurante-delicioso-nvvvgqwyrjauo7wbnsgpdv.streamlit.app/`**

<img width="1917" height="1078" alt="image" src="https://github.com/user-attachments/assets/623d9778-6fa3-4cae-9bf7-0d57cb05acf0" />


### Pasos para replicar el deploy
1. Sube este repositorio a GitHub (con `chroma_db/` ya generado y commiteado,
   o genera la ingesta en el primer arranque — ver nota abajo).
2. Entra a https://share.streamlit.io y conecta tu cuenta de GitHub.
3. Selecciona el repositorio y el archivo principal: `app.py`.
4. En **Advanced settings → Secrets**, agrega:
   ```
   GOOGLE_API_KEY = "tu_clave_aqui"
   ```
5. Deploy. En un par de minutos la app queda disponible con una URL pública.

## 📁 Estructura del proyecto

```
alura-agente/
├── app.py              # Interfaz Streamlit
├── src/
│   ├── ingest.py        # Procesa el PDF y crea la base vectorial
│   └── agent.py          # Lógica del agente (retriever + Gemini)
├── data/                 # PDF(s) fuente
├── requirements.txt
├── .env.example
└── README.md
```

## 👤 Autor

Proyecto desarrollado por Breigzon Turco como Challenge Final del programa
AlurAgente.
