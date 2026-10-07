# Automated ETL Pipeline for Automotive Parts Catalog Integration (n8n)

An end-to-end ETL integration pipeline built in **n8n**, automating data retrieval, normalization, margin calculation, and export for automotive parts flagged as "not found" on an e-commerce portal across multiple suppliers directly into ERP-compatible formats.

---

## Key Features

* **REST API Orchestration:** Complete handling of `OAuth 2.0` (Client Credentials) authorization and asynchronous querying of distributor catalog and pricing endpoints (including Inter Cars WebAPI).
* **Fallback Web Scraping:** Real-time HTML parsing using the `Cheerio` library for suppliers that do not provide a dedicated REST API.
* **PostgreSQL Integration:** Direct SQL queries matching price lists, product categories, and performing automated audits for unresolved items (`INSERT INTO nieznalezione`).
* **Normalization & Deduplication Algorithms:** JavaScript (ES6+) scripts generating index permutations (pure OEM codes, dashed formats, Bosch indexing, dynamic brand prefixes) and removing duplicate entries.
* **Margin & Discount Calculation Engine:** Dynamic joining of input records with master brand discount sheets, automating net purchase and retail pricing computations.
* **ERP-Ready File Generation:** Multi-column `.xls` catalog generation (tailored to standard ERP inventory card structures) uploaded automatically to Google Drive for direct ingestion into the ERP system.

---

## Tech Stack

| Category | Technologies |

| **Integration Engine** | n8n |
| **Scripting Languages** | JavaScript (Node.js / ES6+), SQL |
| **Database** | PostgreSQL |
| **Libraries** | Cheerio |
| **Protocols & Formats** | REST API, OAuth 2.0, HTTP, JSON, XLS |
| **Cloud Integrations** | Google Drive API |

---

## Workflow Architecture

1. **Input Ingestion & Data Preparation:**
   * Fetches the working spreadsheet uploaded by the ERP system (containing customer queries from the web storefront flagged as "not found") from Google Drive.
   * Extracts rows and performs preliminary string sanitation (stripping whitespace, part number deduplication).
2. **Cascade Lookup Strategy:**
   * **Tier 1 (API Lookup):** Generates index permutations with supplier prefixes and queries distributor REST endpoints. On a match, retrieves real-time net pricing and metadata required for proper ERP parsing via quote endpoints.
   * **Tier 2 (Web Scraping Fallback):** If the API returns no results, redirects queries to subsequent suppliers' web catalogs, parsing HTML table structures using `Cheerio`.
   * **Tier 3 (SQL Database Lookup):** Performs parallel lookups across local product catalog tables for alternative suppliers in PostgreSQL.
3. **Data Mapping & Transformation:**
   * Maps supplier brand strings against a unified manufacturer dictionary (OE / Aftermarket).
   * Joins corresponding brand discount percentages from the master matrix file.
4. **Export & Audit Logging:**
   * Splits data streams into dedicated import files formatted for seamless ERP intake.
   * Converts payloads into `.xls` spreadsheets and automatically uploads them to Google Drive for direct ERP ingestion.
   * Inserts unmapped/unresolved part numbers into an audit database table for procurement review.

---

> **Sanitization Notice:**  
> All sensitive production credentials—including API keys, access tokens, session cookies, account identifiers, and database connection strings—have been stripped or replaced with placeholder values (`PLACEHOLDER`) for public presentation.
