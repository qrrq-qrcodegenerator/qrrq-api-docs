# QrrQ API Documentation

Official API documentation for [QrrQ](https://qrrq.io) — Dynamic QR Code Generator & Analytics Platform.

Create, manage, and track QR codes programmatically. Supports 16 QR types, custom designs, scan analytics, and advanced targeting rules.

## Quick Start

```bash
curl -X POST https://qrrq.io/api/v1/qr \
  -H "X-API-Key: your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "url",
    "data": {"url": "https://example.com"},
    "title": "My Website QR",
    "is_dynamic": true
  }'
```

## Authentication

All API requests require an API key. Include it in one of these headers:

```
X-API-Key: your_api_key
```
```
Authorization: Bearer your_api_key
```

Get your API key from [Dashboard > API Keys](https://qrrq.io/dashboard/api-keys). API access requires a paid plan.

## Base URL

```
https://qrrq.io/api/v1
```

## Rate Limits

| Plan | Requests | Period |
|------|----------|--------|
| Pro | 100 | per hour |
| Business | 500 | per hour |
| Enterprise | Custom | Custom |

Rate limit headers are included in every response:
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 97
```

---

## Endpoints

### Create QR Code

```
POST /api/v1/qr
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `type` | string | No | QR type (default: `url`). See [Supported Types](#supported-types) |
| `data` | object | **Yes** | Type-specific data. See [Supported Types](#supported-types) |
| `title` | string | No | QR code title (max 255 chars) |
| `is_dynamic` | boolean | No | `true` for trackable/editable QR codes |
| `design` | object | No | Visual customization. See [Design Options](#design-options) |
| `rules` | object | No | Targeting rules. See [Rules & Targeting](#rules--targeting) |
| `password` | string | No | Password-protect the QR code |
| `custom_slug` | string | No | Custom short URL slug (e.g. `my-brand`) |
| `campaign_start_at` | string | No | Campaign start date (ISO 8601) |
| `campaign_end_at` | string | No | Campaign end date (ISO 8601) |

**Example:**

```bash
curl -X POST https://qrrq.io/api/v1/qr \
  -H "X-API-Key: your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "url",
    "data": {"url": "https://example.com"},
    "title": "My Website QR",
    "is_dynamic": true,
    "design": {
      "foreground": "#1a5c53",
      "background": "#ffffff",
      "pattern": "rounded",
      "cornerStyle": "rounded",
      "logo": "globe",
      "frame": "bottom-text",
      "frameText": "SCAN ME"
    },
    "rules": {
      "scan_limit": 5000,
      "utm": {
        "source": "print",
        "medium": "qrcode",
        "campaign": "spring2026"
      }
    }
  }'
```

**Response:**

```json
{
  "success": true,
  "data": {
    "id": "12345",
    "type": "url",
    "title": "My Website QR",
    "is_dynamic": true,
    "short_code": "abc123",
    "short_url": "https://qrrq.io/r/abc123",
    "image_url": "https://qrrq.io/api/v1/qr/12345/image",
    "status": "active",
    "created_at": "2026-04-15T10:30:00Z"
  }
}
```

---

### List QR Codes

```
GET /api/v1/qr
```

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | integer | 1 | Page number |
| `per_page` | integer | 20 | Items per page (max 50) |
| `type` | string | — | Filter by QR type |
| `status` | string | — | Filter: `active`, `paused`, `expired` |

```bash
curl https://qrrq.io/api/v1/qr?page=1&per_page=10&type=url \
  -H "X-API-Key: your_api_key"
```

---

### Get QR Code

```
GET /api/v1/qr/{id}
```

```bash
curl https://qrrq.io/api/v1/qr/12345 \
  -H "X-API-Key: your_api_key"
```

---

### Update QR Code

```
PUT /api/v1/qr/{id}
```

Only dynamic QR codes can be updated. All parameters are optional — send only the fields you want to change.

| Parameter | Type | Description |
|-----------|------|-------------|
| `title` | string | Update title |
| `status` | string | `active` or `paused` |
| `data` | object | Update QR content |
| `design` | object | Update visual design |
| `rules` | object | Update targeting rules |
| `password` | string | Set/update password |
| `custom_slug` | string | Change short URL slug |

```bash
curl -X PUT https://qrrq.io/api/v1/qr/12345 \
  -H "X-API-Key: your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Updated QR",
    "status": "paused",
    "data": {"url": "https://new-url.com"}
  }'
```

---

### Delete QR Code

```
DELETE /api/v1/qr/{id}
```

```bash
curl -X DELETE https://qrrq.io/api/v1/qr/12345 \
  -H "X-API-Key: your_api_key"
```

---

### Get QR Image

```
GET /api/v1/qr/{id}/image
```

Returns the QR code as a PNG image. **No authentication required** — can be used directly in `<img>` tags.

```html
<img src="https://qrrq.io/api/v1/qr/12345/image" alt="QR Code">
```

---

### Get Analytics

```
GET /api/v1/qr/{id}/analytics
```

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `days` | integer | 30 | Number of days to analyze |

Analytics data varies by plan level:

| Plan Level | Data Included |
|------------|---------------|
| **Basic** | Daily scan counts, total/unique scans |
| **Standard** | + Countries, devices, browsers |
| **Advanced** | + Cities, languages, referrers |

```bash
curl https://qrrq.io/api/v1/qr/12345/analytics?days=30 \
  -H "X-API-Key: your_api_key"
```

---

### Analytics Overview

```
GET /api/v1/analytics/overview
```

Get aggregated analytics across all your QR codes.

```bash
curl https://qrrq.io/api/v1/analytics/overview?days=30 \
  -H "X-API-Key: your_api_key"
```

---

### User Profile

```
GET /api/v1/user/profile
PUT /api/v1/user/profile
```

```bash
curl -X PUT https://qrrq.io/api/v1/user/profile \
  -H "X-API-Key: your_api_key" \
  -H "Content-Type: application/json" \
  -d '{"name": "John Doe", "language": "en"}'
```

---

## Supported Types

| Type | Data Fields | Example |
|------|-------------|---------|
| `url` | `url` | `{"url": "https://example.com"}` |
| `text` | `text` | `{"text": "Hello World"}` |
| `email` | `email`, `subject`, `body` | `{"email": "hi@me.com", "subject": "Hello"}` |
| `phone` | `phone` | `{"phone": "+1234567890"}` |
| `sms` | `phone`, `message` | `{"phone": "+1234567890", "message": "Hi"}` |
| `whatsapp` | `phone`, `message` | `{"phone": "+1234567890", "message": "Hi"}` |
| `wifi` | `ssid`, `password`, `encryption` | `{"ssid": "MyWifi", "password": "pass", "encryption": "WPA"}` |
| `vcard` | `firstName`, `lastName`, `phone`, `email`, `company`, `website`, `address`, `title` | `{"firstName": "John", "lastName": "Doe"}` |
| `location` | `latitude`, `longitude`, `query` | `{"latitude": "48.8566", "longitude": "2.3522"}` |
| `social` | `url` | `{"url": "https://instagram.com/me"}` |
| `event` | `title`, `location`, `start`, `end`, `description` | `{"title": "Meeting", "start": "2026-04-01T10:00"}` |
| `crypto` | `address`, `currency`, `amount` | `{"currency": "bitcoin", "address": "bc1q..."}` |
| `pdf` | `file_token` | `{"file_token": "abc123..."}` |
| `file` | `file_token` | `{"file_token": "abc123..."}` |
| `app` | `ios_url`, `android_url`, `ipad_url` | `{"ios_url": "https://apps.apple.com/...", "android_url": "https://play.google.com/..."}` |
| `multiurl` | `urls` | `{"urls": "[{\"url\":\"https://a.com\"},{\"url\":\"https://b.com\"}]"}` |

> **Note:** `pdf` and `file` types require uploading a file first via the dashboard. The `file_token` is returned after upload.

---

## Design Options

Customize the visual appearance of your QR codes.

| Parameter | Type | Values |
|-----------|------|--------|
| `foreground` | string | Hex color — `#000000` |
| `background` | string | Hex color — `#ffffff` |
| `pattern` | string | `square`, `rounded`, `dots`, `classy`, `classy-rounded`, `extra-rounded` |
| `cornerStyle` | string | `square`, `rounded`, `extra-rounded`, `classy`, `dot` |
| `gradient` | boolean | Enable gradient foreground |
| `gradientColor2` | string | Hex color for gradient end |
| `logo` | string | Built-in logo name (e.g. `instagram`, `facebook`, `globe`, `wifi`) |
| `logo_url` | string | Custom logo URL (HTTPS only) |
| `logoSize` | number | Logo size ratio (0.1 — 0.3) |
| `logoShape` | string | `square`, `rounded`, `circle` |
| `frame` | string | `none`, `bottom-text`, `top-text`, `scan-me` |
| `frameText` | string | Text inside the frame |
| `frameColor` | string | Hex color for frame |
| `margin` | integer | QR margin in modules (0 — 20) |

---

## Rules & Targeting

Dynamic QR codes support advanced targeting rules.

```json
{
  "rules": {
    "scan_limit": 5000,
    "campaign": {
      "start": "2026-04-01T00:00",
      "end": "2026-12-31T23:59"
    },
    "utm": {
      "source": "print",
      "medium": "qrcode",
      "campaign": "spring2026"
    },
    "device_targets": {
      "ios": "https://apps.apple.com/app/...",
      "android": "https://play.google.com/store/apps/..."
    },
    "country_targets": {
      "TR": "https://example.com/tr",
      "DE": "https://example.com/de"
    }
  }
}
```

| Rule | Description |
|------|-------------|
| `scan_limit` | Maximum number of scans before QR expires |
| `campaign.start/end` | Campaign date range (ISO 8601) |
| `utm` | UTM parameters appended to redirect URL |
| `device_targets` | Device-specific redirect URLs (ios/android/desktop) |
| `country_targets` | Country-specific redirect URLs (ISO 3166-1 alpha-2) |
| `language_targets` | Language-specific redirect URLs |
| `ab_test` | A/B test split with percentage weights |

---

## Error Handling

All errors return a consistent JSON format:

```json
{
  "success": false,
  "error": "Human-readable error message"
}
```

| HTTP Status | Meaning |
|-------------|---------|
| `200` | Success |
| `400` | Bad request — invalid parameters |
| `401` | Unauthorized — missing or invalid API key |
| `403` | Forbidden — plan feature not available |
| `404` | Not found — resource doesn't exist or not owned by you |
| `422` | Validation error — data format invalid |
| `429` | Rate limit exceeded |
| `500` | Server error |

---

## Code Examples

### Python

```python
import requests

API_KEY = "your_api_key"
BASE = "https://qrrq.io/api/v1"
headers = {"X-API-Key": API_KEY, "Content-Type": "application/json"}

# Create a QR code
resp = requests.post(f"{BASE}/qr", json={
    "type": "url",
    "data": {"url": "https://example.com"},
    "title": "My QR",
    "is_dynamic": True
}, headers=headers)

qr = resp.json()["data"]
print(f"QR URL: {qr['short_url']}")
print(f"Image: {qr['image_url']}")
```

### Node.js

```javascript
const API_KEY = "your_api_key";
const BASE = "https://qrrq.io/api/v1";

const res = await fetch(`${BASE}/qr`, {
  method: "POST",
  headers: {
    "X-API-Key": API_KEY,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    type: "url",
    data: { url: "https://example.com" },
    title: "My QR",
    is_dynamic: true,
  }),
});

const { data } = await res.json();
console.log(`QR URL: ${data.short_url}`);
```

### PHP

```php
$apiKey = 'your_api_key';
$ch = curl_init('https://qrrq.io/api/v1/qr');
curl_setopt_array($ch, [
    CURLOPT_POST => true,
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_HTTPHEADER => [
        'X-API-Key: ' . $apiKey,
        'Content-Type: application/json',
    ],
    CURLOPT_POSTFIELDS => json_encode([
        'type' => 'url',
        'data' => ['url' => 'https://example.com'],
        'title' => 'My QR',
        'is_dynamic' => true,
    ]),
]);

$response = json_decode(curl_exec($ch), true);
echo "QR URL: " . $response['data']['short_url'];
```

---

## Support

- **Dashboard**: [qrrq.io/dashboard](https://qrrq.io/dashboard)
- **Email**: support@qrrq.io
- **Website**: [qrrq.io](https://qrrq.io)

## License

This API documentation is provided for QrrQ users. The API itself requires a valid QrrQ subscription.
