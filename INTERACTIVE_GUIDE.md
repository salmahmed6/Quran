# Interactive Django Guide - Hands-On Learning

## 🎯 Goal: Run and Test the Lexical Search API

This guide will help you run the API and test it with real examples.

---

## Step 1: Setup Your Environment

### Prerequisites Check
```bash
# Check Python version (should be 3.8+)
python --version

# Check pip is installed
pip --version
```

### Navigate to Project
```bash
cd backend/api/lexical

# Activate virtual environment
# On Windows:
venv\Scripts\activate

# On macOS/Linux:
source venv/bin/activate
```

### Install Dependencies
```bash
pip install -r requirements.txt
```

---

## Step 2: Understand the Database

### Check Database Schema

Run Django shell to explore:

```bash
python manage.py shell
```

**Try these commands in the shell:**

```python
# Import the models
from search.models import Verses, Surah

# Check total verses
print(f"Total verses: {Verses.objects.count()}")

# Get first verse
first_verse = Verses.objects.first()
print(f"First verse: {first_verse}")

# Check all fields
print(f"verse_pk: {first_verse.verse_pk}")
print(f"Surah: {first_verse.surah}")
print(f"Verse text: {first_verse.verse}")

# Search for verses containing "الله"
verses = Verses.objects.filter(verse__icontains="الله")
print(f"Verses with 'الله': {verses.count()}")

# Get a specific surah
from search.models import Surah
al_fatiha = Surah.objects.get(name__icontains="فاتحة")
print(f"Surah ID: {al_fatiha.id}, Name: {al_fatiha.name}")

# Exit shell
exit()
```

---

## Step 3: Test the API Endpoints

### Start the Server

```bash
python manage.py runserver
```

Server starts at: `http://localhost:8000`

---

## Step 4: Try Each Endpoint

### 1. Search Endpoint

**Test in browser:**
```
http://localhost:8000/api/lexical/search/الله
```

**Test with curl:**
```bash
curl http://localhost:8000/api/lexical/search/الله
```

**Test with Python:**
```python
import requests

response = requests.get('http://localhost:8000/api/lexical/search/الله')
data = response.json()

print(f"Found {data['length']} verses")
print(f"First verse: {data['data'][0]['verse']}")
```

**Expected Output:**
```json
{
  "length": 120,
  "data": [
    {
      "verse_pk": "S001V001",
      "page": 1,
      "hizbQuarter": 1,
      "juz": 1,
      "surah": "الفاتحة",
      "verse": "بِسْمِ اللَّهِ الرَّحْمَٰنِ الرَّحِيمِ",
      "verseWithoutTashkeel": "بسم الله الرحمن الرحيم",
      "numberInSurah": 1,
      "numberInQuran": 1
    },
    ...
  ]
}
```

---

### 2. Get All Verses

**Warning:** This returns ALL 6,236 verses (large response!)

```bash
curl http://localhost:8000/api/lexical/all
```

**Python:**
```python
import requests

response = requests.get('http://localhost:8000/api/lexical/all')
data = response.json()

print(f"Total verses in database: {data['length']}")
```

---

### 3. Get Specific Surah

**Test with curl:**
```bash
# Get Al-Fatiha (Surah 1)
curl http://localhost:8000/api/lexical/surah/1

# Get Al-Baqarah (Surah 2)
curl http://localhost:8000/api/lexical/surah/2
```

**Python:**
```python
import requests

response = requests.get('http://localhost:8000/api/lexical/surah/1')
data = response.json()

print(f"Surah has {data['length']} verses")
for verse in data['data'][:3]:  # Show first 3
    print(f"Verse {verse['numberInSurah']}: {verse['verse'][:50]}...")
```

**Expected Output:**
```json
{
  "length": 7,
  "data": [
    {
      "verse_pk": "S001V001",
      "page": 1,
      "hizbQuarter": 1,
      "juz": 1,
      "surah": "الفاتحة",
      "verse": "بِسْمِ اللَّهِ الرَّحْمَٰنِ الرَّحِيمِ",
      ...
    },
    ...
  ]
}
```

---

### 4. Get Verse by Surah and Verse Number

**Test:**
```bash
# Get verse 255 from Surah 2 (Ayat al-Kursi)
curl http://localhost:8000/api/lexical/verse-in-surah/2/255
```

**Python:**
```python
import requests

response = requests.get('http://localhost:8000/api/lexical/verse-in-surah/2/255')
data = response.json()

verse = data['data']
print(f"Verse {verse['numberInSurah']} from {verse['surah']}")
print(f"Text: {verse['verse']}")
```

---

### 5. Get Verse by Absolute Number

```bash
# Get verse 286 (first verse of Quran)
curl http://localhost:8000/api/lexical/verse-in-quran/286
```

---

## Step 5: Understanding Query Logic

### Examine the Search Function

Let's trace through the search function step-by-step:

```python
# From views.py, line 22-31

@api_view(["GET"])
def search(request, words) -> str:
    """
    Search in verses using insensitive contains.
    Return all verses with given words.
    """
    
    # Step 1: Query database
    verses = Verses.objects.filter(
        Q(verse__icontains=words) | Q(verseWithoutTashkeel__icontains=words)
    )
    
    # Step 2: Serialize to JSON
    data = VersesSerializers(verses, many=True).data
    
    # Step 3: Return response
    return Response({"length": len(data), "data": data})
```

**What happens when you call:**
```
http://localhost:8000/api/lexical/search/الرحمن
```

**Step 1: URL Routing**
```
1. URL pattern matches: "api/lexical/search/<str:words>"
2. words parameter = "الرحمن"
3. Django calls: search(request, words="الرحمن")
```

**Step 2: Database Query**
```python
# Verses.objects.filter(...) translates to SQL:
"""
SELECT * FROM search_verses 
WHERE verse LIKE '%الرحمن%' 
   OR verseWithoutTashkeel LIKE '%الرحمن%';
"""
```

**Step 3: Serialization**
```python
# QuerySet (Python objects) → Python dictionaries → JSON
verses = [Verse object 1, Verse object 2, ...]
         ↓
data = [{"verse_pk": "S001V001", ...}, {...}, ...]
```

**Step 4: Response**
```json
{
  "length": 7,
  "data": [...]
}
```

---

## Step 6: Custom Queries

### Try Different Searches

**Search by English:**
```bash
# Won't find anything (database is Arabic only)
curl http://localhost:8000/api/lexical/search/hello
```

**Search with diacritics:**
```bash
# With Tashkeel (diacritics)
curl http://localhost:8000/api/lexical/search/الرحمن

# Without Tashkeel
curl http://localhost:8000/api/lexical/search/الرحمن
```

**Partial word search:**
```bash
# Find verses with "رح" anywhere
curl http://localhost:8000/api/lexical/search/رح
```

---

## Step 7: Debugging Exercise

### Common Issues

**Issue 1: No Results Found**

```python
# If search returns empty, check in Django shell:
from search.models import Verses

# How many total verses?
print(Verses.objects.count())

# Does "الله" exist?
print(Verses.objects.filter(verse__icontains="الله").count())

# What's the actual text?
verse = Verses.objects.first()
print(verse.verse)
```

**Issue 2: Wrong Serialized Fields**

```python
# Check what fields serializer is including:
from search.serializers import VersesSerializers

# List all fields:
print(VersesSerializers().fields.keys())

# Check if 'audio' is included
print('audio' in VersesSerializers().fields)
```

**Issue 3: Database Empty**

```bash
# Check if database has data:
python manage.py shell
```

```python
from search.models import Verses
print(Verses.objects.count())  # Should be 6236

# If 0, you need to load data from quran.db
```

---

## Step 8: Exercise - Add a New Endpoint

### Task: Create Endpoint to Get Verses by Juz

**1. Add to `views.py`:**
```python
@api_view(["GET"])
def get_juz(request, juz_id):
    """
    Get all verses from a specific Juz (1-30).
    """
    verses = Verses.objects.filter(juz=juz_id).order_by('numberInQuran')
    data = VersesSerializers(verses, many=True).data
    return Response({"length": len(data), "data": data})
```

**2. Add to `urls.py`:**
```python
path("api/lexical/juz/<int:juz_id>", get_juz, name="get_juz"),
```

**3. Test it:**
```bash
curl http://localhost:8000/api/lexical/juz/1
```

---

## Step 9: Understanding Django ORM

### QuerySet Operations

```python
# Get all verses
verses = Verses.objects.all()

# Filter by surah
verses = Verses.objects.filter(surah_id=1)

# Count results
count = verses.count()

# Get first result
first = verses.first()

# Iterate through
for verse in verses:
    print(verse.verse)

# Order by
verses = Verses.objects.order_by('numberInQuran')

# Limit results
verses = Verses.objects.all()[:10]  # First 10

# Get one specific verse
verse = Verses.objects.get(verse_pk="S001V001")

# Multiple filters
verses = Verses.objects.filter(
    surah_id=2,
    page=42
)
```

### Q Objects - Complex Queries

```python
from django.db.models import Q

# OR query
verses = Verses.objects.filter(
    Q(verse__icontains="الله") | Q(verse__icontains="رسول")
)

# AND query
verses = Verses.objects.filter(
    Q(juz=1),
    Q(page=1)
)

# NOT query
verses = Verses.objects.exclude(
    surah_id=1
)

# Complex combinations
verses = Verses.objects.filter(
    Q(verse__icontains="الله") | Q(verse__icontains="رسول")
).exclude(
    juz=30
).order_by('-numberInQuran')[:50]
```

---

## Step 10: Performance Testing

### Measure Query Speed

```python
import time
from search.models import Verses

# Time the query
start = time.time()
verses = Verses.objects.filter(verse__icontains="الله")
count = verses.count()
end = time.time()

print(f"Found {count} verses in {end - start:.4f} seconds")
```

### Add Indexing (Optional)

```python
# In models.py, add Meta indexes:
class Verses(models.Model):
    # ... fields ...
    
    class Meta:
        ordering = ["id"]
        indexes = [
            models.Index(fields=['verse']),
            models.Index(fields=['verseWithoutTashkeel']),
            models.Index(fields=['juz', 'page']),
        ]
```

Then run:
```bash
python manage.py makemigrations
python manage.py migrate
```

---

## Summary Checklist

✅ Understood how Django models work  
✅ Learned about serializers  
✅ Tested all API endpoints  
✅ Explored QuerySets and filtering  
✅ Debugged common issues  
✅ Ready to extend the codebase  

---

## Next Steps

1. **Write tests** for the views
2. **Add pagination** to prevent large responses
3. **Implement caching** for faster responses
4. **Add rate limiting** to prevent abuse
5. **Create API documentation** with Swagger/OpenAPI

---

## Helpful Commands

```bash
# Create migrations from model changes
python manage.py makemigrations

# Apply migrations to database
python manage.py migrate

# Create superuser for admin
python manage.py createsuperuser

# Run tests
python manage.py test

# Open Django shell
python manage.py shell

# Start development server
python manage.py runserver

# Check for common issues
python manage.py check
```

