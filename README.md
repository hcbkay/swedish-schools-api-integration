# Swedish Schools API Integration

**Role:** Volunteer Data Analyst  
**Organization:** Swedish nonprofit advocacy organization  (Ki-DS, formerly known as Smartphone Fri Barndom)
**Tech:** Python, REST API, pandas  

## The Problem

Parent signup form needded standardized school names for school selection, segmentation, and locality based visualization. This caused massive data quality issues:

- No way to validate if schools actually existed
- Impossible to analyze signups by school
- Manual data entry taking up to 1 hour every week
- Signups couldn't select their school(s) or there were multiple schools with the same name but in different municipalities 

**Root cause:** No validation 

## The Solution

Integrated with Skolverket (Swedish National Education Agency) API to:
1. Retrieve all 5,184 active Swedish schools from government registry
2. Identify 420 schools with duplicate names (e.g., "Parkskolan" appears in 18 different cities)
3. Resolve duplicates and add locality: "Parkskolan (Stockholm)" vs "Parkskolan"
4. Create validated dropdown for signup forms

**Result:** Eliminated future data entry errors entirely

## How It Works

**Process:**
1. Pull all active schools from Skolverket API (basic endpoint)
2. Separate schools with unique names vs duplicate names
3. Get detailed data for duplicates (includes address/locality info)
4. Create final list: unique names as-is + duplicates with (locality) added
5. Save to JSON for form integration

### Problems I Had to Figure Out

**Nested JSON Structure**
- The API returns really nested data: `['data']['attributes']['addresses'][0]['locality']`
- Had to navigate step by step and handle missing address data for some schools

**Different Field Names**
- Basic API endpoint uses `'name'` but detailed endpoint uses `'schoolName'`
- Caught this after getting errors and had to adjust the code

**API Rate Limiting**
- API only allows 10 calls per 10 seconds
- Kept hitting errors when processing 420 duplicate schools
- Got help from Claude on adding time.sleep() and progress indicators to handle this properly
- Ended up taking about 25 minutes total to process all duplicates and school names

See `skolverket_integration.ipynb` for the full code.

## Results

| Metric | Value |
|--------|-------|
| **Schools Retrieved** | 5,184 |
| **Unique School Names** | 4,764 |
| **Duplicates Resolved** | 420 |
| **Data Quality** | 99.92% |
| **Processing Time** | ~25 minutes |

### Sample Output

```json
{
  "label": "Parkskolan (Stockholm)",
  "value": "12345678"
},
{
  "label": "Parkskolan (Göteborg)",
  "value": "23456789"
}
```

See `sample_output.json` for examples.

## What This Fixed

**Before:**
- Manually adding school names when error was found: 30+ minutes
- Signups confused about school names being the same across municipalities
- No validation for segementation and map visualization on website

**After:**
- Zero manual entry needed (for now) 
- 100% validated school names from official government source
- Dropdown prevents all data entry errors
- Can actually do geographic analysis now since school names are standardized

## Skills Demonstrated

- REST API integration
- JSON data parsing (nested structures)
- Data deduplication logic
- ETL pipeline design
- Error handling for missing data
- Documentation and code organization

## Running the Code

```bash
pip install requests pandas jupyter

jupyter notebook skolverket_integration.ipynb
```

Or open in VS Code, Google Colab, etc.

Output saves to `~/Downloads/swedish_schools_dropdown.json`

---

**Note:** This project uses real API but sample output data for portfolio purposes. The methodology and code are from actual implementation.
