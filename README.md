# Historical Mutual Funds Data

An archive of historical mutual fund pricing information in India. This repository contains the raw data as well as the scripts used to generate it.

The data in the `data/` directory is in the same format as AMFI exports it, without any changes. This notably includes a few errors:

1. A few invalid ISINs (such as `NOTAPP` or `NA` for "Not Applied"), and a few with invalid prefixes (`IINF` instead of `INF`) or lowercase ISINs.
2. Lots of invalid NAVs, such as `"#N/A",'#DIV/0!','N.A.', 'NA', 'B.C.', 'B. C.'`

## Usage

### Installation

You can get the latest dataset at <https://github.com/captn3m0/historical-mf-data/releases/latest/funds.db.zst>. Each dataset includes all historical NAVs at all known times for a given mutual fund. See below for the data format.

### Setup

The dataset does not include _search indexes_ to reduce the download size. Please run the following commands to setup:

```bash
wget https://github.com/captn3m0/historical-mf-data/releases/latest/funds.db.zst
unzstd funds.db.zst
# Create search indexes
echo 'CREATE INDEX "date" ON "nav" ("date","scheme_code")' | sqlite funds.db
```

## Versioning

The versioning scheme follows SemVer, with the date being used for the minor and patch version in a  `MAJOR.MINOR.YYYYMMDD` format. This results in the date being clearly provided in the version number.

The Major number is currently 0, to denote alpha release status. It will be bumped to 1 once the database schema is stable.
Minor releases will be bumped on non-breaking changes to the schema - such as new fields, or indexes being added.
Major version will be bumped only on breaking changes.

## Data Format

The output dataset is a SQLite Database, with the following schema:

### schemes

```sql
scheme_code INTEGER PRIMARY_KEY
scheme_name TEXT
```

### funds

```sql
date
scheme_code INTEGER
nav FLOAT
FOREIGN KEY (scheme_code) REFERENCES schemes(scheme_code));
```

### securities

```sql
isin TEXT UNIQUE
--- 0=Growth/Divident Payout
--- 1=Divident Reinvestment
type INTEGER 
scheme_code INTEGER
FOREIGN KEY (scheme_code) REFERENCES schemes(scheme_code));
```

### nav_by_isin (View)

Helper view to directly query NAV against the ISIN.

```
isin TEXT
date
nav FLOAT
```

## Common Queries

### NAV as per Date from ISIN

Since Data is not always available on all dates, you need to get the latest value before or on that date:

```sql
SELECT date,nav from nav_by_isin
WHERE isin='INF277K01741'
AND date<='2023-03-23'
ORDER BY date DESC
LIMIT 0,1
```

### Latest NAV

```sql
SELECT nav from nav_by_isin
WHERE isin='INF277K01741'
ORDER BY date DESC
LIMIT 0,1
```

### Last 90 Financial Days NAV

```sql
SELECT date,nav from nav_by_isin
WHERE isin='INF277K01741'
ORDER BY date DESC
LIMIT 0,90
```

### Get Metadata of all Mutual Funds from ISIN

```sql
SELECT isin,type,S1.scheme_code,scheme_name FROM securities S1
LEFT JOIN schemes S2 ON S1.scheme_code = S2.scheme_code
```

### Get Information of Specific Funds from ISIN

```sql
SELECT isin,type,S1.scheme_code,scheme_name FROM securities S1
LEFT JOIN schemes S2 ON S1.scheme_code = S2.scheme_code
WHERE isin='INF277K01741'
```

## License

Licensed under the [MIT License](https://nemo.mit-license.org/). See LICENSE file for details.