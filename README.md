# AI Invoice Extractor

Production-ready automation workflow that extracts structured data from invoice PDFs using OpenAI and n8n.

## Overview

Upload an invoice PDF → get structured JSON with invoice number, date, vendor, customer, line items, subtotal, tax, and total. All data persisted to Supabase.

## Features

- PDF invoice parsing (any format)
- AI-powered field extraction (GPT-4o-mini)
- Multi-currency support (EGP/USD/EUR/SAR/AED)
- Line-item extraction
- Persistent storage in Supabase
- REST API via webhook
- Language-agnostic (Arabic + English invoices supported)
- Automatic date normalization (YYYY-MM-DD)

## Extracted Fields

| Field | Type | Description |
|-------|------|-------------|
| `invoice_number` | string | Invoice ID |
| `invoice_date` | date | YYYY-MM-DD format |
| `vendor_name` | string | Seller/issuer |
| `customer_name` | string | Buyer |
| `subtotal` | number | Amount before tax |
| `tax` | number | Tax amount |
| `total` | number | Final amount |
| `currency` | string | EGP/USD/EUR/SAR/AED |
| `items` | array | Line items (description, quantity, unit_price, total) |
## System Architecture

## 1. Request & Routing

The Webhook receives a POST request with a PDF file (multipart/form-data). The binary payload is validated and passed to the extraction stage.

## 2. PDF Extraction

The `Extract PDF` node parses the binary PDF file and returns raw text content. This text is used as input for the AI extraction stage.

## 3. AI Extraction

The `AI Extractor` node sends the raw text to OpenAI GPT-4o-mini with a specialized system prompt that:
- Identifies invoice-specific fields (number, date, vendor, customer, amounts)
- Extracts line items with descriptions, quantities, and prices
- Normalizes dates to YYYY-MM-DD format
- Detects currency from symbols and context
- Returns strictly formatted JSON

## 4. JSON Parsing

The `Parse JSON` node:
- Cleans the AI response (strips markdown fences)
- Parses the JSON output
- Merges with metadata (file name, raw text snippet)
- Handles parsing errors with fallback defaults

## 5. Persistence

The `Save Invoice` node inserts the extracted data into the `bot_invoices` table in Supabase (PostgreSQL). Each row contains:
- Extracted fields (invoice_number, invoice_date, vendor_name, customer_name, subtotal, tax, total, currency)
- Line items as JSONB
- Metadata (file_name, raw_text, status, created_at)

## 6. Response

The `Format Response` node returns a clean JSON response to the client with:
- Status indicator
- Key extracted fields
- Item count
- Confirmation message

## Data Flow

Webhook → Extract PDF → AI Extractor → Parse JSON → Save to Supabase → Format Response → Client.
