# Technical Architecture — CodePocket

## System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        MOBILE APP (Flutter)                      │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────────┐   │
│  │  AI Chat  │ │ Code     │ │ Search   │ │ Offline Cache    │   │
│  │  (Voice/  │ │ Browser  │ │ (Semantic│ │ (SQLite/Hive)    │   │
│  │  Text/    │ │ (IRC,NEC │ │ + Keyword│ │                  │   │
│  │  Photo)   │ │  IPC...) │ │ hybrid)  │ │                  │   │
│  └─────┬─────┘ └────┬─────┘ └────┬─────┘ └────────┬─────────┘   │
│        │             │            │                  │             │
│        └─────────────┴────────────┴──────────────────┘             │
│                              │                                     │
└──────────────────────────────┼─────────────────────────────────────┘
                               │ HTTPS/REST
                               ▼
┌──────────────────────────────────────────────────────────────────┐
│                     API GATEWAY (Cloudflare)                      │
└──────────────────────────┬───────────────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────────────────┐
│                    BACKEND (FastAPI / Python)                     │
│                                                                   │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌──────────────┐  │
│  │  Auth       │ │  Code Q&A  │ │  Code       │ │  Subscription│  │
│  │  (Firebase/ │ │  (RAG +    │ │  Search     │ │  (RevenueCat │  │
│  │   Supabase) │ │   LLM)     │ │  (Hybrid)   │ │   webhook)   │  │
│  └────────────┘ └─────┬──────┘ └──────┬─────┘ └──────────────┘  │
│                       │               │                           │
│  ┌────────────────────┴───────────────┴──────────────────────┐  │
│  │                    CODE DATA LAYER                         │  │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────────────────────┐│  │
│  │  │PostgreSQL│  │ Vector DB│  │ ICC API / NFPA / IAPMO   ││  │
│  │  │(metadata,│  │ (Pinecone│  │ (external code sources)  ││  │
│  │  │ users,   │  │  semantic│  │                          ││  │
│  │  │ saves)   │  │  search) │  │                          ││  │
│  │  └──────────┘  └──────────┘  └──────────────────────────┘│  │
│  └───────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
```

## Data Model

### Core Entities

```sql
-- Users
CREATE TABLE users (
    id UUID PRIMARY KEY,
    email TEXT UNIQUE,
    display_name TEXT,
    trade TEXT,  -- electrician, plumber, gc, handyman, inspector, other
    state TEXT,  -- primary state
    subscription_tier TEXT DEFAULT 'free',
    created_at TIMESTAMP,
    last_active TIMESTAMP
);

-- Code Sources
CREATE TABLE code_sources (
    id UUID PRIMARY KEY,
    publisher TEXT,  -- ICC, NFPA, IAPMO, state, local
    code_family TEXT,  -- IBC, IRC, NEC, UPC, IPC, IMC, IFGC, IECC
    edition_year INT,
    jurisdiction_level TEXT,  -- federal, state, county, city
    jurisdiction_id TEXT,  -- FIPS code or state abbreviation
    source_url TEXT,
    last_updated TIMESTAMP
);

-- Code Sections (ingested from APIs)
CREATE TABLE code_sections (
    id UUID PRIMARY KEY,
    source_id UUID REFERENCES code_sources(id),
    section_number TEXT,  -- "210.52(D)"
    chapter TEXT,
    title TEXT,
    full_text TEXT,
    summary TEXT,  -- AI-generated summary for search
    created_at TIMESTAMP,
    updated_at TIMESTAMP
);

-- AI Q&A History
CREATE TABLE qa_history (
    id UUID PRIMARY KEY,
    user_id UUID REFERENCES users(id),
    question TEXT,
    answer TEXT,
    cited_sections TEXT[],  -- array of section UUIDs
    jurisdiction TEXT,
    model_used TEXT,
    latency_ms INT,
    feedback_score INT,  -- 1-5 rating from user
    created_at TIMESTAMP
);

-- Saved Codes
CREATE TABLE saved_codes (
    id UUID PRIMARY KEY,
    user_id UUID REFERENCES users(id),
    section_id UUID REFERENCES code_sections(id),
    notes TEXT,
    folder TEXT,
    created_at TIMESTAMP
);

-- Jurisdictions
CREATE TABLE jurisdictions (
    id TEXT PRIMARY KEY,  -- FIPS or state code
    name TEXT,
    level TEXT,  -- state, county, city
    parent_id TEXT REFERENCES jurisdictions(id),
    adopted_codes JSONB,  -- {code_family: {edition: year, effective: date}}
    amendments_count INT DEFAULT 0
);
```

## RAG Pipeline (Code Q&A)

```
User Question: "What's the max distance between outlets in a bathroom?"
                              │
                              ▼
                    ┌─────────────────────┐
                    │  Query Enhancement   │
                    │  - Detect trade      │
                    │  - Detect intent     │
                    │  - Identify state    │
                    │  - Expand to code    │
                    │    terms             │
                    └─────────┬───────────┘
                              │
                              ▼
                    ┌─────────────────────┐
                    │  Dual Search         │
                    │  ┌───────────────┐   │
                    │  │ Semantic Search│   │
                    │  │ (Pinecone)     │   │
                    │  └───────┬───────┘   │
                    │  ┌───────────────┐   │
                    │  │ Keyword Search│   │
                    │  │ (Postgres FTS)│   │
                    │  └───────┬───────┘   │
                    │          │           │
                    │     Merge + Rank     │
                    └─────────┬───────────┘
                              │
                              ▼
                    ┌─────────────────────┐
                    │  Context Assembly    │
                    │  - Top 10 sections   │
                    │  - Jurisdiction data │
                    │  - Amendment notes   │
                    │  - Related sections  │
                    └─────────┬───────────┘
                              │
                              ▼
                    ┌─────────────────────┐
                    │  LLM Generation     │
                    │  (GPT-4o-mini or    │
                    │   Claude Haiku)     │
                    │                     │
                    │  System prompt:     │
                    │  "You are a code    │
                    │   expert. Answer    │
                    │   with citations.   │
                    │   If unsure, say    │
                    │   so. Never make    │
                    │   up section #'s."  │
                    └─────────┬───────────┘
                              │
                              ▼
                    ┌─────────────────────┐
                    │  Validation Pass    │
                    │  - Verify citations │
                    │  - Check section #s │
                    │  - Confidence score │
                    └─────────┬───────────┘
                              │
                              ▼
                    Response to User
                    (with cited sections
                     and confidence level)
```

## Flutter App Structure

```
lib/
├── main.dart
├── app.dart
├── core/
│   ├── theme/
│   ├── constants/
│   ├── utils/
│   └── network/
├── features/
│   ├── auth/
│   │   ├── data/
│   │   ├── domain/
│   │   └── presentation/
│   ├── chat/
│   │   ├── data/           # API calls
│   │   ├── domain/         # Models, use cases
│   │   └── presentation/   # Chat UI, voice input
│   ├── codes/
│   │   ├── data/           # Code repository, local DB
│   │   ├── domain/         # Code models
│   │   └── presentation/   # Browser UI, section detail
│   ├── search/
│   │   └── presentation/   # Search bar, results
│   ├── saved/
│   │   └── presentation/   # Bookmarks, folders
│   ├── settings/
│   │   └── presentation/   # Jurisdiction, trade, subscription
│   └── onboarding/
│       └── presentation/   # Trade selection, state selection
├── services/
│   ├── api_client.dart
│   ├── offline_sync.dart
│   ├── voice_input.dart
│   └── camera_input.dart
└── widgets/
    ├── code_card.dart
    ├── citation_chip.dart
    └── chat_bubble.dart
```

## Offline Strategy

1. **Code sections** are cached locally in SQLite after first view
2. **AI Q&A** requires internet (can't run LLM offline)
3. **Recently viewed codes** available offline
4. **Saved/bookmarked codes** always available offline
5. **Search** works offline for cached sections (keyword only, not semantic)
6. **Sync queue** — actions taken offline are queued and synced when back online

## Performance Targets

| Metric | Target |
|--------|--------|
| AI Q&A response time | < 3 seconds |
| Code search results | < 500ms |
| App launch to interactive | < 2 seconds |
| Offline code lookup | < 100ms |
| App size (download) | < 30MB |
| Battery usage (active) | < 5% per hour |

## Security Considerations

- All API communication over HTTPS/TLS 1.3
- Firebase Auth with OAuth (Google, Apple Sign-In)
- No code content stored on device longer than cache TTL
- User data encrypted at rest (both server and device)
- API rate limiting to prevent abuse
- Subscription verification server-side (not client-trusted)
