# 🏗️ Quranic Search Backend - Complete Technical Analysis

## 📋 Table of Contents
- [1. Architecture Overview](#1-architecture-overview)
- [2. Folder Structure & Components](#2-folder-structure--components)
- [3. Data Flow & Request/Response Cycle](#3-data-flow--requestresponse-cycle)
- [4. Key Design Patterns](#4-key-design-patterns)
- [5. Database Schema & Models](#5-database-schema--models)
- [6. Learning Roadmap](#6-learning-roadmap)
- [7. Migration Plan to TypeScript/Express/NestJS](#7-migration-plan-to-typescriptexpressnestjs)
- [8. Visual Diagrams](#8-visual-diagrams)

---

## 1. Architecture Overview

### What This Backend Does
The **Lexical Search API** is a Django-based REST API that provides keyword-based search functionality for the Holy Quran. It stores 6,236 verses (all verses in the Quran) and allows searching by exact keywords or phrases.

### Key Components

```
┌─────────────────────────────────────────────────────────────┐
│                    Client (React Frontend)                   │
└────────────────────────┬────────────────────────────────────┘
                         │ HTTP Requests (GET)
                         ↓
┌─────────────────────────────────────────────────────────────┐
│              Django REST Framework (DRF) Layer                │
│  • URL Routing (search/urls.py)                             │
│  • API Views (search/views.py)                              │
│  • Serializers (search/serializers.py)                      │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ↓
┌─────────────────────────────────────────────────────────────┐
│              Django ORM (Object-Relational Mapping)          │
│  • Models (search/models.py)                                │
│  • QuerySet Operations (filter, get_object_or_404)         │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ↓
┌─────────────────────────────────────────────────────────────┐
│                    SQLite3 Database                          │
│  • Verses Table (6,236 records)                             │
│  • Surah Table (114 Surahs)                                 │
└─────────────────────────────────────────────────────────────┘
```

### Technologies Stack

| Layer | Technology | Version | Purpose |
|-------|-----------|---------|---------|
| **Web Framework** | Django | 5.2.7 | Main backend framework |
| **API Framework** | Django REST Framework | 3.16.1 | RESTful API layer |
| **Database** | SQLite3 | - | Verse storage |
| **CORS** | django-cors-headers | 4.9.0 | Cross-origin requests |
| **Filtering** | django-filter | 25.2 | Query filtering |
| **Server** | Gunicorn | 23.0.0 | Production WSGI server |
| **Static Files** | WhiteNoise | 6.11.0 | Static file serving |

---

## 2. Folder Structure & Components

### Complete Directory Breakdown

```
backend/api/lexical/
├── api/                           # Django Project Configuration
│   ├── __init__.py               # Package initializer
│   ├── settings.py               # Main settings (DATABASES, INSTALLED_APPS, CORS, etc.)
│   ├── urls.py                   # Root URL configuration
│   ├── wsgi.py                   # WSGI entry point for production
│   └── asgi.py                   # ASGI entry point (not used in this project)
│
├── search/                       # Main Application ("search" app)
│   ├── models.py                 # Database models (Verses, Surah)
│   ├── views.py                  # API endpoint logic (search, get_surah, etc.)
│   ├── serializers.py            # Data serialization (JSON conversion)
│   ├── urls.py                   # App-level URL routing
│   ├── admin.py                  # Django admin configuration
│   ├── apps.py                   # App configuration
│   ├── tests.py                  # Unit tests (currently empty)
│   ├── migrations/               # Database migrations
│   ├── templates/                # HTML templates (API docs, 404)
│   └── static/                   # CSS files
│
├── db/                           # Database Files
│   ├── db.sqlite3               # Main Django database
│   ├── quran.db                  # Source Quran database
│   ├── verses.cvs                # CSV data export
│   └── new.py                    # Database migration script
│
├── venv/                         # Python Virtual Environment
├── manage.py                     # Django CLI utility
└── requirements.txt              # Python dependencies
```

### Detailed File Explanations

#### **api/settings.py** (Configuration Hub)
- **SECRET_KEY**: Security key (should be hidden in production)
- **DATABASES**: Defines two SQLite databases
  - `default`: Django system database
  - `quran`: Read-only Quran verse database
- **INSTALLED_APPS**: Lists all Django apps including "search"
- **MIDDLEWARE**: Request processing pipeline (CORS, auth, security)
- **CORS**: Allows all origins with GET-only methods

#### **search/models.py** (Data Layer)
Two main models:

1. **Verses Model**: Stores individual Quranic verses
   - `verse_pk`: Unique identifier (e.g., "S001V001")
   - `verse`: Full Arabic text with diacritics
   - `verseWithoutTashkeel`: Plain Arabic text (for searching)
   - `numberInQuran`: Global verse number (1-6236)
   - `numberInSurah`: Verse number within its Surah
   - Relations: FK to Surah, audio URLs, Quran division fields (juz, page, hizbQuarter)

2. **Surah Model**: Stores chapter information
   - `name`: Surah name with diacritics
   - `nameWithoutTashkeel`: Plain name

#### **search/views.py** (Business Logic)
5 main endpoints:

1. **`get_all()`**: Returns all verses (limited use)
2. **`search(words)`**: Case-insensitive keyword search
   - Searches in both `verse` and `verseWithoutTashkeel`
   - Uses Django Q objects for OR conditions
3. **`get_surah(surah_id)`**: Returns all verses in a surah
4. **`get_verse_in_surah(surah_id, verse_id)`**: Single verse by surah/verse
5. **`get_verse_in_quran(verse_id)`**: Single verse by global ID

#### **search/serializers.py** (Data Transformation)
- Converts Django model instances to JSON
- Uses `ModelSerializer` for automatic field mapping
- Returns structured data for frontend consumption

#### **search/urls.py** (Routing)
- Maps URL patterns to view functions
- Example: `"api/lexical/search/<str:words>"` → `search()`

---

## 3. Data Flow & Request/Response Cycle

### Detailed Request Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                    1. CLIENT REQUEST                            │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             │ GET /api/lexical/search/الله
                             │
                             ↓
┌─────────────────────────────────────────────────────────────────┐
│ 2. URL ROUTING (api/urls.py)                                    │
│    • Checks urlpatterns                                         │
│    • Matches "api/lexical/search/<words>"                       │
│    • Routes to search.urls                                      │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ↓
┌─────────────────────────────────────────────────────────────────┐
│ 3. URL ROUTING (search/urls.py)                                 │
│    • Matches path("api/lexical/search/<str:words>")             │
│    • Calls search() view with words="الله"                      │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ↓
┌─────────────────────────────────────────────────────────────────┐
│ 4. MIDDLEWARE PROCESSING (api/settings.py)                     │
│    • CORS Middleware: Checks origin allowance                  │
│    • Session Middleware: Manages session state                  │
│    • Common Middleware: URL rewriting, etc.                     │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ↓
┌─────────────────────────────────────────────────────────────────┐
│ 5. VIEW EXECUTION (search/views.py)                             │
│                                                                  │
│    @api_view(["GET"])                                           │
│    def search(request, words):                                  │
│        verses = Verses.objects.filter(                          │
│            Q(verse__icontains=words) |                          │
│            Q(verseWithoutTashkeel__icontains=words)            │
│        )                                                         │
│        data = VersesSerializers(verses, many=True).data         │
│        return Response({"length": len(data), "data": data})    │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ↓
┌─────────────────────────────────────────────────────────────────┐
│ 6. DATABASE QUERY (search/models.py → Django ORM)              │
│                                                                  │
│    Verses.objects.filter(...) converts to:                      │
│    SELECT * FROM search_verse                                   │
│    WHERE verse LIKE '%الله%'                                    │
│       OR verseWithoutTashkeel LIKE '%الله%';                   │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ↓
┌─────────────────────────────────────────────────────────────────┐
│ 7. SQLITE3 EXECUTION (db/quran.db)                              │
│    • Query execution                                            │
│    • Returns matching rows                                      │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ↓
┌─────────────────────────────────────────────────────────────────┐
│ 8. DATA SERIALIZATION (search/serializers.py)                   │
│                                                                  │
│    VersesSerializers converts:                                 │
│    • Django model → Python dict → JSON                          │
│    • Formats response structure                                 │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ↓
┌─────────────────────────────────────────────────────────────────┐
│ 9. RESPONSE (DRF Response Object)                               │
│                                                                  │
│    Returns JSON:                                                │
│    {                                                            │
│      "length": 125,                                             │
│      "data": [                                                  │
│        {                                                        │
│          "verse_pk": "S001V001",                                │
│          "verse": "بِسْمِ اللَّهِ الرَّحْمَٰنِ الرَّحِيمِ",     │
│          "surah": "الفاتحة",                                   │
│          ...                                                    │
│        }                                                        │
│      ]                                                          │
│    }                                                            │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ↓
┌─────────────────────────────────────────────────────────────────┐
│                   10. CLIENT RECEIVES JSON                      │
└─────────────────────────────────────────────────────────────────┘
```

### API Endpoints Summary

| Endpoint | Method | Description | Parameters |
|----------|--------|-------------|------------|
| `/api/lexical/all` | GET | Get all verses | None |
| `/api/lexical/search/<words>` | GET | Search verses by keywords | `words` (string) |
| `/api/lexical/surah/<surah_id>` | GET | Get all verses in a surah | `surah_id` (int) |
| `/api/lexical/verse-in-surah/<surah_id>/<verse_id>` | GET | Get specific verse | `surah_id`, `verse_id` (int) |
| `/api/lexical/verse-in-quran/<verse_id>` | GET | Get verse by global ID | `verse_id` (int) |

---

## 4. Key Design Patterns

### 1. **MVC/MVT Pattern** (Model-View-Template)
Django follows **MVT** (Model-View-Template):
- **Model**: Database schema (`search/models.py`)
- **View**: Business logic (`search/views.py`)
- **Template**: HTML rendering (`search/templates/`)

### 2. **Repository Pattern** (via Django ORM)
Models abstract database operations:
```python
# Instead of raw SQL, you use:
Verses.objects.filter(verse__icontains="الله")
```

### 3. **Serialization Pattern**
DRF serializers convert complex types to/from JSON:
```python
# Model → JSON
serializer = VersesSerializers(verse)
json_data = serializer.data
```

### 4. **Decorator Pattern**
`@api_view(["GET"])` decorators define allowed HTTP methods.

### 5. **URL Namespacing**
`app_name = "QuranLexicalSearchAPI"` prevents URL conflicts.

### 6. **QuerySet Lazy Evaluation**
Django queries execute only when data is needed:
```python
verses = Verses.objects.filter(...)  # Not executed yet
data = list(verses)  # Now it executes
```

---

## 5. Database Schema & Models

### Entity Relationship Diagram

```
┌─────────────────────────────────┐
│            SURAH                │
├─────────────────────────────────┤
│ PK  id (auto)                   │
│     name (varchar)              │
│     nameWithoutTashkeel (varchar)│
└───────────┬─────────────────────┘
            │ 1
            │
            │ M (ForeignKey)
            ↓
┌─────────────────────────────────┐
│            VERSES               │
├─────────────────────────────────┤
│ PK  id (auto)                   │
│ PK  verse_pk (varchar, unique) │
│     page (int)                  │
│     hizbQuarter (int)          │
│     juz (int)                   │
│ FK  surah → Surah.id           │
│     verse (varchar)             │
│     verseWithoutTashkeel (varchar)│
│     numberInSurah (int)         │
│     numberInQuran (int, unique) │
│     audio (URL)                 │
│     audio1 (URL, nullable)      │
│     audio2 (URL, nullable)      │
│     sajda (boolean)             │
└─────────────────────────────────┘
```

### Sample Query Execution

```python
# Model Definition (models.py)
class Verses(models.Model):
    verse_pk = models.CharField(max_length=8, unique=True)
    verse = models.CharField(max_length=5000)
    # ... other fields
    surah = models.ForeignKey(Surah, on_delete=models.CASCADE)

# Query in View (views.py)
verses = Verses.objects.filter(
    Q(verse__icontains="الله") | 
    Q(verseWithoutTashkeel__icontains="الله")
)

# Generated SQL (Django ORM)
# SELECT * FROM search_verse
# WHERE verse LIKE '%الله%'
#    OR verseWithoutTashkeel LIKE '%الله%'
# LIMIT 1000;

# Serialization (serializers.py)
data = VersesSerializers(verses, many=True).data

# Response JSON
{
  "length": 125,
  "data": [
    {
      "verse_pk": "S002V255",
      "verse": "اللَّهُ لَا إِلَٰهَ إِلَّا هُوَ الْحَيُّ الْقَيُّومُ...",
      "verseWithoutTashkeel": "الله لا إله إلا هو الحي القيوم...",
      "numberInSurah": 255,
      "numberInQuran": 286,
      "surah": "البقرة",
      "page": 42,
      "juz": 3,
      "hizbQuarter": 10,
      "audio": "https://...",
      "sajda": false
    }
  ]
}
```

---

## 6. Learning Roadmap

### Core Concepts to Master

#### **1. Django Fundamentals** 🐍
**Why**: This is a Django-based project.

**Topics**:
- Django project vs. app structure
- Models and migrations
- Views (function-based and class-based)
- URL routing and URLconfs
- Django templates
- Static files management

**Resources**:
- Django Official Tutorial: https://docs.djangoproject.com/en/5.2/
- "Two Scoops of Django" (book)

**How it applies here**:
- `api/` = Django project
- `search/` = Django app
- Models define schema → migrate to create tables

#### **2. Django REST Framework (DRF)** 🌐
**Why**: All API endpoints use DRF.

**Topics**:
- API Views (`@api_view`, `APIView`, `ViewSet`)
- Serializers (conversion to JSON)
- Request/Response objects
- Permissions and authentication
- Pagination and filtering

**Resources**:
- DRF Tutorial: https://www.django-rest-framework.org/tutorial/

**How it applies here**:
- `@api_view(["GET"])` = function-based API view
- `VersesSerializers` = converts model → JSON
- `Response()` = returns JSON with headers

#### **3. Django ORM (Object-Relational Mapping)** 🗄️
**Why**: Database queries use ORM.

**Topics**:
- QuerySets and lazy evaluation
- Filtering (`filter()`, `exclude()`)
- Query operations (`Q`, `F`)
- Aggregations (`count()`, `exists()`)
- Raw SQL vs. ORM trade-offs

**Resources**:
- Django ORM Docs: https://docs.djangoproject.com/en/5.2/topics/db/

**How it applies here**:
```python
# Filtering with Q objects
verses = Verses.objects.filter(
    Q(verse__icontains="الله") | 
    Q(verseWithoutTashkeel__icontains="الله")
)
```

#### **4. SQLite3 Database** 📊
**Why**: Data is stored in SQLite.

**Topics**:
- SQL basics (SELECT, WHERE, JOIN, LIKE)
- Indexes and query optimization
- Foreign keys and relationships
- Database migrations
- SQLite CLI tools

**Resources**:
- SQLite Tutorial: https://www.sqlitetutorial.net/

**How it applies here**:
- `verse_pk` is indexed for fast lookups
- Foreign key `surah` links Verses → Surah
- Queries search both with/without diacritics

#### **5. REST API Design** 🔌
**Why**: This is a RESTful API.

**Topics**:
- HTTP methods (GET, POST, PUT, DELETE)
- URL naming conventions
- Status codes (200, 404, 500)
- Request/Response format
- API documentation

**Resources**:
- RESTful API Design: https://restfulapi.net/

**How it applies here**:
- All endpoints use GET (read-only)
- URLs are RESTful: `/api/lexical/verse-in-surah/<id>/<id>`
- Consistent JSON response format

#### **6. CORS & Security** 🔒
**Why**: Frontend makes cross-origin requests.

**Topics**:
- Cross-Origin Resource Sharing (CORS)
- Same-origin policy
- CORS headers (`Access-Control-Allow-Origin`)
- Security best practices (SECRET_KEY, DEBUG)

**Resources**:
- MDN CORS: https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS

**How it applies here**:
- `CORS_ALLOW_ALL_ORIGINS = True` (development)
- `CORS_ALLOW_METHODS = ["get"]` (GET-only API)
- Need CORS headers for React frontend

#### **7. Python Virtual Environments** 🐍
**Why**: Isolates dependencies.

**Topics**:
- `venv` module
- `requirements.txt`
- Dependency management
- Activating virtual environments

**Resources**:
- Python venv: https://docs.python.org/3/library/venv.html

**How it applies here**:
- `venv/` folder contains isolated packages
- `requirements.txt` lists all dependencies
- Activate before running: `.\venv\Scripts\Activate.ps1`

#### **8. WSGI & Deployment** 🚀
**Why**: Production needs a WSGI server.

**Topics**:
- WSGI (Web Server Gateway Interface)
- Gunicorn, uWSGI
- Deployment (Heroku, AWS, Docker)
- Environment variables
- Static files serving

**Resources**:
- WSGI Tutorial: https://www.fullstackpython.com/wsgi-servers.html

**How it applies here**:
- `wsgi.py` = entry point for Gunicorn
- `gunicorn` in requirements.txt
- `whitenoise` serves static files

#### **9. Error Handling** ⚠️
**Why**: Robust APIs handle errors gracefully.

**Topics**:
- HTTP status codes
- Exception handling in Django
- `get_object_or_404()`
- Custom error responses

**How it applies here**:
```python
verse = get_object_or_404(Verses, verse_pk=verse_pk)
# Returns 404 if not found
```

#### **10. Testing** 🧪
**Topics**:
- Unit tests (Django `TestCase`)
- API testing (`django-rest-framework-test`)
- Testing best practices

**Resources**:
- Django Testing: https://docs.djangoproject.com/en/5.2/topics/testing/

**Note**: This project has `tests.py` but no tests yet.

---

## 7. Migration Plan to TypeScript/Express/NestJS

### Overview
This section explains how to migrate the Django backend to TypeScript using either Express.js (simpler) or NestJS (enterprise-grade with decorators similar to Spring Boot).

### Migration Options Comparison

| Feature | Django (Current) | Express.js | NestJS |
|--------|------------------|------------|--------|
| **Complexity** | Medium | Low | High |
| **Learning Curve** | Moderate | Steep initially | Steep |
| **Type Safety** | No (Django) | Yes (TypeScript) | Yes (TypeScript) |
| **Decorators** | Limited | Limited | Extensive |
| **Dependency Injection** | No | Manual | Built-in |
| **Testing** | Django TestCase | Jest | Jest + Supertest |
| **ORM** | Django ORM | Prisma/TypeORM | TypeORM/Prisma |
| **REST Framework** | DRF | Express Router | NestJS Controllers |
| **Migration Effort** | - | Medium | High |
| **Production Ready** | ✅ Yes | ✅ Yes (with setup) | ✅ Yes |

---

## Option 1: Express.js Migration

### Step-by-Step Plan

#### **Phase 1: Project Setup** (Day 1)

```bash
# Initialize project
mkdir quranic-search-express
cd quranic-search-express
npm init -y

# Install dependencies
npm install express cors dotenv
npm install -D typescript @types/node @types/express @types/cors ts-node-dev

# Initialize TypeScript
npx tsc --init
```

**Created File Structure**:
```
quranic-search-express/
├── src/
│   ├── models/          # Database models
│   ├── controllers/     # Request handlers
│   ├── routes/          # URL routing
│   ├── middleware/      # CORS, auth, etc.
│   ├── utils/           # Helper functions
│   └── app.ts           # Express app setup
├── tsconfig.json        # TypeScript config
├── package.json
└── .env                  # Environment variables
```

#### **Phase 2: Database Setup** (Day 2-3)

**Option A: Using Prisma** (Recommended)

```bash
npm install @prisma/client
npm install -D prisma
npx prisma init
```

**Define Schema** (`prisma/schema.prisma`):
```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "sqlite"
  url      = "file:./quran.db"
}

model Verse {
  id                      String   @id @default(autoincrement())
  versePk                 String   @unique @map("verse_pk")
  page                    Int
  hizbQuarter             Int
  juz                     Int
  surah                   String
  verse                   String
  verseWithoutTashkeel    String
  numberInSurah           Int
  numberInQuran           Int       @unique
  audio                   String
  audio1                  String?
  audio2                  String?
  sajda                   Boolean

  @@map("verses")
}
```

Generate client: `npx prisma generate`

**Option B: Using TypeORM** (Alternative)

```bash
npm install typeorm reflect-metadata sqlite3
```

**Create Entity** (`src/models/Verse.ts`):
```typescript
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm';

@Entity('verses')
export class Verse {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ unique: true })
  versePk: string;

  @Column()
  page: number;

  @Column()
  hizbQuarter: number;

  @Column()
  juz: number;

  @Column()
  verse: string;

  @Column()
  verseWithoutTashkeel: string;

  @Column()
  numberInSurah: number;

  @Column({ unique: true })
  numberInQuran: number;

  @Column()
  audio: string;

  @Column({ nullable: true })
  audio1?: string;

  @Column({ nullable: true })
  audio2?: string;

  @Column()
  sajda: boolean;
}
```

#### **Phase 3: Create Controllers** (Day 4-5)

**File**: `src/controllers/verseController.ts`

```typescript
import { Request, Response } from 'express';
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

// Map Django views to Express controllers

export const getAll = async (req: Request, res: Response) => {
  try {
    const verses = await prisma.verse.findMany();
    res.json({ length: verses.length, data: verses });
  } catch (error) {
    res.status(500).json({ error: 'Internal server error' });
  }
};

export const search = async (req: Request, res: Response) => {
  try {
    const { words } = req.params;
    
    const verses = await prisma.verse.findMany({
      where: {
        OR: [
          { verse: { contains: words, mode: 'insensitive' } },
          { verseWithoutTashkeel: { contains: words, mode: 'insensitive' } }
        ]
      }
    });

    res.json({ length: verses.length, data: verses });
  } catch (error) {
    res.status(500).json({ error: 'Internal server error' });
  }
};

export const getSurah = async (req: Request, res: Response) => {
  try {
    const { surahId } = req.params;
    const surahPk = `S${String(surahId).padStart(3, '0')}`;
    
    const verses = await prisma.verse.findMany({
      where: { versePk: { startsWith: surahPk } }
    });

    res.json({ length: verses.length, data: verses });
  } catch (error) {
    res.status(500).json({ error: 'Internal server error' });
  }
};

export const getVerseInSurah = async (req: Request, res: Response) => {
  try {
    const { surahId, verseId } = req.params;
    const versePk = `S${String(surahId).padStart(3, '0')}V${String(verseId).padStart(3, '0')}`;
    
    const verse = await prisma.verse.findUnique({
      where: { versePk }
    });

    if (!verse) {
      return res.status(404).json({ error: 'Verse not found' });
    }

    res.json({ data: verse });
  } catch (error) {
    res.status(500).json({ error: 'Internal server error' });
  }
};

export const getVerseInQuran = async (req: Request, res: Response) => {
  try {
    const { verseId } = req.params;
    
    const verse = await prisma.verse.findUnique({
      where: { numberInQuran: parseInt(verseId) }
    });

    if (!verse) {
      return res.status(404).json({ error: 'Verse not found' });
    }

    res.json({ data: verse });
  } catch (error) {
    res.status(500).json({ error: 'Internal server error' });
  }
};
```

#### **Phase 4: Create Routes** (Day 6)

**File**: `src/routes/verseRoutes.ts`

```typescript
import { Router } from 'express';
import * as verseController from '../controllers/verseController';

const router = Router();

// Map Django URLs to Express routes
router.get('/all', verseController.getAll);
router.get('/search/:words', verseController.search);
router.get('/surah/:surahId', verseController.getSurah);
router.get('/verse-in-surah/:surahId/:verseId', verseController.getVerseInSurah);
router.get('/verse-in-quran/:verseId', verseController.getVerseInQuran);

export default router;
```

#### **Phase 5: Setup Express App** (Day 7)

**File**: `src/app.ts`

```typescript
import express from 'express';
import cors from 'cors';
import dotenv from 'dotenv';
import verseRoutes from './routes/verseRoutes';

dotenv.config();

const app = express();
const PORT = process.env.PORT || 8000;

// Middleware
app.use(cors());
app.use(express.json());

// Routes
app.use('/api/lexical', verseRoutes);

// Error handling middleware
app.use((err: Error, req: express.Request, res: express.Response, next: express.NextFunction) => {
  console.error(err.stack);
  res.status(500).json({ error: 'Something went wrong!' });
});

// Start server
app.listen(PORT, () => {
  console.log(`Server is running on http://localhost:${PORT}`);
});

export default app;
```

#### **Phase 6: Testing** (Day 8)

```bash
# Test endpoints
curl http://localhost:8000/api/lexical/search/الله
```

---

## Option 2: NestJS Migration

### Why NestJS?
- Built-in dependency injection
- Modular architecture (similar to Angular)
- Decorator-based (cleaner than Express)
- Built-in validation, guards, pipes
- TypeScript-first

### Step-by-Step Plan

#### **Phase 1: Project Setup**

```bash
npm i -g @nestjs/cli
nest new quranic-search-nest
cd quranic-search-nest
npm install @nestjs/typeorm typeorm sqlite3
npm install @nestjs/config class-validator class-transformer
```

#### **Phase 2: Project Structure**

```
src/
├── verses/
│   ├── verses.module.ts      # Module
│   ├── verses.controller.ts  # Routes (URLs)
│   ├── verses.service.ts     # Business logic (views)
│   └── entities/
│       └── verse.entity.ts   # Model (Django models)
├── app.module.ts             # Root module
└── main.ts                   # Entry point
```

#### **Phase 3: Create Entity**

**File**: `src/verses/entities/verse.entity.ts`

```typescript
import { Entity, PrimaryGeneratedColumn, Column, Index } from 'typeorm';

@Entity('verses')
export class Verse {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ unique: true })
  @Index()
  versePk: string;

  @Column()
  page: number;

  @Column()
  hizbQuarter: number;

  @Column()
  juz: number;

  @Column({ type: 'text' })
  verse: string;

  @Column({ type: 'text' })
  verseWithoutTashkeel: string;

  @Column()
  numberInSurah: number;

  @Column({ unique: true })
  numberInQuran: number;

  @Column()
  audio: string;

  @Column({ nullable: true })
  audio1?: string;

  @Column({ nullable: true })
  audio2?: string;

  @Column()
  sajda: boolean;
}
```

#### **Phase 4: Create Service** (Business Logic)

**File**: `src/verses/verses.service.ts`

```typescript
import { Injectable, NotFoundException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { Verse } from './entities/verse.entity';

@Injectable()
export class VersesService {
  constructor(
    @InjectRepository(Verse)
    private verseRepository: Repository<Verse>,
  ) {}

  async findAll() {
    return this.verseRepository.find();
  }

  async search(words: string) {
    return this.verseRepository
      .createQueryBuilder('verse')
      .where('verse.verse LIKE :words', { words: `%${words}%` })
      .orWhere('verse.verseWithoutTashkeel LIKE :words', { words: `%${words}%` })
      .getMany();
  }

  async findBySurah(surahId: number) {
    const surahPk = `S${String(surahId).padStart(3, '0')}`;
    return this.verseRepository
      .createQueryBuilder('verse')
      .where('verse.versePk LIKE :surahPk', { surahPk: `${surahPk}%` })
      .getMany();
  }

  async findVerseInSurah(surahId: number, verseId: number) {
    const versePk = `S${String(surahId).padStart(3, '0')}V${String(verseId).padStart(3, '0')}`;
    const verse = await this.verseRepository.findOne({ where: { versePk } });
    
    if (!verse) {
      throw new NotFoundException('Verse not found');
    }
    
    return verse;
  }

  async findVerseInQuran(verseId: number) {
    const verse = await this.verseRepository.findOne({ where: { numberInQuran: verseId } });
    
    if (!verse) {
      throw new NotFoundException('Verse not found');
    }
    
    return verse;
  }
}
```

#### **Phase 5: Create Controller** (URL Routing)

**File**: `src/verses/verses.controller.ts`

```typescript
import { Controller, Get, Param, ParseIntPipe } from '@nestjs/common';
import { VersesService } from './verses.service';

@Controller('api/lexical')
export class VersesController {
  constructor(private readonly versesService: VersesService) {}

  @Get('all')
  async getAll() {
    const data = await this.versesService.findAll();
    return { length: data.length, data };
  }

  @Get('search/:words')
  async search(@Param('words') words: string) {
    const data = await this.versesService.search(words);
    return { length: data.length, data };
  }

  @Get('surah/:surahId')
  async getSurah(@Param('surahId', ParseIntPipe) surahId: number) {
    const data = await this.versesService.findBySurah(surahId);
    return { length: data.length, data };
  }

  @Get('verse-in-surah/:surahId/:verseId')
  async getVerseInSurah(
    @Param('surahId', ParseIntPipe) surahId: number,
    @Param('verseId', ParseIntPipe) verseId: number,
  ) {
    const data = await this.versesService.findVerseInSurah(surahId, verseId);
    return { data };
  }

  @Get('verse-in-quran/:verseId')
  async getVerseInQuran(@Param('verseId', ParseIntPipe) verseId: number) {
    const data = await this.versesService.findVerseInQuran(verseId);
    return { data };
  }
}
```

#### **Phase 6: Setup Module**

**File**: `src/verses/verses.module.ts`

```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { VersesController } from './verses.controller';
import { VersesService } from './verses.service';
import { Verse } from './entities/verse.entity';

@Module({
  imports: [TypeOrmModule.forFeature([Verse])],
  controllers: [VersesController],
  providers: [VersesService],
})
export class VersesModule {}
```

**File**: `src/app.module.ts`

```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { VersesModule } from './verses/verses.module';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'sqlite',
      database: 'quran.db',
      entities: [__dirname + '/**/*.entity{.ts,.js}'],
      synchronize: true, // Only for development
    }),
    VersesModule,
  ],
})
export class AppModule {}
```

#### **Phase 7: Run**

```bash
npm run start:dev
```

---

### Comparison Table: Django → Express/NestJS

| Django Component | Express.js Equivalent | NestJS Equivalent |
|------------------|------------------------|-------------------|
| `manage.py` | `package.json` scripts | `nest start` |
| `settings.py` | `.env` + config files | `app.module.ts` + `.env` |
| `models.py` | Prisma schema / TypeORM entities | `entities/` folder |
| `views.py` | `controllers/` folder | `*.service.ts` + `*.controller.ts` |
| `urls.py` | `routes/` folder | `*.controller.ts` decorators |
| `serializers.py` | Manual JSON mapping | DTOs (Data Transfer Objects) |
| `INSTALLED_APPS` | Package dependencies | `*.module.ts` imports |
| Django ORM | Prisma Client / TypeORM | TypeORM (built-in) |
| `@api_view` | Route handlers | `@Controller()` decorators |
| `Response()` | `res.json()` | Automatic JSON (DRF-like) |
| Middleware | Express middleware | `@Injectable()` providers |
| `get_object_or_404()` | Manual if/else | `NotFoundException` |

---

## 8. Visual Diagrams

### Complete System Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                        CLIENT LAYER                                 │
│                        (React Frontend)                             │
│  ┌──────────────────────────────────────────────────────────┐      │
│  │ Components: HomeForm, ResultsForm, Verse, Navbar        │      │
│  └────────────────────────────┬─────────────────────────────┘      │
└────────────────────────────────┼────────────────────────────────────┘
                                 │ HTTP/HTTPS
                                 │ GET /api/lexical/search/الله
                                 ↓
┌─────────────────────────────────────────────────────────────────────┐
│                        API LAYER                                    │
│                     (Django REST API)                               │
│  ┌──────────────────────────────────────────────────────────┐      │
│  │ URL Routing                                                │      │
│  │ • /api/lexical/all                                        │      │
│  │ • /api/lexical/search/<words>                            │      │
│  │ • /api/lexical/surah/<id>                                │      │
│  │ • /api/lexical/verse-in-surah/<id>/<id>                 │      │
│  │ • /api/lexical/verse-in-quran/<id>                       │      │
│  └────────────────────────┬─────────────────────────────────┘      │
│                           │                                         │
│  ┌────────────────────────▼─────────────────────────────────┐      │
│  │ Views (Controllers)                                       │      │
│  │ • search()         ← Search by keywords                  │      │
│  │ • get_all()        ← Get all verses                      │      │
│  │ • get_surah()      ← Get surah verses                    │      │
│  │ • get_verse_...()  ← Get specific verses                 │      │
│  └────────────────────────┬─────────────────────────────────┘      │
│                           │                                         │
│  ┌────────────────────────▼─────────────────────────────────┐      │
│  │ Serializers (Data Transformation)                        │      │
│  │ • VersesSerializers → JSON conversion                    │      │
│  └────────────────────────┬─────────────────────────────────┘      │
└────────────────────────────┼────────────────────────────────────────┘
                             │
                             ↓
┌─────────────────────────────────────────────────────────────────────┐
│                       DATA LAYER                                    │
│                      (Django ORM)                                   │
│  ┌──────────────────────────────────────────────────────────┐      │
│  │ Models                                                    │      │
│  │ • Verses: verse_pk, verse, verseWithoutTashkeel         │      │
│  │ • Surah: name, nameWithoutTashkeel                       │      │
│  │ • Relations: ForeignKey (surah)                          │      │
│  └────────────────────────┬─────────────────────────────────┘      │
│                           │                                         │
│  ┌────────────────────────▼─────────────────────────────────┐      │
│  │ QuerySet Operations                                       │      │
│  │ • Verses.objects.filter()                                 │      │
│  │ • Q objects for OR conditions                             │      │
│  │ • get_object_or_404() for 404 handling                   │      │
│  └────────────────────────┬─────────────────────────────────┘      │
└────────────────────────────┼────────────────────────────────────────┘
                             │ SQL Queries
                             ↓
┌─────────────────────────────────────────────────────────────────────┐
│                    DATABASE LAYER                                    │
│                     (SQLite3)                                        │
│  ┌──────────────────────────────────────────────────────────┐      │
│  │ Tables                                                     │      │
│  │ • search_verse (6,236 rows)                               │      │
│  │   - id, verse_pk (PK), page, hizbQuarter, juz           │      │
│  │   - verse, verseWithoutTashkeel                          │      │
│  │   - numberInSurah, numberInQuran (unique)               │      │
│  │   - audio, audio1, audio2                               │      │
│  │   - sajda                                                │      │
│  │                                                           │      │
│  │ • search_surah (114 rows)                                │      │
│  │   - id (PK)                                              │      │
│  │   - name, nameWithoutTashkeel                            │      │
│  └──────────────────────────────────────────────────────────┘      │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────┐      │
│  │ Sample Query                                               │      │
│  │ SELECT * FROM search_verse                                │      │
│  │ WHERE verse LIKE '%الله%'                                │      │
│  │    OR verseWithoutTashkeel LIKE '%الله%'                 │      │
│  └──────────────────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────────────┘
```

### Request/Response Flow Diagram

```
┌─────────────────────┐
│   User Types Query   │
│   "الله"             │
└──────────┬──────────┘
           │
           ↓
┌───────────────────────────────────────────────────┐
│ React Frontend                                      │
│ • User enters query                                │
│ • Triggers API call                                │
└──────────┬──────────────────────────────────────────┘
           │
           │ fetch('/api/lexical/search/الله')
           ↓
┌───────────────────────────────────────────────────┐
│ Django URL Routing (api/urls.py)                  │
│ • Matches pattern                                  │
│ • Routes to search.urls                            │
└──────────┬──────────────────────────────────────────┘
           │
           ↓
┌───────────────────────────────────────────────────┐
│ App URL Routing (search/urls.py)                  │
│ • path("api/lexical/search/<str:words>")          │
│ • Calls search() view                             │
└──────────┬──────────────────────────────────────────┘
           │
           ↓
┌───────────────────────────────────────────────────┐
│ Middleware Pipeline                                │
│ • CORS: Checks origin                             │
│ • Common: URL processing                           │
│ • Session: Session management                      │
└──────────┬──────────────────────────────────────────┘
           │
           ↓
┌───────────────────────────────────────────────────┐
│ View Function (search/views.py)                   │
│                                                    │
│ @api_view(["GET"])                                │
│ def search(request, words):                       │
│   verses = Verses.objects.filter(                 │
│     Q(verse__icontains=words) |                   │
│     Q(verseWithoutTashkeel__icontains=words)     │
│   )                                                │
│   data = VersesSerializers(verses, many=True).data│
│   return Response({"length": len(data), "data": data})│
└──────────┬──────────────────────────────────────────┘
           │
           ↓
┌───────────────────────────────────────────────────┐
│ Django ORM QuerySet                                │
│ • Converts Python to SQL                          │
│ • Executes: SELECT * FROM search_verse             │
│   WHERE verse LIKE '%الله%'                       │
│   OR verseWithoutTashkeel LIKE '%الله%'           │
└──────────┬──────────────────────────────────────────┘
           │
           ↓
┌───────────────────────────────────────────────────┐
│ SQLite3 Database                                   │
│ • Executes SQL query                              │
│ • Returns matching rows                           │
│ • Example result: 125 verses                      │
└──────────┬──────────────────────────────────────────┘
           │
           ↓
┌───────────────────────────────────────────────────┐
│ Serializer (VersesSerializers)                    │
│ • Converts Django model → Python dict              │
│ • Django REST framework converts dict → JSON      │
└──────────┬──────────────────────────────────────────┘
           │
           ↓
┌───────────────────────────────────────────────────┐
│ JSON Response                                      │
│ {                                                  │
│   "length": 125,                                   │
│   "data": [                                        │
│     {                                              │
│       "verse_pk": "S002V255",                     │
│       "verse": "اللَّهُ لَا...",                 │
│       "surah": "البقرة",                          │
│       "numberInQuran": 286,                        │
│       ...                                          │
│     }                                              │
│   ]                                                │
│ }                                                  │
└──────────┬──────────────────────────────────────────┘
           │
           ↓
┌───────────────────────────────────────────────────┐
│ React Frontend                                     │
│ • Receives JSON                                    │
│ • Renders results                                  │
│ • Displays verses in UI                           │
└────────────────────────────────────────────────────┘
```

### Database Relationships

```
┌─────────────────────────┐
│       SURAH             │
│  (Chapter/114 records)  │
├─────────────────────────┤
│ PK id                   │
│    name                 │
│    nameWithoutTashkeel  │
└──────────┬──────────────┘
           │
           │ 1:N (ForeignKey)
           │
           ↓
┌───────────────────────────────────────┐
│            VERSES                      │
│       (6,236 verse records)            │
├───────────────────────────────────────┤
│ PK    id (auto)                        │
│ PK/UK verse_pk (S001V001)              │
│      page                              │
│      hizbQuarter                      │
│      juz                               │
│ FK    surah → Surah.id                │
│      verse (with diacritics)          │
│      verseWithoutTashkeel (plain)     │
│      numberInSurah                    │
│ UK    numberInQuran (1-6236)         │
│      audio                            │
│      audio1                           │
│      audio2                           │
│      sajda                            │
└───────────────────────────────────────┘

Relationship Type: One Surah → Many Verses
Example: Surah 1 (Al-Fatiha) → 7 verses (S001V001 to S001V007)
```

---

## Summary & Next Steps

### What You've Learned
1. ✅ Django MVT architecture (Model-View-Template)
2. ✅ DRF for REST APIs
3. ✅ Django ORM query patterns
4. ✅ SQLite database structure
5. ✅ API endpoint design
6. ✅ Migration path to TypeScript

### Recommended Learning Path
1. **Week 1-2**: Master Django basics (tutorial, create your own app)
2. **Week 3**: Learn DRF (build a simple API)
3. **Week 4**: Study database design and SQL
4. **Week 5-6**: Migrate this project to Express.js or NestJS
5. **Week 7+**: Advanced topics (authentication, testing, deployment)

### Migration Decision
- **Choose Express.js if**: You want simplicity and fast migration
- **Choose NestJS if**: You want enterprise patterns and TypeScript best practices

---

**📧 Questions?** Feel free to ask about any part of this analysis!
