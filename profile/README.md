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

    %% 4. Core Node.js Services
    subgraph CoreServices["NestJS Microservices"]
        ACC["Accounts Service"]:::svc
        PROV["Provider Directory Service"]:::svc
        BOOK["Booking Service"]:::svc
        ORCH["Orchestrator Service"]:::svc
    end

    %% 5. AI Python Service
    subgraph AIServiceLayer["AI Engine"]
        AIWorker["AI RAG Worker (Python)"]:::ai
        SyncBatch["Batch Sync Script"]:::ai
    end

    %% 6. Infrastructure
    subgraph Infrastructure["Infrastructure"]
        Postgres[("PostgreSQL")]:::db
        Redis[("Redis")]:::db
        Qdrant[("Qdrant (Vector DB)")]:::db
    end

    %% 7. External LLM APIs
    subgraph ExternalAPIs["External AI Models"]
        OpenAI(["OpenAI"]):::ext
        Gemini(["Google Gemini"]):::ext
    end

    %% Relationships
    WebPatient & WebStaff --> Gateway
    Gateway -- "RPC" --> RMQ
    RMQ -- "Consume Task" --> AIWorker
    AIWorker -- "Search" --> Qdrant
    AIWorker -- "LLM" --> OpenAI & Gemini
    CoreServices -.-> Postgres & Redis
    PROV -- "Events" --> RMQ
    RMQ -- "Sync" --> AIWorker
    SyncBatch -.-> Gateway & Qdrant
````

## Explore the Repositories

| Repository | Primary Tech Stack | Description |
| :--- | :--- | :--- |
| [**medicalink-microservice**](https://github.com/MedicaLink-Healthcare/medicalink-microservice) | NestJS, Prisma, Redis, RabbitMQ, PostgreSQL | The core backend engine featuring 7 microservices, event-driven architecture, and Saga orchestration. |
| [**medicalink-ai-service**](https://github.com/MedicaLink-Healthcare/medicalink-ai-service) | Python, Qdrant | Intelligent recommendation worker implementing Hybrid Search RAG to match patients with doctors. |
| [**medicalink-fe-client**](https://github.com/MedicaLink-Healthcare/medicalink-fe-client) | React, TypeScript, Tailwind | Patient-facing portal for booking appointments and interacting with the AI medical assistant. |
| [**medicalink-fe-staff**](https://github.com/MedicaLink-Healthcare/medicalink-fe-staff) | React, TypeScript, TanStack, Shadcn | Advanced dashboard for Doctors and Admins to manage schedules, profiles, and hospital resources. |

## Key Architectural Highlights

  * **Logic & Resource Isolation:** By decoupling the **Python AI Service** from the **NestJS Core Microservices**, we ensure that heavy LLM computations do not impact critical booking transactions.
  * **Event-Driven Integration:** **RabbitMQ** manages both real-time **RPC requests** and **Asynchronous Events** (Pub/Sub) to keep the Qdrant vector index synchronized.
  * **Advanced RAG Pipeline:** Implements *Hybrid Search (Dense + Sparse) -\> FlashRank Reranking -\> Contextual Generation* to eliminate AI hallucinations.
  * **Infrastructure Efficiency:** Utilizing **PostgreSQL with Schema-level separation** provides isolation while significantly reducing operational costs.

-----

**Contact for work:** [dinhducbkdn2004@gmail.com](mailto:dinhducbkdn2004@gmail.com)
