# LPP_RAG
# São Paulo Travel RAG Assistant 
(CrewAI + DuckDB + Streamlit)

A Retrieval-Augmented Generation (RAG) assistant that answers travel-planning questions about **São Paulo, Brazil** using a curated document collection (guides, safety tips, itineraries, money/payment, language basics, and neighborhood recommendations). The app retrieves the most relevant passages from a **DuckDB vector store** and uses a **CrewAI agent powered by OpenAI `gpt-4o-mini`** to generate grounded, user-friendly responses with sources and similarity scores.

## Domain Overview & Problem Statement
Travel advice can be high-stakes: visitors need trustworthy guidance on **where to go, what to do, how to get around, and how to stay safe**. General-purpose chatbots often hallucinate details (prices, “must-do” lists, safety claims, logistics) or do not provide the best information.  
This project solves that by using a RAG pipeline: the assistant *retrieves relevant passages from a curated São Paulo travel knowledge base* and generates responses based on those sources.

## Architecture / Pipeline Explanation
**High-level flow**
1. **User asks a question** in the Streamlit chat interface.
2. The system converts the question into an embedding using **SentenceTransformers (`all-MiniLM-L6-v2`)**.
   - `all-MiniLM-L6-v2 was used for multilingual context.
3. The app queries a **DuckDB vector database** (`sp_vector.duckdb`) for the **top-k most similar chunks**.
4. A **CrewAI agent** decides when retrieval is needed (tool-based design) and uses retrieved context to answer.
5. The Streamlit UI displays:
   - the final answer
   - retrieved sources + **similarity scores** (transparency)

**Components**
- `app.py`: Streamlit UI (chat, sidebar settings, source display, session state)
- `backend/database.py`: DuckDB operations + embedding model loading + similarity query
- `backend/agent.py`: CrewAI agent + tool for retrieval + response generation
- `config.py`: central configuration (DB path, top_k, embedding model, max iterations, default LLM)

## Document Collection Summary (19 documents)
The DuckDB store contains **19 documents**, chunked into **510 total chunks**. The collection was chosen to support curated content and realistic tourist questions, covering:
- **Itineraries & general guides** (e.g., complete 4-day itinerary, “full travel guide”)
- **Food & neighborhoods** (e.g., Pinheiros / Paulista eating guides, essential foods)
- **Safety guidance** (safety tips for São Paulo / Brazil)
- **Logistics** (documents required to travel, money & payments, getting around SP)
- **Culture & communication** (useful Portuguese phrases, gestures/cultural notes)
- **Nightlife** (where to go out and how to navigate nightlife areas)
- **Seasonality** (visiting SP throughout the year)

Example titles in the dataset include information in English and Portuguese, such as:
- “WHAT TO DO in SÃO PAULO | COMPLETE 4-DAY ITINERARY…”
- “Dicas de Segurança – Visite São Paulo”
- “Como se locomover em São Paulo”
- “Nightlife SP”
- “Money & Payment Guide for Americans Visiting Brazil”
- “Best Areas to Choose a Hotel in São Paulo”
- “São Paulo – 140 lugares para você conhecer”

**Why these documents?**  
They reflect the most common visitor intents: itinerary planning, “must-see” attractions, neighborhood selection, safe transportation, food recommendations, and practical travel logistics—while keeping answers grounded in a finite, auditable source set.

## Agent Configuration (Role, Goal, Backstory) + Rationale
This project uses a **CrewAI agent** designed to behave like a practical, safety-conscious local planner.

- **Role:** São Paulo Travel Assistant  
- **Goal:** Provide helpful, realistic travel recommendations grounded in retrieved sources, prioritizing safety and practical logistics.  
- **Backstory / Behavior:** The agent is configured to **decide when database retrieval is needed** (instead of retrieving every time). When retrieval is used, the agent relies on the retrieved passages to reduce hallucinations and increase trustworthiness.  
- **Rationale:** Travel questions often mix subjective preferences (food, nightlife) with objective constraints (safety, documents, transport). Tool-based retrieval ensures factual grounding where needed, while still allowing natural, conversational planning.

## Query Demonstrations (UI Transparency)
The Streamlit app supports:
- a **chat interface**
- **example questions** to quickly demo the assistant
- a retrieved-sources panel that shows:
  - each source chunk
  - **similarity score**
  - (and any metadata exposed in the UI)

This is intended to make the assistant’s reasoning auditable to non-technical users.

## Installation & Setup

### 1) Create environment and install dependencies
- pip install -r requirements.txt´
### 2) Verify configuration
Key settings live in config.py, including:
  - DEFAULT_DB_PATH (DuckDB file path)
  - DEFAULT_TOP_K (retrieval depth)
  - DEFAULT_MODEL (gpt-4o-mini)
  - embedding model name + dimension
    - Note: ensure DEFAULT_DB_PATH correctly points to your DuckDB file location (e.g., backend/sp_vector.duckdb or wherever you placed it in your repo).
### 3) Run the Streamlit app
- streamlit run app.py

# Link to Streamlit deployment: https://lpprag-dgbrzupe6eptc5y2zdyr5l.streamlit.app/
