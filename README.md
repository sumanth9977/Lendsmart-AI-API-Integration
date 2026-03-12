# LendSmart AI - Document Upload & Analysis API

API integration for uploading documents to LendSmart AI and retrieving AI-powered analysis results.

## Quick Start

### Prerequisites

- [Postman](https://www.postman.com/downloads/) (desktop app or web)
- **Invoke URL** and **API Key** provided by the LendSmart AI admin

### Setup

1. **Import the Postman Collection**
   - Open Postman
   - Click **Import** (top-left)
   - Drag and drop `API_Gateway_Postman_Collection.json` or click **Upload Files** and select it
   - The collection **"LendSmart AI — Document Upload & Analysis API"** will appear in your sidebar

2. **Configure Variables**
   - Click on the imported collection name in the sidebar
   - Go to the **Variables** tab
   - Update these two variables with the values provided by your admin:

   | Variable | Description |
   |----------|-------------|
   | `invoke_url` | The API Gateway URL provided by admin |
   | `api_key` | Your API authentication key |

   - Click **Save** (Ctrl+S)

### Usage

#### Step 1 — Upload a Document

1. Expand **Step 1 — Upload Document** in the sidebar
2. Click **Upload Document (Form Data)**
3. In the **Body** tab, click **Select Files** next to the `file` field
4. Choose a document (PDF, DOC, DOCX, TXT, JPG, JPEG, PNG)
5. Click **Send**
6. You will receive a `202 Accepted` response with a `case_id`:
   ```json
   {
     "status": "accepted",
     "message": "File uploaded. Analysis running in background.",
     "case_id": "BHA-20260312-abc12345",
     "poll_url": "/api/v1/cases/BHA-20260312-abc12345",
     "poll_interval_seconds": 10
   }
   ```
   The `case_id` is automatically saved for the next step.

#### Step 2 — Get Analysis Results

1. Expand **Step 2 — Poll Analysis Results** in the sidebar
2. Click **Get Analysis Results**
3. Click **Send**
4. If analysis is still processing, the `analysis` field will be empty — **wait 10 seconds and send again**
5. Once complete, the response includes the full AI analysis:
   ```json
   {
     "case_id": "BHA-20260312-abc12345",
     "analysis": {
       "document_type": "credit_agreement",
       "customer_name": "ACME Corporation",
       "key_terms": [...],
       "important_covenants": [...]
     }
   }
   ```

Analysis typically takes **1–5 minutes** depending on document size and complexity.

## Sample Documents

The `sample_documents/` folder contains test files you can use to verify the integration:

| File | Type | Description |
|------|------|-------------|
| `Drawdown Intent Assurant.pdf` | PDF | Drawdown intent document |
| `Drawdown Intent Assurant.docx` | DOCX | Same document in Word format |
| `Global Mock Scanned Drawdown.pdf` | PDF | Scanned drawdown document (image-based PDF) |
| `Global Mock Image.png` | PNG | Document image for OCR testing |

## API Reference

### POST `/api/v1/upload` — Upload Document

Upload a document for AI analysis. Returns immediately while analysis runs in the background.

**Response: `202 Accepted`**
```json
{
  "status": "accepted",
  "message": "File uploaded. Analysis running in background.",
  "case_id": "BHA-20260312-abc12345",
  "file_id": "FILE20260312120000abc",
  "original_filename": "agreement.pdf",
  "poll_url": "/api/v1/cases/BHA-20260312-abc12345",
  "poll_interval_seconds": 10,
  "case_details": {
    "case_id": "BHA-20260312-abc12345",
    "subject": "agreement.pdf",
    "status": "New",
    "priority": "Medium",
    "created_at": "2026-03-12T10:30:00+05:30",
    "file_count": 1
  }
}
```

### GET `/api/v1/cases/{case_id}` — Get Analysis Results

Retrieve case details and AI analysis results. Poll every 10 seconds until analysis is complete.

**Response: `200 OK`**
```json
{
  "case_id": "BHA-20260312-abc12345",
  "timestamp": "2026-03-12T10:35:00+05:30",
  "case_details": {
    "case_id": "BHA-20260312-abc12345",
    "subject": "agreement.pdf",
    "status": "New",
    "priority": "Medium",
    "file_count": 1
  },
  "files": [
    {
      "file_id": "FILE20260312120000abc",
      "original_filename": "agreement.pdf",
      "file_size": 1024000,
      "mime_type": "application/pdf"
    }
  ],
  "analysis": {
    "document_type": "credit_agreement",
    "customer_name": "ACME Corporation",
    "agreement_effective_date": "2026-01-15",
    "agreement_purpose": "Senior Secured Revolving Credit Facility",
    "parties": [
      {"name": "ACME Corporation", "role": "Borrower"},
      {"name": "First National Bank", "role": "Administrative Agent"}
    ],
    "key_terms": [
      {"term": "Facility Amount", "description": "$50,000,000"},
      {"term": "Maturity Date", "description": "January 15, 2031"},
      {"term": "Interest Rate", "description": "SOFR + 2.50%"}
    ],
    "important_covenants": [
      {
        "term": "Total Leverage Ratio",
        "description": "Total Funded Debt to EBITDA shall not exceed 3.50:1.00",
        "covenant_type": "financial",
        "severity": "high"
      }
    ]
  }
}
```

> If `analysis` is an empty object `{}`, analysis is still processing — poll again after 10 seconds.

### Authentication

All requests require the `X-API-Key` header. This is automatically included by the Postman collection when you set the `api_key` variable.

### Supported File Types

PDF, DOC, DOCX, TXT, JPG, JPEG, PNG, MD

## Troubleshooting

| Issue | Solution |
|-------|----------|
| `403 Forbidden` | Check that your `api_key` variable is set correctly |
| `404 Not Found` | Verify the `invoke_url` variable matches the URL provided by admin |
| Empty `analysis` in response | Analysis is still running — poll again after 10 seconds |
| `500 Internal Server Error` | Contact the LendSmart AI admin |
