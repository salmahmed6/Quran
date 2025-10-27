# Changelog

## [Backend API Update] - 2024

### Changed
- **Removed fields from API responses**: Removed `audio`, `audio1`, `audio2`, and `sajda` fields from all API endpoint responses
- **Updated serializers**: Modified `VersesSerializers` in `backend/api/lexical/search/serializers.py` to exclude audio and sajda fields
- **Updated documentation**: 
  - Rewrote `README.md` with complete API documentation focused on lexical search
  - Updated `API-Test-Examples.md` with clean response format

### New Response Format
All API responses now return only essential verse data:

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

### Technical Details
- **Serializers**: Fields list reduced from 12 to 9 fields (removed: `audio`, `audio1`, `audio2`, `sajda`)
- **Database**: Fields remain in database models for potential future use but are not exposed in API responses
- **Backward Compatibility**: Existing API consumers will need to update their code to remove references to the removed fields

### Files Modified
1. `backend/api/lexical/search/serializers.py` - Updated field list
2. `README.md` - Complete rewrite focusing on lexical search API
3. `API-Test-Examples.md` - Updated response examples

### Migration Notes
No database migration required. Changes only affect response serialization.

