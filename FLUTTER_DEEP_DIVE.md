# Flutter Deep Dive — CodePocket

> Complete technical specification for the CodePocket mobile app.
> Covers architecture, payments, RAG data pipeline, UI design system, and implementation plan.

---

## Table of Contents

1. [Flutter Project Architecture](#1-flutter-project-architecture)
2. [State Management](#2-state-management)
3. [Payment System (RevenueCat)](#3-payment-system-revenuecat)
4. [RAG Data Pipeline](#4-rag-data-pipeline--code-book-collection)
5. [Offline Strategy](#5-offline-strategy)
6. [Voice & Photo Input](#6-voice--photo-input)
7. [UI Design System](#7-ui-design-system)
8. [Screen-by-Screen Specs](#8-screen-by-screen-specs)
9. [Accessibility](#9-accessibility)
10. [Testing Strategy](#10-testing-strategy)
11. [Package Dependencies](#11-package-dependencies)
12. [Implementation Roadmap](#12-implementation-roadmap)

---

## 1. Flutter Project Architecture

### Pattern: Feature-First Clean Architecture

```
lib/
├── main.dart                          # App entry point
├── app.dart                           # MaterialApp + router setup
│
├── core/                              # Shared infrastructure
│   ├── theme/
│   │   ├── app_theme.dart             # Light + dark ThemeData
│   │   ├── app_colors.dart            # Color constants
│   │   ├── app_typography.dart        # Text styles
│   │   └── app_spacing.dart           # Spacing / sizing constants
│   ├── constants/
│   │   ├── api_constants.dart
│   │   └── subscription_constants.dart
│   ├── network/
│   │   ├── api_client.dart            # Dio HTTP client wrapper
│   │   ├── api_interceptor.dart       # Auth token injection
│   │   └── api_result.dart            # Success/Error sealed class
│   ├── storage/
│   │   ├── local_storage.dart         # SharedPreferences wrapper
│   │   └── secure_storage.dart        # flutter_secure_storage wrapper
│   ├── utils/
│   │   ├── validators.dart
│   │   ├── formatters.dart
│   │   └── extensions/
│   │       ├── context_extensions.dart
│   │       └── string_extensions.dart
│   └── widgets/                       # Shared UI components
│       ├── codepocket_logo.dart
│       ├── citation_chip.dart
│       ├── code_section_card.dart
│       ├── chat_bubble.dart
│       ├── loading_indicator.dart
│       ├── error_view.dart
│       ├── empty_state.dart
│       └── paywall_overlay.dart
│
├── features/                          # Feature modules (each self-contained)
│   ├── auth/
│   │   ├── data/
│   │   │   ├── auth_repository.dart
│   │   │   └── auth_remote_data_source.dart
│   │   ├── domain/
│   │   │   ├── user_model.dart
│   │   │   └── auth_use_cases.dart
│   │   └── presentation/
│   │       ├── auth_provider.dart     # Riverpod provider
│   │       ├── login_screen.dart
│   │       └── onboarding/
│   │           ├── trade_selector_screen.dart
│   │           └── state_selector_screen.dart
│   │
│   ├── chat/                          # AI Q&A — THE CORE FEATURE
│   │   ├── data/
│   │   │   ├── chat_repository.dart
│   │   │   ├── chat_remote_data_source.dart
│   │   │   └── chat_local_data_source.dart  # Offline queue
│   │   ├── domain/
│   │   │   ├── message_model.dart
│   │   │   ├── citation_model.dart
│   │   │   └── chat_use_cases.dart
│   │   └── presentation/
│   │       ├── chat_provider.dart
│   │       ├── chat_screen.dart
│   │       ├── widgets/
│   │       │   ├── message_list.dart
│   │       │   ├── input_bar.dart
│   │       │   ├── voice_button.dart
│   │       │   ├── citation_card.dart
│   │       │   └── streaming_text.dart  # Token-by-token display
│   │       └── chat_history_screen.dart
│   │
│   ├── codes/                         # Code Browser
│   │   ├── data/
│   │   │   ├── code_repository.dart
│   │   │   ├── code_local_data_source.dart  # Drift/SQLite
│   │   │   └── code_remote_data_source.dart
│   │   ├── domain/
│   │   │   ├── code_section_model.dart
│   │   │   ├── code_book_model.dart
│   │   │   ├── jurisdiction_model.dart
│   │   │   └── code_use_cases.dart
│   │   └── presentation/
│   │       ├── codes_provider.dart
│   │       ├── code_browser_screen.dart
│   │       ├── code_section_screen.dart
│   │       └── widgets/
│   │           ├── code_book_tile.dart
│   │           ├── section_tree.dart
│   │           └── highlight_text.dart
│   │
│   ├── search/                        # Universal Search
│   │   ├── data/
│   │   │   └── search_repository.dart
│   │   ├── domain/
│   │   │   └── search_result_model.dart
│   │   └── presentation/
│   │       ├── search_provider.dart
│   │       ├── search_screen.dart
│   │       └── widgets/
│   │           └── search_result_tile.dart
│   │
│   ├── saved/                         # Bookmarks / Saved Codes
│   │   ├── data/
│   │   │   └── saved_repository.dart  # Local-only (Drift)
│   │   ├── domain/
│   │   │   └── saved_code_model.dart
│   │   └── presentation/
│   │       ├── saved_provider.dart
│   │       └── saved_screen.dart
│   │
│   ├── subscription/                  # Paywall & Subscription Management
│   │   ├── data/
│   │   │   ├── revenue_cat_service.dart
│   │   │   └── subscription_repository.dart
│   │   ├── domain/
│   │   │   ├── entitlement_model.dart
│   │   │   └── subscription_use_cases.dart
│   │   └── presentation/
│   │       ├── subscription_provider.dart
│   │       ├── paywall_screen.dart
│   │       └── manage_subscription_screen.dart
│   │
│   ├── settings/
│   │   └── presentation/
│   │       ├── settings_screen.dart
│   │       ├── jurisdiction_settings.dart
│   │       └── trade_settings.dart
│   │
│   └── home/
│       └── presentation/
│           ├── home_screen.dart       # Main tab scaffold
│           └── widgets/
│               └── quick_actions.dart
│
└── routing/
    ├── app_router.dart                # GoRouter configuration
    └── route_guards.dart              # Auth + subscription guards
```

### Navigation: GoRouter

```dart
// routing/app_router.dart
final goRouter = GoRouter(
  initialLocation: '/home',
  redirect: (context, state) async {
    final auth = await authRepository.isAuthenticated;
    final onboardingComplete = await localStorage.isOnboardingComplete;

    if (!auth) return '/login';
    if (!onboardingComplete) return '/onboarding/trade';
    return null; // continue to requested route
  },
  routes: [
    ShellRoute(
      builder: (context, state, child) => MainScaffold(child: child),
      routes: [
        GoRoute(path: '/home', builder: (_, __) => HomeScreen()),
        GoRoute(path: '/chat', builder: (_, __) => ChatScreen()),
        GoRoute(path: '/codes', builder: (_, __) => CodeBrowserScreen()),
        GoRoute(path: '/search', builder: (_, __) => SearchScreen()),
        GoRoute(path: '/saved', builder: (_, __) => SavedScreen()),
      ],
    ),
    GoRoute(path: '/login', builder: (_, __) => LoginScreen()),
    GoRoute(path: '/onboarding/trade', builder: (_, __) => TradeSelectorScreen()),
    GoRoute(path: '/onboarding/state', builder: (_, __) => StateSelectorScreen()),
    GoRoute(path: '/paywall', builder: (_, __) => PaywallScreen()),
    GoRoute(path: '/settings', builder: (_, __) => SettingsScreen()),
    GoRoute(path: '/codes/:sectionId', builder: (_, state) =>
      CodeSectionScreen(sectionId: state.pathParameters['sectionId']!)),
  ],
);
```

---

## 2. State Management

### Choice: Riverpod 2.0 (flutter_riverpod)

**Why Riverpod over BLoC?**
- Less boilerplate (no events/states/separate bloc classes)
- Compile-time safety (providers are typed)
- Better for features that mix local + remote state (like ours)
- AsyncValue widget handles loading/error/data automatically
- riverpod_generator for code-generated providers

### Provider Pattern

```dart
// features/chat/chat_provider.dart
@riverpod
class ChatNotifier extends _$ChatNotifier {
  @override
  AsyncValue<List<MessageModel>> build() => const AsyncValue.data([]);

  Future<void> sendMessage(String text, {File? photo}) async {
    // 1. Add user message immediately (optimistic)
    state = AsyncValue.data([
      ...state.value ?? [],
      MessageModel.user(text: text, timestamp: DateTime.now()),
    ]);

    // 2. Add placeholder for AI response
    final aiMessage = MessageModel.ai(streaming: true);
    state = AsyncValue.data([...state.value ?? [], aiMessage]);

    // 3. Stream response from API
    try {
      await for (final chunk in chatRepository.streamResponse(
        question: text,
        jurisdiction: ref.read(jurisdictionProvider),
        photo: photo,
      )) {
        // Update AI message with each chunk
        final messages = [...state.value ?? []];
        messages.last = messages.last.appendChunk(chunk);
        state = AsyncValue.data(messages);
      }
    } catch (e) {
      // Replace last message with error
      final messages = [...state.value ?? []];
      messages.last = messages.last.copyWith(
        streaming: false,
        error: 'Failed to get response. Tap to retry.',
      );
      state = AsyncValue.data(messages);
    }
  }
}
```

---

## 3. Payment System (RevenueCat)

### Why RevenueCat

| Option | Pros | Cons | Verdict |
|--------|------|------|---------|
| **RevenueCat** | Handles iOS + Android subs, receipt validation, analytics, paywall UI, web billing | Free up to $2,500/mo revenue, then % cut | ✅ Use this |
| **StoreKit + Play Billing** direct | No fees | Two codebases, receipt validation is hell, no analytics | ❌ Not worth it |
| **Adapty / Qonversion** | Similar to RevenueCat | Smaller ecosystem, less Flutter support | ❌ RevenueCat is better documented |

**RevenueCat pricing:** Free until $2,500 monthly revenue. After that, they take 1%. For our Phase 1, this is essentially free.

### Setup Steps

1. **Create RevenueCat account** → app.revenuecat.com
2. **Create project** → Add iOS App + Android App
3. **Configure products in App Store Connect** (iOS) + **Google Play Console** (Android):
   - `codepocket_pro_monthly` → $9.99/mo
   - `codepocket_pro_annual` → $89/yr
   - `codepocket_team_monthly` → $29.99/mo
4. **Create Entitlements in RevenueCat:**
   - `pro` → unlocks unlimited AI Q&A, all jurisdictions, offline
   - `team` → unlocks team features + pro features
5. **RevenueCat API key** → added to app

### Flutter Integration

```dart
// features/subscription/data/revenue_cat_service.dart
class RevenueCatService {
  static const _apiKey = 'your_revenuecat_api_key';

  Future<void> initialize(String userId) async {
    await Purchases.configure(PurchasesConfiguration(_apiKey)
      ..appUserID = userId);
  }

  Future<Offering?> getCurrentOffering() async {
    final offerings = await Purchases.getOfferings();
    return offerings.current;
  }

  Future<bool> purchasePackage(Package package) async {
    try {
      final customerInfo = await Purchases.purchasePackage(package);
      return customerInfo.entitlements.all['pro']?.isActive ?? false;
    } on PurchasesException catch (e) {
      if (e.code == PurchasesErrorCode.purchaseCancelledError) {
        return false; // User cancelled
      }
      rethrow;
    }
  }

  Future<bool> isProSubscriber() async {
    final customerInfo = await Purchases.getCustomerInfo();
    return customerInfo.entitlements.all['pro']?.isActive ?? false;
  }

  Future<void> restorePurchases() async {
    await Purchases.restorePurchases();
  }
}
```

### Paywall Screen Design

```
┌──────────────────────────────────┐
│         ✕ Close                   │
│                                   │
│     ⚡ CodePocket Pro             │
│                                   │
│   Never guess on a code          │
│   question again.                 │
│                                   │
│  ┌────────────────────────────┐  │
│  │  ✓ Unlimited AI questions  │  │
│  │  ✓ All 50 states           │  │
│  │  ✓ All code types          │  │
│  │  ✓ Offline access          │  │
│  │  ✓ Code change alerts      │  │
│  │  ✓ Voice & photo input     │  │
│  └────────────────────────────┘  │
│                                   │
│  ┌────────────────────────────┐  │
│  │   $9.99 / month            │  │  ← Most Popular
│  └────────────────────────────┘  │
│  ┌────────────────────────────┐  │
│  │   $89 / year               │  │  ← Save 26%
│  │   ($7.42/mo)               │  │
│  └────────────────────────────┘  │
│                                   │
│  ┌────────────────────────────┐  │
│  │     Start 7-Day Free Trial │  │  ← CTA button
│  └────────────────────────────┘  │
│                                   │
│     Cancel anytime                │
│     Restore purchases             │
│     Terms · Privacy               │
└──────────────────────────────────┘
```

### Free Tier Enforcement

```dart
// Guard that checks before allowing AI questions
class QuestionGuard {
  final RevenueCatService _revenueCat;
  final LocalStorage _localStorage;

  Future<QuestionResult> canAskQuestion() async {
    // Pro users: unlimited
    if (await _revenueCat.isProSubscriber()) {
      return QuestionResult.allowed();
    }

    // Free users: check monthly count
    final usage = await _localStorage.getMonthlyUsage();
    if (usage.questionsUsed < 5) {
      return QuestionResult.allowed(
        questionsRemaining: 5 - usage.questionsUsed - 1,
      );
    }

    return QuestionResult.paywallRequired();
  }
}
```

---

## 4. RAG Data Pipeline & Code Book Collection

### The Big Question: Do We Collect Code Books?

**Yes, but strategically.** Here's the plan:

### Phase 1: Free Public Sources (Week 1-4)

Under **Veeck v. Southern Building Code Congress** (5th Circuit, 2002), model codes **become public domain when enacted into law** by a jurisdiction. This means:

| Source | Access | Cost | What you get |
|--------|--------|------|--------------|
| **ICC free read-only** (codes.iccsafe.org) | Web access, no API | Free | Current I-Codes (IBC, IRC, IPC, IMC, IECC, IFGC) |
| **NFPA free view** (nfpa.org/free-access) | Web view, no save/print | Free | NEC 2020, NFPA 101, NFPA 1 (read-only) |
| **State websites** | Varies | Free | State amendments, adopted code versions |
| **Municode / eCode360** | Web scraping | Free | Local municipal code amendments |
| **UpCodes** (partial) | Public sections | Free | Some code sections are freely accessible |

**Phase 1 ingestion strategy:**
1. Write Python scrapers for ICC's free read-only portal
2. Download publicly available PDF codebooks from state websites
3. Parse into structured JSON (section number, chapter, text, tables)
4. Clean and normalize the text (remove headers/footers, fix OCR artifacts)
5. Chunk into ~500-token overlapping segments for embedding

### Phase 2: ICC Code Connect API (Month 2-3)

Once we have some revenue or funding:
- Contact ICC at panthony@iccsafe.org for Code Connect API pricing
- Get official JSON access to all I-Codes with amendments
- Sign implementation agreement + content license

### Phase 3: NFPA Licensing (Month 4+)

- License NEC and fire codes through NFPA
- Or: use publicly available sections + link to NFPA LiNK for full access

### RAG Architecture

```
                     CODE INGESTION PIPELINE
                     ========================

  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
  │  PDF /   │   │  Parser  │   │  Chunker │   │ Embedder │
  │  HTML /  │──▶│  (extract│──▶│  (split  │──▶│  (OpenAI │
  │  API     │   │  text,   │   │  into    │   │  text-   │
  │  JSON    │   │  tables) │   │  ~500    │   │  embed-  │
  └──────────┘   └──────────┘   │  token   │   │  ding-3- │
                                │  chunks) │   │  small)  │
                                └──────────┘   └─────┬────┘
                                                     │
                                                     ▼
                                              ┌──────────────┐
                                              │  Vector DB   │
                                              │  (Pinecone   │
                                              │   or         │
                                              │  Supabase    │
                                              │   pgvector)  │
                                              └──────┬───────┘
                                                     │
                     QUERY PIPELINE                   │
                     ===============                  │
                                                     │
  User Question ──────┐                              │
                      ▼                              │
              ┌──────────────┐                       │
              │  Query       │                       │
              │  Enhancement │                       │
              │  (detect     │                       │
              │   trade,     │                       │
              │   state,     │                       │
              │   intent)    │                       │
              └──────┬───────┘                       │
                     │                               │
                     ▼                               │
              ┌──────────────┐    ┌──────────┐      │
              │  Hybrid      │───▶│  Merge   │◀─────┘
              │  Search      │    │  & Re-   │    (semantic
              │  (keyword +  │    │  rank     │     results)
              │   semantic)  │    │  (top 10) │
              └──────────────┘    └────┬─────┘
                                     │
                                     ▼
                              ┌──────────────┐
                              │  Context     │
                              │  Assembly    │
                              │  (relevant   │
                              │   sections + │
                              │   jurisdiction│
                              │   metadata)  │
                              └──────┬───────┘
                                     │
                                     ▼
                              ┌──────────────┐
                              │  LLM         │
                              │  (GPT-4o-    │
                              │   mini or    │
                              │   Claude     │
                              │   Haiku)     │
                              │              │
                              │  System:     │
                              │  "Answer     │
                              │   with code  │
                              │   citations. │
                              │   Never      │
                              │   invent     │
                              │   section    │
                              │   numbers."  │
                              └──────┬───────┘
                                     │
                                     ▼
                              ┌──────────────┐
                              │  Validation  │
                              │  - Check     │
                              │    citations │
                              │    exist     │
                              │  - Confidence│
                              │    score     │
                              │  - Disclaimer│
                              └──────┬───────┘
                                     │
                                     ▼
                              Response to User
                              (streaming, with
                               tappable citations)
```

### Code Data Structure

```json
{
  "code_book": {
    "id": "irc_2021",
    "family": "IRC",
    "full_name": "International Residential Code",
    "edition": 2021,
    "publisher": "ICC"
  },
  "section": {
    "number": "R317.1",
    "chapter": "3",
    "chapter_title": "Building Planning",
    "heading": "Protection of Wood and Wood Based Products Against Decay",
    "text": "Wood and wood-based products used in contact with, or embedded in, the ground...",
    "tables": [
      {
        "number": "R317.1(1)",
        "caption": "Nominal Lumber Section Size",
        "rows": [...]
      }
    ]
  },
  "jurisdiction": {
    "state": "ME",
    "adopted_edition": 2021,
    "effective_date": "2022-07-01",
    "amendments": [
      {
        "section": "R317.1",
        "text": "Maine adds: Pressure-treated lumber shall bear the grade mark or certificate...",
        "type": "addition"
      }
    ]
  },
  "chunk": {
    "id": "irc_2021_R317.1_chunk_0",
    "text": "R317.1 Protection of Wood and Wood Based Products Against Decay. Wood and wood-based products...",
    "embedding": [0.0023, -0.0087, ...],  // 1536 dimensions
    "token_count": 478,
    "source_section": "R317.1"
  }
}
```

### Embedding & Chunking Strategy

```python
# Chunking rules for building codes:
# 1. Never split mid-section (keep section number + heading + text together)
# 2. If section > 800 tokens, split at paragraph boundaries
# 3. Tables get their own chunks (with section reference)
# 4. Include metadata: jurisdiction, code family, edition, section number
# 5. Overlap: 50 tokens between chunks for context continuity

CHUNK_CONFIG = {
    "max_tokens": 500,
    "overlap_tokens": 50,
    "min_chunk_tokens": 50,
    "split_on": ["paragraph", "list_item", "table_row"],
    "never_split_patterns": [r"^\d+\.\d+[\.\d]*\s"],  # Section numbers
}

EMBEDDING_CONFIG = {
    "model": "text-embedding-3-small",  # OpenAI, 1536 dims, cheap
    "dimensions": 1536,
    "cost_per_1k_tokens": 0.00002,     # ~$0.02 to embed entire IRC
}
```

### Ingestion Script (Python)

```python
# backend/ingestion/ingest_code_book.py

async def ingest_code_book(source: CodeSource) -> IngestionReport:
    """Full pipeline: download → parse → chunk → embed → store."""

    # 1. Download / extract text
    raw_text = await downloader.fetch(source)

    # 2. Parse into structured sections
    sections = parser.parse(raw_text, format=source.format)
    # Each section: { number, chapter, heading, text, tables }

    # 3. Apply jurisdiction amendments
    if source.amendments:
        sections = amendment_merger.merge(sections, source.amendments)

    # 4. Chunk for embedding
    chunks = chunker.chunk(sections, config=CHUNK_CONFIG)

    # 5. Generate embeddings (batch)
    embeddings = await embedder.embed_batch(
        [c.text for c in chunks],
        model="text-embedding-3-small"
    )

    # 6. Store in vector DB
    await vector_store.upsert(
        vectors=[
            VectorEntry(
                id=f"{source.id}_{chunk.section}_{chunk.index}",
                values=embedding,
                metadata={
                    "code_family": source.family,
                    "edition": source.edition,
                    "section_number": chunk.section,
                    "chapter": chunk.chapter,
                    "jurisdiction": source.jurisdiction,
                    "text_preview": chunk.text[:200],
                    "has_table": chunk.has_table,
                },
                text=chunk.text,
            )
            for chunk, embedding in zip(chunks, embeddings)
        ]
    )

    # 7. Also store full sections in Postgres for exact lookup
    await pg_store.upsert_sections(sections)

    return IngestionReport(
        sections_parsed=len(sections),
        chunks_created=len(chunks),
        embeddings_generated=len(embeddings),
        cost=len(chunks) * 500 * 0.00002 / 1000,  # ~pennies
    )
```

### Estimated Data Volumes

| Code Book | Approx. Sections | Approx. Chunks | Embedding Cost |
|-----------|-----------------|----------------|----------------|
| IRC 2021 | 2,500 | 3,000 | $0.03 |
| IBC 2021 | 3,200 | 4,000 | $0.04 |
| NEC 2020 | 4,000 | 5,000 | $0.05 |
| IPC 2021 | 1,800 | 2,200 | $0.02 |
| IMC 2021 | 1,600 | 2,000 | $0.02 |
| IECC 2021 | 1,200 | 1,500 | $0.02 |
| **All 50 states w/ amendments** | ~70,000 | ~90,000 | ~$0.90 |
| **Total Phase 1** | **~15,000** | **~18,000** | **~$0.18** |

Embedding the entire US code library costs less than a dollar. The LLM query cost is the real expense (~$0.001-0.005 per question).

---

## 5. Offline Strategy

### Local Database: Drift (SQLite)

```dart
// core/storage/local_database.dart
@DriftDatabase(tables: [
  CodeSections,
  CodeBooks,
  Jurisdictions,
  SavedCodes,
  ChatHistory,
  CachedSearchResults,
])
class AppDatabase extends _$AppDatabase {
  AppDatabase() : super(_openConnection());

  @override
  int get schemaVersion => 1;
}

// Tables
class CodeSections extends Table {
  TextColumn get id => text()();                    // 'irc_2021_R317.1'
  TextColumn get codeFamily => text()();            // 'IRC'
  IntColumn get edition => integer()();             // 2021
  TextColumn get sectionNumber => text()();         // 'R317.1'
  TextColumn get chapter => text()();               // '3'
  TextColumn get heading => text()();               // 'Protection of Wood...'
  TextColumn get fullText => text()();              // Full section text
  TextColumn get jurisdiction => text()();          // 'ME'
  DateTimeColumn get lastUpdated => dateTime()();
  BoolColumn get isCached => boolean().withDefault(const Constant(false))();

  @override
  Set<Column> get primaryKey => {id};
}

class SavedCodes extends Table {
  TextColumn get id => text()();
  TextColumn get sectionId => text().references(CodeSections, #id)();
  TextColumn get userNotes => text().withDefault(const Constant(''))();
  TextColumn get folder => text().withDefault(const Constant('default'))();
  DateTimeColumn get savedAt => dateTime()();
}
```

### Offline Behavior Matrix

| Feature | Online | Offline | Sync on reconnect |
|---------|--------|---------|-------------------|
| AI Q&A | ✅ Full | ❌ Show "requires internet" | — |
| Code search (semantic) | ✅ Full | ❌ | — |
| Code search (keyword) | ✅ Full | ✅ Cached sections only | Sync new sections |
| Browse saved codes | ✅ | ✅ Full | — |
| Browse code books | ✅ | ✅ Cached books only | Sync updates |
| Save/bookmark | ✅ | ✅ Queue locally | Push to server |
| Settings | ✅ | ✅ | Sync preferences |
| Subscription check | ✅ | ✅ Cached entitlement | Refresh from RC |

### Cache Strategy

```dart
// Auto-cache when user views a code section
class CodeCacheService {
  // Cache sections the user has viewed (most recent 500)
  static const maxCachedSections = 500;

  // Cache the user's primary code books entirely
  Future<void> cachePrimaryCodeBooks() async {
    final jurisdiction = await getUserJurisdiction(); // e.g., 'ME'
    final trade = await getUserTrade(); // e.g., 'electrician'

    final codeFamilies = _getCodeFamiliesForTrade(trade);
    // Electrician: ['NEC', 'IRC']
    // Plumber: ['IPC', 'UPC', 'IRC']
    // GC: ['IBC', 'IRC', 'IECC']

    for (final family in codeFamilies) {
      final sections = await remoteDataSource.fetchSections(
        family: family,
        jurisdiction: jurisdiction,
      );
      await localDataSource.upsertSections(sections);
    }
  }
}
```

---

## 6. Voice & Photo Input

### Voice: speech_to_text package

```dart
// features/chat/presentation/widgets/voice_button.dart
class VoiceInputButton extends ConsumerStatefulWidget {
  @override
  ConsumerState<VoiceInputButton> createState() => _VoiceInputButtonState();
}

class _VoiceInputButtonState extends ConsumerState<VoiceInputButton> {
  final _speech = SpeechToText();
  bool _isListening = false;
  String _recognizedText = '';

  Future<void> _startListening() async {
    final available = await _speech.initialize(
      onStatus: (status) {
        if (status == 'done') {
          // Submit the recognized text as a question
          if (_recognizedText.isNotEmpty) {
            ref.read(chatProvider.notifier).sendMessage(_recognizedText);
          }
          setState(() => _isListening = false);
        }
      },
    );

    if (available) {
      setState(() => _isListening = true);
      _speech.listen(
        onResult: (result) {
          setState(() => _recognizedText = result.recognizedWords);
        },
        listenOptions: SpeechListenOptions(
          listenMode: ListenMode.search, // Better for short queries
          partialResults: true, // Show as they speak
        ),
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onLongPress: _startListening, // Hold to speak
      onLongPressEnd: (_) => _speech.stop(),
      child: AnimatedContainer(
        duration: const Duration(milliseconds: 200),
        padding: EdgeInsets.all(_isListening ? 20 : 14),
        decoration: BoxDecoration(
          color: _isListening
            ? AppColors.primary.withValues(alpha: 0.2)
            : AppColors.surfaceVariant,
          shape: BoxShape.circle,
        ),
        child: Icon(
          _isListening ? Icons.mic : Icons.mic_none,
          color: _isListening ? AppColors.primary : AppColors.onSurfaceVariant,
        ),
      ),
    );
  }
}
```

### Photo Input: Camera + Vision API

```dart
// Phase 2 feature — v1.1 or v1.2
// User snaps a photo of an installation, asks "is this to code?"

Future<void> analyzePhoto(File photo, String question) async {
  // Send to backend which uses GPT-4o Vision
  final response = await apiClient.analyzePhoto(
    photo: photo,
    question: question,
    jurisdiction: currentJurisdiction,
  );
  // Response includes AI analysis + relevant code citations
}
```

---

## 7. UI Design System

### Design Philosophy

**"Wrench, not swiss army knife."** This app is a tool, not a toy. Every pixel serves a purpose. The aesthetic should feel like a well-made tool — solid, trustworthy, no-nonsense.

**Keywords:** Professional. Clean. Trustworthy. Efficient. Construction-grade.

### Color Palette

```dart
class AppColors {
  // Primary — Deep Navy (trust, authority, professional)
  static const primary = Color(0xFF1B3A5C);         // #1B3A5C
  static const primaryLight = Color(0xFF2D5F8A);    // Lighter variant
  static const primaryDark = Color(0xFF0F2440);     // Darker variant

  // Accent — Safety Orange (construction, attention, CTAs)
  static const accent = Color(0xFFFF6B35);           // #FF6B35
  static const accentLight = Color(0xFFFF8F66);

  // Surfaces
  static const background = Color(0xFFF8F9FA);       // Near-white
  static const surface = Color(0xFFFFFFFF);
  static const surfaceVariant = Color(0xFFF0F2F5);

  // Text
  static const onSurface = Color(0xFF1A1A2E);        // Near-black
  static const onSurfaceVariant = Color(0xFF6B7280); // Gray
  static const onPrimary = Color(0xFFFFFFFF);

  // Semantic
  static const success = Color(0xFF22C55E);
  static const warning = Color(0xFFF59E0B);
  static const error = Color(0xFFEF4444);
  static const info = Color(0xFF3B82F6);

  // Citations (special — code references stand out)
  static const citation = Color(0xFF2563EB);          // Blue link
  static const citationBg = Color(0xFFEFF6FF);        // Light blue bg
}
```

### Typography

```dart
class AppTypography {
  // Font: Inter (free, excellent readability, professional)
  // Fallback: system default

  static const _base = TextStyle(fontFamily: 'Inter');

  // Headings
  static final h1 = _base.copyWith(fontSize: 28, fontWeight: FontWeight.w700, height: 1.2);
  static final h2 = _base.copyWith(fontSize: 22, fontWeight: FontWeight.w600, height: 1.3);
  static final h3 = _base.copyWith(fontSize: 18, fontWeight: FontWeight.w600, height: 1.3);

  // Body
  static final body = _base.copyWith(fontSize: 16, fontWeight: FontWeight.w400, height: 1.5);
  static final bodyBold = _base.copyWith(fontSize: 16, fontWeight: FontWeight.w600, height: 1.5);
  static final bodySmall = _base.copyWith(fontSize: 14, fontWeight: FontWeight.w400, height: 1.5);

  // Code sections (monospace for section numbers)
  static final codeRef = TextStyle(
    fontFamily: 'JetBrains Mono',
    fontSize: 14,
    fontWeight: FontWeight.w500,
  );

  // Caption
  static final caption = _base.copyWith(fontSize: 12, fontWeight: FontWeight.w400, height: 1.4);
}
```

### Spacing System

```dart
class AppSpacing {
  static const xs = 4.0;
  static const sm = 8.0;
  static const md = 16.0;
  static const lg = 24.0;
  static const xl = 32.0;
  static const xxl = 48.0;

  // Border radius
  static const radiusSm = 8.0;
  static const radiusMd = 12.0;
  static const radiusLg = 16.0;
  static const radiusXl = 24.0;
  static const radiusFull = 9999.0;
}
```

### Component Library

```dart
// Reusable card for code sections
class CodeSectionCard extends StatelessWidget {
  final String sectionNumber;
  final String heading;
  final String preview;
  final String codeFamily;
  final VoidCallback? onTap;

  @override
  Widget build(BuildContext context) {
    return Card(
      elevation: 0,
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(AppSpacing.radiusMd),
        side: BorderSide(color: AppColors.surfaceVariant, width: 1),
      ),
      child: InkWell(
        borderRadius: BorderRadius.circular(AppSpacing.radiusMd),
        onTap: onTap,
        child: Padding(
          padding: const EdgeInsets.all(AppSpacing.md),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Row(
                children: [
                  _CodeFamilyBadge(family: codeFamily),
                  const SizedBox(width: AppSpacing.sm),
                  Text(sectionNumber, style: AppTypography.codeRef.copyWith(
                    color: AppColors.citation,
                  )),
                ],
              ),
              const SizedBox(height: AppSpacing.sm),
              Text(heading, style: AppTypography.bodyBold),
              const SizedBox(height: AppSpacing.xs),
              Text(preview,
                style: AppTypography.bodySmall.copyWith(
                  color: AppColors.onSurfaceVariant,
                ),
                maxLines: 2,
                overflow: TextOverflow.ellipsis,
              ),
            ],
          ),
        ),
      ),
    );
  }
}

// Citation chip (tappable, links to code section)
class CitationChip extends StatelessWidget {
  final String sectionNumber;
  final String codeFamily;
  final VoidCallback onTap;

  @override
  Widget build(BuildContext context) {
    return ActionChip(
      onPressed: onTap,
      avatar: Icon(Icons.link, size: 14, color: AppColors.citation),
      label: Text(sectionNumber),
      labelStyle: AppTypography.codeRef.copyWith(
        color: AppColors.citation,
        fontSize: 12,
      ),
      backgroundColor: AppColors.citationBg,
      side: BorderSide.none,
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(AppSpacing.radiusSm),
      ),
    );
  }
}
```

---

## 8. Screen-by-Screen Specs

### Home Screen

```
┌─────────────────────────────────────┐
│  CodePocket                    ⚙️   │  ← AppBar: logo + settings
│─────────────────────────────────────│
│                                      │
│  Good morning, Clyde                 │  ← Greeting (time-aware)
│  Electrician · Maine                 │  ← Trade + jurisdiction
│                                      │
│  ┌─────────────────────────────────┐│
│  │  🔍  Ask a code question...     ││  ← Search/ask bar (taps → chat)
│  └─────────────────────────────────┘│
│                                      │
│  ── Quick Actions ────────────────── │
│  ┌──────┐ ┌──────┐ ┌──────┐ ┌─────┐│
│  │  ⚡  │ │  🔧  │ │  🏠  │ │  📖 ││  ← Trade-specific shortcuts
│  │ Elec │ │ Plumb│ │ Build│ │ Saved││     (based on user's trade)
│  └──────┘ └──────┘ └──────┘ └─────┘│
│                                      │
│  ── Recent Questions ─────────────── │
│  ┌─────────────────────────────────┐│
│  │  "GFCI requirements for         ││
│  │   garage outlets?"    →         ││  ← Chat history cards
│  └─────────────────────────────────┘│
│  ┌─────────────────────────────────┐│
│  │  "Max deck railing height?"     ││
│  │                    →             ││
│  └─────────────────────────────────┘│
│                                      │
│  ── Code Updates ─────────────────── │
│  📢 Maine adopted 2023 NEC          │  ← Code change alerts
│  effective July 2026          Read → │
│                                      │
├──────────────────────────────────────┤
│   🏠      💬      📖      🔖       │  ← Bottom nav
│  Home   Chat   Codes   Saved        │
└──────────────────────────────────────┘
```

### Chat Screen (The Core Feature)

```
┌─────────────────────────────────────┐
│  ← CodePocket Chat          🔖     │  ← Back + save conversation
│─────────────────────────────────────│
│                                      │
│  ┌───────────────────────────────┐  │
│  │  What's the max distance      │  │  ← User bubble (right-aligned)
│  │  between outlets in a         │  │     Navy bg, white text
│  │  bathroom?                    │  │
│  └───────────────────────────────┘  │
│                                      │
│  ┌───────────────────────────────┐  │
│  │  Based on NEC 210.52(D):      │  │  ← AI response (left-aligned)
│  │                               │  │     Light bg, dark text
│  │  At least one 125V, 15A or   │  │
│  │  20A receptacle outlet shall  │  │
│  │  be installed within 36 in.   │  │
│  │  of the outside edge of each  │  │
│  │  basin.                       │  │
│  │                               │  │
│  │  📎 NEC 2020 §210.52(D)      │  │  ← Tappable citation chips
│  │  📎 Maine Ch.12 §1           │  │
│  │                               │  │
│  │  ⚠️ Always verify with your  │  │  ← Disclaimer
│  │  local AHJ.                   │  │
│  └───────────────────────────────┘  │
│                                      │
│  ┌───────────────────────────────┐  │
│  │  Follow-up: Does this apply   │  │  ← Suggested follow-ups
│  │  to GFCI protection too? →    │  │     (tappable)
│  └───────────────────────────────┘  │
│                                      │
│──────────────────────────────────────│
│  ┌──────────────────────────┐  🎤  │  ← Input bar + mic button
│  │  Ask a follow-up...      │      │
│  └──────────────────────────┘      │
├──────────────────────────────────────┤
│   🏠      💬      📖      🔖       │
│  Home   Chat   Codes   Saved        │
└──────────────────────────────────────┘
```

### Code Browser Screen

```
┌─────────────────────────────────────┐
│  Code Books                    🔍   │  ← Search toggle
│─────────────────────────────────────│
│                                      │
│  Maine · All Codes                   │  ← Jurisdiction selector
│                                      │
│  ┌─────────────────────────────────┐│
│  │  ⚡ National Electrical Code    ││  ← Code book tiles
│  │     NEC 2020 · 4,023 sections  ││     Color-coded by trade
│  │                    Browse →     ││
│  └─────────────────────────────────┘│
│  ┌─────────────────────────────────┐│
│  │  🏠 Int'l Residential Code     ││
│  │     IRC 2021 · 2,514 sections  ││
│  │                    Browse →     ││
│  └─────────────────────────────────┘│
│  ┌─────────────────────────────────┐│
│  │  🔧 Int'l Plumbing Code        ││
│  │     IPC 2021 · 1,832 sections  ││
│  │                    Browse →     ││
│  └─────────────────────────────────┘│
│  ┌─────────────────────────────────┐│
│  │  🌡️ Int'l Mechanical Code      ││
│  │     IMC 2021 · 1,601 sections  ││
│  │                    Browse →     ││
│  └─────────────────────────────────┘│
│  ┌─────────────────────────────────┐│
│  │  🔥 Int'l Fire Code            ││
│  │     IFC 2021 · 2,100 sections  ││
│  │                    Browse →     ││
│  └─────────────────────────────────┘│
│                                      │
├──────────────────────────────────────┤
│   🏠      💬      📖      🔖       │
│  Home   Chat   Codes   Saved        │
└──────────────────────────────────────┘
```

### Onboarding: Trade Selector

```
┌─────────────────────────────────────┐
│                                      │
│           ⚡ CodePocket             │
│                                      │
│     What do you do?                  │
│     We'll customize your feed.       │
│                                      │
│  ┌─────────────────────────────────┐│
│  │  ⚡ Electrician            ○    ││  ← Radio-style selection
│  └─────────────────────────────────┘│
│  ┌─────────────────────────────────┐│
│  │  🔧 Plumber                ○    ││
│  └─────────────────────────────────┘│
│  ┌─────────────────────────────────┐│
│  │  🏗️ General Contractor     ○    ││
│  └─────────────────────────────────┘│
│  ┌─────────────────────────────────┐│
│  │  🌡️ HVAC Technician        ○    ││
│  └─────────────────────────────────┘│
│  ┌─────────────────────────────────┐│
│  │  🔍 Home Inspector          ○   ││
│  └─────────────────────────────────┘│
│  ┌─────────────────────────────────┐│
│  │  🔨 Handyman                ○   ││
│  └─────────────────────────────────┘│
│  ┌─────────────────────────────────┐│
│  │  🏛️ Building Inspector      ○   ││
│  └─────────────────────────────────┘│
│  ┌─────────────────────────────────┐│
│  │  📐 Architect / Engineer    ○   ││
│  └─────────────────────────────────┘│
│  ┌─────────────────────────────────┐│
│  │  🎓 Student / Apprentice    ○   ││
│  └─────────────────────────────────┘│
│  ┌─────────────────────────────────┐│
│  │  🏠 DIY Homeowner           ○   ││
│  └─────────────────────────────────┘│
│                                      │
│       ┌──────────────────────┐       │
│       │       Continue       │       │  ← Disabled until selection
│       └──────────────────────┘       │
│                                      │
└──────────────────────────────────────┘
```

### Onboarding: State Selector

```
┌─────────────────────────────────────┐
│  ← Back                             │
│─────────────────────────────────────│
│                                      │
│     Where do you work?               │
│     We'll show the right codes.      │
│                                      │
│  ┌─────────────────────────────────┐│
│  │  🔍  Search states...           ││  ← Search filter
│  └─────────────────────────────────┘│
│                                      │
│  ── Popular ──────────────────────── │
│  [California] [Texas] [Florida]      │  ← Quick-select chips
│  [New York]   [Pennsylvania]         │
│                                      │
│  ── All States ───────────────────── │
│  Alabama                             │
│  Alaska                              │
│  Arizona                             │
│  ...                                 │
│  Maine                               │
│  ...                                 │
│  Wyoming                             │
│                                      │
│       ┌──────────────────────┐       │
│       │     Get Started      │       │
│       └──────────────────────┘       │
│                                      │
└──────────────────────────────────────┘
```

---

## 9. Accessibility

### Requirements

- **Minimum tap target:** 48x48dp (Material standard)
- **Color contrast:** WCAG AA minimum (4.5:1 for text, 3:1 for large text)
- **Screen reader:** All interactive elements labeled with `Semantics()`
- **Font scaling:** Support up to 1.5x system font scale without layout breaking
- **Dark mode:** Full support (contractors work in bright sun AND dark basements)
- **Haptic feedback:** On mic start/stop, on citation tap

### Dark Mode Colors

```dart
class AppColorsDark {
  static const primary = Color(0xFF5B9BD5);        // Lighter navy for dark bg
  static const accent = Color(0xFFFF8F66);          // Lighter orange
  static const background = Color(0xFF0F1117);      // Near-black
  static const surface = Color(0xFF1A1B26);         // Dark surface
  static const surfaceVariant = Color(0xFF252733);  // Slightly lighter
  static const onSurface = Color(0xFFE8E9ED);       // Off-white
  static const onSurfaceVariant = Color(0xFF9CA3AF);// Gray
}
```

---

## 10. Testing Strategy

### Test Pyramid

```
          /  E2E Tests  \           ← integration_test/ (Patrol or Appium)
         /  (10 tests)   \             - Full flow: signup → ask → get answer
        /─────────────────\
       /  Widget Tests     \        ← test/ (Flutter test framework)
      /   (50 tests)        \          - Screen renders, interactions work
     /───────────────────────\
    /   Unit Tests            \      ← test/ (standard Dart tests)
   /    (200 tests)            \        - Providers, repositories, models
  /─────────────────────────────\
```

### Key Test Scenarios

```dart
// Unit: Question counting for free tier
test('free user can ask 5 questions then hits paywall', () async {
  final guard = QuestionGuard(revenueCat: mockRC, storage: mockStorage);
  for (int i = 0; i < 5; i++) {
    final result = await guard.canAskQuestion();
    expect(result.allowed, true);
    mockStorage.incrementUsage();
  }
  final result = await guard.canAskQuestion();
  expect(result.paywallRequired, true);
});

// Widget: Chat screen renders messages
testWidgets('chat screen shows user and AI messages', (tester) async {
  await tester.pumpWidget(testApp(home: ChatScreen()));
  await tester.enterText(find.byType(TextField), 'What is the NEC code for GFCI?');
  await tester.tap(find.byType(SendButton));
  await tester.pumpAndSettle();
  expect(find.text('What is the NEC code for GFCI?'), findsOneWidget);
  expect(find.byType(CitationChip), findsWidgets);
});

// Integration: Full signup flow
testWidgets('new user completes onboarding', (tester) async {
  await tester.pumpWidget(testApp());
  // Login
  await tester.tap(find.byType(SignInWithGoogleButton));
  // Select trade
  await tester.tap(find.text('Electrician'));
  await tester.tap(find.text('Continue'));
  // Select state
  await tester.enterText(find.byType(TextField), 'Maine');
  await tester.tap(find.text('Maine'));
  await tester.tap(find.text('Get Started'));
  // Should be on home screen
  expect(find.text('Good morning'), findsOneWidget);
});
```

---

## 11. Package Dependencies

### pubspec.yaml

```yaml
name: codepocket
description: AI-powered building code assistant for contractors
version: 1.0.0+1

environment:
  sdk: '>=3.2.0 <4.0.0'

dependencies:
  flutter:
    sdk: flutter

  # State Management
  flutter_riverpod: ^2.5.0
  riverpod_annotation: ^2.3.0

  # Navigation
  go_router: ^14.0.0

  # Networking
  dio: ^5.4.0

  # Local Storage
  drift: ^2.16.0
  sqlite3_flutter_libs: ^0.5.0
  shared_preferences: ^2.2.0
  flutter_secure_storage: ^9.2.0

  # Authentication
  firebase_auth: ^4.19.0
  google_sign_in: ^6.2.0
  sign_in_with_apple: ^5.0.0

  # Payments
  purchases_flutter: ^8.0.0  # RevenueCat

  # Voice Input
  speech_to_text: ^7.0.0

  # Camera (Phase 2)
  camera: ^0.11.0
  image_picker: ^1.0.0

  # UI
  flutter_animate: ^4.5.0     # Smooth animations
  shimmer: ^3.0.0              # Loading skeletons
  flutter_markdown: ^0.7.0    # Render AI markdown responses
  infinite_scroll_pagination: ^4.0.0  # Code section lists

  # Utilities
  freezed_annotation: ^2.4.0  # Immutable models
  json_annotation: ^4.9.0     # JSON serialization
  intl: ^0.19.0                # Date/number formatting
  connectivity_plus: ^6.0.0   # Online/offline detection
  url_launcher: ^6.3.0        # Open external links
  package_info_plus: ^8.0.0   # App version info

dev_dependencies:
  flutter_test:
    sdk: flutter
  integration_test:
    sdk: flutter

  # Code Generation
  build_runner: ^2.4.0
  freezed: ^2.5.0
  json_serializable: ^6.8.0
  riverpod_generator: ^2.4.0
  drift_dev: ^2.16.0

  # Linting
  flutter_lints: ^4.0.0

  # Mocking
  mockito: ^5.4.0
```

### Backend (Python)

```txt
# requirements.txt
fastapi==0.111.*
uvicorn==0.30.*
pydantic==2.8.*
sqlalchemy==2.0.*
asyncpg==0.30.*
alembic==1.13.*

# RAG
openai==1.40.*
pinecone-client==5.0.*
langchain==0.3.*
langchain-openai==0.2.*

# Auth
firebase-admin==6.5.*
python-jose==3.3.*

# Code ingestion
beautifulsoup4==4.12.*
pdfplumber==0.11.*
PyYAML==6.0.*
httpx==0.27.*
```

---

## 12. Implementation Roadmap

### Sprint 1 (Week 1-2): Foundation

- [ ] Flutter project setup with clean architecture
- [ ] GoRouter navigation + bottom nav shell
- [ ] Design system (colors, typography, spacing)
- [ ] Core widgets (cards, chips, loading states)
- [ ] Firebase Auth (Google Sign-In + Apple Sign-In)
- [ ] Onboarding flow (trade + state selection)
- [ ] Drift database setup + migration system

### Sprint 2 (Week 3-4): Core Chat

- [ ] FastAPI backend scaffold
- [ ] AI chat endpoint (streaming SSE response)
- [ ] RAG pipeline: ingest first code book (NEC 2020)
- [ ] Chat UI (message list, streaming text, input bar)
- [ ] Citation display (tappable chips → code section)
- [ ] Chat history persistence
- [ ] Voice input (speech_to_text)

### Sprint 3 (Week 5-6): Code Browser + Search

- [ ] Code browser screen (book list → section tree)
- [ ] Code section detail screen (full text, highlight)
- [ ] Search (keyword + semantic via backend)
- [ ] Save/bookmark code sections
- [ ] Offline caching for viewed sections
- [ ] Ingest remaining code books (IRC, IPC, IMC)

### Sprint 4 (Week 7-8): Monetization + Polish

- [ ] RevenueCat integration
- [ ] Paywall screen (free trial CTA)
- [ ] Free tier enforcement (5 questions/month)
- [ ] Settings screen (jurisdiction, trade, subscription)
- [ ] Code change notifications (push)
- [ ] Dark mode
- [ ] Accessibility audit
- [ ] App Store + Play Store listings

### Sprint 5 (Week 9-10): Launch Prep

- [ ] QA testing on real devices (iOS + Android)
- [ ] Performance optimization (startup time, scroll perf)
- [ ] Error handling & edge cases
- [ ] App Store review submission
- [ ] Landing page (codepocket.com)
- [ ] Beta test with 5-10 Maine contractors
- [ ] Iterate on feedback

### Post-Launch (Week 11+)

- [ ] Photo input (GPT-4o Vision)
- [ ] Team plan features
- [ ] Inspector report generation
- [ ] Expand to 10 states
- [ ] YouTube Shorts marketing campaign
- [ ] Reddit/community launch posts

---

## Appendix: Key Flutter Files for Clyde

Since Clyde handles the Unity/editor side, here's what he needs to know about the Flutter setup:

### Running the App
```bash
# Install Flutter (if not already)
flutter doctor

# Get dependencies
flutter pub get

# Run code generation (for Riverpod, Freezed, Drift)
dart run build_runner watch --delete-conflicting-outputs

# Run on connected device or emulator
flutter run

# Run tests
flutter test

# Build for release
flutter build ios    # Requires Mac
flutter build apk    # Android
```

### IDE Setup
- VS Code with Flutter extension (recommended)
- Or Android Studio with Flutter plugin
- Hot reload works — changes appear instantly during development

### Project Conventions
- One feature = one folder under `features/`
- Each feature has `data/`, `domain/`, `presentation/`
- Providers go in `presentation/` (colocated with screens)
- Models use `freezed` for immutability
- All API calls go through `core/network/api_client.dart`
- Never call APIs directly from widgets — always through a provider
