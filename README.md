# Attachment Analysis Engine

An AI-powered **n8n workflow** that automatically monitors incoming Gmail messages with attachments, detects the attachment type, extracts or analyzes its content, normalizes the results, and prepares the information for logging and notification.

The workflow currently supports **PDF documents, XLSX spreadsheets, and image files**.

## Overview

The Attachment Analysis Engine is designed to reduce manual document review by automatically processing business-related email attachments.

The workflow follows this general flow:

```text
Gmail
  ↓
Detect Attachment Type
  ↓
Route by MIME Type
  ├── PDF → Extract Text → AI Document Analysis
  ├── XLSX → Extract Rows → Combine Data → AI Spreadsheet Analysis
  └── Image → AI Vision Analysis
  ↓
Normalize Output
  ↓
Airtable Logging
  ↓
Telegram Notification
```

## Features

* Automatically monitors unread Gmail messages containing attachments
* Downloads email attachments for processing
* Detects file type and MIME type
* Routes files using deterministic n8n Switch logic
* Extracts text from PDF documents
* Extracts and combines spreadsheet data from XLSX files
* Uses AI vision for image analysis
* Uses OpenAI models for document and spreadsheet classification
* Produces structured AI output
* Normalizes results into a consistent format
* Captures sender information
* Supports Airtable logging
* Supports Telegram notifications

## Supported File Types

### PDF

PDF files are processed through n8n's file extraction node.

The extracted text is sent to an AI document analysis agent, which returns structured information including:

```json
{
  "content_nature": "",
  "file_type": "",
  "summary": ""
}
```

The AI focuses on identifying the overall business purpose of the document rather than listing every individual detail.

### XLSX

Excel spreadsheets are extracted into individual data items.

The workflow then combines those items into one consolidated representation before sending the content to the AI analyzer.

The spreadsheet AI evaluates the spreadsheet as a **single business document** instead of separately summarizing every row.

### Images

Image attachments are sent directly to an OpenAI image analysis node.

The model analyzes the visual content and generates a concise description of the image and its likely business purpose.

## Workflow Architecture

### 1. Gmail Trigger

The workflow monitors Gmail every minute for:

```text
Unread messages
+
Messages containing attachments
```

Attachments are automatically downloaded into n8n binary data.

### 2. Detect File Type

A Code node reads the attachment metadata and extracts:

```json
{
  "fileName": "...",
  "fileExtension": "...",
  "mimeType": "..."
}
```

The original binary attachment is preserved for downstream processing.

### 3. File Type Router

An n8n Switch node uses the MIME type to determine which processing path should execute.

Routing rules include:

```text
application/pdf
→ PDF processing

application/vnd.openxmlformats-officedocument.spreadsheetml.sheet
→ XLSX processing

image/*
→ Image processing
```

This keeps file routing deterministic instead of asking an AI model to decide which processor should be used.

## PDF Processing

The PDF branch follows:

```text
Extract from PDF
      ↓
Analyze Document
      ↓
Structured Output Parser
      ↓
Normalize PDF Result
```

The AI identifies the document's business nature and produces a short summary.

The normalization node standardizes the result into:

```json
{
  "content_nature": "...",
  "summary": "...",
  "file_type": "...",
  "sender": "..."
}
```

Fallback values are used when a field is unavailable.

## Spreadsheet Processing

The XLSX branch follows:

```text
Extract from XLSX
      ↓
Combine Rows
      ↓
Analyze Spreadsheet
      ↓
Structured Output Parser
      ↓
Normalize XLSX Result
```

The `Combine Rows` Code node converts all extracted spreadsheet items into one combined text representation.

This allows the AI model to analyze the spreadsheet as a complete dataset rather than receiving disconnected rows.

## Image Processing

The image branch follows:

```text
Analyze Image
      ↓
Normalize Image Result
```

The normalization logic handles different possible AI response structures and extracts the final textual description.

The standardized result contains:

```json
{
  "content_nature": "Image / Visual content",
  "summary": "...",
  "file_type": "...",
  "sender": "..."
}
```

## Airtable Integration

Processed attachment information can be stored in an Airtable table using the following fields:

| Field          | Purpose                            |
| -------------- | ---------------------------------- |
| Sender         | Email sender                       |
| Content Nature | Business purpose or classification |
| File Type      | Original attachment extension      |
| Summary        | AI-generated summary               |

The Airtable node in the supplied workflow is currently disabled and can be enabled after the appropriate Airtable credentials and table configuration are available.

## Telegram Notification

After processing and logging the attachment, the workflow can send a Telegram notification containing information such as:

```text
Sender
File Type
Summary
Attachment Nature
```

This provides a quick way to review incoming business documents without manually opening every email attachment.

## AI Output Structure

For document and spreadsheet analysis, structured output is enforced using an n8n Structured Output Parser.

Example:

```json
{
  "content_nature": "Invoice",
  "file_type": "pdf",
  "summary": "A supplier invoice containing billing and payment information."
}
```

Normalization nodes are used after AI processing so downstream services receive a predictable schema.

## Tech Stack

* n8n
* Gmail
* OpenAI
* GPT-5 Mini
* LangChain nodes for n8n
* JavaScript
* JSON
* Airtable
* Telegram Bot API

## Key Concepts Demonstrated

This project demonstrates:

* Workflow automation
* API and service integration
* Binary file handling
* MIME type detection
* Deterministic workflow routing
* AI-assisted document analysis
* AI vision
* Structured AI outputs
* Data normalization
* Multi-format document processing
* External database integration
* Automated notifications

## Example Use Cases

The workflow can be adapted for processing:

* Invoices
* Receipts
* Purchase orders
* Quotations
* Reports
* Customer lists
* Financial spreadsheets
* Inventory spreadsheets
* Business documents
* Screenshots
* Scanned documents
* Other email-based attachments

## Current Limitations

The current workflow explicitly routes:

* PDF
* XLSX
* Image files

Other file formats would require additional Switch routes and processing nodes.

The workflow also assumes the primary attachment is available as:

```text
attachment_0
```

Additional logic would be required to process multiple attachments from the same email independently.

## Security Notes

Credentials such as Gmail OAuth credentials, OpenAI API credentials, Airtable tokens, and Telegram bot credentials should be configured directly inside n8n.

Do not commit API keys, access tokens, `.env` files, private keys, or other secrets to a public GitHub repository.

Exported n8n workflows should be reviewed before publishing to ensure no sensitive configuration or identifying information is exposed.

## Future Improvements

Potential improvements include:

* Multiple-attachment processing
* DOCX and CSV support
* Additional document classifications
* Confidence scoring
* Human review for uncertain classifications
* Duplicate document detection
* Error handling and retry mechanisms
* Database-based processing history
* Automatic document routing to different departments
* RAG-based document interpretation
* More advanced security and validation rules

## Project Purpose

This project was built to demonstrate how deterministic workflow automation and AI analysis can work together.

n8n controls the ingestion, routing, extraction, normalization, storage, and notification logic, while AI models are used specifically for tasks that require understanding unstructured document or image content.

The result is a reusable attachment-processing pipeline that can serve as the foundation for document intake, operations automation, CRM workflows, and AI-assisted business document processing.
