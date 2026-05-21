# PropiedadesBA – AI Real Estate Chatbot

An AI-powered chatbot for a real estate agency in Córdoba, Argentina. Built with a full RAG (Retrieval-Augmented Generation) pipeline using n8n, Pinecone, OpenAI, and PostgreSQL. The chatbot answers questions about available properties, prices, locations, and agency information — in Spanish or English — with persistent memory across the conversation.

---

## Demo

**Searching for apartments:**

![Demo - Apartment search](https://raw.githubusercontent.com/Franciscojcdev/propiedadesba-rag-chatbot/main/demo-departamento.png)

**Searching for gated community houses:**

![Demo - Gated community](https://raw.githubusercontent.com/Franciscojcdev/propiedadesba-rag-chatbot/main/demo-barrio-cerrado.png)

**Contact information retrieval:**

![Demo - Contact info](https://raw.githubusercontent.com/Franciscojcdev/propiedadesba-rag-chatbot/main/demo-contacto.png)

---

## How It Works

The system is composed of two workflows:

### Workflow 1 — Document Ingestion Pipeline

Automatically processes property documents uploaded to Google Drive and loads them into a Pinecone vector database.

![Ingestion Workflow](https://raw.githubusercontent.com/Franciscojcdev/propiedadesba-rag-chatbot/main/ingestion-workflow.png)

```
Google Drive Trigger → Download File → Pinecone Vector Store
                                              ↑              ↑
                                    Embeddings OpenAI   Default Data Loader
                                                               ↑
                                                    Recursive Text Splitter
```

**Steps:**
1. Google Drive Trigger detects new files uploaded to a specific folder
2. File is downloaded and passed to the Default Data Loader
3. Text Splitter breaks the document into 500-token chunks with 50-token overlap
4. OpenAI generates embeddings (text-embedding-ada-002, 1536 dimensions)
5. Vectors are stored in Pinecone index `propiedadesba` under namespace `propiedades`

### Workflow 2 — RAG Chatbot

A conversational agent that retrieves relevant property information from Pinecone and responds to user queries with context-aware answers.

![Chatbot Workflow](https://raw.githubusercontent.com/Franciscojcdev/propiedadesba-rag-chatbot/main/chatbot-workflow.png)

```
Chat Trigger → AI Agent
                  ↑           ↑              ↑
          OpenAI GPT-4o   Postgres      Pinecone Vector Store (Tool)
             (Model)       Memory              ↑
                                       Embeddings OpenAI
```

**Steps:**
1. User sends a message via the public chat interface
2. AI Agent receives the message and retrieves conversation history from PostgreSQL
3. Pinecone Vector Store tool searches for relevant property documents using semantic similarity
4. Agent generates a response using GPT-4o with the retrieved context and conversation history
5. Response is returned to the user and conversation is stored in PostgreSQL

---

## Tech Stack

| Component | Tool |
|---|---|
| Workflow Automation | n8n (self-hosted on Railway) |
| Vector Database | Pinecone (Serverless, us-east-1) |
| Embeddings | OpenAI text-embedding-ada-002 |
| Chat Model | OpenAI GPT-4o-mini |
| Persistent Memory | PostgreSQL (Railway) |
| Document Storage | Google Drive |
| Infrastructure | Railway + Docker |

---

## Key Features

- **RAG Architecture** — answers are grounded in real property data, not hallucinated
- **Persistent Memory** — conversation history stored in PostgreSQL, context maintained across messages
- **Bilingual** — responds in Spanish or English depending on user input
- **Auto-ingestion** — new property documents added to Google Drive are automatically processed and indexed
- **Production-ready** — deployed on Railway with self-hosted n8n instance

---

## Example Conversations

**Property search:**
> User: ¿Tienen departamentos en Nueva Córdoba?
> Bot: En Nueva Córdoba, tenemos un departamento en venta que podría interesarte:
> **Departamento en Nueva Córdoba**
> - Tipo: Departamento 2 ambientes
> - Precio: USD 85,000
> - Dirección: Av. Hipólito Yrigoyen 1250, Piso 4...

**Follow-up with memory:**
> User: ¿Cuál es el precio?
> Bot: El precio del departamento en Nueva Córdoba es **USD 85,000**.

**Property type search:**
> User: ¿Tienen casas en barrios cerrados?
> Bot: Sí, tenemos casas en barrios cerrados. Aquí te muestro algunas opciones:
> **Casa en Barrio Cerrado Los Robles - Malagueño**
> - Precio: USD 210,000
> - Superficie cubierta: 180 m²...

---

## Knowledge Base

The knowledge base (`propiedadesba_knowledge_base.txt`) includes:
- 3 apartments for sale (Nueva Córdoba, General Paz, Centro)
- 2 houses in gated communities (Malagueño, Villa Allende)
- 2 commercial spaces (Alta Córdoba, Centro)
- 1 land plot (Jardín Espinosa)
- General agency information (services, hours, contact, buying process)

---

## Architecture Diagram

```
                    ┌─────────────────────────────────────┐
                    │         INGESTION PIPELINE           │
                    │                                      │
  Google Drive ──→  │  Download → Chunk → Embed → Store   │ ──→ Pinecone
  (property docs)   │                                      │     (propiedadesba)
                    └─────────────────────────────────────┘

                    ┌─────────────────────────────────────┐
                    │           RAG CHATBOT                │
                    │                                      │
  User Message ──→  │  Chat Trigger → AI Agent            │ ──→ Response
                    │                    │                 │
                    │              ┌─────┴──────┐          │
                    │              ↓            ↓          │
                    │         PostgreSQL    Pinecone       │
                    │          (memory)    (retrieval)     │
                    └─────────────────────────────────────┘
```

---

## Setup

### Prerequisites
- n8n instance (self-hosted or cloud)
- Pinecone account (free Starter plan works)
- OpenAI API key
- Google Drive API credentials (OAuth2)
- PostgreSQL database

### Configuration
1. Import both workflow JSON files into n8n
2. Configure credentials: OpenAI, Pinecone, Google Drive OAuth2, PostgreSQL
3. Create Pinecone index `propiedadesba` (Dense, 1536 dimensions, cosine metric)
4. Upload knowledge base document to your Google Drive folder
5. Run Workflow 1 to ingest documents into Pinecone
6. Publish Workflow 2 and share the chat URL

---

## Author

**Francisco Castro** — AI Automation Consultant  
[LinkedIn](https://linkedin.com/in/francisco-castro-126122191) | [Portfolio](https://franciscojcdev.github.io) | [GitHub](https://github.com/Franciscojcdev)
