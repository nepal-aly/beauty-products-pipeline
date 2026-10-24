# Beauty Products Data Pipeline

An end-to-end ETL pipeline in Python that extracts makeup and perfume data from two public APIs, cleans it with Pandas, loads it into a SQLite database, and exports it to Excel.

---

## Tech Stack

* **Language:** Python 3
* **Extraction:** `requests` (REST APIs, pagination, rate limiting)
* **Transformation:** Pandas, regular expressions
* **Loading:** SQLite (`sqlite3`), Excel (`openpyxl`)
* **Analysis:** SQL (`JOIN`, `GROUP BY`, `CASE`)

---

## Data Sources

| Source | What it contains | License |
|---|---|---|
| [Makeup API](https://makeup-api.herokuapp.com) | 931 makeup products from 58 brands, with prices, tags, and shades | Free public API |
| [Open Beauty Facts](https://world.openbeautyfacts.org) | Perfumes and makeup products from around the world | [ODbL](https://opendatacommons.org/licenses/odbl/1-0/) |

Data from Open Beauty Facts is used under the Open Database License (ODbL).

---

## Pipeline

```text
Makeup API ─────────┐
                    ├──> Extract ──> data/raw/*.csv ──> Transform ──> Load ──> SQLite + Excel
Open Beauty Facts ──┘
```

1. **Extract** — Requests all makeup products in one call, and pages through Open Beauty Facts 100 products at a time, waiting 7 seconds between requests to respect its rate limit.
2. **Raw layer** — Saves the untouched data to `data/raw/` as CSV files.
3. **Transform** — Cleans brands, names, prices, tags, shades, and product sizes (details below).
4. **Load** — Writes four tables to `data/clean/beauty_products.db`.
5. **Export** — Saves every table plus a data quality sheet to `data/clean/beauty_products.xlsx`.

---

## Data Cleaning

| Problem in the raw data | Fix |
|---|---|
| Brand names in lowercase (`nyx`, `l'oreal`, `covergirl`) | Title case plus a fix list (`NYX`, `L'Oréal`, `CoverGirl`) |
| HTML codes in product names (`All About Shadow&trade;`) | Decoded with `html.unescape` |
| Duplicate products with the same brand, name, and type | Removed |
| Prices stored as text, and 40 products priced at `0.0` | Converted to numbers; zero prices set to empty |
| 563 products with no currency | Marked as `Unknown`, so averages use USD prices only |
| Inconsistent tags (`cruelty free`, `Ewg Verified`) and an internal tag (`purpicks`) | Normalized and removed |
| Invalid shade color codes | Kept only valid hex codes like `#f4e5d6` |
| Open Beauty Facts products with no name | Removed |
| Several brands in one field (`Armaf, Armaf Odyssey, Odyssey`) | Kept the first brand |
| Country tags like `en:united-states` | Converted to `United States` |
| Sizes written many ways (`0.25 l`, `1 X 1oz`, `24,5g`, `100 mL`) | Parsed into a number plus a unit (`ml` or `g`) |

### Rows before and after cleaning

| Table | Before | After |
|---|---|---|
| makeup_products | 931 | 916 |
| makeup_colors | 5,329 | 5,254 |
| makeup_tags | 419 | 378 |
| beauty_products | 1,400 | 1,308 |
| **Total** | **8,079** | **7,856** |

---

## Database Tables

* **makeup_products** — one row per product: brand, name, type, category, price, currency, rating, links
* **makeup_colors** — one row per shade, linked by `product_id`
* **makeup_tags** — one row per tag (Vegan, Organic, Gluten Free...), linked by `product_id`
* **beauty_products** — Open Beauty Facts perfumes and makeup: name, brand, countries, size

---

## Sample Insights

* **Most products:** NYX (163), Clinique (93), and Dior (74)
* **Most shades:** Smashbox Be Legendary Lipstick, with 120 shades
* **Highest average price (USD):** Eyeshadow ($20.92) and Foundation ($19.92)
* **Most vegan products:** Eyeshadow (12) and Eyeliner (9)

---

## Project Structure

* **beauty_pipeline.ipynb** — the full pipeline, step by step
* **data/raw/** — raw data as extracted from the APIs
* **data/clean/beauty_products.db** — SQLite database
* **data/clean/beauty_products.xlsx** — Excel file with all tables and a data quality sheet
* **README.md** — project documentation

---

## Installation & Execution

1. **Clone the repository:**
   `git clone https://github.com/nepal-aly/beauty-products-pipeline.git`

2. **Navigate to the directory:**
   `cd beauty-products-pipeline`

3. **Install required dependencies:**
   `pip install requests pandas openpyxl jupyter`

4. **Run the notebook:**
   `jupyter notebook beauty_pipeline.ipynb`

The full run takes about 2–3 minutes because of the Open Beauty Facts rate limit.
