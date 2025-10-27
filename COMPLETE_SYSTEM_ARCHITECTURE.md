# 🌐 Quranic Search - Complete System Architecture

This document provides a complete overview of the entire Quranic Search system, including both the **Lexical Search** (Django) and **Semantic Search** (Flask) backends.

---

## 🏗️ Overall System Architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│                        REACT FRONTEND (Port 3000)                     │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐         │
│  │ HomeForm       │  │ ResultsForm   │  │ Verse Display │         │
│  │ • Query input  │  │ • Search      │  │ • Verse text   │         │
│  │ • Search type  │  │   results     │  │ • Translations│         │
│  │   (lexical/    │  │ • Bookmarks   │  │ • Audio       │         │
│  │    semantic)   │  │ • Pagination  │  │               │         │
│  └────────┬───────┘  └────────┬───────┘  └────────┬───────┘         │
└───────────┼────────────────────┼────────────────────┼───────────────┘
            │                     │                    │
            │                     │                    │
  ┌─────────▼──────────┐  ┌──────▼─────────┐         │
  │  LEXICAL SEARCH     │  │ SEMANTIC       │         │
  │  (Django - Port 8000)│  │ SEARCH        │         │
  │                     │  │ (Flask - Port 5000)      │
  │  • Keyword matching │  │                │         │
  │  • Exact search     │  │  • Word2Vec    │         │
  │  • Fast & simple     │  │  • Meaning-based│       │
  │                     │  │  • ML-powered  │         │
  └──────────┬──────────┘  └──────┬─────────┘         │
             │                    │                    │
             │                    │                    │
             └────────────────────┼────────────────────┘
                                  │
                          ┌───────▼────────┐
                          │ SQLite Database│
                          │ • quran.db     │
                          │ • 6,236 verses │
                          └────────────────┘
```

---

## 📊 System Components Breakdown

### 1. **Frontend (React)**
- **Technology**: React.js with functional components
- **Purpose**: User interface for Quranic search
- **Key Features**:
  - Dual search modes (lexical + semantic)
  - Result display with Arabic text rendering
  - Bookmark functionality
  - Audio playback integration
  - Responsive design

### 2. **Lexical Search Backend (Django)**
- **Technology**: Django 5.2 + Django REST Framework
- **Purpose**: Keyword-based exact matching search
- **Database**: SQLite3 with pre-populated verse data
- **Endpoints**:
  - `/api/lexical/search/<words>` - Search by keywords
  - `/api/lexical/surah/<id>` - Get surah verses
  - `/api/lexical/verse-in-surah/<id>/<id>` - Get specific verse
- **Search Method**: SQL `LIKE` queries (case-insensitive)

### 3. **Semantic Search Backend (Flask)**
- **Technology**: Flask + Gensim (Word2Vec)
- **Purpose**: Meaning-based similarity search using ML
- **Models Used**:
  - Word2Vec Twitter model (Arabic tweets)
  - AraVec (Arabic Word2Vec pre-trained models)
  - FastText embeddings
- **Endpoints**:
  - `/api/semantic/similar-verse/<query>` - Semantic verse search
  - `/api/semantic/similar-word/<word>` - Similar word lookup
- **Search Method**: Cosine similarity on word embeddings

---

## 🔄 Complete Data Flow

### Scenario 1: Lexical Search

```
User Types: "الله"
Search Type: Lexical
                         ↓
┌─────────────────────────────────────────┐
│ Frontend sends request                  │
│ GET http://localhost:8000/api/lexical/  │
│           search/الله                    │
└────────────┬────────────────────────────┘
             │
             ↓
┌─────────────────────────────────────────┐
│ Django Backend (views.py)               │
│ • Receives "الله"                       │
│ • Queries database using SQL LIKE       │
└────────────┬────────────────────────────┘
             │
             ↓
┌─────────────────────────────────────────┐
│ SQL Query                                │
│ SELECT * FROM verses                     │
│ WHERE verse LIKE '%الله%'               │
│   OR verseWithoutTashkeel LIKE '%الله%' │
└────────────┬────────────────────────────┘
             │
             ↓
┌─────────────────────────────────────────┐
│ Results: ~125 verses                     │
│ Including:                               │
│ • Ayat Al-Kursi (2:255)                │
│ • Ikhlas verse (112:1)                  │
│ • And 123 more...                       │
└────────────┬────────────────────────────┘
             │
             ↓
┌─────────────────────────────────────────┐
│ Serialized JSON response                │
│ {                                        │
│   "length": 125,                        │
│   "data": [verse objects...]            │
│ }                                        │
└────────────┬────────────────────────────┘
             │
             ↓
┌─────────────────────────────────────────┐
│ Frontend renders results                 │
│ • Displays 125 verses                   │
│ • Shows surah, verse number            │
│ • Arabic text with diacritics            │
└─────────────────────────────────────────┘
```

### Scenario 2: Semantic Search

```
User Types: "صلاة"
Search Type: Semantic
                         ↓
┌─────────────────────────────────────────┐
│ Frontend sends request                   │
│ GET http://localhost:5000/api/semantic/  │
│           similar-verse/صلاة            │
└────────────┬────────────────────────────┘
             │
             ↓
┌─────────────────────────────────────────┐
│ Flask Backend (app.py)                  │
│ • Receives "صلاة"                        │
│ • Loads Word2Vec model                   │
└────────────┬────────────────────────────┘
             │
             ↓
┌─────────────────────────────────────────┐
│ Semantic Processing                     │
│ 1. Clean & tokenize query               │
│ 2. For each word in query:             │
│    - Get word vector from model        │
│    - Compare with all 6,236 verses     │
│    - Calculate similarity scores       │
│ 3. Methods applied:                     │
│    - Max similarity                     │
│    - Average similarity                │
│    - Frequency (similarity > 0.3)      │
│ 4. Combine results from all methods    │
└────────────┬────────────────────────────┘
             │
             ↓
┌─────────────────────────────────────────┐
│ Results: Top 50 verses ranked by        │
│          semantic similarity            │
│ Including:                              │
│ 1. Verses with "صلاة" (exact)           │
│ 2. Verses with "ركوع" (related concept)│
│ 3. Verses about worship                 │
└────────────┬────────────────────────────┘
             │
             ↓
┌─────────────────────────────────────────┐
│ JSON Response with scores                │
│ {                                        │
│   "verses": [                            │
│     {score: 0.95, verse_id: 52, ...},  │
│     {score: 0.89, verse_id: 123, ...}   │
│   ]                                      │
│ }                                        │
└────────────┬────────────────────────────┘
             │
             ↓
┌─────────────────────────────────────────┐
│ Frontend fetches full verse data        │
│ from Lexical API using verse IDs        │
└─────────────────────────────────────────┘
             │
             ↓
┌─────────────────────────────────────────┐
│ Final result: Semantic verses with      │
│ all metadata (text, audio, surah, etc.) │
└─────────────────────────────────────────┘
```

---

## 🔄 How Lexical + Semantic Work Together

```
User Query: "الصلاة والزكاة" (Prayer and Charity)
Type: Semantic
                            ↓
          ┌─────────────────────────────────┐
          │ Semantic API Processes Query    │
          │ • Uses Word2Vec to find verses  │
          │   related to prayer & charity    │
          │ • Returns top 50 verse IDs      │
          │   + similarity scores           │
          └──────────┬──────────────────────┘
                     │
                     │ verse_ids: [52, 123, 456, ...]
                     ↓
          ┌─────────────────────────────────┐
          │ Lexical API Fetches Full Data   │
          │ • For each verse ID, gets:      │
          │   - Full Arabic text            │
          │   - Verse metadata              │
          │   - Audio URLs                  │
          │   - Surah info                  │
          └──────────┬──────────────────────┘
                     │
                     ↓
          ┌─────────────────────────────────┐
          │ Frontend Displays Combined      │
          │ Results:                        │
          │ • Meaning-based verses          │
          │ • Ranked by relevance           │
          │ • With full Arabic text         │
          └─────────────────────────────────┘
```

---

## 📁 Complete Project Structure

```
quranic-search-v2/
├── frontend/                          # React.js UI
│   ├── public/
│   │   ├── fonts/                     # Arabic fonts (Amiri, Kufi)
│   │   ├── images/
│   │   └── index.html
│   └── src/
│       ├── components/                # Reusable components
│       │   ├── HomeForm/
│       │   ├── ResultsForm/
│       │   ├── Verse/
│       │   └── Navbar/
│       ├── containers/                # Page-level components
│       │   ├── Home/
│       │   ├── Results/
│       │   ├── About/
│       │   └── Bookmarks/
│       ├── App.js
│       └── index.js
│
├── backend/
│   ├── api/
│   │   ├── lexical/                   # Django Lexical Search
│   │   │   ├── api/
│   │   │   │   ├── settings.py        # Django configuration
│   │   │   │   ├── urls.py            # URL routing
│   │   │   │   └── wsgi.py            # WSGI entry point
│   │   │   ├── search/                # Main app
│   │   │   │   ├── models.py          # Verses & Surah models
│   │   │   │   ├── views.py           # API endpoints
│   │   │   │   ├── serializers.py     # JSON conversion
│   │   │   │   ├── urls.py            # App URLs
│   │   │   │   └── admin.py           # Django admin
│   │   │   ├── db/
│   │   │   │   ├── quran.db           # Source Quran database
│   │   │   │   └── db.sqlite3         # Django database
│   │   │   ├── manage.py
│   │   │   └── requirements.txt
│   │   │
│   │   └── semantic/                  # Flask Semantic Search
│   │       ├── app.py                 # Flask app entry
│   │       ├── src/
│   │       │   └── models/
│   │       │       ├── predict.py     # API endpoints
│   │       │       ├── semantic_methods.py   # ML logic
│   │       │       ├── preprocess.py  # Text cleaning
│   │       │       └── pooling.py    # Pooling algorithms
│   │       ├── models/                # Trained Word2Vec models
│   │       ├── notebooks/             # Jupyter notebooks
│   │       ├── data/
│   │       │   ├── external/          # Raw Quran text
│   │       │   └── processed/         # Cleaned text
│   │       └── requirements.txt
│
└── scripts/                           # Helper scripts
    ├── run.sh                         # Start all services
    ├── start.sh                       # Setup & run
    └── down.sh                        # Stop all services
```

---

## 🔑 Key Design Differences: Lexical vs Semantic

### Lexical Search (Django)
| Aspect | Description |
|--------|-------------|
| **Purpose** | Find exact keyword matches |
| **Speed** | Fast (SQL LIKE queries) |
| **Accuracy** | Exact text match |
| **Use Case** | When you know specific words to search |
| **Technology** | Django ORM + SQLite |
| **Example** | Search "الله" → finds all verses with "الله" |

### Semantic Search (Flask)
| Aspect | Description |
|--------|-------------|
| **Purpose** | Find verses by meaning/concept |
| **Speed** | Slower (ML processing) |
| **Accuracy** | Contextual/meaning-based |
| **Use Case** | When you want concept-based results |
| **Technology** | Flask + Gensim Word2Vec |
| **Example** | Search "صلاة" → finds verses about prayer, worship, ركوع (bowing) |

---

## 🚀 Running the Complete System

### Prerequisites
```bash
# Python 3.x
# Node.js 14+
# SQLite3
```

### Step-by-Step Setup

#### 1. Lexical Backend (Django)
```bash
cd backend/api/lexical
python -m venv venv
.\venv\Scripts\Activate.ps1    # Windows PowerShell
# or: source venv/bin/activate  # Linux/Mac
pip install -r requirements.txt
python manage.py runserver
# Runs on http://localhost:8000
```

#### 2. Semantic Backend (Flask)
```bash
cd backend/api/semantic
python -m venv venv
.\venv\Scripts\Activate.ps1
pip install -r requirements.txt
python app.py
# Runs on http://localhost:5000
```

#### 3. Frontend (React)
```bash
cd frontend
npm install
npm start
# Runs on http://localhost:3000
```

### Or Use Helper Scripts
```bash
# Start all services at once
sh scripts/run.sh

# Stop all services
sh scripts/down.sh
```

---

## 📈 Performance Characteristics

### Lexical Search
- **Query Time**: < 50ms
- **Scalability**: Good for SQLite (< 100k records)
- **Bottleneck**: Database query performance
- **Optimization**: Add indexes on `verse` and `verseWithoutTashkeel`

### Semantic Search
- **Query Time**: 1-3 seconds
- **Scalability**: Limited by Word2Vec model size
- **Bottleneck**: Calculating similarity for 6,236 verses × query words
- **Optimization**: 
  - Cache results
  - Use GPU for model inference
  - Pre-compute embedding vectors

---

## 🎯 When to Use Each Search Type

### Use Lexical When:
- ✅ You remember exact Arabic words
- ✅ You want instant results
- ✅ You're searching for specific phrases
- ✅ Examples:
  - "الرحمن الرحيم"
  - "القيامة"
  - "الوضوء"

### Use Semantic When:
- ✅ You want concept-based results
- ✅ You're okay with slower responses
- ✅ You want more relevant/related verses
- ✅ Examples:
  - "prayer" (صلاة) → finds all worship-related verses
  - "charity" (الزكاة) → finds generosity-related verses
  - "God's mercy" → finds رحمة verses

---

## 🔮 Future Enhancements

### For Lexical Search
1. **Full-text search index** (FTS5 in SQLite)
2. **Advanced filtering** (by surah, juz, page)
3. **Search history** (session-based)
4. **Fuzzy matching** (typo tolerance)

### For Semantic Search
1. **Transformer models** (BERT-style for Arabic)
2. **Sentence embeddings** (better than word-level)
3. **Multi-lingual support** (English → Arabic semantic search)
4. **Real-time vector search** (use PostgreSQL + pgvector)

---

## 📚 Summary

This Quranic Search system demonstrates:
- ✅ **Dual-search architecture** (lexical + semantic)
- ✅ **Multi-backend setup** (Django + Flask)
- ✅ **ML integration** (Word2Vec embeddings)
- ✅ **Production-ready structure** (WSGI, static files, CORS)
- ✅ **Arabic NLP** (text preprocessing, tokenization)

**Perfect for learning**:
- Django REST API patterns
- Flask API development
- NLP and word embeddings
- React + Django/Flask integration
- Arabic text processing

---

**Ready to explore!** Start with the **TECHNICAL_ANALYSIS.md** for deep dive into the lexical search architecture, then use this document to understand the complete system.
