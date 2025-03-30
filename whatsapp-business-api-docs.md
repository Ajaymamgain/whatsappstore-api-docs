# WhatsApp Business Management API Documentation

## Overview

The WhatsApp Business Management API enables businesses to interact with customers through WhatsApp at scale. This API documentation covers key endpoints and functionalities needed for the WhatsAppStore application.

## Authentication

The WhatsApp Business API uses OAuth 2.0 for authentication. You'll need to obtain access tokens from Meta for Business.

### Access Token

- **Type**: Bearer Token
- **Format**: `Bearer YOUR_ACCESS_TOKEN`
- **Expiration**: Typically 24 hours
- **Scope Requirements**: `whatsapp_business_messaging`, `whatsapp_business_management`

## Base URL

```
https://graph.facebook.com/v18.0/
```

## Core API Endpoints

### 1. Send Messages

#### Send Text Message

```http
POST /{phone-number-ID}/messages
```

**Headers:**
```
Authorization: Bearer {access-token}
Content-Type: application/json
```

**Request Body:**
```json
{
  "messaging_product": "whatsapp",
  "recipient_type": "individual",
  "to": "{recipient-phone-number}",
  "type": "text",
  "text": {
    "preview_url": false,
    "body": "Hello, this is a test message from WhatsAppStore!"
  }
}
```

**Response:**
```json
{
  "messaging_product": "whatsapp",
  "contacts": [
    {
      "input": "{recipient-phone-number}",
      "wa_id": "{whatsapp-id}"
    }
  ],
  "messages": [
    {
      "id": "wamid.{message-id}"
    }
  ]
}
```

#### Send Template Message

```http
POST /{phone-number-ID}/messages
```

**Headers:**
```
Authorization: Bearer {access-token}
Content-Type: application/json
```

**Request Body:**
```json
{
  "messaging_product": "whatsapp",
  "recipient_type": "individual",
  "to": "{recipient-phone-number}",
  "type": "template",
  "template": {
    "name": "order_confirmation",
    "language": {
      "code": "en_US"
    },
    "components": [
      {
        "type": "header",
        "parameters": [
          {
            "type": "image",
            "image": {
              "link": "https://example.com/logo.png"
            }
          }
        ]
      },
      {
        "type": "body",
        "parameters": [
          {
            "type": "text",
            "text": "John Doe"
          },
          {
            "type": "text",
            "text": "12345"
          },
          {
            "type": "text",
            "text": "$100.00"
          }
        ]
      }
    ]
  }
}
```

### 2. Message Status Updates

#### Mark Message as Read

```http
POST /{phone-number-ID}/messages
```

**Headers:**
```
Authorization: Bearer {access-token}
Content-Type: application/json
```

**Request Body:**
```json
{
  "messaging_product": "whatsapp",
  "status": "read",
  "message_id": "wamid.{message-id}"
}
```

### 3. Media Management

#### Upload Media

```http
POST /{phone-number-ID}/media
```

**Headers:**
```
Authorization: Bearer {access-token}
Content-Type: multipart/form-data
```

**Form Parameters:**
- `file`: The media file to upload
- `type`: The MIME type of the file (e.g., `image/jpeg`)
- `messaging_product`: Always set to `whatsapp`

**Response:**
```json
{
  "id": "media-id"
}
```

#### Send Media Message

```http
POST /{phone-number-ID}/messages
```

**Headers:**
```
Authorization: Bearer {access-token}
Content-Type: application/json
```

**Request Body for Image:**
```json
{
  "messaging_product": "whatsapp",
  "recipient_type": "individual",
  "to": "{recipient-phone-number}",
  "type": "image",
  "image": {
    "id": "media-id"
  }
}
```

### 4. Product Catalog Management

#### Get Business Catalog

```http
GET /{business-id}/catalogs
```

**Headers:**
```
Authorization: Bearer {access-token}
```

**Response:**
```json
{
  "data": [
    {
      "id": "catalog-id",
      "name": "Main Catalog"
    }
  ]
}
```

#### Add Product to Catalog

```http
POST /{catalog-id}/products
```

**Headers:**
```
Authorization: Bearer {access-token}
Content-Type: application/json
```

**Request Body:**
```json
{
  "name": "Product Name",
  "description": "Product description",
  "price": "100.00",
  "currency": "USD",
  "image_urls": ["https://example.com/product1.jpg"],
  "retailer_id": "SKU123"
}
```

#### Get Products from Catalog

```http
GET /{catalog-id}/products
```

**Headers:**
```
Authorization: Bearer {access-token}
```

**Response:**
```json
{
  "data": [
    {
      "id": "product-id",
      "name": "Product Name",
      "description": "Product description",
      "price": "100.00",
      "currency": "USD",
      "image_urls": ["https://example.com/product1.jpg"],
      "retailer_id": "SKU123"
    }
  ]
}
```

### 5. Webhook Management

#### Receive Webhook Events

WhatsApp sends several types of webhooks to your configured endpoint:

- Message received
- Message status updates
- Business account review updates

**Example Webhook Payload for Message Received:**
```json
{
  "object": "whatsapp_business_account",
  "entry": [
    {
      "id": "WHATSAPP_BUSINESS_ACCOUNT_ID",
      "changes": [
        {
          "value": {
            "messaging_product": "whatsapp",
            "metadata": {
              "display_phone_number": "PHONE_NUMBER",
              "phone_number_id": "PHONE_NUMBER_ID"
            },
            "contacts": [
              {
                "profile": {
                  "name": "CONTACT_NAME"
                },
                "wa_id": "CONTACT_WHATSAPP_ID"
              }
            ],
            "messages": [
              {
                "from": "CONTACT_WHATSAPP_ID",
                "id": "MESSAGE_ID",
                "timestamp": "TIMESTAMP",
                "text": {
                  "body": "MESSAGE_BODY"
                },
                "type": "text"
              }
            ]
          },
          "field": "messages"
        }
      ]
    }
  ]
}
```

#### Configure Webhook

To receive webhooks:

1. Create a webhook endpoint in your application
2. Verify the webhook using the challenge response mechanism
3. Configure the webhook in the Meta for Business dashboard:
   - Messages: for receiving messages
   - Message Status Updates: for delivery and read receipts
   - Business Account Updates: for account status changes

## Integration with WhatsAppStore

The WhatsAppStore application should integrate these API endpoints to enable:

1. **Product Catalog Synchronization**: Sync products between the store and WhatsApp
2. **Automated Order Notifications**: Send order confirmations and updates
3. **Message Broadcasting**: Send marketing campaigns to customer segments
4. **Cart Recovery**: Send reminder messages for abandoned carts
5. **Interactive Product Browsing**: Enable products to be viewed and purchased via WhatsApp

## Rate Limits and Best Practices

- **Message Rate Limits**: 
  - Business accounts are subjected to tiered messaging limits
  - New accounts start with lower limits that increase over time
  - Special restrictions apply for template messages
  
- **Best Practices**:
  - Implement webhook signature verification for security
  - Cache media IDs after uploading to avoid redundant uploads
  - Implement exponential backoff for rate-limited requests
  - Store message IDs for proper status tracking
  - Use template messages for business-initiated conversations

## Error Handling

Common error codes:

| Code | Description | Resolution |
|------|-------------|------------|
| 100  | Invalid parameter | Check request body for errors |
| 130  | Rate limited | Implement exponential backoff |
| 131  | Webhook subscribe error | Verify webhook URL |
| 190  | Invalid/expired token | Refresh access token |
| 368  | Temporarily blocked | Wait for the block to expire |
| 400  | Bad request | Check JSON syntax and parameters |
| 401  | Unauthorized | Verify credentials |
| 403  | Forbidden | Check permission scopes |
| 404  | Not found | Verify resource IDs |
| 429  | Too many requests | Implement rate limiting on your side |
| 500  | Internal server error | Retry with exponential backoff |

## Conclusion

The WhatsApp Business Management API provides powerful capabilities for businesses to engage with customers. By integrating these endpoints into the WhatsAppStore application, you can create a seamless shopping experience that extends from your website to WhatsApp messaging.
