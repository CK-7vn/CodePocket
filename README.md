# CodePocket — AI-Powered Building Code Assistant for Contractors

> **Idea Date:** May 10, 2026
> **Proposed by:** Clyde Kallahan
> **Status:** Research Complete — Ready for Validation

---

## The Problem

Every contractor, handyman, electrician, plumber, and home inspector in America has to navigate a tangled mess of building codes on a daily basis:

- **Federal codes** (ADA, OSHA, EPA)
- **State codes** (adopted versions of IBC, IRC, NEC, IPC, UPC, etc.)
- **Local amendments** (city/county modifications that override state code)
- **Multiple code years** (2021, 2018, 2015 — jurisdictions don't upgrade simultaneously)

Right now, looking up a code question means:
1. Pull out a 900-page codebook (if you even own the right one)
2. Navigate ICC, NFPA, or IAPMO websites (clunky, behind paywalls)
3. Call the local building department and hope someone picks up
4. Google it and hope the forum answer is current and jurisdiction-correct
5. Guess, get it wrong, fail inspection, redo the work

**This costs contractors real money.** A failed inspection means a return trip, wasted materials, and delayed payment. A single code violation can cost $500-$5,000+ in rework.

## The Solution

**CodePocket** — a mobile app that lets any contractor or handyman:
1. **Ask a question in plain English** → "What's the maximum distance between electrical outlets in a bathroom in Maine?"
2. **Get an instant, cited answer** → "NEC 210.52(D): At least one receptacle must be installed within 36" of the outside edge of each basin. Maine adopts the 2020 NEC with amendments (Chapter 12, Section 1)."
3. **See the relevant code section** → Full text, with the specific paragraph highlighted
4. **Browse local amendments** → What Maine changed from the base NEC
5. **Save and share** → Bookmark it, generate a report, share with the inspector

### Key Differentiator: Built for the Field, Not the Office

Existing solutions are designed for architects and engineers — desktop-first, expensive, and complex. CodePocket is designed for the guy standing on a job site with gloves on who needs an answer in 30 seconds.

---

## Market Size

### Total Addressable Market (TAM)

| Segment | Count | Source |
|---------|-------|--------|
| US construction businesses | 3.8M | IBISWorld 2025 |
| Licensed contractors (GC, plumbing, electrical, HVAC) | ~1.2M | BLS + state licensing boards |
| Handymen / handyperson businesses | 529K | IBISWorld 2026 |
| Home inspectors | ~30K | ASHI/InterNACHI estimates |
| Building code officials / inspectors | ~100K | ICC membership |
| Apprentices & trade students | ~500K | DOL registered apprentices |

**Total potential users: ~6M+ in the US alone**

### Market Value

| Metric | Value |
|--------|-------|
| US Construction Industry (2025) | $2.2 trillion |
| Handyman Services Industry (2025) | $273.6 billion |
| Building Code Compliance Software Market (2024) | $1.74 billion |
| Projected (2033) | $5.12 billion (12.8% CAGR) |
| On-demand handyman app market (2025) | $4.8 billion |

### Serviceable Market

If we target individual contractors and small businesses (1-10 employees) who don't currently have enterprise code compliance tools:
- **SAM:** ~$350-700M (20-40% of construction compliance software market)
- **SOM (Year 1-2):** $2-5M with 50K-100K paying users at $5-10/month

---

## Competitive Landscape

### Direct Competitors

| Competitor | Type | Price | Target User | Gaps |
|-----------|------|-------|-------------|------|
| **CodePro** | iOS app, AI Q&A | $4-30/mo | All trades | iOS only, limited local amendments, token-based pricing |
| **UpCodes** | Web + mobile, database | $19-39/mo | Architects/Engineers | 800K MAU, desktop-first, expensive for small contractors |
| **Melt Code** | Web platform, AI | TBD | A/E design teams | Enterprise focus, no mobile app |
| **ICC Digital Codes** | Mobile app, PDF viewer | Subscription | Code officials | Just a digital codebook — no AI, no Q&A |
| **OneClick Code** | Mobile app | Free/freemium | Roofers | Roofing codes only |
| **CodeComply.ai** | Desktop AI plan review | Enterprise | Architects | AI reviews plans, not field Q&A |
| **NFPA LiNK** | Web platform + AI | Subscription | Fire safety pros | NFPA codes only, no mobile |

### The Gap Nobody Owns

| Feature | CodePro | UpCodes | Melt Code | **CodePocket** |
|---------|---------|---------|-----------|----------------|
| Mobile-first design | Partial | Partial | ✗ | **✓** |
| AI Q&A with citations | ✓ | ✓ (Copilot) | ✓ | **✓** |
| All trades (elec, plumb, HVAC, building) | Partial | ✓ | ✓ | **✓** |
| Local amendments | Limited | ✓ | ✓ | **✓** |
| Affordable for solo contractors | $4-10 | ✗ ($19+) | Likely ✗ | **✓** |
| Voice/photo input | ✓ | ✗ | ✗ | **✓** |
| Offline access | Limited | ✓ | ✗ | **✓** |
| Android + iOS | iOS only | Both | Web only | **Both** |
| Code change alerts | ✗ | ✓ | ✓ | **✓** |
| Inspector-ready reports | ✗ | Partial | ✓ | **✓** |

### Key Insight

**CodePro is the closest competitor** but is iOS-only, charges per question, and has limited local amendment coverage. **UpCodes has 800K MAU** but charges $19-39/mo and is desktop-first for architects. **Nobody has built a $5-10/mo, mobile-first, AI-powered code assistant for the solo contractor/handyman market.**

---

## Technical Feasibility

### Data Sources

| Source | What it provides | Access method | Cost |
|--------|-----------------|---------------|------|
| **ICC Code Connect API** | IBC, IRC, IPC, IMC, IFGC, IECC + state amendments | REST API (JSON), OAuth2.0 | Subscription (contact ICC for pricing) |
| **ICC Code Adoption API** | Which codes each jurisdiction has adopted | REST API (JSON) | Subscription |
| **NFPA LiNK** | NEC (NFPA 70), NFPA 1, 101, etc. | Platform (no public API) | Subscription ($150-300/yr per title) |
| **IAPMO Codes** | UPC, UMC | Mobile app / digital purchase | Per-code purchase |
| **eCode360 API** | Local municipal codes | REST API (JSON) | Per-municipality |
| **State websites** | State-specific amendments | Web scraping / manual curation | Free (labor-intensive) |
| **Public law databases** | Federal codes (ADA, OSHA, EPA) | Government APIs / scraping | Free |

### Legal Considerations

**Building codes have a complicated copyright status:**

- In **Veeck v. Southern Building Code Congress** (2002), the 5th Circuit ruled that model codes **become public domain when enacted into law** by a jurisdiction
- However, the **model codes themselves** (before adoption) are copyrighted by ICC, NFPA, and IAPMO
- ICC offers free "read-only" access to many codes at codes.iccsafe.org (no API, no bulk download)
- NFPA provides free "view-only" access for some standards (no print/save/search)
- **Strategy:** Use official API partnerships for primary access, supplement with publicly available enacted codes

**Recommended approach:**
1. **Phase 1:** Partner with ICC via Code Connect API (covers ~90% of building codes)
2. **Phase 2:** License NFPA content for NEC/fire codes
3. **Phase 3:** Web-scrape and curate local amendments from municipal websites (with eCode360 API where available)

### Proposed Tech Stack

#### Mobile App
- **Framework:** Flutter (cross-platform iOS + Android from one codebase)
- **Language:** Dart
- **Why Flutter:** 
  - Single codebase for iOS + Android
  - Excellent offline capabilities (SQLite + Hive)
  - Great for form-heavy, reference-style apps
  - Clyde already has Flutter experience from SquishyScout

#### Backend
- **API Server:** Python (FastAPI) on Railway or Fly.io
- **Database:** PostgreSQL (user data, subscriptions, saved codes)
- **Vector DB:** Pinecone or Weaviate (code embeddings for semantic search)
- **LLM Integration:** OpenAI GPT-4o-mini or Claude Haiku (fast, cheap, good at retrieval-augmented generation)
- **RAG Pipeline:** LangChain or LlamaIndex for code-aware Q&A with citations

#### Data Pipeline
- **Code ingestion:** Python scripts to pull from ICC API, parse PDF, extract sections
- **Embedding generation:** OpenAI text-embedding-3-small for semantic search
- **Amendment tracking:** Scheduled jobs to detect code changes and notify users
- **Caching layer:** Redis for hot code sections (repeated queries)

#### Infrastructure
- **Hosting:** Railway (Python backend) + Cloudflare (CDN/static)
- **Auth:** Firebase Auth or Supabase Auth
- **Payments:** RevenueCat (handles iOS/Android subscriptions)
- **Push notifications:** Firebase Cloud Messaging
- **Analytics:** PostHog (self-hosted) or Mixpanel
- **Monitoring:** Sentry

#### Estimated Monthly Costs (at 10K users)
| Component | Cost/mo |
|-----------|---------|
| Backend hosting (Railway) | $20-50 |
| LLM API (GPT-4o-mini) | $100-300 |
| ICC API licensing | TBD (contact required) |
| Database (Postgres + Redis) | $20-40 |
| Vector DB (Pinecone free tier) | $0-70 |
| Firebase/Supabase | $0-25 |
| RevenueCat | $0 (revenue share) |
| **Total (excl. ICC)** | **$140-485/mo** |

---

## ASO / SEO Strategy

### App Store Optimization (ASO)

**Primary keywords (high intent):**
- building code app
- code lookup
- contractor code reference
- NEC app
- electrical code app
- plumbing code app
- building inspector app
- code compliance
- IRC code reference
- local building codes

**Secondary keywords (discovery):**
- contractor tools
- handyman app
- construction reference
- electrician app
- plumber app
- code checker
- permit requirements
- inspection checklist

**App title options:**
- CodePocket — Building Codes & AI Assistant
- CodePocket — Contractor Code Lookup
- CodePocket — Building Code AI

### SEO Strategy (Website)

**Target pages:**
- `/codes/[state]/[code-type]` — State-specific code pages (50 states × 8 code types = 400 pages)
- `/questions/[topic]` — FAQ-style pages answering common code questions
- `/blog/` — Educational content about code changes, tips, industry news

**Content strategy:**
- "What electrical code does [state] use?"
- "2024 NEC changes every electrician needs to know"
- "How to pass a building inspection the first time"
- "[State] building code requirements for decks"

---

## Business Model

### Pricing Tiers

| Tier | Price | Included |
|------|-------|----------|
| **Free** | $0 | 5 AI questions/month, basic code search, 1 jurisdiction |
| **Pro** | $9.99/mo ($89/yr) | Unlimited AI questions, all jurisdictions, offline access, code change alerts |
| **Team** | $29.99/mo | Up to 5 users, shared bookmarks, team reports, priority support |
| **Inspector** | $49.99/mo | Full code library, report generation, inspector-ready PDFs, API access |

### Revenue Projections (Conservative)

| Year | Users | Paying % | Revenue |
|------|-------|----------|---------|
| Year 1 | 50K | 8% | $480K |
| Year 2 | 200K | 12% | $2.9M |
| Year 3 | 500K | 15% | $9M |

### Revenue Drivers
1. **Subscriptions** — primary revenue
2. **ICC/NFPA referral commissions** — when users need full codebook access
3. **Contractor insurance partnerships** — code compliance reduces claims
4. **Training/CEU marketplace** — contractors need continuing education credits
5. **Enterprise/API licensing** — sell to larger firms and inspection software companies

---

## Go-to-Market Strategy

### Phase 1: Launch (Months 1-3)
- Launch on iOS and Android
- Focus on **one state** (Maine — home market, easy to validate locally)
- Partner with 2-3 local contractors for beta testing and testimonials
- Post in r/electricians, r/Construction, r/HomeInspection, r/plumbing
- Create YouTube shorts showing "Ask a code question, get an instant answer"

### Phase 2: Expand (Months 4-6)
- Expand to top 10 states by contractor population (CA, TX, FL, NY, IL, PA, OH, GA, NC, MI)
- Launch referral program (1 free month per referral)
- Attend trade shows (IBS, NECA, PHCC)
- SEO content machine (3-5 articles/week targeting code questions)

### Phase 3: Scale (Months 7-12)
- All 50 states
- Inspector edition with report generation
- API for integration with Procore, BuilderTrend, ServiceTitan
- Licensing deals with trade schools

---

## Risk Analysis

### High Risks
| Risk | Mitigation |
|------|------------|
| **ICC/NFPA licensing costs** unknown | Start with publicly available enacted codes; negotiate API access early |
| **Copyright litigation** from code publishers | Rely on Veeck precedent for enacted codes; license model codes properly |
| **LLM hallucination** giving wrong code answers | Always cite source sections; add confidence scoring; disclaimer for critical decisions |
| **UpCodes** is well-funded (YC) with 800K MAU | Different market — they target architects, we target field contractors |
| **Code accuracy liability** | Clear TOS: informational only, not legal advice; insurance |

### Medium Risks
| Risk | Mitigation |
|------|------------|
| Slow ICC API onboarding | Build with public code access first, add API later |
| Local amendment coverage is vast | Start with state-level codes, add local on demand |
| User acquisition cost | Grassroots — Reddit, trade forums, YouTube shorts |
| Flutter performance on older devices | Test on low-end Android; optimize for speed |

### Low Risks
| Risk | Mitigation |
|------|------------|
| Market demand exists | UpCodes has 800K MAU; CodePro has positive reviews |
| Technical feasibility proven | RAG + LLM is a solved problem; APIs exist |
| Mobile adoption in trades | 97% smartphone penetration; 76% of handymen use apps |

---

## Why This Is a Great Opportunity

1. **Huge, underserved market.** 6M+ potential users in the US. The solo contractor/handyman segment is virtually untouched by existing code compliance tools.

2. **Clear pain point.** Failed inspections cost $500-$5,000+ per incident. A $10/month app that prevents one failed inspection pays for itself 50-500x over.

3. **Timing is right.** AI/RAG makes accurate code Q&A possible for the first time. The tech didn't exist 3 years ago. Smartphones are universal in the trades.

4. **Defensible moat.** Code data relationships (ICC, NFPA, IAPMO) create barriers to entry. Local amendment database grows in value over time. User-generated bookmarks/reports create stickiness.

5. **Multiple revenue paths.** Subscriptions, partnerships, training marketplace, enterprise licensing.

6. **Low initial investment.** Flutter app + Python backend + existing APIs = can build MVP in 2-3 months.

---

## MVP Feature List (v1.0)

### Must Have
- [ ] AI Q&A with code citations (natural language → code reference)
- [ ] Code section browser (searchable, organized by code type)
- [ ] State/jurisdiction selector
- [ ] Offline access to saved codes
- [ ] iOS + Android (Flutter)
- [ ] User accounts + subscription billing

### Should Have
- [ ] Voice input for hands-free use
- [ ] Photo input (snap a picture of an installation, ask "is this to code?")
- [ ] Code change notifications
- [ ] Bookmark / save code sections
- [ ] Dark mode (for dimly lit job sites)

### Nice to Have
- [ ] Inspector-ready PDF reports
- [ ] Team sharing
- [ ] Code comparison across jurisdictions
- [ ] Integration with project management tools
- [ ] Continuing education content

---

## Next Steps

1. **Contact ICC** about Code Connect API pricing for startup
2. **Validate locally** — talk to 10 Maine contractors about pain points
3. **Build MVP** — Flutter app + FastAPI backend + GPT-4o-mini RAG
4. **Beta test** with 5-10 local contractors
5. **Launch** in Maine, iterate, expand

---

## File Structure (Reference)

```
CodePocket/
├── README.md              ← This file
├── COMPETITORS.md         ← Detailed competitor profiles
├── TECHNICAL.md           ← Architecture deep-dive
├── SEO-ASO.md             ← Keyword research + content strategy
├── FINANCIALS.md          ← Revenue models + projections
└── research/
    ├── market-data.md     ← Raw market research notes
    └── legal-notes.md     ← Copyright/legal research notes
```
