# 🎨 Quranic Search - Visual Architecture Diagrams

## 📊 System Overview

```
┌──────────────────────────────────────────────────────────────────────────┐
│                                                                          │
│                    🌐 CLIENT LAYER (Browser)                            │
│                                                                          │
│   ┌──────────────────────────────────────────────────────────────┐      │
│   │   React.js Frontend (Port 3000)                              │      │
│   │   • HomeForm - Search input                                 │      │
│   │   • ResultsForm - Display verses                           │      │
│   │   • Verse Component - Arabic text rendering                 │      │
│   │   • Navbar, Bookmarks, About pages                         │      │
│   └──────────────────────────────────────────────────────────────┘      │
│                              ↑                     ↑                      │
│                              │                     │                      │
│                        JSON Request          JSON Request                │
└──────────────────────────────┼─────────────────────┼─────────────────────┘
                               │                     │
                               │                     │
              ┌────────────────┴────────┐   ┌───────┴────────┐
              │                           │   │                │
              ▼                           ▼   ▼                ▼
┌──────────────────────────┐   ┌─────────────────────────────┐
│  LEXICAL SEARCH API      │   │  SEMANTIC SEARCH API         │
│  Django REST Framework   │   │  Flask + Machine Learning   │
│  (Port 8000)             │   │  (Port 5000)                │
├──────────────────────────┤   ├─────────────────────────────┤
│                          │   │                             │
│  ✅ Fast                 │   │  ✅ Intelligent             │
│  ✅ Exact matching        │   │  ✅ Meaning-based          │
│  ✅ SQL queries           │   │  ✅ Word embeddings        │
│                          │   │                             │
│  Endpoints:              │   │  Endpoints:                 │
│  • /search/<words>       │   │  • /similar-verse/<query>  │
│  • /surah/<id>           │   │  • /similar-word/<word>    │
│  • /verse-in-surah/...  │   │                             │
│  • /verse-in-quran/<id> │   │                             │
│  • /all                  │   │                             │
└────────────┬─────────────┘   └────────────┬────────────────┘
             │                              │
             │                              │
             ▼                              ▼
┌───────────────────────────────────────────────────────────────┐
│                    DATABASE LAYER                            │
│                     SQLite3                                  │
├───────────────────────────────────────────────────────────────┤
│                                                               │
│  📊 Tables:                                                   │
│  ┌────────────────────────┐  ┌──────────────────────┐       │
│  │  search_verse          │  │  search_surah       │       │
│  │  (6,236 rows)         │  │  (114 rows)         │       │
│  ├────────────────────────┤  ├──────────────────────┤       │
│  │  PK  id               │  │  PK  id             │       │
│  │  UK  verse_pk         │  │      name           │       │
│  │      page             │  │      nameWithout... │       │
│  │      juz              │  └──────────────────────┘       │
│  │      hizbQuarter      │                                  │
│  │  FK  surah → Surah    │                                  │
│  │      verse            │                                  │
│  │      verseWithout     │                                  │
│  │      Tashkeel         │                                  │
│  │      numberInSurah    │                                  │
│  │  UK  numberInQuran    │                                  │
│  │      audio            │                                  │
│  │      audio1, audio2   │                                  │
│  │      sajda            │                                  │
│  └────────────────────────┘                                  │
└───────────────────────────────────────────────────────────────┘
```

---

## 🔄 Request Flow: Lexical Search

```
┌────────────────────────────────────────────────────────────┐
│                    USER TYPES QUERY                        │
│                    "الله"                                   │
└────────────────────────┬───────────────────────────────────┘
                         │
                         ▼
┌────────────────────────────────────────────────────────────┐
│  1. FRONTEND (React)                                       │
│     • User types in HomeForm                              │
│     • Selects "Lexical Search"                            │
│     • Triggers API call                                    │
└────────────────────────┬───────────────────────────────────┘
                         │
                         │ GET /api/lexical/search/الله
                         ▼
┌────────────────────────────────────────────────────────────┐
│  2. DJANGO URL ROUTING                                     │
│     • api/urls.py → matches pattern                        │
│     • Routes to search/urls.py                             │
│     • Calls search() view                                  │
└────────────────────────┬───────────────────────────────────┘
                         │
                         ▼
┌────────────────────────────────────────────────────────────┐
│  3. MIDDLEWARE PROCESSING                                  │
│     • CORS Middleware                                      │
│     • Session Middleware                                   │
│     • Security Headers                                     │
└────────────────────────┬───────────────────────────────────┘
                         │
                         ▼
┌────────────────────────────────────────────────────────────┐
│  4. VIEW FUNCTION (views.py)                               │
│                                                             │
│     verses = Verses.objects.filter(                        │
│       Q(verse__icontains="الله") |                        │
│       Q(verseWithoutTashkeel__icontains="الله")           │
│     )                                                       │
│     data = VersesSerializers(verses, many=True).data     │
└────────────────────────┬───────────────────────────────────┘
                         │
                         ▼
┌────────────────────────────────────────────────────────────┐
│  5. DJANGO ORM QUERY                                       │
│     Converts to SQL:                                       │
│     SELECT * FROM search_verse                             │
│     WHERE verse LIKE '%الله%'                             │
│        OR verseWithoutTashkeel LIKE '%الله%'             │
└────────────────────────┬───────────────────────────────────┘
                         │
                         ▼
┌────────────────────────────────────────────────────────────┐
│  6. DATABASE EXECUTION                                     │
│     SQLite executes query                                  │
│     Returns ~125 matching verses                           │
└────────────────────────┬───────────────────────────────────┘
                         │
                         ▼
┌────────────────────────────────────────────────────────────┐
│  7. DATA SERIALIZATION                                     │
│     Verses → Python dict → JSON                            │
│     {                                                       │
│       "length": 125,                                       │
│       "data": [...verse objects...]                        │
│     }                                                       │
└────────────────────────┬───────────────────────────────────┘
                         │
                         ▼
┌────────────────────────────────────────────────────────────┐
│  8. RESPONSE TO FRONTEND                                   │
│     JSON response sent back                                │
│     React renders results                                  │
└────────────────────────────────────────────────────────────┘
```

---

## 🔄 Request Flow: Semantic Search

```
┌────────────────────────────────────────────────────────────┐
│                    USER TYPES QUERY                        │
│                    "صلاة" (Prayer)                          │
└────────────────────────┬───────────────────────────────────┘
                         │
                         ▼
┌────────────────────────────────────────────────────────────┐
│  1. FRONTEND                                               │
│     • Selects "Semantic Search"                            │
│     • Calls Flask API                                      │
└────────────────────────┬───────────────────────────────────┘
                         │
                         │ GET /api/semantic/similar-verse/صلاة
                         ▼
┌────────────────────────────────────────────────────────────┐
│  2. FLASK APP (app.py)                                     │
│     • Receives "صلاة"                                      │
│     • Routes to MostSimilarVerse resource                  │
└────────────────────────┬───────────────────────────────────┘
                         │
                         ▼
┌────────────────────────────────────────────────────────────┐
│  3. SEMANTIC PROCESSING                                    │
│                                                             │
│     a) Clean & tokenize "صلاة"                            │
│     b) Load Word2Vec model                                 │
│     c) Get word vector for "صلاة"                         │
└────────────────────────┬───────────────────────────────────┘
                         │
                         ▼
┌────────────────────────────────────────────────────────────┐
│  4. SIMILARITY CALCULATION                                 │
│     For each verse (6,236 verses):                         │
│     • Compare "صلاة" vector with each word               │
│     • Calculate cosine similarity                          │
│     • Apply 3 methods:                                     │
│       - Max similarity (best match)                       │
│       - Average similarity                                 │
│       - Frequency (similarity > 0.3)                      │
└────────────────────────┬───────────────────────────────────┘
                         │
                         ▼
┌────────────────────────────────────────────────────────────┐
│  5. RESULT AGGREGATION                                     │
│     • Combine scores from all methods                      │
│     • Sort by relevance                                    │
│     • Return top 50 verses with scores                    │
│     • Example:                                             │
│       [                                                    │
│         {score: 0.95, verse_id: 52, ...},                │
│         {score: 0.89, verse_id: 456, ...},               │
│         ...                                                 │
│       ]                                                     │
└────────────────────────┬───────────────────────────────────┘
                         │
                         ▼
┌────────────────────────────────────────────────────────────┐
│  6. FETCH FULL VERSE DATA                                  │
│     Frontend calls Lexical API for each verse_id          │
│     Gets: text, audio, surah, metadata                    │
└────────────────────────┬───────────────────────────────────┘
                         │
                         ▼
┌────────────────────────────────────────────────────────────┐
│  7. DISPLAY RESULTS                                        │
│     • Shows semantically similar verses                   │
│     • Ranked by meaning similarity                        │
│     • Includes prayer-related concepts                     │
└────────────────────────────────────────────────────────────┘
```

---

## 🗄️ Database Relationships

```
┌──────────────────────────────────┐
│         SURAH                     │
│      (114 Chapters)               │
├──────────────────────────────────┤
│  PK  id                           │
│      name                         │
│      nameWithoutTashkeel         │
│                                   │
│  Examples:                        │
│  • id=1, name="الفاتحة"         │
│  • id=2, name="البقرة"           │
└──────────┬────────────────────────┘
           │
           │ 1:N
           │ (Foreign Key)
           │
           ▼
┌──────────────────────────────────────────────────┐
│              VERSES                               │
│           (6,236 verses)                         │
├──────────────────────────────────────────────────┤
│  PK    id                    (auto-increment)    │
│  PK/UK verse_pk             (S001V001 format)    │
│       page                  (1-604)              │
│       hizbQuarter          (1-240)              │
│       juz                   (1-30)              │
│  FK    surah_id            → Surah.id            │
│       verse                 (with diacritics)    │
│       verseWithoutTashkeel  (plain Arabic)       │
│       numberInSurah         (verse # in surah)   │
│  UK    numberInQuran        (1-6236, global)     │
│       audio                 (primary audio URL)  │
│       audio1, audio2        (alternative URLs)   │
│       sajda                 (prostration verse?) │
│                                                      │
│  Example Row:                                       │
│  verse_pk: "S002V255"                               │
│  verse: "الله لا إله إلا هو الحي القيوم..."       │
│  numberInQuran: 286                                 │
│  surah: "البقرة"                                    │
└──────────────────────────────────────────────────┘
```

---

## 🔀 Dual Backend Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      CLIENT (React)                          │
│                                                             │
│  ┌────────────────────┐      ┌────────────────────┐        │
│  │  Search Component   │      │  Results Display   │        │
│  │  • Lexical button   │      │  • Verse list      │        │
│  │  • Semantic button │      │  • Search results  │        │
│  └──────────┬─────────┘      └──────────┬─────────┘        │
│             │                            │                   │
│             │ API Calls                  │ Renders Data      │
│             ▼                            │                   │
│  ┌──────────────────────────────────────────┐              │
│  │        API Layer (Axios/Fetch)            │              │
│  └──┬────────────────────────────────────┬──┘              │
└─────┼────────────────────────────────────┼──────────────────┘
      │                                    │
      │                                    │
  ┌───▼─────────────────┐         ┌───────▼────────────┐
  │  LEXICAL BACKEND    │         │ SEMANTIC BACKEND   │
  │  (Django + DRF)     │         │  (Flask + ML)      │
  ├─────────────────────┤         ├────────────────────┤
  │  Role: Exact match  │         │  Role: ML similarity│
  │  • Fast queries     │         │  • Word2Vec        │
  │  • Simple logic     │         │  • Sentence embed   │
  │  • Database-driven  │         │  • Model inference │
  └───┬─────────────────┘         └───┬───────────────┘
      │                               │
      │                               │
      ▼                               ▼
  ┌────────────────────────┐  ┌────────────────────┐
  │    SQLite Database     │  │  Pre-trained Models│
  │  • Verses table        │  │  • Word2Vec Twitter │
  │  • Surah table         │  │  • AraVec          │
  │                        │  │  • FastText        │
  └────────────────────────┘  └────────────────────┘
```

---

## 🚀 Deployment Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                        PRODUCTION                             │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌────────────────┐    ┌────────────────┐                   │
│  │   Domain.com   │    │   Load Balancer│                   │
│  │   (Nginx)      │────│   (Optional)   │                   │
│  └────────┬───────┘    └────────┬───────┘                   │
│           │                      │                            │
│           │                      │                            │
│  ┌────────▼──────────────────────▼────────┐                │
│  │                                           │                │
│  │         ┌──────────────┐                  │                │
│  │         │   React App   │                  │                │
│  │         │  (Built &     │                  │                │
│  │         │   Served via  │                  │                │
│  │         │   Nginx)      │                  │                │
│  │         └──────┬───────┘                  │                │
│  │                │                           │                │
│  │                │                           │                │
│  │         ┌──────▼───────┐                   │                │
│  │         │ Gunicorn      │                   │                │
│  │         │ (Django WSGI) │                   │                │
│  │         └──────┬───────┘                   │                │
│  │                │                           │                │
│  │         ┌──────▼───────┐                   │                │
│  │         │   uWSGI       │                   │                │
│  │         │  (Flask WSGI) │                   │                │
│  │         └──────┬───────┘                   │                │
│  │                │                           │                │
│  │                │                           │                │
│  │         ┌──────▼───────┐                   │                │
│  │         │  SQLite DB   │                   │                │
│  │         └──────────────┘                   │                │
│  │                                             │                │
│  └─────────────────────────────────────────────┘                │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

---

## 📝 Summary

This is a **production-ready, multi-backend Quranic search system** that demonstrates:
- ✅ Full-stack JavaScript (React) + Python (Django/Flask)
- ✅ Dual search modes (lexical + semantic)
- ✅ REST API design patterns
- ✅ Machine Learning integration (Word2Vec)
- ✅ Arabic NLP processing
- ✅ SQLite database management
- ✅ CORS and security best practices

**Perfect for learning backend architecture, Django, Flask, NLP, and full-stack development!**
