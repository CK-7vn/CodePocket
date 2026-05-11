# Financial Projections — CodePocket

## Revenue Model

### Subscription Tiers

| Tier | Monthly | Annual | Target User |
|------|---------|--------|-------------|
| **Free** | $0 | $0 | Casual users, trial |
| **Pro** | $9.99 | $89/yr (26% savings) | Solo contractors, handymen |
| **Team** | $29.99 | $269/yr | Small crews (up to 5 users) |
| **Inspector** | $49.99 | $449/yr | Building inspectors, code officials |

### Feature Matrix

| Feature | Free | Pro | Team | Inspector |
|---------|------|-----|------|-----------|
| AI Q&A (per month) | 5 | Unlimited | Unlimited | Unlimited |
| Code search | ✓ | ✓ | ✓ | ✓ |
| Jurisdictions | 1 | All | All | All |
| Offline access | ✗ | ✓ | ✓ | ✓ |
| Saved codes | 10 | Unlimited | Unlimited | Unlimited |
| Code change alerts | ✗ | ✓ | ✓ | ✓ |
| Voice/photo input | ✗ | ✓ | ✓ | ✓ |
| Team sharing | ✗ | ✗ | ✓ | ✓ |
| PDF reports | ✗ | ✗ | ✓ | ✓ |
| API access | ✗ | ✗ | ✗ | ✓ |
| Priority support | ✗ | ✗ | ✗ | ✓ |

---

## Cost Structure

### Development Costs (MVP — Months 1-3)

| Item | Cost | Notes |
|------|------|-------|
| Clyde's time | $0 (sweat equity) | Evenings/weekends |
| ICC API license | $500-2,000? | Need to contact — startup pricing TBD |
| LLM API costs | $100-300/mo | GPT-4o-mini at scale |
| Backend hosting | $20-50/mo | Railway or Fly.io |
| Database hosting | $20-40/mo | Postgres + Redis |
| Vector DB | $0-70/mo | Pinecone free tier → paid |
| Apple Developer | $99/yr | Required for App Store |
| Google Play | $25 (one-time) | Required for Play Store |
| Domain (codepocket.com) | $12/yr | If available |
| Firebase/Supabase | $0-25/mo | Auth + push notifications |
| **Total MVP (3 months)** | **~$2,000-4,000** | |

### Monthly Operating Costs (at Scale)

| Users | LLM API | Hosting | ICC License | Other | Total/mo |
|-------|---------|---------|-------------|-------|----------|
| 1K | $50 | $30 | $200 | $50 | $330 |
| 10K | $300 | $50 | $500 | $100 | $950 |
| 50K | $1,200 | $100 | $1,000 | $200 | $2,500 |
| 100K | $2,500 | $200 | $2,000 | $300 | $5,000 |

---

## Revenue Projections

### Scenario 1: Conservative (Solo launch, minimal marketing)

| Metric | Month 3 | Month 6 | Month 12 | Year 2 | Year 3 |
|--------|---------|---------|----------|--------|--------|
| Total users | 1,000 | 5,000 | 25,000 | 200,000 | 500,000 |
| Paying % | 5% | 7% | 10% | 12% | 15% |
| Paying users | 50 | 350 | 2,500 | 24,000 | 75,000 |
| Avg. revenue/user | $8 | $8 | $9 | $9 | $10 |
| **Monthly revenue** | **$400** | **$2,800** | **$22,500** | **$216,000** | **$750,000** |
| **Annual revenue** | — | — | **~$135K** | **$2.6M** | **$9M** |
| Monthly costs | $350 | $600 | $1,500 | $3,500 | $7,000 |
| **Monthly profit** | **$50** | **$2,200** | **$21,000** | **$212,500** | **$743,000** |

### Scenario 2: Moderate (With some marketing, word-of-mouth growth)

| Metric | Month 3 | Month 6 | Month 12 | Year 2 | Year 3 |
|--------|---------|---------|----------|--------|--------|
| Total users | 3,000 | 15,000 | 75,000 | 400,000 | 1M |
| Paying % | 8% | 10% | 12% | 15% | 18% |
| Paying users | 240 | 1,500 | 9,000 | 60,000 | 180,000 |
| Avg. revenue/user | $8 | $9 | $10 | $10 | $10 |
| **Monthly revenue** | **$1,920** | **$13,500** | **$90,000** | **$600,000** | **$1.8M** |
| **Annual revenue** | — | — | **~$540K** | **$7.2M** | **$21.6M** |

### Key Assumptions
- 97% smartphone penetration in trades → no device barrier
- Free tier drives adoption → 5-15% conversion to paid
- CodePro at $10-30/mo proves willingness to pay
- UpCodes at $19-39/mo proves higher price tolerance
- Word-of-mouth is strong in tight-knit trade communities

---

## Break-Even Analysis

| Scenario | Break-Even Point |
|----------|-----------------|
| MVP costs only (~$2K) | ~200 paying users |
| Monthly costs covered | ~100 paying users (at $10/mo avg) |
| Full-time income for Clyde ($60K/yr) | ~500 paying users |

**Realistic break-even: Month 6-8** with conservative growth

---

## Growth Levers (Revenue Expansion)

1. **Geographic expansion** — US first, then Canada, UK, Australia (same code frameworks)
2. **Trade-specific tiers** — Electrician Pro, Plumber Pro (curated code bundles)
3. **Continuing education** — CEU courses tied to code changes ($49-99/course)
4. **Insurance partnerships** — Compliance tracking reduces claims → carriers subsidize
5. **Tool manufacturer partnerships** — Leviton, Southwire, etc. (like Captain Code app)
6. **White-label / API licensing** — Sell code Q&A to Procore, BuilderTrend, ServiceTitan
7. **Municipal contracts** — Provide code lookup tools for building departments

---

## Comparable Exits / Valuations

| Company | Metric | Valuation |
|---------|--------|-----------|
| UpCodes | 800K MAU, ~$15M ARR est. | YC-backed, raised $3.5M+ |
| ServiceTitan | Contractor SaaS | $7.4B valuation (2024) |
| Procore | Construction SaaS | $9B public market cap |
| Housecall Pro | Home service SaaS | $300M+ ARR |

**Realistic outcome:** Build to 100K+ paying users → $10-15M ARR → acquisition by construction SaaS platform (Procore, BuilderTrend, ServiceTitan) for $50-150M within 3-5 years.

**Conservative outcome:** Bootstrapped to $2-5M ARR → sustainable lifestyle business for Clyde & Chelsea.
