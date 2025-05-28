# How-to Guides

## Overview

How-to guides provide step-by-step instructions for common integration tasks and business scenarios. Each guide is designed to be practical, actionable, and focused on achieving specific outcomes.

---

## Guide Categories

### Integration Guides
- **[Accept Your First Payment](./accept-first-payment.md)** - Complete payment processing walkthrough
- **[Set Up Recurring Payments](./recurring-payments.md)** - Subscription and billing automation
- **[Implement Refunds](./implement-refunds.md)** - Refund processing and management
- **[Handle Failed Payments](./handle-failed-payments.md)** - Error recovery and retry logic

### Security & Compliance
- **[Implement Webhook Security](./webhook-security.md)** - Signature verification and best practices
- **[Achieve PCI Compliance](./pci-compliance.md)** - Security requirements and implementation
- **[Set Up Fraud Prevention](./fraud-prevention.md)** - Risk management configuration
- **[Implement 3D Secure](./3d-secure.md)** - Strong customer authentication

### Business Scenarios
- **[Build a Marketplace](./marketplace-integration.md)** - Multi-party payment processing
- **[International Payments](./international-payments.md)** - Global payment processing setup
- **[Mobile App Integration](./mobile-integration.md)** - iOS and Android SDK usage
- **[Testing Strategies](./testing-strategies.md)** - Development and QA approaches

---

## Guide Structure

Each how-to guide follows a consistent structure for optimal user experience:

### 1. Goal Definition
- Clear objective statement
- Success criteria
- Prerequisites and assumptions
- Estimated completion time

### 2. Step-by-Step Instructions
- Numbered sequential steps
- Code examples for each step
- Expected outcomes and verification
- Common troubleshooting tips

### 3. Testing & Validation
- How to test the implementation
- Success indicators
- Error scenarios to validate
- Performance considerations

### 4. Next Steps
- Related guides and features
- Advanced configuration options
- Optimization opportunities
- Support resources

---

## Sample How-to Guide: Accept Your First Payment

### Goal
Set up basic payment processing to accept credit card payments on your website within 30 minutes.

### Prerequisites
- FinSecure account with test API keys
- Basic web development knowledge (HTML, JavaScript)
- HTTPS-enabled website or development environment
- Text editor or IDE

### What You'll Build
A simple payment form that:
- Collects customer payment information securely
- Processes credit card payments
- Handles success and error scenarios
- Provides transaction confirmation

---

### Step 1: Set Up Your Environment (5 minutes)

#### Create API Credentials
1. Log into your FinSecure dashboard
2. Navigate to "API Keys" in the sidebar
3. Copy your test publishable key (`pk_test_...`)
4. Copy your test secret key (`sk_test_...`)
5. Store these securely in your environment variables

```bash
# Add to your environment variables
export FINSECURE_PUBLISHABLE_KEY=pk_test_1234567890abcdef
export FINSECURE_SECRET_KEY=sk_test_abcdef1234567890
```

#### Verify HTTPS Setup
Ensure your development environment uses HTTPS:
```bash
# For local development, you can use tools like:
npx http-server --ssl -p 8443
# or
python -m http.server 8443 --bind 127.0.0.1
```

**Why HTTPS is Required:**
- Protects sensitive payment data in transit
- Required for PCI DSS compliance
- Prevents man-in-the-middle attacks
- Browser security requirements for payment APIs

---

### Step 2: Create the Payment Form (10 minutes)

#### HTML Structure
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>FinSecure Payment Example</title>
    <style>
        .payment-form {
            max-width: 400px;
            margin: 50px auto;
            padding: 20px;
            border: 1px solid #ddd;
            border-radius: 8px;
            font-family: Arial, sans-serif;
        }
        .form-group {
            margin-bottom: 15px;
        }
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }
        input[type="text"], input[type="email"] {
            width: 100%;
            padding: 10px;
            border: 1px solid #ccc;
            border-radius: 4px;
            box-sizing: border-box;
        }
        .card-element {
            padding: 10px;
            border: 1px solid #ccc;
            border-radius: 4px;
            background: white;
        }
        .submit-button {
            width: 100%;
            padding: 12px;
            background: #5469d4;
            color: white;
            border: none;
            border-radius: 4px;
            font-size: 16px;
            cursor: pointer;
        }
        .submit-button:hover {
            background: #4051b5;
        }
        .submit-button:disabled {
            background: #ccc;
            cursor: not-allowed;
        }
        .error-message {
            color: #e74c3c;
            margin-top: 10px;
            font-size: 14px;
        }
        .success-message {
            color: #27ae60;
            margin-top: 10px;
            font-size: 14px;
        }
    </style>
    <script src="https://js.finsecure.com/v1/"></script>
</head>
<body>
    <div class="payment-form">
        <h2>Complete Your Payment</h2>
        <form id="payment-form">
            <div class="form-group">
                <label for="email">Email Address</label>
                <input type="email" id="email" name="email" required>
            </div>
            
            <div class="form-group">
                <label for="amount">Payment Amount</label>
                <input type="text" id="amount" name="amount" value="$20.00" readonly>
            </div>
            
            <div class="form-group">
                <label for="card-element">Credit or Debit Card</label>
                <div id="card-element" class="card-element">
                    <!-- FinSecure Elements will create form elements here -->
                </div>
            </div>
            
            <button type="submit" id="submit-button" class="submit-button">
                <span id="button-text">Pay $20.00</span>
                <span id="spinner" style="display: none;">Processing...</span>
            </button>
            
            <div id="payment-result"></div>
        </form>
    </div>

    <script>
        // Payment form JavaScript will go here
    </script>
</body>
</html>
```

#### Initialize FinSecure Elements
```javascript
// Initialize FinSecure with your publishable key
const finsecure = FinSecure('pk_test_1234567890abcdef');
const elements = finsecure.elements();

// Create card element with styling
const cardElement = elements.create('card', {
    style: {
        base: {
            fontSize: '16px',
            color: '#424770',
            '::placeholder': {
                color: '#aab7c4',
            },
        },
        invalid: {
            color: '#9e2146',
        },
    },
});

// Mount the card element
cardElement.mount('#card-element');

// Handle real-time validation errors from the card element
cardElement.on('change', function(event) {
    const displayError = document.getElementById('payment-result');
    if (event.error) {
        showError(event.error.message);
    } else {
        displayError.textContent = '';
    }
});
```

---

### Step 3: Handle Form Submission (10 minutes)

#### Client-Side Payment Processing
```javascript
// Handle form submission
document.getElementById('payment-form').addEventListener('submit', async function(event) {
    event.preventDefault();
    
    const submitButton = document.getElementById('submit-button');
    const buttonText = document.getElementById('button-text');
    const spinner = document.getElementById('spinner');
    
    // Disable submit button and show processing state
    submitButton.disabled = true;
    buttonText.style.display = 'none';
    spinner.style.display = 'inline';
    
    try {
        // Create payment method
        const {error, paymentMethod} = await finsecure.createPaymentMethod({
            type: 'card',
            card: cardElement,
            billing_details: {
                email: document.getElementById('email').value,
            },
        });
        
        if (error) {
            showError(error.message);
            resetButton();
            return;
        }
        
        // Send payment method to your server
        const response = await fetch('/process-payment', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify({
                payment_method_id: paymentMethod.id,
                amount: 2000, // $20.00 in cents
                currency: 'usd',
                customer_email: document.getElementById('email').value
            }),
        });
        
        const result = await response.json();
        
        if (result.success) {
            showSuccess('Payment successful! Transaction ID: ' + result.transaction_id);
        } else {
            showError(result.error || 'Payment failed. Please try again.');
        }
        
    } catch (error) {
        showError('Network error. Please check your connection and try again.');
    } finally {
        resetButton();
    }
});

function showError(message) {
    const resultDiv = document.getElementById('payment-result');
    resultDiv.innerHTML = `<div class="error-message">${message}</div>`;
}

function showSuccess(message) {
    const resultDiv = document.getElementById('payment-result');
    resultDiv.innerHTML = `<div class="success-message">${message}</div>`;
}

function resetButton() {
    const submitButton = document.getElementById('submit-button');
    const buttonText = document.getElementById('button-text');
    const spinner = document.getElementById('spinner');
    
    submitButton.disabled = false;
    buttonText.style.display = 'inline';
    spinner.style.display = 'none';
}
```

---

### Step 4: Server-Side Payment Processing (15 minutes)

#### Node.js Backend Implementation
```javascript
const express = require('express');
const finsecure = require('@finsecure/node-sdk');
const app = express();

// Initialize FinSecure with your secret key
finsecure.apiKey = process.env.FINSECURE_SECRET_KEY;

app.use(express.json());
app.use(express.static('public')); // Serve your HTML file

app.post('/process-payment', async (req, res) => {
    const { payment_method_id, amount, currency, customer_email } = req.body;
    
    try {
        // Create payment intent
        const paymentIntent = await finsecure.PaymentIntent.create({
            amount: amount,
            currency: currency,
            payment_method_types: ['card'],
            payment_method: payment_method_id,
            confirm: true,
            receipt_email: customer_email,
            description: 'Payment for order #12345',
            metadata: {
                customer_email: customer_email,
                order_id: '12345'
            }
        });
        
        // Handle different payment states
        if (paymentIntent.status === 'succeeded') {
            // Payment successful
            res.json({
                success: true,
                transaction_id: paymentIntent.charges.data[0].id,
                amount: paymentIntent.amount,
                currency: paymentIntent.currency
            });
            
            // Optional: Save transaction to your database
            await saveTransactionToDatabase({
                transaction_id: paymentIntent.charges.data[0].id,
                customer_email: customer_email,
                amount: paymentIntent.amount,
                currency: paymentIntent.currency,
                status: 'completed'
            });
            
        } else if (paymentIntent.status === 'requires_action') {
            // Additional authentication required (3D Secure)
            res.json({
                requires_action: true,
                client_secret: paymentIntent.client_secret
            });
            
        } else {
            // Payment failed
            res.json({
                success: false,
                error: 'Payment failed to process'
            });
        }
        
    } catch (error) {
        console.error('Payment processing error:', error);
        
        // Handle specific error types
        if (error.type === 'FinSecureCardError') {
            res.json({
                success: false,
                error: error.message,
                decline_code: error.decline_code
            });
        } else {
            res.json({
                success: false,
                error: 'An unexpected error occurred'
            });
        }
    }
});

async function saveTransactionToDatabase(transactionData) {
    // Implementation depends on your database choice
    // Example with MongoDB:
    /*
    const db = getDatabase();
    await db.collection('transactions').insertOne({
        ...transactionData,
        created_at: new Date(),
        updated_at: new Date()
    });
    */
}

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
});
```

#### Python Flask Alternative
```python
from flask import Flask, request, jsonify, render_template
import finsecure
import os
from datetime import datetime

app = Flask(__name__)
finsecure.api_key = os.environ.get('FINSECURE_SECRET_KEY')

@app.route('/')
def index():
    return render_template('payment_form.html')

@app.route('/process-payment', methods=['POST'])
def process_payment():
    try:
        data = request.get_json()
        
        # Create payment intent
        payment_intent = finsecure.PaymentIntent.create(
            amount=data['amount'],
            currency=data['currency'],
            payment_method_types=['card'],
            payment_method=data['payment_method_id'],
            confirm=True,
            receipt_email=data['customer_email'],
            description='Payment for order #12345',
            metadata={
                'customer_email': data['customer_email'],
                'order_id': '12345'
            }
        )
        
        if payment_intent.status == 'succeeded':
            # Save transaction record
            save_transaction({
                'transaction_id': payment_intent.charges.data[0].id,
                'customer_email': data['customer_email'],
                'amount': payment_intent.amount,
                'currency': payment_intent.currency,
                'status': 'completed',
                'created_at': datetime.utcnow()
            })
            
            return jsonify({
                'success': True,
                'transaction_id': payment_intent.charges.data[0].id,
                'amount': payment_intent.amount,
                'currency': payment_intent.currency
            })
            
        elif payment_intent.status == 'requires_action':
            return jsonify({
                'requires_action': True,
                'client_secret': payment_intent.client_secret
            })
        else:
            return jsonify({
                'success': False,
                'error': 'Payment failed to process'
            })
            
    except finsecure.error.CardError as e:
        return jsonify({
            'success': False,
            'error': e.user_message,
            'decline_code': e.decline_code
        })
    except Exception as e:
        print(f"Payment processing error: {e}")
        return jsonify({
            'success': False,
            'error': 'An unexpected error occurred'
        })

def save_transaction(transaction_data):
    # Implement database storage
    # Example with SQLAlchemy:
    # db.session.add(Transaction(**transaction_data))
    # db.session.commit()
    pass

if __name__ == '__main__':
    app.run(debug=True, ssl_context='adhoc')
```

---

### Step 5: Testing Your Implementation (10 minutes)

#### Test with Sample Cards
Use these test card numbers in your sandbox environment:

**Successful Payment:**
```
Card Number: 4242424242424242
Expiry: 12/25
CVC: 123
ZIP: 12345
```

**Declined Payment:**
```
Card Number: 4000000000000002
Expiry: 12/25
CVC: 123
ZIP: 12345
```

#### Verification Checklist
- [ ] Payment form loads correctly
- [ ] Card validation works in real-time
- [ ] Successful payment shows confirmation
- [ ] Declined payment shows appropriate error
- [ ] Payment intent created in dashboard
- [ ] Email receipt sent (if configured)

#### Testing Workflow
1. **Open your payment form** in a web browser
2. **Enter test email address** (e.g., test@example.com)
3. **Use successful test card** (4242424242424242)
4. **Submit payment** and verify success message
5. **Check FinSecure dashboard** for transaction record
6. **Test declined card** (4000000000000002)
7. **Verify error handling** displays appropriate message

---

### Troubleshooting Common Issues

#### Problem: "Invalid API Key" Error
**Solution:**
```javascript
// Ensure you're using the correct environment
// Test keys start with pk_test_ and sk_test_
// Live keys start with pk_live_ and sk_live_

// Check environment variable loading
console.log('API Key:', process.env.FINSECURE_SECRET_KEY?.substring(0, 12) + '...');
```

#### Problem: CORS Errors
**Solution:**
```javascript
// Add CORS headers to your server
app.use((req, res, next) => {
    res.header('Access-Control-Allow-Origin', '*');
    res.header('Access-Control-Allow-Headers', 'Content-Type');
    res.header('Access-Control-Allow-Methods', 'POST, GET, OPTIONS');
    next();
});
```

#### Problem: Card Element Not Loading
**Solution:**
```javascript
// Ensure FinSecure.js loads before your script
<script src="https://js.finsecure.com/v1/"></script>
<script>
    // Wait for FinSecure to load
    if (typeof FinSecure === 'undefined') {
        console.error('FinSecure.js failed to load');
    }
</script>
```

---

### Next Steps

#### Immediate Enhancements
1. **[Add Webhook Handling](./webhook-implementation.md)** - Process payment events asynchronously
2. **[Implement Customer Management](./customer-management.md)** - Save customer payment methods
3. **[Add Refund Processing](./implement-refunds.md)** - Handle refund requests

#### Advanced Features
1. **[Set Up Recurring Payments](./recurring-payments.md)** - Subscription billing
2. **[Multi-payment Methods](./payment-methods.md)** - ACH, digital wallets
3. **[International Payments](./international-payments.md)** - Global processing

#### Production Preparation
1. **[Security Hardening](./security-checklist.md)** - Production security review
2. **[Performance Optimization](./performance.md)** - Scale for high volume
3. **[Monitoring Setup](./monitoring.md)** - Transaction monitoring and alerts

---

### Support Resources

- **Developer Documentation:** [developers.finsecure.com](https://developers.finsecure.com)
- **Community Forum:** [community.finsecure.com](https://community.finsecure.com)
- **Technical Support:** support@finsecure.com
- **Status Updates:** [status.finsecure.com](https://status.finsecure.com)

*You've successfully implemented basic payment processing! This foundation can be extended with additional features and optimizations as your business grows.*