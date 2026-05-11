# Competitor Deep Dives — CodePocket

---

## 1. UpCodes

| Field | Details |
|-------|---------|
| **Website** | up.codes |
| **Founded** | 2016 |
| **Backed by** | Y Combinator (W18) |
| **Users** | 800,000 monthly active users |
| **Platform** | Web-first, iOS app |
| **Pricing** | Plus $19/mo ($190/yr), Premium $39/mo ($390/yr), Enterprise custom |
| **Target** | Architects, engineers, code officials, AEC firms |

### Strengths
- Massive code database: millions of sections, 1,700+ state/city codes
- UpCodes Copilot (AI assistant) with inline citations
- Code comparison across jurisdictions
- Revit plugin for real-time code checking
- Project Pages for team collaboration
- 160,000+ local amendments tracked
- 7,000 sections updated monthly

### Weaknesses
- **Desktop-first** — mobile experience is secondary
- **Expensive for solo contractors** — $19/mo minimum is steep for a handyman
- **Complex interface** — designed for architects, not field workers
- **No voice/photo input** — limited field usability
- **No Android app** — mobile is iOS only

### Threat Level: HIGH (but different market segment)
UpCodes dominates the architect/engineer segment. They're not focused on the solo contractor, handyman, or field worker. **Our opportunity is the market they're ignoring.**

---

## 2. CodePro

| Field | Details |
|-------|---------|
| **Website** | codeproapp.com |
| **Platform** | iOS only (no Android) |
| **Pricing** | Lite $6.99, Standard $9.99/mo (15 Q's), Pro $19.99/mo (200 Q's), Team $79.99/mo |
| **Target** | Contractors, handymen, inspectors, DIYers |

### Strengths
- AI-powered Q&A with state-specific answers
- Voice and photo input
- Simple, clean design for field use
- Direct code citations (IRC, NEC, UPC, IPC)
- Good reviews ("game changer", "incredible")
- Free trial available

### Weaknesses
- **iOS only** — completely ignores Android users (~60% of US market)
- **Token/question-based pricing** — feels punitive for heavy users
- **Limited local amendments** — state-level only, not county/city
- **Small team** — appears to be indie/solo developer
- **No offline access**
- **No code browsing** — Q&A only, can't read the code directly

### Threat Level: HIGH (closest direct competitor)
CodePro is the closest thing to CodePocket that exists. But their iOS-only limitation, per-question pricing, and lack of local amendments leave massive room for a better solution.

---

## 3. Melt Code (by MeltPlan)

| Field | Details |
|-------|---------|
| **Website** | meltplan.com/code |
| **Platform** | Web (cloud-based) |
| **Pricing** | Not publicly listed (enterprise) |
| **Target** | Architecture and engineering design teams |

### Strengths
- AI with step-by-step reasoning and full compliance paths
- Direct code citations (IBC, IRC, IEBC, IFC, IMC, IPC, IECC, NEC, NFPA, ADA)
- 95%+ accuracy (validated against inspector exams)
- State-specific amendments for all 50 states
- Construction-aware AI that understands building context

### Weaknesses
- **No mobile app** — web platform only
- **Enterprise pricing** — not accessible for individual contractors
- **Complex output** — designed for design teams, too heavy for field use
- **No voice/photo input**

### Threat Level: MEDIUM
Melt Code proves AI can accurately answer code questions. They're targeting a different market (design teams doing plan reviews). Could potentially pivot to field use.

---

## 4. ICC Digital Codes Premium

| Field | Details |
|-------|---------|
| **Website** | iccsafe.org |
| **Platform** | iOS app |
| **Pricing** | Subscription (varies by code package) |
| **Target** | Code officials, inspectors |

### Strengths
- Official ICC content — the source of truth
- Advanced search, notes, highlights
- Offline access to purchased codes
- Trusted by code enforcement professionals

### Weaknesses
- **Just a digital PDF reader** — no AI, no Q&A
- **Expensive** — pay per code book ($100-200+ each)
- **iOS only** for mobile
- **No cross-referencing** between codes
- **Not designed for contractors**

### Threat Level: LOW
ICC is a data provider, not a competitor. They could potentially become a partner (via their Code Connect API).

---

## 5. OneClick Code

| Field | Details |
|-------|---------|
| **Website** | oneclickcode.com |
| **Platform** | iOS + Android |
| **Pricing** | Free / freemium |
| **Target** | Roofers |

### Strengths
- Instant jurisdiction-specific roofing codes
- Report generation
- Free to use
- Available on both platforms

### Weaknesses
- **Roofing codes only** — very niche
- **No AI** — just a database lookup
- **Limited scope**

### Threat Level: LOW
Single-trade, no AI. Proves there's demand for code lookup apps.

---

## 6. NFPA LiNK

| Field | Details |
|-------|---------|
| **Website** | link.nfpa.org |
| **Platform** | Web platform |
| **Pricing** | Subscription ($150-300/yr per title) |
| **Target** | Fire safety professionals, electricians |

### Strengths
- Official NFPA content (NEC, NFPA 1, 101, etc.)
- AI assistant (CASI) for code questions
- Expert commentary and visual aids
- Notebook/collection features

### Weaknesses
- **NFPA codes only** — no building, plumbing, or mechanical codes
- **Expensive** — $150-300/yr per code title
- **No mobile app** — web only
- **Not contractor-friendly** — designed for fire marshals and engineers

### Threat Level: MEDIUM
NFPA's AI assistant proves LLM-powered code Q&A works. Their scope is limited to NFPA publications.

---

## 7. CodeComply.ai

| Field | Details |
|-------|---------|
| **Website** | codecomply.ai |
| **Platform** | Desktop software |
| **Pricing** | Enterprise |
| **Target** | Architects, plan reviewers |

### Strengths
- AI-powered automatic plan review against codes
- Checks IBC, IRC, NFPA, ADA, FHA + local amendments
- Suggests solutions for compliance issues
- Can analyze building plans directly

### Weaknesses
- **Not a field tool** — desktop plan review software
- **Enterprise only** — not for individual contractors
- **No mobile component**

### Threat Level: LOW
Completely different use case (plan review vs. field Q&A).

---

## Competitive Position Map

```
                     PRICE (Low → High)
                          │
     CodePocket ●         │
     (proposed)           │         ● UpCodes
                          │        ($19-39/mo)
                          │
     ● OneClick Code      │
     (free/cheap)         │         ● Melt Code
                          │        (enterprise)
                          │
     ● CodePro            │
     ($4-30/mo)           │         ● NFPA LiNK
                          │        ($150-300/yr)
                          │
                          │         ● CodeComply
                          │        (enterprise)
                          │
                          │         ● ICC Digital
                          │        ($100-200/code)
                          │
─────────────────────────────────────────────────
  Field/Contractor ◄─────┼──────► Office/Architect
         MOBILE-FIRST              DESKTOP-FIRST
```

**CodePocket's position: Bottom-left quadrant.** Affordable, mobile-first, field-ready. Nobody else is here.
