# API Reference Documentation

## Overview

The FinSecure API Reference provides comprehensive technical documentation for all available endpoints, including request/response formats, authentication methods, error codes, and practical examples.

---

## API Design Principles

### RESTful Architecture
- **Resource-Based URLs:** Endpoints represent business entities
- **HTTP Methods:** Semantic use of GET, POST, PUT, DELETE
- **Status Codes:** Standard HTTP status codes for all responses
- **Stateless Operations:** Each request contains all necessary information

### Consistency Standards
- **Naming Conventions:** snake_case for JSON properties
- **Date Formats:** ISO 8601 format (YYYY-MM-DDTHH:MM:SSZ)
- **Currency Handling:** Smallest unit representation (cents, pence)
- **Pagination:** Consistent cursor-based pagination

### Developer Experience
- **Predictable Patterns:** Similar endpoints follow identical patterns
- **Rich Error Messages:** Detailed error descriptions with resolution guidance
- **Extensive Examples:** Real-world scenarios with complete code samples
- **Comprehensive Testing:** All examples verified in sandbox environment

---

## Authentication

### API Key Authentication

FinSecure uses API keys to authenticate requests. Your API keys carry many privileges, so keep them secure and never share them in publicly accessible code.

#### Key Types

**Publishable Keys** (`pk_test_...` or `pk_live_...`)
- Safe to include in client-side code
- Used for creating payment methods and client-side operations
- Cannot complete payments or access sensitive data

**Secret Keys** (`sk_test_...` or `sk_live_...`)
- Must be kept secure on your server
- Used for server-side operations and completing payments
- Can access all API functionality

#### Authentication Header
```http
Authorization: Bearer sk_test_1234567890abcdef
```

#### Environment-Specific Keys

**Test Mode Keys**
```bash
# Publishable key for client-side operations
FINSECURE_PUBLISHABLE_KEY=pk_test_1234567890abcdef

# Secret key for server-side operations  
FINSECURE_SECRET_KEY=sk_test_abcdef1234567890
```

**Live Mode Keys**
```bash
# Production publishable key
FINSECURE_PUBLISHABLE_KEY=pk_live_1234567890abcdef

# Production secret key (highly sensitive)
FINSECURE_SECRET_KEY=sk_live_abcdef1234567890
```

---

## Core Resources

### Payment Intents

A PaymentIntent guides you through the process of collecting a payment from your customer.

#### Create Payment Intent

```http
POST https://api.finsecure.com/v1/payment_intents
```

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `amount` | integer | **Required** | Amount in smallest currency unit |
| `currency` | string | **Required** | Three-letter ISO currency code |
| `payment_method_types` | array | Optional | Accepted payment methods (default: `["card"]`) |
| `customer` | string | Optional | Customer ID for saved payment methods |
| `description` | string | Optional | Payment description for your records |
| `metadata` | object | Optional | Key-value pairs for additional data |
| `receipt_email` | string | Optional | Email address for payment receipt |
| `setup_future_usage` | string | Optional | `on_session` or `off_session` for saved payments |

**Request Example:**
```json
{
  "amount": 2000,
  "currency": "usd",
  "payment_method_types": ["card", "ach"],
  "customer": "cus_1234567890",
  "description": "Payment for Premium Subscription",
  "metadata": {
    "order_id": "order_12345",
    "customer_segment": "premium"
  },
  "receipt_email": "customer@example.com",
  "setup_future_usage": "off_session"
}
```

**Response Example:**
```json
{
  "id": "pi_1234567890abcdef",
  "object": "payment_intent",
  "amount": 2000,
  "amount_received": 0,
  "currency": "usd",
  "status": "requires_payment_method",
  "client_secret": "pi_1234567890abcdef_secret_abcdef1234567890",
  "created": 1640995200,
  "description": "Payment for Premium Subscription",
  "last_payment_error": null,
  "livemode": false,
  "metadata": {
    "order_id": "order_12345",
    "customer_segment": "premium"
  },
  "next_action": null,
  "payment_method": null,
  "payment_method_types": ["card", "ach"],
  "receipt_email": "customer@example.com",
  "setup_future_usage": "off_session",
  "shipping": null,
  "statement_descriptor": null,
  "statement_descriptor_suffix": null,
  "transfer_data": null,
  "transfer_group": null
}
```

#### Confirm Payment Intent

```http
POST https://api.finsecure.com/v1/payment_intents/{PAYMENT_INTENT_ID}/confirm
```

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `payment_method` | string | **Required** | Payment method ID or object |
| `return_url` | string | Conditional | Required for certain payment methods |
| `receipt_email` | string | Optional | Override receipt email address |

**Request Example:**
```json
{
  "payment_method": "pm_1234567890abcdef",
  "return_url": "https://yoursite.com/payment/return",
  "receipt_email": "customer@example.com"
}
```

**Response Example:**
```json
{
  "id": "pi_1234567890abcdef",
  "object": "payment_intent",
  "amount": 2000,
  "amount_received": 2000,
  "currency": "usd",
  "status": "succeeded",
  "charges": {
    "object": "list",
    "data": [
      {
        "id": "ch_1234567890abcdef",
        "object": "charge",
        "amount": 2000,
        "currency": "usd",
        "description": "Payment for Premium Subscription",
        "paid": true,
        "status": "succeeded",
        "created": 1640995260
      }
    ]
  }
}
```

### Payment Methods

Payment Methods represent how customers pay for transactions.

#### Create Payment Method

```http
POST https://api.finsecure.com/v1/payment_methods
```

**Card Payment Method:**
```json
{
  "type": "card",
  "card": {
    "number": "4242424242424242",
    "exp_month": 12,
    "exp_year": 2025,
    "cvc": "123"
  },
  "billing_details": {
    "name": "John Smith",
    "email": "john.smith@example.com",
    "address": {
      "line1": "123 Main St",
      "city": "San Francisco",
      "state": "CA",
      "postal_code": "94105",
      "country": "US"
    }
  }
}
```

**Bank Account Payment Method:**
```json
{
  "type": "us_bank_account",
  "us_bank_account": {
    "routing_number": "110000000",
    "account_number": "000123456789",
    "account_holder_type": "individual",
    "account_type": "checking"
  },
  "billing_details": {
    "name": "John Smith",
    "email": "john.smith@example.com"
  }
}
```

#### Retrieve Payment Method

```http
GET https://api.finsecure.com/v1/payment_methods/{PAYMENT_METHOD_ID}
```

**Response Example:**
```json
{
  "id": "pm_1234567890abcdef",
  "object": "payment_method",
  "type": "card",
  "card": {
    "brand": "visa",
    "checks": {
      "address_line1_check": "pass",
      "address_postal_code_check": "pass",
      "cvc_check": "pass"
    },
    "country": "US",
    "exp_month": 12,
    "exp_year": 2025,
    "fingerprint": "abcdef1234567890",
    "funding": "credit",
    "last4": "4242",
    "networks": {
      "available": ["visa"],
      "preferred": null
    },
    "three_d_secure_usage": {
      "supported": true
    },
    "wallet": null
  },
  "billing_details": {
    "address": {
      "city": "San Francisco",
      "country": "US",
      "line1": "123 Main St",
      "line2": null,
      "postal_code": "94105",
      "state": "CA"
    },
    "email": "john.smith@example.com",
    "name": "John Smith",
    "phone": null
  },
  "created": 1640995200,
  "customer": "cus_1234567890",
  "livemode": false,
  "metadata": {}
}
```

### Customers

Customer objects allow you to perform recurring charges and track payments that belong to the same customer.

#### Create Customer

```http
POST https://api.finsecure.com/v1/customers
```

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `email` | string | Optional | Customer's email address |
| `name` | string | Optional | Customer's full name |
| `phone` | string | Optional | Customer's phone number |
| `description` | string | Optional | Description of the customer |
| `metadata` | object | Optional | Key-value pairs for additional data |
| `payment_method` | string | Optional | Default payment method |
| `invoice_settings` | object | Optional | Invoice delivery preferences |

**Request Example:**
```json
{
  "email": "customer@example.com",
  "name": "John Smith",
  "phone": "+1-555-123-4567",
  "description": "Premium subscription customer",
  "metadata": {
    "customer_segment": "premium",
    "signup_source": "website"
  },
  "invoice_settings": {
    "default_payment_method": "pm_1234567890abcdef"
  }
}
```

**Response Example:**
```json
{
  "id": "cus_1234567890",
  "object": "customer",
  "created": 1640995200,
  "default_source": null,
  "description": "Premium subscription customer",
  "email": "customer@example.com",
  "invoice_prefix": "ABCD123",
  "invoice_settings": {
    "custom_fields": null,
    "default_payment_method": "pm_1234567890abcdef",
    "footer": null
  },
  "livemode": false,
  "metadata": {
    "customer_segment": "premium",
    "signup_source": "website"
  },
  "name": "John Smith",
  "phone": "+1-555-123-4567",
  "preferred_locales": [],
  "shipping": null,
  "tax_exempt": "none"
}
```

---

## Error Handling

### Error Response Format

All API errors return a consistent JSON structure:

```json
{
  "error": {
    "type": "card_error",
    "code": "card_declined",
    "decline_code": "insufficient_funds",
    "message": "Your card was declined.",
    "param": "payment_method",
    "payment_intent": {
      "id": "pi_1234567890abcdef",
      "status": "requires_payment_method"
    }
  }
}
```

### Error Types

#### Card Errors (`card_error`)
Issues with the customer's payment method.

| Code | Decline Code | Description | Action |
|------|--------------|-------------|--------|
| `card_declined` | `insufficient_funds` | Insufficient funds | Request different payment method |
| `card_declined` | `expired_card` | Card expired | Request updated card information |
| `card_declined` | `incorrect_cvc` | CVC check failed | Request correct CVC |
| `card_declined` | `processing_error` | Processing error | Retry payment |

#### Rate Limit Errors (`rate_limit_error`)
Too many requests made too quickly.

```json
{
  "error": {
    "type": "rate_limit_error",
    "message": "Too many requests made to the API too quickly"
  }
}
```

**Handling Rate Limits:**
```python
import time
import random

def make_api_request_with_backoff(api_function, max_retries=3):
    for attempt in range(max_retries):
        try:
            return api_function()
        except finsecure.error.RateLimitError:
            if attempt == max_retries - 1:
                raise
            # Exponential backoff with jitter
            delay = (2 ** attempt) + random.uniform(0, 1)
            time.sleep(delay)
```

#### Invalid Request Errors (`invalid_request_error`)
Request parameters are invalid or missing.

```json
{
  "error": {
    "type": "invalid_request_error",
    "message": "Missing required param: amount",
    "param": "amount"
  }
}
```

#### Authentication Errors (`authentication_error`)
API key is invalid or missing.

```json
{
  "error": {
    "type": "authentication_error",
    "message": "Invalid API Key provided"
  }
}
```

#### API Errors (`api_error`)
Internal server errors.

```json
{
  "error": {
    "type": "api_error",
    "message": "An error occurred while processing your request"
  }
}
```

### Error Handling Best Practices

#### Comprehensive Error Handling
```javascript
async function processPayment(paymentData) {
  try {
    const paymentIntent = await finsecure.paymentIntents.create(paymentData);
    return { success: true, paymentIntent };
  } catch (error) {
    // Handle specific error types
    switch (error.type) {
      case 'card_error':
        return {
          success: false,
          error_type: 'payment_failed',
          message: error.message,
          decline_code: error.decline_code,
          user_action: getCardErrorUserAction(error.code)
        };
        
      case 'rate_limit_error':
        return {
          success: false,
          error_type: 'rate_limit',
          message: 'Too many requests. Please try again in a moment.',
          retry_after: 60 // seconds
        };
        
      case 'invalid_request_error':
        return {
          success: false,
          error_type: 'validation_error',
          message: `Invalid request: ${error.message}`,
          param: error.param
        };
        
      case 'authentication_error':
        return {
          success: false,
          error_type: 'authentication',
          message: 'Authentication failed. Please check your API keys.'
        };
        
      case 'api_error':
      default:
        return {
          success: false,
          error_type: 'system_error',
          message: 'A system error occurred. Please try again later.'
        };
    }
  }
}

function getCardErrorUserAction(errorCode) {
  const actions = {
    'card_declined': 'Please use a different payment method or contact your bank.',
    'expired_card': 'Please update your card information with a valid expiration date.',
    'incorrect_cvc': 'Please check your card\'s security code and try again.',
    'processing_error': 'Please try your payment again.'
  };
  return actions[errorCode] || 'Please try a different payment method.';
}
```

---

## Webhooks

### Overview

Webhooks allow FinSecure to notify your application when events occur in your account. This is particularly useful for asynchronous events like successful payments, failed charges, or subscription updates.

### Event Types

#### Payment Events
- `payment_intent.succeeded` - Payment completed successfully
- `payment_intent.payment_failed` - Payment failed
- `payment_intent.requires_action` - Additional authentication required
- `payment_intent.canceled` - Payment was canceled

#### Customer Events
- `customer.created` - New customer created
- `customer.updated` - Customer information updated
- `customer.deleted` - Customer deleted

#### Subscription Events
- `invoice.payment_succeeded` - Subscription payment successful
- `invoice.payment_failed` - Subscription payment failed
- `customer.subscription.created` - New subscription created
- `customer.subscription.updated` - Subscription modified

### Webhook Endpoint Configuration

#### Endpoint Requirements
- **HTTPS Only:** All webhook endpoints must use HTTPS
- **Response Time:** Respond within 10 seconds
- **Status Codes:** Return 2xx status code for successful processing
- **Idempotency:** Handle duplicate events gracefully

#### Signature Verification
```python
import hmac
import hashlib
import json

def verify_webhook_signature(payload, signature, webhook_secret):
    """Verify webhook signature to ensure authenticity"""
    expected_signature = hmac.new(
        webhook_secret.encode('utf-8'),
        payload.encode('utf-8'),
        hashlib.sha256
    ).hexdigest()
    
    # Extract signature from header (format: "sha256=signature")
    received_signature = signature.replace('sha256=', '')
    
    return hmac.compare_digest(expected_signature, received_signature)

def handle_webhook_event(request):
    """Process incoming webhook events"""
    payload = request.body
    signature = request.headers.get('FinSecure-Signature')
    
    # Verify signature
    if not verify_webhook_signature(payload, signature, WEBHOOK_SECRET):
        return HttpResponse(status=400)
    
    # Parse event
    try:
        event = json.loads(payload)
    except json.JSONDecodeError:
        return HttpResponse(status=400)
    
    # Handle event type
    event_type = event['type']
    event_data = event['data']['object']
    
    if event_type == 'payment_intent.succeeded':
        handle_successful_payment(event_data)
    elif event_type == 'payment_intent.payment_failed':
        handle_failed_payment(event_data)
    elif event_type == 'customer.subscription.created':
        handle_new_subscription(event_data)
    
    return HttpResponse(status=200)
```

### Webhook Event Example

```json
{
  "id": "evt_1234567890abcdef",
  "object": "event",
  "api_version": "2023-10-16",
  "created": 1640995200,
  "data": {
    "object": {
      "id": "pi_1234567890abcdef",
      "object": "payment_intent",
      "amount": 2000,
      "currency": "usd",
      "status": "succeeded",
      "metadata": {
        "order_id": "order_12345"
      }
    }
  },
  "livemode": false,
  "pending_webhooks": 1,
  "request": {
    "id": "req_1234567890abcdef",
    "idempotency_key": null
  },
  "type": "payment_intent.succeeded"
}
```

---

## API Testing

### Test Environment

Use test API keys for development and testing:
- **Base URL:** `https://api-sandbox.finsecure.com`
- **Test Cards:** Special card numbers for different scenarios
- **No Real Money:** All transactions are simulated

### Test Card Numbers

#### Successful Payments
```
4242424242424242 - Visa (US)
4000056655665556 - Visa (debit)
5555555555554444 - Mastercard
2223003122003222 - Mastercard (2-series)
4000002760003184 - Visa (3D Secure)
```

#### Declined Payments
```
4000000000000002 - Card declined (generic)
4000000000009995 - Insufficient funds
4000000000009987 - Lost card
4000000000009979 - Stolen card
4100000000000019 - Fraudulent card
```

#### Specific Error Testing
```
4000000000000127 - Incorrect CVC
4000000000000069 - Expired card
4000000000000259 - Processing error
4000000000000341 - Incorrect number
```

### Testing Checklist

#### Basic Integration Testing
- [ ] Create payment intent successfully
- [ ] Confirm payment with valid card
- [ ] Handle declined card gracefully
- [ ] Process refunds correctly
- [ ] Verify webhook delivery

#### Error Handling Testing
- [ ] Invalid API key authentication
- [ ] Missing required parameters
- [ ] Invalid parameter values
- [ ] Rate limit handling
- [ ] Network timeout handling

#### Security Testing
- [ ] Webhook signature verification
- [ ] HTTPS-only communication
- [ ] API key security
- [ ] PCI DSS compliance
- [ ] Data encryption validation

---

*This API reference provides comprehensive technical documentation for integrating FinSecure payment processing. For additional support, visit our [Developer Portal](https://developers.finsecure.com) or contact our technical support team.*