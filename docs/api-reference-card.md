# eSign SDK v2.1.0 - API Reference Card

**Quick Reference for All API Endpoints**

Base URL: `http://localhost:8080/api/esign`

---

## Configuration Endpoints

### Set eSign Mode
```
POST /set-mode
Body: {"mode": "2.1" | "3.2"}
Response: {"success": true, "mode": "2.1", "modeName": "OTP-based Signing"}
```

### Get Configuration
```
GET /config
Response: {"success": true, "config": {...}}
```

### Reset Session
```
POST /reset
Response: {"success": true, "message": "Session reset successfully"}
```

---

## Signing Workflow Endpoints

### Step 1: Upload PDF
```
POST /step1/upload
Content-Type: multipart/form-data
Body: file=@document.pdf
Response: {"success": true, "step": 1, "fileName": "...", "documentId": "..."}
```

### Step 2: Set Signer Details
```
POST /step2/signer-details
Body: {
  "signerName": "John Doe",
  "docInfo": "Contract",
  "pageNo": 1,
  "x1": 350, "y1": 50, "x2": 550, "y2": 120
}
Response: {"success": true, "step": 2}
```

### Step 3: Generate Hash
```
POST /step3/generate-hash
Response: {"success": true, "step": 3, "hashAlgorithm": "SHA-256"}
```

### Step 4: Convert to Hex
```
POST /step4/convert-hex
Response: {"success": true, "step": 4, "hashHex": "..."}
```

### Step 5: Generate Transaction ID
```
POST /step5/generate-txn
Response: {"success": true, "step": 5, "transactionId": "UKC:public:..."}
```

### Step 6: Construct XML
```
POST /step6/construct-xml
Response: {"success": true, "step": 6, "xmlPreview": "..."}
```

### Step 7: Sign XML
```
POST /step7/sign-xml
Response: {"success": true, "step": 7, "signatureAlgorithm": "RSA-SHA256"}
```

### Step 8: Generate ESP Form
```
POST /step8/generate-esp-form
Response: {"success": true, "step": 8, "htmlForm": "...", "espUrl": "..."}
```

### Step 9: Embed Signature
```
POST /step9/embed-signature
Body: {"outputFileName": "signed.pdf"}
Response: {"success": true, "step": 9, "signedPdfPath": "..."}
```

---

## ESP Callback Endpoints

### ESP Response (eSign 2.1)
```
POST /esp-response
Body: esignresp=<XML response>
Response: HTML page
```

### ESP GetDocs (eSign 3.2)
```
POST /3.2/getdocs/
Body: eSignResponse=<XML response>
Response: "OK"
```

### ESP Callback (eSign 3.2)
```
GET/POST /3.2/callback/
Query: txnRef=<base64>, status=<status>
Response: HTML page
```

---

## Utility Endpoints

### Serve PDF Document
```
GET /documents/{documentId}.pdf
Response: PDF file (application/pdf)
```

### Get ESP Response Data
```
GET /esp-response-data
Response: {"success": true, "espResponseReceived": true, "responseXml": "..."}
```

### Embed Signature (Manual)
```
POST /embed-signature
Body: {"transactionId": "UKC:public:..."}
Response: {"success": true, "signedPdfPath": "..."}
```

---

## Complete Workflow Example

```bash
# Set mode
curl -X POST http://localhost:8080/api/esign/set-mode \
  -H "Content-Type: application/json" \
  -d '{"mode":"2.1"}'

# Upload PDF
curl -X POST http://localhost:8080/api/esign/step1/upload \
  -F "file=@document.pdf"

# Set signer details
curl -X POST http://localhost:8080/api/esign/step2/signer-details \
  -H "Content-Type: application/json" \
  -d '{"signerName":"John Doe","docInfo":"Contract"}'

# Execute steps 3-8
curl -X POST http://localhost:8080/api/esign/step3/generate-hash
curl -X POST http://localhost:8080/api/esign/step4/convert-hex
curl -X POST http://localhost:8080/api/esign/step5/generate-txn
curl -X POST http://localhost:8080/api/esign/step6/construct-xml
curl -X POST http://localhost:8080/api/esign/step7/sign-xml
curl -X POST http://localhost:8080/api/esign/step8/generate-esp-form

# After ESP signing, embed signature
curl -X POST http://localhost:8080/api/esign/step9/embed-signature \
  -H "Content-Type: application/json" \
  -d '{"outputFileName":"signed.pdf"}'
```

---

## Error Responses

All endpoints return errors in this format:
```json
{
  "success": false,
  "error": "Error message description"
}
```

**Common HTTP Status Codes:**
- `200` - Success
- `400` - Bad Request (invalid parameters)
- `404` - Not Found
- `500` - Internal Server Error

---

## Session Management

- All endpoints use HTTP sessions (cookies)
- Session persists across steps 1-9
- Use same browser/client for entire workflow
- Call `/reset` to clear session

---

## eSign Mode Differences

| Feature | eSign 2.1 | eSign 3.2 |
|---------|-----------|-----------|
| **Authentication** | Aadhaar OTP | eKYC |
| **Callback** | `/esp-response` | `/3.2/getdocs/` + `/3.2/callback/` |
| **Document URL** | Not required | Required (public URL) |
| **Response** | Synchronous | Asynchronous |

---

## Quick Tips

- **Always set mode first** (`/set-mode`)  
- **Follow steps in order** (1 → 2 → 3 → ... → 9)  
- **Use same session** for all steps  
- **Public callback URL** required for ESP  
- **Check session data** with `/esp-response-data`  

---

## Support

**Full Documentation**: eSign-sdk-technical-documentation.md  
**Quick Start**: quick-start-guide.md  
**Email**: support@www.esign.network  
**Website**: https://www.esign.network/esign-sdk

---

© 2025 Capricorn Technologies
