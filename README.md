# PDFApi.dev Documentation

<img src="https://pdfapi.dev/logo.svg" alt="pdfapi.dev logo" width="200"/>

PDFApi.dev is a powerful HTML to PDF conversion service that supports:
- CSS styling and custom fonts
- Headers and footers
- Images and other assets
- Multiple page formats
- Configurable margins and scaling

## Official Client Libraries

- [Node.js/TypeScript](https://github.com/pdfapi-dev-sdk/pdfapi-js-client)
- [Python](https://github.com/pdfapi-dev-sdk/pdfapi-python-client)
- [Java](https://github.com/pdfapi-dev-sdk/pdfapi-java-client)
- [Go](https://github.com/pdfapi-dev-sdk/pdfapi-go-client)

## REST API Documentation

### Authentication

All API requests require an API key that should be included in the `Api-Key` header:

```bash
Api-Key: your-api-key-here
```

### Converting HTML to PDF

The conversion process consists of three steps:
1. Initialize conversion
2. Upload assets (optional)
3. Perform conversion

#### 1. Initialize Conversion

```http
POST /api/conversions
Content-Type: application/json

{
    "format": "A4",
    "scale": 1.0,
    "margin": {
        "top": 20,
        "bottom": 20,
        "left": 20,
        "right": 20
    },
    "landscape": false,
    "headerFile": "header.html",
    "footerFile": "footer.html"
}
```

Parameters:
- `format` (optional): Page format (`A0`-`A6`, `Letter`, `Legal`, `Tabloid`)
- `scale` (optional): Scale factor for the output (default: 1.0)
- `margin` (optional): Page margins in pixels
- `landscape` (optional): Landscape orientation (default: false)
- `headerFile` (optional): Filename for the header template
- `footerFile` (optional): Filename for the footer template

Response:
```json
{
    "id": "conversion-id"
}
```

#### 2. Upload Assets (Optional)

```http
POST /api/conversions/{conversionId}/assets
Content-Type: multipart/form-data

asset=@file.css
```

Use this endpoint to upload:
- CSS files
- Images
- Fonts
- Header/footer templates

#### 3. Perform Conversion

```http
POST /api/conversions/{conversionId}/convert
Content-Type: multipart/form-data

index=@index.html
```

The main HTML file should be uploaded as 'index'. Upon successful conversion, the response will include a `Location` header pointing to the result URL.

Response headers:
```http
HTTP/1.1 200 OK
Location: /api/conversions/{conversionId}
```

#### 4. Get Result

The result can be fetched from the URL provided in the `Location` header of the conversion response:

```http
GET /api/conversions/{conversionId}
```

Response:
- Status 200: PDF file in response body
- Status 204: Conversion in progress, retry after delay

### Error Handling

The API uses standard HTTP status codes and returns detailed error information in the response body:

```json
{
    "type": "https://pdfapi.dev/errors/validation",
    "title": "Validation Error",
    "status": 400,
    "detail": "Detailed error message",
    "instance": "/api/conversions/123",
    "properties": {
        "additionalInfo": "value"
    }
}
```

Common error status codes:
- `400` Bad Request - Invalid parameters or input
- `401` Unauthorized - Missing or invalid API key
- `403` Forbidden - API key doesn't have required permissions
- `404` Not Found - Resource not found
- `429` Too Many Requests - Rate limit exceeded
- `500` Internal Server Error - Server error

### Example: Complete Conversion Flow

Here's a complete example using curl:

```bash
# 1. Initialize conversion
CONVERSION_ID=$(curl -X POST "https://api.pdfapi.dev/api/conversions" \
  -H "Api-Key: your-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "format": "A4",
    "margin": {"top": 20, "bottom": 20, "left": 20, "right": 20}
  }' | jq -r '.id')

# 2. Upload CSS file
curl -X POST "https://api.pdfapi.dev/api/conversions/$CONVERSION_ID/assets" \
  -H "Api-Key: your-api-key" \
  -F "asset=@style.css"

# 3. Upload image
curl -X POST "https://api.pdfapi.dev/api/conversions/$CONVERSION_ID/assets" \
  -H "Api-Key: your-api-key" \
  -F "asset=@image.png"

# 4. Perform conversion and get result URL
RESULT_URL=$(curl -X POST "https://api.pdfapi.dev/api/conversions/$CONVERSION_ID/convert" \
  -H "Api-Key: your-api-key" \
  -F "index=@index.html" \
  -i | grep -i "Location" | cut -d' ' -f2 | tr -d '\r')

# 5. Get result (with retries if needed)
curl -X GET "https://api.pdfapi.dev$RESULT_URL" \
  -H "Api-Key: your-api-key" \
  --output result.pdf
```

### Headers and Footers

Header and footer templates must be complete HTML documents. They can include special variables:
- `{page}` - Current page number
- `{pages}` - Total number of pages
- `{title}` - Document title
- `{url}` - Document URL

Example header template:
```html
<!DOCTYPE html>
<html>
<head>
    <style>
        body {
            font-family: Arial, sans-serif;
            font-size: 10px;
            margin: 0;
            padding: 8px;
        }
        .header {
            text-align: center;
            border-bottom: 1px solid #ddd;
        }
    </style>
</head>
<body>
    <div class="header">Page {page} of {pages}</div>
</body>
</html>
```

## Rate Limits and Quotas

- Free tier: 100 conversions per month
- Rate limit: 10 requests per minute
- Maximum input file size: 10MB
- Maximum output PDF size: 50MB

For higher limits, please contact our sales team or visit [pdfapi.dev](https://pdfapi.dev/pricing).

## Support

- Documentation: [docs.pdfapi.dev](https://docs.pdfapi.dev)
- Email: support@pdfapi.dev
- GitHub Issues: [Report a bug](https://github.com/pdfapi/pdfapi/issues) 
