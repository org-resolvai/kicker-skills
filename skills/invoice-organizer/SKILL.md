---
name: invoice-organizer
description: Organize invoices and receipts for tax preparation. Extracts key info (vendor, amount, date, category) from invoice files and saves structured data to data/invoices/.
---

# Invoice Organizer Skill

Automatically organize invoices and receipts for the user's tax preparation.

## Steps

### 1. Receive Invoice Files

The user will provide invoice files in one of these ways:

- Upload an image (photo of a receipt)
- Upload a PDF
- Paste invoice text or details directly
- Share a URL to an online receipt

### 2. Extract Invoice Information

From each invoice, extract these fields:

| Field         | Description                         | Example             |
| ------------- | ----------------------------------- | ------------------- |
| `vendor`      | Company/merchant name               | Amazon, Starbucks   |
| `amount`      | Total amount with currency          | $49.99, ¥3,500      |
| `currency`    | ISO currency code                   | USD, CNY, JPY       |
| `date`        | Invoice/receipt date                | 2026-03-10          |
| `category`    | Expense category (see below)        | software            |
| `description` | Brief description of items/services | AWS monthly hosting |
| `tax`         | Tax amount if visible               | $4.50               |
| `payment`     | Payment method if visible           | Visa \*1234         |

#### Expense Categories

Use these standard categories:

- `software` — SaaS, licenses, app subscriptions
- `hardware` — computers, peripherals, devices
- `cloud` — cloud hosting, compute, storage
- `office` — office supplies, furniture
- `travel` — flights, hotels, transportation
- `meals` — business meals, food
- `telecom` — phone, internet, communication
- `professional` — consulting, legal, accounting
- `advertising` — marketing, ads, promotions
- `education` — courses, books, conferences
- `other` — anything that doesn't fit above

### 3. Save to Inbox

Save each invoice record as a JSON file in `data/invoices/`:

```bash
bash({ command: "mkdir -p data/invoices" })
```

**Filename format:** `{date}_{vendor_slug}_{amount}.json`

Example: `2026-03-10_amazon_49.99.json`

```
write({
  file_path: "data/invoices/2026-03-10_amazon_49.99.json",
  content: JSON.stringify({
    vendor: "Amazon",
    amount: 49.99,
    currency: "USD",
    date: "2026-03-10",
    category: "software",
    description: "AWS monthly hosting fee",
    tax: 4.50,
    payment: "Visa *1234",
    source: "receipt_photo",
    addedAt: "2026-03-12T08:30:00Z"
  }, null, 2)
})
```

### 4. If the User Provides an Image

For receipt photos or scanned invoices:

1. Describe the image content to extract text and amounts
2. Parse the extracted text for invoice fields
3. Save the JSON record to `data/invoices/`
4. If the original file is available in the workspace, copy it alongside the JSON:
   `data/invoices/2026-03-10_amazon_49.99.jpg`

### 5. Summary Report

When the user asks for a summary (e.g., "show my invoices", "tax summary"):

1. Read all JSON files from `data/invoices/`
2. Aggregate by category and month
3. Present a summary table:

```
Invoice Summary (2026 Q1):

Category      | Count | Total
------------- | ----- | --------
software      |     5 | $249.95
cloud         |     3 | $1,200.00
meals         |     8 | $320.00
------------- | ----- | --------
Total         |    16 | $1,769.95

Monthly Breakdown:
- January:  $580.00 (6 invoices)
- February: $650.50 (5 invoices)
- March:    $539.45 (5 invoices)
```

### 6. Export CSV

When the user asks to export (e.g., "export csv", "generate spreadsheet"):

1. Read all JSON files from `data/invoices/`
2. Generate a CSV file at `data/invoices/summary.csv`:

```
bash({ command: "mkdir -p data/invoices" })
```

```
write({
  file_path: "data/invoices/summary.csv",
  content: "date,vendor,amount,currency,tax,category,description,payment\n2026-03-10,Amazon,49.99,USD,4.50,software,AWS monthly hosting fee,Visa *1234\n..."
})
```

CSV columns: `date`, `vendor`, `amount`, `currency`, `tax`, `category`, `description`, `payment`

This file can be imported into accounting software (QuickBooks, Xero, etc.).

### 7. Rename Original Files

If the user provides raw files to organize, standardize filenames:

**Format:** `YYYY-MM-DD_Vendor_Invoice_Description.ext`

Example: `2026-03-10_Amazon_Invoice_AWS-Hosting.pdf`

```bash
bash({ command: "mv data/invoices/original-receipt.pdf 'data/invoices/2026-03-10_Amazon_Invoice_AWS-Hosting.pdf'" })
```

- Preserve original file — copy first if the user wants to keep the original name
- Handle ambiguous files by flagging them for the user to review

## Notes

- Always confirm extracted data with the user before saving, especially amounts
- Vendor names should be normalized (e.g., "AMAZON.COM" → "Amazon", "STARBUCKS #12345" → "Starbucks")
- Amounts should be stored as numbers, not strings
- If currency is ambiguous, ask the user
- Duplicate detection: check if a file with the same date + vendor + amount already exists before saving
- All paths are relative to the workspace root — `data/` is persisted across container restarts
