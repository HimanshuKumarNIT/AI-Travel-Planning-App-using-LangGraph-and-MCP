# ✈️ AI Travel Planning System using LangGraph & MCP

An AI-powered travel planning application that uses **LangGraph** to
orchestrate specialized agents and **Model Context Protocol (MCP)** to
connect those agents with external travel services.

## 🚀 Project Overview

A user can provide a natural-language request such as:

> Plan a 7-day Japan trip including flights, hotels and sightseeing
> under ₹2 lakhs.

The system: 1. Understands the travel request. 2. Retrieves flight
information. 3. Searches for hotel options. 4. Retrieves current weather
and forecast information. 5. Combines the collected information. 6.
Generates a structured day-by-day itinerary.

The agents are orchestrated sequentially using LangGraph, while external
capabilities are exposed through MCP servers.

## 🏗️ System Architecture

``` text
                         ┌──────────────────────┐
                         │      User Query      │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │   Streamlit Frontend │
                         └──────────┬───────────┘
                                    │
                                    ▼
                    ┌──────────────────────────────┐
                    │        LangGraph             │
                    │    Agent Orchestrator        │
                    └──────────────┬───────────────┘
                                   │
              ┌────────────────────┼────────────────────┐
              ▼                    ▼                    ▼
       ┌─────────────┐      ┌─────────────┐      ┌─────────────┐
       │ Flight Agent│      │ Hotel Agent │      │Weather Agent│
       └──────┬──────┘      └──────┬──────┘      └──────┬──────┘
              │                    │                    │
              ▼                    ▼                    ▼
       AviationStack MCP      Tavily MCP        Weather MCP
              │                    │                    │
              └────────────────────┼────────────────────┘
                                   │
                                   ▼
                         ┌──────────────────────┐
                         │  Itinerary Agent     │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │  Final Travel Plan   │
                         └──────────────────────┘

                         PostgreSQL Checkpoint
```

## 🤖 Multi-Agent Workflow

### 1. ✈️ Flight Agent

Responsible for flight-related information.

It interacts with the **AviationStack MCP server** for: - Airport
information - Airline information - Flight-related data

### 2. 🏨 Hotel Agent

Responsible for hotel research.

It uses **Tavily MCP** for web searches based on destination, duration,
requirements and budget.

### 3. 🌤️ Weather Agent

Responsible for destination extraction and weather information.

It: 1. Extracts the destination from the user query. 2. Calls the
Weather MCP server. 3. Retrieves current weather. 4. Retrieves forecast
information.

The custom Weather MCP server uses the **OpenWeather API**.

### 4. 🗺️ Itinerary Agent

The final agent combines: - Flight information - Hotel research -
Weather information - Original user requirements

It then generates a structured travel itinerary containing day-wise
activities, travel considerations, hotel recommendations, weather-aware
planning and budget considerations.

## 🔄 LangGraph Workflow

``` text
START
  │
  ▼
Flight Agent
  │
  ▼
Hotel Agent
  │
  ▼
Weather Agent
  │
  ▼
Itinerary Agent
  │
  ▼
 END
```

A shared `TravelState` contains:

``` text
messages
user_query
flight_results
hotel_results
weather_results
itinerary
llm_calls
```

## 🔌 Model Context Protocol (MCP)

MCP separates **AI reasoning** from **external tool capabilities**.

This project uses three MCP integrations.

### ✈️ AviationStack MCP

A local MCP server maintained inside:

``` text
aviationstack-mcp/
```

It provides aviation-related tools to the Flight Agent.

### 🔎 Tavily MCP

A remote MCP integration that provides web-search capabilities to the
Hotel Agent.

``` text
Hotel Agent
     │
     ▼
 Tavily MCP
     │
     ▼
 Web Search
```

### 🌤️ Custom Weather MCP

Implemented using FastMCP.

File:

``` text
custom_weather_mcp_server.py
```

Tools:

``` text
get_current_weather(city)
get_forecast(city)
```

Flow:

``` text
Weather Agent
      │
      ▼
Weather MCP Server
      │
      ▼
 OpenWeather API
```

## 🧠 LLM

The application uses a Groq-hosted LLM for: - Destination extraction -
Agent reasoning - Travel information interpretation - Itinerary
generation

Example:

``` python
llm = ChatGroq(
    model="openai/gpt-oss-120b"
)
```

## 💾 PostgreSQL Checkpointing

The project uses **PostgreSQL** with LangGraph's `PostgresSaver`.

``` text
LangGraph
    │
    ▼
PostgresSaver
    │
    ▼
PostgreSQL
```

A unique thread ID is used for each travel-planning session so graph
execution state can be persisted.

## 🖥️ Streamlit Interface

The frontend provides: - Travel query input - Agent execution - Live
pipeline progress - Flight results - Hotel results - Weather
information - Generated itinerary

Example:

``` text
🤖 Agent Pipeline — Live

✈️ Flight Agent
      ↓
🏨 Hotel Agent
      ↓
🌤️ Weather Agent
      ↓
🗺️ Itinerary Agent
      ↓
✅ Final Travel Plan
```

## 📁 Project Structure

``` text
AI-Travel-Planning-App-using-LangGraph-and-MCP/
│
├── aviationstack-mcp/
│   ├── src/
│   ├── tests/
│   ├── pyproject.toml
│   ├── requirements.txt
│   ├── README.md
│   ├── LICENSE
│   ├── .gitignore
│   └── uv.lock
│
├── custom_weather_mcp_server.py
├── main.py
├── mcp_client.py
├── frontend.py
├── requirements.txt
├── README.md
└── .gitignore
```

> `.env` and virtual environments are intentionally excluded from
> GitHub.

## 🛠️ Tech Stack

  Category              Technology
  --------------------- ------------------------------
  Language              Python
  Agent Orchestration   LangGraph
  Agent Framework       LangChain
  Tool Protocol         Model Context Protocol (MCP)
  LLM                   Groq
  Flight Data           AviationStack
  Web Search            Tavily
  Weather Data          OpenWeather
  Database              PostgreSQL
  Checkpointing         LangGraph PostgresSaver
  MCP Server            FastMCP
  Frontend              Streamlit
  Configuration         python-dotenv

## ⚙️ Installation

### 1. Clone the repository

``` bash
git clone <YOUR_GITHUB_REPOSITORY_URL>
cd AI-Travel-Planning-App-using-LangGraph-and-MCP
```

### 2. Create the main virtual environment

``` bash
python -m venv .venv
.venv\Scripts\activate
```

### 3. Install dependencies

``` bash
pip install -r requirements.txt
```

Required integrations if not already included:

``` bash
pip install psycopg[binary]
pip install langgraph-checkpoint-postgres
pip install langchain-mcp-adapters
```

## ✈️ AviationStack MCP Setup

The AviationStack MCP server is maintained inside:

``` text
aviationstack-mcp/
```

It has its own virtual environment.

``` bash
cd aviationstack-mcp
python -m venv .venv
.venv\Scripts\activate
```

Install its dependencies according to its `pyproject.toml` /
`requirements.txt`, then return:

``` bash
cd ..
```

> Do not commit `aviationstack-mcp/.venv/`.

## 🔐 Environment Variables

Create a `.env` file in the project root:

``` env
GROQ_API_KEY=your_groq_api_key
TAVILY_API_KEY=your_tavily_api_key
AVIATIONSTACK_API_KEY=your_aviationstack_api_key
OPENWEATHER_API_KEY=your_openweather_api_key
DATABASE_URL=postgresql://username:password@localhost:5432/database_name
```

Never commit `.env`.

Recommended `.gitignore`:

``` gitignore
.env
.venv/
__pycache__/
*.pyc
aviationstack-mcp/.venv/
```

## 🐘 PostgreSQL Setup

Create a PostgreSQL database:

``` sql
CREATE DATABASE travel_planner;
```

Configure:

``` env
DATABASE_URL=postgresql://postgres:YOUR_PASSWORD@localhost:5432/travel_planner
```

If the password contains URL-sensitive characters, encode them.

For example:

``` text
@  →  %40
```

## ▶️ Run the Application

From the project root:

``` bash
streamlit run frontend.py
```

Open:

``` text
http://localhost:8501
```

## 🧪 Example Query

``` text
Plan a 7-day Japan trip including flights, hotels and sightseeing under ₹2 lakhs.
```

Workflow:

``` text
User Query
    ↓
Flight Agent
    ↓
Hotel Agent
    ↓
Weather Agent
    ↓
Itinerary Agent
    ↓
Final Travel Plan
```

## 🔍 MCP Tool Flow

``` text
                    MCP Client
                        │
          ┌─────────────┼─────────────┐
          │             │             │
          ▼             ▼             ▼
      AviationStack   Tavily      Weather MCP
          │             │             │
          ▼             ▼             ▼
       Flights       Web Search   OpenWeather
```

`MultiServerMCPClient` handles communication between the application and
MCP servers.

## 🧩 Why LangGraph?

The application requires multiple specialized agents to execute as a
controlled workflow.

Instead of:

``` text
User
 ↓
One LLM
 ↓
Everything
```

the project uses:

``` text
User
 ↓
Flight Agent
 ↓
Hotel Agent
 ↓
Weather Agent
 ↓
Itinerary Agent
```

Benefits: - Modular agent design - Shared state - Explicit workflow -
Easier debugging - Extensible architecture - Persistent execution state

## 🔌 Why MCP?

Without MCP:

``` text
Agent
 ├── AviationStack API logic
 ├── Tavily API logic
 └── OpenWeather API logic
```

With MCP:

``` text
Agent
   │
   ▼
MCP Tool
   │
   ▼
External Service
```

This separates tool integration from agent reasoning and makes external
capabilities easier to replace or extend.

## 🎯 Key Features

-   **Multi-Agent Architecture** --- specialized agents handle different
    travel responsibilities.
-   **MCP-Based Tool Integration** --- external services are exposed
    through MCP.
-   **Custom Weather MCP Server** --- built using FastMCP.
-   **Web Research** --- Tavily MCP provides web-search capabilities.
-   **Flight Information** --- AviationStack MCP provides
    aviation-related information.
-   **Weather-Aware Planning** --- current weather and forecast can
    influence planning.
-   **Persistent Graph State** --- PostgreSQL with LangGraph
    checkpointing.
-   **Interactive UI** --- Streamlit-based interface.

## 🧱 Design Principles

### 1. Separation of Responsibilities

Each agent has a focused responsibility.

### 2. Tool/Reasoning Separation

MCP handles external tools while agents handle reasoning.

### 3. Shared State

LangGraph provides shared state between workflow nodes.

### 4. Modular Architecture

Agents and MCP servers can be modified independently.

### 5. Environment-Based Configuration

API keys and database credentials are kept outside source code.

## 📈 Future Improvements

The current workflow is sequential:

``` text
Flight → Hotel → Weather → Itinerary
```

Possible improvements: - Parallel execution of independent agents -
Budget validation agent - Transportation planning agent -
Activity/recommendation agent - Robust error handling and retries -
Conditional agent routing - Structured outputs using Pydantic -
Automated budget calculation - Date-aware flight search - Multi-city
itinerary planning - Authentication and user accounts - Docker
deployment - Production database configuration - Observability and
tracing

## ⚠️ Limitations

The travel plan depends on external APIs and web-search results.

Therefore: - Flight availability and prices can change. - Hotel prices
and availability can change. - Weather forecasts can change. - Web
results may be incomplete or outdated. - The itinerary is an AI-assisted
recommendation, not a confirmed booking.

The application does not directly book flights or hotels.

## 🔒 Security

API credentials are loaded through environment variables.

Sensitive files such as `.env` are excluded from version control.

Virtual environments are also excluded:

``` text
.venv/
aviationstack-mcp/.venv/
```

Never hardcode API keys or database credentials.

## 💡 Architecture Decisions

The AviationStack MCP server is kept as a separate component:

``` text
Main Application
      │
      ▼
MCP Client
      │
      ▼
AviationStack MCP Server
      │
      ▼
AviationStack API
```

The custom weather integration follows:

``` text
Weather Agent
      │
      ▼
Custom Weather MCP
      │
      ▼
OpenWeather API
```

This keeps external integrations modular and independent from the main
LangGraph workflow.


## ⭐ Project Highlights

> **LangGraph** orchestrates the multi-agent workflow.

> **MCP** provides a standardized interface between agents and external
> tools.

> **AviationStack MCP** handles aviation-related operations.

> **Tavily MCP** enables web research.

> **Custom Weather MCP** connects the system with OpenWeather.

> **Groq** provides the LLM reasoning layer.

> **PostgreSQL + PostgresSaver** provides persistent LangGraph state.

> **Streamlit** provides the interactive user interface.

## 📌 Quick Start

``` bash
git clone <YOUR_GITHUB_REPOSITORY_URL>
cd AI-Travel-Planning-App-using-LangGraph-and-MCP
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
streamlit run frontend.py
```

## ⭐ Final Result

A user can provide a single natural-language travel request, and the
system coordinates multiple specialized agents and MCP tools to
transform that request into an organized, weather-aware travel plan.

``` text
                  Natural Language Request
                           │
                           ▼
                    ┌──────────────┐
                    │  LangGraph   │
                    └──────┬───────┘
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
          Flights        Hotels       Weather
             │             │             │
          Aviation       Tavily      OpenWeather
             │             │             │
             └─────────────┼─────────────┘
                           ▼
                    Itinerary Agent
                           │
                           ▼
                   Complete Travel Plan
```
