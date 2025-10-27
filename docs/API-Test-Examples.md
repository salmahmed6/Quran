##  API Usage Examples

*   [API Usage Examples](#api-usage-examples)
    *   [Search Verses by Keyword](#1-search-verses-by-keyword)
    *   [Get All Verses](#2-get-all-verses)
    *   [Get Surah by ID](#3-get-surah-by-id)
    *   [Get Verse by Surah and Verse Number](#4-get-verse-by-surah-and-verse-number)
    *   [Get Verse by Absolute Verse Number](#5-get-verse-by-absolute-verse-number)
*   [Response Format](#response-format)


### 1. Search Verses by Keyword

Search for verses containing a specific Arabic word or phrase.

**Endpoint:**
```
GET /api/lexical/search/الله
```

**cURL:**
```bash
curl http://localhost:8000/api/lexical/search/الله
```

**Response:**
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
    }
    // ... more results
  ]
}
```

### 2. Get All Verses

Retrieve all 6,236 verses from the Quran (use with caution in production).

**Endpoint:**
```
GET /api/lexical/all
```

**cURL:**
```bash
curl http://localhost:8000/api/lexical/all
```

### 3. Get Surah by ID

Get all verses from a specific Surah (1-114).

**Endpoint:**
```
GET /api/lexical/surah/1
```

**cURL:**
```bash
curl http://localhost:8000/api/lexical/surah/1
```

### 4. Get Verse by Surah and Verse Number

Retrieve a specific verse by its Surah number and verse number within that Surah.

**Endpoint:**
```
GET /api/lexical/verse-in-surah/2/255
```

**cURL:**
```bash
curl http://localhost:8000/api/lexical/verse-in-surah/2/255
```

**Response:**
```json
{
  "data": {
    "verse_pk": "S002V255",
    "page": 42,
    "hizbQuarter": 5,
    "juz": 3,
    "surah": "البقرة",
    "verse": "اللَّهُ لَا إِلَٰهَ إِلَّا هُوَ الْحَيُّ الْقَيُّومُ...",
    "verseWithoutTashkeel": "الله لا إله إلا هو الحي القيوم...",
    "numberInSurah": 255,
    "numberInQuran": 286
  }
}
```

### 5. Get Verse by Absolute Verse Number

Get a verse by its sequential number in the entire Quran (1-6236).

**Endpoint:**
```
GET /api/lexical/verse-in-quran/286
```

**cURL:**
```bash
curl http://localhost:8000/api/lexical/verse-in-quran/286
```

---

##  Response Format

All API responses follow a consistent structure:

**Success Response (Search/List endpoints):**
```json
{
  "length": <number_of_results>,
  "data": [
    {
      "verse_pk": "S001V001",
      "page": <page_number>,
      "hizbQuarter": <quarter_number>,
      "juz": <juz_number>,
      "surah": "<surah_name>",
      "verse": "<verse_text_with_tashkeel>",
      "verseWithoutTashkeel": "<verse_text_without_tashkeel>",
      "numberInSurah": <verse_number_in_surah>,
      "numberInQuran": <absolute_verse_number>
    }
  ]
}
```

**Success Response (Single verse endpoints):**
```json
{
  "data": {
    "verse_pk": "S001V001",
    "page": <page_number>,
    "hizbQuarter": <quarter_number>,
    "juz": <juz_number>,
    "surah": "<surah_name>",
    "verse": "<verse_text_with_tashkeel>",
    "verseWithoutTashkeel": "<verse_text_without_tashkeel>",
    "numberInSurah": <verse_number_in_surah>,
    "numberInQuran": <absolute_verse_number>
  }
}
```

**Error Response (404):**
```json
{
  "detail": "Not found."
}
```
