%%{init: {'theme': 'base', 'themeVariables': {'primaryColor': '#7C3AED', 'primaryTextColor': '#fff'}}}%%

graph TB
    subgraph "Tenant A (smith-realty.platform.com)"
        TA_LP["Landing Page<br/>Brand: Smith Realty"]
        TA_CB["Chatbot<br/>Persona: Alex"]
        TA_AD["Admin Dashboard"]
    end

    subgraph "Tenant B (jones-homes.platform.com)"
        TB_LP["Landing Page<br/>Brand: Jones Homes"]
        TB_CB["Chatbot<br/>Persona: Maria"]
        TB_AD["Admin Dashboard"]
    end

    subgraph "Shared Platform Layer"
        GW["API Gateway<br/>Tenant Resolution Middleware"]
        
        subgraph "Shared Services"
            LS["Lead Service"]
            CS["Chatbot Service"]
            CF["Content Factory"]
            NE["Nurture Engine"]
        end

        subgraph "Shared Data (Row-Level Isolation)"
            DB["PostgreSQL<br/>tenantId on every row"]
            CACHE["Redis Cache<br/>tenant-prefixed keys"]
            VEC["Vector Store<br/>tenant namespaces"]
        end
    end

    subgraph "Per-Tenant Integrations"
        TA_HS["Tenant A HubSpot"]
        TA_TW["Tenant A Twilio"]
        TB_HS["Tenant B HubSpot"]
        TB_TW["Tenant B Twilio"]
    end

    TA_LP --> GW
    TA_CB --> GW
    TA_AD --> GW
    TB_LP --> GW
    TB_CB --> GW
    TB_AD --> GW

    GW --> LS
    GW --> CS
    GW --> CF
    GW --> NE

    LS --> DB
    CS --> VEC
    CF --> DB
    NE --> DB
    LS --> CACHE

    LS --> TA_HS
    LS --> TB_HS
    NE --> TA_TW
    NE --> TB_TW
