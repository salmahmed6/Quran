<h1 align="center">Quranic Lexical Search API</h1>
<h4 align="center">A Django REST API for Lexical Search in the Quran</h4>

##  Table of Contents

*   [About the Project](#about-the-project)
    *   [What is this API?](#what-is-this-api)
    *   [Why Lexical Search?](#why-lexical-search)
    *   [Features](#features)
    *   [Architecture](#architecture)
*   [Tech Stack](#tech-stack)
*   [API Endpoints](#api-endpoints)
*   [Directory Structure](#directory-structure)
*   [Getting Started](#getting-started)
    *   [Prerequisites](#prerequisites)
    *   [Installation](#installation)
    *   [Running the API](#running-the-api)
*   [Response Format](#response-format)
*   [Development](#development)
---

##  About the Project

This project provides a **Django REST API** for performing lexical (keyword-based) search in the Quran. It enables developers and researchers to programmatically search, retrieve, and access verses from the Quran using simple HTTP requests.

###  What is this API?

The **Quranic Lexical Search API** is a backend service that provides programmatic access to all 6,236 verses of the Holy Quran. It allows you to:

- Search for specific Arabic words or phrases across all verses
- Retrieve all verses from a specific Surah
- Access individual verses by their position in a Surah or in the entire Quran
- Get verses with or without Tashkeel (diacritical marks)

###  Why Lexical Search?

Lexical (keyword-based) search is essential for:
- Finding exact word occurrences in the Quran
- Matching specific Arabic terms or phrases
- Case-insensitive text matching across the entire corpus
- Quick and accurate retrieval based on exact text patterns

###  Features

- **RESTful API**: Clean, intuitive REST endpoints
- **Case-Insensitive Search**: Find words regardless of letter case
- **Dual Text Matching**: Search in verses both with and without Tashkeel (diacritical marks)
- **Structured Responses**: JSON responses with verse metadata (Surah, Juz, page, audio URLs, etc.)
- **Fast & Efficient**: Built on Django with SQLite3 for quick responses
- **CORS Enabled**: Ready for cross-origin requests from web applications
- **Admin Interface**: Django admin for managing verses and Surahs

###  Architecture

```
Client Request
    ↓
Django URL Router
    ↓
View Handler (DRF)
    ↓
Database Query (SQLite3)
    ↓
JSON Response
    ↓
Client
```

**Request Flow:**
1. Client sends HTTP GET request to an endpoint
2. Django URL router matches the path to a view function
3. View performs a database query using Django ORM
4. Response is serialized to JSON using Django REST Framework
5. JSON response is returned to the client

---

##  Tech Stack

| Component | Technology | Description |
|-----------|-----------|-------------|
| **Backend Framework** | [Django 5.2](https://www.djangoproject.com/) | High-level Python web framework |
| **API Framework** | [Django REST Framework](https://www.django-rest-framework.org/) | Toolkit for building Web APIs |
| **Database** | [SQLite3](https://www.sqlite.org/) | Lightweight embedded database |
| **CORS** | [django-cors-headers](https://github.com/adamchainz/django-cors-headers) | Cross-Origin Resource Sharing |
| **Server** | [Gunicorn](https://gunicorn.org/) | Python WSGI HTTP Server |
| **Static Files** | [WhiteNoise](https://whitenoise.evans.io/) | Static file serving |

---

##  API Endpoints

All endpoints are prefixed with `/api/lexical/`

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/lexical/search/<word>` | Search verses containing a specific word/phrase |
| `GET` | `/api/lexical/all` | Retrieve all verses in the database |
| `GET` | `/api/lexical/surah/<surah_id>` | Get all verses from a specific Surah |
| `GET` | `/api/lexical/verse-in-surah/<surah_id>/<verse_id>` | Get a specific verse within a Surah |
| `GET` | `/api/lexical/verse-in-quran/<verse_id>` | Get a verse by its absolute number in the Quran |
| `GET` | `/` | API documentation page |

---

##  Directory Structure

```
backend/
└── api/
    └── lexical/
        ├── api/                          # Django project settings
        │   ├── __init__.py
        │   ├── settings.py                # Application configuration
        │   ├── urls.py                    # Main URL routing
        │   ├── wsgi.py                    # WSGI config
        │   └── asgi.py                    # ASGI config
        ├── search/                        # Search application
        │   ├── migrations/                # Database migrations
        │   ├── templates/                 # HTML templates for API docs
        │   ├── static/                    # Static files (CSS)
        │   ├── __init__.py
        │   ├── admin.py                   # Django admin configuration
        │   ├── models.py                   # Database models (Verses, Surah)
        │   ├── serializers.py             # API serializers
        │   ├── urls.py                    # URL patterns
        │   └── views.py                   # API views
        ├── db/                            # Database files
        │   ├── db.sqlite3                 # Main database
        │   └── quran.db                   # Quran data
        ├── db.sqlite3                     # Migrated database
        ├── manage.py                      # Django management script
        └── requirements.txt               # Python dependencies
```

---

##  Getting Started

###  Prerequisites

Before you begin, ensure you have the following installed:

- **Python 3.8+** ([Download Python](https://www.python.org/downloads/))
- **pip** (Python package installer)
- **Git** (optional, for cloning the repository)

### 💻 Installation

1. **Clone the repository** (or navigate to the existing project):
   ```bash
   git clone https://github.com/ahr9n/quranic-search-v2.git
   cd quranic-search-v2/backend/api/lexical
   ```

2. **Create and activate a virtual environment**:
   ```bash
   # Create virtual environment
   python -m venv venv
   
   # Activate virtual environment
   # On Windows:
   venv\Scripts\activate
   # On macOS/Linux:
   source venv/bin/activate
   ```

3. **Install dependencies**:
   ```bash
   pip install -r requirements.txt
   ```

4. **Run database migrations**:
   ```bash
   python manage.py migrate
   ```

5. **Load the Quran database** (if not already loaded):
   The database file `quran.db` should be located in the `db/` directory. If migrations don't populate the data automatically, you may need to load it using custom scripts or management commands.

###  Running the API

Start the development server:

```bash
python manage.py runserver
```

The API will be available at `http://localhost:8000/`

To run with Gunicorn (production-like):

```bash
gunicorn api.wsgi:application --bind 0.0.0.0:8000
```

---

##  Development

### Running Tests

```bash
python manage.py test
```

### Accessing Django Admin

Create a superuser (if not already created):

```bash
python manage.py createsuperuser
```

Then visit `http://localhost:8000/admin/` and log in with your credentials.

### Project Settings

Key settings are configured in `api/settings.py`:

- `CORS_ALLOW_ALL_ORIGINS = True` - Enables CORS for all origins (configure for production)
- `ALLOWED_HOSTS = ["*"]` - Allows all hosts (restrict for production)
- `DATABASES` - Configured to use SQLite3

### Database Models

- **Verses**: Main model containing verse text, metadata (page, juz, surah), and audio URLs
- **Surah**: Model for Surah names (with and without Tashkeel)
