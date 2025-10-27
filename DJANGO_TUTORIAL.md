# Django Lexical Search Tutorial

## Table of Contents
1. [Project Overview](#project-overview)
2. [Django Concepts Explained](#django-concepts-explained)
3. [File-by-File Breakdown](#file-by-file-breakdown)
4. [Search System Analysis](#search-system-analysis)
5. [How to Extend](#how-to-extend)

---

## Project Overview

### What This Project Does

This is a **REST API** that allows users to search through the Holy Quran (6,236 verses) using **lexical (keyword-based) search**. Users can:

- Search for verses containing specific Arabic words
- Retrieve all verses from a specific Surah
- Get individual verses by various identifiers
- Access verse text with or without Arabic diacritics (Tashkeel)

### Architecture Flow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                        REQUEST FLOW                              │
└─────────────────────────────────────────────────────────────────┘

1. CLIENT SENDS REQUEST
   curl http://localhost:8000/api/lexical/search/الله
                            ↓
2. DJANGO URL ROUTER (urls.py)
   - Matches URL pattern to view function
   - Extracts parameters (e.g., "الله")
                            ↓
3. VIEW FUNCTION (views.py)
   - Receives request object
   - Queries database using Django ORM
   - Filters verses matching search terms
                            ↓
4. QUERY DATABASE (models.py)
   - Django ORM translates to SQL
   - SQLite executes query
   - Returns QuerySet (list of Verses objects)
                            ↓
5. SERIALIZE DATA (serializers.py)
   - Convert Verses objects → Python dictionaries
   - Convert Python dictionaries → JSON
                            ↓
6. RETURN RESPONSE
   Returns JSON: {"length": 120, "data": [...]}
```

### Django Project Structure

```
backend/api/lexical/
├── api/                      # Django project configuration
│   ├── settings.py          # All app settings (database, apps, etc.)
│   ├── urls.py              # Root URL configuration
│   └── wsgi.py              # Web Server Gateway Interface
├── search/                   # Main application (your Django app)
│   ├── models.py            # Database table definitions
│   ├── views.py             # Request handling logic
│   ├── serializers.py      # JSON conversion logic
│   ├── urls.py              # URL routing for this app
│   ├── admin.py             # Admin interface customization
│   ├── apps.py              # App configuration
│   └── migrations/          # Database migration files
└── db.sqlite3               # SQLite database file
```

---

## Django Concepts Explained

### What is Django?

**Django** is a Python web framework that follows the **MVC** (Model-View-Controller) pattern, but Django calls it **MVT** (Model-View-Template).

#### Core Concepts:

1. **ORM (Object-Relational Mapping)**: Write Python code instead of SQL
   ```python
   # Instead of: SELECT * FROM verses WHERE verse LIKE '%الله%'
   Verses.objects.filter(verse__icontains="الله")
   ```

2. **Models**: Define database tables using Python classes
3. **Views**: Handle HTTP requests and return responses
4. **URLs**: Route HTTP requests to appropriate view functions
5. **Migrations**: Track database schema changes over time

### Django vs. Plain Python

```python
# WITHOUT DJANGO (plain Python + SQL):
import sqlite3
conn = sqlite3.connect('quran.db')
cursor = conn.cursor()
cursor.execute("SELECT * FROM verses WHERE verse LIKE ?", ('%الله%',))
results = cursor.fetchall()
conn.close()

# WITH DJANGO:
from .models import Verses
results = Verses.objects.filter(verse__icontains="الله")
```

**Benefits:**
- No SQL to write
- Automatic database operations
- Database-agnostic (can switch from SQLite to PostgreSQL easily)
- Built-in admin interface
- Security features (SQL injection prevention)

---

## File-by-File Breakdown

### 1. __init__.py

**Purpose**: Makes `search/` directory a Python package

```python
# Empty file, but very important!
```

**Explanation:**
- In Python, any folder containing an `__init__.py` becomes a **package**
- This allows you to import modules: `from search.models import Verses`
- Without it, Python won't recognize the directory as importable
- Django convention: Every Django app has this file

**Real-world analogy**: Like labeling a folder "Important Documents" - without the label, you can't find it in your filing system.

---

### 2. models.py - Database Structure

This file defines **what data you store** and **how it's organized**.

#### Understanding Models

A Django **Model** = **Database Table**

Each class becomes a table, each field becomes a column.

#### Verse Model Breakdown

```python
class Verses(models.Model):  # ← Inherits from Django's Model class
```

**Line-by-Line Explanation:**

**Lines 1-3: Imports**
```python
from django.db import models
```
- `models` provides field types (CharField, IntegerField, ForeignKey, etc.)
- Think of it as: "Give me building blocks for my database structure"

**Lines 6-53: Verses Model**

```python
verse_pk = models.CharField(
    max_length=8,                  # Maximum 8 characters
    null=False,                    # Cannot be NULL in database
    blank=False,                   # Cannot be empty in forms
    unique=True,                   # Must be unique (like primary key)
    verbose_name="مفتاح الآية"      # Arabic label for admin interface
)
```

**Field Types Explained:**

1. **CharField** - Text up to max_length
2. **PositiveIntegerField** - Only positive numbers (0, 1, 2...)
3. **ForeignKey** - Link to another table (Surah table)
4. **URLField** - Validates it's a URL
5. **BooleanField** - True/False

**Critical Concept: ForeignKey**

```python
surah = models.ForeignKey(
    "Surah",                      # References the Surah table
    on_delete=models.CASCADE,    # If Surah deleted, delete its verses
    null=True,                   # Can be NULL (optional)
    blank=True,                  # Can be empty in forms
    related_name="surah",       # How to access verses from Surah
    verbose_name="السورة"        # Arabic label
)
```

**What this means in English:**
- Each verse belongs to ONE Surah
- Each Surah can have MANY verses
- If you delete a Surah, all its verses are deleted too

**Database View:**
```
VERSE Table:                  SURAH Table:
┌──────────────────┐         ┌────────────────┐
│ verse_pk         │         │ id (PK)        │
│ surah_id ────────┼────────→│ name            │
│ verse            │         └────────────────┘
│ verseWithout...  │
└──────────────────┘
```

**Lines 46-49: Meta Class**
```python
class Meta:
    ordering = ["id"]              # Default sort by ID
    verbose_name = "آية"            # Singular name (admin)
    verbose_name_plural = "الآيات"  # Plural name (admin)
```

**Lines 51-52: __str__ Method**
```python
def __str__(self):
    return self.verse[:50]  # Show first 50 chars in admin
```

#### Surah Model Breakdown

```python
class Surah(models.Model):
    name = models.CharField(max_length=50, verbose_name="اسم السورة")
    nameWithoutTashkeel = models.CharField(
        max_length=30, 
        verbose_name="اسم السورة بدون تشكيل"
    )
```

**Purpose:** Store Surah information (114 Surahs in Quran)

**Fields:**
- `name`: Surah name WITH Arabic diacritics (e.g., "الفاتحة")
- `nameWithoutTashkeel`: Surah name WITHOUT diacritics (e.g., "الفاتحة")

**Why two fields?** 
- Users might search "الفاتحة" or "الفاتحة"
- Searching without diacritics is easier for users

---

### 3. serializers.py - JSON Conversion

#### What is Serialization?

**Serialization** = Converting Python objects → JSON

**Example:**
```python
# Python object (Django model instance)
verse = Verses.objects.get(verse_pk="S001V001")
# verse is a Python object with attributes

# Serialized to JSON
{
  "verse_pk": "S001V001",
  "page": 1,
  "juz": 1,
  "verse": "بِسْمِ اللَّهِ...",
  ...
}
```

#### Code Breakdown

**Line 1: Import DRF Serializers**
```python
from rest_framework import serializers
```
- DRF = Django REST Framework
- Provides tools for building APIs

**Line 6-23: VersesSerializers Class**

```python
class VersesSerializers(serializers.ModelSerializer):
```

**Inheritance Explained:**
- `serializers.ModelSerializer` is a pre-built class
- Automatically creates serializer from a Model
- You just specify which fields to include

**Lines 7-8: Custom Field Serialization**
```python
# surah = serializers.SerializerMethodField(source='get_surah_name')
surah = serializers.StringRelatedField()
```

**What this does:**
- `surah` is a ForeignKey to the Surah model
- Instead of returning `{"surah": 1}` (the ID)
- Returns `{"surah": "الفاتحة"}` (the name string)

**Lines 10-22: Fields to Include in JSON**

```python
class Meta:
    model = Verses
    fields = [
        "verse_pk",
        "page",
        "hizbQuarter",
        "juz",
        "surah",
        "verse",
        "verseWithoutTashkeel",
        "numberInSurah",
        "numberInQuran",
    ]
```

**Note:** `audio`, `audio1`, `audio2`, and `sajda` fields are NOT included! They exist in the model but won't appear in API responses.

---

### 4. views.py - Request Handling

#### What are Views?

**Views** = Functions that handle HTTP requests and return HTTP responses

**Flow:**
1. Client sends HTTP request
2. Django calls your view function
3. View queries database
4. View returns JSON response

#### Code Breakdown

**Lines 1-7: Imports**

```python
from django.db.models import Q
```
- `Q` object = Advanced query filtering
- Allows "OR" conditions: `filter(A | B)` = A OR B

```python
from django.shortcuts import get_object_or_404, render
```
- `get_object_or_404`: Get object or return 404 error
- `render`: Render HTML template

```python
from rest_framework.decorators import api_view
from rest_framework.response import Response
```
- `@api_view`: Decorator to make function accept only specific HTTP methods
- `Response`: Return JSON responses

**Line 10-18: get_all Function**

```python
@api_view(["GET"])  # ← Decorator: only allow GET requests
def get_all(request) -> None:
```

**What @api_view does:**
- Without it: Works but less efficient
- With it: Adds extra features (automatic JSON parsing, CSRF protection)

```python
verses = Verses.objects.all()  # Get all verses from database
```

**QuerySet Explained:**
- `Verses.objects.all()` returns a QuerySet
- QuerySet = Lazy evaluation (doesn't hit database until needed)
- Like a promise: "I'll get all verses when you actually need them"

```python
data = VersesSerializers(verses, many=True).data
```

**Two-Step Process:**
1. `VersesSerializers(verses, many=True)` - Create serializer with data (many=True = multiple objects)
2. `.data` - Convert to Python dictionary

```python
return Response({"length": len(data), "data": data})
```

**Response Structure:**
```json
{
  "length": 6236,
  "data": [
    {...verse 1...},
    {...verse 2...},
    ...
  ]
}
```

**Lines 21-31: search Function**

```python
@api_view(["GET"])
def search(request, words) -> str:
```

**URL Parameter:** `words` comes from URL pattern: `<str:words>`

**Lines 27-29: The Search Logic**
```python
verses = Verses.objects.filter(
    Q(verse__icontains=words) | Q(verseWithoutTashkeel__icontains=words)
)
```

**Breaking this down:**
- `filter()`: Get rows matching criteria
- `Q(...) | Q(...)`: OR condition (verses matching EITHER condition)
- `verse__icontains`: Case-insensitive contains (LIKE in SQL)

**What "__icontains" means:**
- `verse__icontains="الل"` = "verse contains 'الل', case doesn't matter"
- SQL equivalent: `WHERE verse LIKE '%الل%'`

**Lines 34-44: get_surah Function**

```python
surah_pk = f"S{str(surah_id).zfill(3)}"
```

**What `.zfill(3)` does:**
- Pads string with zeros to make it 3 digits
- Examples: `zfill(3)`: "1" → "001", "12" → "012"

**Example:**
```python
surah_id = 2
surah_pk = f"S{str(2).zfill(3)}"  # "S002"
```

**Lines 47-57: get_verse_in_surah Function**

```python
verse_pk = f"S{str(surah_id).zfill(3)}V{str(verse_id).zfill(3)}"
```

**Example:**
```python
surah_id = 2, verse_id = 255
verse_pk = f"S{002}V{255}"  # "S002V255"
```

**Lines 55: get_object_or_404**
```python
verse = get_object_or_404(Verses, verse_pk=verse_pk)
```

**Behavior:**
- If found: Return the verse object
- If not found: Return 404 HTTP error (not the object)

**Why use this?**
- Returns proper HTTP status codes
- Standard REST API practice

**Lines 60-69: get_verse_in_quran Function**

Similar to above, but searches by absolute verse number (1-6236)

**Lines 72-73: api_docs Function**

```python
def api_docs(request):
    return render(request, "search/api.html")
```

**Renders HTML template** instead of JSON
- Used for displaying API documentation

---

### 5. urls.py - URL Routing

#### What is URL Routing?

**URL routing** = Mapping URLs to view functions

**Analogy:** Like a phone operator: "Call for John goes to extension 123"

#### Code Breakdown

**Line 3: Import All Views**
```python
from .views import *
```
- `*` = Import all functions from views.py
- Allows using them without prefix (e.g., `search` not `views.search`)

**Line 5: App Name**
```python
app_name = "QuranLexicalSearchAPI"
```
- Namespace for URLs (prevents conflicts if multiple apps have same URL name)

**Lines 7-23: URL Patterns**

```python
path("", api_docs, name="api_docs"),
```
- `""` = Root URL (e.g., `http://localhost:8000/`)
- `api_docs` = View function to call
- `name="api_docs"` = Name for reverse URL lookup

```python
path("api/lexical/search/<str:words>", search, name="search"),
```

**URL Pattern Breakdown:**
- `"api/lexical/search/<str:words>"` = URL pattern
- `<str:words>` = Captures URL segment as string, passes to view as `words` parameter
- Example: `/api/lexical/search/الله` → calls `search(request, words="الله")`

**Line 14-16: Multi-Parameter URL**
```python
path(
    "api/lexical/verse-in-surah/<int:surah_id>/<int:verse_id>",
    get_verse_in_surah,
    name="get_verse_in_surah",
),
```

**Example:**
- URL: `/api/lexical/verse-in-surah/2/255`
- Calls: `get_verse_in_surah(request, surah_id=2, verse_id=255)`
- `<int:...>` = Converts to integer automatically

---

### 6. apps.py - Application Configuration

**Lines 1-6:**

```python
from django.apps import AppConfig

class SearchConfig(AppConfig):
    default_auto_field = "django.db.models.BigAutoField"
    name = "search"
```

**Purpose:** Configuration for the "search" app

**default_auto_field:**
- Sets default primary key type to BigAutoField
- New Django 3.2+ feature

**name:**
- App name = "search" (must match directory name)

---

### 7. admin.py - Django Admin Interface

#### What is Django Admin?

**Django Admin** = Automatic web interface for managing data (built-in!)

**URL:** `http://localhost:8000/admin/`

**Lines 5-6: Customize Admin**
```python
admin.site.site_header = "Quran Search"
admin.site.site_title = "Quran Search"
```

**Changes admin page title**

**Lines 12-25: Verses Admin**

```python
@admin.register(Verses)  # ← Decorator: register model in admin
class ModelVerses(admin.ModelAdmin):
    list_display = [...]  # Columns to show in list view
    list_filter = [...]   # Filters in sidebar
    search_fields = [...] # Searchable fields
```

**What this creates:**
- Web interface to view, add, edit, delete verses
- Filter by Surah, Juz, etc.
- Search by verse text

**Lines 28-31: Surah Admin**

Similar setup for Surah model

---

### 8. tests.py - Testing

**Current Status:** Empty (no tests written yet)

**Why tests matter:**
- Ensure code works correctly
- Prevent bugs
- Refactor with confidence

**Example test (not in code, but you should add):**
```python
from django.test import TestCase
from .models import Verses, Surah

class SearchTestCase(TestCase):
    def test_search_finds_verses(self):
        # Test that search returns results
        response = self.client.get('/api/lexical/search/الله')
        self.assertEqual(response.status_code, 200)
```

---

## Search System Analysis

### Current Search Implementation

**Type:** Lexical (keyword-based) search

**Method:** Simple LIKE query in database

```python
Verses.objects.filter(
    Q(verse__icontains=words) | Q(verseWithoutTashkeel__icontains=words)
)
```

**SQL Equivalent:**
```sql
SELECT * FROM verses 
WHERE verse LIKE '%words%' 
   OR verseWithoutTashkeel LIKE '%words%';
```

### How Search Works

1. User searches for "الل"
2. Database scans EVERY verse in database
3. Checks if verse or verseWithoutTashkeel contains "الل"
4. Returns ALL matching verses
5. Serializes to JSON and returns

### Limitations

**No ranking:** Results not ordered by relevance
**No preprocessing:** No stemming, stopword removal
**Slow for large datasets:** No indexing optimization
**No fuzzy matching:** Exact substring match only

### What's NOT Implemented

**Advanced features:**
- ❌ Inverted index (faster lookups)
- ❌ TF-IDF ranking (relevance scoring)
- ❌ BM25 algorithm (better ranking)
- ❌ Caching (Redis/Memcached)
- ❌ Full-text search with PostgreSQL
- ❌ Fuzzy matching (typos)
- ❌ Arabic language-specific processing

---

## How to Extend

### 1. Add Ranking to Results

```python
def search(request, words) -> str:
    verses = Verses.objects.filter(...)
    
    # Add ranking logic
    results = []
    for verse in verses:
        score = verse.verse.count(words)  # Count occurrences
        results.append((score, verse))
    
    # Sort by score
    results.sort(key=lambda x: x[0], reverse=True)
    
    # Extract just the verses
    verses = [verse for score, verse in results]
    
    data = VersesSerializers(verses, many=True).data
    return Response({"length": len(data), "data": data})
```

### 2. Add Pagination

```python
from rest_framework.pagination import PageNumberPagination

def search(request, words) -> str:
    verses = Verses.objects.filter(...)
    
    paginator = PageNumberPagination()
    paginator.page_size = 50  # 50 results per page
    
    result_page = paginator.paginate_queryset(verses, request)
    
    data = VersesSerializers(result_page, many=True).data
    return paginator.get_paginated_response(data)
```

### 3. Add Caching

```python
from django.core.cache import cache

def search(request, words) -> str:
    # Try cache first
    cache_key = f"search_{words}"
    cached_result = cache.get(cache_key)
    
    if cached_result:
        return Response(cached_result)
    
    verses = Verses.objects.filter(...)
    data = VersesSerializers(verses, many=True).data
    
    response_data = {"length": len(data), "data": data}
    
    # Cache for 1 hour
    cache.set(cache_key, response_data, 3600)
    
    return Response(response_data)
```

### 4. Switch to PostgreSQL with Full-Text Search

**In settings.py:**
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'quran_db',
        'USER': 'your_user',
        'PASSWORD': 'your_password',
    }
}
```

**Add GIN index:**
```python
# In models.py
from django.contrib.postgres.search import SearchVectorField

class Verses(models.Model):
    # ... existing fields ...
    search_vector = SearchVectorField(null=True)
```

---

## Common Issues & Debugging

### Issue 1: ImportError

**Error:** `ModuleNotFoundError: No module named 'search'`

**Fix:** Make sure you're in correct directory and virtual environment is activated

### Issue 2: No Database Tables

**Error:** `django.db.utils.OperationalError: no such table`

**Fix:** Run migrations
```bash
python manage.py makemigrations
python manage.py migrate
```

### Issue 3: CORS Error

**Error:** `Access to XMLHttpRequest has been blocked by CORS policy`

**Fix:** Already enabled in settings.py
```python
CORS_ALLOW_ALL_ORIGINS = True
```

### Issue 4: Serializer Returns Empty

**Check:**
1. Is data in database? (Check admin interface)
2. Are correct fields listed in `fields = [...]`?
3. Is `many=True` set correctly?

---

## Learning Resources

### Django
- Official Docs: https://docs.djangoproject.com/
- Django for Beginners: Book by William Vincent

### Django REST Framework
- Official Docs: https://www.django-rest-framework.org/
- DRF Tutorial: Built into Django docs

### Search
- Elasticsearch: https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html
- PostgreSQL Full-Text Search
- Lucene (search engine library)

---

## Summary

### What You Learned

1. **Django Models** = Database tables defined in Python
2. **Django Views** = Functions that handle HTTP requests
3. **Django Serializers** = Convert Python objects to JSON
4. **Django URLs** = Route HTTP requests to views
5. **QuerySets** = Lazy database queries
6. **Django ORM** = Database operations without SQL

### Key Concepts

- **@api_view decorator**: Restricts HTTP methods
- **Q objects**: Complex database filters (OR, AND)
- **ForeignKey**: Relationships between tables
- **get_object_or_404**: Handle missing objects gracefully
- **Serializer**: Python ↔ JSON conversion

### Current System Limitations

- Basic keyword search only
- No ranking/relevance scoring
- No caching (slow for large datasets)
- No advanced search features

### Next Steps

1. Add pagination
2. Implement caching
3. Add ranking algorithm
4. Write tests
5. Add rate limiting
6. Consider Elasticsearch for advanced search

