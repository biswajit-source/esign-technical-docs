# eSign API Integration Guide

## Overview

The eSign API provides a REST interface for digitally signing PDF documents using Aadhaar-based authentication. The API supports both **eSign 2.1 (OTP-based)** and **eSign 3.2 (eKYC-based)** signing modes.

**Base URL:** `https://your-domain.com/api/v1/esign`

**Supported Formats:** XML (default) and JSON

---

## Table of Contents

1. [Authentication](#1-authentication)
2. [Sign Document Endpoint](#2-sign-document-endpoint)
3. [Request Parameters](#3-request-parameters)
4. [Signing Options](#4-signing-options)
5. [Multi-Page Signing](#5-multi-page-signing)
6. [Response Format](#6-response-format)
7. [Retrieve Signed Document](#7-retrieve-signed-document)
8. [Check Transaction Status](#8-check-transaction-status)
9. [Webhook Callbacks](#9-webhook-callbacks)
10. [Error Codes](#10-error-codes)
11. [Complete Examples](#11-complete-examples)

---

## 1. Authentication

All requests must include authentication credentials in the `auth` block.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `command` | String | Yes | Must be `"esign"` |
| `token` | String | Yes | Your API authentication token |
| `key` | String | Yes | Your API key |

---

## 2. Sign Document Endpoint

### POST `/api/v1/esign`

Initiates a document signing request.

**Headers:**

| Header | Value | Description |
|--------|-------|-------------|
| `Content-Type` | `application/xml` or `application/json` | Request format |
| `Accept` | `application/xml` or `application/json` | Response format |

---

## 3. Request Parameters

### XML Format

```xml
<?xml version="1.0" encoding="UTF-8"?>
<request>
    <auth>
        <command>esign</command>
        <token>your-api-token</token>
        <key>your-api-key</key>
    </auth>
    <parameter>
        <uploadpdf>
            <pdf64>BASE64_ENCODED_PDF_CONTENT</pdf64>
            <title>Document Title</title>
            <mode>online-aadhaar-otp</mode>
            <txn>CLIENT_TRANSACTION_ID</txn>
            <signername>Signer Name</signername>
            <callbackurl>https://your-callback-url.com/webhook</callbackurl>
            <option>
                <cood>350,50,550,120</cood>
                <pagenum>all</pagenum>
                <reason>Digital Signature</reason>
                <location>India</location>
            </option>
        </uploadpdf>
    </parameter>
</request>
```

### JSON Format

```json
{
    "auth": {
        "command": "esign",
        "token": "your-api-token",
        "key": "your-api-key"
    },
    "parameter": {
        "uploadpdf": {
            "pdf64": "BASE64_ENCODED_PDF_CONTENT",
            "title": "Document Title",
            "mode": "online-aadhaar-otp",
            "txn": "CLIENT_TRANSACTION_ID",
            "signername": "Signer Name",
            "callbackurl": "https://your-callback-url.com/webhook",
            "option": {
                "cood": "350,50,550,120",
                "pagenum": "all",
                "reason": "Digital Signature",
                "location": "India"
            }
        }
    }
}
```

### Parameter Reference

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `pdf64` | String | Yes | Base64 encoded PDF document |
| `pdfurl` | String | No | Alternative: URL to fetch PDF from (instead of pdf64) |
| `title` | String | Yes | Document title (max 50 characters) |
| `mode` | String | Yes | Signing mode (see [Modes](#signing-modes)) |
| `txn` | String | No | Your transaction ID for tracking |
| `signername` | String | Yes | Name of the signer |
| `ekycid` | String | Conditional | eKYC ID (required for `online-aadhaar-ekyc` mode) |
| `callbackurl` | String | No | Webhook URL for completion notification |
| `option` | Object | No | Signing options (see [Options](#4-signing-options)) |

### Signing Modes

| Mode | Description | Version |
|------|-------------|---------|
| `online-aadhaar-otp` | OTP-based Aadhaar authentication | eSign 2.1 |
| `online-aadhaar-ekyc` | eKYC-based Aadhaar authentication | eSign 3.2 |

---

## 4. Signing Options

The `option` block controls signature appearance and placement.

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `cood` | String | `350,50,550,120` | Signature box coordinates: `x1,y1,x2,y2` |
| `pagenum` | String | `1` | Page selection for signature (see [Multi-Page](#5-multi-page-signing)) |
| `reason` | String | `Digital Signature` | Reason for signing |
| `location` | String | `India` | Signing location |
| `customtext` | String | - | Custom text to display in signature |
| `greenticked` | String | `false` | Show green checkmark: `true` or `false` |
| `dateformat` | String | `dd-MM-yyyy HH:mm:ss` | Date format in signature |
| `enabletimestamp` | String | `false` | Enable trusted timestamp |
| `enableltv` | String | `false` | Enable Long Term Validation |
| `lockpdf` | String | `false` | Lock PDF after signing |

### Coordinate System

Coordinates define the signature box position on the PDF page:

```
(x1, y1) ─────────────────┐
    │                     │
    │   Signature Box     │
    │                     │
    └─────────────────────┘ (x2, y2)
```

- **Origin (0,0):** Bottom-left corner of the page
- **Units:** Points (1 point = 1/72 inch)
- **Standard A4 page:** 595 x 842 points

**Common Placements:**

| Position | Coordinates |
|----------|-------------|
| Bottom-right | `400,50,550,120` |
| Bottom-left | `50,50,200,120` |
| Top-right | `400,750,550,820` |
| Center-bottom | `200,50,400,120` |

---

## 5. Multi-Page Signing

The `pagenum` option supports flexible page selection for adding signature fields.

### Supported Formats

| Value | Description | Example Result (5-page doc) |
|-------|-------------|----------------------------|
| `1` | Single page | Page 1 only |
| `first` | First page | Page 1 |
| `last` | Last page | Page 5 |
| `all` | All pages | Pages 1, 2, 3, 4, 5 |
| `1-3` | Page range | Pages 1, 2, 3 |
| `1,3,5` | Specific pages | Pages 1, 3, 5 |
| `2-4` | Middle range | Pages 2, 3, 4 |

### Examples

**Sign all pages:**
```xml
<option>
    <pagenum>all</pagenum>
</option>
```

**Sign first and last pages:**
```xml
<option>
    <pagenum>1,5</pagenum>
</option>
```

**Sign pages 1 through 3:**
```xml
<option>
    <pagenum>1-3</pagenum>
</option>
```

### How Multi-Page Works

1. Primary signature is placed on the first selected page
2. All other selected pages get signature fields linked to the same cryptographic signature
3. All signature fields validate as a single signature in Adobe Reader
4. The signature box appears at the same coordinates on each page

---

## 6. Response Format

### Success Response

**XML:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<response>
    <success>OK</success>
    <command>esign</command>
    <requestid>1702819200123456789</requestid>
    <date>2024-12-17T10:30:00+05:30</date>
    <responsedata>
        <response>
            <txn>CLIENT_TRANSACTION_ID</txn>
            <reference>a1b2c3d4e5f6</reference>
            <redirecturl>https://api.example.com/api/v1/esign/redirect/a1b2c3d4e5f6</redirecturl>
            <getsigneddocurl>https://api.example.com/api/v1/esign/signed/a1b2c3d4e5f6</getsigneddocurl>
        </response>
    </responsedata>
</response>
```

**JSON:**
```json
{
    "success": "OK",
    "command": "esign",
    "requestid": "1702819200123456789",
    "date": "2024-12-17T10:30:00+05:30",
    "responsedata": {
        "response": {
            "txn": "CLIENT_TRANSACTION_ID",
            "reference": "a1b2c3d4e5f6",
            "redirecturl": "https://api.example.com/api/v1/esign/redirect/a1b2c3d4e5f6",
            "getsigneddocurl": "https://api.example.com/api/v1/esign/signed/a1b2c3d4e5f6"
        }
    }
}
```

### Response Fields

| Field | Description |
|-------|-------------|
| `success` | `OK` for success, `FAILED` for error |
| `reference` | Unique reference ID for tracking |
| `txn` | Your original transaction ID |
| `redirecturl` | URL to redirect user for Aadhaar authentication |
| `getsigneddocurl` | URL to retrieve signed document after completion |

### Error Response

**XML:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<response>
    <success>FAILED</success>
    <command>esign</command>
    <requestid>1702819200123456789</requestid>
    <date>2024-12-17T10:30:00+05:30</date>
    <e>
        <code>AUTH_001</code>
        <message>Invalid authentication token</message>
    </e>
</response>
```

---

## 7. Retrieve Signed Document

After signing is complete, retrieve the signed PDF using these endpoints.

### GET `/api/v1/esign/document/{reference}`

Returns the signed PDF file directly.

**Response:**
- Content-Type: `application/pdf`
- Body: Binary PDF data

**HTTP Status Codes:**

| Status | Description |
|--------|-------------|
| 200 | Success - PDF returned |
| 202 | Accepted - Signing still in progress |
| 404 | Not found - Invalid reference |
| 410 | Gone - Signing failed |

### GET `/api/v1/esign/signed/{reference}`

Returns signed document with metadata in XML/JSON format.

**Response (XML):**
```xml
<response>
    <success>OK</success>
    <command>esign</command>
    <responsedata>
        <response>
            <signpdf64>BASE64_ENCODED_SIGNED_PDF</signpdf64>
            <pdfurl>https://api.example.com/api/v1/esign/document/a1b2c3d4e5f6</pdfurl>
            <txn>CLIENT_TRANSACTION_ID</txn>
            <reference>a1b2c3d4e5f6</reference>
        </response>
    </responsedata>
</response>
```

---

## 8. Check Transaction Status

### GET `/api/v1/esign/status/{reference}`

Check the current status of a signing transaction.

**Response (XML):**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<statusresponse>
    <reference>a1b2c3d4e5f6</reference>
    <txn>CLIENT_TRANSACTION_ID</txn>
    <status>COMPLETED</status>
    <signername>John Doe</signername>
    <createdat>2024-12-17T10:30:00</createdat>
    <updatedat>2024-12-17T10:35:00</updatedat>
    <completedat>2024-12-17T10:35:00</completedat>
    <getsigneddocurl>https://api.example.com/api/v1/esign/signed/a1b2c3d4e5f6</getsigneddocurl>
</statusresponse>
```

### Status Values

| Status | Description |
|--------|-------------|
| `PENDING` | Request created, awaiting user action |
| `PROCESSING` | ESP response received, embedding signature |
| `COMPLETED` | Signing completed successfully |
| `FAILED` | Signing failed (check error details) |

---

## 9. Webhook Callbacks

If `callbackurl` is provided, a POST request is sent when signing completes or fails.

### Success Callback

```xml
<callback>
    <status>COMPLETED</status>
    <reference>a1b2c3d4e5f6</reference>
    <txn>CLIENT_TRANSACTION_ID</txn>
    <signername>John Doe</signername>
    <getsigneddocurl>https://api.example.com/api/v1/esign/signed/a1b2c3d4e5f6</getsigneddocurl>
    <completedat>2024-12-17T10:35:00</completedat>
</callback>
```

### Failure Callback

```xml
<callback>
    <status>FAILED</status>
    <reference>a1b2c3d4e5f6</reference>
    <txn>CLIENT_TRANSACTION_ID</txn>
    <error>
        <code>ESP_001</code>
        <message>User cancelled the signing process</message>
    </error>
</callback>
```

---

## 10. Error Codes

### Authentication Errors

| Code | Description |
|------|-------------|
| `AUTH_001` | Invalid or missing authentication token |
| `AUTH_002` | Invalid API key |
| `AUTH_003` | Token expired |

### Validation Errors

| Code | Description |
|------|-------------|
| `VAL_001` | Missing required field |
| `VAL_002` | Invalid PDF data |
| `VAL_003` | Invalid mode specified |
| `VAL_004` | Invalid coordinates format |
| `VAL_005` | PDF exceeds size limit |

### Processing Errors

| Code | Description |
|------|-------------|
| `PROC_001` | PDF data is required |
| `PROC_002` | Unsupported signing mode |
| `PROC_003` | Failed to generate eSign request |
| `PROC_999` | General processing error |

### ESP Errors

| Code | Description |
|------|-------------|
| `ESP_001` | User cancelled signing |
| `ESP_002` | OTP verification failed |
| `ESP_003` | Aadhaar authentication failed |
| `ESP_004` | Session expired |
| `ESP_911` | ESP signature verification failed |

### System Errors

| Code | Description |
|------|-------------|
| `SYS_001` | Internal system error |
| `SYS_002` | Service temporarily unavailable |

---

## 11. Complete Examples

### Example 1: Simple Single-Page Signing (OTP Mode)

**Request:**
```bash
curl -X POST https://api.example.com/api/v1/esign \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{
    "auth": {
        "command": "esign",
        "token": "your-token",
        "key": "your-key"
    },
    "parameter": {
        "uploadpdf": {
            "pdf64": "'$(base64 -w0 document.pdf)'",
            "title": "Employment Contract",
            "mode": "online-aadhaar-otp",
            "txn": "TXN123456",
            "signername": "Rahul Sharma",
            "callbackurl": "https://myapp.com/webhook/esign"
        }
    }
}'
```

### Example 2: Multi-Page Signing (All Pages)

**Request (XML):**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<request>
    <auth>
        <command>esign</command>
        <token>your-token</token>
        <key>your-key</key>
    </auth>
    <parameter>
        <uploadpdf>
            <pdf64>JVBERi0xLjQKJe...</pdf64>
            <title>Multi-Page Agreement</title>
            <mode>online-aadhaar-otp</mode>
            <txn>TXN789012</txn>
            <signername>Priya Patel</signername>
            <option>
                <cood>400,50,550,120</cood>
                <pagenum>all</pagenum>
                <reason>Agreement Acceptance</reason>
                <location>Mumbai, India</location>
            </option>
        </uploadpdf>
    </parameter>
</request>
```

### Example 3: eKYC Mode (eSign 3.2)

**Request (JSON):**
```json
{
    "auth": {
        "command": "esign",
        "token": "your-token",
        "key": "your-key"
    },
    "parameter": {
        "uploadpdf": {
            "pdf64": "JVBERi0xLjQKJe...",
            "title": "KYC Document",
            "mode": "online-aadhaar-ekyc",
            "ekycid": "EKYC123456789",
            "txn": "TXN345678",
            "signername": "Amit Kumar",
            "option": {
                "cood": "350,700,500,770",
                "pagenum": "first",
                "reason": "KYC Verification"
            }
        }
    }
}
```

### Example 4: Custom Page Selection

**Sign only pages 1, 3, and 5:**
```json
{
    "parameter": {
        "uploadpdf": {
            "pdf64": "...",
            "title": "Contract",
            "mode": "online-aadhaar-otp",
            "signername": "User",
            "option": {
                "pagenum": "1,3,5"
            }
        }
    }
}
```

**Sign pages 2 through 4:**
```json
{
    "parameter": {
        "uploadpdf": {
            "pdf64": "...",
            "title": "Contract",
            "mode": "online-aadhaar-otp",
            "signername": "User",
            "option": {
                "pagenum": "2-4"
            }
        }
    }
}
```

---

## Integration Flow

```
┌─────────────┐     1. POST /api/v1/esign      ┌─────────────┐
│   Client    │ ─────────────────────────────▶ │  eSign API  │
│ Application │                                │   Server    │
└─────────────┘                                └─────────────┘
       │                                              │
       │         2. Response with redirecturl         │
       │ ◀─────────────────────────────────────────── │
       │                                              │
       │    3. Redirect user to redirecturl           │
       ▼                                              │
┌─────────────┐                                       │
│    User     │                                       │
│   Browser   │                                       │
└─────────────┘                                       │
       │                                              │
       │    4. Auto-redirect to ESP Portal            │
       ▼                                              │
┌─────────────┐                                       │
│     ESP     │                                       │
│   Portal    │                                       │
│  (Aadhaar)  │                                       │
└─────────────┘                                       │
       │                                              │
       │    5. User completes OTP/eKYC                │
       │                                              │
       │    6. ESP callback with signature            │
       │ ─────────────────────────────────────────▶   │
       │                                              │
       │         7. Redirect to success page          │
       │ ◀─────────────────────────────────────────── │
       │                                              │
       │                      8. Webhook notification │
       │                      (if callbackurl set)    │
┌─────────────┐ ◀──────────────────────────────────── │
│   Client    │                                       │
│   Server    │                                       │
└─────────────┘                                       │
       │                                              │
       │    9. GET /api/v1/esign/document/{ref}       │
       │ ─────────────────────────────────────────▶   │
       │                                              │
       │         10. Signed PDF returned              │
       │ ◀─────────────────────────────────────────── │
```

---

## Best Practices

1. **Always store the `reference`** - This is your key to retrieve the signed document
2. **Implement webhook callback** - For real-time completion notifications
3. **Handle all status codes** - Including 202 (still processing) when polling
4. **Use appropriate coordinates** - Test signature placement with sample documents
5. **Set meaningful transaction IDs** - Helps with tracking and debugging
6. **Validate PDF before sending** - Ensure the PDF is not corrupted or password-protected

---

## Support

For technical support or questions, contact your integration team or refer to the SDK documentation.

**Version:** 1.0  
**Last Updated:** December 2025