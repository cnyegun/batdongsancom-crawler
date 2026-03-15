# batdongsan.com.vn Crawler

Crawl apartment listing data from [batdongsan.com.vn](https://batdongsan.com.vn/ban-can-ho-chung-cu) for ML/data analysis projects.

Uses headless Chrome via Playwright with stealth to bypass bot detection. Supports parallel workers for fast crawling.

## Extracted fields

| Field | Example |
|-------|---------|
| `product_id` | `45179819` |
| `title` | `Căn hộ 2PN Vinhomes Grand Park` |
| `price_text` | `4,68 tỷ` |
| `area_text` | `71 m²` |
| `price_per_m2_text` | `65,92 tr/m²` |
| `bedrooms` | `2` |
| `bathrooms` | `2` |
| `location` | `Hồ Chí Minh` |
| `description` | Full listing description text |
| `post_date` | `15/03/2026` |
| `contact_name` | `Nguyễn Văn A` |
| `url` | Full listing URL |
| `page_num` | `1` |

## Setup

```bash
pip install -r requirements.txt
python -m playwright install chromium
```

## Usage

```bash
# Crawl pages 1-50 with 4 workers (default)
python crawl.py

# Crawl all ~2184 pages with 8 workers
python crawl.py --pages 1 2184 --workers 8

# Custom output file
python crawl.py --pages 1 100 --output data.csv

# Run in background
nohup python crawl.py --pages 1 2184 --workers 8 --output apartments.csv > crawl.log 2>&1 &
tail -f crawl.log
```

## How it works

1. Page range is split evenly across workers
2. Each worker launches its own headless Chrome and crawls its assigned pages
3. A fresh browser context is created per page to avoid bot detection
4. Browser is restarted every 20 pages to prevent memory leaks
5. Each worker writes results to its own `.tmp` CSV file (crash-safe)
6. After all workers finish, `.tmp` files are merged, deduplicated, and sorted into the final CSV

## Notes

- Each page has ~20 listings. The site currently has ~43,000 total listings (~2184 pages).
- Price values are in Vietnamese format (`4,68 tỷ` = 4.68 billion VND, `65,92 tr/m²` = 65.92 million VND/m²).
- Workers: 4-8 is safe. Higher counts are faster but risk getting blocked. Memory usage is ~150MB per worker (Chromium).
- If the crawl is interrupted, `.tmp` files preserve all progress. Just restart — stale `.tmp` files are cleaned up automatically before a new run.
