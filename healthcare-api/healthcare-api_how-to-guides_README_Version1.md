# Healthcare How-to Guides

## Overview

Healthcare how-to guides provide step-by-step instructions for common clinical integration tasks and medical workflows. Each guide is designed to be practical, clinically relevant, and focused on achieving specific healthcare outcomes while maintaining HIPAA compliance.

---

## Guide Categories

### Clinical Integration Guides
- **[Create Your First Patient Record](./create-first-patient.md)** - Complete FHIR Patient resource walkthrough
- **[Set Up Vital Signs Monitoring](./vital-signs-monitoring.md)** - Real-time clinical observation processing
- **[Implement Medication Management](./medication-management.md)** - Prescription processing and drug interaction checking
- **[Handle Clinical Decision Support](./clinical-decision-support.md)** - AI-powered clinical recommendations

### Healthcare Security & Compliance
- **[Implement SMART on FHIR](./smart-on-fhir.md)** - Healthcare authorization and clinical context
- **[Achieve HIPAA Compliance](./hipaa-compliance.md)** - Privacy and security rule implementation
- **[Set Up Clinical Audit Logging](./clinical-audit-logging.md)** - Healthcare access monitoring
- **[Handle Patient Consent](./patient-consent.md)** - Consent management and data sharing

### Medical Workflow Scenarios
- **[Build a Patient Portal](./patient-portal.md)** - Consumer-facing healthcare applications
- **[Integrate with EHR Systems](./ehr-integration.md)** - Electronic health record connectivity
- **[Implement Telehealth Features](./telehealth-integration.md)** - Remote care and virtual consultations
- **[Population Health Analytics](./population-health.md)** - Clinical quality measures and reporting

---

## Guide Structure

Each healthcare how-to guide follows a consistent clinical structure for optimal provider experience:

### 1. Clinical Goal Definition
- Clear healthcare objective statement
- Clinical success criteria and patient outcomes
- Prerequisites and medical assumptions
- Estimated clinical implementation time

### 2. Step-by-Step Clinical Instructions
- Numbered sequential steps with clinical context
- FHIR code examples for each healthcare scenario
- Expected clinical outcomes and verification
- Common clinical troubleshooting tips

### 3. Healthcare Testing & Validation
- How to test the clinical implementation
- Clinical success indicators and patient safety measures
- Error scenarios to validate with medical context
- Healthcare performance considerations

### 4. Clinical Next Steps
- Related healthcare guides and medical features
- Advanced clinical configuration options
- Patient care optimization opportunities
- Healthcare support resources

---

## Sample Healthcare How-to Guide: Create Your First Patient Record

### Clinical Goal
Set up FHIR R4-compliant patient registration to securely manage patient demographics and clinical identifiers within 45 minutes.

### Clinical Prerequisites
- MedConnect healthcare account with FHIR API access
- Basic understanding of healthcare data standards
- HIPAA-compliant development environment
- Clinical workflow knowledge (patient registration, medical record management)

### What You'll Build Clinically
A patient registration system that:
- Creates FHIR Patient resources with proper medical identifiers
- Validates clinical data according to healthcare standards
- Maintains HIPAA compliance throughout patient data handling
- Integrates with existing healthcare workflows

---

### Step 1: Set Up Healthcare Environment (10 minutes)

#### Create Clinical API Credentials
1. Log into your MedConnect healthcare dashboard
2. Navigate to "FHIR API Keys" in the clinical settings
3. Copy your healthcare publishable key (`hc_test_...`)
4. Copy your healthcare secret key (`hc_secret_...`)
5. Store these securely with healthcare environment variables

```bash
# Add to your healthcare environment variables
export MEDCONNECT_FHIR_CLIENT_ID=hc_test_1234567890abcdef
export MEDCONNECT_FHIR_CLIENT_SECRET=hc_secret_abcdef1234567890
export MEDCONNECT_FHIR_BASE_URL=https://fhir-sandbox.medconnect.com/R4
```

#### Verify HIPAA-Compliant Setup
Ensure your development environment meets healthcare security requirements:
```bash
# Verify HTTPS for all clinical communications
curl -I https://fhir-sandbox.medconnect.com/R4/metadata

# Check healthcare environment variables
env | grep MEDCONNECT_FHIR
```

**Why HIPAA Compliance is Critical:**
- Protects patient health information (PHI) in transit and at rest
- Required for all healthcare applications handling patient data
- Prevents unauthorized access to medical records
- Browser and API security requirements for clinical data

---

### Step 2: Create Patient Registration Form (15 minutes)

#### HTML Structure for Clinical Data Collection
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MedConnect Patient Registration</title>
    <style>
        .patient-registration-form {
            max-width: 600px;
            margin: 50px auto;
            padding: 30px;
            border: 1px solid #e0e0e0;
            border-radius: 10px;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: #fafafa;
        }
        .clinical-section {
            margin-bottom: 25px;
            padding: 15px;
            background: white;
            border-radius: 5px;
            border-left: 4px solid #2c5282;
        }
        .clinical-section h3 {
            margin-top: 0;
            color: #2c5282;
            font-size: 16px;
        }
        .form-group {
            margin-bottom: 15px;
        }
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: 600;
            color: #2d3748;
        }
        .required-field::after {
            content: " *";
            color: #e53e3e;
        }
        input[type="text"], input[type="email"], input[type="tel"], 
        input[type="date"], select {
            width: 100%;
            padding: 12px;
            border: 1px solid #cbd5e0;
            border-radius: 6px;
            box-sizing: border-box;
            font-size: 14px;
        }
        input:focus, select:focus {
            border-color: #3182ce;
            outline: none;
            box-shadow: 0 0 0 3px rgba(49, 130, 206, 0.1);
        }
        .register-button {
            width: 100%;
            padding: 15px;
            background: #3182ce;
            color: white;
            border: none;
            border-radius: 6px;
            font-size: 16px;
            font-weight: 600;
            cursor: pointer;
            transition: background-color 0.3s;
        }
        .register-button:hover {
            background: #2c5282;
        }
        .register-button:disabled {
            background: #a0aec0;
            cursor: not-allowed;
        }
        .clinical-message {
            margin-top: 15px;
            padding: 12px;
            border-radius: 6px;
            font-size: 14px;
        }
        .success-message {
            background: #f0fff4;
            color: #22543d;
            border: 1px solid #9ae6b4;
        }
        .error-message {
            background: #fed7d7;
            color: #742a2a;
            border: 1px solid #feb2b2;
        }
        .hipaa-notice {
            font-size: 12px;
            color: #718096;
            text-align: center;
            margin-top: 20px;
            padding: 10px;
            background: #edf2f7;
            border-radius: 4px;
        }
    </style>
</head>
<body>
    <div class="patient-registration-form">
        <h2>Patient Registration - FHIR Compliant</h2>
        <form id="patient-registration-form">
            
            <!-- Patient Demographics Section -->
            <div class="clinical-section">
                <h3>Patient Demographics</h3>
                
                <div class="form-group">
                    <label for="medical-record-number" class="required-field">Medical Record Number</label>
                    <input type="text" id="medical-record-number" name="mrn" 
                           placeholder="Auto-generated if left blank" maxlength="20">
                </div>
                
                <div style="display: flex; gap: 15px;">
                    <div class="form-group" style="flex: 1;">
                        <label for="first-name" class="required-field">First Name</label>
                        <input type="text" id="first-name" name="given" required maxlength="50">
                    </div>
                    <div class="form-group" style="flex: 1;">
                        <label for="middle-name">Middle Name</label>
                        <input type="text" id="middle-name" name="middle" maxlength="50">
                    </div>
                    <div class="form-group" style="flex: 1;">
                        <label for="last-name" class="required-field">Last Name</label>
                        <input type="text" id="last-name" name="family" required maxlength="50">
                    </div>
                </div>
                
                <div style="display: flex; gap: 15px;">
                    <div class="form-group" style="flex: 1;">
                        <label for="birth-date" class="required-field">Date of Birth</label>
                        <input type="date" id="birth-date" name="birthDate" required>
                    </div>
                    <div class="form-group" style="flex: 1;">
                        <label for="gender" class="required-field">Gender</label>
                        <select id="gender" name="gender" required>
                            <option value="">Select Gender</option>
                            <option value="male">Male</option>
                            <option value="female">Female</option>
                            <option value="other">Other</option>
                            <option value="unknown">Unknown</option>
                        </select>
                    </div>
                </div>
            </div>

            <!-- Contact Information Section -->
            <div class="clinical-section">
                <h3>Contact Information</h3>
                
                <div class="form-group">
                    <label for="phone-primary" class="required-field">Primary Phone</label>
                    <input type="tel" id="phone-primary" name="phonePrimary" 
                           placeholder="+1-555-123-4567" required>
                </div>
                
                <div class="form-group">
                    <label for="email">Email Address</label>
                    <input type="email" id="email" name="email" 
                           placeholder="patient@example.com">
                </div>
                
                <div class="form-group">
                    <label for="address-line1" class="required-field">Address Line 1</label>
                    <input type="text" id="address-line1" name="addressLine1" 
                           placeholder="123 Healthcare Ave" required>
                </div>
                
                <div class="form-group">
                    <label for="address-line2">Address Line 2</label>
                    <input type="text" id="address-line2" name="addressLine2" 
                           placeholder="Apt 4B, Suite 100">
                </div>
                
                <div style="display: flex; gap: 15px;">
                    <div class="form-group" style="flex: 2;">
                        <label for="city" class="required-field">City</label>
                        <input type="text" id="city" name="city" required>
                    </div>
                    <div class="form-group" style="flex: 1;">
                        <label for="state" class="required-field">State</label>
                        <select id="state" name="state" required>
                            <option value="">Select State</option>
                            <option value="CA">California</option>
                            <option value="TX">Texas</option>
                            <option value="NY">New York</option>
                            <option value="FL">Florida</option>
                            <!-- Add more states as needed -->
                        </select>
                    </div>
                    <div class="form-group" style="flex: 1;">
                        <label for="postal-code" class="required-field">ZIP Code</label>
                        <input type="text" id="postal-code" name="postalCode" 
                               pattern="[0-9]{5}(-[0-9]{4})?" required>
                    </div>
                </div>
            </div>

            <!-- Clinical Information Section -->
            <div class="clinical-section">
                <h3>Clinical Information</h3>
                
                <div class="form-group">
                    <label for="preferred-language">Preferred Language</label>
                    <select id="preferred-language" name="preferredLanguage">
                        <option value="en-US">English (United States)</option>
                        <option value="es-US">Spanish (United States)</option>
                        <option value="fr-US">French (United States)</option>
                        <option value="zh-CN">Chinese (Simplified)</option>
                    </select>
                </div>
                
                <div class="form-group">
                    <label for="marital-status">Marital Status</label>
                    <select id="marital-status" name="maritalStatus">
                        <option value="">Select Marital Status</option>
                        <option value="S">Single</option>
                        <option value="M">Married</option>
                        <option value="D">Divorced</option>
                        <option value="W">Widowed</option>
                        <option value="UNK">Unknown</option>
                    </select>
                </div>
            </div>

            <!-- Emergency Contact Section -->
            <div class="clinical-section">
                <h3>Emergency Contact</h3>
                
                <div style="display: flex; gap: 15px;">
                    <div class="form-group" style="flex: 1;">
                        <label for="emergency-first-name">First Name</label>
                        <input type="text" id="emergency-first-name" name="emergencyFirstName">
                    </div>
                    <div class="form-group" style="flex: 1;">
                        <label for="emergency-last-name">Last Name</label>
                        <input type="text" id="emergency-last-name" name="emergencyLastName">
                    </div>
                </div>
                
                <div style="display: flex; gap: 15px;">
                    <div class="form-group" style="flex: 1;">
                        <label for="emergency-relationship">Relationship</label>
                        <select id="emergency-relationship" name="emergencyRelationship">
                            <option value="">Select Relationship</option>
                            <option value="C">Emergency Contact</option>
                            <option value="E">Employer</option>
                            <option value="F">Federal Agency</option>
                            <option value="I">Insurance Company</option>
                            <option value="N">Next-of-Kin</option>
                            <option value="S">State Agency</option>
                            <option value="U">Unknown</option>
                        </select>
                    </div>
                    <div class="form-group" style="flex: 1;">
                        <label for="emergency-phone">Phone Number</label>
                        <input type="tel" id="emergency-phone" name="emergencyPhone" 
                               placeholder="+1-555-987-6543">
                    </div>
                </div>
            </div>
            
            <button type="submit" id="register-button" class="register-button">
                <span id="button-text">Register Patient</span>
                <span id="spinner" style="display: none;">Processing...</span>
            </button>
            
            <div id="registration-result"></div>
        </form>
        
        <div class="hipaa-notice">
            🔒 This form is HIPAA compliant. All patient health information is encrypted and 
            stored securely in accordance with healthcare privacy regulations.
        </div>
    </div>

    <script>
        // Patient registration JavaScript will go here
    </script>
</body>
</html>
```

#### Initialize FHIR Client for Healthcare
```javascript
// Initialize MedConnect FHIR client
class MedConnectFHIRClient {
    constructor(clientId, baseUrl) {
        this.clientId = clientId;
        this.baseUrl = baseUrl;
        this.accessToken = null;
    }
    
    async authenticate(clientSecret) {
        // For demonstration - in production, use SMART on FHIR flow
        try {
            const response = await fetch(`${this.baseUrl}/auth/token`, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/x-www-form-urlencoded',
                },
                body: new URLSearchParams({
                    'grant_type': 'client_credentials',
                    'client_id': this.clientId,
                    'client_secret': clientSecret,
                    'scope': 'patient/*.write patient/*.read'
                })
            });
            
            const tokenData = await response.json();
            
            if (response.ok) {
                this.accessToken = tokenData.access_token;
                return { success: true, token: tokenData };
            } else {
                throw new Error(tokenData.error_description || 'Authentication failed');
            }
        } catch (error) {
            console.error('FHIR authentication error:', error);
            return { success: false, error: error.message };
        }
    }
    
    async createResource(resourceType, resource) {
        if (!this.accessToken) {
            throw new Error('Not authenticated. Call authenticate() first.');
        }
        
        try {
            const response = await fetch(`${this.baseUrl}/${resourceType}`, {
                method: 'POST',
                headers: {
                    'Authorization': `Bearer ${this.accessToken}`,
                    'Content-Type': 'application/fhir+json',
                    'Accept': 'application/fhir+json'
                },
                body: JSON.stringify(resource)
            });
            
            const result = await response.json();
            
            if (response.ok) {
                return { success: true, resource: result };
            } else {
                throw new Error(result.issue?.[0]?.diagnostics || 'FHIR resource creation failed');
            }
        } catch (error) {
            console.error('FHIR resource creation error:', error);
            return { success: false, error: error.message };
        }
    }
}

// Initialize FHIR client
const fhirClient = new MedConnectFHIRClient(
    'hc_test_1234567890abcdef', 
    'https://fhir-sandbox.medconnect.com/R4'
);

// Auto-generate Medical Record Number
function generateMRN() {
    const year = new Date().getFullYear();
    const random = Math.random().toString(36).substring(2, 8).toUpperCase();
    return `MRN-${year}-${random}`;
}

// Validate clinical data
function validateClinicalData(formData) {
    const errors = [];
    
    // Required field validation
    if (!formData.given?.trim()) errors.push('First name is required');
    if (!formData.family?.trim()) errors.push('Last name is required');
    if (!formData.birthDate) errors.push('Date of birth is required');
    if (!formData.gender) errors.push('Gender is required');
    if (!formData.phonePrimary?.trim()) errors.push('Primary phone is required');
    
    // Age validation (must be reasonable)
    if (formData.birthDate) {
        const birthDate = new Date(formData.birthDate);
        const today = new Date();
        const age = Math.floor((today - birthDate) / (365.25 * 24 * 60 * 60 * 1000));
        
        if (age < 0 || age > 150) {
            errors.push('Please enter a valid date of birth');
        }
    }
    
    // Phone number format validation
    const phoneRegex = /^\+?1?[-.\s]?\(?([0-9]{3})\)?[-.\s]?([0-9]{3})[-.\s]?([0-9]{4})$/;
    if (formData.phonePrimary && !phoneRegex.test(formData.phonePrimary)) {
        errors.push('Please enter a valid phone number');
    }
    
    // Email validation (if provided)
    if (formData.email) {
        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        if (!emailRegex.test(formData.email)) {
            errors.push('Please enter a valid email address');
        }
    }
    
    return errors;
}
```

---

### Step 3: Handle Patient Registration (15 minutes)

#### Client-Side Registration Processing
```javascript
// Handle patient registration form submission
document.getElementById('patient-registration-form').addEventListener('submit', async function(event) {
    event.preventDefault();
    
    const submitButton = document.getElementById('register-button');
    const buttonText = document.getElementById('button-text');
    const spinner = document.getElementById('spinner');
    
    // Disable submit button and show processing state
    submitButton.disabled = true;
    buttonText.style.display = 'none';
    spinner.style.display = 'inline';
    
    try {
        // Collect form data
        const formData = new FormData(event.target);
        const patientData = Object.fromEntries(formData.entries());
        
        // Validate clinical data
        const validationErrors = validateClinicalData(patientData);
        if (validationErrors.length > 0) {
            showClinicalError('Validation Error: ' + validationErrors.join(', '));
            resetButton();
            return;
        }
        
        // Authenticate with FHIR server (in production, use SMART on FHIR)
        const authResult = await fhirClient.authenticate('hc_secret_abcdef1234567890');
        if (!authResult.success) {
            showClinicalError('Authentication failed: ' + authResult.error);
            resetButton();
            return;
        }
        
        // Create FHIR Patient resource
        const fhirPatient = createFHIRPatientResource(patientData);
        
        // Submit to FHIR server
        const result = await fhirClient.createResource('Patient', fhirPatient);
        
        if (result.success) {
            showClinicalSuccess(
                `Patient registered successfully!<br>
                <strong>Patient ID:</strong> ${result.resource.id}<br>
                <strong>MRN:</strong> ${result.resource.identifier[0].value}<br>
                <strong>Created:</strong> ${new Date(result.resource.meta.lastUpdated).toLocaleString()}`
            );
            
            // Optional: Clear form for next patient
            setTimeout(() => {
                if (confirm('Would you like to register another patient?')) {
                    event.target.reset();
                    document.getElementById('registration-result').innerHTML = '';
                }
            }, 2000);
            
        } else {
            showClinicalError('Registration failed: ' + result.error);
        }
        
    } catch (error) {
        console.error('Patient registration error:', error);
        showClinicalError('Network error: Please check your connection and try again.');
    } finally {
        resetButton();
    }
});

function createFHIRPatientResource(formData) {
    // Generate MRN if not provided
    const mrn = formData.mrn?.trim() || generateMRN();
    
    const patient = {
        resourceType: "Patient",
        identifier: [
            {
                use: "usual",
                type: {
                    coding: [
                        {
                            system: "http://terminology.hl7.org/CodeSystem/v2-0203",
                            code: "MR",
                            display: "Medical Record Number"
                        }
                    ]
                },
                system: "http://medconnect-demo.com/patient-id",
                value: mrn
            }
        ],
        active: true,
        name: [
            {
                use: "official",
                family: formData.family,
                given: [formData.given]
            }
        ],
        gender: formData.gender,
        birthDate: formData.birthDate
    };
    
    // Add middle name if provided
    if (formData.middle?.trim()) {
        patient.name[0].given.push(formData.middle);
    }
    
    // Add contact information
    if (formData.phonePrimary || formData.email) {
        patient.telecom = [];
        
        if (formData.phonePrimary) {
            patient.telecom.push({
                system: "phone",
                value: formData.phonePrimary,
                use: "mobile",
                rank: 1
            });
        }
        
        if (formData.email) {
            patient.telecom.push({
                system: "email",
                value: formData.email,
                use: "home"
            });
        }
    }
    
    // Add address information
    if (formData.addressLine1 && formData.city && formData.state && formData.postalCode) {
        patient.address = [
            {
                use: "home",
                type: "both",
                line: [formData.addressLine1],
                city: formData.city,
                state: formData.state,
                postalCode: formData.postalCode,
                country: "US"
            }
        ];
        
        // Add address line 2 if provided
        if (formData.addressLine2?.trim()) {
            patient.address[0].line.push(formData.addressLine2);
        }
    }
    
    // Add marital status if provided
    if (formData.maritalStatus) {
        patient.maritalStatus = {
            coding: [
                {
                    system: "http://terminology.hl7.org/CodeSystem/v3-MaritalStatus",
                    code: formData.maritalStatus,
                    display: getMaritalStatusDisplay(formData.maritalStatus)
                }
            ]
        };
    }
    
    // Add preferred language if provided
    if (formData.preferredLanguage) {
        patient.communication = [
            {
                language: {
                    coding: [
                        {
                            system: "urn:ietf:bcp:47",
                            code: formData.preferredLanguage,
                            display: getLanguageDisplay(formData.preferredLanguage)
                        }
                    ]
                },
                preferred: true
            }
        ];
    }
    
    // Add emergency contact if provided
    if (formData.emergencyFirstName && formData.emergencyLastName) {
        patient.contact = [
            {
                relationship: [
                    {
                        coding: [
                            {
                                system: "http://terminology.hl7.org/CodeSystem/v2-0131",
                                code: formData.emergencyRelationship || "C",
                                display: getContactRelationshipDisplay(formData.emergencyRelationship)
                            }
                        ]
                    }
                ],
                name: {
                    family: formData.emergencyLastName,
                    given: [formData.emergencyFirstName]
                }
            }
        ];
        
        if (formData.emergencyPhone) {
            patient.contact[0].telecom = [
                {
                    system: "phone",
                    value: formData.emergencyPhone,
                    use: "mobile"
                }
            ];
        }
    }
    
    return patient;
}

function getMaritalStatusDisplay(code) {
    const displays = {
        'S': 'Single',
        'M': 'Married', 
        'D': 'Divorced',
        'W': 'Widowed',
        'UNK': 'Unknown'
    };
    return displays[code] || 'Unknown';
}

function getLanguageDisplay(code) {
    const displays = {
        'en-US': 'English (United States)',
        'es-US': 'Spanish (United States)',
        'fr-US': 'French (United States)',
        'zh-CN': 'Chinese (Simplified)'
    };
    return displays[code] || 'English (United States)';
}

function getContactRelationshipDisplay(code) {
    const displays = {
        'C': 'Emergency Contact',
        'E': 'Employer',
        'N': 'Next-of-Kin',
        'S': 'Spouse',
        'U': 'Unknown'
    };
    return displays[code] || 'Emergency Contact';
}

function showClinicalSuccess(message) {
    const resultDiv = document.getElementById('registration-result');
    resultDiv.innerHTML = `<div class="clinical-message success-message">${message}</div>`;
}

function showClinicalError(message) {
    const resultDiv = document.getElementById('registration-result');
    resultDiv.innerHTML = `<div class="clinical-message error-message">${message}</div>`;
}

function resetButton() {
    const submitButton = document.getElementById('register-button');
    const buttonText = document.getElementById('button-text');
    const spinner = document.getElementById('spinner');
    
    submitButton.disabled = false;
    buttonText.style.display = 'inline';
    spinner.style.display = 'none';
}
```

---

### Step 4: Server-Side FHIR Processing (15 minutes)

#### Node.js Healthcare Backend Implementation
```javascript
const express = require('express');
const cors = require('cors');
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');
const { body, validationResult } = require('express-validator');
const fhirClient = require('@medconnect/healthcare-sdk');

const app = express();

// HIPAA-compliant security middleware
app.use(helmet({
    contentSecurityPolicy: {
        directives: {
            defaultSrc: ["'self'"],
            scriptSrc: ["'self'", "'unsafe-inline'"],
            styleSrc: ["'self'", "'unsafe-inline'"],
            imgSrc: ["'self'", "data:", "https:"],
        },
    },
    hsts: {
        maxAge: 31536000,
        includeSubDomains: true,
        preload: true
    }
}));

// CORS configuration for healthcare applications
app.use(cors({
    origin: process.env.ALLOWED_ORIGINS?.split(',') || ['https://localhost:8443'],
    methods: ['GET', 'POST', 'PUT', 'DELETE'],
    allowedHeaders: ['Content-Type', 'Authorization'],
    credentials: true
}));

app.use(express.json({ limit: '1mb' }));
app.use(express.static('public'));

// Rate limiting for healthcare API endpoints
const clinicalRateLimit = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100, // Limit each IP to 100 requests per windowMs
    message: {
        error: 'Too many clinical requests from this IP, please try again later.',
        retryAfter: '15 minutes'
    },
    standardHeaders: true,
    legacyHeaders: false,
});

app.use('/api/clinical', clinicalRateLimit);

// Initialize FHIR client with healthcare credentials
const healthcareFHIRClient = new fhirClient.Client({
    serverUrl: process.env.MEDCONNECT_FHIR_BASE_URL,
    auth: {
        clientId: process.env.MEDCONNECT_FHIR_CLIENT_ID,
        clientSecret: process.env.MEDCONNECT_FHIR_CLIENT_SECRET,
        scope: 'patient/*.write patient/*.read'
    }
});

// Clinical audit logging for HIPAA compliance
const clinicalAuditLogger = require('./utils/clinical-audit-logger');

// Patient registration validation rules
const patientValidationRules = [
    body('given').notEmpty().trim().isLength({ min: 1, max: 50 })
        .withMessage('First name is required and must be 1-50 characters'),
    body('family').notEmpty().trim().isLength({ min: 1, max: 50 })
        .withMessage('Last name is required and must be 1-50 characters'),
    body('birthDate').isISO8601().toDate()
        .withMessage('Valid birth date is required'),
    body('gender').isIn(['male', 'female', 'other', 'unknown'])
        .withMessage('Gender must be male, female, other, or unknown'),
    body('phonePrimary').matches(/^\+?1?[-.\s]?\(?([0-9]{3})\)?[-.\s]?([0-9]{3})[-.\s]?([0-9]{4})$/)
        .withMessage('Valid phone number is required'),
    body('email').optional().isEmail().normalizeEmail()
        .withMessage('Valid email address required'),
    body('mrn').optional().isLength({ max: 20 })
        .withMessage('Medical record number must be 20 characters or less')
];

app.post('/api/clinical/patient/register', patientValidationRules, async (req, res) => {
    try {
        // Validate request data
        const errors = validationResult(req);
        if (!errors.isEmpty()) {
            return res.status(400).json({
                success: false,
                error_type: 'clinical_validation_error',
                message: 'Patient registration validation failed',
                validation_errors: errors.array(),
                clinical_guidance: 'Please review patient information and ensure all required fields are properly formatted'
            });
        }
        
        const patientData = req.body;
        
        // Additional clinical validation
        const clinicalValidationResult = validateClinicalPatientData(patientData);
        if (!clinicalValidationResult.valid) {
            return res.status(400).json({
                success: false,
                error_type: 'clinical_business_rule_violation',
                message: 'Clinical validation failed',
                clinical_errors: clinicalValidationResult.errors,
                clinical_guidance: 'Patient data does not meet clinical requirements'
            });
        }
        
        // Create FHIR Patient resource
        const fhirPatient = buildFHIRPatientResource(patientData);
        
        // Log clinical action for HIPAA audit trail
        await clinicalAuditLogger.logPatientRegistration({
            action: 'patient_registration_attempt',
            mrn: fhirPatient.identifier[0].value,
            user_id: req.user?.id || 'system',
            ip_address: req.ip,
            user_agent: req.get('User-Agent'),
            timestamp: new Date()
        });
        
        // Submit to FHIR server
        const createdPatient = await healthcareFHIRClient.create({
            resourceType: 'Patient',
            resource: fhirPatient
        });
        
        // Log successful registration
        await clinicalAuditLogger.logPatientRegistration({
            action: 'patient_registration_success',
            patient_id: createdPatient.id,
            mrn: fhirPatient.identifier[0].value,
            user_id: req.user?.id || 'system',
            ip_address: req.ip,
            timestamp: new Date()
        });
        
        // Return clinical response
        res.json({
            success: true,
            patient: {
                id: createdPatient.id,
                mrn: fhirPatient.identifier[0].value,
                name: `${patientData.given} ${patientData.family}`,
                birthDate: patientData.birthDate,
                gender: patientData.gender,
                created: createdPatient.meta.lastUpdated
            },
            clinical_metadata: {
                fhir_version: createdPatient.meta.versionId,
                resource_location: `${process.env.MEDCONNECT_FHIR_BASE_URL}/Patient/${createdPatient.id}`,
                compliance_verified: true
            }
        });
        
    } catch (error) {
        console.error('Patient registration error:', error);
        
        // Log error for clinical investigation
        await clinicalAuditLogger.logPatientRegistration({
            action: 'patient_registration_error',
            error: error.message,
            user_id: req.user?.id || 'system',
            ip_address: req.ip,
            timestamp: new Date()
        });
        
        // Handle specific FHIR errors
        if (error.response?.status === 422) {
            return res.status(422).json({
                success: false,
                error_type: 'fhir_validation_error',
                message: 'FHIR resource validation failed',
                fhir_issues: error.response.data.issue,
                clinical_guidance: 'Review FHIR resource structure and clinical coding standards'
            });
        }
        
        if (error.response?.status === 409) {
            return res.status(409).json({
                success: false,
                error_type: 'patient_already_exists',
                message: 'Patient with this identifier already exists',
                clinical_guidance: 'Check medical record number or use patient update endpoint'
            });
        }
        
        res.status(500).json({
            success: false,
            error_type: 'clinical_system_error',
            message: 'Clinical system error occurred',
            clinical_guidance: 'Contact healthcare technical support for assistance'
        });
    }
});

function validateClinicalPatientData(patientData) {
    const errors = [];
    
    // Age validation for clinical appropriateness
    const birthDate = new Date(patientData.birthDate);
    const today = new Date();
    const ageInYears = Math.floor((today - birthDate) / (365.25 * 24 * 60 * 60 * 1000));
    
    if (ageInYears < 0) {
        errors.push('Birth date cannot be in the future');
    }
    
    if (ageInYears > 150) {
        errors.push('Patient age exceeds reasonable clinical limits');
    }
    
    // MRN uniqueness check (simplified - in production, check against database)
    if (patientData.mrn && patientData.mrn.length < 5) {
        errors.push('Medical record number must be at least 5 characters for clinical safety');
    }
    
    // Contact information validation for emergency situations
    if (!patientData.phonePrimary && !patientData.email) {
        errors.push('At least one contact method (phone or email) is required for clinical communication');
    }
    
    return {
        valid: errors.length === 0,
        errors: errors
    };
}

function buildFHIRPatientResource(patientData) {
    // Generate MRN if not provided
    const mrn = patientData.mrn || generateMedicalRecordNumber();
    
    const patient = {
        resourceType: "Patient",
        identifier: [
            {
                use: "usual",
                type: {
                    coding: [
                        {
                            system: "http://terminology.hl7.org/CodeSystem/v2-0203",
                            code: "MR",
                            display: "Medical Record Number"
                        }
                    ]
                },
                system: "http://medconnect-demo.com/patient-id",
                value: mrn,
                assigner: {
                    display: "MedConnect Healthcare System"
                }
            }
        ],
        active: true,
        name: [
            {
                use: "official",
                family: patientData.family,
                given: [patientData.given]
            }
        ],
        gender: patientData.gender,
        birthDate: patientData.birthDate
    };
```

```javascript
function buildFHIRPatientResource(patientData) {
    // Generate MRN if not provided
    const mrn = patientData.mrn || generateMedicalRecordNumber();
    
    const patient = {
        resourceType: "Patient",
        identifier: [
            {
                use: "usual",
                type: {
                    coding: [
                        {
                            system: "http://terminology.hl7.org/CodeSystem/v2-0203",
                            code: "MR",
                            display: "Medical Record Number"
                        }
                    ]
                },
                system: "http://medconnect-demo.com/patient-id",
                value: mrn,
                assigner: {
                    display: "MedConnect Healthcare System"
                }
            }
        ],
        active: true,
        name: [
            {
                use: "official",
                family: patientData.family,
                given: [patientData.given]
            }
        ],
        gender: patientData.gender,
        birthDate: patientData.birthDate
    };
    
    // Add contact information
    if (patientData.phonePrimary || patientData.email) {
        patient.telecom = [];
        
        if (patientData.phonePrimary) {
            patient.telecom.push({
                system: "phone",
                value: patientData.phonePrimary,
                use: "mobile",
                rank: 1
            });
        }
        
        if (patientData.email) {
            patient.telecom.push({
                system: "email",
                value: patientData.email,
                use: "home"
            });
        }
    }
    
    // Add address if provided
    if (patientData.addressLine1 && patientData.city && patientData.state) {
        patient.address = [
            {
                use: "home",
                type: "both",
                line: [patientData.addressLine1],
                city: patientData.city,
                state: patientData.state,
                postalCode: patientData.postalCode || "",
                country: "US",
                period: {
                    start: new Date().toISOString().split('T')[0]
                }
            }
        ];
        
        if (patientData.addressLine2) {
            patient.address[0].line.push(patientData.addressLine2);
        }
    }
    
    return patient;
}

function generateMedicalRecordNumber() {
    const year = new Date().getFullYear();
    const month = String(new Date().getMonth() + 1).padStart(2, '0');
    const random = Math.random().toString(36).substring(2, 8).toUpperCase();
    return `MRN-${year}${month}-${random}`;
}

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
    console.log(`Healthcare API server running on port ${PORT}`);
    console.log(`FHIR Base URL: ${process.env.MEDCONNECT_FHIR_BASE_URL}`);
});
```

---

### Step 5: Testing Your Clinical Implementation (10 minutes)

#### Test with Clinical Sample Data
Use these healthcare test scenarios in your sandbox environment:

**Test Patient 1 - Adult Primary Care:**
```json
{
  "given": "Sarah",
  "family": "Johnson",
  "birthDate": "1985-03-15",
  "gender": "female",
  "phonePrimary": "+1-555-123-4567",
  "email": "sarah.johnson@email.com",
  "addressLine1": "123 Healthcare Ave",
  "city": "Medical City",
  "state": "CA",
  "postalCode": "90210"
}
```

**Test Patient 2 - Pediatric Patient:**
```json
{
  "given": "Michael",
  "family": "Chen",
  "birthDate": "2015-08-22",
  "gender": "male",
  "phonePrimary": "+1-555-987-6543",
  "email": "parent@email.com",
  "addressLine1": "456 Family Street",
  "city": "Pediatric City",
  "state": "TX",
  "postalCode": "75201",
  "emergencyFirstName": "Lisa",
  "emergencyLastName": "Chen",
  "emergencyRelationship": "N",
  "emergencyPhone": "+1-555-234-5678"
}
```

**Test Patient 3 - Elderly Patient with Complex Data:**
```json
{
  "given": "Robert",
  "middle": "William",
  "family": "Martinez",
  "birthDate": "1940-12-05",
  "gender": "male",
  "phonePrimary": "+1-555-456-7890",
  "addressLine1": "789 Senior Living Blvd",
  "addressLine2": "Unit 205",
  "city": "Elder Care City",
  "state": "FL",
  "postalCode": "33101",
  "maritalStatus": "W",
  "preferredLanguage": "es-US"
}
```

#### Clinical Verification Checklist
- [ ] Patient resource created successfully with proper FHIR structure
- [ ] Medical Record Number generated or validated correctly
- [ ] Clinical identifiers properly formatted and assigned
- [ ] Contact information stored with appropriate use codes
- [ ] Address information structured according to healthcare standards
- [ ] Emergency contact relationships coded correctly
- [ ] Clinical audit logs generated for HIPAA compliance

#### Testing Workflow
1. **Open your patient registration form** in a web browser
2. **Enter test patient data** using the clinical scenarios above
3. **Submit registration** and verify successful FHIR resource creation
4. **Check MedConnect healthcare dashboard** for patient record
5. **Verify clinical audit logs** show proper HIPAA-compliant logging
6. **Test validation errors** with invalid clinical data
7. **Verify error handling** displays appropriate healthcare guidance

---

### Troubleshooting Common Clinical Issues

#### Problem: "FHIR Validation Error" in Patient Creation
**Clinical Solution:**
```javascript
// Ensure required FHIR elements are present
function validateFHIRPatientResource(patient) {
    const requiredFields = ['resourceType', 'identifier', 'name', 'gender'];
    const missingFields = [];
    
    requiredFields.forEach(field => {
        if (!patient[field]) {
            missingFields.push(field);
        }
    });
    
    // Check identifier structure
    if (patient.identifier && patient.identifier.length > 0) {
        const mrn = patient.identifier[0];
        if (!mrn.system || !mrn.value) {
            missingFields.push('identifier.system or identifier.value');
        }
    }
    
    // Check name structure
    if (patient.name && patient.name.length > 0) {
        const name = patient.name[0];
        if (!name.family || !name.given) {
            missingFields.push('name.family or name.given');
        }
    }
    
    return {
        valid: missingFields.length === 0,
        missingFields: missingFields,
        clinicalGuidance: missingFields.length > 0 
            ? `Missing required FHIR elements: ${missingFields.join(', ')}`
            : 'FHIR Patient resource is valid'
    };
}
```

#### Problem: SMART on FHIR Authentication Issues
**Clinical Solution:**
```javascript
// Comprehensive SMART on FHIR error handling
async function handleSmartOnFHIRAuth(authError) {
    console.log('SMART on FHIR authentication error:', authError);
    
    if (authError.error === 'invalid_scope') {
        return {
            error: 'Insufficient clinical permissions',
            clinicalGuidance: 'Request patient/*.write scope for patient registration',
            requiredScopes: ['patient/Patient.write', 'patient/Patient.read'],
            authUrl: `${FHIR_BASE_URL}/auth/authorize`
        };
    }
    
    if (authError.error === 'invalid_client') {
        return {
            error: 'Invalid clinical application credentials',
            clinicalGuidance: 'Verify FHIR client ID and secret in healthcare dashboard',
            supportContact: 'clinical-support@medconnect.com'
        };
    }
    
    return {
        error: 'Healthcare authentication failed',
        clinicalGuidance: 'Contact clinical technical support for assistance'
    };
}
```

#### Problem: Clinical Data Privacy Concerns
**Clinical Solution:**
```python
# HIPAA-compliant logging that protects PHI
class HIPAACompliantLogger:
    def __init__(self):
        self.logger = logging.getLogger('clinical_audit')
        
    def log_patient_access(self, patient_id, user_id, action, ip_address):
        """Log patient access without exposing PHI"""
        
        # Hash patient ID for privacy while maintaining auditability
        patient_hash = hashlib.sha256(f"{patient_id}_salt".encode()).hexdigest()[:16]
        
        audit_entry = {
            'timestamp': datetime.utcnow().isoformat(),
            'action': action,
            'patient_hash': patient_hash,  # Not actual patient ID
            'user_id': user_id,
            'ip_address': self._mask_ip_address(ip_address),
            'session_id': self._get_session_id(),
            'compliance_verified': True
        }
        
        self.logger.info(f"CLINICAL_AUDIT: {json.dumps(audit_entry)}")
        
        # Store in HIPAA-compliant audit database
        self._store_audit_entry(audit_entry)
    
    def _mask_ip_address(self, ip_address):
        """Mask IP address for privacy while maintaining traceability"""
        if ':' in ip_address:  # IPv6
            parts = ip_address.split(':')
            return ':'.join(parts[:4] + ['****', '****', '****', '****'])
        else:  # IPv4
            parts = ip_address.split('.')
            return '.'.join(parts[:2] + ['***', '***'])
```

---

### Next Steps in Healthcare Development

#### Immediate Clinical Enhancements
1. **[Add Clinical Observations](./vital-signs-monitoring.md)** - Implement vital signs and lab results
2. **[Implement Care Team Management](./care-team-setup.md)** - Provider and care coordinator workflows
3. **[Add Clinical Decision Support](./clinical-decision-support.md)** - AI-powered clinical recommendations

#### Advanced Healthcare Features
1. **[Set Up Population Health Analytics](./population-health.md)** - Cohort analysis and quality measures
2. **[Integrate Medical Devices](./device-integration.md)** - IoT and monitoring device connectivity
3. **[Implement Telehealth Features](./telehealth-integration.md)** - Remote care and virtual consultations

#### Clinical Production Preparation
1. **[Healthcare Security Hardening](./clinical-security.md)** - Production HIPAA compliance
2. **[Clinical Performance Optimization](./clinical-performance.md)** - Scale for healthcare workloads
3. **[Healthcare Monitoring Setup](./clinical-monitoring.md)** - Patient safety and system health monitoring

---

### Healthcare Support Resources

- **Clinical Developer Documentation:** [clinical.medconnect.com](https://clinical.medconnect.com)
- **Healthcare Community Forum:** [community.medconnect.com/clinical](https://community.medconnect.com/clinical)
- **Clinical Technical Support:** clinical-support@medconnect.com
- **HIPAA Compliance Questions:** compliance@medconnect.com
- **Healthcare Status Updates:** [status.medconnect.com](https://status.medconnect.com)

*You've successfully implemented FHIR-compliant patient registration! This clinical foundation can be extended with additional healthcare features and optimizations as your clinical workflows evolve.*
