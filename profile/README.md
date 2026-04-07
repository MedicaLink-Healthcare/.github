# MedicaLink Healthcare Ecosystem
**A High-Performance Microservices Platform & AI-Powered Doctor Recommendation Engine**

---

## System Architecture & AI RAG Flow

The Medicalink platform employs a distributed Microservices architecture. The core transaction services run on Node.js (NestJS), while the heavy AI computations and vector searches are isolated in a dedicated Python service.

```mermaid
flowchart TD
    %% Define styles for distinct components
    classDef client fill:#F5F3FF,stroke:#8B5CF6,stroke-width:1px,color:#4C1D95;
    classDef gw fill:#EFF6FF,stroke:#3B82F6,stroke-width:1px,color:#1E3A8A;
    classDef svc fill:#ECFDF5,stroke:#10B981,stroke-width:1px,color:#064E3B;
    classDef ai fill:#FEF2F2,stroke:#EF4444,stroke-width:1px,color:#7F1D1D;
    classDef bus fill:#FEF9C3,stroke:#EAB308,stroke-width:1px,color:#713F12;
    classDef db fill:#FFF7ED,stroke:#FB923C,stroke-width:1px,color:#7C2D12;
    classDef ext fill:#F3F4F6,stroke:#6B7280,stroke-width:1px,color:#1F2937,stroke-dasharray: 5 5;

    %% 1. Client Layer
    subgraph Clients["Frontend Clients"]
        WebPatient["Web Client (Patient)"]:::client
        WebStaff["Web Portal (Admin/Doctor)"]:::client
    end

    %% 2. Entry Point
    subgraph GatewayLayer["API Layer"]
        Gateway["API Gateway (NestJS)"]:::gw
    end

    %% 3. Message Broker
    subgraph EventBus["Asynchronous Communication"]
        RMQ{"RabbitMQ (AMQP)"}:::bus
    end

    %% 4. Core Node.js Services (Monorepo)
    subgraph CoreServices["NestJS Microservices (medicalink-microservice)"]
        ACC["Accounts Service"]:::svc
        PROV["Provider Directory Service"]:::svc
        BOOK["Booking Service"]:::svc
        ORCH["Orchestrator Service"]:::svc
    end

    %% 5. AI Python Service (Standalone Repo)
    subgraph AIServiceLayer["AI Recommendation Engine (medicalink-ai-service)"]
        AIWorker["AI RAG Worker (Python/LangChain)"]:::ai
        SyncBatch["Batch Sync Script"]:::ai
    end

    %% 6. Data Persistence & Cache
    subgraph Infrastructure["Databases & Caching"]
        Postgres[("PostgreSQL (Schema-separated)")]:::db
        Redis[("Redis (Cache & Queue)")]:::db
        Qdrant[("Qdrant (Vector Database)")]:::db
    end

    %% 7. External LLM APIs
    subgraph ExternalAPIs["External AI Models"]
        OpenAI(["OpenAI (Embeddings & Chat)"]):::ext
        Gemini(["Google Gemini (Chat)"]):::ext
    end

    %% --- Relationships and Data Flow ---

    %% Client to Gateway
    WebPatient -- "HTTP POST (Symptoms)" --> Gateway
    WebStaff -- "HTTP/REST" --> Gateway

    %% Gateway to Microservices & AI
    Gateway -- "RPC: account.* / booking.*" --> ACC & BOOK
    Gateway -- "RPC: ai.doctor-recommendation.request" --> RMQ

    %% Core Services DB Access
    ACC -.-> |"schema: accounts"| Postgres
    PROV -.-> |"schema: provider"| Postgres
    BOOK -.-> |"schema: booking"| Postgres
    ACC & PROV -.-> |"Cache"| Redis

    %% Event-driven AI Ingestion
    PROV -- "Publishes: doctor.profile.created/updated" --> RMQ
    RMQ -- "Consumes Event" --> AIWorker
    AIWorker -- "1. Embed text" --> OpenAI
    AIWorker -- "2. Upsert Vector (Hybrid)" --> Qdrant

    %% RAG Retrieval Flow
    RMQ -- "RPC Request (Symptoms)" --> AIWorker
    AIWorker -- "1. Embed Question" --> OpenAI
    AIWorker -- "2. Hybrid Search (Dense+Sparse) & Rerank" --> Qdrant
    AIWorker -- "3. Generate Contextual Response" --> OpenAI & Gemini
    AIWorker -- "4. Return JSON (doctor_id)" --> RMQ
    RMQ -- "Reply" --> Gateway

    %% Batch Sync Flow
    SyncBatch -.-> |"GET /api/doctors/profile/public"| Gateway
    SyncBatch -.-> |"Batch Upsert Vectors"| Qdrant
