# Steam Games: SQL and EDA

A data-cleaning and SQL analysis project on the Steam Dataset 2025. The raw catalogue of 239,664 store listings is cleaned, validated and exported as a relational SQLite database of 150,279 games, which is then analysed in SQL.

**Status:** data cleaning and export complete; SQL and EDA notebook in progress.

## Demo

In progress. Results will be added after the SQL and EDA notebook.

## How to run

**Requirements:** Python 3.12 and the packages in `requirements.txt`.

```bash
pip install -r requirements.txt
```

**1. Get the data.** Download the Steam Dataset 2025 from Kaggle
(`crainbramp/steam-dataset-2025-multi-modal-gaming-analytics`) and use only the
`steam_dataset_2025_csv_package_v1` folder. The `reviews.csv` file is not used.

**2. Build the raw database.** In DB Browser for SQLite (standard build), create a new database at
`data/steam_data.sqlite`. Then use **File > Import > Table from CSV file** for each of these eleven files:
`applications`, `genres`, `categories`, `developers`, `publishers`, `platforms`,
`application_genres`, `application_categories`, `application_developers`,
`application_publishers`, `application_platforms`.

Accept the dialog defaults and set only the table name to the file name without `.csv`. The defaults are:
"Column names in first line" **unticked**, comma separator, double-quote character, UTF-8, "Trim fields" ticked.
Because the header is not treated as column names, it is imported as the first data row and every column is text.
The notebook accounts for both facts. Do not tick "Column names in first line", or the notebook will drop a real row from each table.

**3. Run the notebook.** Open `notebooks/01_data_cleaning_and_setup.ipynb` in JupyterLab and choose
**Kernel > Restart Kernel and Run All Cells**. It writes `data/processed/steam_clean.sqlite` (about 151 MB).
The `data/` folder is gitignored, so this file must be regenerated locally.

The schema is documented in [`docs/data-dictionary.md`](docs/data-dictionary.md).

## Approach

The cleaning notebook follows a fixed sequence: load, inspect, missing values, duplicates, categorical and
numerical checks, dates, relational integrity, cleaning, validation, export. Every cleaning decision is
explained in a markdown cell beside the code. Before export, 54 automated checks confirm key integrity,
foreign keys, value ranges and the counts recorded in earlier sections. The export enforces foreign keys and
is verified by reopening the file with independent connections.

| Stage | Rows |
|---|---|
| Raw listings | 239,664 |
| Games (`type = 'game'`) | 150,279 |
| Paid games | 131,285 |
| Paid games with a price | 90,311 |
| Paid games with a USD price | 88,943 |

## Data-quality findings

- **Platform flags contradicted the junction table.** The raw `supports_mac` and `supports_linux` flags were true for about 99.9% of apps, while the platform table gave about 23% and 17%. The junction table is treated as the source of truth and the flags are dropped.
- **Translated genres.** 154 genre IDs were translations of each other and were consolidated into 34 canonical genres (33 occur on games).
- **Categories** were filtered to English entries; 0.35% of category rows were dropped.
- **Prices** are stored in cents across 33 currencies. Only USD listings are converted (no exchange-rate data), so price analysis covers USD listings only.
- **Missing prices.** 31% of paid games have no price. This is not explained by unreleased status and is concentrated in recent listings, so price analyses skew toward older listings.
- **Discount** is a point-in-time snapshot at the scrape date (2025-09-29), not a price history.
- **Sparse fields.** Metacritic scores are missing for 97.2% of games and recommendations for 86.8%; both support subsample analysis only.
- **Dates.** 906 listings are future-dated, including sentinel-like years, and three games carry an epoch-zero date. Dates outside 1990-01-01 to 2025-09-29 are set to null and unreleased listings are flagged. 2025 is a partial year.
- **Relational coverage.** 7,266 games (4.8%) have no genre, category, developer or publisher rows and are flagged `has_relational_metadata = 0`.
- **Developers and publishers** sharing a trimmed, lowercased name were merged. Other spelling variants remain.
- **`required_age`** is 0 for 99.1% of games and is a weak variable.
- **Entity key.** `appid` is the key; no name-based deduplication was applied.

## Key results

In progress.

## What I would improve

- Replace the manual DB Browser import with a script that builds the raw database directly from the CSVs.
- Normalise remaining developer and publisher name variants beyond case and whitespace.
- Investigate why so many recent listings have no price.
- Add tests for the cleaning logic outside the notebook.